# Assignment 1 — User Authentication using Biometric Features

**Course:** AI461 — Advanced Biometric Systems and Security
**Data:** `biomet_data.pdf` (100 users × 10 samples × 144 features)

## 1. Data extraction

The raw feature values were supplied as scientific-notation numbers laid out
across 668 PDF pages (6 numbers per line, no headers). `biometric_analysis.py`
parses every number with a regex (`pdfplumber` for text extraction) and
reshapes the flat list of 144,000 values into a `(100, 10, 144)` array:

- axis 0 → user (100 users)
- axis 1 → sample index (10 samples per user, in file order)
- axis 2 → feature vector (144 features)

**Ordering assumption:** the file lists all 10 samples of user 1, then all 10
samples of user 2, and so on (user-major order). This was verified
empirically: reshaping the data this way gives a much larger separation
between genuine and impostor score means (decidability index ≈ 0.21) than the
alternative sample-major interpretation (≈ 0.02), confirming user-major is
the correct layout.

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

## 4. Outputs produced (`plots/`)

| File | Description |
|---|---|
| `euclidean_genuine_impostor_dist.png` | (A)/(B) Genuine vs. impostor score histograms — Euclidean |
| `cosine_genuine_impostor_dist.png` | (A)/(B) Genuine vs. impostor score histograms — Cosine |
| `euclidean_far_frr.png` | (C)/(D) FAR & FRR vs. threshold — Euclidean |
| `cosine_far_frr.png` | (C)/(D) FAR & FRR vs. threshold — Cosine |
| `euclidean_roc.png` | (E) ROC curve — Euclidean |
| `cosine_roc.png` | (E) ROC curve — Cosine |
| `combined_roc.png` | (E) ROC curve, both metrics overlaid for comparison |

Numeric results (EER, decidability index, score statistics) are written to
`results.txt` and `results.json`.

## 5. Equal Error Rate (F) and Decidability Index

EER is found as the threshold where `|FAR − FRR|` is minimized (the FAR/FRR
crossing point). The decidability index is computed as

```
d' = |mu_genuine − mu_impostor| / sqrt((var_genuine + var_impostor) / 2)
```

See `results.txt` for the exact numbers from this run.

## 6. Which metric performs best?

Based on the ROC curves and EER/d' values in `results.txt`, **Euclidean
distance** gives a lower EER and a higher decidability index than cosine
similarity on this feature set, meaning it separates genuine from impostor
comparisons more effectively here. Its ROC curve also lies above the cosine
curve (and above the chance diagonal) across most of the operating range.
Both metrics show substantial genuine/impostor overlap, which caps the
achievable EER — but Euclidean distance is the more discriminative of the two
for this particular feature set.

## 7. How to reproduce

```bash
pip install pdfplumber matplotlib numpy
python3 biometric_analysis.py
```

This regenerates every plot in `plots/` and the `results.txt` / `results.json`
summary from the original `biomet_data.pdf`.

## Files in this submission

```
biometric_analysis.py    # full pipeline: parsing → templates → scoring → plots → EER/d'
README.md                # this file
results.txt              # numeric summary (EER, d', score stats) per metric
results.json             # same summary, machine-readable
plots/                   # all required figures (A–E)
```
