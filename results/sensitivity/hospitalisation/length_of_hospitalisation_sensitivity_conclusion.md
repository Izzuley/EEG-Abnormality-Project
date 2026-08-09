# Length of Hospitalisation Sensitivity Analysis Conclusion

**Sensitivity model, not main model.**

`LENGTH OF HOSPITALISATION` was intentionally retained as a candidate predictor in this sensitivity notebook only. It must be interpreted as a possible illness-severity or hospital-course proxy, not as a clean pre-EEG predictor. The main EEG abnormality model should still exclude it because it may represent post-admission/post-outcome leakage.

## BorutaSHAP Status

- `LENGTH OF HOSPITALISATION`: **accepted**.
- Included in final sensitivity CatBoost selected feature set: **True**.
- Dedicated SHAP dependence plot source: Final sensitivity CatBoost model.
- Approximate threshold where its SHAP contribution begins pushing prediction toward abnormal EEG: **approximately 10.00 days**.

## CatBoost Held-Out Test Comparison

| metric | main_catboost | sensitivity_catboost | sensitivity_minus_main |
| --- | --- | --- | --- |
| f1 | 0.853933 | 0.854167 | 0.000234082 |
| recall_sensitivity | 0.904762 | 0.97619 | 0.0714286 |
| precision | 0.808511 | 0.759259 | -0.0492514 |
| specificity | 0.4375 | 0.1875 | -0.25 |
| balanced_accuracy | 0.671131 | 0.581845 | -0.0892857 |
| accuracy | 0.775862 | 0.758621 | -0.0172414 |
| roc_auc | 0.809524 | 0.790179 | -0.0193452 |

## Confusion Matrix Comparison

| model | tn | fp | fn | tp |
| --- | --- | --- | --- | --- |
| Main CatBoost | 7 | 9 | 4 | 38 |
| Sensitivity CatBoost: LENGTH OF HOSPITALISATION candidate | 3 | 13 | 1 | 41 |

## Selected Features

- Main CatBoost selected features: BLOOD UREA, ALBUMIN, SEIZURES, GENERALISED SEIZURE
- Sensitivity CatBoost selected features: LENGTH OF HOSPITALISATION, CREATININE, ALBUMIN, SEIZURES, GENERALISED SEIZURE

## Interpretation

This sensitivity analysis explores whether hospitalisation length behaves like an illness-course signal in model explanations. Even if selected or visually informative in SHAP, it should not be promoted into the main deployment-valid prediction model without a revised clinical question and timing justification.

Research-only limitation: this is a small, single-centre, retrospective dataset and the outputs are not medical advice, diagnostic decision support, or deployment-ready evidence.
