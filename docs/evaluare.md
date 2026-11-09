# Experimental Evaluation & Testing Protocol

*File path: `docs/evaluare.md`*

---

## 1. Evaluation Dimensions & Quantitative Metrics

To satisfy the technical requirements of the UPT Faculty of Automation and Computers, all system objectives are paired with empirical, reproducible metrics:

| Dimension | Metrics & Units | Evaluation Methodology | Baseline / Reference |
| :--- | :--- | :--- | :--- |
| **Audio Performance (O3, O5)** | • **Mouth-to-Ear Latency** ($ms$)<br>• **Jitter** ($ms$)<br>• **Packet Loss** (%)<br>• **Voice Intelligibility** (PESQ / STOI: 1.0–5.0) | Calibrated synthetic acoustic tones injected into the microphone; playback timestamps captured via PTP/NTP clock synchronization.<br>Evaluated across 3 transports:<br>1. *Record-and-play* (Firebase Storage)<br>2. *Chunked HTTP/WebSocket*<br>3. *WebRTC bidirectional audio* | Previous PiCam implementation: no audio support (file transfer latency: 2500–4000 ms). |
| **Edge Vision Inference (O2)** | • **Sustained FPS** ($frames/sec$)<br>• **Inference Latency** ($ms$)<br>• **Precision, Recall, F1-Score** on *Upright*, *Sitting*, and *Fallen* classes | Benchmarked against the UR Fall Detection Dataset and 50 locally recorded scenarios under controlled lighting conditions. Monitored via OpenCV timestamps and `psutil`. | Cloud Vision API in PiCam: 500–2000 ms latency per call, ~5 FPS maximum frame throughput. |
| **System & Thermal Dynamics (O1)** | • **CPU Temperature** ($^\circ C$)<br>• **Power Dissipation** ($W$ and $mA$)<br>• **UPS Battery Runtime** ($minutes$) | In-line USB-C power meter logging under 100% compute load (continuous YOLO inference + active WebRTC audio) over 60-minute stress runs. | Raspberry Pi 4B without active cooling: frequent thermal throttling at $>80^\circ C$. |
| **Alert Verification (O4)** | • **Alert Trigger Time** ($s$ - event to alarm)<br>• **Button Dismissal Reliability** (%) | 50 physical trial runs evaluating fall detection, local voice prompt playback, and tactile "I'm OK" dismissal versus caregiver escalation. | Passive asynchronous logging without local voice verification. |

---

## 2. Experimental Measurement Setup

```
+-------------------------------------------------------------------+
|                     LATENCY MEASUREMENT BENCH                     |
|                                                                   |
|   +-------------------+                     +-----------------+   |
|   | Audio Generator   |                     | Timestamp Log   |   |
|   | (Calibrated Tone) |                     | (Oscilloscope)  |   |
|   +---------+---------+                     +--------+--------+   |
|             | $T_0$                                  ^            |
|             v                                        | $T_1$      |
|     [ Edge Microphone ]                     [ Client Speaker ]    |
|             |                                        ^            |
|             +---------> [ Network Transport ] -------+            |
|                          (WebRTC / Chunked)                       |
|                                                                   |
|   $$\text{Mouth-to-Ear Latency} = T_1 - T_0$$                    |
+-------------------------------------------------------------------+
```

---

## 3. Benchmark Scenarios

### Scenario A: Network Variation Test
Evaluate all three audio transport pipelines across three network profiles using `tc` (Traffic Control) on Linux:
1. **Ideal LAN:** 0% packet loss, $<2$ ms jitter, $<5$ ms RTT.
2. **Typical Wi-Fi:** 2% packet loss, 15 ms jitter, 30 ms RTT.
3. **Degraded WAN / 4G:** 5% packet loss, 50 ms jitter, 100 ms added delay.

### Scenario B: Sustained Thermal Stress Run
Execute the full runtime stack (Camera Ingestion + YOLO Edge Inference + I²C Sensor Polling + WebRTC Stream) for 60 continuous minutes:
* Record CPU temperature every 5 seconds.
* Check for thermal throttling flags via `vcgencmd get_throttled`.
* Verify that FPS remains above the 15 FPS target throughout the entire test.