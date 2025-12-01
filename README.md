# Avito Deal Probability – Regression Pipeline

## Overview
This project builds a full end-to-end regression pipeline for predicting the `deal_probability` of Avito listings.  
The workflow includes dataset loading, preprocessing, feature engineering, encoding, outlier treatment, model training, Optuna hyperparameter tuning, feature importance analysis, SHAP explainability, and ensemble learning.  
The project evaluates both baseline and optimized versions of KNN, Random Forest, XGBoost, LightGBM, CatBoost, Logistic Regression (WOE), Voting Regressor, and Stacking Regressor.

---

## Dataset
The dataset can be viewed or downloaded from Google Drive:

 **Google Drive Dataset:**  
https://docs.google.com/spreadsheets/d/1Qv5rrzmkFZCzbKnovFikmisG-KiIz8DM/edit?usp=drive_link&ouid=117339924051817037398&rtpof=true&sd=true

A 10% sample is used during initial development to speed up training and experimentation.

---

## Data Preprocessing
The preprocessing pipeline includes the following steps:

- Removal of low-variance columns (features with >98% identical values).
- Removal of single-unique-value columns.
- Handling missing values:
  - param_3 filled with "Unknown".
  - All other categorical features filled with mode.
  - Numerical features filled with mean.
- Grouping rare categories:
  - category_name grouped → "Other" where frequency is low.
  - param_1, param_2, param_3 grouped into new clustered feature groups.
- Target encoding with smoothing applied to selected categorical features.
- General cleaning and preparation of variables for modeling.

---

## Feature Engineering
The dataset is enriched with additional predictive signals:

- Popularity statistics:
  - Category-level frequency.
  - Region-level activity.
- Text-derived features such as description length.
- Date features extracted from activation date:
  - year, month, weekday.
- City grouping:
  - Top 15 cities preserved, all others set to `"Other"`.
- Grouping high-cardinality values (real estate, cars, item size, etc.) into logical clusters.
- Removal of unused columns such as IDs, raw text, and redundant parameters.

---

## Outlier Treatment
Outlier clipping is applied for the KNN dataset using the IQR method  
(interquartile range), ensuring extreme values do not distort distance-based models.

---

## Dataset Preparation for Model Groups
The project uses four separate datasets, each designed for a specific model family:

### **1. Logistic Regression Dataset (data_lr)**
- WOE (Weight of Evidence) transformation applied to all suitable features.
- Low-correlation variables removed.
- Multicollinearity reduced by dropping highly correlated WOE variables.

### **2. KNN Dataset (data_knn)**
- Outlier clipping using IQR.
- Low-correlation and multicollinear features removed.
- Categorical variables encoded with One-Hot Encoding.
- Numerical features scaled using StandardScaler.

### **3. RF, XGBoost, LightGBM Dataset (data_rf_xgb_lgb_cb)**
- All categorical variables processed with Label Encoding.
- Data prepared for tree-based models that do not require scaling.

### **4. CatBoost Dataset (data_cbc)**
- Native CatBoost categorical feature handling.
- Categorical column indices passed directly to the CatBoost model.

---

## Model Training (Baseline Models)
The following baseline regressors are trained and evaluated:

- **KNN Regressor**  
- **Random Forest Regressor**  
- **XGBoost Regressor**  
- **LightGBM Regressor**  
- **CatBoost Regressor**  
- **Custom CatBoost with categorical indices**  
- **Logistic Regression (WOE-based dataset)**  

Each model is evaluated using:
- R²  
- RMSE  
- MAE  
- MSE  

Train/test performance gaps are analyzed to detect overfitting or underfitting.

---

## Hyperparameter Optimization (Optuna)
All models undergo Optuna hyperparameter tuning:

- KNN Optuna-optimized  
- Random Forest Optuna-optimized  
- XGBoost Optuna-optimized  
- LightGBM Optuna-optimized  
- CatBoost Optuna-optimized  

After tuning, each model is retrained using the best trial parameters and compared with baseline performance.  
The highest-performing model is then selected for further feature importance and explainability analysis.

---

## Feature Importance
Feature importance analysis is performed **based on the best-performing model** selected from the tuned results.  
This ensures that the interpretation reflects the strongest and most reliable model in the pipeline.

- **CatBoost Feature Importance:**  
  Ranked feature importances are extracted, and only meaningful features (importance between 0.01 and 0.35) are visualized for clarity.

- **Reduced Feature Re-training:**  
  The selected important features are used to retrain the model, improving stability and reducing noise.

- **SHAP Analysis:**  
  SHAP values are calculated for the top model to explain global and local feature contributions, showing how each feature influences deal probability.

Key influential features include:
price, description length, region activity, category encodings, grouped parameters, and item sequence number.

---

## Ensemble Models
The project includes two ensemble techniques for improved performance:

### **Voting Regressor**
Combines:
- Optimized LightGBM  
- Optimized XGBoost  
- Optimized CatBoost  

### **Stacking Regressor**
- Base models: XGBoost, LightGBM, CatBoost  
- Final estimator: Random Forest  
- Passthrough enabled  
- 5-fold cross-validation  

These ensembles can outperform individual models when the base learners perform well; however, if the underlying models are weak, the ensemble may also reflect that lower performance.


---

## Final Output & Results
The project concludes with:

- A complete comparison of all baseline and optimized models  
- Final ranking based on Test R²  
- Train/Test gap analysis  
- Best performing ensemble configuration  
- Summary of most impactful features  
- SHAP-based interpretability results  

This pipeline delivers a fully structured, production-ready regression solution for predicting Avito deal probability.
