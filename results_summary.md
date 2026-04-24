# Assignment 5 — Results Summary

**Course:** DATA 1204 — Stat and Pred Modeling for Analytics
**Task:** Comparative Binary Classification with XGBoost

---

## 1. Problem Statement

- Binary classification task: predict whether a company will go bankrupt.
- Target variable: `Bankrupt?` (1 = bankrupt, 0 = not bankrupt).
- Business context: risk team flags companies for review.
- Business note: missing a bankrupt company is worse than incorrectly flagging a healthy company

---

## 2. Dataset Summary

- Shape: 6,819 rows × 96 columns (95 features + 1 target).
- All features numeric: 93 `float64`, 2 `int64`.
- No missing values.
- No duplicate rows.
- No categorical columns — no encoding needed.
- Source file: `training/data.csv`.

---

## 3. Class Imbalance

- Not bankrupt (class 0): 6,599 rows → 96.77%.
- Bankrupt (class 1): 220 rows → 3.23%.
- Highly imbalanced (~30:1 ratio).
- Stratified 70/15/15 split preserves class ratio across train / val / test.
- Train: 4,773 rows (154 positive) · Val: 1,023 (33 positive) · Test: 1,023 (33 positive).

---

## 4. Discrimination Metric — PR-AUC

- PR-AUC = area under the Precision-Recall curve.
- Chosen because positives are rare, PR-AUC focuses on performance on the minority class.
- Unlike ROC-AUC, it is not inflated by the huge pool of true negatives.
- Measures how well the model *ranks* bankrupt companies above non-bankrupt ones.
- A bigger value means better discrimination.
- Threshold-free → not affected by the 0.5 cutoff.

---

## 5. Calibration Metric — Brier Score

- Brier score = mean squared error between predicted probabilities and actual outcomes.
- Lower = better (0 is perfect).
- Tells us whether predicted probabilities are *realistic* (e.g., a 0.3 prediction should be right ~30% of the time).
- Important because the risk team uses probabilities to prioritize reviews, not just 0/1 labels.

---

## 6. Feature Selection

- **Feature Set A:** all 95 cleaned features.
- **Feature Set B:** top 25 features selected by XGBoost feature importance.
- Selector model fit on **training split only** → no leakage from validation or test.
- Goal of Set B: test whether a smaller, more explainable model performs comparably.

---

## 7. Experiment Table (5 runs, threshold = 0.5)

All 13 required columns included. Split across 3 tables for readability. `exp_id` + `model` act as row IDs in each.

**Table 7a — Configuration**

| exp_id | model                     | feature_set | main_settings                                              |
|--------|---------------------------|-------------|------------------------------------------------------------|
| 1      | Logistic Regression       | A           | class_weight=balanced, scaled inputs                       |
| 2      | XGBoost Baseline          | A           | 200 trees, depth=4, lr=0.05                                |
| 3      | XGBoost Imbalance         | A           | baseline + scale_pos_weight                                |
| 4      | XGBoost Tuned             | A           | 400 trees, depth=3, lr=0.03, subsample=0.9, colsample=0.7  |
| 5      | XGBoost Selected Features | B           | top 25 features, depth=3, gamma=0.1, reg_lambda=2.0        |

**Table 7b — Discrimination & Calibration Metrics**

| exp_id | model                     | train_pr_auc | val_pr_auc | overfit_gap | val_roc_auc | val_brier |
|--------|---------------------------|--------------|------------|-------------|-------------|-----------|
| 1      | Logistic Regression       | 0.4755       | 0.2729     | 0.2026      | 0.8852      | 0.0934    |
| 2      | XGBoost Baseline          | 1.0000       | 0.5993     | 0.4007      | 0.9641      | 0.0198    |
| 3      | XGBoost Imbalance         | 0.9994       | 0.4915     | 0.5079      | 0.9610      | 0.0293    |
| 4      | XGBoost Tuned             | 0.9847       | 0.6153     | 0.3694      | 0.9659      | 0.0189    |
| 5      | XGBoost Selected Features | 0.9096       | 0.5145     | 0.3951      | 0.9624      | 0.0212    |

**Table 7c — Threshold Metrics & Selection**

| exp_id| model               | threshold | val_precision| val_recall| val_f1_or_f2 | selected_finalist |  notes                 |
|-------|---------------------|-----------|--------------|------------|--------------|------------------|------------------------|
| 1     | Logistic Regression | 0.5      | 0.1769        | 0.7879     | 0.4660       | No               | simple baseline        |
| 2     | XGBoost Baseline    | 0.5      | 0.7143        | 0.3030     | 0.3425       | No               | baseline XGBoost       |
| 3     | XGBoost Imbalance   | 0.5      | 0.4074        | 0.6667     | 0.5914       | No               |imbalance handling added|
| 4     | XGBoost Tuned       | 0.5      | 0.7692        | 0.3030     | 0.3448       | **Yes**          |light tuning across core 
                                                                                                         parameters              |
| 5     | XGBoost Selected Features| 0.5 | 0.6923        | 0.2727     | 0.3103       | No               |smaller feature set     |

Note: `val_f1_or_f2` column reports the F2 score (β=2), since false negatives are weighted more heavily than false positives.

---

## 8. Winning Model

- **Winner:** Experiment 4 — XGBoost Tuned (Feature Set A).
- Settings: 400 trees, depth=3, lr=0.03, subsample=0.9, colsample=0.7, reg_lambda=2.0, gamma=0.1.
- **Why it won:**
  - Highest validation PR-AUC (0.6153).
  - Lowest validation Brier score (0.0189) → best calibrated.
  - Smaller overfit gap than the baseline XGBoost (0.37 vs 0.40).
  - Regularization (lower lr, gamma, reg_lambda) tamed the overfit seen in Exp 2.

---

## 9. Threshold Selection (validation only)

- Threshold grid: 0.05 → 0.95 in steps of 0.05.
- Chose threshold that maximized F2 (recall weighted more since false negatives are costly).
- **Final threshold: 0.15** → val F2 = 0.7143, val precision = 0.52, val recall = 0.79.

---

## 10. Final Test Results (held-out, threshold = 0.15)

| metric           | value  |
|------------------|--------|
| test PR-AUC      | 0.5403 |
| test ROC-AUC     | 0.9534 |
| test Brier       | 0.0217 |
| test precision   | 0.4186 |
| test recall      | 0.5455 |
| test F2          | 0.5143 |
| final threshold  | 0.15   |

---

## 11. Confusion Matrix (test set)

|              | Pred 0 | Pred 1 |
|--------------|--------|--------|
| **Actual 0** | 965    | 25     |
| **Actual 1** | 15     | 18     |

- True Positives: 18 · False Negatives: 15 · False Positives: 25 · True Negatives: 965.
- Model caught 18 of 33 bankruptcies (55%) while flagging only 25 non-bankrupt companies.

---

## 12. Calibration Result

- Test Brier score: **0.0217** → strong calibration.
- Calibration curve tracks close to the diagonal in low-probability bins.
- Some drift in higher-probability bins (expected with few positive examples).
- Predicted probabilities are trustworthy enough for triage / prioritization.

---

## 13. Feature Importance (top drivers)

- Top features driving bankruptcy predictions (by XGBoost gain):
  - Persistent EPS in the Last Four Seasons
  - Liability to Equity
  - Equity to Liability
  - Net Value Growth Rate
  - Borrowing dependency
  - Continuous interest rate (after tax)
  - Net worth / Assets
  - Net Income to Total Assets
  - Quick Ratio
  - Debt ratio %
- Pattern: profitability, leverage, and liquidity ratios dominate — consistent with financial intuition.

**Do these features make sense?**

- Yes. The top drivers fall into three well-known bankruptcy-risk categories:
  - *Profitability:* Persistent EPS, Net Income to Total Assets, ROA — weak profit = higher risk.
  - *Leverage / solvency:* Liability to Equity, Borrowing dependency, Debt ratio %, Net worth/Assets — higher debt load = higher risk.
  - *Liquidity:* Quick Ratio — inability to cover short-term obligations = higher risk.
- These align with classic credit-risk frameworks (e.g. Altman Z-score uses similar ratios).

**Limitation of this interpretation:**

- XGBoost feature importance (gain-based) only tells us *which features the model used*, not the *direction* of the effect or whether the relationship is causal.
- A feature can score high simply because it is correlated with another predictive feature, or because it helped split noisy nodes — not because it truly drives bankruptcy.
- For directional / causal insight, SHAP values or a logistic regression coefficient analysis would be a better follow-up.

---

## 14. Overfitting Discussion

- Calculated as `overfit_gap = train PR-AUC − val PR-AUC`.
- Baseline XGBoost (Exp 2) hit train PR-AUC = 1.00 → clear overfitting.
- Tuned model (Exp 4) reduced overfit gap by adding regularization: lower learning rate (0.03), shallower trees (depth 3), gamma, and higher reg_lambda.
- Remaining gap (0.37) is non-trivial but acceptable given strong val Brier and PR-AUC.
- Test PR-AUC (0.54) dropped from val PR-AUC (0.62) → mild generalization gap, likely tied to the small test positive count (n=33).
- No test data was ever used for model or threshold selection.

---

## 15. AI Usage

- I used Codex in VS Code to help organize the notebook and write reusable evaluation code.
- Codex helped me set up the experiment workflow and plotting code.
- I manually verified that the split was stratified and that the test set was not used too early.
- I also checked that feature selection was done using training data only.
- I reviewed AI suggestions instead of accepting them blindly.
