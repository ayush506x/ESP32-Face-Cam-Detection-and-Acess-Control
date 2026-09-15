# Advanced Face Detection System — ESP32-CAM

Real-time facial detection and access-control system running inference **directly on ESP32-CAM hardware**. Fully on-device — no cloud round-trip for detection — built for low-latency edge deployment.

![Platform](https://img.shields.io/badge/platform-ESP32--CAM-blue)
![Framework](https://img.shields.io/badge/framework-Arduino-00979D)
![Detection](https://img.shields.io/badge/detection-ESP--WHO%20%2F%20ESP--Face-orange)
![License](https://img.shields.io/badge/license-All%20Rights%20Reserved-red)

---

## Overview

This project turns an ESP32-CAM module into a standalone, edge-based access-control node. Faces are detected on-device using the **ESP-Face / ESP-WHO** detection pipeline, and a **relay** is triggered to actuate a door lock/strike when access is granted — all without sending video to an external server.

## Features

- 📷 Real-time face detection running entirely on the ESP32-CAM (no cloud inference)
- 🔓 Relay-driven access control (door lock / electric strike)
- ⚡ Low-latency edge pipeline — decision made on-device, in real time
- 🌐 Optional Wi-Fi streaming/monitoring interface
- 🔌 Simple GPIO relay integration, easy to wire into existing lock hardware

## Hardware Requirements

| Component | Notes |
|---|---|
| ESP32-CAM (AI-Thinker or compatible) | Main compute + camera module |
| FTDI / USB-TTL programmer | For flashing (ESP32-CAM has no onboard USB) |
| Relay module (5V, 1-channel) | Drives the door lock / electric strike |
| Door lock / electric strike | The actuated access-control hardware |
| 5V power supply | Stable supply recommended — brownouts affect camera init |
| Jumper wires / breadboard | Wiring relay + programmer to the board |

### Wiring Summary

| ESP32-CAM Pin | Connects To |
|---|---|
| 5V | Relay VCC / Power supply |
| GND | Relay GND / Power supply GND |
| GPIO (configurable, see `config.h`) | Relay IN |
| U0R / U0T | FTDI TX/RX (flashing only) |

> Update the exact GPIO pin used for the relay trigger in your config file to match your wiring.

## Software Requirements

- [Arduino IDE](https://www.arduino.cc/en/software) (1.8.x or 2.x) with ESP32 board support installed
- **ESP-Face / ESP-WHO** face detection library
- ESP32 Arduino Core (board package)

## Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/<your-username>/esp32cam-face-access-control.git
   cd esp32cam-face-access-control
   ```

2. **Install board support**
   In Arduino IDE: `File > Preferences > Additional Boards Manager URLs`, add the ESP32 package URL, then install "esp32" via Boards Manager.

3. **Install the detection library**
   Add the ESP-Face / ESP-WHO library to your Arduino `libraries` folder (or install via Library Manager if available for your core version).

4. **Configure Wi-Fi and pins**
   Open the config header/sketch and set:
   - Wi-Fi SSID/password (if using the streaming interface)
   - Relay GPIO pin
   - Camera model (AI-Thinker pin mapping is default for most ESP32-CAM boards)

5. **Select board and upload**
   - Board: `AI Thinker ESP32-CAM`
   - Partition scheme: `Huge APP (3MB No OTA/1MB SPIFFS)`
   - Wire GPIO0 to GND to enter flash mode, upload, then disconnect GPIO0 and reset to run.

## Usage

1. Power on the board — it initializes the camera and detection pipeline.
2. When a face is detected in frame, the system evaluates it and triggers the relay to unlock/actuate the connected hardware.
3. (Optional) Connect to the device's Wi-Fi stream/monitoring endpoint to view live detection status.

## Project Structure

```
esp32cam-face-access-control/
├── src/                # Main sketch(es) and detection logic
├── include/ or config/ # Pin mappings, Wi-Fi credentials, relay config
├── docs/                # Wiring diagrams, setup notes
├── README.md
└── .gitignore
```

> Adjust this tree to match your actual folder layout before pushing.

## How It Works

1. The onboard camera captures frames continuously.
2. Frames are passed through the ESP-Face/ESP-WHO detection model running on the ESP32's own processor — no data leaves the device.
3. On a positive face detection (optionally matched against an authorized set), the system pulses the relay GPIO to actuate the connected lock.
4. The system resets and continues monitoring.

## Roadmap / Possible Extensions

- [ ] Face **recognition** (identity matching), not just detection
- [ ] Local logging of access events (SD card / SPIFFS)
- [ ] Web dashboard for access history
- [ ] Multi-relay support for multiple doors

## License

All rights reserved. No license is currently granted for reuse, modification, or distribution of this code. Contact the repository owner for permissions.

## Acknowledgements

- [Espressif ESP-WHO / ESP-Face](https://github.com/espressif/esp-who) — on-device face detection framework
- ESP32 Arduino Core community
