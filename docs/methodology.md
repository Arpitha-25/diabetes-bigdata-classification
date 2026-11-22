# Project Methodology

## 1. Introduction

The project develops a Big Data-driven explainable hybrid machine learning and deep learning framework for diabetes classification using BRFSS 2024 and 2025 survey datasets.

The methodology combines distributed data processing concepts, machine learning, deep learning, ensemble learning and explainable AI.

The complete implementation is organised into data acquisition, preprocessing, model development, hybrid stacking, evaluation and explainability stages.

---

## 2. Dataset Acquisition

The datasets are obtained from the Centers for Disease Control and Prevention (CDC) Behavioral Risk Factor Surveillance System (BRFSS).

The annual datasets used are:

- BRFSS 2024
- BRFSS 2025

The original files are provided in XPT format.

### Dataset size

| Dataset | Number of records |
|---|---:|
| BRFSS 2024 | 457,670 |
| BRFSS 2025 | 356,158 |
| Combined raw dataset | 813,828 |
| Final analytical dataset | 784,473 |

The two annual datasets are combined after inspecting differences in their schemas and variable definitions.

---

## 3. Big Data Processing Using Apache Spark

Apache Spark is used as the primary data processing engine.

PySpark DataFrames and Spark SQL support the processing of the large annual survey datasets.

### Processing steps

1. Load the annual XPT files.
2. Inspect dataset schemas.
3. Identify common and year-specific variables.
4. Harmonise the datasets.
5. Apply target eligibility rules.
6. Perform data cleaning and validation.
7. Prepare the consolidated analytical dataset.
8. Persist the harmonised data in Parquet format.

### Big Data characteristics

**Volume:** The combined annual datasets contain 813,828 raw records, with 784,473 records retained for the final analytical dataset.

**Variety:** The dataset contains demographic, socioeconomic, behavioural and health-related variables with different data types and annual schema variations.

**Veracity:** Data validation, missing-value checks, response-code handling and target eligibility filtering are applied to improve data quality.

### Processing environment

- Apache Spark / PySpark 3.5.9
- Google Colab
- Google Drive
- Spark execution mode: `local[*]`

The implementation uses Spark on a single Colab machine. It does not represent a multi-node distributed cluster experiment.

---

## 4. Target Variable Construction

The target variable is derived from the BRFSS variable `DIABETE4`.

The project uses the following response coding:

- Code 1: Diabetes-positive class.
- Code 3: Diabetes-negative class.

Other responses, including pregnancy-only, prediabetes/borderline, refused, don't know and missing responses, are excluded according to the documented eligibility rules.

The objective is to classify reported diabetes status rather than predict future disease onset.

---

## 5. Data Preprocessing

The analytical dataset contains demographic, socioeconomic, behavioural and health-condition predictors.

### Feature categories

**Categorical variables:**
- Age group
- Sex
- Education
- Employment
- Income
- Marital status
- Smoking status
- Checkup

**Continuous variables:**
- BMI
- Physical unhealthy days
- Mental unhealthy days

**Binary variables:**
- Exercise
- Alcohol use
- Asthma
- Kidney disease
- Heart attack
- Coronary heart disease
- Stroke
- Arthritis
- Depression
- Medical cost barrier
- Difficulty walking
- Difficulty dressing
- Difficulty running errands

### Preprocessing techniques

- Numerical missing values are imputed using median imputation.
- Numerical features are standardised using StandardScaler.
- Categorical missing values are imputed using the most frequent value.
- Categorical features are encoded using OneHotEncoder.
- Unknown categorical values are handled using `handle_unknown="ignore"`.

The resulting transformed feature representation contains 72 features.

Survey design metadata are retained separately and are not automatically used as predictive features.

---

## 6. Dataset Partitioning

The dataset is divided into training, validation and testing partitions.

| Partition | Records |
|---|---:|
| Training | 549,129 |
| Validation | 117,671 |
| Testing | 117,673 |

The split is stratified by survey year and target class using random seed 42.

This supports reproducibility and maintains representation across the partitions.

---

## 7. Machine Learning Models

Five individual machine learning and deep learning approaches are implemented alongside the hybrid stacking model.

### 7.1 Logistic Regression

Logistic Regression is used as a baseline linear classification algorithm.

It estimates the probability of the positive class using a logistic function.

### 7.2 Random Forest

Random Forest is an ensemble learning method that combines predictions from multiple decision trees.

It captures nonlinear relationships and interactions between input variables.

### 7.3 Spark Gradient-Boosted Trees

Spark MLlib Gradient-Boosted Trees are used as a Big Data-oriented tree ensemble baseline.

The model is trained using the Spark machine learning pipeline.

### 7.4 XGBoost

XGBoost is a gradient boosting algorithm that builds decision trees sequentially to improve classification performance.

It is also used as one of the base learners in the hybrid framework.

### 7.5 Deep Neural Network

A Deep Neural Network is implemented using PyTorch.

The network uses multiple fully connected layers with batch normalisation, ReLU activation and dropout.

The architecture contains hidden layers of 256, 128 and 64 units.

---

## 8. Proposed Hybrid Stacking Framework

The proposed hybrid model combines XGBoost and Deep Neural Network predictions through a Logistic Regression meta-learner.

### Architecture

```text
              Input Features
                    |
          Preprocessing Pipeline
                    |
           -------------------
           |                 |
        XGBoost             DNN
           |                 |
           -------------------
                    |
          Out-of-Fold Predictions
                    |
          Logistic Regression
             Meta-Learner
                    |
            Final Prediction
```

### Hybrid implementation

1. Divide the training data into three folds.
2. Train the base learners using fold-specific training partitions.
3. Generate out-of-fold predictions for the held-out folds.
4. Combine the OOF predictions.
5. Train the Logistic Regression meta-learner using these predictions.
6. Fit the final base models using the complete training data.
7. Generate base-model predictions for validation and test data.
8. Pass the predictions to the trained meta-learner.
9. Generate final hybrid classification probabilities.

Fold-specific preprocessing is used to avoid information leakage during OOF generation.

---

## 9. Model Evaluation

The classification models are evaluated using multiple metrics.

### Evaluation metrics

**ROC-AUC:** Measures the ability of the classifier to distinguish between the two classes across thresholds.

**PR-AUC:** Evaluates the precision-recall trade-off, particularly useful when the target classes are imbalanced.

**Accuracy:** Measures the proportion of correctly classified observations.

**Precision:** Measures the proportion of predicted positive cases that are actually positive.

**Recall:** Measures the proportion of actual positive cases correctly identified.

**Specificity:** Measures the proportion of actual negative cases correctly identified.

**F1-score:** Harmonic mean of precision and recall.

---

## 10. Explainable AI Using SHAP

SHAP is used to understand the contribution of input features to the predictions of the XGBoost component.

The analysis includes:

- SHAP summary plots.
- Global feature importance.
- Grouped SHAP feature importance.
- BMI dependence analysis.
- Individual prediction explanations.

Grouped SHAP importance is calculated by aggregating contributions from transformed one-hot encoded features back to their original variables.

SHAP values explain model behaviour and should not be interpreted as causal effects.

The SHAP analysis applies to the XGBoost component, not directly to the final hybrid stacking model.

---

## 11. Fairness and Subgroup Analysis

The framework examines model performance across demographic subgroups.

The analysis includes:

- Age-group performance.
- Sex-group performance.
- Intersectional subgroup performance.
- Recall comparisons.
- Confidence interval analysis.

Wilson confidence intervals are used for respondent-level subgroup proportions.

Subgroup results are interpreted cautiously, particularly when the number of positive observations is small.

The analysis is diagnostic and does not independently establish the presence or absence of algorithmic bias.

---

## 12. Calibration Analysis

Calibration evaluates whether predicted probabilities correspond to observed positive-class frequencies.

The following metrics are used:

- Brier Score
- Log Loss
- Expected Calibration Error (ECE)

A calibration curve is also generated to compare predicted probabilities with observed outcome frequencies.

---

## 13. Temporal Validation

A separate temporal experiment is conducted using 2024 data for training and 2025 data for holdout evaluation.

This experiment examines whether model discrimination remains broadly consistent across the two survey years.

The temporal experiment is conducted separately for XGBoost and DNN.

Since only two annual datasets are available, the results should not be interpreted as evidence of long-term generalisation.

---

## 14. Scalability Analysis

A controlled aggregation benchmark compares Spark and Pandas across different dataset fractions.

The benchmark uses:

- 25% of the dataset.
- 50% of the dataset.
- 75% of the dataset.
- 100% of the dataset.

Execution times are recorded and compared.

The benchmark is performed on a single Colab machine. It therefore measures local execution behaviour and does not establish distributed cluster scalability.

---

## 15. Statistical Comparison

Paired respondent-level bootstrap comparisons are performed using 2,000 bootstrap replicates.

The comparisons examine differences in ROC-AUC and PR-AUC between:

- DNN and XGBoost.
- Hybrid stacking and XGBoost.
- Hybrid stacking and DNN.

Confidence intervals and p-values are used to assess the observed differences.

These are respondent-level bootstrap estimates, not survey-design-adjusted significance tests.

---

## 16. Survey-Weighted Evaluation

Survey-weighted metrics are calculated to account for the BRFSS survey weighting information.

The evaluation includes:

- Weighted ROC-AUC.
- Weighted PR-AUC.
- Weighted accuracy.
- Weighted precision.
- Weighted recall.
- Weighted specificity.
- Weighted F1-score.
- Weighted prevalence.

A PSU-based bootstrap procedure is used to estimate uncertainty.

The implementation is a simplified design-aware approximation and should not be interpreted as an official CDC survey analysis.

---

## 17. Responsible Use and Limitations

The project uses publicly available, self-reported survey data.

Important limitations include:

- Self-reported responses may contain measurement and reporting errors.
- BRFSS is repeated cross-sectional rather than longitudinal.
- The model classifies reported diabetes status rather than predicting future onset.
- SHAP explanations do not establish causality.
- Subgroup results may be uncertain for small sample sizes.
- Survey-weighted variance estimates depend on the approximations documented in the implementation.
- Spark was executed locally on a single machine.

The framework is intended for academic research and methodological evaluation, not individual clinical diagnosis or treatment decisions.

---

## 18. Conclusion

The methodology integrates Apache Spark-based data engineering, comparative machine learning, deep learning, hybrid stacking and explainable AI.

The evaluation combines predictive performance with subgroup analysis, calibration, temporal validation, scalability benchmarking and survey-weighted metrics.

This provides a comprehensive experimental framework for studying diabetes classification using large-scale public health survey data.
