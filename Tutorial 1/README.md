# Assignment 1 — User Authentication using Biometric Features

**Course:** AI461 — Advanced Biometric Systems and Security
**Data:** `biomet_data.csv` (144 features × 1000 samples)

## 1. Data extraction

The provided CSV has **no header row** and is shaped **144 rows × 1000
columns** — i.e. each **row is one feature measured across all 1000
samples**, and each **column is one sample's full 144-dim feature vector**.
This is the transpose of the usual (samples × features) orientation, so
`biometric_analysis.ipynb` explicitly transposes the array before reshaping
it into `(100 users, 10 samples, 144 features)`:

- axis 0 → user (100 users)
- axis 1 → sample index (10 samples per user, in file order)
- axis 2 → feature vector (144 features)

**Ordering assumption:** columns are grouped user-major — columns 0–9 are
user 1's 10 samples, columns 10–19 are user 2's, and so on. This was verified
empirically: this grouping gives a much larger separation between genuine and
impostor score means (decidability index ≈ 1.76–1.93) than the alternative
sample-major interpretation (≈ 0.02), confirming user-major is the correct
layout.

> **Note on an earlier version of this analysis:** a prior pass parsed an
> equivalent PDF export of this same data row-by-row without transposing,
> which silently scrambled every feature vector (no error was thrown — the
> shapes still "worked," they were just meaningless). That produced near-random
> EER (~46–53%) and very low d′ (~0.1–0.2). This version corrects that by
> respecting the CSV's actual (features × samples) orientation, and also
> switches the FAR/FRR/ROC plots to log scale to match the assignment's
> reference diagram.

## 2. Train / test split and templates

- **Enrollment (training) set:** first 5 samples per user (samples 1–5)
- **Test set:** last 5 samples per user (samples 6–10, "6 months later")
- **Template:** one 144-d template per user, obtained by averaging the 5
  enrollment feature vectors (as shown in the assignment's pipeline diagram)

## 3. Matching / scoring

Every one of the 500 test samples (100 users × 5 test samples) is compared
against all 100 templates:

- **Genuine comparisons:** test sample vs. its own user's template → 500 scores
- **Impostor comparisons:** test sample vs. every other user's template →
  500 × 99 = 49,500 scores

Two matching distances are used:

| Metric | Similar means | Decision rule (accept) |
|---|---|---|
| Euclidean distance | small distance | `distance ≤ threshold` |
| Cosine similarity | high similarity | `similarity ≥ threshold` |

## 4. Outputs produced

Running `biometric_analysis.ipynb` top to bottom renders, per metric
(Euclidean and cosine):

- (A)/(B) Genuine vs. impostor score histograms
- (C) FAR vs. threshold (log scale)
- (D) FRR vs. threshold (log scale)
- (F, visualized) FAR/FRR crossing plot marking the EER point (log scale)
- (E) ROC curve, FAR axis on log scale
- A combined ROC comparing both metrics

Numeric results (EER, decidability index, score statistics) are saved to
`results.json` at the end of the notebook.

## 5. Equal Error Rate (F) and Decidability Index

EER is found as the threshold where `|FAR − FRR|` is minimized (the FAR/FRR
crossing point). The decidability index is computed as

```
d' = |mu_genuine − mu_impostor| / sqrt((var_genuine + var_impostor) / 2)
```

## 6. Results and which metric performs best

| Metric | EER | Decidability index (d′) |
|---|---|---|
| Euclidean | 11.6% | 1.76 |
| Cosine | 8.2% | 1.93 |

**Cosine similarity** gives both a lower EER and a higher decidability index
than Euclidean distance on this feature set, meaning it separates genuine
from impostor comparisons slightly more effectively here. Both metrics
perform well (d′ well above 1, EER in single/low-double digits), so either is
a reasonable choice — but cosine has the edge for this particular dataset.

## 7. How to reproduce

```bash
pip install pandas matplotlib numpy jupyter
jupyter nbconvert --to notebook --execute --inplace biometric_analysis.ipynb
```

This regenerates every plot and the `results.json` summary from
`biomet_data.csv`.

## Files in this submission

```
biometric_analysis.ipynb  # full pipeline: loading -> templates -> scoring -> plots -> EER/d' (pre-run, outputs embedded)
README.md                 # this file
results.json              # numeric summary (EER, d', score stats) per metric
biomet_data.csv            # source data (144 features x 1000 samples)
```
