# 🩺 HCP Targeting & Prescription Prediction

### Using Data Science to Identify Future High-Prescribing Healthcare Providers

<p align="center">
  <img src="https://img.shields.io/badge/Project-Data%20Science-6C63FF?style=for-the-badge" alt="Project Type"/>
  <img src="https://img.shields.io/badge/Status-In%20Progress-FFA500?style=for-the-badge" alt="Project Status"/>
  <img src="https://img.shields.io/badge/Python-Data%20Analysis-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python"/>
  <img src="https://img.shields.io/badge/Pandas-Data%20Processing-150458?style=for-the-badge&logo=pandas&logoColor=white" alt="Pandas"/>
</p>

---

## 📌 Overview

How can pharmaceutical companies identify healthcare providers who are likely to prescribe a particular medication more frequently in the future?

This project explores a **data-driven approach to Healthcare Professional (HCP) targeting** using historical prescription data.

The objective is to develop a predictive analytics workflow that can help prioritize healthcare providers based on their potential future prescribing behaviour for **Jardiance (empagliflozin)**.

Rather than relying only on historical prescription volume, the planned approach combines provider-level prescription patterns and competitor-drug information to predict future high-prescriber status.

> **Project focus:** Transform historical prescription data into actionable insights for healthcare provider prioritization.

## 🎯 Business Problem

Pharmaceutical companies have limited time and resources for engaging healthcare providers. Identifying which providers may have higher future prescribing activity can help teams prioritize their outreach more effectively.

This project aims to address three questions:

- Which historical prescribing patterns may help identify future high prescribers?
- Can competitor-drug prescription activity provide useful predictive signals?
- How can a predictive model support more informed provider prioritization?

The model is intended to support decision-making, not replace professional judgment or establish causal relationships.

## 🔬 Project Approach

The project follows a structured data science workflow:

| Stage | Approach |
|---|---|
| 01 · Data Collection | Retrieve public Medicare Part D prescriber data through the CMS API. |
| 02 · Data Exploration | Inspect data structure, data types, missing values and prescription distributions. |
| 03 · Data Preparation | Validate records and prepare consistent provider-level datasets across years. |
| 04 · Feature Engineering | Develop features from historical Jardiance and competitor-drug prescription activity. |
| 05 · Target Definition | Define high-prescriber status using future-year prescription activity. |
| 06 · Model Development | Train classification models to predict future high-prescriber status. |
| 07 · Model Evaluation | Evaluate predictive performance using appropriate classification and ranking metrics. |
| 08 · Business Insights | Translate model outputs into practical provider-prioritization recommendations. |

*The workflow above describes the planned project. Model development and evaluation are not yet complete.*

## 📊 Dataset

**Data source:** CMS — Centers for Medicare & Medicaid Services

**Dataset:** Medicare Part D Prescribers — by Provider and Drug

🔗 [Explore the official CMS dataset](https://data.cms.gov/provider-summary-by-type-of-service/medicare-part-d-prescribers/medicare-part-d-prescribers-by-provider-and-drug)

The initial extract contains Jardiance prescription records for reporting year 2021, retrieved through the CMS data API.

### Initial Data Exploration

| Check | Result |
|---|---:|
| Jardiance records retrieved | 87,157 |
| Unique provider NPIs in the extract | 87,157 |
| Exact duplicate rows | 0 |
| Missing values in selected columns | 0 |
| Minimum total claims | 11 |
| Maximum total claims | 1,734 |

These results describe the initial Jardiance extract only. The missing-value check covered selected columns, and the dataset still requires further validation before modelling.

### Key Variables

| Variable | Description |
|---|---|
| `Prscrbr_NPI` | Unique provider identifier |
| `Prscrbr_Type` | Provider type |
| `Prscrbr_City` | Provider's city |
| `Prscrbr_State_Abrvtn` | Provider's state |
| `Brnd_Name` | Drug brand name |
| `Gnrc_Name` | Generic drug name |
| `Tot_Clms` | Total prescription claims |
| `Tot_30day_Fills` | Total 30-day prescription fills |
| `Tot_Day_Suply` | Total days of medication supplied |
| `Tot_Drug_Cst` | Total drug cost |
| `Tot_Benes` | Reported beneficiary count |

## 🧠 Planned Predictive Modelling

The central idea is to use information from one year to predict prescribing behaviour in a later year.

**Historical data → Feature engineering → Classification model → Future high-prescriber prediction**

The planned modelling workflow includes:

- Defining a provider-level prediction population.
- Building historical prescribing features for Jardiance and relevant competitor drugs.
- Creating a binary target based on future-year Jardiance prescription activity.
- Using a time-based training and validation strategy to reduce temporal data leakage.
- Comparing model performance with suitable baseline approaches.

The exact target definition, feature set and model selection will be finalized after the multi-year dataset has been prepared.

## 📈 Evaluation Strategy

The planned evaluation will consider both predictive performance and the practical value of the model for prioritization.

| Metric | Purpose |
|---|---|
| Precision | Measure the proportion of predicted high prescribers who meet the target definition. |
| Precision@K | Assess the quality of the highest-ranked providers selected for prioritization. |
| Lift | Compare the high-prescriber rate in the selected group with the overall rate. |
| ROC-AUC | Measure the model's ability to distinguish between the two target classes across thresholds. |
| PR-AUC | Evaluate the precision-recall trade-off, particularly if the target classes are imbalanced. |

No model performance results are reported yet because model training and evaluation are still pending.

## 🛠️ Tech Stack

- **Python** — Data analysis and modelling
- **Pandas** — Data manipulation and preprocessing
- **NumPy** — Numerical operations
- **Matplotlib / Seaborn** — Exploratory data visualizations
- **Scikit-learn** — Planned classification modelling and evaluation
- **Google Colab / Jupyter Notebook** — Development environment
- **Git & GitHub** — Version control and project documentation

## 📁 Repository Structure

```text
hcp-targeting-prediction/
│
├── notebooks/
│   └── 01_data_exploration.ipynb
│
├── README.md
├── requirements.txt
└── .gitignore
```

The repository structure will evolve as data preparation, feature engineering and modelling are implemented.

## 🚧 Current Progress

- [x] Define the business problem and project objective
- [x] Access the CMS data API
- [x] Retrieve the initial Jardiance extract
- [x] Inspect columns and data types
- [x] Check duplicate rows and missing values in selected columns
- [x] Convert total claims to a numeric format
- [x] Examine descriptive statistics and high-claims records
- [ ] Collect competitor-drug data
- [ ] Prepare additional reporting years
- [ ] Perform comprehensive data-quality validation
- [ ] Build provider-level features
- [ ] Define the future-year prediction target
- [ ] Train and validate classification models
- [ ] Evaluate model performance
- [ ] Develop business recommendations

## ⚠️ Data Limitations & Responsible Use

- Medicare Part D data represents reported prescription activity and does not provide a complete view of every prescription or every patient.
- Prescription claims and beneficiary counts represent different measures.
- Suppressed or unavailable values must be handled carefully and should not automatically be interpreted as zero.
- Historical associations do not establish that any particular factor causes future prescribing behaviour.
- The initial extract is not sufficient on its own to train the planned future-year prediction model.
- Predictions would be analytical decision-support signals, not recommendations for individual patient treatment.

## 🌟 Project Goal

Build an end-to-end data science project that demonstrates how public healthcare data can be transformed into meaningful features, evaluated through time-aware predictive modelling, and translated into practical business insights.

The emphasis is on **data quality, sound validation, interpretable results and responsible decision-making**.

---

<p align="center">
  <b>Built with Python, curiosity, and a data-driven mindset.</b>
  <br/>
  <sub>Academic portfolio project · Work in progress</sub>
</p>
