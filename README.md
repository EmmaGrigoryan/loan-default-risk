# loan-default-risk
End-to-end credit risk analysis of a loan portfolio using the Kaggle Give Me Some Credit dataset. Covers data cleaning and EDA in Python, business queries in SQL, validation and pivot summaries in Excel, ML default prediction (logistic regression, random forest, XGBoost), and a Power BI risk dashboard.
## Results (Week 1 baseline)

| Model                      | Accuracy | Precision | Recall | AUC   |
|----------------------------|----------|-----------|--------|-------|
| Predict "nobody defaults"  | 0.933    | n/a       | 0.000  | 0.500 |
| Logistic regression        | 0.937    | 0.611     | 0.173  | 0.857 |

## Week 1 notes

- Built a baseline default model on the Give Me Some Credit data (150,000 borrowers, 6.7% default rate).
- Split 80/20 first, and learned every cleaning value (medians, caps) from the training part only to avoid leakage.
- Found placeholder codes (96/98) in the past-due columns, missing income in 20% of rows, and broken DebtRatio values where income was missing or near zero; fixed them and kept flag columns.
- Logistic regression reached AUC 0.857 on the test set; accuracy is misleading because "nobody defaults" is already 93.3% accurate.
- At the default 0.5 threshold recall is only 0.17, so choosing the threshold is the next thing to improve.

## Results (Week 2: model comparison)

| Model                       | CV AUC (mean) | Test AUC |
|-----------------------------|---------------|----------|
| Logistic regression         | 0.8502        | 0.857    |
| Logistic, balanced weights  | 0.8552        |          |
| Random forest               | 0.8638        |          |
| LightGBM default            | 0.8647        |          |
| LightGBM tuned (final)      | 0.8651        | 0.8685   |

Cross-validation: 5 folds on the training data (std of the baseline: 0.0043). The test set was used once, for the final model and the baseline.

**Final model: tuned LightGBM.** The three tree models are statistically tied (within 0.0013, about a third of the fold-to-fold spread), and all beat the logistic regression by about 0.015 AUC. I chose LightGBM because it is at least as accurate as the random forest, trains faster, and tuning did not change its score, so it is not fragile. The logistic regression stays as the explainable reference, since a lender has to justify a refusal.

## Week 2 notes

- Used 5-fold cross-validation (with a pipeline so scaling does not leak) to compare models fairly; baseline CV AUC 0.850 +/- 0.004.
- Class weights barely change AUC but move the 0.5 threshold: recall 0.175 -> 0.739, precision 0.594 -> 0.217.
- Random forest (0.864) and LightGBM (0.865) are statistically tied and about 0.015 AUC above the logistic regression.
- Tuning LightGBM gave +0.0004 AUC, which is noise; the model is not sensitive to its settings.
- Limitations: medians and caps were learned on the whole training part, not inside each fold, and the best tuning score is slightly optimistic.