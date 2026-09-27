# Reliable Machine Learning on Imperfect Data

### A Study of Model Robustness, Prediction Reliability, and Data Quality

Machine-learning models are commonly evaluated on relatively clean datasets, but real-world data often contains missing values, outliers, and incorrect labels. These imperfections can affect model performance in different ways and may also make individual predictions less reliable.

This project investigates **machine-learning robustness under imperfect data** and explores whether **model disagreement and prediction confidence can be used to identify potentially difficult or unreliable predictions**.

The study uses the **UCI Default of Credit Card Clients** dataset and follows a controlled experimental framework for introducing different types of data imperfections.

---

## Research Questions

The study investigates five main questions:

1. **Missing Data:** How does increasing missingness affect predictive performance across different ML models?
2. **Feature Outliers:** How sensitive are different models to increasing levels of feature outliers?
3. **Label Noise:** How does increasing label noise affect model performance and prediction reliability?
4. **Model Disagreement:** Do different models disagree more frequently on difficult-to-classify samples?
5. **Unreliable Samples:** Can model disagreement combined with prediction confidence identify potentially unreliable predictions?

---

## Experimental Framework

The research follows:

**Research Question → Hypothesis → Experiment → Result → Interpretation → Conclusion**

### Models

Three core models were used for the robustness experiments:

* Logistic Regression
* Random Forest
* Gradient Boosting

The model-disagreement experiment expanded the model pool to five:

* Logistic Regression
* Decision Tree
* Random Forest
* Gradient Boosting
* Support Vector Machine

### Evaluation Metrics

Multiple metrics were used because the dataset has an imbalanced target distribution:

* Accuracy
* Precision
* Recall
* F1-score
* ROC-AUC

Accuracy was therefore not treated as the sole measure of model performance.

---

# Experiments

## Experiment 1 — Missing Data

### Objective

Investigate how increasing feature missingness affects model performance.

### Setup

Missing values were introduced using controlled MCAR-style cell-level corruption at:

**0%, 5%, 10%, 20%, 30%**

The validation set remained clean.

### Finding

Increasing missingness generally produced gradual declines in Accuracy, Precision, and ROC-AUC.

However, Recall and F1-score showed model-dependent and non-monotonic behavior.

**Conclusion:** H1 was **partially supported**.

The effect of missing data depends on both the model and the evaluation metric.

---

## Experiment 2 — Feature Outliers

### Objective

Investigate model sensitivity to increasing numerical feature outliers.

### Setup

Synthetic IQR-based outliers were introduced at:

**0%, 1%, 5%, 10%**

Only numerical training features were contaminated.

### Finding

The observed changes were relatively small and non-monotonic.

Random Forest remained comparatively stable, while Logistic Regression and Gradient Boosting showed modest changes across the tested corruption levels.

**Conclusion:** H2 was **partially supported**.

The results characterize the specific synthetic outlier mechanism used and should not be interpreted as evidence that real-world outliers are harmless.

---

## Experiment 3 — Label Noise

### Objective

Investigate how incorrect training labels affect model performance.

### Setup

Random symmetric label flipping was introduced at:

**0%, 5%, 10%, 20%, 30%**

The validation labels remained clean.

### Finding

Increasing label noise generally reduced:

* Accuracy
* Precision
* F1-score
* ROC-AUC

The effect was particularly visible in Logistic Regression's Accuracy and Gradient Boosting's ROC-AUC.

**Conclusion:** H3 was **supported** under the experimental setup.

---

# Experiment 4 — Model Disagreement and Prediction Reliability

The final experiment moved from **dataset-level performance** to **sample-level prediction behavior**.

Five different models were trained on the clean training data and evaluated on the validation set.

For every validation sample, the predictions of all five models were compared.

### Disagreement

With five models, disagreement was measured according to the proportion of models that did not support the majority prediction.

Observed disagreement levels:

* **0.0** — complete agreement
* **0.2** — one model disagrees
* **0.4** — two models disagree

### Relationship with Prediction Error

| Disagreement | Error Rate |
| -----------: | ---------: |
|          0.0 |     14.61% |
|          0.2 |     28.38% |
|          0.4 |     39.16% |

The error rate increased by approximately **24.55 percentage points** between complete agreement and the highest observed disagreement level.

**Conclusion:** H4 was **supported**.

Higher model disagreement was associated with a higher frequency of prediction errors.

---

# Prediction Confidence

Model disagreement was complemented with prediction confidence.

For each model:

**Confidence = max(P(class 0), P(class 1))**

Mean confidence was then calculated across the five-model pool.

| Disagreement | Mean Confidence | Error Rate |
| -----------: | --------------: | ---------: |
|          0.0 |          0.7789 |     14.61% |
|          0.2 |          0.7044 |     28.38% |
|          0.4 |          0.6616 |     39.16% |

As disagreement increased:

* Mean confidence decreased.
* Prediction error increased.

This provides additional evidence that disagreement can act as a signal of prediction difficulty.

---

# Potentially Unreliable Sample Detection

A simple screening rule was defined before evaluating the detection results:

* Disagreement ≥ **0.2**
* Mean confidence < **0.70**

The rule flagged:

**355 / 4,500 validation samples = 7.89%**

### Error Rate

| Group                  | Error Rate |
| ---------------------- | ---------: |
| Not flagged            |     18.96% |
| Potentially unreliable |     43.10% |

The flagged group therefore had an error rate approximately **24.14 percentage points higher** than the unflagged group.

Among the 939 majority-prediction errors, the detector identified 153:

**Error-capture rate = 16.30%**

This demonstrates that the detector can concentrate higher-risk predictions into a relatively small subset.

However, it also produced false alarms and missed many errors.

**Conclusion:** H5 was **partially supported**.

The method is better interpreted as a **risk-screening mechanism**, not a definitive unreliable-prediction detector.

---

# Final Test-Set Evaluation

After model selection and robustness experiments, the frozen core models were evaluated on the previously untouched test set.

| Model               |   Accuracy |  Precision |     Recall |         F1 |    ROC-AUC |
| ------------------- | ---------: | ---------: | ---------: | ---------: | ---------: |
| Logistic Regression |     0.6873 |     0.3769 | **0.6315** | **0.4720** |     0.7182 |
| Random Forest       |     0.8127 |     0.6404 |     0.3504 |     0.4530 |     0.7686 |
| Gradient Boosting   | **0.8160** | **0.6556** |     0.3554 |     0.4609 | **0.7811** |

The models exhibit different Precision–Recall trade-offs.

This reinforces the importance of using multiple evaluation metrics rather than relying only on Accuracy.

---

# Key Findings

The study produced several broader observations:

### 1. Robustness is corruption-specific

Missing data, feature outliers, and label noise produced different effects on model performance.

### 2. Robustness is metric-dependent

Accuracy, Precision, Recall, F1-score, and ROC-AUC can respond differently to the same corruption.

### 3. Models respond differently to the same imperfection

No universal model-robustness ordering was established. Model sensitivity depended on both the corruption type and evaluation metric.

### 4. Label noise produced the clearest degradation

Among the tested corruption mechanisms, increasing random label noise produced the most consistent decline across several metrics.

### 5. Model disagreement is informative

Samples with greater disagreement among models had substantially higher prediction-error rates.

### 6. Confidence complements disagreement

Higher disagreement was associated with lower mean model confidence.

### 7. Reliability can be investigated at the sample level

Combining disagreement and confidence identified a small subset of validation samples with substantially elevated prediction risk.

---

# Hypothesis Summary

| Hypothesis                                                                     | Outcome             |
| ------------------------------------------------------------------------------ | ------------------- |
| H1 — Missingness generally reduces performance                                 | Partially supported |
| H2 — Outliers negatively affect some models more than others                   | Partially supported |
| H3 — Label noise reduces performance and reliability                           | Supported           |
| H4 — Model disagreement is associated with difficult samples                   | Supported           |
| H5 — Disagreement + low confidence can identify potentially unreliable samples | Partially supported |

---

# Dataset

**UCI Default of Credit Card Clients**

The dataset contains information about credit-card clients and whether they defaulted on their payment in the following month.

Dataset characteristics:

* 30,000 observations
* 23 explanatory features
* Binary target
* Significant class imbalance
* Numerical, categorical, and ordinal variables

The dataset is available through the UCI Machine Learning Repository.

---

# Methodology

The overall pipeline was:

```text
Raw Dataset
     ↓
Data Understanding & EDA
     ↓
Train / Validation / Test Split
     ↓
Preprocessing
     ↓
Clean Baseline Models
     ↓
Hyperparameter Tuning
     ↓
Frozen Model Configurations
     ↓
─────────────────────────────────
│       Robustness Experiments   │
│                                │
│  Missing Data                  │
│  Feature Outliers              │
│  Label Noise                   │
─────────────────────────────────
     ↓
Model Disagreement Analysis
     ↓
Prediction Confidence
     ↓
Potentially Unreliable Samples
     ↓
Final Test Evaluation
     ↓
Research Findings
```

---

# Reproducibility

A multi-seed reproducibility check was performed for selected corruption levels using seeds:

* 42
* 123
* 2026

The general performance patterns remained broadly stable across the tested seeds, although some numerical variation was observed.

This suggests that the main observations are not entirely dependent on a single corruption realization.

---

# Limitations

The study has several limitations:

* Experiments were conducted on a single dataset.
* Corruption mechanisms were synthetic.
* The main experiments used limited random seeds.
* Only a limited model pool was evaluated.
* The outlier mechanism modified individual numerical cells independently.
* Label noise was introduced through random symmetric flipping.
* Reliability thresholds were heuristic.
* No independent ground-truth reliability label was available.
* The reliability detector was evaluated on the validation set rather than an independent reliability holdout.

Therefore, the findings should be interpreted within the experimental setup rather than as universal claims about machine-learning robustness.

---

# Future Work

Potential extensions include:

* Evaluating the framework on multiple datasets and domains.
* Comparing MCAR, MAR, and MNAR missingness.
* Introducing structured and multivariate outliers.
* Studying systematic and class-dependent label noise.
* Expanding model diversity with XGBoost, LightGBM, and neural networks.
* Calibrating prediction probabilities.
* Developing learned risk scores instead of fixed thresholds.
* Evaluating selective prediction and abstention.
* Testing reliability detection under corrupted validation data.
* Performing large-scale multi-seed experiments with confidence intervals and statistical analysis.

A particularly interesting direction is:

```text
Data Imperfection
       ↓
Model Disagreement
       ↓
Prediction Confidence
       ↓
Prediction Risk
       ↓
Human Review / Abstention
```

This moves the research question from simply asking **"How accurate is the model?"** toward investigating **when a model's prediction should be trusted or reviewed further**.

---

# Project Structure

```text
reliable-ml-imperfect-data/
│
├── README.md
│
├── notebooks/
│   └── Reliable_ML_on_Imperfect_Data.ipynb
│
├── data/
│   └── README.md
│
├── src/
│
├── results/
│   ├── figures/
│   └── tables/
│
├── report/
│   └── research_report.pdf
│
└── requirements.txt
```

---

# Technologies

* Python
* Pandas
* NumPy
* Matplotlib
* Scikit-learn
* Google Colab
* Jupyter Notebook

---

# Research Focus

**Machine Learning · Robustness · Imperfect Data · Model Reliability · Prediction Uncertainty · Model Disagreement · Data Quality**

---

## Author

**Divyansh Singh**

B.Tech — Computer Science & Engineering
UIET, CSJMU, Kanpur

BS/Diploma — Data Science and Applications
IIT Madras

---

> This project is an experimental study of machine-learning robustness and prediction reliability under controlled data imperfections. The results are specific to the dataset, models, corruption mechanisms, and experimental settings described in the study.
