# Kp Index Prediction Using Solar Wind Data

## Project Overview

This project predicts the **Kp geomagnetic index 30–90 minutes ahead** using solar wind measurements. The predicted Kp index is used to estimate **aurora visibility** using time-series machine learning.


## What is the Kp Index?

The **Kp index (0–9)** measures disturbances in Earth’s magnetic field:

* Low Kp → Quiet geomagnetic conditions
* High Kp → Geomagnetic storms and possible aurora

Kp is **not measured by satellites**. It is calculated from **ground-based magnetometer stations worldwide**.


## Solar Wind Data and L1 Satellites

Satellites at **Lagrange Point 1 (L1)** measure solar wind before it reaches Earth. The main parameters used in this project are:

* Solar wind speed (V)
* Proton density (D)
* IMF Bz (GSM coordinate system)
* Proton temperature (optional)
* Flow pressure  (optional) 


## Time Shifting Concept

Solar wind takes **30–90 minutes** to travel from L1 to Earth.
Therefore, solar wind data must be **time shifted** so timestamps represent when the solar wind reaches Earth.

### Analogy

Weather sensor at 5:00 AM → predicts temperature at 6:00 AM
Similarly: Solar wind at 04:30–05:30 → predicts Kp at 06:00


## Dataset Used

### Input Dataset

* **NASA OMNI dataset**
* Already time-shifted to Earth impact time
* Standard dataset used in space weather research

### Target Variable

* **Kp index from OMNI dataset** (ground station measurements synchronized with solar wind)
* OMNI provides **Kp × 10**, so values are divided by 10 during preprocessing


## Features Used

* Solar wind speed (V)
* Proton density (D)
* IMF Bz (GSM)
* Proton temperature (optional)
* Flow pressure (optional) 
* Past Kp values: Kp(t−1) to Kp(t−6) to capture storm persistence


## Prediction Task

The model predicts:

**Kp(t + 3 hours)**

Kp is reported every 3 hours (00–03, 03–06, 06–09, etc.).


## Time Window Selection

To predict Kp 3 hours ahead, the model uses **1 hour of solar wind data before the prediction time**.

Example:
To predict Kp at 06:00 → use solar wind data from 04:30 to 05:30.


## Data Range

* Start Date: 2025-01-01
* End Date: 2026-01-01


## Data Processing Steps

1. Download OMNI dataset
2. Convert TXT file to CSV using Python
3. Rename columns and clean data
4. Convert Kp × 10 to real Kp
5. Handled missing values / noisy data 
6. Check numeric data types and convert it


## Exploratory Data Analysis (EDA)

EDA was performed to understand dataset patterns and relationships:

* **Basic Info & Descriptive Stats**: `df.info()` and `df.describe()`  
* **Time Series Plots**: Visualize Kp and solar wind parameters over time  
* **Correlation Heatmap**: Examine relationships between Kp and input features  
* **Distribution Plots**: Histograms for Kp and solar wind parameters to check skewness  
* **Boxplots**: Detect extreme values and outliers  
* **Scatter Plots vs Kp**: Visualize the influence of each parameter on Kp  
* **Rolling Averages**: 24h and 72h Kp rolling means to observe trends  
* **Lagged Visualization**: Plot lagged solar wind features (1, 2, 3, 6 hours) to inspect temporal dependencies

## Feature Engineering

Feature engineering was performed to capture temporal dependencies, physical relationships, and storm dynamics in geomagnetic activity.

### 1. Handling Missing Values

The OMNI dataset may contain missing and flagged values. Missing values were handled using:
- Linear interpolation for short gaps  
- Forward-fill and backward-fill for small consecutive gaps  
This prevents data leakage while preserving time-series continuity.

### 2. Lag Features (Temporal Dependency)

Geomagnetic activity depends on recent solar wind conditions, so lagged features were created:

**Lagged Solar Wind Parameters**
- Bz(t−1h), Bz(t−2h), Bz(t−3h), Bz(t−6h)  
- Solar Wind Speed V(t−1h), V(t−2h), V(t−3h)  
- Proton Density D(t−1h), D(t−2h), D(t−3h)

**Lagged Kp Values**
- Kp(t−1) to Kp(t−6)  
These capture geomagnetic storm persistence and recovery effects.

### 3. Rolling Statistics (Trend & Variability)

Rolling window statistics were computed to capture recent trends:

**Rolling Windows**
- 1 hour, 3 hours, 6 hours  

**Computed Metrics**
- Rolling mean  
- Rolling standard deviation (3h, 6h)  
- Rolling minimum and maximum  

These features capture short-term variability and extreme solar wind conditions.

### 4. Rate of Change Features (Dynamics)

To detect sudden solar wind changes that trigger geomagnetic storms:

- ΔBz = Bz(t) − Bz(t−1)  
- ΔV = V(t) − V(t−1)  
- ΔD = D(t) − D(t−1)  

These represent rapid changes in IMF and solar wind parameters.

### 5. Physics-Based Coupling Features

Domain-specific interaction terms were engineered based on space weather physics:

- Bz × V (solar wind coupling strength)  
- Bz × Density  
- V² × Density (dynamic pressure proxy)  
- Electric Field Proxy = V × |Bz|  
- Southward Electric Field = V × |Bz| when Bz < 0  
- Alfvén Mach number proxy  

These features approximate solar wind–magnetosphere energy coupling.

### 6. Southward Bz Duration Features

Southward IMF (Bz < 0) is critical for geomagnetic storms. Features include:

- Binary Bz_negative indicator  
- Consecutive duration of Bz < 0 (storm buildup indicator)  
- Southward Bz strength magnitude  

These capture storm onset and persistence behavior.

### 7. Time-Based Features

To capture diurnal and seasonal variations:

- Hour of day  
- Day of year  
- Month  

(Optionally cyclic sine/cosine encoding can be used for periodic patterns.)

### Final Feature Dataset

After feature engineering:
- Lag features  
- Rolling statistics  
- Change rates  
- Physics-based interaction features  
- Time-based features  

## Dataset and Features

- **NASA OMNI dataset** (time-shifted to Earth impact)
- **Target:** Kp index (from OMNI), 3 hours ahead (`Kp_3h_ahead`)
- **Input Features:**
  - Solar wind speed (V)
  - Proton density (D)
  - IMF Bz (GSM)
  - Proton temperature (optional)
  - Flow pressure (optional)
  - Lagged Kp values (Kp(t−1) to Kp(t−6))
  - Lagged solar wind features, rolling statistics, rate-of-change features
  - Physics-based coupling features (e.g., Bz × V)
  - Time-based features (hour of day, day of year, month)


## Target Creation and Time-Based Split

- Created a `Timestamp` column from `Year`, `DOY`, `Hour`.
- Generated **future target**: `Kp_3h_ahead = Kp.shift(-3)`.
- Chronological **train-test split**:
  - Train: first 80% of data
  - Test: last 20% of data
- Saves:
  - `full_dataset_with_target.csv`
  - `train.csv`
  - `test.csv`


## Models and Pipelines

### 1. Linear Regression
- Baseline model, simple and interpretable.
- Handles numeric features only.
- Metrics: MAE, RMSE, R²
- Saved model: `models/linear_regression/linear_regression_kp.pkl`

### 2. Random Forest Regression
- Captures non-linear relationships and feature interactions.
- Parameters: `n_estimators=200`, `max_depth=20`.
- Metrics: MAE, RMSE, R²
- Saved model: `models/random_forest/random_forest_kp.pkl`

### 3. Neural Network (MLP)
- Captures complex non-linear and temporal dependencies.
- Architecture: `(128, 64, 32)` hidden layers, ReLU activation, Adam optimizer.
- Features scaled using StandardScaler.
- Metrics: MAE, RMSE, R²
- Saved model: `models/neural_network/mlp_model.pkl`  
- Saved scaler: `models/neural_network/scaler.pkl`


## Evaluation
- All models evaluated on **chronologically separated test data**.
- Visualizations: Actual vs Predicted Kp over time.
- Metrics summary available in CSV in respective `models/` folders.

## Notes

* Raw datasets are not uploaded due to size limits.
* Scripts are provided to reproduce preprocessing locally.
* This project follows standard space weather forecasting methodology.