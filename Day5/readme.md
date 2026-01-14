# Collecting real data and performing some basic ML modeling (Day 5.2)

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


## 3) What is performed in feature extraction?
**Program used:** `m5imu_data_analysis.py`

This program loads raw IMU data and produces both:
1) visual analysis plots  
2) time-window based features (motion descriptors)

### (Preprocessing) Timestamp cleaning
While loading the raw IMU log, some values in the `Timestamp` column were stored as strings or contained invalid entries, which caused errors during plotting and time-based feature extraction.
To fix this, timestamps were converted into numeric format and invalid rows were removed.
This preprocessing ensures a valid time axis for plotting and prevents errors in window-based feature extraction.

```python
df['Timestamp'] = pd.to_numeric(df['Timestamp'], errors='coerce')
df = df.dropna(subset=['Timestamp'])
```

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


## 4) Comment on the analysis performed (Use and Need)
The analysis is useful because:
- Raw IMU plots help verify the data quality (spikes, drift, noise)
- Gravity removal improves interpretation of acceleration for motion tasks
- Window-based features convert raw time series into ML-friendly input
- Features such as SMV max/STD help classify motion intensity and detect events

**Conclusion:** Feature extraction is essential to compress raw sensor time-series into meaningful numerical descriptors for learning algorithms.


---


## Task 2 – Smooth IMU Data Collection & Roll Estimation

### 1) Process of collecting smooth IMU data
**Code used:** `IMU_SD_ST_AGEQ.ino`

**Action**
- Perform a slow-moving roll motion (slow motion reduces disturbance and improves stability)

**Procedure**
1. Upload `IMU_SD_ST_AGEQ.ino` to M5Stack Core2.
2. Perform slow roll motion while the device is recording.
3. IMU signals are saved as smooth dataset CSV (e.g., `imu_smooth.csv`).

### 2) What does the program do? (IMU_SD_ST_AGEQ.ino)
This code records IMU signals at a fixed rate and outputs a stable dataset for roll estimation:
- IMU sampling at constant rate (e.g., 100 Hz)
- smoothing / noise suppression
- preparation for roll estimation using accel + gyro fusion signals
- produces smooth dataset suitable for regression modeling

### 3) Ground-truth evaluation (How is ground-truth evaluated?)
In this experiment, the **ground-truth (pseudo ground truth)** roll angle was obtained from the **on-board device fusion output (Madgwick / AHRS)**.
This fusion combines accelerometer + gyroscope to compute device orientation.

Therefore:
- **Ground Truth (pseudo GT):** roll from device fusion
- **Prediction:** roll estimated by regression models using IMU data/features

### 4) What is R2 metric?
R2 (coefficient of determination) measures regression prediction quality.

- **R2 = 1.0:** perfect prediction
- **R2 = 0.0:** no explanatory power
- **R2 < 0:** worse than predicting the mean

In this task, R2 is used to compare roll estimation accuracy against the ground truth (pseudo ground truth).

### 5) Regression algorithms used (description)

**(1) Linear Regression**
- Assumes roll is a linear combination of features (ax, ay, az)
- Fast and interpretable, but limited for nonlinear mapping

**(2) Non-linear Regression (Polynomial Regression)**
- Expands features using polynomial terms
- Models nonlinear roll relationship better than linear regression

**(3) Decision Tree Regression**
- Uses rule-based splitting (if-else thresholds)
- Interpretable but may overfit and produce less smooth outputs

**(4) Random Forest Regression**
- Ensemble of decision trees (averaging)
- Reduces overfitting and improves generalization

**(5) SVM Regression (RBF Kernel)**
- Learns nonlinear mapping using kernel trick
- Strong performance for complex relations but needs tuning

**(6) MLP Neural Network Regression**
- Feedforward neural network for nonlinear approximation
- Flexible and strong but depends on training stability

### 6) Performance comparison (R2 scores)

| Model | R2 Score |
|------|----------|
| Linear Regression | 0.9975 |
| Non-linear Regression (Poly) | 0.9998 |
| Decision Tree | 0.9888 |
| Random Forest | 0.9995 |
| SVM Regression | 0.9997 |
| MLP NN Regression | 0.9998 |

### 7) Which algorithm is the best?
The best performance (highest R2) was achieved by:
- **Non-linear (Polynomial) Regression: R2 = 0.9998**
- **MLP Neural Network Regression: R2 = 0.9998**

Both methods match the ground truth (pseudo ground truth) almost perfectly.

**Conclusion:** For smooth roll estimation under slow-motion conditions, Polynomial Regression and MLP NN produce the best prediction accuracy.


## Final Summary
- Task 1: Successfully collected raw IMU data and performed analysis + feature extraction.
- Task 2: Collected smooth IMU data and compared regression models for roll estimation.
- Ground truth (pseudo ground truth) was defined using the device fusion roll output.
- Best algorithms: **Polynomial Regression** and **MLP NN Regression** (R2 = 0.9998).
