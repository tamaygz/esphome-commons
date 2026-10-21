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