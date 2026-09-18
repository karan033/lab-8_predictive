# Agricultural Predictive Analytics: Crop Yield Prediction

![Python](https://img.shields.io/badge/Python-3.13-blue.svg)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.6.1-orange.svg)
![pandas](https://img.shields.io/badge/pandas-2.2.3-green.svg)
![Status](https://img.shields.io/badge/Status-Completed-success.svg)

> **Advanced Predictive Analytics (Lab 08)**
> An end-to-end development of a reproducible, leakage-safe yield regression pipeline to predict rice crop yield using guided district-level analytics, emphasizing rigorous temporal splitting and responsible agricultural analytics.

---

## Table of Contents

1. [Overview](#overview)
2. [Dataset](#dataset)
3. [Project Workflow](#project-workflow)
4. [Models Evaluated](#models-evaluated)
5. [Key Results](#key-results)
6. [How to Run](#how-to-run)

---

## Overview

This repository documents the implementation and audit of a reproducible yield regression pipeline designed to predict crop yield across Indian districts. 

The prediction contract focuses on forecasting one rice-producing district-season-harvest-year observation, with the target variable being crop yield measured in tonnes per hectare (t/ha). The predictors are strictly whitelisted to pre-harvest information: state, district, season, and harvest year. This establishes a prospective-design backtest utilizing a historical data source, explicitly acknowledging the limitations of aggregate administrative records.

---

## Dataset

The dataset leveraged is the ICRISAT panel dataset. A robust data ingestion and unit harmonization pipeline was implemented to derive the target yield.

### Dataset Profile
* **Source File:** `main merge (droped _merge==2) (560 dist 1990-2015).xls`
* **Records:** 11,969 canonical records (filtered from 12,803 raw rows by removing missing values, non-positive yields, and invalid years).
* **Date Range:** 1990 to 2015.
* **Target:** `yield_t_ha` (tonnes per hectare).
* **Locations:** 536 Unique Districts across 20 Unique States.

---

## Project Workflow

The project follows a rigorous machine learning pipeline designed to prevent data leakage and ensure proper temporal evaluation.

1. **Data Ingestion & Unit Harmonization:** Derived the target yield (t/ha) from production and area pairs, filtering out invalid records to generate a secure canonical dataset.
2. **Temporal Splitting:** Partitioned the dataset strictly chronologically to simulate a realistic forecasting scenario: Train (1990–2011), Validation (2012–2013), and locked Test (2014–2015) sets.
3. **Exploratory Data Analysis (EDA):** Conducted EDA (yield distributions, year coverage) exclusively on the training split to prevent lookahead bias.
4. **Preprocessing Pipelines:** Transformed categorical features using `OneHotEncoder` (ignoring unknowns) and scaled continuous harvest years using a median `SimpleImputer` and `StandardScaler`. Preprocessing was fitted strictly on the training split.
5. **Candidate Benchmarking & Rolling Origins:** Evaluated simple and non-linear candidate models against the validation set and measured stability across development-year rolling origins.
6. **Selection:** Locked the champion model based strictly on generating the minimum Validation Mean Absolute Error (MAE).
7. **Single-Pass Locked-Test Evaluation:** Scored the selected champion model and the median baseline exactly once on the unseen 2014-2015 test block.
8. **Error Diagnostics:** Analyzed Actual vs. Predicted plots and residual distributions to assess heteroscedasticity and bounds of reliability.

---

## Models Evaluated

The following regression pipelines were configured and evaluated:

* **`Median Baseline`:** A naive reference predicting the training median for every observation.
* **`Ridge Trend`:** A regularized linear model utilizing an alpha of 1.0 and an 'lsqr' solver.
* **`Decision Tree Regressor`:** A tree-based model restricted to a max depth of 6 and minimum leaf samples of 10.
* **`Random Forest Regressor`:** An ensemble of 60 trees, bounded by a max depth of 12 and minimum leaf size of 5.

---

## Key Results

The **Ridge Trend** model was firmly selected as the champion model after achieving the lowest MAE during validation benchmarking.

### Validation Benchmarking

| Model | Validation MAE (t/ha) | Validation RMSE | R-Squared (R2) |
| :--- | :---: | :---: | :---: |
| **Ridge Trend** | 0.449 | 0.621 | 0.596 |
| **Random Forest** | 0.522 | 0.691 | 0.500 |
| **Decision Tree** | 0.706 | 0.882 | 0.186 |
| **Median Baseline** | 0.889 | 1.155 | -0.395 |

### Final Locked Test Performance

| Model | Test MAE (t/ha) | Test RMSE | R-Squared (R2) |
| :--- | :---: | :---: | :---: |
| **Ridge Trend** | 0.462 | 0.623 | 0.635 |
| **Median Baseline** | 0.915 | 1.191 | -0.332 |

### Major Findings

* The `Ridge Trend` formulation substantially minimized predictive errors in out-of-sample data, explaining roughly 63.5% of the variance (R2) on the unseen test set.
* The median baseline completely failed to adapt to spatial and temporal variations, yielding the highest error penalty.
* Incorporating a standardized linear year trend combined with categorical spatial offsets handled shifting yields more effectively than depth-restricted tree-based models.
* **Responsible Analytics Constraint:** The models provide baseline estimates derived strictly from historical and categorical markers. Because they lack inputs for unobserved weather events, soil nutrient concentrations, irrigation levels, and crop varieties, these statistics must not be treated as individualized operational prescriptions without localized expert agronomic validation.

---

## How to Run

### Prerequisites

The project was developed in a Python environment with the following specifications:

| Software | Version |
| :--- | :---: |
| Python | 3.13.15 |
| scikit-learn | 1.6.1 |
| pandas | 2.2.3 |

*Install required dependencies:*
```bash
pip install xlrd openpyxl scikit-learn pandas numpy matplotlib joblib
```
