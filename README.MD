# ESPHome Commons

This repository contains common configurations and packages for ESPHome devices. These configurations can be included in your ESPHome projects to simplify setup and ensure consistency across multiple devices.

## Usage

To use these common configurations in your ESPHome project, include the relevant YAML files in your device configuration.

### Example

Here is an example of how to include the common configurations in your ESPHome device configuration:

```yaml
substitutions:
  name: my_device
  friendly_name: My Device
  platform: esp8266
  board: nodemcuv2
  wifi_ssid: "your_wifi_ssid"
  wifi_password: "your_wifi_password"
  api_password: "your_api_password"
  logger_level: DEBUG
  web_server_port: 80
  time_id: homeassistant_time
  sleep_duration: 60s

packages:
  base: !include common/device_base_esp8266.yaml
  wifi: !include common/wifi.yaml
  api: !include common/api.yaml
  logger: !include common/logger.yaml
  web_server: !include common/web_server.yaml
  time: !include common/time.yaml
  sleepfunc: !include common/features/sleepfunc.yaml
```

For more examples check /examples

## Available Configurations

### Common Defaults

- **File:** `common/common_defaults.yaml`
- **Description:** Contains default substitutions for device name, friendly name, version, log level, and log baud rate.

### Device Base

- **File:** `common/device_base.yaml`
- **Description:** Base configuration for ESPHome devices, including common packages like WiFi, API, logger, status sensor, restart button, version sensor, and time.

### Device Base for ESP32

- **File:** `common/device_base_esp32.yaml`
- **Description:** Base configuration for ESP32 devices (`board` e.g. `esp32dev`) and common packages.

### Device Base for ESP8266

- **File:** `common/device_base_esp8266.yaml`
- **Description:** Base configuration for ESP8266 devices, including platform-specific settings and common packages.

### WiFi Configuration

- **File:** `common/wifi.yaml`
- **Description:** WiFi configuration with support for multiple networks, fallback hotspot, OTA updates, and WiFi signal sensors.
- **Variables:**
  - `wifi_ssid`: Your WiFi SSID
  - `wifi_password`: Your WiFi password

### API Configuration

- **File:** `common/api.yaml`
- **Description:** Home Assistant API configuration with encryption.
- **Variables:**
  - `api_password`: Your API password

### Logger Configuration

- **File:** `common/logger.yaml`
- **Description:** Logger configuration with customizable log level and baud rate.
- **Variables:**
  - `logger_level`: Log level (e.g., DEBUG, INFO, WARN, ERROR)

### Web Server Configuration

- **File:** `common/web_server.yaml`
- **Description:** Web server configuration with authentication.
- **Variables:**
  - `web_server_port`: Port for the web server

### Time Configuration

- **File:** `common/time.yaml`
- **Description:** Time configuration using Home Assistant as the time source.
- **Variables:**
  - `time_id`: ID for the time component

### Binary Sensors

- **Files:** `common/binary_sensor/gpio_sensor.yaml`, `common/binary_sensor/status.yaml`
- **Description:** Configurations for GPIO and status binary sensors.

### Motion Sensor and Endstop

- **Files:** `common/binary_sensor/motion_sensor.yaml`, `common/binary_sensor/endstop.yaml`
- **Description:**
  - `motion_sensor.yaml`: PIR/radar motion sensor with runtime configurable (HA) trigger/clear delays, an `Enabled` switch and diagnostics (`Last Detected` timestamp, `Last Duration`, `Detections` counter).
  - `endstop.yaml`: GPIO endstop/limit switch/hall sensor, typically wired to the stepper home signal.
- **Variables:** (motion) `mtime_on`, `mtime_off` initial trigger/clear delay in **milliseconds**; `motion_start_callback_script`, `motion_end_callback_script`; (endstop) `endstop_pin`, `endstop_inverted`, `endstop_press_callback_script`, `endstop_release_callback_script`.
- The default callback scripts are now named `<sensor_id>_signal_start_dummy` / `<sensor_id>_signal_stop_dummy`, so multiple GPIO sensors can be used in one device.

### Stepper Motors

- **Files:** `common/stepper/uln2003.yaml`, `common/stepper/a4988.yaml` (hardware drivers), `common/stepper/stepper_control.yaml` (generic control layer, included by the drivers), `common/features/stepper_homing.yaml` (optional homing)
- **Description:** Full stepper support built from reusable pieces:
  - **Configurable from HA (config category, restored after reboot):** max speed, acceleration, deceleration, move step, soft limit min/max, soft limits on/off, two preset positions, hold torque (ULN2003), homing speed / max distance / backoff / direction / home position, home on boot.
  - **Control:** target position number, move forward/backward by *Move Step*, stop, go to min/max/zero/preset 1/preset 2, set current position as zero. Scripts `<id>_move_to`, `<id>_move_relative`, `<id>_stop`, `<id>_set_position` can be used in your own automations.
  - **Sensors/diagnostics:** current position, angle, steps remaining, moving, status (Idle/Moving/Homing), homed, last moved, total steps travelled (odometer), home signal, homing result, last homed.
  - The last position is persisted after each move and restored on boot (open loop steppers have no position feedback, so re-home if the shaft may have been moved by hand or power was lost mid-move).
- **Homing:** *Manual*: jog to the desired spot and press `Set Home Here`. *Automatic*: press `Home` (or enable `Home On Boot`); the motor searches at homing speed until the **home signal** is asserted, backs off and defines this as home position. Any source can be the home signal: call the scripts `<id>_home_signal_on` / `<id>_home_signal_off` (e.g. via `endstop_press_callback_script`, `motion_start_callback_script`, or a HA automation).
- **Variables:** `stepper_id`, `stepper_name`, pins (`stepper_pin_a..d` / `stepper_step_pin`, `stepper_dir_pin`), `stepper_position_min/max`, `stepper_max_speed`, `stepper_acceleration`, `stepper_deceleration`, `stepper_steps_per_revolution`, `stepper_preset_1/2`, and the `stepper_homing_*` variables documented in `features/stepper_homing.yaml`.
- **Example:** `example-esp-devices/halloween-stepper.yaml`. Requires `time.yaml` (included by the device base) for the timestamp sensors.

### Stepper Facing (move/face to target source)

- **Files:** `common/features/stepper_facing.yaml` (once per stepper), `common/features/stepper_target_source.yaml` (once per source, 0..n times)
- **Description:** The stepper turns to the position recorded for whichever *target source* signals. A source is a motion sensor on the device (`motion_start_callback_script: <stepper_id>_face_<target_id>`, `motion_end_callback_script: <stepper_id>_release_<target_id>`) or an external trigger (HA action `esphome.<node>_<stepper_id>_face_source` with `source: <target_id>`).
- **Teaching positions (interactive, persisted):** `Teach Start` → move the stepper to face the source → `Teach Confirm` (next source is prompted; `Teach Skip` keeps the stored position, `Teach Cancel` aborts). `Teach Prompt` shows what is asked. Also available as API actions `<stepper_id>_teach_start/_confirm/_skip/_cancel`; each source also has a `Record <name> Position` button and editable `Position`/`Taught` entities. Facing is suspended while teaching.
- **Safety net:** with `Face Target Source` enabled, sources that were never taught are not moved to; a warning is logged (boot and every blocked trigger), `Teach Required` (problem) turns on and `Facing Status` reports it (Teaching/Disabled/Teach required/Ready).
- **Facing behaviour (all configurable from HA, restored after reboot):**
  - `Face Interruptible` (default off): while a facing action runs (from the signal until the target – or, with *Return To Idle*, the idle position – is reached) signals from other sources are ignored; on = a new signal takes over immediately.
  - `Cooldown Min/Max` (seconds): after an action is done new signals are ignored for this time; a fixed time, or random in the range when Max > Min.
  - `Return To Idle` + `Return Mode`: *Motion Clear* (return when the source calls `<stepper_id>_release_<target_id>`, e.g. `motion_end_callback_script`) or *Timer* (after `Hold Time Min/Max`, random in the range). `Return Timeout` is a safety limit for sources that never clear (0 = off). `Idle Position` number, `Set Idle Position Here` and `Return To Idle Now` buttons (API action `<stepper_id>_face_return_now`).
  - `Face Speed/Acceleration/Deceleration` and `Idle Speed/Acceleration/Deceleration`: separate motion profiles for facing the target and resuming idle (0 = general stepper setting).
- **State/events for HA:** text sensor `Facing State` (`idle`, `moving_to_target`, `facing_target`, `resuming_idle`, `aborted`, `disabled`), `Facing Target` (source being faced) and the HA event `esphome.<stepper_id>_facing` (`event_type` = `state_changed`/`signal_ignored`, `state`, `previous_state`, `target`, `target_name`, `reason`, `position`) to trigger HA automations. Events require *Allow the device to perform Home Assistant actions* in the ESPHome integration.
- **Other entities:** `Taught Sources`, `Registered Sources`, `Last Faced Source`, `Last Taught`, `Facing Cooldown`/`Remaining`, per-source `Last Triggered`.
- **Variables:** `stepper_facing_enabled`, `stepper_facing_interruptible`, `stepper_facing_cooldown_min/_max`, `stepper_return_idle`, `stepper_return_mode`, `stepper_return_hold_min/_max`, `stepper_return_timeout`, `stepper_idle_position`, `stepper_face_*` / `stepper_idle_*` (speed, acceleration, deceleration), `stepper_facing_event`; per source `target_id`, `target_name`, `target_order`, `target_default_position`. See `example-esp-devices/halloween-stepper.yaml`.

### Buttons

- **File:** `common/button/restart.yaml`
- **Description:** Configuration for a restart button.

### Features

- **Files:** `common/features/maximumactive.yaml`, `common/features/sleepfunc.yaml`
- **Description:** Configurations for features like maximum active time and sleep functions.
- **Variables:**
  - `sleep_duration`: Duration for sleep mode (e.g., 60s)

### Sensors

- **File:** `common/sensor/uptime.yaml`
- **Description:** Configuration for an uptime sensor.

### Switches

- **File:** `common/switch/relay01s.yaml`
- **Description:** Configuration for a GPIO relay switch with optional timer (turn off after interval has passed)

### Text Sensors

- **File:** `common/text_sensor/version.yaml`
- **Description:** Configuration for a version text sensor.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.