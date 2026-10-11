# Centralized, Local and Federated Intrusion Detection on UNSW-NB15

Code and results for the paper **"Centralized, Local and Federated Intrusion Detection on UNSW-NB15 under Temporal Evaluation and Client Heterogeneity: A Budget-Matched, Audited Empirical Study"** by Abyaz Ashiq and Asmita Padmanabhan.

This repository contains the Colab notebook that runs every experiment in the paper and the full set of result files it produced (tables, figures, per-run outputs, manifest and audit files).

## Research question

How does IDS generalization change when centralized, local and federated (FedAvg) learning are evaluated under temporal distribution shift and increasing client-level data heterogeneity?

## What is in this repository

```
.
├── UNSW_NB15_Publication_Experiment_v4.ipynb   # the full experiment (Colab notebook)
├── UNSW_NB15_Publication_Experiment_v4.pdf     # printed copy of the notebook with outputs (read-only)
├── results_publication/
│   ├── tables/                  # CSV tables (summary, paired tests, per-family, heterogeneity, lineage)
│   ├── figures/                 # fig1 to fig6 (PNG)
│   ├── runs/                    # one JSON per (seed, condition) unit; makes the run resumable
│   ├── experiment_manifest.json # configuration, file hashes, software versions, verification gates
│   ├── leakage_audit.json       # train/validation/test overlap and leakage checks
│   └── final_report.txt         # text summary produced at the end of the run
├── README.md
└── LICENSE
```

## Data

The raw data are **not** included in this repository. Download the public UNSW-NB15 dataset from the official page of the UNSW Canberra Cyber Range Lab:

https://research.unsw.edu.au/projects/unsw-nb15-dataset

You need the four raw CSV parts (`UNSW-NB15_1.csv` to `UNSW-NB15_4.csv`, 2,540,047 flows in total). Place them in:

```
UNSW_Project/data/raw/
```

The files have no header row. The notebook applies the published 49-column schema and checks the column count of every file.

To confirm you have the same files used in the paper, compare their SHA-256 hashes (the notebook also verifies and records these):

| File | Rows | SHA-256 |
|---|---|---|
| UNSW-NB15_1.csv | 700,001 | `7d851bbeabd27894ce39c8e78835c73341fc946652fb7743b9eff193b55eb511` |
| UNSW-NB15_2.csv | 700,001 | `6130ad02873cc6069ae695cf2844f2e8c2e9a9a1b7532dd82ab8f202757cacf8` |
| UNSW-NB15_3.csv | 700,001 | `ae990a96c3dfcd425ce2801aadb1727a34d5e0ae6d8215dbcdae60dedfaef640` |
| UNSW-NB15_4.csv | 440,044 | `cdf563692d51d405541dd659ddcdad9fa01f001f05fe9fc4b67f00ca12fbc96a` |

## How to run

### In Google Colab (as used for the paper)

1. Upload the four raw CSV files to your Google Drive at `MyDrive/UNSW_Project/data/raw/`.
2. Open `UNSW_NB15_Publication_Experiment_v4.ipynb` in Colab (File, then Upload notebook).
3. Choose **Runtime, then Run all**. The notebook mounts your Drive and writes all outputs to `MyDrive/UNSW_Project/results_publication/`.
4. The full protocol took about **90 minutes** (89.6 min in our run).

**The run is resumable.** Each (seed, condition) unit is saved to `results_publication/runs/`, and file names carry the notebook version. If Colab disconnects, run all cells again and finished units are skipped.

### Quick pipeline check

To test that everything works end to end before the full run, set the environment variable `UNSW_PROFILE=smoke` (2 seeds, 2 alphas, small caps). The default profile is `full`.

### Outside Colab

Set `UNSW_PROJECT` to your project folder (default `./UNSW_Project`) and run the notebook with Jupyter. The Google Drive step is skipped automatically when `google.colab` is not available.

## What the notebook runs

| Condition | Centralized | Local | FedAvg |
|---|---|---|---|
| Random stratified split (60/20/20), baseline | yes | | |
| Temporal split (60/20/20, chronological) | yes | | |
| Temporal + Dirichlet alpha = 10 / 1 / 0.5 / 0.1 | yes (pooled, budget-matched) | yes | yes |
| Duplicate sensitivity (kept vs removed) | yes | | yes |
| Auxiliary: unseen attack-family generalization (DoS, Reconnaissance) | yes | yes | yes |
| Temporal split-fraction sensitivity (50/25/25, 70/10/20) | yes | | |

Every condition is repeated over **5 seeds: 42, 123, 456, 789, 101112**.

Key design choices (full detail in the paper):

- Same MLP (64-32, ReLU, Adam, learning rate 1e-3, batch size 256) for every method.
- Paradigm comparison uses the three budget-matched methods (centralized-matched, local, FedAvg), each with 10 epochs of `partial_fit` (FedAvg: 10 rounds x 1 local epoch). The early-stopping centralized model is a labelled reference only.
- Binary detector (benign vs attack). Metrics: recall, FPR, PR-AUC, precision, Macro-F1. Per-family results report detection recall only.
- 5 clients; one pure Dirichlet draw per (seed, alpha); training sample capped at 300,000 flows.
- Exact duplicates (features + label + family) removed for the primary analysis (513,438 flows, 20.21%); a kept-duplicates ablation is included.
- Paired statistics over seeds: mean difference with 95% t-interval and Hedges' g (primary), plus paired t-test and Wilcoxon with Holm correction (secondary).
- The validation block is reserved and never used for model selection or tuning.
- Preprocessing is shared across clients, so this is not a privacy-preserving federated pipeline.

## Software versions

| Package | Version |
|---|---|
| scikit-learn | 1.6.1 |
| pandas | 2.2.3 |
| NumPy | 2.1.3 |
| SciPy | 1.16.3 |

The exact Python version and platform string are recorded in `results_publication/experiment_manifest.json`.

## Verification gates

The run enforces ten verification gates, all of which passed: all work units completed; five seeds present; all metrics finite; identical test set for every method within a condition and seed; preprocessing fitted on training rows only; no duplicate or index overlap between train, validation and test in the deduplicated splits; finite FedAvg weights; static checks that no identifier, timestamp, session or target-derived column is a feature; and FedAvg results at every alpha.

## Scope and limitations

Results hold for one dataset (UNSW-NB15), one architecture (a 64-32 MLP), one aggregation rule (FedAvg), five clients, ten rounds and five seeds. The experiments do not measure concept drift, demonstrate privacy preservation or support zero-day detection claims. See Section 6 of the paper for the full list.



Code is released under the MIT License (see `LICENSE`). The UNSW-NB15 data is not redistributed here and remains subject to its own terms on the official UNSW page.

## Contact

Open an issue on this repository or contact the authors at [email to be added].
