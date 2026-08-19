# PADV activity 1

# Predicting Anemia for Community Healthcare Screening

A machine learning project that flags individuals likely to have anemia using
routine blood test data, to support referral and triage decisions in
community healthcare settings.

## Overview

Anemia is common but often under-diagnosed, especially where access to full
diagnostic workups is limited. This project builds and compares two
classification models — a Decision Tree and a Support Vector Machine (SVM) —
that predict anemia status from complete blood count (CBC) indices and
nutritional biomarkers, using the SKILICARSLAN Anemia Dataset (15,300
records, 29 attributes).

The goal is a **screening tool, not a diagnostic replacement**: it flags
high-risk individuals from an inexpensive CBC panel so they can be
prioritized for confirmatory testing and clinical referral.

## Repository Contents

| File | Description |
|---|---|
| `anemia_prediction_notebook.ipynb` | End-to-end Jupyter notebook: data cleaning, EDA, model training, and evaluation |
| `Anemia_Prediction_Report.pdf` | 8-page written report covering problem statement, methodology, results, and conclusions |
| `anemia_visuals_A4_page1.png` | Print-ready A4 page — EDA charts (4 panels) |
| `anemia_visuals_A4_page2.png` | Print-ready A4 page — model evaluation charts (3 panels) |
| `SKILICARSLAN_Anemia_DataSet.xlsx` | Source dataset (not included in this repo — see Data section) |

## Data

The dataset contains CBC parameters (WBC, RBC, HGB, HCT, MCV, MCH, MCHC, RDW,
PLT, and differential counts), nutritional biomarkers (Ferritin, Folate,
B12), gender, and pre-derived anemia labels (`All_Class` and per-subtype
columns for HGB, iron, folate, and B12 anemia).

The four anemia subtypes were consolidated into a single binary target,
`Target` (1 = any anemia present, 0 = healthy), reflecting the practical
screening question: *does this person need referral?*

**Anemia prevalence in the cleaned dataset: 36.4%**

## Methodology

### Data Cleaning
- Duplicate records removed.
- Biologically implausible outliers (e.g. MCHC > 45 g/dL, WBC > 50×10⁹/L)
  capped at the 1st/99th percentiles rather than dropped, to preserve sample
  size in the minority (anemic) class.

### Feature Set — and a Deliberate Exclusion
Ten features were used for modeling: `GENDER`, `WBC`, `RBC`, `MCV`, `MCH`,
`MCHC`, `RDW`, `FERRITTE`, `FOLATE`, `B12`.

**Hemoglobin (HGB) and hematocrit (HCT) are intentionally excluded** from the
model inputs. The anemia label is clinically defined by hemoglobin
thresholds, so including HGB as a predictor would let the model simply
recover the labeling rule instead of learning a genuinely predictive pattern
from indirect indicators — the kind of signal actually available in a
pre-screening context, before a confirmatory hemoglobin test is run. (An
earlier version of this analysis included HGB and produced a trivial 100%
accuracy for exactly this reason; it has been corrected here.)

### Models
- **Decision Tree** (`max_depth=5`, `class_weight='balanced'`) — chosen for
  interpretability; its rules can be read directly and turned into a simple
  checklist for community health workers.
- **SVM** (RBF kernel, `class_weight='balanced'`, features standardized) —
  chosen for its ability to capture non-linear relationships between
  correlated blood indices.

Both were trained on a stratified 75/25 train/test split (`random_state=42`).

## Results

| Model | Accuracy | Precision | Recall | F1-Score | ROC-AUC |
|---|---|---|---|---|---|
| Decision Tree | 0.931 | 0.901 | 0.912 | 0.906 | 0.981 |
| **SVM** | **0.973** | **0.938** | **0.993** | **0.965** | **0.999** |

The **SVM is the recommended model**. It outperforms the Decision Tree on
every metric and, critically, achieves 99.3% recall — missing only 10 of
1,385 true anemia cases in the test set. For a screening tool, missed cases
(false negatives) are far costlier than unnecessary follow-up tests (false
positives), making recall the metric that matters most here.

**RBC and MCH** emerged as the dominant predictors once HGB was excluded,
together accounting for ~93% of the Decision Tree's splitting importance —
consistent with established clinical understanding of anemia.

## How to Use

1. Open `anemia_prediction_notebook.ipynb` in Jupyter.
2. Place `SKILICARSLAN_Anemia_DataSet.xlsx` in the same directory as the
   notebook.
3. Run all cells in order. Each cell is self-contained and commented.
4. See `Anemia_Prediction_Report.pdf` for the full write-up, or the two A4
   PNGs for print-ready visualizations.

### Requirements
```
pandas
numpy
matplotlib
seaborn
scikit-learn
openpyxl
```

## Limitations

- Anemia labels are pre-derived from lab thresholds in the source dataset,
  not independently confirmed clinical diagnoses.
- Performance should be validated on a new patient population before any
  real-world deployment.
- This is a triage/screening aid, not a diagnostic tool — positive
  predictions should always be followed by confirmatory clinical testing.

## Future Work

- Multi-class subtype prediction (iron- vs. folate- vs. B12-deficiency
  anemia) to suggest *which* follow-up test or supplement is most relevant.
- External validation against confirmed clinical diagnoses.
- A simplified, offline decision aid derived from the Decision Tree for use
  without computational tools.
 
