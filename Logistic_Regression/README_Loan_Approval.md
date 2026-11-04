# Loan Approval Prediction Using Logistic Regression

A Jupyter notebook that trains and evaluates a logistic regression model to predict whether a loan application will be approved. It builds a single preprocessing-and-model pipeline, saves it to disk, and uses it to score new applicants.

**Notebook:** `Loan_Approval_Prediction.ipynb`

## Table of Contents

- [Project Overview](#project-overview)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Model Performance](#model-performance)
- [Getting Started](#getting-started)
- [Predicting for New Applicants](#predicting-for-new-applicants)
- [Project Structure](#project-structure)
- [Tools and Libraries](#tools-and-libraries)
- [Model Improvements (Part 2)](#model-improvements-part-2)
- [Changelog](#changelog)
- [Disclaimer](#disclaimer)
- [License](#license)

## Project Overview

The notebook walks through a complete machine-learning workflow:

1. Load and explore historical loan data
2. Clean the data and build preprocessing pipelines
3. Train a logistic regression classifier
4. Evaluate it with accuracy, a classification report, a confusion matrix and a ROC curve
5. Save the fitted pipeline with `joblib`
6. Predict outcomes for new, unseen applicants and export them to Excel
7. Test improvements: cross-validation, class weights, threshold tuning, other models and coefficient inspection (Part 2)

## Dataset

**Required file:** `loan_data.csv` (614 records, 13 columns). It holds historical applications with the target column `Loan_Status` (`Y` = approved, `N` = rejected).

| Column | Type | Description |
|--------|------|-------------|
| Loan_ID | Text | Applicant identifier (dropped before training) |
| Gender | Categorical | `Male` or `Female` |
| Married | Categorical | `Yes` or `No` |
| Dependents | Categorical | Number of dependents (4 categories) |
| Education | Categorical | `Graduate` or `Not Graduate` |
| Self_Employed | Categorical | `Yes` or `No` |
| ApplicantIncome | Integer | Applicant income |
| CoapplicantIncome | Float | Co-applicant income |
| LoanAmount | Float | Loan amount requested |
| Loan_Amount_Term | Float | Loan term in months |
| Credit_History | Float | `1.0` for meets guidelines, `0.0` otherwise |
| Property_Area | Categorical | `Urban`, `Semiurban` or `Rural` |
| Loan_Status | Categorical | Target: `Y` or `N` |

**Data quality**

- No duplicate records
- Missing values in 8 columns, the largest being `Credit_History` (50), `Self_Employed` (32) and `LoanAmount` (22)
- Class balance: 422 of 614 applications (about 69%) are approved

**Optional file:** `new_loan_data.csv` holds new applicants with the same feature columns (no `Loan_Status` needed). A `Loan_ID` column is carried into the exported results if present. If the file is missing, the notebook scores one built-in example applicant instead.

> The datasets are not included in this repository. Add `loan_data.csv` to the project folder before running the notebook.

## Methodology

| Step | Action |
|------|--------|
| Cleaning | Drop `Loan_ID`, strip column names, convert blank and infinite values to `NaN`, drop rows with missing or invalid targets |
| Target encoding | `N` becomes 0 and `Y` becomes 1 |
| Numeric features | Median imputation, then `StandardScaler` |
| Categorical features | Most-frequent imputation, then `OneHotEncoder(handle_unknown="ignore")` |
| Split | 80% train (491 rows) and 20% test (123 rows), stratified, `random_state=42` |
| Model | `LogisticRegression(max_iter=1000)` inside a scikit-learn `Pipeline` |

Because imputation, scaling and encoding are part of the pipeline, they are learned from the training data only and reused unchanged for the test set and for new applicants.

## Model Performance

Results on the 123-row test set:

| Metric | Value |
|--------|-------|
| Accuracy | **86.18%** |

| Class | Precision | Recall | F1-score | Support |
|-------|-----------|--------|----------|---------|
| Rejected | 0.96 | 0.58 | 0.72 | 38 |
| Approved | 0.84 | 0.99 | 0.91 | 85 |
| Macro average | 0.90 | 0.78 | 0.81 | 123 |
| Weighted average | 0.88 | 0.86 | 0.85 | 123 |

**Confusion matrix** (rows are actual, columns are predicted):

|  | Predicted Rejected | Predicted Approved |
|--|--------------------|--------------------|
| **Actual Rejected** | 22 | 16 |
| **Actual Approved** | 1 | 84 |

**How to read this:** The model finds almost every approved loan (99% recall) but misses many rejections. It wrongly approves 16 of 38 applications that were actually rejected (58% recall for the Rejected class). Part 2 of the notebook tests class weights and threshold tuning against this weakness (see [Model Improvements](#model-improvements-part-2)).

The notebook also plots the confusion matrix and the ROC curve, and prints the ROC-AUC score. Run the notebook to see those values.

## Getting Started

### Prerequisites

- Python 3.9 or later
- Jupyter Notebook or JupyterLab

### Installation

```bash
git clone <your-repo-url>
cd <your-repo-folder>
pip install pandas numpy matplotlib seaborn scikit-learn openpyxl joblib jupyter
```

### Run the notebook

1. Place `loan_data.csv` in the project folder.
2. Start Jupyter:

   ```bash
   jupyter notebook Loan_Approval_Prediction.ipynb
   ```

3. Run the cells from top to bottom.

### Outputs

| File | Created by |
|------|-----------|
| `loan_approval_model.pkl` | Step 13: the saved, fitted pipeline |
| `loan_prediction_results.xlsx` | Step 17: exported predictions with `Predicted_Loan_Status` (`Y`/`N`) and `Approval_Probability` |

## Predicting for New Applicants

The notebook scores the applicants in `new_loan_data.csv` if it exists. Otherwise it scores this built-in example:

```python
new_data = {
    'Gender': 'Male', 'Married': 'Yes', 'Dependents': '0',
    'Education': 'Graduate', 'Self_Employed': 'No',
    'ApplicantIncome': 5000, 'CoapplicantIncome': 2000,
    'LoanAmount': 150, 'Loan_Amount_Term': 360,
    'Credit_History': 1, 'Property_Area': 'Urban'
}
```

The new data is aligned to the training columns, categorical values are converted to text to match training (so `Dependents` values such as `0` match `"0"`), and any missing field is imputed by the pipeline. The built-in example applicant is predicted as **approved** (`1`).

To reuse the saved model elsewhere:

```python
import joblib
import pandas as pd

model = joblib.load("loan_approval_model.pkl")
applicants = pd.DataFrame([new_data])
predictions = model.predict(applicants)               # 1 = approved, 0 = rejected
probabilities = model.predict_proba(applicants)[:, 1] # approval probability
```

## Project Structure

```
.
├── Loan_Approval_Prediction.ipynb   # Main notebook
├── loan_data.csv                    # Training data (add this yourself)
├── new_loan_data.csv                # New applicants to score (optional)
├── loan_approval_model.pkl          # Saved pipeline (generated)
├── loan_prediction_results.xlsx     # Predictions export (generated)
└── README.md
```

## Tools and Libraries

| Library | Use |
|---------|-----|
| pandas, NumPy | Data handling |
| scikit-learn | Pipelines, preprocessing, logistic regression, random forest, gradient boosting, cross-validation, metrics |
| Matplotlib, Seaborn | Confusion matrix and ROC plots |
| joblib | Saving and loading the model |
| openpyxl | Excel export |

## Model Improvements (Part 2)

Part 2 of the notebook (Steps 18 to 21) keeps the baseline model from Steps 1 to 17 unchanged and tests ways to improve on it. It reports **recall for the Rejected class** alongside accuracy and ROC-AUC, since that is the baseline's weak spot.

| Step | What it does |
|------|--------------|
| 18. Cross-validation | Stratified 5-fold cross-validation of the baseline (accuracy, ROC-AUC, Rejected recall, macro F1), as a steadier estimate than one 123-row test split |
| 19. Rejected-loan recall | Tests **class weights** (`class_weight="balanced"`) and **threshold tuning**. The threshold is chosen from out-of-fold predictions on the training set only, then applied once to the test set |
| 20. Model comparison | Compares logistic regression, random forest and gradient boosting (with and without class weights) on the same folds, sorted by ROC-AUC |
| 21. Coefficients | Shows logistic regression coefficients and odds ratios, plus a plot of the top 10 features |

Run the notebook on your own data to see these results; they depend on your dataset. Coefficients show association in the data, not cause and effect.

## Changelog

Fixes made to the original notebook:

- **Step 17 export fixed.** It called `results.to_excel(...)` but `results` was never created. The cell now builds it from the applicant data, with `Predicted_Loan_Status` (`Y`/`N`) and `Approval_Probability` columns.
- **`new_loan_data.csv` is now used.** Step 14 loads it if present and falls back to the built-in example otherwise. The introduction now lists it as optional. `Loan_ID` is kept in the output.
- **Text format of new data.** Step 15 converts categorical columns to text so numeric `Dependents` values from a CSV match the categories seen in training.
- **Blank-value regex.** Steps 4 and 15 used `r"^\\s*$"`, which matches a literal backslash rather than whitespace-only cells. It is now `r"^\s*$"`.
- **Final notes cell.** The "Important notes" cell was a code cell containing markdown and would raise a syntax error if run. It is now a markdown cell.

Added: Part 2 (Steps 18 to 21) described above.

The fixes were checked by running the full notebook end to end on synthetic data with the same columns, both with and without `new_loan_data.csv`. The numbers in [Model Performance](#model-performance) are from the original notebook run and are unchanged.

## Disclaimer

This is an educational machine-learning example, not a validated lending decision system. `Approval_Probability` is the model's estimated probability, not a guarantee of approval.

## License

Add a license of your choice (e.g. MIT) before publishing.
