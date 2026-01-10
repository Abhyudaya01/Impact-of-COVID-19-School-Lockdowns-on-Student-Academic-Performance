# COVID-19 Student Performance Panel Analysis

This project analyzes a panel dataset of students observed across multiple time periods during the COVID‑19 pandemic to understand how school lockdowns and socioeconomic status (SES) relate to changes in academic performance, with a focus on reading scores. [file:1]

The workflow covers data cleaning, feature engineering, exploratory analysis, and a linear regression model to quantify how COVID status, SES, and score volatility are associated with changes in reading performance over time. [file:1]

---

## Dataset

The project uses a single panel dataset, `COVID19ConstructedDatasetPANEL`, where each row corresponds to one `studentID` at a given `timeperiod`. [file:1]

Key groups of variables include: [file:1]

- **Demographics**
  - `studentID`, `school`, `gradelevel`, `gender`, `timeperiod`
- **Socioeconomic / access indicators**
  - `householdincome`, `freelunch`, `numcomputers`, `familysize`, `fathereduc`, `mothereduc`
- **COVID‑related indicator**
  - `covidpositive` (COVID status for the student)
- **Academic outcomes**
  - `readingscore`, `writingscore`, `mathscore`
  - `readingscoreSL`, `writingscoreSL`, `mathscoreSL` (second-language variants)

Descriptive statistics show, for example, that reading, writing, and math scores are all roughly centered in the mid‑70s with standard deviations around 13–14 points, and SES variables such as household income have long right tails, which motivates winsorization. [file:1]

---

## Problem Statement and Hypothesis

The central hypothesis is:

> If COVID‑19 school lockdowns occur (independent variable), then average student test scores (dependent variables) will decrease, especially among low‑SES students with limited technology access. [file:1]

To address this, the project focuses on modeling **percentage change in reading scores over time** and how it relates to: [file:1]

- Baseline academic performance  
- Score changes and volatility over time  
- SES variables (income, free lunch, computers, family size, parental education)  
- COVID positivity and basic demographics  

---

## Data Cleaning and Preprocessing

A detailed cleaning pipeline is implemented to make the panel data reliable for modeling. [file:1]

Main steps:

- **Duplicate handling**
  - Created a working copy of the data and dropped exact duplicate rows; the run used here reported `duplicatesdropped = 0`. [file:1]
- **Type enforcement**
  - Coerced numeric features (income, counts, parental education, all score fields) to numeric types. [file:1]
  - Casted categorical variables (`gender`, `covidpositive`, `freelunch`, `school`, `gradelevel`, `timeperiod`) to categorical integer codes for clarity and memory efficiency. [file:1]
- **Normalization of count-like fields**
  - Rounded `numcomputers`, `familysize`, `fathereduc`, and `mothereduc` to integers, enforced nonnegative bounds, and set `familysize >= 1`. [file:1]
- **Score constraints and audit flags**
  - Clipped all score variables to the valid range \([0, 100]\) and created indicator columns marking whether any score was clipped; in this run `scorerowsclipped = 0`, meaning all scores were already in range. [file:1]
- **Income winsorization**
  - Constructed `householdincomewins` by capping raw income at the 1st and 99th percentiles to reduce the influence of extreme tails. [file:1]
  - The run reported `incomewinsorbounds ≈ (9651.34, 156855.61)`. [file:1]
- **Quality checks**
  - Verified that each `(studentID, timeperiod)` combination appears exactly once, confirming a clean panel structure. [file:1]

Boxplots before and after cleaning are used to visually confirm reduced outlier influence and more stable SES and score distributions. [file:1]

---

## Feature Engineering

To test the hypothesis about changes over time and instability in performance, the notebook engineers several new features. [file:1]

**Core engineered features:**

- **Percentage change features**
  - `readingscorepctchange`, `writingscorepctchange`, `mathscorepctchange`  
  - These capture improvement or decline in scores between time periods, allowing the model to focus on *change* rather than static levels. [file:1]
- **Rolling volatility features (3‑period)**
  - `readingscorerollingvol`, `writingscorerollingvol`, `mathscorerollingvol`  
  - 3‑period rolling standard deviations that measure instability or volatility in each subject’s performance over time. [file:1]

A **reduced modeling dataset** is then created with only the features that are directly relevant to the hypothesis. [file:1]

Selected columns: [file:1]

- `readingscore`, `writingscore`, `mathscore`
- `readingscorepctchange`, `writingscorepctchange`, `mathscorepctchange`
- `readingscorerollingvol`, `writingscorerollingvol`, `mathscorerollingvol`
- `gender`, `school`, `freelunch`, `covidpositive`

Dimensionality summary: the original dataset has 8400 rows and 34 columns, while the reduced dataset has 8400 rows and 13 columns (21 columns removed). [file:1]

Formal dimensionality reduction methods like PCA are not used because: [file:1]

1. The dataset combines categorical and continuous variables.  
2. All selected features are substantively meaningful for interpretation.  
3. PCA would reduce interpretability, which is central to the project’s goals.  

---

## Exploratory Data Analysis

EDA focuses on distribution shapes, SES patterns, and relationships between SES/COVID and performance changes. [file:1]

Highlights:

- **Distributions**
  - Histograms and KDE plots of score change and rolling volatility features show wide variation in percentage changes and volatility across students and time. [file:1]
- **SES and score change**
  - Scatterplots such as `freelunch` vs `readingscorepctchange` and correlations are examined to see whether low‑SES indicators are associated with steeper declines in reading performance. [file:1]
- **Panel structure check**
  - Verified: total rows = 8400, unique students = 1400, confirming 6 time periods per student. [file:1]

These diagnostics support the modeling choice to explicitly include SES and COVID status alongside change and volatility features. [file:1]

---

## Modeling Approach

The predictive model is a standard **Linear Regression** (scikit‑learn) targeting **`readingscorepctchange`**. [file:1]

**Modeling steps:** [file:1]

1. **Target definition**
   - Target variable: `readingscorepctchange`.  
2. **Feature selection for the model**
   - Inputs include:
     - `writingscorepctchange`, `mathscorepctchange`
     - `readingscorerollingvol`, `writingscorerollingvol`, `mathscorerollingvol`
     - `gender`, `school`, `freelunch`, `covidpositive` [file:1]
3. **Train–test split**
   - 80% of data for training, 20% for testing to evaluate performance on unseen cases. [file:1]
4. **Model training**
   - `LinearRegression` is fit on the training set, using the engineered features and demographic/COVID indicators as predictors. [file:1]
5. **Prediction and evaluation**
   - Predictions on the test set are generated via `model.predict(X_test)`. [file:1]
   - **Evaluation metric:** Mean Squared Error (MSE). The final reported MSE is approximately **491.49**. [file:1]

Linear Regression is chosen because the primary goal is **interpretability**—understanding how COVID status, SES, and volatility relate to changes in reading scores—rather than maximizing raw predictive accuracy with complex models. [file:1]

---

## Model Diagnostics

Residual analysis is used to check whether the linear model assumptions are reasonably satisfied. [file:1]

Diagnostics include:

- **Residuals vs predicted values**
  - Scatterplot of residuals versus predictions shows residuals scattered around zero without strong patterns, suggesting no obvious nonlinearity or heteroscedasticity. [file:1]
- **Residual distribution**
  - Histogram and KDE of residuals indicate a roughly centered distribution with no extreme skew, consistent with a stable fit. [file:1]

Because Linear Regression is a low‑variance model and the residual diagnostics look stable, no additional post‑processing steps (like pruning or ensemble compression) are applied. [file:1]

---

## Files and Structure

Suggested repository structure:

```text
.
├── data/
│   └── COVID19ConstructedDatasetPANEL.csv      # Original Kaggle dataset (not committed if large/licensed)
├── notebooks/
│   └── 602_p5_abhyudayalohani-1.ipynb          # Main analysis notebook
├── README.md
└── requirements.txt                            # Optional: Python dependencies
