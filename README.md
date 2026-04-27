
# Transformer Thermal and Vibration Anomaly Detection — Technical Manual & Service Specification v0.1

**Author:** Elektro Gorenjska d.d., Slovenian DSO
**Version:** 1.0
**Last updated:** 15 Dec 2025

---

# 1. Business Context & Definitions

This service focuses on the development and operation of a transformer vibration and thermography anomaly detection system for a pilot distribution transformer monitoring location. The goal is to provide a continuous, data-driven view of transformer health by combining mechanical vibration signals and thermal imaging into a unified analytical framework.

The system integrates accelerometer-based vibration data from an MPU6050 sensor with infrared thermal images captured using an MLX90640 sensor. The sensing setup has been developed in-house and retrofitted onto an operational distribution transformer for pilot monitoring purposes. This allows real-world condition monitoring without interrupting normal transformer operation.

Vibration data is sampled at 1 kHz and processed into one-minute FFT snapshots, where the 200 most significant frequency components are retained to represent the spectral behaviour of the transformer. In parallel, infrared thermal images are captured every five minutes with a resolution of 32 × 24 pixels. These images capture the spatial temperature distribution across the transformer surface.

All data is timestamped and synchronized using the company NTP server and stored in an on-prem SQL database. At the time of writing, more than one year of continuous measurements is already available for the monitored transformer unit.

In addition to sensor data, the system supports time alignment with transformer electrical measurements such as current, voltage, active and reactive power, flicker, and THD. This combined dataset enables a more complete interpretation of both electrical and physical behaviour.

Elektro Gorenjska will use the outputs of this service to detect abnormal operating conditions, identify possible mechanical or thermal degradation, support preventive maintenance decisions, and provide an additional health-related signal for integration into the internal Asset Health Index (AHI) platform.

---

## Key Terms

• **Pilot Distribution Transformer:** The specific transformer on which the monitoring system is deployed for experimental and operational evaluation.
• **Vibration Data:** Accelerometer-based measurements used as an indirect indicator of mechanical condition, often linked to core, winding, and structural dynamics.
• **FFT Snapshot:** A frequency-domain representation of a one-minute vibration signal, computed using Fast Fourier Transform and reduced to the 200 most dominant frequency components.
• **Frequency Component:** A single spectral element defined by frequency and magnitude, representing energy distribution within the vibration signal.
• **Infrared Thermal Image:** A 32 × 24 pixel thermal matrix capturing surface temperature distribution using the MLX90640 sensor.
• **Thermal Pattern:** Spatial temperature distribution within a thermal image; deviations may indicate overheating or developing faults.
• **Load and Power-Quality Data:** Electrical measurements (current, voltage, power, THD, flicker) aligned with sensor data for contextual analysis.
• **Time Alignment:** Synchronization of all data sources using a unified timestamp reference to enable combined analysis.
• **Baseline Behaviour:** The expected operational behaviour of the transformer derived from historical data under normal conditions.
• **Anomaly Detection:** Identification of deviations from baseline behaviour that may indicate abnormal or degraded operation.
• **Preventive Maintenance Alarm:** A system-generated signal indicating that transformer condition requires inspection before failure occurs.
• **AHI Platform:** Internal Asset Health Index system used for transformer condition monitoring and maintenance planning.
• **Health-Related Signal:** Any derived metric or alarm intended as input to the AHI platform.

---

## 1.1 Elektro Gorenjska (Transformer Owner) Context

This service is intended for Elektro Gorenjska as the operator of the monitored distribution transformer. Its primary purpose is to enhance visibility into transformer condition and enable earlier detection of abnormal behaviour patterns that may indicate degradation or failure.

Although the current dataset originates from a single transformer, the intention is to develop a generalized solution that can be applied to multiple units in the future. The system must therefore be designed with standardized inputs and a consistent analytical structure, allowing it to scale across additional transformers equipped with the same sensing infrastructure.

In practice, the service compares current operational behaviour against historical baseline patterns under similar electrical loading and environmental conditions. The system must therefore be flexible enough to learn unit-specific behaviour while preserving a consistent detection logic across all deployments.

The provider is responsible for delivering a production-ready analytical solution, including the methodology used to derive anomaly indicators and deviation metrics. Elektro Gorenjska provides the data, infrastructure, and integration pathway into the AHI platform.

The chosen analytical approach may include robust statistics, clustering, or machine learning methods. However, the final solution must remain explainable, reproducible, and suitable for long-term operational use.

---

# 2. Problem Statement

The objective is to develop a production-ready anomaly detection service for a monitored distribution transformer using synchronized vibration, thermal imaging, and electrical operating data.

The solution shall be implemented as a Python-based analytical system, suitable for containerized deployment and scheduled daily execution. It must include clear documentation and a structure that allows integration into existing operational workflows and the AHI platform.

The system must produce at least one daily alarm indicator and an associated anomaly score. In addition, it must provide supporting contextual information to allow operators to interpret why an alarm has been triggered.

The service must be built around standardized input data types, reflecting the current pilot setup. These include:

* vibration FFT snapshots
* electrical load and power-quality data (current, voltage, power, THD, flicker)
* infrared thermal images

While preprocessing and feature engineering approaches may be defined by the provider, the system must remain tightly aligned with the available data structure. Flexibility should not come at the cost of operational robustness.

A key requirement is generalization: the same solution should be applicable to additional transformers in the future without redesign. Each transformer should be able to develop its own baseline behaviour model while sharing the same analytical framework.

The service must follow a strictly causal design, meaning only data available at the time of computation may be used.

The provider may choose any suitable combination of robust statistics and machine learning methods (e.g., clustering, prediction-based anomaly scoring, deviation modelling), provided that the final solution remains reproducible and operationally stable.

Finally, the system must provide interpretable explanations for alarms. At minimum, it should indicate which variables or conditions contributed to the anomaly detection decision, such as:

* deviations in vibration frequency components
* abnormal thermal patterns
* inconsistencies between electrical load and physical response

Where full causal explanation is not possible, the system must still provide statistical interpretability (median, percentile ranges, min/max boundaries, deviation scores).

---

# 3. Data Description

All sensor data is timestamped and synchronized using a centralized NTP server. Data acquisition is performed via a Linux-based system that communicates with ESP32-based sensing devices through API calls. These devices execute vibration sampling or thermal image capture.

At present, no known clock drift or systematic timing issues have been observed. However, standard validation procedures must still be included in the analytical pipeline, including checks for missing values, corrupted records, and synchronization consistency.

---

## 3.1 Transformer Metadata

Table 1 summarizes the metadata available for the monitored transformer. This information defines the identity and physical location of the asset and may also be used for external enrichment, such as weather API integration.

### Table 1: Transformer metadata

| Variable         | Name         | Type  | Unit            | Description               |
| ---------------- | ------------ | ----- | --------------- | ------------------------- |
| Transformer ID   | id           | Str   | -               | Unique ID                 |
| Deployment Year  | deploy_year  | Int   | -               | Installation year         |
| Power Rating     | power_rating | Float | kVA             | Nominal transformer power |
| Location address | address      | Str   | -               | City, address, number     |
| Latitude         | latitude     | Float | Decimal degrees | Geographic latitude       |
| Longitude        | longitude    | Float | Decimal degrees | Geographic longitude      |

---

## 3.2 Data Dictionary of Vibration FFT Snapshots

Table 2 describes vibration data obtained from the MPU6050 sensor. Each record represents a one-minute FFT snapshot capturing dominant vibration frequencies.

### Table 2: FFT snapshots data

| Variable           | Variable name | Type   | Unit                | Description            | Example             |
| ------------------ | ------------- | ------ | ------------------- | ---------------------- | ------------------- |
| Transformer ID     | id            | String | -                   | Unique transformer ID  | TR-001              |
| Timestamp          | timestamp     | Int    | YYYY-MM-DD HH:MM:SS | Time of acquisition    | 2026-03-30 08:00:00 |
| Sampling frequency | fs            | Int    | Hz                  | Sampling rate          | 1000                |
| FFT window length  | fft_window    | Int    | s                   | Signal window duration | 60                  |
| Frequency bin      | fft_bin       | Float  | Hz                  | Frequency component    | 50.0                |
| Magnitude          | fft_magnitude | Float  | -                   | Signal magnitude       | 0.018               |

Each one-minute snapshot contains the 200 strongest frequency components, preserving dominant spectral characteristics of the transformer vibration.

---

## 3.3 Infrared Thermal Images

Table 3 describes thermal imaging data captured using the MLX90640 sensor.

### Table 3: Infrared thermal images data

| Variable       | Variable name  | Type         | Unit                | Description           | Example             |
| -------------- | -------------- | ------------ | ------------------- | --------------------- | ------------------- |
| Transformer ID | transformer_id | String       | -                   | Unique transformer ID | TR-001              |
| Timestamp      | timestamp      | Timestamp    | YYYY-MM-DD HH:MM:SS | Capture time          | 2026-03-30 08:00:00 |
| Image width    | image_width    | Integer      | pixels              | Image width           | 32                  |
| Image height   | image_height   | Integer      | pixels              | Image height          | 24                  |
| Thermal matrix | thermal_matrix | Array/Matrix | °C                  | Full temperature grid | [[21.3, 21.5, ...]] |

At this stage, the full thermal matrix is stored without precomputed features. Derived metrics (e.g., hotspots, gradients, regions of interest) may be calculated during processing.

---

## 3.4 Transformer Operating and Power-Quality Data

Table 4 summarizes electrical measurements aligned with sensor data.

### Table 4: Transformer operating and power quality data

| Variable                     | Variable name                      | Type  | Unit        | Description                 | Example |
| ---------------------------- | ---------------------------------- | ----- | ----------- | --------------------------- | ------- |
| Energy                       | energy                             | Float | kWh         | Energy measurement          | 300 s   |
| THD current phase 1/2/3      | THD_I1, THD_I2, THD_I3             | Float | %           | Current harmonic distortion | 600 s   |
| THD voltage phase 1/2/3      | THD_U1, THD_U2, THD_U3             | Float | %           | Voltage harmonic distortion | 600 s   |
| Active power phase 1/2/3     | P1, P2, P3                         | Float | kW          | Active power                | 300 s   |
| Reactive power phase 1/2/3   | Q1, Q2, Q3                         | Float | kvar        | Reactive power              | 300 s   |
| Apparent power phase 1/2/3   | S1, S2, S3                         | Float | kVA         | Apparent power              | 300 s   |
| Active/reactive/apparent max | P_max, Q_max, S_max                | Float | kW/kvar/kVA | Max values                  | 60 s    |
| Total power                  | P_total, Q_total, S_total          | Float | kW/kvar/kVA | Total power                 | 60 s    |
| Voltage phase 1/2/3          | U1, U2, U3                         | Float | V           | Phase voltages              | 300 s   |
| Voltage max/min              | U1_max/min, U2_max/min, U3_max/min | Float | V           | Extremes                    | 60 s    |
| Power factor                 | ePF1, ePF2, ePF3                   | Float | -           | Phase PF                    | 300 s   |
| Total PF                     | PF                                 | Float | -           | System PF                   | 300 s   |
| Current                      | I                                  | Float | A           | Total current               | 300 s   |
| Current phase                | I1, I2, I3                         | Float | A           | Phase currents              | 300 s   |
| Current max/min              | I1_max/min, I2_max/min, I3_max/min | Float | A           | Extremes                    | 60 s    |
| Neutral current              | Inc                                | Float | A           | Neutral current             | 300 s   |
| Neutral current max          | Inc_max                            | Float | A           | Max neutral current         | 60 s    |
| Voltage unbalance            | Uu                                 | Float | %           | Phase imbalance             | 300 s   |
| Flicker                      | flicker                            | Float | -           | Flicker index               | -       |

---

# 4. Analytics, Scope & Update Frequency

The analytics are executed using rolling historical windows that include only data available at the time of computation. The goal is to continuously model the transformer’s normal behaviour and detect deviations that may indicate early-stage faults or abnormal operating conditions.

The system analyses vibration FFT snapshots, thermal images, and electrical measurements jointly. The combined interpretation allows the detection of anomalies that would not be visible from a single data source.

The service runs on a daily update cycle aligned with existing AHI workflows. Each execution produces updated anomaly indicators and scores.

Outputs include:

* Alarm flag (binary indicator of abnormal state)
* Anomaly score (severity estimate)
* Baseline reference behaviour
* Deviation metrics per sensor type
* Contributing variables or features
* Statistical context (median, percentiles, min/max ranges)

Although the current deployment is limited to a single transformer, the architecture is explicitly designed to scale to multiple units using the same input structure.

---

# 5. Evaluation Protocols & Metrics

The evaluation framework ensures that the system produces reliable, interpretable, and operationally useful anomaly detection outputs.

Evaluation focuses on:

* detection quality
* stability over time
* interpretability of alarms
* reproducibility of results

---

## 5.1 Data Usage & Analytical Protocol

* Analytics are computed using rolling historical windows ending at execution time
* No future data may be used (strict causality)
* Synthetic anomalies may be used due to limited real fault labels
* All results must be reproducible using fixed model versions and configurations

---

## 5.2 Data Gaps and Exceptions

* Missing or corrupted data segments are excluded from evaluation
* All anomalies in synchronization or quality must be logged
* System assumes generally stable data but still performs consistency checks

---

## 5.3 Service Evaluation Metrics & KPIs

| KPI                           | Description                          | Target    | Method                       |
| ----------------------------- | ------------------------------------ | --------- | ---------------------------- |
| Anomaly Detection Performance | Accuracy on synthetic/reviewed cases | >95%      | Precision/Recall/F1, ROC-AUC |
| False Alarm Rate              | False positives in normal conditions | ≤ 5/year  | Baseline replay              |
| Detection Delay               | Time from anomaly onset to detection | Minimized | Synthetic tests              |
| Anomaly Score Stability       | Stability in normal conditions       | <10% CV   | Rolling analysis             |
| Explainability Coverage       | Alarms with explanations             | 100%      | Output audit                 |
| Statistical Context Coverage  | Availability of baseline context     | 100%      | Log review                   |
| Data Coverage                 | Successful daily runs                | 100%      | Execution logs               |
| Reproducibility               | Identical outputs per input          | 100%      | Controlled reruns            |

---

# 6. Deliverables & Submissions

The provider shall deliver a complete analytical solution together with documentation and deployment materials suitable for operational use in an on-prem environment.

The scope is strictly analytical: orchestration and broader integration may be handled separately by Elektro Gorenjska.

---

## 6.1 Deliverable Reports

1. **Pre-Service Deliverable — Design & Setup Report**
   Defines the full analytical approach, data structure, preprocessing assumptions, anomaly logic, and deployment plan. Includes example configuration and integration concept.

2. **Intermediate Deliverable — Performance Report**
   Summarizes system behaviour during pilot operation, including early anomaly detection results, KPI tracking, and observed issues.

3. **Final Deliverable — Evaluation Report**
   Provides final performance assessment, lessons learned, and recommendations for scaling to additional transformers.

---

## 6.2 Technical Specifications & Submissions

* Python-based analytical solution delivered in Docker format for on-prem Linux execution
* Full technical documentation including architecture, inputs, outputs, and execution flow
* Trained ML models (if applicable) with versioning
* Example input/output dataset demonstrating execution
* Deployment manual with installation and runtime instructions
* Handover session covering deployment and operations
* Training session for operators and engineers
* Strict NDA compliance; no external data sharing
* Fully on-prem execution required (no external APIs except optional approved services)
* Kubernetes packaging optional only if justified


---
