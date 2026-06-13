# Rolling-window Feature Enrichment and Delta-target Analysis in Cross-participant Clinical Outcome Prediction

This repository contains the implementation code and reproducible notebook for the paper:

> **Rolling-window Feature Enrichment and Delta-target Analysis in Cross-participant Clinical Outcome Prediction**  
> Mahmoud Z A Nijim, Shahrooz Abghari, Veselka Boeva  
> *Blekinge Institute of Technology, Department of Computer Science, Karlskrona, Sweden*  
> MAISON-LLF Data Challenge, 9th ARIAL Workshop, IJCAI-ECAI 2026

---

## Abstract

This study explores clinical outcome prediction using the MAISON-LLF dataset, which comprises multimodal sensor data collected from older adults recovering from lower-limb fractures in community settings. The study proposes a leakage-safe supervised regression pipeline for strict unseen-participant evaluation under Leave-One-Participant-Out Cross-Validation (LOPO-CV). The proposed approach uses rolling-window enrichment to convert daily sensor readings into short- and medium-term temporal summaries. It also evaluates an absolute-versus-delta target formulation, comparing absolute clinical-score prediction with change-from-baseline prediction using each participant's first available clinical assessment as the baseline.

ElasticNet, LightGBM, and Random Forest are compared against a Naive Mean baseline. In absolute-score prediction, LightGBM achieves the lowest MAE for the Social Isolation Scale, while ElasticNet achieves the lowest MAE for both the Oxford Hip Score and the Oxford Knee Score. However, the improvements over the Naive Mean baseline are modest. The delta-target formulation substantially reduces MAE across targets, but the Naive Mean Delta baseline remains strong. Overall, the findings suggest that participant mean-shift strongly affects cross-participant prediction in MAISON-LLF data, and that simple baselines are necessary for interpreting model performance in small wearable-sensor rehabilitation datasets.

---

## Dataset

The experiments use **MAISON-LLF v4**, an extended version of the MAISON-LLF dataset for monitoring older adults after lower-limb fractures in community settings.

- **Dataset record**: https://zenodo.org/records/17943110
- **MAISON-LLF Data Challenge page**: https://sites.google.com/view/arial2026/maison-data-challege
- **Version used in this study**: MAISON-LLF v4
- **Participants**: 18 older adults
- **Daily records**: 1,008 records
- **Observation period**: 56 days per participant
- **Modalities**: acceleration, heart rate, motion, position, sleep, and step data
- **Clinical targets**:
  - **SIS**: Social Isolation Scale
  - **OHS**: Oxford Hip Score
  - **OKS**: Oxford Knee Score

This repository does **not** redistribute the MAISON-LLF dataset. The `data/` folder is intentionally left empty and is included only to show where users should place the dataset locally.

After downloading the dataset, place the following files inside the local `data/` directory:

```text
data/maison-llf-features.csv
data/maison-llf-demographics.csv
data/maison-llf-feature-descriptions.xlsx
```

The dataset files should not be committed to this repository. Users should obtain the dataset from the official dataset record or the MAISON-LLF Data Challenge page and follow the dataset provider's conditions.

---

## Method overview

The workflow implements a leakage-safe supervised regression pipeline for strict unseen-participant evaluation using Leave-One-Participant-Out Cross-Validation (LOPO-CV).

The analysis focuses on two main strategies:

### 1. Rolling-window feature enrichment

Daily sensor readings are converted into short- and medium-term temporal summaries. For each safe daily sensor feature, rolling statistics are computed over 3-day, 7-day, and 14-day windows. This allows the models to use temporal recovery patterns rather than isolated daily sensor measurements.

### 2. Delta-target analysis

The study compares absolute clinical-score prediction with change-from-baseline prediction. In the delta-target setting, each participant's first available clinical assessment is used as the baseline, and the model predicts the change from that baseline.

This analysis is used to examine whether participant-level mean-shift is a major source of prediction error under LOPO-CV.

---

## Models

The notebook evaluates the following supervised regression models:

- ElasticNet
- Random Forest
- LightGBM

The models are compared against a leakage-safe **Naive Mean baseline**.

A tuned Ridge model is also included as an exploratory reference model in the absolute-versus-delta and statistical comparison results.

---

## Evaluation

The main evaluation uses **Leave-One-Participant-Out Cross-Validation (LOPO-CV)**.

- Each fold holds out one participant for testing.
- The model is trained on the remaining participants.
- Hyperparameter tuning is performed only inside the training participants using GroupKFold.
- Preprocessing, scaling, encoding, and feature construction are fitted only on training participants.

The main metric is:

- **MAE**: Mean Absolute Error

The secondary metric is:

- **R²**: Coefficient of determination

MAE is treated as the main metric because it is easier to interpret in clinical score units and is more stable under strict cross-participant evaluation.

---

## Statistical testing

The notebook reports non-parametric Friedman tests across the 18 LOPO folds. The Friedman test is used as an overall non-parametric comparison of model ranks for each target.

To keep the statistical comparison consistent, the same five models are compared in both the absolute-score and delta-target settings:

```text
Naive Mean
Ridge (tuned)
ElasticNet
Random Forest
LightGBM
```

When the Friedman test rejects the null hypothesis at the 0.05 significance level, Nemenyi post-hoc comparisons are reported using average model ranks and the critical difference. Since five models are compared, k = 5, N = 18, and q_alpha = 2.728, giving a critical difference of approximately CD = 1.44.

The Nemenyi test is used to identify which model-rank differences exceed the critical difference. The statistical testing supports a cautious interpretation of the model comparisons, since the fold-level standard deviations are large relative to the mean MAE differences between models.

---

## Repository structure

```text
.
├── README.md
├── requirements.txt
├── maison_llf_clinical_outcome_prediction.ipynb
├── data/
│   └── .gitkeep
└── outputs/
    ├── tables/
    └── figures/
```

---

## Installation

Create and activate a Python environment, then install the required packages:

```bash
pip install -r requirements.txt
```

Required packages include:

```text
numpy
pandas
scipy
scikit-learn
matplotlib
lightgbm
shap
openpyxl
jupyter
ipykernel
```

---

## Running the notebook

Open the notebook:

```text
maison_llf_clinical_outcome_prediction.ipynb
```

Then run all cells from top to bottom.

The notebook will:

1. Load the MAISON-LLF dataset files from the local `data/` directory.
2. Remove leakage-prone clinical and timestamp-related variables.
3. Generate rolling-window features.
4. Run LOPO-CV experiments for absolute-score prediction.
5. Run LOPO-CV experiments for delta-target prediction.
6. Save result tables and figures.
7. Run Friedman statistical tests and Nemenyi post-hoc comparisons using the same five-model comparison set.
8. Produce exploratory SHAP summaries for the LightGBM models.

---

## Main outputs

The notebook writes output files to the `outputs/` directory.

### Tables

```text
outputs/tables/lopo_metrics_summary.csv
outputs/tables/lopo_metrics_summary_delta.csv
outputs/tables/lopo_metrics_per_participant.csv
outputs/tables/lopo_metrics_per_participant_delta.csv
outputs/tables/friedman_overall_tests.csv
outputs/tables/nemenyi_average_ranks.csv
outputs/tables/nemenyi_posthoc_pairs.csv
outputs/tables/baselines.csv
```

### Figures

```text
outputs/figures/absolute_vs_delta_mae.png
outputs/figures/absolute_mae_summary.png
outputs/figures/model_comparison_sis.png
outputs/figures/model_comparison_ohs.png
outputs/figures/model_comparison_oks.png
outputs/figures/model_comparison_sis_delta.png
outputs/figures/model_comparison_ohs_delta.png
outputs/figures/model_comparison_oks_delta.png
outputs/figures/shap_summary_sis.png
outputs/figures/shap_summary_ohs.png
outputs/figures/shap_summary_oks.png
```

---

## Reproducibility note

The paper tables should be treated as the reference results. When rerunning the notebook, small numerical differences may occur depending on package versions, the local Python environment, and LightGBM behavior.

These small differences do not change the main conclusions:

- LightGBM performs best for SIS in the absolute-score setting.
- ElasticNet performs best for OHS and OKS in the absolute-score setting.
- Delta-target prediction reduces MAE substantially across targets.
- The Naive Mean Delta baseline remains strong.
- Strict cross-participant prediction remains difficult in this small-cohort LOPO-CV setting.

To reproduce the results as closely as possible:

1. Use the same MAISON-LLF dataset files.
2. Install dependencies from `requirements.txt`.
3. Run the notebook from a clean kernel.
4. Compare the generated summary tables with the reported manuscript tables.

---

## Citation

Citation details for this paper will be added after the official proceedings information becomes available.

---

## License and data-use notice

No software license is included in this repository unless a `LICENSE` file is added. The MAISON-LLF dataset remains subject to its own license and data-use terms. Users should obtain the dataset from the official dataset record or the MAISON-LLF Data Challenge page and follow the dataset provider's conditions.
