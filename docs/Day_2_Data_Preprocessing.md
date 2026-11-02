# ✦ Day 02 — Data Integration & Feature Engineering

> Transforming raw prescription records into a structured, provider-level dataset for predictive modelling.

---

## 🎯 Today's Objective

Expand the initial Jardiance dataset by incorporating competitor drug data, perform essential preprocessing, and engineer provider-level features.

The goal was to move from individual drug-level prescription records to a unified dataset containing one row per healthcare provider (HCP).

## 01 · Data Collection

**Source:** CMS Medicare Part D Prescribers Dataset  
**Reporting Year:** 2021  
**Access Method:** Official CMS Data API

Three competitor drugs belonging to the SGLT2 inhibitor class were retrieved to complement the existing Jardiance dataset.

| Drug Brand | Records |
|:---|---:|
| Jardiance | 87,157 |
| Farxiga | 38,921 |
| Invokana | 12,880 |
| Steglatro | 634 |
| **Total** | **139,592** |

All datasets were collected using API pagination and combined into a single DataFrame.

## 02 · Dataset Integration

The four individual drug datasets were concatenated to create a unified prescription dataset.

| Dataset Metric | Value |
|:---|---:|
| Total Records | 139,592 |
| Unique Providers | 101,500 |
| Total Columns | 22 |
| Duplicate Records | 0 |

**Key Identifier:** `Prscrbr_NPI`

NPI (National Provider Identifier) was used to identify individual healthcare providers across the prescription records.

## 03 · Data Inspection & Cleaning

The combined dataset was inspected to understand its structure, data types and missing values.

**Key observations:**
- 22 columns were initially stored as `object` data types.
- Empty values were observed in beneficiary-related fields.
- No unexpected non-numeric values were detected in the selected numerical metrics.

### Numerical Data Conversion

The following core prescription metrics were converted into appropriate numerical formats:

- `Tot_Clms`
- `Tot_30day_Fills`
- `Tot_Day_Suply`
- `Tot_Drug_Cst`

<details>
  <summary>Data quality check</summary>

  | Feature | Missing Values After Conversion |
  |:---|---:|
  | Tot_Clms | 0 |
  | Tot_30day_Fills | 0 |
  | Tot_Day_Suply | 0 |
  | Tot_Drug_Cst | 0 |

</details>

Beneficiary-related and age-specific fields were excluded from the initial feature aggregation to avoid treating unavailable or potentially suppressed values as actual zero values.

## 04 · Provider-Level Feature Engineering

**The key transformation of Day 2.**

The original dataset contained multiple rows for the same provider because each provider could prescribe multiple drug brands.

A pivot-table transformation was applied using:

| Parameter | Configuration |
|:---|:---|
| Index | `Prscrbr_NPI` |
| Columns | `Brnd_Name` |
| Values | Four core prescription metrics |
| Aggregation | Sum |
| Missing drug-metric combinations | Filled with 0 |

This transformed the dataset into a provider-level structure, with separate features for each drug.

### Engineered Features

For each of the four drugs, the following metrics were created:

- Total Claims
- Total 30-Day Fills
- Total Day Supply
- Total Drug Cost

**Example feature names:**

`Jardiance_Tot_Clms`  
`Farxiga_Tot_Clms`  
`Invokana_Tot_Clms`  
`Steglatro_Tot_Clms`

Column names were flattened to create a clean, model-friendly structure.

## 05 · Final Dataset

<details>
  <summary>Final dataset overview</summary>

  | Metric | Result |
  |:---|---:|
  | Total Providers | 101,500 |
  | Total Columns | 17 |
  | Missing Values | 0 |
  | Duplicate Rows | 0 |
  | Duplicate NPIs | 0 |

</details>

<details>
  <summary>Dataset structure</summary>

  | Component | Description |
  |:---|:---|
  | Provider Identifier | NPI |
  | Jardiance Features | Claims, fills, supply, cost |
  | Farxiga Features | Claims, fills, supply, cost |
  | Invokana Features | Claims, fills, supply, cost |
  | Steglatro Features | Claims, fills, supply, cost |

</details>

**Exported File:** `provider_level_2021.csv`

The processed dataset was successfully saved and downloaded for use in the next stage.

## 06 · Key Achievements

- [x] Retrieved competitor prescription data through the CMS API.
- [x] Integrated four SGLT2 inhibitor datasets.
- [x] Inspected data types and missing values.
- [x] Converted core numerical metrics.
- [x] Aggregated prescription records by provider.
- [x] Engineered drug-specific provider features.
- [x] Validated dataset structure and data quality.
- [x] Exported the final provider-level dataset.

## 07 · What's Next?

### Day 03 — Target Variable Construction

The next stage focuses on obtaining the corresponding 2022 prescription data and defining the prediction target.

**Planned approach:**

- Use 2021 provider-level features as model inputs (X).
- Determine 2022 Jardiance prescription performance.
- Define the high-prescriber classification target (y).
- Prepare the time-based modelling dataset.

The objective is to identify healthcare providers who are likely to become high Jardiance prescribers based on their earlier prescription patterns.

---

<div align="center">

**DAY 02 · COMPLETED**

*From raw prescription records to provider-level features.*

</div>
