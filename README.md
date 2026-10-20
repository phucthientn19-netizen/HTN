# HTN — IoT Smart Power Monitoring and Control System

An ESP32-C3-based IoT system for monitoring electrical parameters and remotely controlling electrical loads using ESP-IDF and ESP RainMaker.

## Overview

This project implements an embedded smart power monitoring and control system using the ESP32-C3 microcontroller. It combines AC voltage and current measurement, environmental monitoring, relay-based load control, local LCD displays, and cloud-connected device management.

The firmware collects sensor data, calculates electrical measurements, displays readings locally, and reports monitoring data to ESP RainMaker for remote access.

## Key Features

- **AC Voltage Monitoring:** Measures AC voltage using a ZMPT101B voltage sensor.
- **Dual-Channel Current Monitoring:** Measures current on two channels using ACS712 current sensors.
- **Power Estimation:** Calculates estimated power for each channel from measured voltage and current.
- **Temperature and Humidity Monitoring:** Reads environmental data using a DHT11 sensor.
- **Dual Relay Control:** Controls two electrical loads through GPIO-connected relay modules.
- **Remote IoT Control:** Provides relay control and sensor monitoring through ESP RainMaker.
- **Dual LCD Display:** Displays relay status, temperature, humidity, voltage, current, and estimated power locally.
- **Periodic Cloud Reporting:** Publishes sensor measurements to ESP RainMaker approximately every three seconds.
- **OTA Firmware Updates:** Enables over-the-air firmware updates through ESP RainMaker.
- **Device Diagnostics:** Integrates ESP Insights for device monitoring and diagnostics.
- **Concurrent Firmware Tasks:** Uses FreeRTOS tasks, a queue, and a mutex to coordinate sensor acquisition and LCD updates.

## Hardware Components

| Component | Purpose |
|---|---|
| ESP32-C3 | Main microcontroller and Wi-Fi connectivity |
| ACS712 × 2 | AC current sensing for two channels |
| ZMPT101B | AC voltage sensing |
| DHT11 | Temperature and humidity measurement |
| Relay modules × 2 | Switching two electrical loads |
| I2C LCD × 2 | Local measurement and status display |

## Pin Configuration

The following GPIO assignments are defined in `main/app_main.c`.

| Signal | ESP32-C3 GPIO |
|---|---:|
| Relay 1 | GPIO 4 |
| Relay 2 | GPIO 5 |
| DHT11 data | GPIO 0 |
| I2C SDA | GPIO 6 |
| I2C SCL | GPIO 7 |
| ACS712 channel 1 | GPIO 1 / ADC1 Channel 1 |
| ACS712 channel 2 | GPIO 2 / ADC1 Channel 2 |
| ZMPT101B | GPIO 3 / ADC1 Channel 3 |

The two LCDs use I2C addresses `0x27` and `0x26`.

Verify the sensor modules, GPIO assignments, voltage levels, and board pinout before wiring the hardware.

## Software Architecture

### 1. Sensor Acquisition

The firmware reads temperature and humidity from the DHT11 sensor and samples the two current channels and voltage channel through the ADC.

For AC measurements, the firmware samples signals over an approximately 40 ms window and calculates an RMS-based value. Sensor offsets and calibration constants are used to convert ADC readings into estimated electrical measurements.

### 2. Power Estimation

The firmware estimates power for each channel using:

\[
P_1 = V_{\mathrm{AC}} \times I_1
\]

\[
P_2 = V_{\mathrm{AC}} \times I_2
\]

These calculations provide apparent-power estimates when voltage and current are RMS values. Accurate real-power measurement for AC loads with non-unity power factor requires additional phase-aware measurement and calibration.

### 3. Relay Control

Two relay outputs are controlled through GPIO 4 and GPIO 5. ESP RainMaker write callbacks process remote relay commands and update the corresponding GPIO output states.

### 4. Local Display

Two I2C LCDs display the system status and measurement values:

- **LCD 1:** Relay 1 status, temperature, voltage, channel 1 current, and estimated power.
- **LCD 2:** Relay 2 status, humidity, voltage, channel 2 current, and estimated power.

An I2C mutex protects shared display access, while a FreeRTOS queue transfers sensor data to the display task.

### 5. Cloud Monitoring and Device Management

ESP RainMaker provides remote relay control and sensor-data reporting. The firmware also enables OTA updates and ESP Insights integration for device maintenance and diagnostics.

## Firmware Structure

```text
HTN/
├── CMakeLists.txt
├── partitions.csv
├── partitions_4mb_optimised.csv
├── main/
│   ├── app_main.c
│   ├── CMakeLists.txt
│   ├── Kconfig.projbuild
│   └── idf_component.yml
├── components/
│   ├── esp32-dht11/
│   ├── i2c_lcd/
│   └── esp_rainmaker/
└── managed_components/
```

- `main/app_main.c`: Main application, sensor acquisition, relay control, LCD display, and ESP RainMaker integration.
- `components/esp32-dht11/`: DHT11 sensor driver.
- `components/i2c_lcd/`: I2C LCD driver.
- `components/esp_rainmaker/`: ESP RainMaker component source.
- `main/idf_component.yml`: Application component dependencies.
- `CMakeLists.txt`: ESP-IDF project build configuration.

## Development Environment

- **Microcontroller:** ESP32-C3
- **Framework:** ESP-IDF
- **Language:** C
- **RTOS:** FreeRTOS
- **Connectivity:** Wi-Fi
- **IoT Platform:** ESP RainMaker
- **Device Diagnostics:** ESP Insights
- **Build System:** CMake
- **Interfaces:** GPIO, ADC, I2C

The project manifest requires ESP-IDF version 5.0 or later. Use a compatible ESP-IDF environment and resolve the declared component dependencies before building.

## Build and Flash

1. Install ESP-IDF by following the official [ESP-IDF documentation](https://docs.espressif.com/projects/esp-idf/en/stable/esp32c3/get-started/).
2. Open an ESP-IDF terminal and navigate to the project root.
3. Set the target to ESP32-C3.
4. Configure the project and verify the required component dependencies.
5. Build and flash the firmware using the ESP-IDF tools.
6. Monitor the serial output and complete the ESP RainMaker network provisioning process.

Follow the official [ESP RainMaker documentation](https://rainmaker.espressif.com/docs/get-started.html) to configure cloud connectivity and device provisioning.

## Safety Considerations

This project involves AC voltage and current sensing and may control mains-powered electrical loads.

- Use properly rated, isolated sensor modules and relay hardware.
- Verify isolation, grounding, wiring, fuse protection, and enclosure safety.
- Never connect mains voltage directly to ESP32 GPIO pins or ADC inputs.
- Perform initial firmware tests with safe, low-voltage setups whenever possible.
- Sensor calibration and measurement accuracy must be validated against suitable reference instruments before practical deployment.

## Future Improvements

- Improve voltage and current calibration.
- Validate measurement accuracy under different load conditions.
- Implement more accurate real-power and energy calculations.
- Add historical measurement logging and energy-consumption analysis.
- Improve fault handling and communication recovery.

## Author

**phucthientn19-netizen**

GitHub: [https://github.com/phucthientn19-netizen](https://github.com/phucthientn19-netizen)

---

*An embedded systems and IoT project focused on electrical monitoring, remote load control, and connected-device management using ESP32-C3.*
