# SCAI-2026-SmartShield
ESP32-based smart worker safety monitoring system featuring gas, temperature, humidity, and motion sensing, OpenCV fatigue detection, and an intelligent risk prediction dashboard.
# Smart Worker Safety & Environmental Hazard Monitoring System

**IEEE — Smart Sensing & Intelligent Electronic Systems**
Team RAKSHAK · 2026

Trend-based hazard sensing, contactless fatigue detection, and an explainable ML risk model — built and demoed entirely through **online simulation**.

---

## 📌 Project Status

> **Current stage: circuit setup✅**
> The full pipeline — smart sensing simulation done only building the electronic circuit (Cirkit Designer ). Real hardware deployment has **not** been done yet.
> code writing and logic framing(on going) and researching for additional suitable features
---

## 1. The Problem

Traditional workplace safety systems react **late** — only after a hazard has already become dangerous.

| Issue | Why it matters |
|---|---|
| **Gas & fume buildup** | Toxic/combustible gas concentrations rise gradually. Most systems only alarm after crossing a fixed danger threshold, losing the early-warning window. |
| **Falls go unnoticed** | In remote or noisy industrial zones, a fall or impact may go unseen or unreported for critical minutes. |
| **Fatigue is invisible** | Drowsy or exhausted workers are far more accident-prone, yet fatigue is almost never actively monitored on the floor. |

A control room needs **early trends**, not late alarms.

---

## 2. The Solution

A wearable-plus-camera system that watches the **trend**, not the threshold.

- **Rolling-window anomaly detection** — every sensor is compared against its own recent trend; a rising slope triggers an alert before a fixed danger line is reached.
- **Contactless fatigue detection** — a webcam-based Eye Aspect Ratio (EAR) model flags drowsiness in real time, no wearable needed for this channel.(in progress)
- **Explainable ML risk scoring** — a simple, interpretable model fuses all channels into one **Safe / Caution / Critical** label — never a black box.(in progress)
- **Live control-room dashboard** — plain-language status, colour-coded readings, and trend graphs built for a supervisor, not an engineer.

---

## 3. Architecture

Data flow from sensor to decision:

```
Sensing layer
  MQ2 gas · MPU6050 motion · DHT22 temp/humidity + webcam fatigue channel (OpenCV)
        │
        ▼
Data bridge
  ESP32-S3 (Cirkit Designer) → JSON over WiFi/HTTPS → Flask bridge server
        │
        ▼
ML risk classifier(in progress)
  Rolling features → explainable Safe / Caution / Critical prediction
        │
        ▼
Safety dashboard(in progress)
  Colour-coded status, plain-language alerts, live trend graphs
```

---

## 4. Components List

All components below have been simulated end-to-end (no physical build yet).

| # | Component | Type | Description |
|---|---|---|---|
| 1 | **ESP32-S3** | Microcontroller | Central controller handling sensing, alert logic, and WiFi/HTTPS communication to the bridge server. |
| 2 | **MQ2 Gas Sensor** | Analog sensor | Measures gas/smoke concentration; feeds the rolling-window anomaly detector for early gas-leak warning. |
| 3 | **MPU6050** | I2C accelerometer/gyroscope | Detects motion patterns and falls/impacts via acceleration spikes. |
| 4 | **DHT22** | Temperature & humidity sensor | Tracks heat-stress trend (temperature and humidity over time). |
| 5 | **SSD1306 OLED Display** | On-device display | Shows a live on-device status readout at the sensor node. |
| 6 | **Buzzer + LED** | Local alert module | Provides an immediate local audible + visual alert, independent of network connectivity. |
| 7 | **Webcam + OpenCV / MediaPipe FaceMesh** | Vision module | Captures facial landmarks live to compute the Eye Aspect Ratio (EAR) for contactless fatigue detection.(in progress) |
| 8 | **ML Risk Classifier** (Logistic Regression / Decision Tree) | Software / ML model | Uses rolling mean, standard deviation, and rate-of-change features (gas, temp, motion) plus the fatigue score to output an explainable Safe/Caution/Critical label.(in progress) |
| 9 | **Safety Dashboard** | Software / frontend | Displays colour-coded cards, live trend graphs, and a timestamped alert history log for supervisors.(in progress) |

---

## 5. How It Works

### 5.1 Trend Detection (not static thresholds)
1. **Continuous sampling** — gas, temperature, and acceleration are read every cycle and pushed into a rolling 10-sample window.
2. **Compare to recent average** — each new reading is compared to the window's own moving average, not a single hardcoded number.
3. **Flag a rising trend** — a jump beyond a set delta (e.g. gas +15, temp +2°C, or an acceleration spike) marks a genuine anomaly.
4. **Alert with cooldown** — buzzer + LED fire immediately, with an 8-second cooldown so one event doesn't spam repeatedly.

*Why this matters:* catching a developing hazard early gives workers time to react — a static threshold alarm only fires once it's already severe.

### 5.2 Contactless Fatigue Detection (OpenCV)(in progress)
- **Face & eye tracking** — MediaPipe FaceMesh locates facial landmarks live from the webcam feed.
- **Eye Aspect Ratio (EAR)** — a geometric ratio computed from eye landmarks that drops sharply when eyes close.
- **Sustained-closure check** — if EAR stays below threshold for consecutive frames, it's classified as drowsiness, not a blink.
- **Fused into the risk model** — the fatigue flag joins the hazard data as a second, independent input to the ML classifier.

Why EAR works well for a live demo: no training data needed (purely geometric), works on any webcam, and is triggerable on demand (just close your eyes on camera).

### 5.3 Explainable Risk Model(in progress)
- **Feature engineering** — rolling mean, standard deviation, and rate-of-change for gas, temperature, and motion, plus the live fatigue score.
- **Simple, explainable model** — logistic regression or a decision tree; every prediction can be traced back to which feature drove it.
- **Three-tier output** — Safe · Caution · Critical, a label a supervisor can act on immediately.
- **Sanity-checked** against the rule-based thresholds from the sensing layer — the model must agree with common sense before it's trusted.

---

## 6. Dashboard(in progress)

Built for a supervisor, not an engineer:

- **Colour-coded cards** — green / amber / red at a glance (Gas, Temp, Humidity, Motion).
- **Live trend graphs** — see the pattern building, not just a single number.
- **Alert history log** — timestamped record for later review.

---

## 7. Advantages

- Early, trend-based warning — not reactive alarms
- Dual-modality sensing: wearable hazard data + contactless fatigue
- Explainable ML — every risk label is traceable, not a black box
- Fully simulated online — no hardware dependency for demos
- Plain-language dashboard usable by non-technical supervisors
- Local alerting works even if WiFi drops mid-operation

---

## 8. Current Progress vs. Ongoing Work

### ✅ Completed
- End-to-end simulation of the sensing layer (MQ2, MPU6050, DHT22) in Cirkit Designer
- ESP32-S3 firmware logic for rolling-window anomaly detection

### 🔄 Ongoing / Immediate Next Steps
- Testing the ML classifier against a wider range of simulated anomaly scenarios
- Refining EAR thresholds for more robust drowsiness detection across different lighting/webcam conditions
- Tuning rolling-window parameters (window size, delta thresholds) for fewer false positives
- Integrating and stress-testing the full sensor-to-dashboard pipeline under simulated network drop conditions
- Documentation and packaging of the simulation codebase for handoff/reproducibility
- Contactless fatigue detection module (OpenCV + MediaPipe EAR)
- Explainable ML risk classifier (Safe / Caution / Critical)
- Flask bridge server for JSON data transport over WiFi/HTTPS
- Live control-room dashboard with colour-coded cards, trend graphs, and alert history

---

## 9. Future Improvements

| Term | Improvement | Description |
|---|---|---|
| **Near term** | Real-hardware validation | Move from simulation to a physical prototype; calibrate MQ2 against certified gas references. |
| **Mid term** | PPE compliance detection | Extend the CV module with helmet/vest detection alongside fatigue monitoring. |
| **Mid term** | Multi-worker deployment | Scale the dashboard to track and rank risk across an entire team in one view. |
| **Long term** | Predictive risk forecasting | Move from "detect now" to "predict the next 10 minutes" using time-series forecasting on the same feature set. |

Additional ideas worth exploring:
- Cloud-based historical data storage and analytics
- Mobile app / SMS alerts for supervisors off-site
- Integration with existing industrial SCADA/IoT platforms
- Battery/power optimization for a wearable form factor
- Edge-deployment of the ML model directly on the ESP32-S3 to reduce latency

---

## 10. Tools & References

- Project repository: `<add GitHub/GitLab link>`
- Simulation environment (Cirkit Designer): `[<add link>](https://app.cirkitdesigner.com/)`
- Live dashboard demo: ``
- Presentation slides: ``
- Related datasheets:
  - MQ2 Gas Sensor: `https://www.pololu.com/file/0j309/mq2.pdf`
  - MPU6050: `(https://cdn.sparkfun.com/datasheets/Sensors/Accelerometers/RM-MPU-6000A.pdf`
  - DHT22: `https://cdn.sparkfun.com/assets/f/7/d/9/c/DHT22.pdf`
  - ESP32-S3: `https://cdn.sparkfun.com/datasheets/IoT/esp32_datasheet_en.pdf`

---

## 11. Team

- Smart Worker Safety & Environmental Hazard Monitoring System — Team Project, 2026
- IEEE · Smart Sensing & Intelligent Electronic Systems

---

*This README was generated from the project's presentation deck and reflects progress up to the simulation stage. Update the "Ongoing / Immediate Next Steps" and "Tools & References" sections as the project moves toward hardware validation.*
