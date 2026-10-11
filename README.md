# Trust & Safety Risk Operations Analytics

An interpretable machine-learning portfolio project exploring email spam-risk
prioritization, threshold selection, and review workload.

## Project highlights

- Historical Apache SpamAssassin public corpus: 6,047 messages.
- Structured message features engineered with Python and Pandas.
- Source-aware train/test split to reduce source-folder overlap.
- Majority-class baseline and Logistic Regression pipeline.
- Validation-based threshold selection and held-out test evaluation.
- Precision, recall, F1, ROC AUC, confusion matrix, and review-rate analysis.
- Coefficient interpretation and feature sensitivity analysis.

## Held-out test results

At a provisional threshold of 0.40:

- Accuracy: 90.16%
- Spam precision: 93.35%
- Spam recall: 67.40%
- Spam F1: 0.7828
- ROC AUC: 0.9401
- False-positive rate: 1.71%
- Messages flagged for hypothetical review: 361 of 1,900

See `reports/project_report.md` for the methodology, full metrics, limitations,
and recommendations. The values above describe this historical test set and
should not be interpreted as expected production performance.

## Technology

Python, Pandas, NumPy, scikit-learn, Logistic Regression, data-quality checks,
feature engineering, classification metrics, and threshold analysis.

## Responsible-use note

This is an independent portfolio prototype, not a Google product or an analysis
of internal Google data. It uses historical spam data and does not demonstrate
performance on modern Gmail traffic. Raw messages should not be published.
