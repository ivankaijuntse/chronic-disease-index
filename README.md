# Chronic-Disease Death Ratios and Human Development

**DATASCI 112 Final Project**

This project investigates whether the *composition* of chronic-disease deaths can act as a signal of national health and development. It combines country-level cause-of-death data with Human Development Index (HDI), life expectancy, and gross national income (GNI) per capita, then applies regression, classification, clustering, and distance-based analysis.

The analysis covers **184 countries from 1990 through 2019** in the final merged dataset.

## Project goals

The notebooks address four questions:

1. Can a country's chronic-disease death ratio predict HDI, life expectancy, or GNI per capita?
2. How does the ratio vary across development, life-expectancy, and income groups, and can it predict those groups?
3. How far is each country's ratio from an empirical reference ratio, and does that distance correlate with development indicators?
4. Can historical country and development data predict a later chronic-disease death ratio?

## What is the death ratio?

For each country-year, the project selects nine chronic causes of death:

- Alzheimer's disease and other dementias
- Parkinson's disease
- Cardiovascular diseases
- Neoplasms
- Diabetes mellitus
- Chronic kidney disease
- Chronic respiratory diseases
- Cirrhosis and other chronic liver diseases
- HIV/AIDS

For disease $i$, its ratio is calculated as:

$$
r_i = \frac{\text{deaths from disease } i}{\text{total deaths across the nine selected causes}}
$$

Together, the nine values form a vector that sums to 1. This represents the distribution of deaths among the selected causes; it is not an individual person's probability of dying from a disease.

## Reported results

These figures come from the outputs saved in the notebooks.

| Analysis | Method | Saved result |
| --- | --- | --- |
| Predicting development metrics | Tuned K-nearest-neighbors regression with feature search | Best saved $R^2$: **0.892** for GNI per capita, **0.885** for HDI, and **0.880** for life expectancy |
| Predicting development groups | Tuned K-nearest-neighbors classification | **92.33%** accuracy for the HDI-derived class, **85.61%** for life-expectancy bins, and **72.15%** for GNI groups |
| Comparing countries with a reference ratio | Euclidean distance and Spearman rank correlation | Distance correlated with HDI at $\rho=-0.568$ and life expectancy at $\rho=-0.648$; the sign is negative because smaller distance is considered better |
| Predicting 2019 ratios | Multi-output linear and KNN regression | KNN had lower single-year holdout MSE for the United States, Germany, Nigeria, and Thailand; cross-validation results were mixed |

The reference ratio in Question 3 is the mean 2019 ratio for Japan, Switzerland, Singapore, South Korea, and Italy. Monaco was excluded because its small population could make its ratio unusually variable.

## Repository contents

| File | Purpose |
| --- | --- |
| [`Cleaned Data.ipynb`](Cleaned%20Data.ipynb) | Downloads the cause-of-death data, selects the nine causes, reshapes the development indicators, merges the sources, and writes `merged-df.csv` |
| [`Question 1 Ready to submit.ipynb`](Question%201%20Ready%20to%20submit.ipynb) | Uses KNN regression and feature selection to predict GNI per capita, HDI, and life expectancy from the death ratio |
| [`Question2.ipynb`](Question2.ipynb) | Compares ratios across metric-derived groups and trains KNN classifiers to predict those groups |
| [`Question2-Copy1.ipynb`](Question2-Copy1.ipynb) | Byte-for-byte duplicate of `Question2.ipynb`, retained from the original project files |
| [`Question3.ipynb`](Question3.ipynb) | Builds the empirical reference ratio, ranks countries by Euclidean distance, and compares the ranking with HDI and life expectancy |
| [`Question4.ipynb`](Question4.ipynb) | Compares linear regression and KNN for a 2019 multi-output prediction exercise |
| [`merged-df.csv`](merged-df.csv) | Prepared analysis dataset with 5,520 country-year rows covering 184 countries and 1990–2019 |
| [`Human Development Index - Full.csv`](Human%20Development%20Index%20-%20Full.csv) | Wide-format development dataset used to obtain HDI, life expectancy, and GNI per capita |

## Data sources

- [Cause of Deaths around the World (Historical Data)](https://www.kaggle.com/datasets/iamsouravbanerjee/cause-of-deaths-around-the-world/data), downloaded in the notebooks with `kagglehub`
- [UNDP Human Development Index data](https://hdr.undp.org/data-center/human-development-index). The included `Human Development Index - Full.csv` contains UNDP-style development fields, but its exact original download URL and version were not recorded in the project.

The included `merged-df.csv` contains the fields used by the analysis, so Questions 2–4 can be explored without downloading the raw mortality dataset again.

## Setup

The notebooks were created with Python 3.12 and Jupyter. From the project directory, create a virtual environment:

```bash
python -m venv .venv
```

Activate the environment:

```bash
# macOS or Linux
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1
```

Install the required packages:

```bash
python -m pip install jupyterlab kagglehub matplotlib numpy pandas scipy scikit-learn seaborn
```

Then launch JupyterLab:

```bash
jupyter lab
```

## Recommended run order

To reproduce the full data-preparation workflow:

1. Run `Cleaned Data.ipynb`. This requires internet access for the Kaggle download and uses the included HDI CSV. It regenerates `merged-df.csv`.
2. Run `Question 1 Ready to submit.ipynb` independently. It downloads and prepares its own source data.
3. Run `Question2.ipynb`, `Question3.ipynb`, and `Question4.ipynb` in any order; each reads `merged-df.csv`.

The saved outputs can also be reviewed without rerunning the notebooks. Some searches are intentionally expensive:

- The exhaustive feature search in Question 1 may take more than 30 minutes.
- The tuning step in Question 2 may take 10 minutes or more.
- `Cleaned Data.ipynb` contains a stale saved `NameError` from an out-of-order execution. Running the notebook from the first cell resolves it because `df_hdi` is defined immediately beforehand.

## Methods and tools

- Data cleaning and reshaping: pandas, NumPy
- Visualization: Matplotlib, Seaborn
- Regression: linear regression, KNN regression, multi-output regression
- Classification and exploratory grouping: KNN classification, K-means
- Model selection: train/test splits, cross-validation, grid search, feature-subset search
- Comparison and ranking: Euclidean distance, Spearman rank correlation

## Interpretation and limitations

This is an exploratory country-level analysis, not a causal or clinical model.

- The ratio captures the relative composition of deaths among nine selected causes, not disease incidence, individual risk, or healthcare quality by itself.
- Country age structure, population size, reporting quality, and differences in coding practices are not directly controlled.
- Rows with missing indicators are dropped in several analyses.
- The HDI-based "Global North/South" labels in Question 2 are metric-derived classes, not geographic definitions.
- Some saved regression and classification scores are optimistic because tuning and evaluation reuse the same broader sample, and repeated country-year observations can leak country and time information across random folds.
- The Question 3 reference ratio is chosen using life expectancy, so its association with life expectancy is partly circular.
- Question 4 is a four-country, one-year backtest using contemporaneous 2019 indicators, not a production forecast of an unseen future period.
- No random seed is fixed, so split-based results can vary when cells are rerun.

The results should therefore be read as evidence of association and predictive signal within this dataset, not proof that the death ratio causes or fully measures human development.
