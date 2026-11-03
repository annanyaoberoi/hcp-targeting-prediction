# 🧠 Day 5 — Model Training & Evaluation

### HCP Targeting & Prescription Prediction

<p align="center">
  <strong>From prepared data to predictive machine learning models</strong>
</p>

---

## 📌 1. Objective

The objective of Day 5 was to train and evaluate machine learning classification models to predict whether a Healthcare Provider (HCP) would become a **high Jardiance prescriber in 2022**, using prescription behaviour from 2021.

We experimented with three classification algorithms and compared their performance to identify the strongest candidate for our HCP targeting use case.

## 🎯 2. Understanding the ML Problem

This project uses **Supervised Machine Learning — Binary Classification**.

- **Input (X):** 21 features representing HCP prescription behaviour in 2021.
- **Target (y):** `high_prescriber_2022`
- **Class 1:** High Jardiance prescriber.
- **Class 0:** Not a high Jardiance prescriber.

The target was constructed using the 80th percentile of 2022 Jardiance claims.

| Target class | Providers | Distribution |
|---|---:|---:|
| 0 — Not high prescriber | 80,870 | 79.67% |
| 1 — High prescriber | 20,630 | 20.33% |
| **Total** | **101,500** | **100%** |

## ⚙️ 3. Training and Testing Data

We divided the prepared dataset into training and testing sets using an 80:20 split.

| Dataset | Records | Features |
|---|---:|---:|
| Training set | 81,200 | 21 |
| Testing set | 20,300 | 21 |

**Method used:** `train_test_split`

Configuration:
- Test size: 20%
- Random state: 42
- Stratification: Enabled

Stratification maintains approximately the same target-class distribution in both datasets.

## 🤖 4. Machine Learning Algorithms

We trained three classification algorithms.

### 4.1 Logistic Regression

Logistic Regression is a classification algorithm that estimates the probability of an observation belonging to a particular class.

**Why we used it:**
- Provides a simple baseline model.
- Predicts the probability of an HCP being a high prescriber.
- Helps establish a benchmark for more complex models.

**Preprocessing:** StandardScaler was applied through a Pipeline because Logistic Regression benefits from features being on comparable scales.

### 4.2 Random Forest Classifier

Random Forest is an ensemble learning algorithm that combines predictions from multiple decision trees.

Each tree learns patterns from the training data, and their combined predictions produce the final classification.

**Why we used it:**
- Can capture nonlinear relationships.
- Handles interactions between features.
- Combines multiple decision trees to make predictions.

**Configuration:**
- Number of trees: 200
- Class weight: Balanced
- Random state: 42

### 4.3 Gradient Boosting Classifier

Gradient Boosting is an ensemble learning algorithm that builds decision trees sequentially. Each new tree attempts to improve the errors made by the existing ensemble.

**Why we used it:**
- Captures complex patterns in prescription behaviour.
- Can model nonlinear relationships.
- Combines multiple weak learners into a stronger predictive model.

**Configuration:**
- Number of estimators: 100
- Learning rate: 0.1
- Maximum tree depth: 3
- Random state: 42

---

## 📊 5. Model Evaluation Metrics

Since our dataset contains approximately 80% negative and 20% positive cases, accuracy alone is not sufficient to evaluate model performance.

We used six evaluation metrics:

| Metric | Purpose |
|---|---|
| Accuracy | Overall proportion of correct predictions |
| Precision | How many predicted high prescribers were actually high prescribers |
| Recall | How many actual high prescribers the model identified |
| F1 Score | Harmonic balance between precision and recall |
| ROC-AUC | Measures how well the model separates the two classes across thresholds |
| PR-AUC | Evaluates the precision-recall trade-off, particularly useful with class imbalance |

## 📈 6. Model Performance Comparison

All three models were evaluated on the same test dataset.

| Evaluation Metric | Logistic Regression | Random Forest | Gradient Boosting |
|---|---:|---:|---:|
| Accuracy | 90.00% | 88.82% | **90.05%** |
| Precision | **84.67%** | 76.66% | 82.03% |
| Recall | 62.00% | 64.71% | **65.37%** |
| F1 Score | 71.58% | 70.18% | **72.75%** |
| ROC-AUC | 92.01% | 90.10% | **92.13%** |
| PR-AUC | **81.97%** | 78.14% | 81.85% |

### Key Observations

**Logistic Regression**
- Achieved the highest Precision (84.67%).
- Achieved the highest PR-AUC (81.97%).
- Provided a strong baseline.

**Random Forest**
- Achieved a Recall of 64.71%.
- Had lower overall performance than the other two models on most evaluation metrics.

**Gradient Boosting**
- Achieved the highest Accuracy (90.05%).
- Achieved the highest Recall (65.37%).
- Achieved the highest F1 Score (72.75%).
- Achieved the highest ROC-AUC (92.13%).

---

## 🏆 7. Provisional Model Selection

### Gradient Boosting Classifier

Based on the current test results, Gradient Boosting is our provisional leading model.

**Reasons for selecting it:**

1. Highest F1 Score among the three models.
2. Highest Recall, identifying the largest proportion of actual high prescribers.
3. Highest ROC-AUC.
4. Highest overall Accuracy.

For an HCP targeting application, identifying more potentially valuable providers is important. Gradient Boosting currently offers the strongest overall balance according to F1 Score.

However, Logistic Regression remains a strong alternative because of its higher Precision and PR-AUC.

Final selection should also consider the business cost of false positives versus false negatives.

## 💡 8. Business Interpretation

The model comparison helps demonstrate how machine learning can support pharmaceutical targeting decisions.

Instead of treating every HCP equally, a predictive model can help prioritize providers based on their likelihood of becoming high Jardiance prescribers.

Potential business applications include:

- Prioritizing HCPs for engagement.
- Supporting more focused targeting strategies.
- Identifying providers with promising prescription patterns.
- Helping allocate limited sales and marketing resources.

The model provides predictions to support decisions; it does not replace business judgement.

## 🧪 9. Tools and Libraries Used

| Tool / Library | Purpose |
|---|---|
| Python | Programming language |
| Pandas | Data manipulation and comparison |
| Scikit-learn | Model training and evaluation |
| StandardScaler | Feature scaling |
| LogisticRegression | Baseline classification |
| RandomForestClassifier | Ensemble classification |
| GradientBoostingClassifier | Boosting-based classification |
| Google Colab | Development environment |

## 📁 10. Output Generated

The following output was saved during Day 5:

- `model_comparison.csv` — comparison of all three models using six evaluation metrics.

This file contains the model performance results and can be used in the final project documentation.

## ⚠️ 11. Important Considerations

- The models use 2021 prescription features to predict 2022 outcomes.
- The 2022 outcome columns are excluded from the input feature matrix to prevent target leakage.
- The test set was kept separate during model fitting and evaluation.
- The dataset has class imbalance, so multiple evaluation metrics were considered.
- The current evaluation uses a random stratified holdout split, not a separate future-year temporal validation.
- Medicare Part D data represents reported Medicare Part D prescribing activity, not every prescription written by an HCP.

---

## ✅ Day 5 Deliverables

- [x] Understand supervised binary classification.
- [x] Prepare training and testing datasets.
- [x] Train Logistic Regression.
- [x] Train Random Forest Classifier.
- [x] Train Gradient Boosting Classifier.
- [x] Generate test-set predictions.
- [x] Evaluate all three models.
- [x] Compare model performance.
- [x] Identify a provisional leading model.
- [x] Export model comparison results.

### 🚀 Next: Day 6 — Model Finalization & Feature Importance

The next stage will focus on finalizing the predictive model, understanding which features influence predictions, and testing the model on sample HCP records.

---

<p align="center">
  <strong>HCP Targeting & Prescription Prediction</strong><br/>
  <sub>Day 5 — Model Training & Evaluation | Python · Scikit-learn · Machine Learning</sub>
</p>
