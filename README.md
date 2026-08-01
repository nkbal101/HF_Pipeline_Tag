# Automatic Pipeline Tag Prediction from Model Card Text

**DATASCI 266: Natural Language Processing with Deep Learning**
UC Berkeley, School of Information — Summer 2026

Novejot Kaur Bal ([nkbal@berkeley.edu](mailto:nkbal@berkeley.edu)) · Noshen Nooren Habib ([noshen@berkeley.edu](mailto:noshen@berkeley.edu))

## Overview

Roughly half of the models on the Hugging Face Hub have no `pipeline_tag`, the metadata field that declares a model's primary task and powers search, filtering, and downstream recommendation systems. This project tests whether a model's card text alone is enough to automatically recover a missing `pipeline_tag`, using a TF-IDF + logistic regression baseline and fine-tuned RoBERTa and ModernBERT classifiers across the ten most frequent pipeline tags. We evaluate all three models on held-out labeled data, run a data-scaling ablation (25/50/75/100% of training data), and apply the fine-tuned models to a pool of ~115,000 previously untagged model cards to check whether the gains hold up on genuinely unlabeled data. Full methodology, results, and discussion are in the final report.

## Repository Structure

| Path | Description |
|---|---|
| `266_Summer2026_Final_Report_Automatic_Pipeline_Tag_Prediction.docx` | Final project report. |
| `00_NB_Baseline_Final.ipynb` | **Final** baseline model: TF-IDF + logistic regression training, evaluation, and error analysis. |
| `00_NB_RoBERTa_Final.ipynb` | **Final** RoBERTa-base fine-tuning notebook (512-token truncation). |
| `00_NB_ModernBERT_512_Tokens_Final.ipynb` | **Final** ModernBERT-base fine-tuning notebook (512-token truncation). |
| `NH_data_cleaning.ipynb` / `NH_Data_Cleaning_Steps.md` | Data cleaning pipeline (missing-value handling, leakage checks, YAML stripping, deduplication, quality filtering, top-10 tag selection) and a written walkthrough of each step with row counts. |
| `NB_EDA.ipynb` | Exploratory data analysis on the cleaned labeled dataset. |
| `NB_Plot_Per_Class_F1_Results.ipynb` | Generates the per-pipeline-tag F1 comparison figures used in the report. |
| `NB_Plot_Data_Ablation_Results.ipynb` | Generates the data-scaling ablation (macro-F1 vs. training set size) figure. |
| `NH_pipeline_tag_definitions.md` | Reference definitions and examples for each of the ten pipeline tags used in this project. |
| `Baseline_Model_Artifacts/` | Saved baseline artifacts: fitted TF-IDF vectorizer, label encoder, and logistic regression model (`.joblib`). |
| `Untagged_Data/` | Cleaning, inference, and comparative EDA notebooks for the unlabeled-inference experiment: preparing the untagged model card pool, running all three trained models against it, and comparing their confidence and agreement. |
| `Archive/` | Earlier, superseded notebook versions (e.g. sliding-window RoBERTa experiments) kept for reference; not used in the final results. |
| `NH_Baseline_Final_with_model_saved.ipynb` | Earlier baseline run used to generate the saved `Baseline_Model_Artifacts/`; superseded by `00_NB_Baseline_Final.ipynb` for reported results. |

The `00_`-prefixed notebooks are the ones that produced the model performance numbers reported in the final paper. Everything else supports, documents, or extends that core pipeline.

## What's Not Included

- **RoBERTa and ModernBERT model artifacts** (fine-tuned weights, tokenizer files, checkpoints) are not included in this repository — they exceed GitHub's file size limits. Only the baseline's lightweight `.joblib` artifacts are checked in.
- **Datasets** (raw crawl, cleaned labeled/untagged parquet files, ablation splits) are not included for the same reason. All datasets used across these notebooks are available and can be shared on request.

## Data Source

[`librarian-bots/model_cards_with_metadata`](https://huggingface.co/datasets/librarian-bots/model_cards_with_metadata), a continuously updated Hugging Face Hub crawl. Because the source dataset grows over time, notebooks pulled at different points in the project reflect slightly different raw row counts; see `NH_Data_Cleaning_Steps.md` for the cleaning run tied to the reported results.

## Reproducing This Work

All notebooks were developed and run in Google Colab with data and model artifacts stored on Google Drive (`/content/drive/MyDrive/266-pipeline-tag-prediction`). To rerun a notebook, mount your own Drive, update `DRIVE_DIR` to a folder containing the required input file(s) (noted at the top of each notebook), and run cells in order. Key dependencies: `transformers`, `datasets`, `scikit-learn`, `pandas`, and `torch`.
