# Collecting real data and performing some basic ML mode (Day 5.2)

## Task 1 – Raw IMU Data Collection & Analysis

## 1) Process of collecting raw IMU data
**Code used:** `IMU_SD_serialtrigger.ino`

### Procedure
1. Upload `IMU_SD_serialtrigger.ino` to M5Stack Core2 using Arduino IDE.
2. Connect device through serial monitor.
3. Start/stop recording using serial trigger.
4. IMU raw measurements are recorded and saved to SD card as a `.csv` file (ex: `raw_imu_log.csv`).

### Raw signals collected
- Accelerometer: `ax, ay, az` (g)
- Gyroscope: `gx, gy, gz` (deg/s)
- Timestamp (ms)

---

## 2) Why perform smoothing/filtering of raw IMU data?
Raw IMU data is not directly suitable for ML because it contains:
- high-frequency sensor noise
- vibration artifacts
- gravity mixed with motion acceleration (for accelerometer)
- short spikes/outliers

Therefore smoothing/filtering is required to:
- stabilize signals
- reduce noise
- extract meaningful motion-related patterns
- improve reliability of ML features

---

## 3) What is performed in feature extraction?
**Program used:** `m5imu_data_analysis.py`

This program loads raw IMU data and produces both:
1) visual analysis plots  
2) time-window based features (motion descriptors)

### (a) Examples of feature extraction
- **SMV (Signal Magnitude Vector)**  
  SMV = √(ax² + ay² + az²)  
  → measures overall acceleration intensity
- **SMV Mean / Max (Window 250ms)**
  - Mean SMV → overall activity level
  - Max SMV → detects sudden impact/motion
- **SMV Standard Deviation**
  - distinguishes stable vs dynamic motion
- **Zero Crossing Rate (ZCR)**
  - counts how often signal crosses 0 within window
  - captures oscillation / vibration behavior
- **Gravity Removal (High-pass filter)**
  - isolates dynamic acceleration by removing gravity component

---

## 4) Comment on the analysis performed (Use and Need)
The analysis is useful because:
- Raw IMU plots help verify the data quality (spikes, drift, noise)
- Gravity removal improves interpretation of acceleration for motion tasks
- Window-based features convert raw time series into ML-friendly input
- Features such as SMV max/STD help classify motion intensity and detect events

**Conclusion:** Feature extraction is essential to compress raw sensor time-series into meaningful numerical descriptors for learning algorithms.

---

## Task 2 – Smooth IMU Data Collection & Roll Estimation

## 1) Process of collecting smooth IMU data
**Code used:** `IMU_SD_ST_AGEQ.ino`  
**Action:** slow-moving roll motion (deliberately slow to reduce disturbance)

### Procedure
1. Upload `IMU_SD_ST_AGEQ.ino` to M5Stack Core2.
2. Perform slow roll movement while the device records.
3. Data is saved into SD card (ex: `imu_smooth.csv`).

---

## 2) What does the program do? (IMU_SD_ST_AGEQ.ino)
This program records IMU signals and performs processing for stable roll estimation:
- sensor sampling at fixed rate (ex: 100 Hz)
- smoothing (moving average / low-noise handling)
- IMU fusion preparation
- outputs a “smooth dataset” that is more suitable for roll regression modeling

---

## 3) Ground-truth evaluation: How is ground-truth evaluated?
Ground-truth roll angle is obtained from **device fusion output** (on-board sensor fusion algorithm).  
This output combines accelerometer + gyroscope information to compute orientation.  
Therefore in this experiment:

- **Ground Truth (GT):** roll from device fusion
- **Prediction:** roll estimated by regression algorithms using IMU features

---

## 4) What is R2 metric?
R2 (coefficient of determination) measures how well a regression model predicts the target:

- **R2 = 1.0:** perfect prediction
- **R2 = 0.0:** model explains none of the variance
- **R2 < 0:** worse than predicting mean value

In this task, R2 is used to compare the roll estimation accuracy against ground truth.

---

## 5) Regression algorithms used (Description)

### (1) Linear Regression
Assumes roll is a linear combination of accelerometer features (ax, ay, az).
- fast, interpretable, but limited for nonlinear mapping

### (2) Non-linear Regression (Polynomial Regression)
Expands features into polynomial terms to model nonlinear roll relationships.
- captures curvature/nonlinearity better than linear regression

### (3) Decision Tree Regression
Uses rule-based splitting (if-else thresholds) to predict roll.
- interpretable but can overfit and show limited smoothness

### (4) Random Forest Regression
Ensemble of many decision trees (averaging predictions).
- reduces overfitting and improves generalization

### (5) SVM Regression (RBF Kernel)
Uses kernel trick to model nonlinear relationship.
- strong performance on complex manifolds, requires hyperparameter tuning

### (6) MLP Neural Network Regression
Feedforward neural network that learns highly nonlinear mapping.
- flexible and powerful but needs training stability

---

## 6) Performance comparison (R2 scores)
The following R2 scores were obtained:

| Model | R2 Score |
|------|----------|
| Linear Regression | 0.9975 |
| Non-linear Regression (Poly) | 0.9998 |
| Decision Tree | 0.9888 |
| Random Forest | 0.9995 |
| SVM Regression | 0.9997 |
| MLP NN Regression | 0.9998 |
| Analytic / Fusion reference | 0.9998 |

---

## 7) Which algorithm is the best?
Best performance (highest R2) was achieved by:

- **Non-linear (Polynomial) Regression: R2 = 0.9998**
- **MLP Neural Network Regression: R2 = 0.9998**

Both methods match ground truth almost perfectly.

**Conclusion:** For smooth roll estimation under slow-motion conditions, Polynomial Regression and MLP NN give the best prediction accuracy.

---

# Final Summary
- Task 1 successfully collected raw IMU data and performed signal analysis + feature extraction.
- Task 2 successfully collected smooth IMU data and evaluated multiple regression models for roll estimation.
- R2 metric was used to compare models against ground truth (device fusion roll).
- Best models: **Polynomial Regression** and **MLP NN Regression**.
