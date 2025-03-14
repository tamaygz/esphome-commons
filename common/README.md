# ESPHome Commons

This repository contains common configurations and packages for ESPHome devices. These configurations can be included in your ESPHome projects to simplify setup and ensure consistency across multiple devices.

## Directory Structure

```
common/
  api.yaml
  common_defaults.yaml
  device_base_esp8266.yaml
  device_base.yaml
  logger.yaml
  time.yaml
  web_server.yaml
  wifi.yaml
  binary_sensor/
    gpio_sensor.yaml
    status.yaml
  button/
    restart.yaml
  features/
    maximumactive.yaml
    sleepfunc.bak
    sleepfunc.yaml
  sensor/
    uptime.yaml
  switch/
    relay01s.yaml
  text_sensor/
    version.yaml
api.yaml
common_defaults.yaml
device_base_esp8266.yaml
device_base.yaml
esphome-commons.code-workspace
logger.yaml
time.yaml
web_server.yaml
wifi.yaml
```

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

packages:
  base: !include common/device_base_esp8266.yaml
  wifi: !include common/wifi.yaml
  api: !include common/api.yaml
  logger: !include common/logger.yaml
  web_server: !include common/web_server.yaml
  time: !include common/time.yaml
  sleepfunc: !include common/features/sleepfunc.yaml
```

## Available Configurations

### Common Defaults

- **File:** `common/common_defaults.yaml`
- **Description:** Contains default substitutions for device name, friendly name, version, log level, and log baud rate.

### Device Base

- **File:** `common/device_base.yaml`
- **Description:** Base configuration for ESPHome devices, including common packages like WiFi, API, logger, status sensor, restart button, version sensor, and time.

### Device Base for ESP8266

- **File:** `common/device_base_esp8266.yaml`
- **Description:** Base configuration for ESP8266 devices, including platform-specific settings and common packages.

### WiFi Configuration

- **File:** `common/wifi.yaml`
- **Description:** WiFi configuration with support for multiple networks, fallback hotspot, OTA updates, and WiFi signal sensors.

### API Configuration

- **File:** `common/api.yaml`
- **Description:** Home Assistant API configuration with encryption.

### Logger Configuration

- **File:** `common/logger.yaml`
- **Description:** Logger configuration with customizable log level and baud rate.

### Web Server Configuration

- **File:** `common/web_server.yaml`
- **Description:** Web server configuration with authentication.

### Time Configuration

- **File:** `common/time.yaml`
- **Description:** Time configuration using Home Assistant as the time source.

### Binary Sensors

- **Files:** `common/binary_sensor/gpio_sensor.yaml`, `common/binary_sensor/status.yaml`
- **Description:** Configurations for GPIO and status binary sensors.

### Buttons

- **File:** `common/button/restart.yaml`
- **Description:** Configuration for a restart button.

### Features

- **Files:** `common/features/maximumactive.yaml`, `common/features/sleepfunc.yaml`
- **Description:** Configurations for features like maximum active time and sleep functions.

### Sensors

- **File:** `common/sensor/uptime.yaml`
- **Description:** Configuration for an uptime sensor.

### Switches

- **File:** `common/switch/relay01s.yaml`
- **Description:** Configuration for a GPIO relay switch with optional timer.

### Text Sensors

- **File:** `common/text_sensor/version.yaml`
- **Description:** Configuration for a version text sensor.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.