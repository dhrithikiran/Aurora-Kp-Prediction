# Kp Index Prediction for Aurora Forecasting

Predicts the geomagnetic Kp index 3 hours ahead using solar wind data from NASA's OMNI dataset, then interprets the predicted Kp value to estimate aurora visibility.

---

## Problem Statement

Ground-based magnetometers measure the Kp index after a geomagnetic storm has already begun. This project predicts Kp in advance using upstream solar wind measurements at L1, giving a 3-hour warning window for aurora visibility forecasting.

**Target:** `Kp_3h_ahead` — continuous regression output (0–9 scale)

---

## Dataset

- **Source:** NASA OMNI dataset (Jan 2025 – Jan 2026)
- **Why OMNI:** Already time-shifted to Earth impact time, no manual propagation delay needed
- **Raw format:** Fixed-width `.txt` file, converted to CSV
- **Input features:** IMF Bz (GSM), Solar wind speed, Proton density, Proton temperature, Flow pressure, past Kp values
- **Target:** Kp × 10 stored in OMNI → divided by 10 to get real Kp

---

## Approach

### Preprocessing
- Loaded raw `.txt`, assigned column names, converted `Kp_x10 / 10` → `Kp`
- Filled missing values with column median

### EDA
- Time-series plots of Kp and all solar wind parameters
- Correlation heatmap, distribution histograms, boxplots, scatter plots vs Kp
- Rolling averages (24h, 72h) and lagged feature visualizations

### Feature Engineering
Starting from 10 raw columns → **66 features** after engineering:
- **Lag features:** Bz, speed, density, Kp at t−1, t−2, t−3, t−6
- **Rolling statistics:** Mean, std, min/max over 1h, 3h, 6h windows
- **Rate of change:** ΔBz, ΔV, ΔDensity (storm onset detection)
- **Physics-based:** Bz×V, Electric field proxy, Alfvén Mach proxy, dynamic pressure proxy
- **Southward Bz:** Binary flag, consecutive southward duration, southward magnitude
- **Time features:** Hour of day, day of year, month

### Train-Test Split
- Chronological split — **no random shuffling** (prevents data leakage)
- Train: first 80% | Test: last 20%
- Final dataset: **8,933 rows × 66 columns**

### Models

| Model | Why Used |
|---|---|
| Linear Regression | Baseline — interpretable, shows physical feature relationships |
| Random Forest (200 trees, depth 20) | Handles nonlinear storm dynamics, robust to noise |
| Neural Network (128→64→32, ReLU, Adam) | Captures complex temporal patterns |

All models evaluated on MAE, RMSE, and R².

---

## Results

Predictions and metrics saved per model:

```
models/
├── linear_regression/   → linear_regression_kp.pkl, metrics.csv, test_predictions.csv
├── random_forest/       → random_forest_kp.pkl, metrics.csv, test_predictions.csv
└── neural_network/      → mlp_model.pkl, scaler.pkl, metrics.csv, test_predictions.csv
```

---

## Aurora Interpretation

Predicted Kp values are mapped to real-world aurora visibility using NOAA's G-scale:

| Kp | Storm Level | Aurora Visibility |
|---|---|---|
| 0–3 | G0 Quiet | Polar regions only |
| 4 | G0 Unsettled | Northern Scotland, Scandinavia |
| 5 | G1 Minor | Scotland, northern England |
| 6 | G2 Moderate | Southern England, central Europe |
| 7+ | G3–G5 | Northern USA, southern Europe, further south |

---

## Limitations

- Only 1 year of data (~8,933 samples) — limited for neural network training
- Median fill for missing values may misrepresent storm-time gaps
- OMNI sentinel value replacement (`9999`, `99999`) runs after CSV save — ordering issue in pipeline
- 299 NaNs remain in derived features after cleanup, handled by imputer in model files
- Models predict continuous Kp but storm peaks (rare, high Kp events) are underrepresented in training data due to class imbalance

---

## Future Improvements

- Use multiple years of OMNI data to improve model generalisation
- Fix sentinel value replacement ordering in preprocessing pipeline
- Add cyclical encoding (sin/cos) for hour and day of year features
- Experiment with LSTM or temporal models better suited for time-series
- Incorporate real-time DSCOVR/ACE data feed for live aurora forecasting
- Add storm classification layer on top of regression output (Kp ≥ 5 = storm alert)

---

## How to Run

```bash
# 1. Preprocess raw OMNI txt file
python preprocessing.py

# 2. Exploratory data analysis
jupyter notebook eda.ipynb

# 3. Feature engineering
python featureengineering.py

# 4. Create target and split data
python train_split.py

# 5. Train and evaluate models
python 1_Linear_Regression.py
python 2_Random_Forest.py
python 3_Neural_Network.py

# 6. Aurora interpretation
python aurora_interpretation.py
```

---

## Dependencies

```
pandas  numpy  matplotlib  seaborn  scikit-learn  joblib
```