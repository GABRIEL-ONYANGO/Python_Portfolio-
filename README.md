# German Credit Data: Descriptive Data Analysis

A Jupyter notebook that profiles, cleans and explores the German Credit dataset using Python. It covers initial data analysis, data wrangling and exploratory data analysis (EDA) of 1,000 loan applicants classified as good or bad credit risks.

**Notebook:** `Data_Analysis.ipynb`

## Table of Contents

- [Project Overview](#project-overview)
- [Dataset](#dataset)
- [Analysis Workflow](#analysis-workflow)
- [Key Findings](#key-findings)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Tools and Libraries](#tools-and-libraries)
- [Limitations and Next Steps](#limitations-and-next-steps)
- [License](#license)

## Project Overview

The goal is to understand who the borrowers in the dataset are and how their characteristics relate to credit risk. The notebook:

1. Checks the structure and quality of the data
2. Cleans and prepares it for analysis
3. Explores the distributions and relationships between variables, particularly Credit amount and Risk

## Dataset

**Source file:** `german_credit_data.xlsx` (1,000 rows, 10 columns)

| Column | Type | Description |
|--------|------|-------------|
| Age | Integer | Applicant age (19 to 75) |
| Sex | Categorical | `male` or `female` |
| Job | Integer | Job skill level (0 to 3) |
| Housing | Categorical | `own`, `rent` or `free` |
| Saving accounts | Categorical | Savings level; 4 categories, 183 missing |
| Checking account | Categorical | Checking level; 3 categories, 394 missing |
| Credit amount | Integer | Loan amount (250 to 18,424) |
| Duration | Integer | Loan duration in months (4 to 72) |
| Purpose | Categorical | Purpose of the loan; 8 categories (e.g. `car`, `radio/TV`) |
| Risk | Categorical | Target label: `good` or `bad` |

> The dataset file is not included in this repository. Download the German Credit dataset (Excel version) and place it in the project folder.

## Analysis Workflow

### 1. Initial Data Analysis (IDA)

- Shape, column names and data types
- Completeness check with `df.info()`
- Descriptive statistics with `describe()` and `describe(include='all')`
- Missing values, duplicate rows (none found) and unique values per column

### 2. Data Wrangling

| Step | Action |
|------|--------|
| Missing values | Filled the missing `Saving accounts` and `Checking account` values with `"Unknown"` |
| Data types | Converted `Sex` and `Housing` to the `category` type |
| Normalization | Min-max scaled `Credit amount` into `Credit amount_normalized` (scikit-learn `MinMaxScaler`) |
| Binning | Created `Age_groups` from Age (see below) |
| Encoding | One-hot encoded the categorical variables with `pd.get_dummies(drop_first=True)` into `df_encode` |

**Age groups** (lower bound inclusive, upper bound exclusive):

| Group | Age range |
|-------|-----------|
| Youth | 18 to 30 |
| Young Adult | 31 to 40 |
| Middle Age | 41 to 50 |
| Oldie | 51 to 60 |
| Senior | 61 and over |

### 3. Exploratory Data Analysis (EDA)

- Summary statistics for Credit amount
- Mean Credit amount grouped by Risk
- Age distribution histogram
- Housing category count plot
- Scatter plot of Age vs Credit amount, coloured by Risk
- Correlation heatmap of the encoded variables

## Key Findings

- **Dataset size and quality:** 1,000 records with no duplicates. Only the savings and checking account columns have missing values (18.3% and 39.4%).
- **Applicants:** The average age is about 35.5 years (median 33), and 69% are male. 71.3% own their housing.
- **Loan amounts are right-skewed:** The mean is 3,271 and the median is 2,320, with a maximum of 18,424. The average duration is about 21 months.
- **Purpose:** `car` is the most common purpose (337 loans).
- **Risk balance:** 700 applicants are labelled `good`, so about 30% are `bad`.
- **Bad risk goes with larger loans:** Mean Credit amount is about 3,938 for `bad` against about 2,985 for `good`.

## Getting Started

### Prerequisites

- Python 3.9 or later
- Jupyter Notebook or JupyterLab

### Installation

```bash
git clone <your-repo-url>
cd <your-repo-folder>
pip install numpy pandas scikit-learn matplotlib seaborn openpyxl jupyter
```

### Run the notebook

1. Place `german_credit_data.xlsx` in the project folder.
2. Open `Data_Analysis.ipynb` and update the file path in the data-loading cell. The notebook currently points to a local Windows path:

   ```python
   german_credit = r"C:\Users\USER\Downloads\german_credit_data.xlsx"
   ```

   Change it to a relative path, for example:

   ```python
   german_credit = "german_credit_data.xlsx"
   ```

3. Start Jupyter and run all cells:

   ```bash
   jupyter notebook Data_Analysis.ipynb
   ```

## Project Structure

```
.
├── Data_Analysis.ipynb        # Main analysis notebook
├── german_credit_data.xlsx    # Dataset (add this yourself)
└── README.md
```

## Tools and Libraries

| Library | Use |
|---------|-----|
| pandas | Data loading, cleaning, grouping |
| NumPy | Numerical operations |
| scikit-learn | Min-max normalization |
| Matplotlib | Plotting |
| Seaborn | Statistical visualizations |
| openpyxl | Reading the `.xlsx` file |

## Limitations and Next Steps

- `"Unknown"` is used for missing account values, so those applicants form their own group rather than being imputed.
- The age bins start at 18; any applicant younger than 18 would be left unassigned. The youngest applicant here is 19.
- The correlation heatmap is based on one-hot encoded data and is meant for a broad overview.
- Possible extensions: compare Risk across Purpose, Housing and Age_groups, check outliers in Credit amount, and build a classification model to predict Risk.

## License

For educational purposes only.
