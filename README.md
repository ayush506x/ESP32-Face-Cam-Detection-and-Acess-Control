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
# Code & Setup Guide — ESP32-CAM Face Detection Access Control

This document walks through the full implementation, step by step, so it can be committed to the repo (e.g. as `docs/CODE_GUIDE.md`) alongside the source files.

---

## 1. Project File Layout

```
esp32cam-face-access-control/
├── src/
│   └── main.ino              # Main sketch — camera init, detection loop, relay trigger
├── include/
│   ├── config.h               # Pin mapping, relay pin, thresholds
│   └── secrets.h               # Wi-Fi credentials (gitignored)
├── docs/
│   └── CODE_GUIDE.md          # This file
├── README.md
└── .gitignore
```

---

## 2. `include/secrets.h` — Wi-Fi Credentials (keep out of git)

```cpp
#ifndef SECRETS_H
#define SECRETS_H

#define WIFI_SSID     "your-network-name"
#define WIFI_PASSWORD "your-network-password"

#endif
```

This file is already listed in `.gitignore` — never commit real credentials.

---

## 3. `include/config.h` — Pins & Detection Settings

```cpp
#ifndef CONFIG_H
#define CONFIG_H

// ---------- Camera model: AI-Thinker ESP32-CAM pin map ----------
#define PWDN_GPIO_NUM     32
#define RESET_GPIO_NUM    -1
#define XCLK_GPIO_NUM      0
#define SIOD_GPIO_NUM     26
#define SIOC_GPIO_NUM     27
#define Y9_GPIO_NUM       35
#define Y8_GPIO_NUM       34
#define Y7_GPIO_NUM       39
#define Y6_GPIO_NUM       36
#define Y5_GPIO_NUM       21
#define Y4_GPIO_NUM       19
#define Y3_GPIO_NUM       18
#define Y2_GPIO_NUM        5
#define VSYNC_GPIO_NUM    25
#define HREF_GPIO_NUM     23
#define PCLK_GPIO_NUM     22

// ---------- Relay (access control actuator) ----------
#define RELAY_PIN         13      // Update to match your wiring
#define RELAY_ACTIVE_HIGH true    // Set false if your relay module is active-low
#define UNLOCK_HOLD_MS    3000    // How long the relay stays energized

// ---------- Detection tuning ----------
#define DETECTION_CONFIDENCE_MIN 0.6   // Minimum score to count as a valid detection
#define DETECTION_COOLDOWN_MS    5000  // Prevents relay re-trigger spam

#endif
```

---

## 4. `src/main.ino` — Main Sketch

```cpp
#include <WiFi.h>
#include "esp_camera.h"
#include "fd_forward.h"      // ESP-Face detection API (from ESP-WHO / ESP-Face)
#include "config.h"
#include "secrets.h"

// Camera config object populated from config.h pin map
camera_config_t camera_config;

unsigned long lastTriggerTime = 0;

void setupRelay() {
  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, RELAY_ACTIVE_HIGH ? LOW : HIGH); // start de-energized
}

void triggerRelay() {
  digitalWrite(RELAY_PIN, RELAY_ACTIVE_HIGH ? HIGH : LOW);
  delay(UNLOCK_HOLD_MS);
  digitalWrite(RELAY_PIN, RELAY_ACTIVE_HIGH ? LOW : HIGH);
}

void setupCamera() {
  camera_config.pin_pwdn  = PWDN_GPIO_NUM;
  camera_config.pin_reset = RESET_GPIO_NUM;
  camera_config.pin_xclk  = XCLK_GPIO_NUM;
  camera_config.pin_sscb_sda = SIOD_GPIO_NUM;
  camera_config.pin_sscb_scl = SIOC_GPIO_NUM;
  camera_config.pin_d7 = Y9_GPIO_NUM;
  camera_config.pin_d6 = Y8_GPIO_NUM;
  camera_config.pin_d5 = Y7_GPIO_NUM;
  camera_config.pin_d4 = Y6_GPIO_NUM;
  camera_config.pin_d3 = Y5_GPIO_NUM;
  camera_config.pin_d2 = Y4_GPIO_NUM;
  camera_config.pin_d1 = Y3_GPIO_NUM;
  camera_config.pin_d0 = Y2_GPIO_NUM;
  camera_config.pin_vsync = VSYNC_GPIO_NUM;
  camera_config.pin_href  = HREF_GPIO_NUM;
  camera_config.pin_pclk  = PCLK_GPIO_NUM;
  camera_config.xclk_freq_hz = 20000000;
  camera_config.pixel_format = PIXFORMAT_GRAYSCALE; // grayscale is faster for detection
  camera_config.frame_size = FRAMESIZE_QVGA;         // 320x240 — good balance for on-device inference
  camera_config.fb_count = 2;

  esp_err_t err = esp_camera_init(&camera_config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x\n", err);
    ESP.restart();
  }
}

void connectWiFi() {
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to Wi-Fi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConnected. IP: " + WiFi.localIP().toString());
}

void setup() {
  Serial.begin(115200);
  setupRelay();
  setupCamera();
  connectWiFi();
  Serial.println("System ready — monitoring for faces.");
}

void loop() {
  camera_fb_t *fb = esp_camera_fb_get();
  if (!fb) {
    Serial.println("Camera capture failed");
    delay(200);
    return;
  }

  // Run detection on the captured frame using ESP-Face
  box_array_t *boxes = face_detect(fb, /* detection config from ESP-Face */ nullptr);

  bool faceFound = (boxes != nullptr && boxes->len > 0);

  if (faceFound) {
    unsigned long now = millis();
    if (now - lastTriggerTime > DETECTION_COOLDOWN_MS) {
      Serial.println("Face detected — triggering relay.");
      triggerRelay();
      lastTriggerTime = now;
    }
  }

  if (boxes) {
    free(boxes->box);
    free(boxes->score);
    free(boxes);
  }

  esp_camera_fb_return(fb);
  delay(100); // throttle loop rate
}
```

> `face_detect(...)` and the exact ESP-Face/ESP-WHO API signature vary slightly by library version — match this call to the version you have installed (check the example sketches shipped with the library for the exact function signature and detection-config struct).

---

## 5. Step-by-Step: From Zero to Flashed Device

1. **Wire the hardware**
   - ESP32-CAM → FTDI programmer (U0T→RX, U0R→TX, 5V, GND)
   - GPIO0 → GND (flash mode only, remove after upload)
   - Relay IN → `RELAY_PIN` (default GPIO13), relay VCC/GND → 5V/GND
   - Relay COM/NO → wired into your door lock/strike circuit

2. **Install Arduino IDE + ESP32 core**
   - Add ESP32 board URL in Preferences → Boards Manager → install `esp32`

3. **Install ESP-Face / ESP-WHO**
   - Clone/install into your Arduino `libraries/` folder per the library's own install instructions

4. **Fill in credentials**
   - Copy your Wi-Fi SSID/password into `include/secrets.h`

5. **Set your board + upload settings**
   - Board: `AI Thinker ESP32-CAM`
   - Partition Scheme: `Huge APP (3MB No OTA/1MB SPIFFS)`
   - Upload Speed: `115200`

6. **Flash**
   - Ground GPIO0, press reset, hit Upload
   - After success, disconnect GPIO0 from GND and reset again to boot normally

7. **Verify over Serial Monitor (115200 baud)**
   - Confirm Wi-Fi connects and "System ready" prints
   - Present a face to the camera and confirm the relay trigger log line + audible relay click

8. **Push to GitHub**
   ```bash
   git init
   git add .
   git commit -m "Initial commit: ESP32-CAM face detection access control"
   git branch -M main
   git remote add origin https://github.com/<your-username>/esp32cam-face-access-control.git
   git push -u origin main
   ```

---

## 6. Troubleshooting Notes (worth keeping in the repo)

| Symptom | Likely Cause |
|---|---|
| Camera init fails (`0x105`) | Insufficient power — use a dedicated 5V/2A supply, not the FTDI's 5V pin |
| Upload fails / times out | GPIO0 not grounded during flashing, or wrong board selected |
| Brownout reset loop | Power supply sag — add a capacitor across 5V/GND near the board |
| Detection never fires | Check `pixel_format`/`frame_size` match what the detection library expects; verify lighting |
| Relay chatters or fails silently | Confirm `RELAY_ACTIVE_HIGH` matches your specific relay module's logic level |

---

## 7. Notes for the Reader

The code above is a representative implementation matching the wiring and logic described in the README (Arduino + ESP-Face/ESP-WHO, relay-driven door lock). Replace the sketch, config values, and detection API calls with your actual working code before committing — this is meant as the documented skeleton to slot your real implementation into.
