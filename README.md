# Statistical Learning of Calcium-Signaling Responses in Neurodegenerative Disease

This repository contains the computational workflow developed for a master's thesis on the classification of neurodegenerative disease states from calcium-signaling responses in cultured neurons.

The analysis investigates whether quantitative features of calcium-associated fluorescence responses can distinguish samples associated with **amyotrophic lateral sclerosis (ALS)**, **spinal muscular atrophy (SMA)**, and a **control** condition, and tests whether the learned patterns generalize to observations from previously unseen donors.

## Project overview

Primary rat cerebellar granule neurons were exposed to cerebrospinal fluid (CSF) obtained from ALS, SMA, and control donors. Calcium-associated fluorescence responses were measured using Fluo-4 AM and summarized through direct response characteristics and fitted kinetic parameters.

The computational workflow starts from preprocessed Excel workbooks produced by the experimental group. Upstream fluorescence normalization, peak detection, and biexponential fitting were performed separately and are not reimplemented here.

The main analysis comprises:

- extraction and harmonization of ROI-level response features;
- quality control and exclusion of invalid fluorescence traces;
- exploratory analysis of K+ and CSF response characteristics;
- correlation and missing-data assessment;
- multinomial logistic-regression classification;
- supervised feature selection using minimum-redundancy maximum-relevance (mRMR);
- Random Forest and Histogram Gradient Boosting models;
- model complexity and regularization sensitivity analyses;
- ROI-level cross-validation and leave-one-donor-out generalization analysis.

## Main dataset

The final modeling dataset contains **997 ROIs** from **5 CSF donors** across **6 experiments**:

| Class   | ROIs |
| ------- | ---- |
| SMA     | 545  |
| ALS     | 269  |
| Control | 183  |

The dataset is hierarchical: many ROI-level observations originate from the same experiment and donor. For this reason, performance measured by ordinary ROI-level cross-validation is interpreted separately from performance on a completely held-out donor.

## Main findings

The primary multinomial logistic-regression model using the combined direct-response feature set achieved:

- **ROI-level balanced accuracy:** 0.920
- **ROI-level macro-F1:** 0.904
- **Mean recall for ROIs from a held-out donor:** 0.209

The large difference between ROI-level discrimination and held-out-donor performance indicates that a substantial part of the predictive structure is associated with donor and/or experimental origin rather than a demonstrated disease-generalizable calcium-response signature.

Additional feature ratios, biexponential kinetic parameters, mRMR-selected subsets, Random Forest, and Histogram Gradient Boosting did not consistently resolve this generalization problem.

Accordingly, the models in this repository should be interpreted as an analysis of the available experimental dataset, not as validated diagnostic classifiers or established disease biomarkers.

## Repository structure

```text
MasterProjekatPetarJ/
├── README.md
├── environment.yml
├── environment-linux-64.lock.txt
├── .gitattributes
├── data/
│   ├── raw/
│   ├── unpacked/
│   └── extracted/
├── notebooks/
│   ├── 01_extract_features.ipynb
│   ├── 02_eda.ipynb
│   └── 03_modeling.ipynb
├── results/
│   ├── figures/
│   ├── modeling/
│   └── tables/
└── testing/
    └── k_amplitude_testing/
```

## Notebook workflow

The main notebooks are intended to be run in this order:

1. **`01_extract_features.ipynb`**: loads the experimental workbooks, validates their structure, applies ROI-level quality-control rules, extracts response and fit-derived features, and constructs the consolidated modeling table.
2. **`02_eda.ipynb`**: performs exploratory data analysis, including class/donor distributions, response-frequency summaries, feature distributions, correlations, missingness, and descriptive comparisons.
3. **`03_modeling.ipynb`**: contains the main classification workflow: logistic regression, ROI-level cross-validation, leave-one-donor-out analysis, fit-derived predictor sets, mRMR feature selection, Random Forest, Histogram Gradient Boosting, and sensitivity analyses.

The separate `testing/k_amplitude_testing/` workflow contains an exploratory analysis of K+ amplitudes across a broader collection of experiments. It was used to assess whether unusually weak K+ responses formed a reproducible low-response cell population that would justify additional ROI exclusion. This analysis remained exploratory and did **not** change the quality-control criterion used in the main modeling dataset.

## Environment

The Conda environment used for the implementation of this project can be recreated from the provided environment file. The analysis uses packages including Python 3.13, NumPy, pandas, scikit-learn, Feature-engine, Matplotlib, openpyxl, JupyterLab, and ipykernel. For exact package constraints, see [`environment.yml`](environment.yml).

## Reproducibility notes

- Randomized analyses use `random_state=42` where applicable.
- Imputation, scaling, and supervised feature selection are performed within the training data of each modeling split to avoid data leakage.
- Continuous missing values are median-imputed.
- Logistic-regression predictors are standardized before fitting.
- Class weighting is used to reduce the influence of class imbalance.
- The main ROI-level evaluation uses 5-fold stratified cross-validation.
- Biological generalization is evaluated separately by withholding all ROIs from one donor during training.
- The only control donor cannot be evaluated by leave-one-donor-out classification because removing that donor eliminates the control class from the training set.
- mRMR configurations selected using the same donor holdouts are treated as exploratory rather than as independently validated performance estimates.

## Predictor groups

The analysis considers direct K+ response characteristics, direct CSF response characteristics, binary response-status variables, combined K+ and CSF features, CSF/K+ ratios, biexponential fit parameters, reduced kinetic representations, and supervised mRMR-selected subsets.

## Models

The main classifier is **multinomial logistic regression** with L2 regularization and balanced class weights.

Additional analyses use:

- **Random Forest**, to test whether nonlinear threshold-based relationships and feature interactions improve prediction;
- **Histogram Gradient Boosting**, as a more flexible sequential tree ensemble;
- **mRMR feature selection**, using both mutual-information difference (MID) and mutual-information quotient (MIQ) criteria.

Tree-based models were additionally evaluated under more strongly regularized configurations to test whether reducing model complexity improved generalization to held-out donors.

## Data and privacy

The source workbooks originate from experiments involving patient-derived CSF. The raw experimental files which are included in the public repository and which are necessary for reproducing the analysis from the beginning contain no patient-identifying information.

## Associated thesis

This repository accompanies the corresponding master's thesis at the University of Belgrade, Faculty of Biology.

**Title:**  
*Development of machine learning model for the classification of neurodegenerative conditions based on neuronal calcium signaling dynamics*


