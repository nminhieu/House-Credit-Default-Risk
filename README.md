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

