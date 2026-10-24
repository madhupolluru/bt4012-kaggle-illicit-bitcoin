# BT4012 Kaggle Competition 2026: Illicit Bitcoin Transaction Detection

This repository contains the code for my BT4012 Kaggle competition submission. The task is to predict the probability that each Bitcoin transaction in a later time period (steps 36 to 49) is illicit, using the Elliptic transaction graph. Submissions are scored by ROC-AUC.

The notebook shows the full progression from my first Kaggle submission to the final model. Each step trains a model, writes the submission file that was uploaded, and records its Kaggle public score.

**Final model:** 30 LightGBM models (3 configurations x 10 seeds, rank-averaged), trained on the 162 transaction features (three time-proxy columns removed) plus visible-neighbour graph features: 1-hop incoming and outgoing neighbour means, differences between each transaction and its neighbourhood, and 2-hop neighbour means. Public AUC: **0.95915**.

## Kaggle progression

| Step | Submission file | Idea tested | Public AUC |
|---|---|---|---|
| 1 | submission_main.csv | MLP-heavy blend (best on validation) | 0.92846 |
| 2 | submission_lgb_notime.csv | LightGBM alone, time-proxy features removed | 0.94401 |
| 3 | submission_rf_notime.csv | Random Forest | 0.91961 |
| 4 | submission_lgb_goss.csv | Best LightGBM from the validation tuning sweep | 0.94320 |
| 5 | submission_lgb_flexible.csv | Flexible instead of cautious LightGBM | 0.94811 |
| 6 | submission_flex_ensemble.csv | Ensemble of flexible models with recent training windows | 0.95268 |
| 7 | submission_graph_1hop.csv | + 1-hop neighbour features | 0.95696 |
| 8 | **submission_graph_ensemble.csv** | **+ 2-hop neighbour features (final model)** | **0.95915** |
| 9 | submission_graph_plus_std.csv | + neighbour spread (rejected) | 0.95885 |

## Repository contents

- `e0969403.ipynb`: the full pipeline. Exploratory data analysis, preprocessing (removal of time-proxy features), forward-in-time validation, the nine modelling steps above, results and feature importance. The notebook is saved with its outputs.
- `requirements.txt`: Python dependencies.

## How to run

1. Download the competition data and place `train.csv`, `test.csv`, `txs_edgelist.csv` and `sample_submission.csv` in a folder called `data/` next to the notebook. The data is not included in this repository.
2. Install the dependencies (tested with Python 3.13):
   ```
   pip install -r requirements.txt
   ```
3. Open `e0969403.ipynb` in Jupyter and run all cells. A full run takes about 30 minutes on a laptop.
4. All nine submission files are written to `submissions/`. The final submission is `submissions/submission_graph_ensemble.csv`.

## Notes on leakage

- Models are trained only on the labelled rows of `train.csv`.
- `time_step` and the three features that encode it (feat_100, feat_101, feat_103) are never model inputs.
- Graph features are computed only over transactions that are visible in both training and test data (labelled training transactions and test transactions), so they are built the same way for both. Every edge links two transactions in the same time step, so no feature uses information from a later period.
