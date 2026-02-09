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


## Notes

* Raw datasets are not uploaded due to size limits.
* Scripts are provided to reproduce preprocessing locally.
* This project follows standard space weather forecasting methodology.


