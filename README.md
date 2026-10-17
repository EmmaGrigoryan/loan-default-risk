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