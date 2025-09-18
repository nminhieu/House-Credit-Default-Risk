# Exploratory Data Analysis (EDA)

In the EDA phase, we focus on analyzing data characteristics from multiple tables (Bureau, Bureau Balance, Application, Credit Card Balance, …) to detect trends, anomalies, and relationships between features and the target variable `TARGET` (default = 1, non-default = 0).

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

## 5. Conclusions from EDA
- Dataset has multiple quality issues (missing values, outliers, irrelevant variables).  
- Many categorical features are imbalanced → need to be handled during feature engineering.  
- Several standout features with strong predictive power: `EXT_SOURCE` variables, job/income information, address characteristics.  
- Time-related variables (`DAYS_*`) contain many errors but also hold important signals for credit risk.
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

## 📌 Conclusion
The preprocessing and feature engineering pipeline transforms raw, complex financial data into structured, insightful features. By combining **domain knowledge, statistical transformations, and temporal analysis**, this workflow provides a robust foundation for machine learning models in **credit risk prediction**.


