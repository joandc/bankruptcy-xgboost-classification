# Assignment 5 — Comparative Binary Classification with XGBoost

**Course:** DATA 1204 — Stat and Pred Modeling for Analytics <br/>
**Task:** Predict company bankruptcy using XGBoost on an imbalanced dataset.

This repository contains the notebook and supporting files for Assignment 5.
---

## Assignment Overview

The goal of this assignment is to build binary classification models that predict whether a company is likely to go bankrupt. The target variable is `Bankrupt?`, where the positive class is `1`.

This is an imbalanced classification problem, so the project focuses on:

- proper train, validation, and test splitting
- leakage control
- model comparison using the same workflow
- discrimination using `PR-AUC`
- calibration using `Brier score`
- overfitting checks using train and validation performance

---

## Repository Structure

```
.
├── jdc_assignment5.ipynb                  # main notebook (all code + experiments)
├── results_summary.md                     # write-up (problem, experiments, final results)
├── README.md                              # this file
├── .gitignore
└── training/
    └── data.csv                           # bankruptcy dataset (not tracked in git)
```

---

## Requirements

- Python 3.10 or newer
- Jupyter Notebook or VS Code with the Jupyter extension
- Python packages:
  - `pandas`
  - `numpy`
  - `scikit-learn`
  - `xgboost`
  - `matplotlib`
  - `seaborn`

---

## Setup

**1. Clone the repo**

```bash
git clone <repo-url>
cd <repo-folder>
```

**2. Create a virtual environment (recommended)**

```bash
# macOS / Linux
python3 -m venv .venv
source .venv/bin/activate

# Windows
python -m venv .venv
.venv\Scripts\activate
```

**3. Install dependencies**

```bash
pip install pandas numpy scikit-learn xgboost matplotlib seaborn jupyter
```

**4. Add the dataset**

- Place `data.csv` inside the `training/` folder.
- Final path should be: `training/data.csv`.

---

## How to Run

**Option 1 — Jupyter Notebook**

```bash
jupyter notebook jdc_assignment5.ipynb
```

Then click **Kernel → Restart & Run All**.

**Option 2 — VS Code**

1. Open the folder in VS Code.
2. Open `jdc_assignment5.ipynb`.
3. Select the `.venv` Python kernel.
4. Click **Run All**.

Runtime is about 2–4 minutes on a normal laptop.

---

## What the Notebook Does

1. Loads the dataset and runs basic sanity checks (shape, dtypes, duplicates, nulls).
2. Splits data 70/15/15 (train / val / test) with stratification on the target.
3. Runs 5 experiments — Logistic Regression, plus 4 XGBoost variants.
4. Picks a finalist using **validation** PR-AUC and Brier score.
5. Tunes the decision threshold on **validation only** (max F2).
6. Evaluates the finalist on the **held-out test set** exactly once.
7. Generates the confusion matrix, calibration curve, and feature importance plot.

---

## Key Results

- Finalist: **XGBoost Tuned** (Experiment 4).
- Threshold: **0.15** (chosen by max validation F2).
- Test PR-AUC: **0.5403**
- Test ROC-AUC: **0.9534**
- Test Brier: **0.0217**
- Test Recall: **0.5455** · Test Precision: **0.4186** · Test F2: **0.5143**

Full breakdown in `results_summary.md`.

---

## Reproducibility

- All splits use `random_state=42`.
- Feature selection is done on the **training split only** — no leakage.
- The test set is only used in the final evaluation section.

---

## AI Usage

Codex in VS Code was used to help organize the notebook and refactor repetitive evaluation code. All suggestions were reviewed manually before acceptance. See the **AI Usage** section in `results_summary.md` for details.
