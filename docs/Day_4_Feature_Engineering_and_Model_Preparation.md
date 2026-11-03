# 🩷 DAY 04 — Feature Engineering & Model Preparation

<p align="center">
  <img src="https://img.shields.io/badge/PROJECT-HCP%20TARGETING-800020?style=for-the-badge" />
  <img src="https://img.shields.io/badge/DAY-04-FFFFFF?style=for-the-badge&labelColor=800020" />
  <img src="https://img.shields.io/badge/STATUS-COMPLETED-2E7D32?style=for-the-badge" />
</p>

<p align="center">
  <b>Transforming raw prescription data into meaningful predictive features.</b>
</p>

---

## 📌 1. Day 04 Overview

The objective of Day 4 was to prepare the provider-level dataset for machine learning by analysing existing features, engineering additional variables, understanding feature distributions, examining correlations, and preparing training and testing datasets.

Starting with **16 original features**, we created **5 additional features**, resulting in a final dataset containing **21 predictive features**.

### Today's Objectives

- Inspect the existing feature matrix.
- Analyse zero values and feature distributions.
- Understand skewness and extreme values.
- Perform feature engineering.
- Analyse relationships between features.
- Prepare training and testing datasets.
- Validate the final model inputs.

---

## 🔍 2. Feature Inspection

The original feature matrix contained 16 numerical features representing four SGLT2 inhibitor drugs:

| Drug | Prescription metrics |
|---|---|
| Jardiance | Claims, fills, days supply, drug cost |
| Farxiga | Claims, fills, days supply, drug cost |
| Invokana | Claims, fills, days supply, drug cost |
| Steglatro | Claims, fills, days supply, drug cost |

**Initial dataset dimensions:**

| Parameter | Value |
|---|---:|
| Healthcare providers | 101,500 |
| Original features | 16 |
| Missing values | 0 |

All features were numerical and suitable for further analysis.

---

## 📊 3. Zero-Value Analysis

We examined the number and percentage of zero values across the original features.

### Key Observations

| Drug | Zero percentage |
|---|---:|
| Steglatro | 99.38% |
| Invokana | 87.31% |
| Farxiga | 61.65% |
| Jardiance | 14.13% |

**Interpretation:**

- Steglatro and Invokana had very high zero-value percentages.
- Farxiga also had a considerable number of zero values.
- Most providers in the modelling cohort had reported Jardiance prescription activity.

Zero values were retained because they represent meaningful prescribing information rather than missing observations.

Providers who prescribe competitor drugs may represent relevant opportunities for pharmaceutical targeting.

---

## ⚙️ 4. Feature Engineering

Feature engineering was performed to derive additional business-relevant information from existing prescription metrics.

Five new features were created.

### Feature 1 — Total SGLT2 Claims

**Purpose:** Measure the overall SGLT2 prescribing activity of each healthcare provider.

Formula:

`Total SGLT2 Claims = Jardiance + Farxiga + Invokana + Steglatro Claims`

```python
X_engineered = X.copy()

X_engineered["total_sglt2_claims_2021"] = (
    X_engineered["Jardiance_Tot_Clms"] +
    X_engineered["Farxiga_Tot_Clms"] +
    X_engineered["Invokana_Tot_Clms"] +
    X_engineered["Steglatro_Tot_Clms"]
)
```

**Observed mean:** 47.33 claims per provider.

### Feature 2 — Competitor Claims

**Purpose:** Measure prescription activity involving competing SGLT2 brands.

Formula:

`Competitor Claims = Farxiga + Invokana + Steglatro Claims`

```python
X_engineered["competitor_claims_2021"] = (
    X_engineered["Farxiga_Tot_Clms"] +
    X_engineered["Invokana_Tot_Clms"] +
    X_engineered["Steglatro_Tot_Clms"]
)
```

**Observed mean:** 14.66 claims per provider.

### Feature 3 — Jardiance Claim Share

**Purpose:** Measure the proportion of an HCP's total SGLT2 claims attributed to Jardiance.

Formula:

`Jardiance Claim Share = Jardiance Claims / Total SGLT2 Claims`

```python
X_engineered["jardiance_claim_share_2021"] = (
    X_engineered["Jardiance_Tot_Clms"] /
    X_engineered["total_sglt2_claims_2021"]
)
```

**Observed mean:** 0.7304 (approximately 73%).

This feature captures Jardiance's share of the provider's SGLT2 prescription activity.

### Feature 4 — Number of Competitor Drugs Prescribed

**Purpose:** Identify how many different competitor brands each HCP prescribed.

```python
competitor_cols = [
    "Farxiga_Tot_Clms",
    "Invokana_Tot_Clms",
    "Steglatro_Tot_Clms"
]

X_engineered["num_competitor_drugs_2021"] = (
    (X_engineered[competitor_cols] > 0).sum(axis=1)
)
```

**Observed distribution:**

| Competitor brands prescribed | HCP count |
|---:|---:|
| 0 | 56,095 |
| 1 | 38,555 |
| 2 | 6,670 |
| 3 | 180 |

This feature captures competitor adoption breadth rather than just total prescription volume.

### Feature 5 — Total SGLT2 Drug Cost

**Purpose:** Measure the combined reported drug cost across all four SGLT2 brands.

```python
cost_cols = [
    "Jardiance_Tot_Drug_Cst",
    "Farxiga_Tot_Drug_Cst",
    "Invokana_Tot_Drug_Cst",
    "Steglatro_Tot_Drug_Cst"
]

X_engineered["total_sglt2_drug_cost_2021"] = (
    X_engineered[cost_cols].sum(axis=1)
)
```

**Observed mean:** 44,237.41.

The metric represents reported drug cost in the CMS dataset, not the healthcare provider's personal expenditure.

---

## 📈 5. Feature Validation

After feature engineering, all five newly created features were inspected using descriptive statistics.

| Engineered feature | Mean |
|---|---:|
| Total SGLT2 claims | 47.33 |
| Competitor claims | 14.66 |
| Jardiance claim share | 0.7304 |
| Number of competitor drugs | 0.5166 |
| Total SGLT2 drug cost | 44,237.41 |

**Validation results:**

- All five features contained 101,500 observations.
- No missing values were found.
- Jardiance claim share ranged from 0 to 1.
- Competitor drug count ranged from 0 to 3.
- All calculated features had valid observed ranges.

---

## 📉 6. Skewness Analysis

Skewness was calculated to understand the distribution of numerical features and identify the presence of extreme values.

### Observed Skewness

| Feature | Skewness |
|---|---:|
| Competitor claims | 8.54 |
| Total SGLT2 claims | 8.04 |
| Total SGLT2 drug cost | 6.66 |
| Number of competitor drugs | 0.85 |
| Jardiance claim share | -1.04 |

### Key Findings

- Claims and drug cost features showed strong positive skewness.
- A relatively small number of providers had exceptionally high prescription volumes.
- Competitor drug count showed moderate positive skewness.
- Jardiance claim share showed negative skewness, with many providers having values close to 1.

**Preprocessing decision:** Log transformation was considered but not applied at this stage. Original features were retained to keep the modelling process straightforward and interpretable.

---

## 🔗 7. Feature Correlation Analysis

A correlation matrix and heatmap were generated to examine relationships between the predictive features.

```python
import matplotlib.pyplot as plt
import seaborn as sns

correlation_matrix = X_engineered.corr()

plt.figure(figsize=(16, 12))

sns.heatmap(
    correlation_matrix,
    cmap="coolwarm",
    center=0,
    annot=False
)

plt.title("Feature Correlation Heatmap")
plt.show()
```

### Key Observations

**1. Related prescription metrics**

Claims, 30-day fills, days supply, and drug cost for the same drug showed strong relationships.

**2. Total and individual claims**

Total SGLT2 claims were related to individual drug claims because the total is calculated by adding the four brand-level claim counts.

**3. Competitor prescribing**

Competitor claims and the number of competitor drugs prescribed provided additional information about competitor adoption.

**4. Jardiance claim share**

Jardiance claim share showed a negative relationship with competitor adoption.

### Modelling Decision

Correlated features were retained at this stage.

Tree-based algorithms can generally work with correlated features without requiring feature scaling. Their predictive performance and feature importance will be examined during model development.

---

## 🧹 8. Final Feature Matrix

The final feature matrix was prepared using:

- 16 original prescription features.
- 5 engineered features.

Optional log-transformed features were excluded.

```python
# Remove optional log-transformed columns if present
log_cols = [
    col for col in X_engineered.columns
    if col.endswith("_log")
]

X_engineered = X_engineered.drop(columns=log_cols)

print("Final feature matrix shape:", X_engineered.shape)
print("Missing values:", X_engineered.isnull().sum().sum())
```

### Final Dataset Dimensions

| Parameter | Value |
|---|---:|
| Healthcare providers | 101,500 |
| Original features | 16 |
| Engineered features | 5 |
| **Total features** | **21** |
| Missing values | **0** |

---

## ✂️ 9. Train-Test Split

The prepared dataset was divided into training and testing subsets.

- Training dataset: 80%
- Testing dataset: 20%

The `train_test_split()` function from Scikit-learn was used.

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X_engineered,
    y,
    test_size=0.20,
    random_state=42,
    stratify=y
)
```

### Parameters Explained

| Parameter | Purpose |
|---|---|
| `test_size=0.20` | Reserves 20% for testing |
| `random_state=42` | Ensures reproducibility |
| `stratify=y` | Preserves target-class proportions |

Stratification was particularly important because the target variable was imbalanced.

### Split Results

| Dataset | Records | Features |
|---|---:|---:|
| Training | 81,200 | 21 |
| Testing | 20,300 | 21 |
| **Total** | **101,500** | **21** |

### Target Distribution After Splitting

| Target class | Training | Testing |
|---|---:|---:|
| Non-high prescriber (0) | 79.67% | 79.67% |
| High prescriber (1) | 20.33% | 20.33% |

Both datasets maintained the same target-class distribution.

**Note:** This is a provider-level random holdout split. The prediction setup remains based on 2021 prescribing features and 2022 outcomes.

---

## ✅ 10. Final Preparation Validation

The final checks confirmed that the training and testing datasets were ready for model development.

```python
print("X_train missing values:", X_train.isnull().sum().sum())
print("X_test missing values:", X_test.isnull().sum().sum())

print("y_train missing values:", y_train.isnull().sum())
print("y_test missing values:", y_test.isnull().sum())
```

### Validation Summary

| Validation | Result |
|---|---|
| Training feature missing values | 0 |
| Testing feature missing values | 0 |
| Training target missing values | 0 |
| Testing target missing values | 0 |
| Training records | 81,200 |
| Testing records | 20,300 |
| Stratified split | Successful |

<blockquote>
**Final Outcome:** The dataset is clean, validated, and prepared for machine learning model development.
</blockquote>

---

## 🏁 11. Day 04 Deliverables

By the end of Day 4, the following components were completed:

- [x] Inspected all 16 original features.
- [x] Analysed zero-value distribution.
- [x] Examined feature skewness.
- [x] Created five engineered features.
- [x] Validated the engineered features.
- [x] Generated a feature correlation heatmap.
- [x] Prepared the final 21-feature matrix.
- [x] Performed stratified train-test splitting.
- [x] Verified missing values and dataset dimensions.

---

## 🚀 12. Next — Day 05: Machine Learning Model Development

With feature engineering and data preparation complete, the next phase focuses on training and evaluating machine learning models.

Planned algorithms:

1. Logistic Regression — baseline classification model.
2. Decision Tree — interpretable tree-based model.
3. Random Forest — ensemble learning model.
4. Gradient Boosting — sequential ensemble learning model.

The models will be evaluated using suitable classification metrics, considering the imbalance in the target variable.

---

<p align="center">
  <b>DAY 04 COMPLETED</b><br/>
  <i>From raw prescription data to a machine-learning-ready dataset.</i>
</p>

<p align="center">
  🩷 HCP TARGETING & PRESCRIPTION PREDICTION
</p>
