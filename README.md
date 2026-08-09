# EEG Abnormality Prediction in Elderly Patients

An end-to-end, explainable machine-learning project for predicting whether an elderly patient's EEG finding is normal or abnormal using routinely collected demographic, clinical, seizure, comorbidity, and laboratory variables.

The project demonstrates how a small, imbalanced clinical dataset can be handled with leakage-aware preprocessing, model-specific feature selection, stratified validation, threshold tuning, and a multi-method explainability framework. It is a research workflow, not a deployed diagnostic system.

## Project at a glance

| Item | Description |
|---|---|
| Clinical setting | Elderly patients who underwent EEG at Hospital Canselor Tuanku Muhriz (HCTM) |
| Dataset | 288 retrospective patient records with 50 raw columns |
| Target | `EEG_ABNORMAL`: normal EEG = 0, any abnormal EEG finding = 1 |
| Class distribution | 211 abnormal cases (73.3%) and 77 normal cases (26.7%) |
| Modelling task | Imbalanced binary classification using tabular clinical data |
| Compared models | Logistic Regression, Linear SVM, Random Forest, and CatBoost |
| Final model | CatBoost with four BorutaSHAP-accepted predictors |
| Validation | Stratified train/test split, five-fold stratified cross-validation, and independent held-out evaluation |
| Explainability | Standard SHAP, Conditional SHAP, EBM, and exploratory DML |
| Status | Completed research analysis; external validation is still required |

This project predicts an EEG result from clinical variables. It does not classify raw EEG waveforms or replace neurological interpretation.

## Objective

The primary objective was to build a compact model that could:

- detect abnormal EEG findings with strong sensitivity;
- identify a small and clinically discussable predictor set;
- avoid variables likely to leak information from diagnosis or the subsequent hospital course;
- test whether model explanations remain stable when feature dependence is considered;
- separate predictive importance from exploratory adjusted association;
- document uncertain variables through sensitivity analysis rather than silently including them in the main model.

## Analytical workflow

### 1. Leakage-aware data preparation

The data was split before fitting preprocessing, feature selection, model tuning, or threshold selection. The held-out test set was reserved for final evaluation.

The workflow includes:

- training-derived numerical imputation;
- explicit handling of unknown or not-documented clinical categories;
- clinically cautious treatment of outliers rather than automatic deletion;
- train-derived capping and transformations where required;
- model-specific encoding and scaling;
- exclusion of clear target, diagnostic-pathway, and post-outcome variables from the main model.

Length of hospitalisation was excluded from the main model because it is generally known after the relevant prediction point. It was assessed separately as a leakage sensitivity analysis.

### 2. Class-imbalance controls

The abnormal EEG class represented 73.3% of the cohort. Three complementary controls were used:

- a stratified train/test split;
- five-fold stratified cross-validation during model benchmarking and tuning;
- balanced class weights for Logistic Regression, Linear SVM, Random Forest, and CatBoost.

The final decision threshold was also selected from training out-of-fold predictions. The held-out test set was not used to choose the threshold.

### 3. Model-specific feature selection

Each model used a feature-selection method suited to its modelling family.

| Model | Feature-selection method | Selected terms/features |
|---|---|---:|
| Logistic Regression | L1-regularised Logistic Regression CV | 51 |
| Linear SVM | RFECV with a linear SVM | 69 |
| Random Forest | Boruta | 3 |
| CatBoost | BorutaSHAP with CatBoost | 4 accepted, 1 tentative |

The selected CatBoost predictors were:

1. `ALBUMIN`
2. `SEIZURES`
3. `BLOOD UREA`
4. `GENERALISED SEIZURE`

`SODIUM` was marked tentative by BorutaSHAP. It was therefore excluded from the main model and evaluated in a separate sensitivity analysis.

### 4. Model benchmarking

Training-set out-of-fold results from five-fold stratified cross-validation were:

| Model | F1 | Recall | Specificity | Balanced accuracy | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Linear SVM | 0.862 | 0.976 | 0.197 | 0.587 | 0.757 |
| Random Forest | 0.840 | 0.899 | 0.328 | 0.614 | 0.712 |
| CatBoost | 0.826 | 0.799 | 0.623 | 0.711 | 0.784 |
| Logistic Regression | 0.822 | 0.781 | 0.672 | 0.727 | 0.775 |

Linear SVM achieved the highest cross-validated F1 and recall, but at the cost of very low specificity. CatBoost was retained for the final research model because it produced a compact all-relevant feature set, maintained a more balanced error profile, supported native mixed-data modelling, and aligned with the project's interpretability objective.

## Final CatBoost model

The final CatBoost configuration was selected using training out-of-fold predictions:

| Parameter | Value |
|---|---:|
| Depth | 3 |
| Iterations | 250 |
| Learning rate | 0.03 |
| L2 leaf regularisation | 3 |
| Decision threshold | 0.29 |

### Held-out test performance

| Metric | Result |
|---|---:|
| F1 | 0.854 |
| Recall / sensitivity | 0.905 |
| Precision | 0.809 |
| Specificity | 0.438 |
| Balanced accuracy | 0.671 |
| Accuracy | 0.776 |
| ROC-AUC | 0.810 |

Held-out confusion matrix:

- true negatives: 7;
- false positives: 9;
- false negatives: 4;
- true positives: 38.

The model detected most abnormal EEG cases, but its modest specificity means that it generated a meaningful number of false-positive abnormal predictions. This trade-off must be considered before any prospective or clinical evaluation.

## Explainable AI framework

The final model was examined using multiple complementary methods. Each method answers a different question; agreement is treated as robustness or triangulation, not proof of causality.

### Standard SHAP

Standard SHAP explained the frozen CatBoost model on the 58 held-out observations.

| Rank | Feature | Mean absolute SHAP |
|---:|---|---:|
| 1 | Albumin | 0.879 |
| 2 | Seizures | 0.568 |
| 3 | Blood Urea | 0.429 |
| 4 | Generalised Seizure | 0.357 |

The analysis includes global importance, beeswarm, dependence, and predefined non-identifying local waterfall outputs.

### Conditional SHAP

Dependence-aware Conditional SHAP was implemented through:

```text
Python pyshapr -> R shapr -> conditional inference trees (CTREE)
```

The method explained the same model, observations, features, order, and raw-margin output scale as Standard SHAP. Conditional distributions and categorical levels were learned from training data only.

| Rank | Feature | Mean absolute Conditional SHAP |
|---:|---|---:|
| 1 | Albumin | 0.807 |
| 2 | Blood Urea | 0.550 |
| 3 | Seizures | 0.521 |
| 4 | Generalised Seizure | 0.497 |

Validation results:

- maximum additivity residual: `2.61e-08`;
- Spearman rank correlation with Standard SHAP: 0.80;
- Kendall rank correlation: 0.667;
- top-three feature overlap: 100%.

The main explanation hierarchy was therefore stable. Albumin remained first, Blood Urea and Seizures exchanged second and third place, and Generalised Seizure remained fourth.

### Explainable Boosting Machine

An Explainable Boosting Machine was fitted as a separate transparent additive model for feature-shape triangulation. Its held-out sanity evaluation achieved:

- ROC-AUC: 0.804;
- balanced accuracy: 0.751.

The EBM and Standard SHAP analyses agreed on the risk direction for Albumin and Blood Urea. Their exploratory sustained-transition locations were:

| Feature | EBM transition | Standard SHAP transition | Direction |
|---|---:|---:|---|
| Albumin | 31.5 | 31.5 | Lower values associated with higher predicted abnormality |
| Blood Urea | approximately 11-12 | approximately 18-19 | Higher values associated with higher predicted abnormality |

These values describe model behaviour. They are not validated clinical cut-offs.

### Exploratory Double Machine Learning

Cross-fitted residualisation DML was used on the training data as an exploratory adjusted-association analysis.

After false-discovery-rate correction:

- lower Albumin retained an adjusted association with abnormal EEG;
- Seizures retained a positive adjusted association;
- Blood Urea retained a smaller positive adjusted association;
- Generalised Seizure remained uncertain when adjusted separately.

Generalised Seizure is nested within the broader Seizures variable, which limits independent overlap. The DML analysis adjusts only for observed variables and does not establish causality.

## Sensitivity analyses

### Sodium

BorutaSHAP classified Sodium as tentative rather than accepted. A separate CatBoost sensitivity model was fitted using the four accepted features plus Sodium.

| Model | F1 | Recall | Specificity | Balanced accuracy | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Main CatBoost | 0.854 | 0.905 | 0.438 | 0.671 | 0.810 |
| Main features + Sodium | 0.872 | 0.976 | 0.313 | 0.644 | 0.768 |

Adding Sodium increased recall and F1 but reduced specificity, balanced accuracy, and ROC-AUC. Sodium therefore remains an exploratory predictor rather than part of the main model.

### Length of hospitalisation

Length of hospitalisation was evaluated only in a separate sensitivity notebook because it can act as a post-admission illness-course variable and introduce target leakage.

| Model | F1 | Recall | Specificity | Balanced accuracy | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Main CatBoost | 0.854 | 0.905 | 0.438 | 0.671 | 0.810 |
| Hospitalisation sensitivity model | 0.854 | 0.976 | 0.188 | 0.582 | 0.790 |

The sensitivity model detected more abnormal cases but produced substantially more false positives and weaker class-balanced discrimination. Length of hospitalisation remains excluded from the main model.

## Deliverables

- [Main project notebook](Project%20Files/EEGAbnormality_Project.ipynb)
- [Length of hospitalisation sensitivity notebook](Project%20Files/EEGAbnormality_LengthOfHospitalisation_Sensitivity.ipynb)
- Curated aggregate result tables under `results/`
- Paper-ready global, comparison, EBM, uncertainty, and SHAP dependence figures under `figures/`

The EDA notebook is not required for the client-facing or paper-oriented release.

## Technology stack

| Area | Tools |
|---|---|
| Data processing | Python, pandas, NumPy |
| Machine learning | scikit-learn, CatBoost |
| Feature selection | L1 regularisation, RFECV, Boruta, BorutaSHAP |
| Standard explanation | SHAP |
| Conditional explanation | pyshapr, rpy2, R shapr, CTREE |
| Interpretable modelling | InterpretML Explainable Boosting Machine |
| Adjusted-association analysis | statsmodels and cross-fitted residualisation DML |
| Reporting | Jupyter, Matplotlib, CSV outputs, and PDF reporting |

The validated execution environment used Python 3.11 and R 4.5 with fixed random seeds.

Conditional SHAP additionally requires R `shapr` 1.0.8 and its CTREE dependencies. Python package requirements are listed in `requirements.txt`.

## Reproducibility and data access

The notebooks use relative project paths and save regenerated artifacts under `outputs/`. The curated repository release stores approved aggregate tables under `results/` and figures under `figures/`. Reproduction requires an authorised local copy of the source dataset and compatible Python and R dependencies.

The medical dataset is intentionally not included in a public repository. It must not be uploaded, redistributed, or exposed through notebook outputs. A public release should contain only sanitised notebooks, aggregate results, approved figures, and documentation.

Patient-level SHAP tables, local explanations, raw data extracts, identifiers, and small-cell clinical summaries should remain outside the public repository unless disclosure approval has been obtained.

## Limitations

- The cohort is small, retrospective, and drawn from a single centre.
- The outcome is imbalanced, with abnormal EEG findings forming the majority class.
- Specificity remains modest despite strong sensitivity.
- Feature selection was restricted to the training partition but was not nested separately within every benchmarking fold.
- SHAP and EBM explain model behaviour rather than clinical causality.
- DML cannot remove bias from unmeasured confounding or uncertain temporal ordering.
- Seizure-related predictors require clear confirmation that they were available before the EEG result.
- The exploratory EBM/SHAP transitions are not clinical thresholds.
- External and prospective validation is required before considering real-world use.

## Capabilities demonstrated

This project demonstrates the ability to deliver:

- an end-to-end healthcare tabular-ML workflow;
- leakage-aware preprocessing for mixed clinical data;
- imbalanced-class model development and evaluation;
- model-specific feature selection and transparent model comparison;
- training-only hyperparameter and threshold tuning;
- Python-to-R integration for dependence-aware explainability;
- multiple complementary XAI and statistical triangulation methods;
- sensitivity analyses for tentative and leakage-prone predictors;
- reproducible research outputs suitable for technical and clinical review;
- clear communication of model performance, uncertainty, and limitations.

## Research-use statement

This repository documents an exploratory medical research analysis. The model is not medical advice, is not a diagnostic device, and is not ready for clinical deployment or patient-level decision support.
