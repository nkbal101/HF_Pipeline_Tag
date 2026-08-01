# Data Cleaning Steps: Pipeline Tag Prediction

**DATASCI 266 Final Project**
Source: `librarian-bots/model_cards_with_metadata` (Hugging Face Hub crawl, pulled July 2026)
Raw dataset size: 677,750 rows

This document walks through each cleaning step in order, what it does, why it's needed, and what came out of it. The final row counts reflect the most recent notebook run.

---

## Step 1: Load Raw Dataset

Loaded the full crawl from Hugging Face. Each row is one model repo, with columns for `modelId`, `pipeline_tag`, `card` (the raw README text), download counts, license, and a few other metadata fields.

**Result:** 677,750 raw rows.

Note: this crawl is not a full mirror of every model on the Hub (2M+), it's a curated subset. That's an accepted tradeoff for this project, not something the cleaning steps below try to fix.

---

## Step 2: Handle Missing Values

Dropped rows with no `pipeline_tag` (no label to train on) and rows with empty or null card text (nothing to learn from).

**Result:** 333,960 rows (49.3%) had no `pipeline_tag` and were dropped. 343,790 rows remain.

This is a known limitation of Hub metadata, not something the cleaning process can recover. Worth noting the missing rate came in about 9 points higher than the project proposal's original ~40% estimate, most likely because the crawl has grown since the proposal was scoped. The write-up should cite this run's number rather than the original estimate.

---

## Step 3: Check for Label Leakage

`pipeline_tag` is metadata that lives inside the raw card text (in a YAML header), so training on the raw text risks the model just pattern-matching the label back out of the input instead of learning from the actual description.

Checked two things separately:
- **Literal `pipeline_tag` field text** (always a real leakage artifact): found in 184,934 rows (53.8%) of the raw text.
- **The tag value itself appearing somewhere in the text** (often legitimate, e.g. a card that describes itself as a "translation model"): found in 255,707 rows (74.4%).

This step only measures the problem. It gets fixed in Step 4.

---

## Step 4: Strip YAML Frontmatter

Model cards start with a `---\n ... \n---` YAML block (license, tags, pipeline_tag, base_model) followed by the actual prose. Stripped that block out and kept only the prose for training. The removed YAML is kept in a separate column for reference, not discarded.

**Result:** 340,099 rows (98.9%) had a detected YAML block that got stripped.

Rechecked leakage on the cleaned text:
- Literal `pipeline_tag` field text: down to 2,804 rows (0.8%). This is a confirmed leakage artifact (a leftover metadata fragment the strip regex missed), not something legitimate. **These 2,804 rows were dropped.**
- Tag value present in prose: down to 46,024 rows (14.2% of what remained after the drop above). Most of these are legitimate, cards naturally describing what they do. A closer look found 14,249 of them with the tag value appearing near the very start of the text (first 200 characters), which is the subset most likely to still be a stray metadata fragment rather than genuine prose. These were flagged for manual review but not automatically dropped, since removing them without confirming would risk throwing out real training signal.

**Result after the literal-leak drop:** 340,986 rows.

---

## Step 5: Deduplicate Cards

Two duplication patterns are common in Hub crawls: exact duplicate cards across different model IDs (requantizations, LoRA reposts), and boilerplate cards that are 95%+ auto-generated template text with only the model name swapped in.

Caught both with a normalized-text hash: lowercase, strip the model's own name, strip digits, collapse whitespace, then hash. Handled two cases:
- Rows that normalize to a completely empty string (nothing left after stripping name/digits/punctuation) have no training signal at all, these were dropped outright rather than deduplicated.
- Everything else went through standard dedup, keeping one representative per duplicate group.

**Result:**
- 4,037 rows dropped for normalizing to empty content.
- 17,428 duplicate groups found, covering 186,170 rows (54.6% of what remained).
- 168,742 duplicate rows dropped, keeping one representative per group.
- **168,207 rows remain.**

Spot-checked the two largest duplicate groups (47,601 and 17,893 rows). Both are the default `transformers` push_to_hub template with unfilled placeholder text (`<!-- Provide a quick summary of what the model is/does. -->`). This confirms dedup is removing genuine boilerplate, not merging distinct cards. The middle of the duplicate-group size distribution hasn't been spot-checked yet, that's a reasonable next step but not a blocker.

---

## Step 6: Quality / Length Filtering

Filtered on three thresholds: character length between 100 and 100,000, and a minimum unique-word ratio of 0.15 (a proxy for boilerplate-heavy text with little unique content).

**Result:** 952 rows dropped, 167,255 remain.

Checked the unique-word-ratio distribution before trusting the threshold: mean 0.666, with even the 1st percentile sitting at 0.30, well above the 0.15 cutoff. Only 140 rows actually fell below it. In practice, this filter is doing almost nothing on the ratio axis, nearly all of the 952 dropped rows came from the character-length bounds instead. The 0.15 threshold could be raised (something in the 0.35-0.40 range) if a stricter quality filter is wanted later, but it isn't blocking training as is.

---

## Step 7: Top-10 Tag Selection & Class Balance

Selected the 10 most frequent `pipeline_tag` categories and filtered to just those, scoping the project to a manageable multi-class problem.

**Result:** 141,036 rows across 10 classes.

| pipeline_tag | rows |
|---|---|
| text-generation | 67,841 |
| text-to-image | 21,831 |
| image-text-to-text | 12,309 |
| text-classification | 10,218 |
| translation | 8,589 |
| automatic-speech-recognition | 5,829 |
| token-classification | 4,033 |
| sentence-similarity | 3,726 |
| robotics | 3,355 |
| image-classification | 3,305 |

Class imbalance is significant (text-generation is roughly 20x the smallest class), which is expected and is why macro-F1 is the evaluation metric for this project, not raw accuracy. `CAP_MAJORITY_CLASSES` is left off, imbalance is handled at training/evaluation time rather than by undersampling here.

Previewed minimum per-class training counts at the planned 25/50/75/100% data-scaling ablation (assuming an 80% train split). Smallest class at 25% scale is image-classification with 661 rows, above the 500-row floor considered risky for stable metrics, but the tightest of the ten. Worth watching this class specifically when the ablation results come in.

---

## Step 8-9: Final Summary & Save

Final row counts by stage:

| Stage | Rows | Dropped |
|---|---|---|
| Raw | 677,750 | — |
| After missing-value drop | 343,790 | 333,960 |
| After literal-leak drop | 340,986 | 2,804 |
| After dedup | 168,207 | 172,779 |
| After quality filter | 167,255 | 952 |
| After top-10 tag filter | 141,036 | 26,219 |

**Final retention: 141,036 of 677,750 rows (20.8%).**

Roughly half of the loss is unavoidable (no label to train on), about a third is duplicate/boilerplate removal that arguably helps more than it hurts (the biggest duplicate groups are confirmed template spam, and dedup disproportionately thins the majority class), and the rest is deliberate scoping to the top 10 tags.

Columns kept for training: `modelId`, `pipeline_tag`, `label`, `text` (cleaned prose), `char_len`, `word_count`.

Saved as:
- `./cleaned_data/model_cards_cleaned.parquet` (221.8 MB)
- `./cleaned_data/metadata.json` (label mappings, tag list, stage counts, filter thresholds)
- Copied to Google Drive: `/content/drive/MyDrive/266-pipeline-tag-prediction`

---

## What's Ready for Training

The cleaned dataset (141,036 rows, 10 balanced-enough classes, leakage-checked, deduplicated, quality-filtered) is ready to load into the training notebook for the TF-IDF baseline comparison and both transformer fine-tuning runs (RoBERTa-base, ModernBERT-base).

## Pending Quality Checks (not blockers, worth tracking)

- **14,249 rows** with the tag value appearing near the start of the text haven't been manually confirmed as legitimate prose vs. residual metadata. Flagged, not dropped.
- **Quality filter threshold** (0.15) is barely active given the real data distribution. Fine to leave as is, but could be tightened later.
- **Middle of the duplicate-group distribution** hasn't been spot-checked, only the two largest groups were verified as genuine boilerplate.

None of these affect the validity of moving forward with training now. They're worth resolving during error analysis if results come back surprising.
