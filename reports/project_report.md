# Trust & Safety Risk Operations Analytics

## 1. Executive summary

This project develops an interpretable machine-learning prototype for prioritizing
potentially unwanted email messages for risk review. It combines data-quality
controls, structured feature engineering, source-aware evaluation, Logistic
Regression, threshold selection, and operational workload analysis.

The project uses the historical Apache SpamAssassin Public Mail Corpus. It is an
independent portfolio project and is not affiliated with Google or based on
internal Google data.

At the selected classification threshold of 0.40,
the model achieved 90.16% accuracy, 93.35%
spam precision, and 67.40% spam recall on the held-out
test set. It flagged 361 of
1900 test messages for hypothetical review.

## 2. Business problem

Trust & Safety operations often need to identify potentially harmful or unwanted
activity while balancing detection coverage, false positives, and limited review
capacity. This project demonstrates a small-scale version of that analytical
problem using spam classification as a proxy use case.

The intended workflow is risk prioritization for human review—not automatic
blocking or a claim of production-ready abuse detection.

## 3. Dataset and scope

- Source: Apache SpamAssassin Public Mail Corpus.
- Historical data: principally from the early 2000s.
- Total messages: 6,047.
- Legitimate messages (ham): 4,150.
- Spam messages: 1,897.
- Input: locally extracted email messages.
- Target: corpus folder-derived label (`ham` or `spam`).

The corpus is historical and may not represent modern email threats, user
behaviour, or current production traffic. Raw messages are not intended for
publication with this repository.

## 4. Data preparation and quality

The workflow downloaded and extracted five corpus archives, generated checksums,
built a message manifest, parsed messages, and created a structured feature table.

The structured dataset contains 6,047 messages and 18 numeric predictive features,
along with message identifiers, source-folder metadata, and labels. Parsing errors
were recorded separately.

The quality checks found no missing values, constant features, non-finite feature
values, or exact duplicate raw messages based on SHA-256 hashes. Source-folder
metadata was excluded from model predictors because it mapped directly to labels
and could create target leakage. Message identifiers were also excluded.

## 5. Feature engineering

The numeric predictors describe message structure and simple content statistics,
including:

- Message size and body length.
- Header counts and presence of selected headers.
- MIME structure, HTML/plain-text parts, and attachment count.
- URL and exclamation-mark counts.
- Uppercase-character ratio.

These features avoid advanced NLP, transformer models, and deep learning, making
the pipeline relatively easy to explain and reproduce.

## 6. Evaluation design

The project used a source-aware split, holding out entire source folders rather
than randomly mixing messages from every folder.

- Training partition: 4,147 messages.
- Held-out test partition: 1,900 messages.
- No source folders were shared between the two partitions.

Threshold selection was performed using an internal validation subset of the
training partition. The selected threshold was then applied to the held-out test
set.

This design tests performance across the selected corpus folders, but it does
not guarantee generalization to a modern or independently collected email stream.
The source-folder composition and label distribution differ between partitions.

## 7. Baseline and model

A majority-class baseline predicted every test message as legitimate. It achieved
73.68% accuracy but detected no spam, illustrating why accuracy alone is
insufficient for an imbalanced classification problem.

The primary model is a scikit-learn pipeline comprising StandardScaler and
Logistic Regression. The scaler is fitted on training data, preventing the test
set from influencing preprocessing.

## 8. Threshold selection

The threshold was selected using validation data, considering precision, recall,
F1 score, false-positive rate, and the number of messages that would be flagged
for review.

A threshold of 0.40 was selected provisionally because it provided the highest
F1 score among the tested validation thresholds while maintaining high precision.
This is an operational choice for a review-oriented prototype, not a universal
optimum. Different error costs and review-capacity constraints could justify
another threshold.

## 9. Final held-out test results

| Metric | Result |
|---|---:|
| Classification threshold | 0.40 |
| Test messages | 1,900 |
| Accuracy | 90.16% |
| Spam precision | 93.35% |
| Spam recall | 67.40% |
| Spam F1 score | 0.7828 |
| False-positive rate | 1.71% |
| ROC AUC | 0.9401 |
| Average precision | 0.9041 |
| True negatives | 1,376 |
| False positives | 24 |
| False negatives | 163 |
| True positives | 337 |
| Messages flagged for review | 361 |
| Review rate | 19.00% |

### Operational interpretation

At the selected threshold, the model correctly flagged
337 spam messages and missed
163. It incorrectly flagged
24 legitimate messages.

These figures demonstrate the trade-off between spam detection and legitimate
message disruption. A real deployment would require a defined cost model,
capacity limits, monitoring, and human-review safeguards.

## 10. Model interpretation and sensitivity

The Logistic Regression coefficients identify associations between predictors
and model scores. Positive coefficients are associated with higher predicted
spam risk, while negative coefficients are associated with lower predicted risk,
conditional on the other predictors.

The `url_count` feature had a strong negative coefficient, despite similar spam
prevalence among messages with and without URLs in the training data. Removing
this feature reduced test ROC AUC from 0.9401 to 0.9187 and test spam recall
from 67.4% to 63.0%, while reducing false positives from 24 to 21.

The feature was retained in the current model because it contributed to predictive
performance in this diagnostic comparison. Its counterintuitive coefficient
requires further investigation. Correlated predictors and corpus-specific patterns
may contribute to the observed relationship.

This sensitivity analysis reused the held-out test set after its initial
evaluation, so it is exploratory rather than an independent confirmation of
performance. Further model selection should use validation or cross-validation
within training data, followed by evaluation on a fresh external dataset.

## 11. Limitations and responsible use

1. The dataset is historical and may not reflect contemporary spam or abuse.
2. This project is not a Google system and does not establish performance on
   Gmail or other live services.
3. The source-aware split is useful but is not a substitute for external
   validation on newer data.
4. Predictive associations are not causal explanations.
5. The chosen threshold is provisional and depends on operational priorities.
6. False positives can disrupt legitimate users; automatic enforcement would
   require additional safeguards and evaluation.
7. Raw email content should not be published as part of the portfolio.

## 12. Recommended next steps

- Validate on a newer, independently sourced dataset.
- Define an explicit review-capacity and error-cost policy.
- Evaluate threshold stability and calibration.
- Investigate feature correlations and unexpected coefficients using training
  data only.
- Track precision, recall, false positives, and distribution shifts over time.
- Add a concise dashboard showing model performance and review workload.

## 13. Reproducibility and artifacts

Key artifacts are stored in the project directory:

- `data/metadata/message_manifest.csv`
- `data/metadata/split_manifest.csv`
- `data/processed/structured_features.csv`
- `reports/feature_quality_audit.csv`
- `reports/baseline_metrics.csv`
- `reports/threshold_validation_results.csv`
- `reports/final_test_metrics.csv`
- `reports/logistic_regression_feature_coefficients.csv`
- `reports/url_feature_sensitivity_check.csv`

The raw corpus and extracted messages are intentionally excluded from the
recommended public repository. Publish the code, documentation, and appropriate
derived artifacts only after reviewing dataset terms and privacy considerations.
