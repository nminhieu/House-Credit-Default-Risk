# Exploratory Data Analysis (EDA)

In the EDA phase, I focus on analyzing data characteristics from multiple tables (Bureau, Bureau Balance, Application, Credit Card Balance, …) to detect trends, anomalies, and relationships between features and the target variable `TARGET` (default = 1, non-default = 0).

---

## 1. Bureau EDA

### Categorical Variables
- **CREDIT_TYPE**: Many rare credit types → grouped into `'Rare'` (except *Consumer credit* and *Credit card*).  
- **CREDIT_ACTIVE**: The values `'Sold'` and `'Bad Debt'` were replaced with `'Active'` to focus on the current credit status.  
- **CREDIT_CURRENCY** and **SK_ID_BUREAU**: Removed due to redundancy or irrelevance.

### Numerical Variables
- **YEARS_CREDIT**: Similar distribution between the two groups → unlikely to be a strong predictor.  
- **DAYS_CREDIT_ENDDATE**: Outliers (~115 years) detected → need to be handled during preprocessing.

---

## 2. Bureau Balance EDA

- **STATUS**:  
  - Main values: `C` (closed), `X` (unknown), `0` (no overdue).  
  - Values `1–5` are rare → imbalanced data.  
- **MONTHS_BALANCE**:  
  - Represents the time offset relative to the application date.  
  - Mostly concentrated between -1 to -5 months, with some older contracts around -50 months.  

---

## 3. Missing Values & Data Quality

- **AMT_APPLICATION**: ~5% missing.  
- **PRODUCT_COMBINATION**: ~6% missing.  
- Some tables have up to **~55% of columns with NaN**, many exceeding 70%.  
- `DAYS_*` variables contain many invalid values (`365243`, equivalent to ~1000 years) → need removal or replacement.  

---

## 4. Key Observations on Features

### Application Data
- **CNT_CHILDREN**: Average <1, but extreme outliers (12 children).  
- **AMT_INCOME_TOTAL**: Outlier up to 117 million USD.  
- **DAYS_EMPLOYED**: Outlier ~1000 years.  
- **Target**: Imbalanced dataset (Defaulters ~8.1%, Non-defaulters ~91.9%).

### Categorical Variables
- Some important features correlated with `TARGET`:  
  - `OCCUPATION_TYPE`, `ORGANIZATION_TYPE`, `NAME_INCOME_TYPE`.  
  - Applicants with stable jobs/income are less likely to default.  
- **Address (REGION / CITY)**: People whose residential address ≠ work/contact address → higher risk of default.  
- **CODE_GENDER**: Males have higher default rate than females (10.15% vs 7%).  
- **FLAG_PHONE**: Applicants who provide a personal phone number have lower default rates.  
- **WALLSMATERIAL_MODE**: Customers living in wooden or low-quality material houses → higher risk of default.

### Numerical Variables
- **AMT_CREDIT, AMT_ANNUITY, AMT_GOODS_PRICE**: Defaulters often borrow smaller amounts → suggesting weaker financial status.  
- **DAYS_BIRTH**: Age group 30–40 has the highest default rate.  
- **DAYS_REGISTRATION, DAYS_ID_PUBLISH**: Registration/document changes close to application date → higher risk of default.  
- **OWN_CAR_AGE**: Newer car owners (low car age) → less likely to default.  
- **EXT_SOURCE_1/2/3**: Strong correlation with `TARGET` → key predictive features.

---


# Feature Engineering & Preprocessing

This repository documents the **feature engineering and preprocessing pipeline** applied to multiple datasets for credit risk analysis and predictive modeling. The goal of this process is to transform raw financial and behavioral data into structured, domain-relevant features that enhance model performance, interpretability, and robustness.

---

## 📂 Datasets and Processing Steps

### 1. Bureau Balance
The `bureau_balance.csv` dataset contains monthly credit status data linked via `SK_ID_BUREAU`.

- **Preprocessing**
  - Label-encoded the `STATUS` column (e.g., Closed loans = 0, other statuses progressively encoded by severity).
  - Created `WEIGHTED_STATUS = STATUS / (MONTHS_BALANCE + 1)` to assign higher importance to recent months.
  - Applied Exponential Weighted Moving Average (EWM, α = 0.8) to capture recent credit behavior.

- **Aggregation**
  - Summarized at loan level: mean, max, first, and last values of `STATUS` and `WEIGHTED_STATUS`.
  - Computed time-specific metrics: recent two years (`YEAR_0`, `YEAR_1`) and older records (`YEAR_REST`).
  - Filled missing values with 0 for consistency.

- **Applications**
  - Metrics like `STATUS_mean` and `STATUS_max` indicate default risk levels.
  - Time-based features capture shifts in credit behavior over years.
  - Aggregated data merged with customer-level datasets for modeling.

---

### 2. Bureau
The `bureau.csv` dataset tracks loan-level credit history per customer (`SK_ID_CURR`).

- **Domain-Specific Features**
  - `CREDIT_DURATION = DAYS_CREDIT_ENDDATE - DAYS_CREDIT`
  - `FLAG_OVERDUE_RECENT` → recent overdue indicator.
  - Ratios: `MAX_AMT_OVERDUE_DURATION_RATIO`, `CURRENT_DEBT_TO_CREDIT_RATIO`, `AMT_ANNUITY_CREDIT_RATIO`.
  - Dependency metrics: `CNT_PROLONGED_MAX_OVERDUE_MUL`.

- **Preprocessing**
  - Cleaned erroneous time values (e.g., unrealistic `DAYS_CREDIT_ENDDATE`).
  - One-hot encoded categorical features: `CREDIT_ACTIVE`, `CREDIT_TYPE`, `CREDIT_CURRENCY`.

- **Aggregation**
  - Aggregated loan-level data to customer level.
  - Separated metrics by `CREDIT_ACTIVE` status (Active vs. Closed).
  - Merged with `bureau_balance` aggregations.

- **Applications**
  - Provides customer-level credit history snapshots.
  - Overdue and ratio-based features highlight financial stress and risk.

---

### 3. Previous Application
The `previous_application.csv` dataset contains past loan applications.

- **Preprocessing**
  - Replaced placeholder values (e.g., `365243`) with NaN.
  - Filled missing categorical values with `XNA`.
  - Created `MISSING_VALUES_TOTAL_PREV` = count of missing values per row.

- **Feature Engineering**
  - Discrepancy features: `AMT_DECLINED`, `AMT_CREDIT_GOODS_RATIO`, `AMT_CREDIT_APPLICATION_RATIO`.
  - Interest-related: `INTEREST_DOWNPAYMENT`, `INTEREST_CREDIT`.
  - Temporal: `DAYS_FIRST_LAST_DUE_DIFF`, `APPLICATION_AMT_TO_DECISION_RATIO`.

- **Aggregation**
  - Aggregated by customer (`SK_ID_CURR`) over:
    - Most recent 5 applications.
    - Earliest 2 applications.
    - All applications.

- **Applications**
  - Captures loan approval discrepancies, repayment efficiency, and historical patterns.
  - Provides insights into borrower consistency.

---

### 4. Credit Card Balance
The `credit_card_balance.csv` dataset records monthly credit card usage and repayment.

- **Preprocessing**
  - Outlier handling: capped extreme values in `AMT_PAYMENT_CURRENT`.
  - Missing values counted as `MISSING_VALS_TOTAL_CC`.
  - Converted negative `MONTHS_BALANCE` to absolute values.

- **Feature Engineering**
  - Utilization ratios: `BALANCE_LIMIT_RATIO`, `AMT_DRAWING_SUM`.
  - Repayment patterns: `MIN_PAYMENT_RATIO`, `PAYMENT_MIN_DIFF`.
  - Delinquency risk: `SK_DPD_RATIO`.
  - Exponential Weighted Moving Averages (EWMAs) applied to highlight recent trends.

- **Aggregation**
  - By loan (`SK_ID_PREV`) and customer (`SK_ID_CURR`).
  - Grouped by contract status (Active, Completed).
  - Time-specific aggregations by year cohorts.

- **Applications**
  - Captures repayment consistency, utilization intensity, and delinquency patterns.
  - Provides temporal dynamics for predictive modeling.

---

### 5. Installment Payments
The `installments_payments.csv` dataset details customer installment behaviors.

- **Preprocessing & Features**
  - Timeliness: `DAYS_PAYMENT_RATIO`, `DAYS_PAYMENT_DIFF`.
  - Adequacy: `AMT_PAYMENT_RATIO`, `AMT_PAYMENT_DIFF`.
  - Smoothed features: EWMs for timeliness and adequacy (`EXP_*`).
  - Loan duration: `TOTAL_TERM = CNT_INSTALMENT + CNT_INSTALMENT_FUTURE`.

- **Aggregation**
  - At loan level (`SK_ID_PREV`): mean, sum, max, last.
  - Time-based: recent year, first five installments.
  - At customer level (`SK_ID_CURR`): aggregated loan summaries.

- **Applications**
  - Identifies customers with habitual late or underpayments.
  - Captures evolving repayment behaviors for risk models.

---

### 6. Application Train/Test
The main training and testing datasets (`application_train.csv` and `application_test.csv`).

- **Preprocessing**
  - Removed low-variance features (e.g., document flags).
  - Fixed erroneous placeholders (`DAYS_EMPLOYED = 365243 → NaN`).
  - Created `MISSING_VALS_TOTAL_APP`.
  - Standardized data types.

- **Feature Engineering**
  - Ratios: `CREDIT_INCOME_RATIO`, `AMT_ANNUITY_CREDIT_RATIO`.
  - Differences: `AGE_EMPLOYED_DIFF`, `INCOME_ANNUITY_DIFF`.
  - Interaction terms: `EXT_SOURCE_MUL`, `REGION_RATING_MUL`.
  - Target-based features using KNN mean target encoding.

- **Applications**
  - Final dataset enriched with domain-specific, ratio-based, and interaction features.
  - Forms the backbone of predictive modeling pipelines.

---

## ⚙️ Pipeline Highlights
- **Cleaning & Standardization**: Removed erroneous values, imputed missing data.  
- **Domain-Specific Feature Engineering**: Ratios, flags, differences, temporal metrics.  
- **Aggregation**: Loan-level → customer-level metrics.  
- **Temporal Emphasis**: EWMs highlight recent behavior.  
- **Integration**: All datasets merged via `SK_ID_CURR` for comprehensive modeling.  

---

## 🚀 Applications in Credit Risk Modeling
- Default probability estimation (`STATUS_mean`, `STATUS_max`).  
- Behavioral trend detection (`EXP_WEIGHTED_STATUS_last`, `EXP_DAYS_PAYMENT_RATIO`).  
- Risk segmentation via overdue and debt-to-credit ratios.  
- Time-series enriched features enable **robust predictive modeling**.  

---
# 📊 Loan Default Prediction - Modelling

## 🎯 Objective
The primary goal of this modeling task was to **predict the likelihood of loan default by clients**, aiding in effective **risk management** and **decision-making**.  
Two models were developed and evaluated: **Decision Tree** and **Logistic Regression with Elastic Net regularization**.

---

## 🌳 Decision Tree

### 🔹 Why Decision Tree?
The **Decision Tree algorithm** was chosen as the baseline model because:
- **Interpretability**: The resulting tree structure can be easily visualized and understood.  
- **Minimal preprocessing**: Handles both numerical and categorical data without scaling.  
- **Efficiency**: Works well for initial benchmarks.  

### ⚙️ Hyperparameter Tuning
Hyperparameters were tuned using **Random Search + Cross-Validation**.  
The search grid included:
- `max_depth`: [5, 10, 15, 20, None]  
- `min_samples_split`: [2, 5, 10, 20]  
- `min_samples_leaf`: [1, 5, 10]  
- `max_features`: ["sqrt", "log2", None]  
- `criterion`: ["gini", "entropy"]

The **optimal parameters** selected were:  
`max_depth=5, min_samples_split=10, min_samples_leaf=5, max_features=None, criterion="entropy"`

### 📈 Results
- **Best threshold (J-statistic)**: 0.0861  
- **Training set**:
  - ROC-AUC: `0.7577`  
  - Precision: `0.1652`  
  - Recall: `0.6930`  
- **Cross-validation**:
  - ROC-AUC: `0.7516`  
  - Precision: `0.1639`  
  - Recall: `0.6922`  

✅ The Decision Tree baseline demonstrates **balanced recall and precision** while maintaining **competitive discriminatory power**.

---

## 📉 Logistic Regression with Elastic Net

### 🔹 Why Logistic Regression + Elastic Net?
Logistic Regression provides a **probabilistic framework** for binary classification.  
By using **Elastic Net regularization (L1 + L2)**, the model gains:
- **Feature selection ability (L1)**  
- **Stability and robustness (L2)**  
- **Better generalization** on high-dimensional data  

### ⚙️ Parameters
The model was trained using **SGD with Elastic Net regularization**, with parameters:
```python
params = {
    'loss': 'log_loss',
    'penalty': 'elasticnet',
    'random_state': 42,
    'class_weight': 'balanced',
    'n_jobs': -1,
    'learning_rate': 'adaptive',
    'l1_ratio': 0.16,
    'eta0': 0.011616
}


⚙️ Hyperparameter Tuning
random_search_cv(hyperparams, n_iter=67, n_jobs=2)
hyperparams = {'alpha': np.logspace(-4, 2)}

📈 Results
- **Best threshold (J-statistic)**: 0.4719 
- **Training set**:
  - ROC-AUC: `0.7965`  
  - Precision: `0.1807`  
  - Recall: `0.7400`  
- **Cross-validation**:
  - ROC-AUC: `0.7894`  
  - Precision: `0.1784`  
  - Recall: `0.7299`  
