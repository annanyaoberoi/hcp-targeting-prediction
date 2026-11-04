# 🚀 Day 6 — Model Finalization & HCP Prediction

<p align="center">
  <strong>HCP Targeting & Prescription Prediction</strong><br/>
  <em>Turning Machine Learning Predictions into Actionable HCP Insights</em>
</p>

---

## 1. Final Model Selection

After training and evaluating three classification algorithms, Gradient Boosting was selected as the final model.

**Selected Model:** `GradientBoostingClassifier`

The model was selected because it achieved the highest F1 Score, Recall, ROC-AUC and Accuracy among the three evaluated models.

### Model Performance Comparison

| Evaluation Metric | Logistic Regression | Random Forest | Gradient Boosting |
|---|---:|---:|---:|
| Accuracy | 90.00% | 88.82% | **90.05%** |
| Precision | **84.67%** | 76.66% | 82.03% |
| Recall | 62.00% | 64.71% | **65.37%** |
| F1 Score | 71.58% | 70.18% | **72.75%** |
| ROC-AUC | 92.01% | 90.10% | **92.13%** |
| PR-AUC | **81.97%** | 78.14% | 81.85% |

**Final Model: Gradient Boosting**

Gradient Boosting provided the strongest overall balance between precision and recall, making it the preferred model for identifying potential high prescribers.

---

## 2. Feature Importance Analysis

Feature importance was calculated using the trained Gradient Boosting model's `feature_importances_` attribute.

The purpose was to understand which 2021 prescribing features contributed most to the model's predictions.

### Top 6 Important Features

| Feature | Importance |
|---|---:|
| Jardiance_Tot_Clms | 94.21% |
| Jardiance_Tot_30day_Fills | 2.37% |
| total_sglt2_claims_2021 | 1.87% |
| Jardiance_Tot_Day_Suply | 0.66% |
| total_sglt2_drug_cost_2021 | 0.35% |
| Jardiance_Tot_Drug_Cst | 0.20% |

### Key Observation

Historical Jardiance prescription claims were the most influential feature, contributing approximately 94.21% of the model's total impurity-based feature importance.

This suggests that previous Jardiance prescribing activity is a strong predictive signal for future high-prescriber classification.

**Note:** Feature importance represents relative model contribution, not causation or a percentage of real-world behaviour explained. Correlated features can also influence how importance is distributed.

---

## 3. HCP-Level Predictions

After finalizing the model, predictions were generated for unseen test records.

Two prediction methods were used:

- `predict()` — predicts the final class (0 or 1).
- `predict_proba()` — estimates the probability of belonging to each class.

For this project, the probability of class 1 was extracted using:

```python
sample_probabilities = final_model.predict_proba(sample_hcps)[:, 1]
```

The default classification threshold is 0.5.

- Class 0: Not classified as a high prescriber.
- Class 1: Classified as a high prescriber.

### Sample Prediction Results

| NPI | Predicted Class | Probability |
|---|---:|---:|
| 1104972751 | 0 | 8.72% |
| 1467701847 | 0 | 1.67% |
| 1366685828 | 0 | 21.99% |
| 1023288990 | 0 | 1.30% |
| 1124273313 | 0 | 18.47% |
| 1255714069 | 1 | 95.16% |

The model identified one high-probability provider in this sample with an estimated probability of 95.16%.

These probabilities are model estimates and do not guarantee future prescribing behaviour.

---

## 4. HCP Ranking and Prioritization

To make predictions more useful from a business perspective, healthcare providers were ranked according to their predicted high-prescriber probabilities.

The ranking was performed in descending order of probability.

```python
ranked_hcps = hcp_prediction_df.sort_values(
    by="High_Prescriber_Probability",
    ascending=False
).reset_index(drop=True)

ranked_hcps.insert(
    0, "Rank", range(1, len(ranked_hcps) + 1)
)
```

### Top Ranked HCPs

| Rank | NPI | Predicted Class | Probability |
|---:|---|---:|---:|
| 1 | 1255714069 | 1 | 95.16% |
| 2 | 1366685828 | 0 | 21.99% |
| 3 | 1124273313 | 0 | 18.47% |
| 4 | 1104972751 | 0 | 8.72% |
| 5 | 1366596405 | 0 | 4.70% |

**Business relevance:** Ranking allows a pharmaceutical company to prioritize providers with higher predicted probabilities for further analysis and potential engagement.

Ranking is relative: a provider can have a higher rank than another provider while still having a predicted class of 0.

---

## 5. Model Persistence Using Joblib

The final trained model was saved using Joblib so that it could be reused without training it again.

```python
import joblib

joblib.dump(
    final_model,
    "hcp_gradient_boosting_model.pkl"
)
```

### Loading the Saved Model

```python
loaded_model = joblib.load(
    "hcp_gradient_boosting_model.pkl"
)

test_predictions = loaded_model.predict(X_test.head())

print("Loaded model predictions:", test_predictions)
```

**Verification Output:**

```text
Loaded model predictions: [0 0 0 0 0]
```

The saved model was successfully loaded and generated predictions, confirming that model persistence worked.

---

## 6. Final Project Validation

The final sanity check confirmed that the complete modelling pipeline was functioning correctly.

| Validation Check | Result |
|---|---:|
| Total providers | 101,500 |
| Original features | 16 |
| Engineered features | 21 |
| Training records | 81,200 |
| Testing records | 20,300 |
| Class 0 | 80,870 |
| Class 1 | 20,630 |
| Missing values in modelling features | 0 |
| Final algorithm | Gradient Boosting |
| Model save/load verification | Successful |

---

## 7. Final Project Deliverables

The following output files were generated:

| File | Description |
|---|---|
| `model_comparison.csv` | Comparison of three classification models |
| `feature_importance.csv` | Feature importance scores |
| `sample_hcp_predictions.csv` | Sample provider-level predictions |
| `ranked_hcp_predictions.csv` | Providers ranked by predicted probability |
| `hcp_gradient_boosting_model.pkl` | Saved final machine learning model |
| `hcp_targeting_model_dataset.csv` | Final modelling dataset |

---

## 8. Conclusion

The project successfully developed a machine learning pipeline to predict potential high Jardiance prescribers using historical Medicare Part D prescription data.

By combining historical prescribing patterns, competitor drug activity, feature engineering and classification algorithms, the project demonstrates how predictive analytics can support data-driven healthcare provider targeting.

The final Gradient Boosting model achieved:

- **90.05% Accuracy**
- **72.75% F1 Score**
- **92.13% ROC-AUC**
- **81.85% PR-AUC**

The project also generates provider-level predictions, probability scores and rankings that can support more focused HCP prioritization.

### Limitations & Future Improvements

- The dataset represents Medicare Part D prescription activity, not all prescriptions or all healthcare providers.
- The target threshold is based on the 80th percentile of the selected 2021 provider cohort.
- The current evaluation uses a random provider-level holdout rather than an independent temporal test on a later year.
- Further validation across additional years and provider cohorts is needed.
- Future work could include hyperparameter tuning, probability calibration, explainability using SHAP and a deployment-ready prediction dashboard.

---

<p align="center">
  <strong>DAY 6 COMPLETED</strong><br/>
  <em>From historical prescription data to predictive HCP prioritization.</em>
</p>
