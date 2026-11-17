# Literature Review & State of the Art

*File path: `docs/literatura.md`*

---

## 1. Reading Sheets (10 Scientific References)

### Paper 01: OpenCV Fundamentals
* **Reference:** G. Bradski and A. Kaehler, *Learning OpenCV: Computer Vision with the OpenCV Library*, O'Reilly Media.
* **Core Method:** Background subtraction (`absdiff`), morphological operations (dilation/erosion), and bounding contour extraction.
* **Metrics / Performance:** Sub-10ms CPU processing overhead on downscaled streams (640x480).
* **Identified Limitation:** Highly sensitive to illumination shifts and shadows; cannot classify semantic object categories.
* **Project Relevance:** Reused as a low-power motion-trigger stage before executing full deep neural network inference.

### Paper 02: Vision & Voice Fall Monitoring
* **Reference:** MDPI Computers (2025), *Real-Time Fall Monitoring for Seniors via YOLO and Voice Interaction*.
* **Core Method:** YOLO visual fall classification combined with an interactive voice confirmation state machine.
* **Metrics / Performance:** >90% fall detection accuracy; 80% reduction in false emergency dispatches via voice query.
* **Identified Limitation:** Heavy reliance on external cloud TTS/STT services introduces 2–3s latency spikes.
* **Project Relevance:** Establishes the foundation for our on-device automated voice check-in logic and alarm states.

### Paper 03: Edge Vision Optimization
* **Reference:** L. Rey, A. M. Bernardos, A. D. Dobrzycki, D. Carramiñana, L. Bergesio, J. A. Besada, J. R. Casar, *A Performance Analysis of You Only Look Once Models for Deployment on Constrained Computational Edge Devices in Drone Applications*, MDPI Electronics, vol. 14, no. 3, art. 638 (2025). DOI: 10.3390/electronics14030638.
* **Core Method:** Benchmark of YOLO models on constrained edge devices, including the Raspberry Pi 5, using the NCNN framework; reports inference time and energy per inference.
* **Metrics / Performance:** On the Raspberry Pi 5 (FP32), YOLOv8n averages 118 ms per iteration (≈ 8.5 FPS, derived) and YOLOv8s reaches 7.32 FPS. The authors report that neither FP16 nor INT8 was feasible on the Raspberry Pi 5 with NCNN.
* **Identified Limitation:** Drone-oriented evaluation; no INT8/FP16 results on the Raspberry Pi 5; no audio or environmental telemetry.
* **Project Relevance:** Provides a realistic FP32 baseline for objective O2 (≥ 15 FPS), which therefore must be validated experimentally. It motivates the motion-trigger gating from Paper 01, reduced input resolution and frame skipping.

### Paper 04: Kinematic Fall Detection
* **Reference:** K. de Miguel, A. Brunete, M. Hernando, E. Gambao, *Home Camera-Based Fall Detection System for the Elderly*, MDPI Sensors, vol. 17, no. 12, art. 2864 (2017). DOI: 10.3390/s17122864.
* **Core Method:** Background subtraction (MoG2), Kalman filtering and Lucas-Kanade optical flow feed a k-Nearest Neighbours classifier (k=3) using ellipse angle, the width/height ratio ($Width / Height$) and the ratio change speed.
* **Metrics / Performance:** 96% sensitivity and 97.6% specificity (Table 2; Table 3 lists 96.9%); roughly 7–8 analysed images per second on a Raspberry Pi 2.
* **Identified Limitation:** Classical 2D pipeline with low throughput on its target hardware; no voice-based confirmation of user consciousness post-event.
* **Project Relevance:** Supports the use of bounding-box aspect-ratio and ratio-change-speed features as a geometric validation stage after YOLO detection in our multi-stage fall pipeline.

### Paper 05: Low-Latency Audio Streaming
* **Reference:** Indico / GStreamer, *Cutting Audio Latency with Bidirectional WebRTC*.
* **Core Method:** Jitter buffer tuning, Opus speech codec configuration, and DTLS-SRTP encryption in WebRTC.
* **Metrics / Performance:** Sub-150ms mouth-to-ear latency achieved across wide-area networks.
* **Identified Limitation:** Requires STUN/TURN traversal configurations on restrictive symmetric NATs.
* **Project Relevance:** Defines the technical benchmark for the two-way Push-to-Talk caregiver channel.

### Paper 06: Hardware WebRTC on Raspberry Pi
* **Reference:** Mazupo et al., *pi-webrtc: Hardware WebRTC Streaming & Remote Control for Raspberry Pi*.
* **Core Method:** Native H.264 video encoding integrated into a real-time WebRTC media pipeline.
* **Metrics / Performance:** 150–200ms glass-to-glass latency across local and unconstrained networks.
* **Identified Limitation:** Designed strictly for unidirectional video and basic UART/serial controls; no two-way audio.
* **Project Relevance:** Serves as the structural reference for integrating Picamera2 with WebRTC peer connections.

### Paper 07: Ambient Companion Robotics
* **Reference:** USTP Research, *Smart Companion 2*.
* **Core Method:** Stationary/mobile domestic assistant featuring pan-tilt computer vision and ambient environmental sensing.
* **Metrics / Performance:** Validated user acceptance in assisted living; high satisfaction with vocal reminders.
* **Identified Limitation:** High mechanical complexity and commercial hardware costs.
* **Project Relevance:** Demonstrates the value of combining ambient telemetry with video monitoring to safeguard privacy.

### Paper 08: Microclimate Sensor Reliability
* **Reference:** M. Sulzer, A. Christen, A. Matzarakis, *A Low-Cost Sensor Network for Real-Time Thermal Stress Monitoring and Communication in Occupational Contexts*, MDPI Sensors, vol. 22, no. 5, art. 1828 (2022). DOI: 10.3390/s22051828.
* **Core Method:** Calibration and evaluation of low-cost sensors (AM2320, DHT22, MCP9808, BME680, BME280, HTU21D) against reference instruments: a Fluke 1502 thermometer in a climate chamber (temperature) and a Campbell Scientific CS215A probe in a Stevenson screen (humidity).
* **Metrics / Performance:** After calibration, temperature RMSE/MBE/MAE are below 0.1 K for all sensors, while the raw BME280 error is about 1 K. DHT22 gave the smallest humidity (vapor pressure) errors; indoor units stayed reliable for six months, whereas outdoor units drifted toward 100% RH after a few months.
* **Identified Limitation:** Does not compare communication interfaces (I²C vs. single-wire) and does not include the SHT4x.
* **Project Relevance:** Supports per-unit calibration of the BME280/SHT4x (raw offset ≈ 1 K) and indoor placement. Because this study does not show a DHT22 accuracy deficit, the DHT22 → BME280/SHT4x migration is justified by read latency, Raspberry Pi 5 driver stability and the I²C bus (manufacturer datasheets: Bosch BME280, Sensirion SHT4x), not by accuracy.

### Paper 09: Power Management on Linux Edge Devices
* **Reference:** M. Zheng, J. Tucek, F. Qin, M. Lillibridge, *Understanding the Robustness of SSDs under Power Fault*, 11th USENIX Conference on File and Storage Technologies (FAST '13), 2013. Extended in: *Reliability Analysis of SSDs under Power Fault*, ACM Trans. Comput. Syst., vol. 34, no. 4, art. 10 (2016). DOI: 10.1145/2992782.
* **Core Method:** A power-fault injection framework that cuts power during writes; 15 commodity SSDs from five vendors were tested over more than 3,000 fault-injection cycles.
* **Metrics / Performance:** 13 of the 15 devices showed failures, including bit corruption, shorn writes, unserializable writes, metadata corruption and total device failure (the 2016 extension reports 14 of 17 devices).
* **Identified Limitation:** Evaluates SSDs rather than SD cards, and does not evaluate a UPS or graceful-shutdown mechanism.
* **Project Relevance:** Provides evidence that abrupt power loss corrupts flash storage, which justifies the UPS HAT and the safe-shutdown daemon. Filesystem integrity with and without the UPS will be validated by our own repeated power-cut experiment.

### Paper 10: Comparative Push-to-Talk Evaluation
* **Reference:** M. Sacchetto, P. Gastaldi, C. Chafe, C. Rottondi, A. Servetti, *Web-Based Networked Music Performances via WebRTC: A Low-Latency PCM Audio Solution*, Journal of the Audio Engineering Society (2022). DOI: 10.17743/jaes.2022.0021. Complemented by: *Comparative Performance Evaluation of MQTT, WebSocket, and Firebase on ESP32-C3 for Real-Time IoT Communication*, JATI, vol. 10, no. 3 (June 2026).
* **Core Method:** Sacchetto et al. send 16-bit PCM stereo audio over the WebRTC RTCDataChannel (Web Audio and AudioWorklets) and compare it with the standard RTCPeerConnection path across configurations, browsers and operating systems. The JATI study compares MQTT, WebSocket and Firebase on latency, success rate and bandwidth.
* **Metrics / Performance:** Sacchetto et al. report that their approach outperforms the standard RTCPeerConnection (exact latency figures not verified). The JATI study finds that WebSocket gives the lowest latency and highest reliability, and that Firebase shows significantly higher latency.
* **Identified Limitation:** Neither work targets push-to-talk voice: the first addresses networked music performance, the second uses generic messages on an ESP32. No single study compares WebRTC, WebSocket chunked streaming and Firebase Storage file transfer for voice.
* **Project Relevance:** Reference for audio-latency measurement and transport trade-offs; the three-way comparison in objective O5 addresses the gap left by these works.

---

## 2. Comparative Synthesis Table

| # | Reference | Domain / Focus | Key Method | Reported Results | Missing Feature / Limitation | Adaptation in Project |
| :-: | :--- | :--- | :--- | :--- | :--- | :--- |
| **01** | Bradski et al. | Edge (CPU) | Frame Differencing | Sub-10ms execution | No object classification | Pre-filter for YOLO activation |
| **02** | MDPI (2025) | Hybrid Edge/Cloud | YOLO + Voice Dialog | >90% fall detection | 2–3s cloud audio lag | Local on-device voice prompts |
| **03** | Rey et al. (2025) | Edge (ARM) | YOLO Benchmark (NCNN, FP32) | YOLOv8n ≈ 8.5 FPS on RPi 5 | No INT8/FP16 on RPi 5; drone context | Realistic baseline for O2; frame gating |
| **04** | de Miguel et al. (2017) | Edge (CPU) | Ratio Features + kNN | 96% sensitivity, 97.6% specificity | 2D only; no voice confirmation | Geometric heuristic validation |
| **05** | Indico / GStreamer | Network / Edge | WebRTC Audio (Opus) | < 150 ms latency | Symmetric NAT overhead | Opus jitter buffer optimization |
| **06** | Mazupo (pi-webrtc) | Edge (Linux) | Hardware WebRTC | 150–200 ms latency | Unidirectional video only | Dual audio/video WebRTC pipeline |
| **07** | USTP Research | Physical Node | Ambient Companion | Positive clinical UX | High Bill-of-Materials cost | Cost-effective stationary unit |
| **08** | Sulzer et al. (2022) | Sensor Accuracy | Calibration vs. Reference | < 0.1 K after calibration; BME280 raw ≈ 1 K | No interface comparison; no SHT4x | Per-unit calibration of BME280/SHT4x |
| **09** | Zheng et al. (FAST 2013) | Storage / OS | Power-Fault Injection | 13 of 15 SSDs showed failures | SSDs, not SD cards; no UPS tested | Motivates UPS HAT; own power-cut test |
| **10** | Sacchetto et al. (JAES 2022) | Transport Layer | WebRTC Data-Channel PCM Audio | Outperforms standard RTCPeerConnection | Music use case; no WebSocket/Firebase | Latency measurement reference |

---

## 3. Positioning and Novelty

Existing commercial solutions predominantly consist of closed IP cameras that continuously transmit video feeds to third-party cloud servers, posing severe privacy risks for indoor residential monitoring. Academic alternatives often focus either on complex robotic mobility or pure kinematic analysis without real-time interaction.

This bachelor's thesis positions itself as an **edge-autonomous, privacy-first companion**:
1. **Edge-Native Inference:** Human and fall detection are computed entirely on the local Raspberry Pi 5 processor, retaining raw imagery within the local trust domain.
2. **Hybrid Incident Resolution:** False alarms are dismissed on-device via automated voice verification ("Are you okay?") and tactile pushbuttons ("I'm OK"), preventing unnecessary emergency dispatches.
3. **Rigorous Transport Comparison:** It provides an objective quantitative comparison between WebRTC, WebSocket chunked streaming, and file-based audio transfers for IoT voice applications.