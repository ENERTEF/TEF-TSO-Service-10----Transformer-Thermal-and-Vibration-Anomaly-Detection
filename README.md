# Transformer Thermal and Vibration Anomaly Detection

**Service:** Transformer Thermal and Vibration Anomaly Detection
**Document Type:** Technical Manual & Service Specification v 0.1
**Author:** Elektro Gorenjska d.d., Slovenian DSO
**Version:** 1.0
**Last updated:** 15 Dec 2025

---

# 1 Business Context & Definitions

This service focuses on the development and operation of a transformer vibration and thermography anomaly detection service for a pilot distribution transformer monitoring location.

The service combines:

* accelerometer-based vibration data from an MPU6050 sensor
* infrared thermal images captured using an MLX90640 sensor

The sensing setup:

* developed in-house
* retrofitted onto an existing in-service distribution transformer

---

## Data acquisition details

* Vibration sampling:

  * 1 kHz sampling rate
  * stored as one-minute FFT snapshots
  * each snapshot contains the 200 highest magnitude components

* Thermal imaging:

  * captured every 5 minutes
  * resolution: 32 × 24 pixels

* Data properties:

  * timestamped
  * synchronized with company NTP server
  * stored in on-prem SQL server
  * > 1 year of continuous measurements available

---

## Additional data alignment

Sensor data can be aligned with:

* current
* voltage
* active power
* reactive power
* flicker
* THD

---

## Service purpose

Outputs are used to:

* detect abnormal operating behaviour
* identify mechanical or thermal issues
* support preventive maintenance
* provide input to AHI platform

---

## Key Terms

| Term                           | Definition                                                                          |
| ------------------------------ | ----------------------------------------------------------------------------------- |
| Pilot Distribution Transformer | The transformer monitored in the pilot deployment                                   |
| Vibration Data                 | Accelerometer-based mechanical condition indicator                                  |
| FFT Snapshot                   | One-minute frequency-domain representation storing 200 highest magnitude components |
| Frequency Component            | Individual frequency + magnitude from FFT spectrum                                  |
| Infrared Thermal Image         | 32×24 pixel thermographic image                                                     |
| Thermal Pattern                | Spatial temperature distribution in IR image                                        |
| Load and Power-Quality Data    | Measurements like current, voltage, power, THD, flicker                             |
| Time Alignment                 | Synchronization of all data sources via timestamp                                   |
| Baseline Behaviour             | Learned normal operating behaviour                                                  |
| Anomaly Detection              | Identification of deviations from baseline                                          |
| Preventive Maintenance Alarm   | Signal indicating abnormal behaviour                                                |
| AHI Platform                   | Internal Asset Health Index platform                                                |
| Health-Related Signal          | Derived indicator for AHI integration                                               |

---

## 1.1 Elektro Gorenjska Context

This service is intended to:

* improve visibility into transformer behaviour
* detect deviations early
* support earlier maintenance

---

### Generalization goal

Although current data is from one unit:

* solution must be generalized
* standardized inputs required
* scalable to multiple transformers

---

### Operational concept

* compare current vs expected behaviour under similar load/PQ
* adapt per transformer profile
* maintain consistent structure across deployments

---

### Responsibilities

**Provider:**

* delivers analytical solution
* defines methodology
* provides anomaly indicators

**Elektro Gorenjska:**

* provides data
* manages AHI integration

---

### Method flexibility

Allowed methods include:

* robust statistics
* clustering
* prediction models
* ML techniques

Constraints:

* explainable
* reproducible
* scalable

---

# 2 Problem Statement

The goal is to build a production-ready anomaly detection service using:

* vibration FFT data
* thermal images
* operating data

---

## Technical requirements

* Python-based solution
* containerized deployment
* daily execution
* integration-ready

---

## Outputs

* daily alarm indicator
* anomaly score
* explanation of alarm

---

## Inputs

* FFT snapshots
* PQ/load data (current, voltage, THD, flicker, etc.)
* thermal images

---

## Design constraints

* standardized inputs
* robust structure
* no unnecessary flexibility

---

## Analytical protocol

* strictly causal (no future data)
* reproducible
* explainable

---

## Explanation requirement

Output must include:

* contributing variables
* deviation from baseline
* statistical context:

  * median
  * min/max
  * percentiles

---

# 3 Data Description

* all data timestamped via NTP
* ESP32-based sensing
* triggered via Linux system API
* no known clock drift

---

## Data quality

* no major issues identified
* validation still required:

  * missing values
  * invalid records
  * synchronization

---

## 3.1 Transformer Metadata

### Table 1: Transformer metadata

| Variable         | Name         | Type  | Unit            | Description                |
| ---------------- | ------------ | ----- | --------------- | -------------------------- |
| Transformer ID   | id           | Str   | -               | Unique ID                  |
| Deployment Year  | deploy_year  | Int   | -               | Installation year          |
| Power Rating     | power_rating | Float | kVA             | Nominal transformer power  |
| Location address | address      | Str   | -               | City, address, address no. |
| Latitude         | latitude     | Float | Decimal degrees | Latitude                   |
| Longitude        | longitude    | Float | Decimal degrees | Longitude                  |

---

## 3.2 Vibration FFT Snapshots

### Table 2: FFT snapshots data

| Variable           | Variable name | Type   | Unit                | Description           | Example             |
| ------------------ | ------------- | ------ | ------------------- | --------------------- | ------------------- |
| Transformer ID     | id            | String | -                   | Unique transformer ID | TR-001              |
| Timestamp          | timestamp     | Int    | YYYY-MM-DD HH:MM:SS | Acquisition time      | 2026-03-30 08:00:00 |
| Sampling frequency | fs            | Int    | Hz                  | Raw signal frequency  | 1000                |
| FFT window length  | fft_window    | Int    | s                   | Signal duration       | 60                  |
| Frequency bin      | fft_bin       | Float  | Hz                  | FFT frequency         | 50.0                |
| Magnitude          | fft_magnitude | Float  | -                   | FFT magnitude         | 0.018               |

Each snapshot stores:

* 200 highest magnitude frequency components

---

## 3.3 Infrared Thermal Images

### Table 3: Thermal images data

| Variable       | Variable name  | Type         | Unit                | Description              | Example             |
| -------------- | -------------- | ------------ | ------------------- | ------------------------ | ------------------- |
| Transformer ID | transformer_id | String       | -                   | Unique ID                | TR-001              |
| Timestamp      | timestamp      | Timestamp    | YYYY-MM-DD HH:MM:SS | Capture time             | 2026-03-30 08:00:00 |
| Image width    | image_width    | Integer      | pixels              | Width                    | 32                  |
| Image height   | image_height   | Integer      | pixels              | Height                   | 24                  |
| Thermal matrix | thermal_matrix | Array/Matrix | °C                  | 32×24 temperature matrix | [[21.3, 21.5, ...]] |

Derived features (optional later):

* max temperature
* gradients
* regional values

---

## 3.4 Transformer Operating & PQ Data

### Table 4: Operating and power quality data

| Variable          | Variable name             | Type  | Unit        | Description  | Example         |
| ----------------- | ------------------------- | ----- | ----------- | ------------ | --------------- |
| Energy            | energy                    | Float | kWh         | 300 s        | Energy          |
| THD current       | THD_I1, THD_I2, THD_I3    | Float | %           | 600 s        | Current THD     |
| THD voltage       | THD_U1, THD_U2, THD_U3    | Float | %           | 600 s        | Voltage THD     |
| Active power      | P1, P2, P3                | Float | kW          | 300 s        | Active power    |
| Reactive power    | Q1, Q2, Q3                | Float | kvar        | 300 s        | Reactive power  |
| Apparent power    | S1, S2, S3                | Float | kVA         | 300 s        | Apparent power  |
| Power max         | P_max, Q_max, S_max       | Float | kW/kvar/kVA | 60 s         | Max values      |
| Total power       | P_total, Q_total, S_total | Float | kW/kvar/kVA | 60 s         | Total values    |
| Voltage           | U1, U2, U3                | Float | V           | 300 s        | Voltage         |
| Voltage extrema   | U1_max/min, etc.          | Float | V           | 60 s         | Min/max         |
| Power factor      | ePF1, ePF2, ePF3          | Float | -           | 300 s        | Phase PF        |
| Total PF          | PF                        | Float | -           | 300 s        | Total PF        |
| Current total     | I                         | Float | A           | 300 s        | Total current   |
| Current phase     | I1, I2, I3                | Float | A           | 300 s        | Phase current   |
| Current extrema   | I1_max/min, etc.          | Float | A           | 60 s         | Min/max         |
| Neutral current   | Inc                       | Float | A           | 300 s        | Neutral current |
| Neutral max       | Inc_max                   | Float | A           | 60 s         | Max neutral     |
| Voltage unbalance | Uu                        | Float | %           | 300 s        | Unbalance       |
| Flicker           | flicker                   | Float | -           | if available | Flicker         |

---

# 4 Analytics, Scope & Update Frequency

* Temporal scope: rolling historical window
* strictly causal

---

## Analytical scope

* learn baseline behaviour
* detect deviations

---

## Update frequency

* daily execution
* continuous ingestion

---

## Output format

* alarm flag
* anomaly score
* baseline reference
* deviation metrics
* contributing variables
* statistical context (median, min/max, percentiles)

---

## Scalability

* single unit now
* generalized design for multiple units

---

# 5 Evaluation Protocols & Metrics

## Goal

Ensure:

* reliability
* accuracy
* explainability

---

## 5.1 Data Usage

* rolling window
* causal only
* synthetic anomalies allowed
* reproducibility required

---

## 5.2 Data Gaps

* exclude corrupted data
* log synchronization issues

---

## 5.3 KPIs

| KPI                           | Description                | Target   | Method                         |
| ----------------------------- | -------------------------- | -------- | ------------------------------ |
| Anomaly Detection Performance | Detection quality          | >95%     | Precision, Recall, F1, ROC-AUC |
| False Alarm Rate              | False positives            | ≤5/year  | Replay                         |
| Detection Delay               | Time to alarm              | Minimize | Synthetic events               |
| Score Stability               | Variance in stable periods | <10% CV  | Rolling                        |
| Explainability Coverage       | Alarms explained           | 100%     | Audit                          |
| Statistical Context           | Baseline included          | 100%     | Audit                          |
| Data Coverage                 | Daily execution            | 100%     | Logs                           |
| Reproducibility               | Same outputs               | 100%     | Rerun                          |

---

# 6 Deliverables & Submissions

## Scope

* analytical solution only
* integration handled by Elektro Gorenjska

---

## 6.1 Deliverable Reports

### 1. Pre-Service Report

* analytical approach
* preprocessing
* anomaly logic
* deployment
* documentation
* handover plan

---

### 2. Intermediate Report

* performance
* data coverage
* alarm behaviour
* KPI tracking
* refinements

---

### 3. Final Report

* final results
* evaluation
* lessons learned
* scaling recommendations

---

## 6.2 Technical Specifications

### Source Code

* Python
* Docker-based
* Linux-compatible

---

### Documentation

* architecture
* input/output
* preprocessing
* execution steps
* interpretation

---

### Model Artefacts

* trained models
* versioning

---

### Example Setup

* input + output demo

---

### Deployment Manual

* installation
* dependencies
* execution

---

### Handover Session

* deployment
* execution
* outputs

---

### Training Session

* usage
* interpretation

---

### Security

* NDA required
* no external data sharing
* on-prem only
* no external APIs (except approved optional sources like weather)

---

### Optional Extension

* Kubernetes deployment (optional only if justified)

---
