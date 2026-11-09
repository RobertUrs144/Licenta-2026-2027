# Project Specification: Edge-Cloud Fall Detection & Push-to-Talk Companion

*File path: `docs/proiect.md`*

---

## 1. Context and Problem Statement

The baseline research project (**PiCam**) successfully implemented an end-to-end 4-tier IoT architecture for residential monitoring, utilizing a Raspberry Pi 4B, MJPEG video streaming tunneled through ngrok, cloud-side image classification using Google Cloud Vision API, and 1-Wire environmental sensing via a DHT22 sensor.

Adapting this baseline to the critical domain of ambient assisted living (AAL) and remote elderly care revealed significant architectural bottlenecks:
1. **Cloud Inference Latency and Operational Costs:** Ingesting video frames into Google Cloud Vision introduces round-trip latencies between 500 ms and 2000 ms. In acute emergency events such as sudden falls, cloud latency impedes immediate intervention. Furthermore, pay-per-call API fees and continuous transmission of private indoor imagery create recurring expenses and severe privacy concerns.
2. **Absence of Bidirectional Voice Channels:** The MJPEG streaming pipe is strictly unidirectional and bandwidth-heavy. In critical situations, the caregiver has no mechanism to converse with the resident to assess consciousness or offer reassurance.
3. **Hardware Fragility:** The DHT22 sensor suffers from high read latencies (3+ seconds) and driver instability on modern 64-bit kernels on the Raspberry Pi 5. Moreover, without an uninterruptible power supply, sudden power cuts can corrupt the ext4 filesystem.

---

## 2. Measurable Objectives

* **O1: Platform Migration & Edge Node Hardening:** Migrate the processing core to a Raspberry Pi 5 (8 GB) powered by an official 27 W PSU, active cooling, an industrial $I^2C$ sensor (BME280 or SHT4x), and an onboard UPS HAT with graceful shutdown handling.
* **O2: On-Device Vision Inference (Edge Fall Detection):** Deploy a quantized YOLOv8-nano neural network running locally on the Raspberry Pi 5 ARM Cortex-A76 CPU, sustaining $\ge 15$ FPS without cloud vision dependencies.
* **O3: Low-Latency Full-Duplex Push-to-Talk (PTT) Audio:** Establish an authenticated, sub-200 ms bidirectional audio bridge between the caregiver mobile application and the edge device.
* **O4: Automated Voice Check-in & Tangible Confirmation:** Implement a deterministic edge state machine that triggers local voice verification ("Are you okay?") and sounds an alarm when a fall is detected, allowing dismissal via a physical "I'm OK" button or escalation to the caregiver.
* **O5: Rigorous Experimental Benchmarking:** Quantitatively compare three audio transport protocols (WebRTC, WebSocket Chunked, and Firebase Storage File Transfer) across mouth-to-ear latency, jitter, packet loss, and intelligibility.

---

## 3. Hardware & Software Bill of Materials

### Hardware Specifications

| Component | Technical Specification | Operational Role |
| :--- | :--- | :--- |
| **SBC Compute Unit** | Raspberry Pi 5 (8 GB LPDDR4X, Broadcom BCM2712 Quad-Core Cortex-A76 @ 2.4 GHz) | Central processor for hardware loop, edge YOLO inference, and WebRTC streaming |
| **Power Unit** | Official RPi 27 W USB-C PSU + RPi Active Cooler | Stable 5.1V / 5A power delivery and active thermal throttling prevention |
| **Backup Power** | UPS HAT with integrated Li-ion cell | Prevents ext4 filesystem corruption during power outages; manages graceful shutdown |
| **Camera Sensor** | Sony IMX219 Module (8 MP, MIPI CSI-2 interface) | High-speed video acquisition handled via `libcamera` and `Picamera2` |
| **Pan-Tilt Mechanism**| Dual SG90 Micro Servos + PCA9685 16-channel 12-bit I²C PWM driver | Centers camera viewpoint dynamically on the subject or responds to manual PTZ commands |
| **Environmental Sensor**| BME280 or SHT4x | Precision ambient temperature, relative humidity, and barometric pressure tracking |
| **Audio Subsystem** | USB Speakerphone (or INMP441 I²S mic + MAX98357A I²S amp + 3 W speaker) | Bidirectional Push-to-Talk voice streaming and local automated voice check-in prompts |
| **Tactile & Acoustic I/O**| GPIO tactile pushbuttons, active buzzer, dual-color status LED | Local acknowledgment of safety status ("I'm OK") and acoustic distress signaling |

### Software Stack

* **Edge Operating System:** Raspberry Pi OS 64-bit (Debian 12 Bookworm), Python 3.12+
* **Vision & Inference:** OpenCV 4.10, Picamera2, ONNX Runtime / NCNN / Ultralytics
* **Audio & Streaming:** `aiortc` (WebRTC), PyAudio / ALSA, Opus Audio Codec
* **Cloud Infrastructure:** Firebase Realtime Database (telemetry, signaling), Firebase Cloud Storage (event clips), Firebase Authentication
* **Client App:** React Native (Expo SDK 56), `@react-navigation`, `react-native-webrtc`, Chart.js

---

## 4. Technical Feasibility & Scope Boundaries

### In Scope
* Edge-computed bounding box extraction and fall posture classification.
* Three-way audio transport comparison (WebRTC vs. WebSocket Chunked vs. Firebase Storage).
* Local voice prompt generation, tactile switch event handling, and buzzer alarms.
* Bidirectional WebRTC signaling mediated via Firebase RTDB.