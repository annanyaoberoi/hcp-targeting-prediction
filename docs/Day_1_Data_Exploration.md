# 🔬 Day 1 — Data Exploration & Quality Assessment

<p align="center">
  <img src="https://img.shields.io/badge/Phase-01%20%7C%20Data%20Exploration-6C63FF?style=for-the-badge" alt="Data Exploration"/>
  <img src="https://img.shields.io/badge/Status-Completed-2EA44F?style=for-the-badge" alt="Completed"/>
  <img src="https://img.shields.io/badge/Tools-Python%20%7C%20Pandas-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python and Pandas"/>
</p>

> **Objective:** Understand the business problem, explore the initial prescription dataset, perform basic data-quality checks, and prepare the data for subsequent analysis.

---

## 📑 Table of Contents

1. [Understanding the Business Problem](#1-understanding-the-business-problem)
2. [Key Terminology](#2-key-terminology)
3. [Dataset Overview](#3-dataset-overview)
4. [Data Quality Assessment](#4-data-quality-assessment)
5. [Data Type Conversion](#5-data-type-conversion)
6. [Descriptive Statistics](#6-descriptive-statistics)
7. [Initial Findings](#7-initial-findings)
8. [Current Limitations](#8-current-limitations)
9. [Interview Preparation](#9-interview-preparation)
10. [Day 1 Checklist](#10-day-1-checklist)

---

## 1. Understanding the Business Problem

### 💡 What is HCP Targeting?

HCP stands for Healthcare Professional. In pharmaceutical analytics, HCP targeting involves identifying healthcare providers who may be relevant for a particular product or business objective.

Pharmaceutical companies have limited resources for engaging healthcare providers. Historical prescription data can help them understand prescribing patterns and potentially prioritize providers for future engagement.

### 🎯 Our Project Objective

The goal is to develop a data science workflow that predicts which healthcare providers are likely to become **high prescribers of Jardiance in a future year**.

The project aims to answer:

- Which historical prescription patterns may indicate future prescribing activity?
- Can competitor-drug information provide useful predictive signals?
- Can a classification model help prioritize healthcare providers?

### 🔄 Planned Prediction Workflow

```text
Historical Prescription Data
            |
            ▼
      Data Exploration
            |
            ▼
    Data Cleaning & Validation
            |
            ▼
      Feature Engineering
            |
            ▼
   Define Future-Year Target
            |
            ▼
   Classification Model
            |
            ▼
 Future High-Prescriber Predictions
            |
            ▼
   Provider Prioritization Insights
```

**Important:** This is the planned workflow. Only the initial data exploration and preliminary quality checks have been completed so far.

---

## 2. Key Terminology

| Term | Meaning |
|---|---|
| HCP | Healthcare Professional |
| NPI | National Provider Identifier, a unique identifier for US healthcare providers |
| CMS | Centers for Medicare & Medicaid Services |
| Medicare Part D | US programme providing prescription drug coverage |
| Jardiance | Brand name of empagliflozin |
| Empagliflozin | Generic name of Jardiance's active ingredient |
| Prescription Claim | A prescription transaction represented in the claims data |
| Feature | An input variable used by a predictive model |
| Target Variable | The outcome the model is trained to predict |
| Classification | A machine learning task that predicts a category, such as high prescriber or not high prescriber |
| Data Leakage | Using information during training that would not legitimately be available at prediction time |

### 🧠 Understanding the Prediction Target

The planned target is a binary variable:

- `1` → Provider meets the defined future high-prescriber criterion.
- `0` → Provider does not meet that criterion.

The exact threshold and eligible provider population must be finalized after the multi-year dataset is prepared.

The model will use historical information from year **T** to predict the target in year **T+1**.

This temporal separation is important because the goal is to predict future behaviour, not merely describe past prescribing activity.

---

## 3. Dataset Overview

### 📊 Data Source

**Source:** Centers for Medicare & Medicaid Services (CMS)

**Dataset:** Medicare Part D Prescribers — by Provider and Drug

🔗 [Official CMS dataset page](https://data.cms.gov/provider-summary-by-type-of-service/medicare-part-d-prescribers/medicare-part-d-prescribers-by-provider-and-drug)

The initial extract contains Jardiance records for the reporting year identified in the current project workflow as 2021. The reporting-year metadata should be checked again before combining multiple years for modelling.

### Initial Extract

| Property | Value |
|---|---:|
| Records retrieved | 87,157 |
| Unique provider NPIs | 87,157 |
| Drug selected | Jardiance |
| Claims column | `Tot_Clms` |
| Working file | `jardiance_2021.csv` |

**Note:** The CSV filename is a working filename, not independent evidence of the reporting year.

### Important Columns

| Column | Description |
|---|---|
| `Prscrbr_NPI` | Provider identifier |
| `Prscrbr_Last_Org_Name` | Provider's last name or organization name |
| `Prscrbr_First_Name` | Provider's first name |
| `Prscrbr_City` | Provider's city |
| `Prscrbr_State_Abrvtn` | Provider's state abbreviation |
| `Prscrbr_Type` | Provider type |
| `Brnd_Name` | Drug brand name |
| `Gnrc_Name` | Drug generic name |
| `Tot_Clms` | Total prescription claims |
| `Tot_30day_Fills` | Total 30-day prescription fills |
| `Tot_Day_Suply` | Total days of medication supplied |
| `Tot_Drug_Cst` | Total drug cost |
| `Tot_Benes` | Reported beneficiary count |

The dataset also contains age-65-and-older statistics and suppression flags.

### ⚠️ Claims Are Not the Same as Patients

`Tot_Clms` measures prescription claims. It should not be interpreted as the number of unique patients.

Similarly, beneficiary counts and prescription claims represent different measures and should not be used interchangeably.

---

## 4. Data Quality Assessment

Before modelling, it is important to understand the dataset's structure and identify obvious quality issues.

### 4.1 Check Dataset Size

```python
print("Total rows:", len(df_jardiance))
print("Unique doctors:", df_jardiance["Prscrbr_NPI"].nunique())
```

**What these functions do:**

- `len(df_jardiance)` returns the number of rows.
- `nunique()` counts distinct values in the NPI column.

**Observed result:**

- Total rows: 87,157
- Unique NPIs: 87,157

Every row in this filtered extract has a distinct NPI. This does not mean each doctor appears only once in the complete provider-and-drug dataset.

### 4.2 Check Duplicate Rows

```python
print("Duplicate rows:", df_jardiance.duplicated().sum())
```

**Concepts:**

- `duplicated()` identifies rows that exactly duplicate an earlier row across the DataFrame's columns.
- `sum()` counts the rows marked as duplicates.

**Observed result:** `0`

No exact duplicate rows were found in this extract.

This check does not establish that all records are valid or that no duplicates could occur under a different business key.

### 4.3 Check Missing Values

```python
print("Missing values:")
print(
    df_jardiance[
        ["Prscrbr_NPI", "Brnd_Name", "Tot_Clms"]
    ].isna().sum()
)
```

**Concepts:**

- Selecting columns limits the check to the specified fields.
- `isna()` identifies standard missing values such as `NaN`.
- `sum()` counts the missing values in each column.

**Observed result:**

| Column | Missing values |
|---|---:|
| `Prscrbr_NPI` | 0 |
| `Brnd_Name` | 0 |
| `Tot_Clms` | 0 |

This result applies only to the three selected columns and standard pandas missing values. It does not check every column or automatically identify blank strings and suppressed values.

---

## 5. Data Type Conversion

### Why do data types matter?

Pandas needs to interpret values correctly for calculations and data processing.

During the initial inspection, the following types were observed:

```text
Prscrbr_NPI    object
Brnd_Name      object
Tot_Clms       object
```

The NPI is an identifier, and the brand name is text, so keeping them as object/string-like data is appropriate.

However, `Tot_Clms` contains numerical claim counts and needs a numeric type for statistical calculations.

### Conversion Code

```python
df_jardiance["Tot_Clms"] = pd.to_numeric(
    df_jardiance["Tot_Clms"],
    errors="coerce"
)

print(df_jardiance["Tot_Clms"].dtype)
```

### Understanding the Code

| Component | Purpose |
|---|---|
| `pd.to_numeric()` | Converts values to a numeric representation |
| `errors="coerce"` | Converts unparseable values to `NaN` |
| `.dtype` | Displays the resulting data type |

**Observed result:** `int64`

The claims column was successfully converted to an integer data type.

After conversion, the missing-value check returned zero missing claims values.

```python
print(
    "Missing claims after conversion:",
    df_jardiance["Tot_Clms"].isna().sum()
)
```

**Observed result:** `0`

This indicates that no claims values became standard missing values during this conversion.

---

## 6. Descriptive Statistics

### Code Used

```python
print(df_jardiance["Tot_Clms"].describe())
```

The `describe()` function summarizes the numerical distribution.

### Observed Results

| Statistic | Value |
|---|---:|
| Count | 87,157 |
| Mean | 38.05 |
| Standard deviation | 44.97 |
| Minimum | 11 |
| 25th percentile | 15 |
| Median (50th percentile) | 24 |
| 75th percentile | 43 |
| Maximum | 1,734 |

*Values are rounded for readability.*

### Understanding the Statistics

**Mean — 38.05**

The arithmetic average of the claims values.

**Median — 24**

The middle value when observations are ordered. Half the observations are at or below the median, and half are at or above it, subject to the usual treatment of ties.

**Standard Deviation — 44.97**

Measures the spread of claims values around their mean.

**Quartiles**

- 25th percentile: 15
- 50th percentile: 24
- 75th percentile: 43

These values help describe the distribution without relying only on the average.

### What Does Mean > Median Suggest?

The mean of approximately 38 is higher than the median of 24.

This suggests that higher claims values may be pulling the average upward. The distribution may be right-skewed, but a histogram or other distribution analysis would be needed to assess its shape more directly.

The maximum value of 1,734 is substantially higher than the median.

However, **a high value is not automatically a data error or an outlier that should be removed**. It should be investigated before any treatment is chosen.

---

## 7. Initial Findings

The initial exploration produced the following findings:

1. The Jardiance extract contains 87,157 records and 87,157 distinct NPIs.
2. No exact duplicate rows were detected.
3. The selected NPI, brand-name, and claims columns had no standard missing values.
4. `Tot_Clms` was initially stored as an object and was successfully converted to `int64`.
5. Claims counts vary considerably across records.
6. The mean is higher than the median, suggesting that high values may influence the average.
7. Further validation is needed before the data can be used for predictive modelling.

### Current Data Status

| Task | Status |
|---|---|
| Retrieve initial Jardiance extract | Completed |
| Inspect dataset structure | Completed |
| Check exact duplicate rows | Completed |
| Check missing values in selected columns | Completed |
| Convert claims to numeric | Completed |
| Review descriptive statistics | Completed |
| Validate all columns and suppression flags | Pending |
| Confirm reporting-year metadata | Pending recheck |
| Prepare multi-year modelling dataset | Pending |

---

## 8. Current Limitations

The current extract is an initial exploration dataset, not the complete modelling dataset.

The following steps remain:

- Verify reporting-year metadata using the official dataset information.
- Collect competitor-drug data, including relevant SGLT2 inhibitors.
- Collect additional reporting years.
- Inspect missing values, blank strings, suppression flags, and data types across relevant columns.
- Establish a consistent provider population and construct provider-level features.
- Define the future-year target precisely.
- Train and evaluate a model using a time-based validation strategy.

The project should not report model accuracy, predictive performance, or business impact until those analyses have actually been completed.

---

## 9. Interview Preparation

### Q1. What was the objective of your initial data exploration?

**Answer:**

"My objective was to understand the structure and quality of the initial Jardiance prescription dataset before performing further analysis. I inspected its columns and data types, checked duplicate rows and missing values in selected fields, converted total claims to a numeric format, and examined descriptive statistics."

### Q2. How many records did your initial extract contain?

**Answer:**

"The initial Jardiance extract contained 87,157 records and 87,157 unique provider NPIs. These counts apply to the filtered extract, not necessarily to the entire CMS provider-and-drug dataset."

### Q3. Why did you convert `Tot_Clms` to a numeric data type?

**Answer:**

"The claims column was initially stored as an object. I converted it to a numeric format using pandas so that I could calculate descriptive statistics and prepare the variable for further analysis."

### Q4. What did you learn from the descriptive statistics?

**Answer:**

"The mean claims count was approximately 38, while the median was 24. This difference suggests that higher values may be pulling the mean upward. The maximum was 1,734, so I would investigate the distribution and validate unusually high records before deciding whether any outlier treatment is necessary."

### Q5. Why can't you build the final predictive model from the current extract alone?

**Answer:**

"The current extract contains only Jardiance records for one reporting year. The planned model requires historical features and a future-year target. Therefore, I need to prepare data across multiple years and include relevant competitor-drug information before constructing the modelling dataset."

### Q6. What is the difference between claims and beneficiaries?

**Answer:**

"Claims represent prescription transactions, whereas beneficiaries represent reported individuals associated with the prescription data. They are different measures and should not be interpreted interchangeably."

### Q7. Does a high number of claims prove that a doctor is a better prescriber?

**Answer:**

"No. A high claims count only indicates higher recorded prescription activity in the dataset. It does not establish prescribing quality, patient outcomes, or clinical appropriateness."

---

## 10. Day 1 Checklist

- [x] Understand the business problem.
- [x] Learn the basic healthcare and prescription-data terminology.
- [x] Access the CMS API and retrieve the initial Jardiance extract.
- [x] Inspect the dataset columns and structure.
- [x] Check unique NPIs and exact duplicate rows.
- [x] Check missing values in selected columns.
- [x] Convert `Tot_Clms` to a numeric format.
- [x] Review descriptive statistics and high-claims records.
- [ ] Recheck reporting-year metadata.
- [ ] Prepare a multi-year dataset for predictive modelling.

---

<p align="center">
  <b>Day 1: Understand the data before building the model.</b>
  <br/>
  <sub>HCP Targeting & Prescription Prediction · Technical Documentation</sub>
</p>
