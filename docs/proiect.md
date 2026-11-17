# Project Specification: Edge-Cloud Fall Detection & Push-to-Talk Companion

*File path: `docs/proiect.md`*

---

## 1. Context and Problem Statement

The baseline research project (**PiCam**) successfully implemented an end-to-end 4-tier IoT architecture for residential monitoring, utilizing a Raspberry Pi 4B, MJPEG video streaming tunneled through ngrok, cloud-side image classification using Google Cloud Vision API, and 1-Wire environmental sensing via a DHT22 sensor.

Adapting this baseline to the critical domain of ambient assisted living (AAL) and remote elderly care revealed significant architectural bottlenecks:
1. **Cloud Inference Latency and Operational Costs:** Ingesting video frames into Google Cloud Vision introduces round-trip latencies between 500 ms and 2000 ms. In acute emergency events such as sudden falls, cloud latency impedes immediate intervention. Furthermore, pay-per-call API fees and continuous transmission of private indoor imagery create recurring expenses and severe privacy concerns.
2. **Absence of Bidirectional Voice Channels:** The MJPEG streaming pipe is strictly unidirectional and bandwidth-heavy. In critical situations, the caregiver has no mechanism to converse with the resident to assess consciousness or offer reassurance.
3. **Hardware Fragility:** The DHT22 sensor suffers from slow read cycles (3+ seconds, on top of a datasheet minimum sampling interval of about 2 seconds) and driver instability on modern 64-bit kernels on the Raspberry Pi 5. Moreover, without an uninterruptible power supply, sudden power cuts can corrupt the ext4 filesystem.

---

## 2. Measurable Objectives

* **O1: Platform Migration & Edge Node Hardening:** Migrate the processing core to a Raspberry Pi 5 (8 GB) powered by an official 27 W PSU, active cooling, an industrial $I^2C$ sensor (SHT4x), and an onboard UPS HAT with graceful shutdown handling. *Acceptance:* no thermal throttling or undervoltage flags during a 60-minute full-load run; graceful shutdown completes on battery; the filesystem remains consistent after a forced power-cut series with the UPS.
* **O2: On-Device Vision Inference (Edge Fall Detection):** Deploy a quantized YOLOv8-nano neural network running locally on the Raspberry Pi 5 ARM Cortex-A76 CPU, without cloud vision dependencies. *Targets (proposed, to be confirmed by an early benchmark):* end-to-end throughput (capture → detection → post-processing) of $\ge 15$ FPS at a documented input resolution, and fall-event sensitivity of $\ge 90\%$ with a documented false-alarm rate per day. *Fallback if the FPS target is not reached:* motion-gated inference, frame skipping or a reduced input size, reporting the achieved FPS and its effect on detection delay.
* **O3: Low-Latency Bidirectional Push-to-Talk (PTT) Audio:** Establish an authenticated, bidirectional audio bridge between the caregiver mobile application and the edge device, operated as push-to-talk (one talker at a time) over a transport that supports simultaneous two-way media, with echo suppression on the device. *Target:* mouth-to-ear latency below 200 ms (median, with p95 reported) in each direction under the Typical Wi-Fi network profile defined in the evaluation protocol.
* **O4: Automated Voice Check-in & Tangible Confirmation:** Implement a deterministic edge state machine that triggers local voice verification ("Are you okay?") and sounds an alarm when a fall is detected, allowing dismissal via a physical "I'm OK" button or escalation to the caregiver. *Proposed timing (to be confirmed):* voice prompt within 2 s of a confirmed fall; 15 s response window; caregiver escalation reaching the app within 5 s after the window expires.
* **O5: Rigorous Experimental Benchmarking:** Quantitatively compare three audio transport protocols (WebRTC, WebSocket Chunked, and Firebase Storage File Transfer) across mouth-to-ear latency, time-to-first-audio, jitter, packet loss, and intelligibility, under four emulated network profiles.

---

## 3. Hardware & Software Bill of Materials

### Hardware Specifications

| Component | Technical Specification | Operational Role |
| :--- | :--- | :--- |
| **SBC Compute Unit** | Raspberry Pi 5 (8 GB LPDDR4X, Broadcom BCM2712 Quad-Core Cortex-A76 @ 2.4 GHz) | Central processor for hardware loop, edge YOLO inference, and WebRTC streaming |
| **Power Unit** | Official RPi 27 W USB-C PSU + RPi Active Cooler | Stable 5.1V / 5A power delivery and active thermal throttling prevention |
| **Backup Power** | UPS HAT with integrated Li-ion cell, rated for Raspberry Pi 5 (5 V output; continuous current at or above the measured peak budget) | Prevents ext4 filesystem corruption during power outages; manages graceful shutdown. The Pi 5 limits USB peripheral current when it cannot negotiate a 5 A supply, so this behaviour must be verified with the chosen HAT |
| **Camera Sensor** | Sony IMX219 Module (8 MP, MIPI CSI-2 interface) + 15-to-22-pin adapter cable (Raspberry Pi 5 camera connector) | High-speed video acquisition handled via `libcamera` and `Picamera2` |
| **Pan-Tilt Mechanism (optional / stretch goal)** | Dual SG90 Micro Servos + PCA9685 16-channel 12-bit I²C PWM driver, powered from a separate 5 V rail (not from the Pi supply) | Centers camera viewpoint on the subject or responds to manual PTZ commands. Not required by O1–O5; motion-trigger frame differencing must be paused or compensated while the camera moves |
| **Environmental Sensor**| SHT4x (BME280 only if barometric pressure is later needed) | Ambient temperature and relative humidity tracking; mounted away from the CPU and heatsink to limit self-heating, with a per-unit calibration offset if needed |
| **Audio Subsystem** | USB Speakerphone with hardware echo cancellation (primary); alternative: INMP441 I²S mic + MAX98357A I²S amp + 3 W speaker, which requires software acoustic echo cancellation (e.g. the WebRTC AEC or `speexdsp`) | Bidirectional Push-to-Talk voice streaming and local automated voice check-in prompts |
| **Tactile & Acoustic I/O**| GPIO tactile pushbuttons, active buzzer, dual-color status LED | Local acknowledgment of safety status ("I'm OK") and acoustic distress signaling |

### Power Budget

The total peak current must stay within the continuous output rating of the UPS HAT. The table is filled from measurements (in-line USB-C power meter) before the final hardware configuration is frozen.

| Consumer | Idle (A @ 5 V) | Peak (A @ 5 V) | Supply Rail |
| :--- | :--- | :--- | :--- |
| Raspberry Pi 5 + Active Cooler (full inference load) | *to be measured* | *to be measured* | UPS HAT |
| USB speakerphone / I²S audio | *to be measured* | *to be measured* | UPS HAT |
| Camera, sensor, buttons, buzzer, LED | *to be measured* | *to be measured* | UPS HAT |
| Pan-tilt servos (if built) | *to be measured* | *to be measured* | Separate 5 V rail |

### Video Path (Design Decision)

Raw video never leaves the device continuously. Inference runs on the Raspberry Pi 5, and Firebase Storage receives only short event clips when a fall is detected. An optional live view is streamed over WebRTC only while the caregiver requests it. The Raspberry Pi 5 has no hardware H.264 encoder, so the live view uses software encoding at a reduced resolution and frame rate, and its CPU cost is included in the 60-minute stress run of the evaluation protocol.

### Software Stack

* **Edge Operating System:** Raspberry Pi OS 64-bit (Debian 12 Bookworm) with the distribution's system Python (3.11). `Picamera2` and `libcamera` come from `apt`, and the application runs in a virtual environment created with `--system-site-packages`. One OS release and package set is pinned for all experiments.
* **Vision & Inference:** OpenCV 4.10, Picamera2, ONNX Runtime / NCNN / Ultralytics
* **Audio & Streaming:** `aiortc` (WebRTC), PyAudio / ALSA, Opus Audio Codec, acoustic echo cancellation (hardware in the speakerphone or software)
* **Cloud Infrastructure:** Firebase Realtime Database (telemetry, signaling), Firebase Cloud Storage (event clips), Firebase Authentication; a STUN/TURN server (e.g. self-hosted coturn) for NAT traversal; a WebSocket relay service for the chunked transport (e.g. a containerized service on a cloud platform that supports WebSockets)
* **Client App:** React Native (Expo SDK 56) built as a development build (custom dev client, required by `react-native-webrtc` and not available in Expo Go), `@react-navigation`, `react-native-webrtc`, and a React Native-compatible charting library (e.g. Victory Native) or Chart.js inside a WebView

---

## 4. Technical Feasibility & Scope Boundaries

### In Scope
* Edge-computed bounding box extraction and fall posture classification.
* Three-way audio transport comparison (WebRTC vs. WebSocket Chunked vs. Firebase Storage).
* Local voice prompt generation, tactile switch event handling, and buzzer alarms.
* Bidirectional WebRTC signaling mediated via Firebase RTDB.
* Event-clip upload to Firebase Storage and optional on-demand live view (see Video Path).
* Power-loss protection (UPS HAT, graceful shutdown) and environmental sensing with the SHT4x.
* Optional / stretch: pan-tilt camera control, if time permits and it does not affect O1–O5.


### Risks & Mitigations

| Risk | Impact | Mitigation |
| :--- | :--- | :--- |
| 15 FPS not reachable on the Pi 5 CPU (published FP32 benchmarks report lower throughput) | O2 target missed | Early benchmark; motion-gated inference, frame skipping or reduced input size; report achieved FPS and detection delay |
| `aiortc` audio latency above 200 ms | O3 target missed | Early latency spike test; tune Opus frame size and jitter buffer; document the achieved latency |
| UPS HAT cannot supply peak current or limits USB current on the Pi 5 | Brownouts, undervoltage, throttling | Measure the power budget; verify HAT compatibility; power servos separately |
| Python / `Picamera2` environment conflicts | Camera pipeline fails | Use system Python with a `--system-site-packages` virtual environment; pin the OS release |
| NAT traversal failures on restrictive networks | WebRTC connection fails | Deploy a TURN server; evaluate with and without relay |
| False alarms or missed falls | Alert fatigue or missed emergency | Multi-stage validation (aspect-ratio heuristic) and voice check-in; report event-level metrics |