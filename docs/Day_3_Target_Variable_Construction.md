# DAY 03 — 2022 DATA COLLECTION & TARGET VARIABLE CONSTRUCTION

<p align="center">
  <strong>HCP Targeting & Prescription Prediction</strong><br/>
  <em>Building a time-based prediction dataset using CMS Medicare Part D data</em>
</p>

---

## 1. Objective

The objective of Day 3 was to collect 2022 prescription data, aggregate it at the healthcare provider level, and construct the target variable for our machine learning problem.

Our prediction strategy:

- **Input features (X):** Provider-level prescription behaviour from 2021.
- **Target (y):** Whether a provider became a high Jardiance prescriber in 2022.

This creates a time-based prediction problem where historical prescribing behaviour is used to predict future prescribing patterns.

## 2. 2022 Data Collection

Collected prescription records for four SGLT2 inhibitor drugs from the CMS Medicare Part D Prescribers dataset using the CMS Data API.

| Drug | Generic Name | Records Collected |
|---|---|---:|
| Jardiance | Empagliflozin | 120,885 |
| Farxiga | Dapagliflozin | 60,436 |
| Invokana | Canagliflozin | 10,677 |
| Steglatro | Ertugliflozin | 664 |
| **Total** | | **192,662** |

**Dataset overview:**

- Total prescription records: 192,662
- Unique providers: 139,260
- Total columns: 22

## 3. Data Cleaning & Type Conversion

Initially, the dataset columns were stored as object data types.

Converted the following four core prescription metrics into numerical formats using `pd.to_numeric()`:

| Column | Description |
|---|---|
| `Tot_Clms` | Total prescription claims |
| `Tot_30day_Fills` | Total standardized 30-day fills |
| `Tot_Day_Suply` | Total days of medication supplied |
| `Tot_Drug_Cst` | Total drug cost |

Used `errors="coerce"` to handle invalid numerical values safely.

**Missing-value observations:**

Some beneficiary and age-related fields contained blanks or suppressed values. These were not blindly replaced with zero because missing or suppressed information does not necessarily mean zero.

All four core numerical metrics had **zero missing values** after conversion.

## 4. Provider-Level Aggregation

Aggregated the 2022 prescription records using the provider's NPI as the unique identifier.

Created separate metrics for each of the four drugs using a pivot table.

```python
features = [
    "Tot_Clms",
    "Tot_30day_Fills",
    "Tot_Day_Suply",
    "Tot_Drug_Cst"
]

provider_2022_df = combined_2022_df.pivot_table(
    index="Prscrbr_NPI",
    columns="Brnd_Name",
    values=features,
    aggfunc="sum",
    fill_value=0
)

provider_2022_df.columns = [
    f"{drug}_{metric}"
    for metric, drug in provider_2022_df.columns
]

provider_2022_df = provider_2022_df.reset_index()
```

**Aggregation results:**

| Validation | Result |
|---|---:|
| Unique providers | 139,260 |
| Dataset shape | 139,260 × 17 |
| Duplicate NPIs | 0 |
| Missing values | 0 |

Each row now represents one healthcare provider, with separate prescription metrics for each drug.

## 5. Understanding 2022 Jardiance Prescriptions

Before constructing the target variable, examined the distribution of Jardiance claims.

| Metric | Value |
|---|---:|
| Total providers | 139,260 |
| Providers with zero Jardiance claims | 18,375 |
| Providers with Jardiance claims | 120,885 |
| Mean claims | 36.10 |
| Median claims | 22 |
| 75th percentile | 42 |
| Maximum claims | 3,138 |

This distribution helped establish an appropriate threshold for identifying high Jardiance prescribers.

## 6. Creating the Modelling Cohort

Our 2021 provider-level dataset contained 101,500 unique providers.

Since the model uses 2021 data as its input, we retained this 2021 provider population as the modelling cohort.

Merged the 2022 Jardiance claims with the 2021 provider-level dataset using `Prscrbr_NPI`.

Used a **left join** to preserve all providers from 2021.

Providers without a matching 2022 Jardiance record were assigned zero 2022 Jardiance claims.

```python
modeling_df = provider_df.merge(
    outcomes_2022,
    on="Prscrbr_NPI",
    how="left",
    validate="one_to_one"
)

modeling_df["jardiance_claims_2022"] = (
    modeling_df["jardiance_claims_2022"]
    .fillna(0)
    .astype(int)
)
```

**Final cohort validation:**

| Validation | Result |
|---|---:|
| Total providers | 101,500 |
| Total columns | 18 |
| Duplicate NPIs | 0 |
| Missing values | 0 |
| Zero Jardiance-claim providers | 21,186 |

## 7. Target Variable Construction

### Business definition

A pharmaceutical company cannot engage with every healthcare provider equally. Therefore, identifying providers with high future Jardiance prescribing activity can help prioritize HCP targeting efforts.

For this project, defined a high prescriber as a provider whose 2022 Jardiance claims were at or above the **80th percentile** of the modelling cohort.

### Target definition

| Target | Meaning |
|---|---|
| `1` | High Jardiance prescriber |
| `0` | Not a high Jardiance prescriber |

### Implementation

```python
threshold = modeling_df[
    "jardiance_claims_2022"
].quantile(0.80)

modeling_df["high_prescriber_2022"] = (
    modeling_df["jardiance_claims_2022"] >= threshold
).astype(int)
```

### Target distribution

The calculated 80th-percentile threshold was **60 claims**.

| Class | Providers | Percentage |
|---|---:|---:|
| Not High Prescriber (0) | 80,870 | 79.67% |
| High Prescriber (1) | 20,630 | 20.33% |
| **Total** | **101,500** | **100%** |

The high-prescriber class represents approximately 20% of the cohort. The slight difference from exactly 20% is due to providers tied at the threshold.

## 8. Feature Matrix & Target Separation

Prepared the feature matrix and target vector for subsequent machine learning.

- `X`: 16 prescription-related features from 2021.
- `y`: Binary target `high_prescriber_2022` from 2022.

```python
feature_cols = [
    col for col in provider_df.columns
    if col != "Prscrbr_NPI"
]

X = modeling_df[feature_cols]
y = modeling_df["high_prescriber_2022"]

print("Feature matrix shape:", X.shape)
print("Target shape:", y.shape)
```

**Validation results:**

| Component | Shape |
|---|---|
| Feature matrix (X) | 101,500 × 16 |
| Target vector (y) | 101,500 |
| Missing values in X | 0 |
| Missing values in y | 0 |

**Important — Data Leakage Prevention**

The following columns must not be used as model input features:

- `jardiance_claims_2022`: Used to construct the target.
- `high_prescriber_2022`: The target itself.
- `Prscrbr_NPI`: Provider identifier, not a predictive feature.

Only 2021 prescription features will be used for model training.

## 9. Final Dataset Export

Saved the prepared dataset as:

`hcp_targeting_model_dataset.csv`

```python
modeling_df.to_csv(
    "hcp_targeting_model_dataset.csv",
    index=False
)
```

The dataset contains the 2021 provider-level features, 2022 Jardiance claims and the binary target variable.

## 10. Day 3 Outcome

<details>
<summary><strong>Completed Tasks</strong></summary>

- [x] Collected 2022 CMS Medicare Part D data.
- [x] Processed four SGLT2 inhibitor drugs.
- [x] Converted core numerical columns.
- [x] Aggregated data at provider level.
- [x] Examined the 2022 Jardiance claims distribution.
- [x] Created the 2021 modelling cohort.
- [x] Merged 2021 features with 2022 outcomes.
- [x] Constructed the binary target variable.
- [x] Separated X and y.
- [x] Validated and exported the final dataset.

</details>

## 11. Project Progress

| Project Stage | Status |
|---|---|
| Day 1 — Data Collection & Exploration | Completed |
| Day 2 — Competitor Data & Preprocessing | Completed |
| Day 3 — Target Variable Construction | Completed |
| Day 4 — Feature Engineering & Model Preparation | Upcoming |
| Day 5 — Machine Learning Model Development | Upcoming |
| Day 6 — Evaluation & Business Insights | Upcoming |

---

<p align="center">
  <strong>DAY 03 COMPLETED</strong><br/>
  <em>From historical prescription data to a future prediction target.</em>
</p>
