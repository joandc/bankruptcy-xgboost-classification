# Assignment 5: Comparative Binary Classification with XGBoost

This repository contains the notebook and supporting files for Assignment 5.

## Files

- `assignment5_template.ipynb`: notebook template for the assignment workflow
- `assignment5_summary_template.md`: short summary template
- `.gitignore`: common files and folders to exclude from Git

## Main Goal

Build a binary classification model to predict whether a company is likely to go bankrupt using `training/data.csv`.

## Required Tools

- Python 3.x
- pandas
- numpy
- scikit-learn
- xgboost
- matplotlib
- seaborn

## Setup

Install the required packages in your environment:

```bash
pip install pandas numpy scikit-learn xgboost matplotlib seaborn
```

## How To Run

1. Place the assignment dataset where the notebook can access `training/data.csv`, or update the `DATA_PATH` variable in the notebook.
2. Open `assignment5_template.ipynb`.
3. Run the notebook from top to bottom without skipping cells.
4. Rename the final notebook to `lastname_firstname_assignment5.ipynb` before submission.

## Assignment Notes

- Use only `training/data.csv` for modeling.
- Keep the test set untouched until the final evaluation section.
- Run exactly 5 experiments.
- Use `0.5` as the threshold for all experiment comparisons.
- After choosing the final model, select one final threshold on the validation set only and reuse it on the test set.

## Submission Checklist

- Notebook
- Results summary
- GitHub repository link
- Walkthrough video
