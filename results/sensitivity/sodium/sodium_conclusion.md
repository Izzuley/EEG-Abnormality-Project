**Sodium sensitivity conclusion**

- Adding `SODIUM` changed held-out F1 by `+0.018`, balanced accuracy by `-0.027`, and ROC-AUC by `-0.042` compared with the main BorutaSHAP accepted-feature CatBoost model.
- The sodium SHAP dependence shows a moderate relationship with model contribution (correlation -0.50), which may be clinically interpretable with domain review.
- SODIUM should remain tentative and be discussed only as exploratory unless future validation shows consistent improvement.
- The main final CatBoost model is unchanged and continues to use BorutaSHAP accepted features only; this sodium analysis is exploratory and leakage-safe, with fitting and threshold tuning performed on training data only.