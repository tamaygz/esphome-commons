# Quick Fix Guide for ESPHome Commons

This document provides concrete fixes for all critical issues found in the code review.

---

## 🚨 CRITICAL FIXES (DO THESE FIRST)

### Fix 1: Remove !extend from motion_sensor.yaml

**File:** `binary_sensor/motion_sensor.yaml`

**Current Code (BROKEN):**
```yaml
binary_sensor:
  - id: !extend ${sensor_id}
    pin:
      inverted: false
    filters:
      - delayed_on_off: 
          time_on: ${mtime_on}
          time_off: ${mtime_off}
```

**Fixed Code:**
```yaml
# Don't use !extend - instead, create a complete sensor definition
binary_sensor:
  - platform: gpio
    id: ${sensor_id}
    name: "${sensor_name}"
    device_class: motion
    pin:
      number: ${sensor_pin}
      mode: INPUT_PULLUP
      inverted: false  # Override for motion sensor
    filters:
      - delayed_on_off: 
          time_on: ${mtime_on}
          time_off: ${mtime_off}
    on_press:
      - script.execute: ${motion_start_callback_script}
    on_release:
      - script.execute: ${motion_end_callback_script}

script:
  - id: ${sensor_id}_signal_start_dummy
    then:
      - logger.log:
          format: "[START] No script was executed but the dummy instead."
          tag: ${sensor_id}
  - id: ${sensor_id}_signal_stop_dummy
    then:
      - logger.log:
          format: "[STOP] No script was executed but the dummy instead."
          tag: ${sensor_id}
```

---

### Fix 2: Remove !extend from maximumactive.yaml

**File:** `features/maximumactive.yaml`

**Current Code (BROKEN):**
```yaml
switch:
  - id: !extend ${monitored_switch}
    on_turn_on:
      - script.stop: maxactive_watchdog
      # ... rest of actions
```

**Fixed Code - Option A (Recommended):**
Document that this feature requires the monitored switch to be defined with specific actions.

**Fixed Code - Option B (Alternative):**
Create the switch entirely within this package:

```yaml
substitutions:
  monitored_switch_id: relay_switch
  monitored_switch_name: "Relay"
  monitored_switch_pin: GPIO0

switch:
  - platform: gpio
    id: ${monitored_switch_id}
    name: ${monitored_switch_name}
    pin: ${monitored_switch_pin}
    on_turn_on:
      - script.stop: maxactive_watchdog
      - globals.set:
          id: maxactive_last_activation
          value: !lambda "return id(ha_time).now().timestamp;"
      # ... rest of on_turn_on actions
    on_turn_off:
      - script.stop: maxactive_watchdog
```

---

### Fix 3: Fix scheduler.yaml variable naming

**File:** `features/scheduler.yaml`

**Lines to change:**
```yaml
# Line 60-66: Change from weekly_schedule to ${id}_weekly_schedule
- lambda: |-
    id(${id}_weekly_schedule)[0] = monday;
    id(${id}_weekly_schedule)[1] = tuesday;
    id(${id}_weekly_schedule)[2] = wednesday;
    id(${id}_weekly_schedule)[3] = thursday;
    id(${id}_weekly_schedule)[4] = friday;
    id(${id}_weekly_schedule)[5] = saturday;
    id(${id}_weekly_schedule)[6] = sunday;

# Line 75: Already correct
- lambda: "id(${id}_weekly_schedule)[0] = x.c_str();"

# Line 136: Change from weekly_schedule to ${id}_weekly_schedule
std::string schedule = id(${id}_weekly_schedule)[current_day];
```

---

### Fix 4: Remove invalid input_text from scheduler.yaml

**File:** `features/scheduler.yaml`

**Remove lines 17-38:**
```yaml
# DELETE THIS ENTIRE SECTION - it's not valid ESPHome YAML
input_text:
  ${id}_monday_schedule:
    name: ${friendly_name} Monday Schedule
    icon: mdi:calendar
  # ... all the other days ...
```

**Add this comment at the top instead:**
```yaml
# SETUP REQUIRED IN HOME ASSISTANT:
# Create the following input_text helpers in Home Assistant:
#   - input_text.${id}_monday_schedule
#   - input_text.${id}_tuesday_schedule
#   - input_text.${id}_wednesday_schedule
#   - input_text.${id}_thursday_schedule
#   - input_text.${id}_friday_schedule
#   - input_text.${id}_saturday_schedule
#   - input_text.${id}_sunday_schedule
#
# Format for schedule: "HH:MM-HH:MM,HH:MM-HH:MM"
# Example: "08:00-12:00,14:00-18:00" for on from 8am-12pm and 2pm-6pm
```

---

### Fix 5: Fix wifi.yaml typo

**File:** `wifi.yaml`

**Line 31:**
```yaml
# Change from:
name: WIIF IP Address

# To:
name: WIFI IP Address
```

---

### Fix 6: Fix web_server.yaml security

**File:** `web_server.yaml`

**Current (INSECURE):**
```yaml
substitutions:
  web_user: admin
  web_password: replaceme123

web_server:
  auth:
    username: ${web_user}
    password: ${web_password}
  local: true
```

**Fixed (SECURE):**
```yaml
# No default passwords! User must provide via secrets
web_server:
  auth:
    username: !secret web_username
    password: !secret web_password
  local: true
```

---

### Fix 7: Fix example file paths

**File:** `example-esp-devices/thermostat-control.yaml`

**Change all paths from:**
```yaml
packages:
  base: !include common/device_base_esp8266.yaml
  web_server: !include common/web_server.yaml
  sleepmode: !include
    file: common/features/sleepfunc.yaml
```

**To:**
```yaml
packages:
  base: !include ../device_base_esp8266.yaml
  web_server: !include ../web_server.yaml
  sleepmode: !include
    file: ../features/sleepfunc.yaml
```

---

### Fix 8: Fix .gitignore typo

**File:** `.gitignore`

**Line 6:**
```
# Change from:
/.vsclde/

# To:
/.vscode/
```

---

## ⚠️ HIGH PRIORITY FIXES

### Fix 9: Fix duplicate script IDs

**File:** `binary_sensor/gpio_sensor.yaml`

**Lines 27-36, change from:**
```yaml
script:
  - id: signal_start_dummy
    then:
      - logger.log:
          format: "[START] No script was executed but the dummy instead."
          tag: ${sensor_id}
  - id: signal_stop_dummy
    then:
      - logger.log:
          format: "[STOP] No script was executed but the dummy instead."
          tag: ${sensor_id}
```

**To:**
```yaml
script:
  - id: ${sensor_id}_signal_start_dummy
    then:
      - logger.log:
          format: "[START] No script was executed but the dummy instead."
          tag: ${sensor_id}
  - id: ${sensor_id}_signal_stop_dummy
    then:
      - logger.log:
          format: "[STOP] No script was executed but the dummy instead."
          tag: ${sensor_id}
```

**Also update lines 6-7:**
```yaml
substitutions:
  signal_start_callback_script: ${sensor_id}_signal_start_dummy
  signal_end_callback_script: ${sensor_id}_signal_stop_dummy
```

---

### Fix 10: Replace deprecated function in intervalinterpreter.yaml

**File:** `features/intervalinterpreter.yaml`

**Lines 72, 85:**

**Change from:**
```cpp
int local_start_time = system_get_time();
int release_time = system_get_time();
```

**To:**
```cpp
unsigned long local_start_time = micros();
unsigned long release_time = micros();
```

**Also update variable types in globals (lines 12-19):**
```yaml
globals:
  - id: ${id}_start_time
    type: unsigned long  # Changed from int
    restore_value: no
    initial_value: "0"
  - id: ${id}_last_input_time
    type: unsigned long  # Changed from int
    restore_value: no
    initial_value: "0"
```

---

## 📋 Testing Checklist

After making these fixes, test each component:

- [ ] Test gpio_sensor.yaml independently
- [ ] Test motion_sensor.yaml
- [ ] Test relay01s.yaml  
- [ ] Test maximumactive.yaml with a switch
- [ ] Test sleepfunc.yaml
- [ ] Test scheduler.yaml (after creating HA input_text helpers)
- [ ] Test intervalinterpreter.yaml
- [ ] Test thermostat-control.yaml example
- [ ] Test doorbell-interceptor.yaml example
- [ ] Verify no hardcoded passwords
- [ ] Verify all secrets are in secrets.yaml
- [ ] Verify examples compile
- [ ] Test on actual hardware if possible

---

## 🎯 Priority Order

1. **Fix !extend issues** (Fixes 1-2) - Won't work at all otherwise
2. **Fix scheduler.yaml** (Fixes 3-4) - Won't compile otherwise
3. **Fix paths** (Fix 7) - Examples won't work otherwise
4. **Fix security** (Fix 6) - Critical vulnerability
5. **Fix typos** (Fixes 5, 8) - Quick wins
6. **Fix duplicate IDs** (Fix 9) - Prevents multi-sensor configs
7. **Fix deprecated functions** (Fix 10) - Future compatibility

---

## ⏱️ Estimated Time per Fix

- Fixes 1-2 (!extend removal): 2-3 hours total
- Fixes 3-4 (scheduler): 30 minutes
- Fixes 5, 8 (typos): 5 minutes
- Fix 6 (security): 30 minutes
- Fix 7 (paths): 15 minutes
- Fix 9 (duplicate IDs): 15 minutes
- Fix 10 (deprecated function): 30 minutes

**Total: ~4-5 hours of work**

---

## 📝 Additional Recommendations

### Create secrets.yaml.example

Add this file to the repository root:

```yaml
# Required secrets for ESPHome Commons
# Copy this file to secrets.yaml and fill in your values

# WiFi Configuration
wifi_ssid: "your_wifi_ssid"
wifi_password: "your_wifi_password"
wifi_alternative_ssid: "your_backup_wifi_ssid"
hotspot_password: "your_fallback_hotspot_password"

# OTA Updates
ota_password: "your_ota_password"

# API Encryption
api_ekey: "your_32_character_base64_encryption_key"

# Web Server Authentication
web_username: "admin"
web_password: "your_secure_web_password"
```

---

*This guide provides all the specific code changes needed to fix the critical issues found in the review.*
