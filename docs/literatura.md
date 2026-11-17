# Literature Review & State of the Art

*File path: `docs/literatura.md`*

---

## 1. Reading Sheets (14 References)

### Paper 01: OpenCV Fundamentals (textbook, supplementary)
* **Reference:** G. Bradski and A. Kaehler, *Learning OpenCV: Computer Vision with the OpenCV Library*, O'Reilly Media.
* **Core Method:** Background subtraction (`absdiff`), morphological operations (dilation/erosion), and bounding contour extraction.
* **Metrics / Performance:** Sub-10ms CPU processing overhead on downscaled streams (640x480).
* **Identified Limitation:** Highly sensitive to illumination shifts and shadows; cannot classify semantic object categories.
* **Project Relevance:** Reused as a low-power motion-trigger stage before executing full deep neural network inference.

### Paper 02: Vision & Voice Fall Monitoring
* **Reference:** *Real-Time Fall Monitoring for Seniors via YOLO and Voice Interaction*, MDPI **Future Internet**, vol. 17, no. 8, art. 324 (2025). *(Authors and DOI: copy from the MDPI page before submission.)*
* **Problem:** Vision-only fall detectors raise false alarms; the paper adds a spoken confirmation step before alerting.
* **Core Method:** YOLO-based fall detection combined with a voice-dialogue confirmation step (speech recognition, mixing local and Google STT).
* **Data / Hardware:** See the paper (datasets and platform to be noted in the reading sheet).
* **Metrics / Performance:** 97.5% detection accuracy; the voice confirmation reduced false positives from 12 to 3 (≈ 75%) in the reported test.
* **Identified Limitation:** Part of the speech pipeline relies on cloud STT, so response time depends on the network; the voice step is a one-way confirmation, not a two-way caregiver channel.
* **Project Relevance:** Closest prior work to our voice check-in. We keep the idea of voice confirmation but run it on-device and add a bidirectional push-to-talk channel plus a measured latency comparison.

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

### Paper 05: Audio Delay in Web Conference Tools
* **Reference:** K. Tsioutas and G. Xylomenos, *Audio Delay in Web Conference Tools*, Proc. WECML 2022 (workshop; no DOI given in the text).
* **Problem:** How large is the real mouth-to-ear delay of common web conferencing tools, and are they usable for latency-critical audio?
* **Core Method:** Impulse signals are played and looped back so that both are recorded on two channels of one audio interface (single clock, no endpoint synchronisation); 50 pulses per 60 s run; two-way delay is measured and one-way is estimated as half. LAN and WAN (ADSL) tests.
* **Data / Hardware:** Windows 10 laptops, university LAN and a home ADSL line; Sonobus, Jitsi, Teams, Zoom, Facebook, Skype, Google Meet.
* **Metrics / Performance:** Two-way delay (WAN / LAN): Sonobus 67 / 46 ms; Jitsi 259 / 235 ms (lowest among web tools); Teams 363 / 326 ms; Zoom 400 / 388 ms; Google Meet 1025 / 560 ms.
* **Identified Limitation:** Small data set from a single date and time; server location and routing not controlled; echo-cancellation workarounds may have changed tool behaviour; no embedded devices.
* **Project Relevance:** Method reference for our single-clock, two-channel latency measurement (Section 4 of `evaluare.md`). Its numbers give a realistic range for WebRTC-based tools and support the target of < 200 ms median being measured, not assumed.

### Paper 06: WebRTC on Resource-Constrained Devices
* **Reference:** B. Diallo, A. Ouamri, M. Keche, *A Hybrid Approach for WebRTC Video Streaming on Resource-Constrained Devices*, MDPI Electronics, vol. 12, no. 18, art. 3775 (2023). DOI: 10.3390/electronics12183775.
* **Problem:** Can browser-based WebRTC video run efficiently on constrained devices, and do hybrid apps (React Native, Electron) help?
* **Core Method:** Compares browser P2P WebRTC with hybrid clients (React Native on Android, Electron on Raspberry Pi) and checks interoperability.
* **Data / Hardware:** Raspberry Pi 2 Model B, Android smartphones, a Windows desktop; WLAN only; 21 calls of 5 minutes at 320×240, 25 fps.
* **Metrics / Performance:** CPU, RAM and bytes per second sampled every 5 s. On the Raspberry Pi the Electron app averaged about 56% CPU and 38% RAM versus above 60% CPU and 41% RAM for the browsers; on Android the React Native app used about 84–85% CPU versus about 100% for Chrome/Firefox; all mixed web/hybrid calls interoperated.
* **Identified Limitation:** No latency measurement; low resolution; WLAN only; small number of calls; Raspberry Pi 2, not Pi 5; no audio analysis.
* **Project Relevance:** Shows that WebRTC between a Raspberry Pi and a React Native client is feasible and gives a resource-measurement method (CPU/RAM sampling). Latency, audio quality and WAN behaviour, which it does not cover, are measured in our evaluation.

### Paper 07: Robotic Fall Detection with Voice Dialogue
* **Reference:** D. Feilacher, A. Jakl, B. Girsule, S. Lehner, M. Heller, H. Smith, S. Dominka, *Enhancing Home Safety through Multisensory Robotic Detection of Fallen Persons and Fall Hazard Warning*, Sensors & Transducers, vol. 271, no. 4, pp. 13–23 (December 2025).
* **Problem:** Detect a person lying on the floor at home and warn about fall hazards, with the user confirming by voice.
* **Core Method:** "Smart Companion", a robot vacuum (Dreame D10s Plus with Valetudo firmware) takes an image every two seconds; a fine-tuned YOLOv8x (classes person / lying person / no person) raises an alert from score thresholds over recent images; a voice dialogue confirms before emergency services are notified with a picture.
* **Data / Hardware:** About 21k training and 5k test images; Raspberry Pi 4B controller, Jetson Orin Nano base station for inference, Echo Dot microphone and cloud voice service; field study with six older adults (184 days, over 100k images).
* **Metrics / Performance:** Recall 0.86 and precision 0.93; one false alarm in six months; user ratings of 4.8/5 for trust in detection and 4.9/5 for privacy protection.
* **Identified Limitation:** Only six participants; no response-time measurement and not designed for immediate emergency response; periodic image capture; heavy model on a separate base station; depends on robot firmware and cloud voice.
* **Project Relevance:** Closest system-level prior work. Our project differs by running detection on a single Raspberry Pi 5, adding a two-way push-to-talk channel and quantitatively evaluating latency, transport options and power loss. It also gives a precedent for event-level metrics and a field-study design.

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

### Paper 11: Fall Detection Survey
* **Reference:** X. Wang, J. Ellul, G. Azzopardi, *Elderly Fall Detection Systems: A Literature Survey*, Frontiers in Robotics and AI, vol. 7, art. 71 (2020). DOI: 10.3389/frobt.2020.00071.
* **Problem:** Summarise fall detection with sensor networks and IoT and identify open issues.
* **Core Method:** Literature survey of wearable, ambient and camera-based approaches and of sensor fusion.
* **Data / Hardware:** Not applicable (survey).
* **Metrics / Performance:** Concludes that single-sensor systems suffer mostly from false alarms and that fusing signals from several sensors can give higher accuracy, fewer false alarms and more robustness.
* **Identified Limitation:** Survey only; predates recent YOLO-based edge systems and says nothing about two-way voice communication.
* **Project Relevance:** Justifies combining vision with ambient sensing and voice confirmation, and frames false alarms as the central problem.

### Paper 12: Fall Detection on an Embedded Platform
* **Reference:** B. Kwolek, M. Kepski, *Human fall detection on embedded platform using depth maps and wireless accelerometer*, Computer Methods and Programs in Biomedicine, vol. 117, no. 3, pp. 489–501 (2014). DOI: 10.1016/j.cmpb.2014.09.005.
* **Problem:** Low-cost fall detection with a very low false-alarm ratio.
* **Core Method:** An accelerometer flags a potential fall; depth-map analysis and an SVM classifier confirm it.
* **Data / Hardware:** PandaBoard ES embedded platform, Kinect depth camera, wireless accelerometer; introduces the UR Fall Detection Dataset (70 sequences: 30 falls and 40 activities of daily living).
* **Metrics / Performance:** All intentional falls were detected; sensitivity, specificity, precision and accuracy are reported on the UR dataset.
* **Identified Limitation:** Needs a depth sensor and a worn accelerometer; not RGB-only, not YOLO, no voice interaction.
* **Project Relevance:** Source of the UR Fall Detection Dataset used for evaluation. The dataset page labels frames -1 (not lying), 1 (lying) and 0 (transition, not used for classification) and does not define a sitting class, so the label-to-event mapping in `evaluare.md` must be documented explicitly.

### Paper 13: YOLO
* **Reference:** J. Redmon, S. Divvala, R. Girshick, A. Farhadi, *You Only Look Once: Unified, Real-Time Object Detection*, CVPR 2016, pp. 779–788. DOI: 10.1109/CVPR.2016.91.
* **Problem:** Real-time object detection with a single network pass.
* **Core Method:** Frames detection as regression from full images to bounding boxes and class probabilities.
* **Data / Hardware:** Titan X GPU; PASCAL VOC and artwork datasets.
* **Metrics / Performance:** 45 FPS for the base model and 155 FPS for Fast YOLO on a GPU.
* **Identified Limitation:** More localisation errors than region-based detectors, weaker on small objects; GPU numbers do not transfer to a Raspberry Pi CPU.
* **Project Relevance:** Foundational reference for the detector family. YOLOv8 itself is an Ultralytics release (AGPL) rather than a peer-reviewed paper, so this and Paper 03 are the citable sources.

### Paper 14: Integer Quantization
* **Reference:** B. Jacob, S. Kligys, B. Chen, M. Zhu, M. Tang, A. Howard, H. Adam, D. Kalenichenko, *Quantization and Training of Neural Networks for Efficient Integer-Arithmetic-Only Inference*, CVPR 2018, pp. 2704–2713. DOI: 10.1109/CVPR.2018.00286.
* **Problem:** Run neural networks efficiently on devices without fast floating-point hardware.
* **Core Method:** Integer-only quantization scheme with a co-designed training procedure.
* **Data / Hardware:** MobileNets on ARM CPUs; ImageNet classification and COCO detection.
* **Metrics / Performance:** Better accuracy-latency trade-off than floating point on typical ARM CPUs, with accuracy close to the float model.
* **Identified Limitation:** Quantization-aware training, not the post-training INT8 export used with YOLO on Raspberry Pi; Paper 03 shows INT8 was not feasible with NCNN on the Pi 5.
* **Project Relevance:** Background for the FP32/FP16/INT8 comparison in Scenario B; results must be measured on our own pipeline.

---

## 2. Comparative Synthesis Table

| # | Reference | Domain / Focus | Key Method | Reported Results | Missing Feature / Limitation | Adaptation in Project |
| :-: | :--- | :--- | :--- | :--- | :--- | :--- |
| **01** | Bradski et al. | Edge (CPU) | Frame Differencing | Sub-10ms execution | No object classification | Pre-filter for YOLO activation |
| **02** | *Future Internet* 17(8) (2025) | Hybrid Edge/Cloud | YOLO + Voice Dialog | 97.5% accuracy; false positives 12 → 3 | Cloud STT; one-way confirmation only | On-device voice prompts + two-way PTT |
| **03** | Rey et al. (2025) | Edge (ARM) | YOLO Benchmark (NCNN, FP32) | YOLOv8n ≈ 8.5 FPS on RPi 5 | No INT8/FP16 on RPi 5; drone context | Realistic baseline for O2; frame gating |
| **04** | de Miguel et al. (2017) | Edge (CPU) | Ratio Features + kNN | 96% sensitivity, 97.6% specificity | 2D only; no voice confirmation | Geometric heuristic validation |
| **05** | Tsioutas & Xylomenos (WECML 2022) | Network / Audio | Single-clock loopback delay | Jitsi 259 / 235 ms two-way (WAN / LAN) | Small, single-day data set; no embedded device | Latency measurement method |
| **06** | Diallo et al. (2023) | Edge (Pi) / Mobile | Hybrid vs. browser WebRTC | Electron ≈ 56% CPU vs. > 60% (browsers) on Pi 2 | No latency; 320×240; WLAN only | CPU/RAM sampling; Pi ↔ React Native feasibility |
| **07** | Feilacher et al. (2025) | Robot / Home | YOLOv8x + voice dialogue | Recall 0.86, precision 0.93; 1 false alarm / 6 months | 6 users; no response time; separate base station | Single-Pi design, measured latency |
| **08** | Sulzer et al. (2022) | Sensor Accuracy | Calibration vs. Reference | < 0.1 K after calibration; BME280 raw ≈ 1 K | No interface comparison; no SHT4x | Per-unit calibration of BME280/SHT4x |
| **09** | Zheng et al. (FAST 2013) | Storage / OS | Power-Fault Injection | 13 of 15 SSDs showed failures | SSDs, not SD cards; no UPS tested | Motivates UPS HAT; own power-cut test |
| **10** | Sacchetto et al. (JAES 2022) | Transport Layer | WebRTC Data-Channel PCM Audio | Outperforms standard RTCPeerConnection | Music use case; no WebSocket/Firebase | Latency measurement reference |
| **11** | Wang et al. (2020) | Survey | Literature review | Sensor fusion lowers false alarms | Predates edge YOLO systems | Motivates vision + voice + ambient fusion |
| **12** | Kwolek & Kepski (2014) | Embedded / Dataset | Depth + accelerometer + SVM | All intentional falls detected | Needs depth sensor; no sitting label | UR dataset and label mapping |
| **13** | Redmon et al. (2016) | Detection | Single-pass detector | 45 FPS (GPU) | GPU numbers; weak on small objects | Detector family reference |
| **14** | Jacob et al. (2018) | Edge inference | Integer-only quantization | Better accuracy-latency on ARM | QAT, not post-training INT8 | FP32/FP16/INT8 comparison |

---

## 3. Positioning and Novelty

Commercial solutions are largely closed IP cameras that stream video to third-party clouds, which raises privacy concerns for indoor residential monitoring. Academic work already combines YOLO-based fall detection with voice confirmation (Paper 02) and robot-based home monitoring with voice dialogue (Paper 07), and surveys report that sensor fusion reduces false alarms (Paper 11). What these works do not provide is a quantitative, reproducible evaluation of the communication and power layers on a single low-cost edge device.

This bachelor's thesis therefore positions itself as an **edge-autonomous, privacy-first companion** and claims the following contributions, each to be demonstrated by measurement:
1. **Edge-Native Inference:** Detection runs on the local Raspberry Pi 5; raw video stays on the device, and only short event clips are uploaded when an alert is escalated.
2. **Hybrid Incident Resolution:** False alarms are dismissed on-device via voice verification ("Are you okay?") and a physical "I'm OK" button, and a caregiver can talk back through push-to-talk.
3. **Transport Comparison:** A measured comparison of WebRTC, WebSocket chunked streaming and file-based audio transfer for voice, using a single-clock latency method (Paper 05) and PESQ/STOI quality metrics.
4. **Power-Loss Resilience:** A repeated power-cut experiment with and without a UPS HAT, which the reviewed literature does not cover for SD-card-based devices (Paper 09 concerns SSDs).
