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

## Business conclusion (Week 3)

**What drives default.** Permutation importance on the tuned LightGBM shows that credit utilization is the strongest driver (AUC drop 0.071), followed by payment history (90+ days late 0.035, 30-59 days late 0.031, 60-89 days late 0.015). Age, debt ratio, number of credit lines and income have small effects. This describes what the model relies on, not what causes default.

**Cost of each error (assumptions, not data).** Loan 10,000, loss given default 60%, profit on a repaid loan 8%.
- Bad loan approved: loses 6,000.
- Good loan rejected: loses 800 of profit.
- A bad loan approved costs 7.5 times a good loan rejected.

**Decision rule.** Approve a borrower if predicted PD < margin / (LGD + margin) = 0.1176. The model's probabilities are well calibrated around this threshold (predicted 11.6%, actual 11.6% in the relevant bin).

**Result on the test set (30,000 borrowers).**

| Rule | Approved | Default rate of approved | Profit per applicant |
|---|---|---|---|
| Approve everyone | 100% | 6.7% | 345.5 |
| Model, threshold 0.5 (Week 1 default) | 98.0%* | 5.6%* | 412.8* |
| Model, cost-based threshold 0.1176 | 85.9% | 2.78% | 525.3 |

*from out-of-fold predictions on the training data.

Using the model with the cost-based threshold earns about 180 more per applicant (+52%) than approving everyone, at the price of rejecting 14% of applicants. Per 1,000 applicants, that is about 180,000 under these assumptions.

**Robustness.** For LGD in {40%, 60%, 80%} and margin in {5%, 8%, 12%}, the model beats approving everyone in all 9 scenarios, by 74 to 323 per applicant, and the formula threshold is within 0.012 of the best searched threshold. With LGD 80% and margin 5%, approving everyone loses money (-68 per applicant) while the model earns +255.

**Limits.** The costs are assumptions; the dataset has no loan amounts, recoveries or interest rates. Every borrower is given the same loan of 10,000, and the margin is a flat share of the loan, ignoring term and funding cost. The data is a historical Kaggle sample, so results for a real portfolio would need real loan terms and recovery data.## Business conclusion (Week 3)

**What drives default.** Permutation importance on the tuned LightGBM shows that credit utilization is the strongest driver (AUC drop 0.071), followed by payment history (90+ days late 0.035, 30-59 days late 0.031, 60-89 days late 0.015). Age, debt ratio, number of credit lines and income have small effects. This describes what the model relies on, not what causes default.

**Cost of each error (assumptions, not data).** Loan 10,000, loss given default 60%, profit on a repaid loan 8%.
- Bad loan approved: loses 6,000.
- Good loan rejected: loses 800 of profit.
- A bad loan approved costs 7.5 times a good loan rejected.

**Decision rule.** Approve a borrower if predicted PD < margin / (LGD + margin) = 0.1176. The model's probabilities are well calibrated around this threshold (predicted 11.6%, actual 11.6% in the relevant bin).

**Result on the test set (30,000 borrowers).**

| Rule | Approved | Default rate of approved | Profit per applicant |
|---|---|---|---|
| Approve everyone | 100% | 6.7% | 345.5 |
| Model, threshold 0.5 (Week 1 default) | 98.0%* | 5.6%* | 412.8* |
| Model, cost-based threshold 0.1176 | 85.9% | 2.78% | 525.3 |

*from out-of-fold predictions on the training data.

Using the model with the cost-based threshold earns about 180 more per applicant (+52%) than approving everyone, at the price of rejecting 14% of applicants. Per 1,000 applicants, that is about 180,000 under these assumptions.

**Robustness.** For LGD in {40%, 60%, 80%} and margin in {5%, 8%, 12%}, the model beats approving everyone in all 9 scenarios, by 74 to 323 per applicant, and the formula threshold is within 0.012 of the best searched threshold. With LGD 80% and margin 5%, approving everyone loses money (-68 per applicant) while the model earns +255.

**Limits.** The costs are assumptions; the dataset has no loan amounts, recoveries or interest rates. Every borrower is given the same loan of 10,000, and the margin is a flat share of the loan, ignoring term and funding cost. The data is a historical Kaggle sample, so results for a real portfolio would need real loan terms and recovery data.

## Week 3 notes

- Permutation importance (on a validation part of the training data) shows utilization and payment history drive the tuned LightGBM; flags like OverLimit are redundant for tree models.
- Defined error costs from assumptions (loan 10,000, LGD 60%, margin 8%): a bad loan approved costs 7.5 times a good loan rejected.
- Chose the approval threshold from the costs (PD < 0.1176), not from the test set; out-of-fold calibration showed predicted and actual default rates match around it.
- On the test set the rule earns 525 per applicant against 345 for approving everyone, while rejecting 14% of applicants.
- The conclusion holds in all 9 tested cost scenarios, but the costs are assumptions, so real loan data would be needed to make it a decision.