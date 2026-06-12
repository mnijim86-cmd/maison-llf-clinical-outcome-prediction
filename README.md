# MAISON-LLF Clinical Outcome Prediction

This repository contains a reproducible notebook for clinical outcome prediction using the MAISON-LLF dataset. The workflow implements a leakage-safe supervised regression pipeline for strict unseen-participant evaluation using Leave-One-Participant-Out Cross-Validation (LOPO-CV).

The analysis focuses on two main strategies:

1. **Rolling-window feature enrichment**: converts daily sensor readings into short- and medium-term temporal summaries.
2. **Delta-target analysis**: compares absolute clinical-score prediction with change-from-baseline prediction.

The notebook evaluates ElasticNet, LightGBM, and Random Forest models against a Naive Mean baseline for predicting three clinical outcome scores. A tuned Ridge model is also included as an exploratory reference model in the absolute-versus-delta and statistical comparison results:

- **SIS**: Social Isolation Scale
- **OHS**: Oxford Hip Score
- **OKS**: Oxford Knee Score

---

## Repository structure

```text
.
├── README.md
├── requirements.txt
├── maison_llf_clinical_outcome_prediction.ipynb
├── data/
│   └── README.md
└── outputs/
    ├── tables/
    └── figures/
```

---

## Dataset

This repository does **not** redistribute the MAISON-LLF dataset. Users should obtain the dataset from the official dataset record or the MAISON-LLF Data Challenge page.

- Dataset record: https://zenodo.org/records/17943110
- MAISON-LLF Data Challenge page: https://sites.google.com/view/arial2026/maison-data-challege

After downloading the dataset, place the following files inside the `data/` directory:

```text
data/maison-llf-features.csv
data/maison-llf-demographics.csv
data/maison-llf-feature-descriptions.xlsx
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

1. Load the MAISON-LLF dataset files.
2. Remove leakage-prone clinical and timestamp-related variables.
3. Generate rolling-window features.
4. Run LOPO-CV experiments for absolute-score prediction.
5. Run LOPO-CV experiments for delta-target prediction.
6. Save result tables and figures.
7. Run Friedman statistical tests and Nemenyi post-hoc comparisons using the same five-model comparison set: Naive Mean, Ridge, ElasticNet, Random Forest, and LightGBM.
8. Produce exploratory SHAP summaries for LightGBM.

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

## Statistical testing

The notebook reports non-parametric Friedman tests across the 18 LOPO folds. The Friedman test is used as an overall non-parametric comparison of model ranks for each target.

To keep the statistical comparison consistent, the same five models are compared in both the absolute-score and delta-target settings:

Naive Mean
Ridge (tuned)
ElasticNet
Random Forest
LightGBM

When the Friedman test rejects the null hypothesis at the 0.05 significance level, Nemenyi post-hoc comparisons are reported using average model ranks and the critical difference. Since five models are compared, (k=5), (N=18), and (q_{\alpha}=2.728), giving a critical difference of approximately (CD=1.44).

The Nemenyi test is used to identify which model-rank differences exceed the critical difference. The statistical testing supports a cautious interpretation of the model comparisons, since the fold-level standard deviations are large relative to the mean MAE differences between models.

---

## Citation


---

## License and data-use notice

No software license is included in this repository unless a `LICENSE` file is added. The MAISON-LLF dataset remains subject to its own license and data-use terms. Users should obtain the dataset from the official dataset record or the MAISON-LLF Data Challenge page and follow the dataset provider's conditions.