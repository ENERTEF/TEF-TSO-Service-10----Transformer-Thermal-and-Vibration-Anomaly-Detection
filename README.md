# Transformer Thermal and Vibration Anomaly Detection  
### Technical Manual & Service Specification v0.1

**Author:** Elektro Gorenjska d.d., Slovenian DSO  
**Version:** 1.0  
**Last updated:** 15 Dec 2025  

---

# 1. Business Context & Definitions

This service focuses on developing and operating a transformer vibration and thermography anomaly detection system for a pilot distribution transformer.

The solution combines:

- **Vibration data** from an MPU6050 accelerometer  
- **Infrared thermal images** from an MLX90640 sensor  

The sensing setup is already installed on a live transformer and collects:

- Vibration sampled at **1 kHz**, stored as **1-minute FFT snapshots**  
  - Each snapshot contains the **200 highest magnitude frequency components**
- Thermal images captured every **5 minutes**
  - Resolution: **32 × 24 pixels**

All data:
- Is timestamped and synchronized via **NTP**
- Is stored in an **on-prem SQL database**
- Has **>1 year of historical data available**

Additionally, sensor data can be aligned with:
- Current
- Voltage
- Active power
- Reactive power
- Flicker
- THD

### Purpose

The service will be used to:
- Detect abnormal transformer behaviour  
- Identify mechanical and thermal issues  
- Support preventive maintenance  
- Provide health-related signals for integration into the **AHI platform**

---

## Key Terms

- **Pilot Distribution Transformer**  
  Transformer where the monitoring system is deployed  

- **Vibration Data**  
  Accelerometer measurements indicating mechanical condition  

- **FFT Snapshot**  
  One-minute frequency-domain signal representation (top 200 components)  

- **Frequency Component**  
  Frequency + magnitude pair from FFT  

- **Infrared Thermal Image**  
  32×24 temperature matrix from MLX90640  

- **Thermal Pattern**  
  Spatial temperature distribution across the transformer  

- **Load and Power-Quality Data**  
  Electrical measurements aligned with sensor data  

- **Time Alignment**  
  Synchronization of all datasets via timestamps  

- **Baseline Behaviour**  
  Learned normal operating conditions  

- **Anomaly Detection**  
  Detection of deviations from baseline  

- **Preventive Maintenance Alarm**  
  Signal indicating inspection is needed  

- **AHI Platform**  
  Internal Asset Health Index system  

- **Health-Related Signal**  
  Output metric used by AHI  

---

## 1.1 Elektro Gorenjska Context

This service is designed for Elektro Gorenjska to:

- Improve visibility into transformer behaviour  
- Detect early degradation  
- Enable proactive maintenance  

Although currently deployed on a **single transformer**, the goal is:

- Build a **generalized solution**
- Use **standardized inputs**
- Scale to multiple transformers  

The system must:
- Learn **unit-specific behaviour**
- Maintain **consistent logic across all units**

### Responsibilities

- **Provider:** delivers analytical solution + methodology  
- **DSO:** provides data + integration into AHI  

The solution must be:
- Explainable  
- Reproducible  
- Scalable  

---

# 2. Problem Statement

Develop a **production-ready anomaly detection system** using:

- Vibration FFT data  
- Thermal images  
- Electrical operating data  

### Requirements

- Python-based solution  
- Containerized deployment (Docker)  
- Daily execution  
- Integration-ready (AHI platform)  

### Outputs

- Daily alarm indicator  
- Anomaly score  
- Supporting explanation  

### Input Data Types

- FFT snapshots  
- Load & PQ measurements  
- Thermal images  

### Design Principles

- Standardized inputs  
- No over-flexibility  
- Scalable architecture  
- Strictly **causal analysis** (no future data)

### Methods (provider-defined)

- Robust statistics  
- Machine learning  
- Clustering  
- Prediction-based anomaly detection  

### Explainability

Each alarm must include:

- Triggering variables  
- Statistical deviation  
- Baseline comparison  

If full explanation is not possible:
- Provide statistical context (median, percentiles, limits)

---

# 3. Data Description

### Data Characteristics

- Fully timestamped  
- NTP synchronized  
- Stored in SQL  
- Triggered via ESP32 devices  

No known:
- Clock drift  
- Major data quality issues  

Still required:
- Validation checks (missing/invalid/sync)

---

## 3.1 Transformer Metadata

| Variable | Name | Type | Unit | Description |
|----------|------|------|------|-------------|
| Transformer ID | id | Str | - | Unique ID |
| Deployment Year | deploy_year | Int | - | Installation year |
| Power Rating | power_rating | Float | kVA | Nominal transformer power |
| Location address | address | Str | - | City, address |
| Latitude | latitude | Float | Decimal degrees | Geographic position |
| Longitude | longitude | Float | Decimal degrees | Geographic position |

---

## 3.2 Vibration FFT Snapshots

| Variable | Name | Type | Unit | Description | Example |
|----------|------|------|------|-------------|--------|
| Transformer ID | id | String | - | Unique ID | TR-001 |
| Timestamp | timestamp | Int | YYYY-MM-DD HH:MM:SS | Acquisition time | 2026-03-30 08:00:00 |
| Sampling frequency | fs | Int | Hz | Sampling rate | 1000 |
| FFT window | fft_window | Int | s | Duration | 60 |
| Frequency bin | fft_bin | Float | Hz | Frequency | 50.0 |
| Magnitude | fft_magnitude | Float | - | Amplitude | 0.018 |

Each snapshot contains **200 dominant frequency components**.

---

## 3.3 Infrared Thermal Images

| Variable | Name | Type | Unit | Description | Example |
|----------|------|------|------|-------------|--------|
| Transformer ID | transformer_id | String | - | Unique ID | TR-001 |
| Timestamp | timestamp | Timestamp | YYYY-MM-DD HH:MM:SS | Capture time | 2026-03-30 08:00:00 |
| Image width | image_width | Integer | pixels | Width | 32 |
| Image height | image_height | Integer | pixels | Height | 24 |
| Thermal matrix | thermal_matrix | Array | °C | Full matrix | [[21.3, ...]] |

Derived features may include:
- Max/min temperature  
- Gradients  
- Regions  

---

## 3.4 Transformer Operating & PQ Data

| Variable | Name | Type | Unit | Description |
|----------|------|------|------|-------------|
| Energy | energy | Float | kWh | Energy |
| THD current | THD_I1–3 | Float | % | Harmonics |
| THD voltage | THD_U1–3 | Float | % | Harmonics |
| Active power | P1–3 | Float | kW | Power |
| Reactive power | Q1–3 | Float | kvar | Power |
| Apparent power | S1–3 | Float | kVA | Power |
| Max power | P_max/Q_max/S_max | Float | kW/kvar/kVA | Max values |
| Total power | P_total/Q_total/S_total | Float | kW/kvar/kVA | Total |
| Voltage | U1–3 | Float | V | Voltage |
| Voltage extrema | U_max/min | Float | V | Range |
| Power factor | ePF1–3 | Float | - | Phase PF |
| Total PF | PF | Float | - | System PF |
| Current | I | Float | A | Total |
| Phase current | I1–3 | Float | A | Phase |
| Current extrema | I_max/min | Float | A | Range |
| Neutral current | Inc | Float | A | Neutral |
| Neutral max | Inc_max | Float | A | Max |
| Voltage unbalance | Uu | Float | % | Imbalance |
| Flicker | flicker | Float | - | PQ metric |

---

# 4. Analytics, Scope & Update Frequency

### Temporal Scope
- Rolling historical window  
- Strictly causal  

### Analytical Scope
- Combine vibration + thermal + electrical data  
- Learn baseline behaviour  
- Detect deviations  

### Update Frequency
- Daily execution  

### Output

- Alarm flag  
- Anomaly score  
- Baseline reference  
- Deviation metrics  
- Contributing variables  
- Statistical context  

### Scalability
- Designed for multiple transformers  

---

# 5. Evaluation Protocols & Metrics

### Focus
- Reliability  
- Accuracy  
- Stability  
- Explainability  

---

## 5.1 Data Protocol

- Rolling historical data  
- No future data  
- Synthetic anomalies allowed  
- Fully reproducible  

---

## 5.2 Data Gaps

- Exclude corrupted data  
- Log all issues  
- Assume stable inputs  

---

## 5.3 KPIs

| KPI | Description | Target | Method |
|-----|-------------|--------|--------|
| Detection Performance | Accuracy | >95% | Precision/Recall/F1 |
| False Alarm Rate | False positives | ≤5/year | Baseline replay |
| Detection Delay | Speed | Minimize | Synthetic tests |
| Score Stability | Variability | <10% CV | Rolling analysis |
| Explainability | Explained alarms | 100% | Audit |
| Context Coverage | Statistical context | 100% | Logs |
| Data Coverage | Daily runs | 100% | Logs |
| Reproducibility | Same output | 100% | Rerun |

---

# 6. Deliverables & Submissions

The provider delivers:
- Analytical solution  
- Documentation  
- Deployment artefacts  

---

## 6.1 Reports

1. **Pre-Service Report**
   - Methodology  
   - Data requirements  
   - Architecture  

2. **Intermediate Report**
   - Early results  
   - KPI progress  
   - Issues  

3. **Final Report**
   - Full evaluation  
   - Lessons learned  
   - Scaling recommendations  

---

## 6.2 Technical Deliverables

- Python code (Dockerized)  
- Full documentation  
- ML models (if used)  
- Example I/O setup  
- Deployment manual  
- Handover session  
- Training session  

### Security

- NDA required  
- Fully on-prem  
- No external data sharing  
- No cloud dependency  

### Optional

- Kubernetes deployment (optional)

---
