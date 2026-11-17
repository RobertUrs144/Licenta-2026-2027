# Experimental Evaluation & Testing Protocol

*File path: `docs/evaluare.md`*

---

## 1. Evaluation Dimensions & Quantitative Metrics

To satisfy the technical requirements of the UPT Faculty of Automation and Computers, all system objectives are paired with empirical, reproducible metrics:

| Dimension | Metrics & Units | Evaluation Methodology | Baseline / Reference |
| :--- | :--- | :--- | :--- |
| **Audio Performance (O3, O5)** | • **Mouth-to-Ear Latency** ($ms$), both directions (edge → caregiver and caregiver → edge)<br>• **Time-to-First-Audio** after the push-to-talk press ($ms$)<br>• **Jitter** ($ms$) and **Packet Loss** (%) — defined per transport (see methodology)<br>• **Voice Intelligibility** (PESQ as MOS-LQO, ≈ 1.0–4.5; STOI, 0–1) | Latency: a calibrated reference signal (tone/click) is split so that it is recorded on channel 1 of a two-channel audio interface (or two-channel oscilloscope) and, after the full path, the client speaker output is recorded on channel 2. Both channels share **one clock**, so the delay is obtained by cross-correlation and no PTP/NTP synchronization between devices is needed. The measured path includes capture, codec, buffering, transport and playout.<br>Intelligibility: standard speech clips (not tones), identical codec and bitrate across transports (or documented differences), compared against the clean reference with PESQ and STOI.<br>Jitter / loss: for WebRTC, taken from `getStats()` (jitter, packets lost, jitter-buffer delay); for TCP-based transports (WebSocket, HTTP, Firebase), where network loss appears as latency, use application-level metrics: frame inter-arrival jitter, playout underruns and stall count.<br>Evaluated across 3 transports:<br>1. *Record-and-play* (Firebase Storage)<br>2. *Chunked HTTP/WebSocket*<br>3. *WebRTC bidirectional audio* | Previous PiCam implementation: no audio support (file transfer latency: 2500–4000 ms). |
| **Edge Vision Inference (O2)** | • **Sustained FPS** ($frames/sec$): capture, inference and end-to-end reported separately<br>• **Inference Latency** ($ms$) per pipeline stage<br>• **Frame-level Precision, Recall, F1-Score** on *Upright*, *Sitting*, and *Fallen* classes<br>• **Event-level** fall sensitivity and specificity, false alarms per day, detection delay ($s$)<br>• **Precision trade-off:** FP32 vs. FP16 vs. INT8 (accuracy vs. FPS) | Benchmarked against the UR Fall Detection Dataset and 50 locally recorded scenarios under controlled lighting conditions. The dataset's own frame labels must be mapped explicitly to the *Upright / Sitting / Fallen* classes (verify the label scheme on the dataset page). The scenario matrix includes forward, backward and lateral falls, falls from a chair, and non-fall activities (rapid sitting, bending, deliberate lying on a sofa). FPS is measured **while the WebRTC audio and sensor tasks run concurrently**. Monitored via OpenCV timestamps and `psutil`. Recorded participants give informed consent (GDPR). | Cloud Vision API in PiCam: 500–2000 ms latency per call, ~5 FPS maximum frame throughput. |
| **System & Thermal Dynamics (O1)** | • **CPU Temperature** ($^\circ C$)<br>• **ARM Clock Frequency** (MHz) and throttling / undervoltage flags<br>• **Power Dissipation** ($W$ and $mA$)<br>• **UPS Battery Runtime** ($minutes$) at idle and at typical load, until graceful shutdown | In-line USB-C power meter logging under 100% compute load (continuous YOLO inference + active WebRTC audio) over 60-minute stress runs, with ambient temperature fixed and logged. Battery runtime is measured separately by cutting mains power and timing the interval until the daemon completes a graceful shutdown. | Raspberry Pi 4B without active cooling: frequent thermal throttling at $>80^\circ C$. |
| **Power-Loss Resilience (O1)** | • **Boot Success Rate** (%) after forced power cuts<br>• **Filesystem Integrity** (`fsck` / journal errors, count)<br>• **Graceful Shutdown Time** ($s$) | Repeated forced power cuts (target: 30–50 per configuration) **with and without** the UPS HAT, during active writes (event clips, logs). After each cut: boot, run a filesystem check and verify that the application restarts. Results are compared between the two configurations. | Raspberry Pi without UPS: filesystem corruption risk on abrupt power loss. |
| **Sensor Reliability (O1)** | • **Read Latency** ($ms$)<br>• **Read Failure Rate** (%) over 24 h<br>• **Bias vs. Reference Thermometer** ($^\circ C$, %RH) | BME280/SHT4x polled over I²C for 24 h and compared with a reference thermometer/hygrometer. The sensor is mounted away from the CPU and heatsink to limit self-heating, and a per-unit calibration offset is applied if needed. | PiCam DHT22: read latency of 3+ seconds and driver instability on 64-bit kernels. |
| **Alert Verification (O4)** | • **Alert Trigger Time** ($s$ - event to alarm)<br>• **Detection → Voice Prompt Latency** ($s$)<br>• **Escalation Time** ($s$ - until the caregiver app receives the notification)<br>• **Button Dismissal Reliability** (%) and **Missed Escalations** (count)<br>• **False Alarms** with vs. without voice check | 50 physical trial runs evaluating fall detection, local voice prompt playback (with a fixed response timeout), and tactile "I'm OK" dismissal versus caregiver escalation. Success rates are reported with Wilson 95% confidence intervals. | Passive asynchronous logging without local voice verification. |

---

## 2. Experimental Measurement Setup

```
+-------------------------------------------------------------------+
|                     LATENCY MEASUREMENT BENCH                     |
|                                                                   |
|   +-------------------+                                           |
|   | Audio Generator   |---- reference signal ----+                |
|   | (Tone / Click /   |                          |                |
|   |  Speech clip)     |                          v                |
|   +---------+---------+              +------------------------+   |
|             |                        | 2-channel recorder     |   |
|             v                        | (single shared clock)  |   |
|     [ Edge Microphone ]              |  ch1: reference (T0)   |   |
|             |                        |  ch2: speaker out (T1) |   |
|             v                        +------------^-----------+   |
|   [ Network Transport ]                           |               |
|   (WebRTC / WebSocket / Firebase)                 |               |
|             |                                     |               |
|             +-----------> [ Client Speaker ] -----+               |
|                                                                   |
|   Repeat in the reverse direction (client mic -> edge speaker).   |
+-------------------------------------------------------------------+
```

Mouth-to-ear latency is computed as the delay between the two recorded channels (cross-correlation peak):

$$\text{Mouth-to-Ear Latency} = T_1 - T_0$$

Because both channels are sampled by the same converter clock, no clock synchronization between the edge device and the caregiver's phone is required. The same procedure is repeated in the reverse direction (caregiver microphone to edge speaker).

---

## 3. Benchmark Scenarios

### Scenario A: Network Variation Test
Evaluate all three audio transport pipelines across four network profiles using `tc netem` (Traffic Control) on Linux. The exact `netem` parameters (delay, jitter, loss) are logged for every run, impairment is applied in both directions (e.g. via an `ifb` device or on an intermediate bridge/router), and the values below are the configured targets, not measured results:
1. **Ideal LAN:** no added impairment (baseline); the measured RTT and jitter are recorded.
2. **Typical Wi-Fi:** 2% packet loss, 15 ms jitter, 30 ms RTT.
3. **Degraded WAN / 4G:** 5% packet loss, 50 ms jitter, 100 ms added delay.
4. **Stress profile:** 10–15% packet loss (optionally bursty loss), 50 ms jitter, 100 ms added delay.

Test conditions to be fixed and documented: whether WebRTC connects peer-to-peer (STUN) or through a TURN relay; the location of the WebSocket relay server; the audio chunk size used by the chunked and Firebase transports; and the codec and bitrate of each pipeline.

### Scenario B: Sustained Thermal Stress Run
Execute the full runtime stack (Camera Ingestion + YOLO Edge Inference + I²C Sensor Polling + WebRTC Stream) for 60 continuous minutes at a fixed, logged ambient temperature:
* Record CPU temperature and ARM clock frequency every 5 seconds.
* Check for thermal throttling and undervoltage flags via `vcgencmd get_throttled`.
* Compute FPS in 1-second windows; the run passes if FPS is at or above 15 in at least 95% of the windows, and the minimum and 1st-percentile FPS are reported.

### Scenario C: Power-Loss and Battery Test
* **Forced power cuts:** cut mains power during active writes, 30–50 times with the UPS HAT and the same number of times without it. After each cut, boot the device, run a filesystem check, record journal or `fsck` errors, and confirm that the application restarts.
* **Battery runtime:** cut mains power at idle and at typical load and time the interval until the UPS daemon completes a graceful shutdown; record the shutdown duration.

### Scenario D: Sensor Stability Run
Poll the BME280/SHT4x over I²C for 24 hours alongside a reference thermometer/hygrometer; record read latency, failed reads and the bias against the reference.

---

## 4. Statistical Protocol & Acceptance Criteria

* **Repetitions:** each audio condition (transport × network profile) is repeated at least 30 times; latency is reported as median, p95 and p99 with confidence intervals.
* **Proportions:** success rates from the 50-trial experiments (button dismissal, escalation, detection) are reported with Wilson 95% confidence intervals.
* **Reproducibility:** software versions, model files, input resolution, audio codec settings and ambient conditions are recorded for every run.
* **Acceptance criteria:**

| Objective | Pass Criterion |
| :--- | :--- |
| **O1** | No thermal throttling or undervoltage flags during the 60-minute run; graceful shutdown completes on battery; filesystem remains consistent after the forced power-cut series with the UPS. |
| **O2** | FPS ≥ 15 in at least 95% of 1-second windows on the Raspberry Pi 5 with no cloud vision calls. |
| **O3** | Mouth-to-ear latency < 200 ms (reported as median and p95) in the target network profile. |
| **O4** | Alert and voice prompt triggered after a detected fall; "I'm OK" button dismisses the alarm; escalation reaches the caregiver app when there is no response. |
| **O5** | All three transports measured under every network profile with identical test signals and the same statistical protocol. |