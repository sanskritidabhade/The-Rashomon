# Lab Log - Study [3] (UCI-derived CVD diagnosis, Scientific Reports 2026)

**Author:** Ricardo Andrade (study lead) | **Project:** The Rashomon Effect in Cardiovascular Risk Prediction
**Related file:** `manifest_study3_uci_cvd.yaml` (IDs O1-O12 below match its obstacle log)

> **How this log was consolidated.** It is translated from the original working notes (written in Spanish). Entries were not dated when first written; dates are given only where known (`[date: ?]` = to be filled in). Statements in the original notes that later turned out to be wrong are corrected in place and listed in section 7. Entries marked *(added)* were written during consolidation from logged run outputs, not from the original notes.

---

## 1. Frozen starting point

- **Author's repository:** `https://github.com/SakibHasanSimanto/CVD-AI-Research`
  - Commit `9e10369b833643ddaacd54586e8f73d5046f8455` ("Update README.md"), author Kazi Sakib Hasan, dated Sun Jun 15 15:37:05 2025 +0600.
  - Cloned on branch `repro-ricar` (HEAD) from `origin/main`.
- **Repository contents:** README only. Code and data are not in the repository (obstacle O1).
- **Notebooks (Google Colab), accessed 2026-11-09:**

| Notebook | Role | URL |
|---|---|---|
| cvd dataset preparation | Clustering only; does not prepare the dataset | https://colab.research.google.com/drive/1fx3u_yBNG4aVKqZ_3VskwVZXbjuj7sM- |
| cvd modeling 1 | Not reviewed yet | https://colab.research.google.com/drive/1MNmdesc6JqmOuUXSyotzq_Tak5xXpvro |
| cvd modeling 2 | Not reviewed yet | https://colab.research.google.com/drive/1Zx0GTOdJYghEAIGale2AMWQAHmB1aaRH |
| cvd Regional bias and generalizations | Out of scope | https://colab.research.google.com/drive/1eQ3kwMdNflD6z9WryG_vYmR705CqribC |
| cvd SHAP DiCE | Out of scope | https://colab.research.google.com/drive/18tMdCHef42eE_bSUJLB2anWoTwU6kWFZ |
| cvd Tabular Neural | **Source of Table 5** | https://colab.research.google.com/drive/11bDv8obml4qHmCoT4-E0sGvnzQy3r8VV |
| cvd casuality | CGEVC and ATE estimates | https://colab.research.google.com/drive/1iKTXCDyvLg-MjmVhYhY8uMvoltykGe0H |
| cvd Tabular Neural GC | Gaussian Copula variant, not described in the paper | https://colab.research.google.com/drive/1NBe0NirCsf-68UjmJXaYqZyu2F2InFx9 |
| cvd Gender Bias mitigation | CTGAN step; writes the final CSV | https://colab.research.google.com/drive/17zliIkXw4MlEyH696gB8UzZKju_ohdQg |

- **Handling rule:** original notebooks are never edited. Work is done in personal copies ("Save a copy in Drive"); originals are kept as downloaded `.ipynb` files in `original_notebooks/`.

## 2. Data verification

- Datasets used: two Kaggle files (Cleveland, 303 rows; merged Statlog/Cleveland/Hungary/Switzerland/Long Beach, 1190 rows) and three CSVs from the author's Drive folder.
- **Result: the datasets are correct.** The 1190-row Kaggle file minus 272 exact duplicates gives 918 rows (508 with disease, 410 without), identical (ignoring row order) to the author's `combined_statslogExcluded.csv` and matching the paper (Fig. 3).
- The SMOTE file has 1016 rows (508/508). The final CTGAN file has **1542 rows** (916/626; sex 771/771), of which about 624 rows (about 40 %) are not identical to any real row.
- Files are fingerprinted with SHA-256 (`sha256sum`) so that anyone can confirm they use the exact same inputs. Hashes are in the manifest (`data_lineage`).

## 3. Obstacle log

| ID | Category | Description | Time |
|---|---|---|---|
| O1 | code / environment | Code is available but lives outside the repository (Colab and Drive), and versions are not pinned. | not recorded |
| O2 | environment | Every install is an unpinned `pip install`. | not recorded |
| O3 | environment | `rtdl` (FT-Transformer) requires `torch<2`, incompatible with current Colab. FT-Transformer postponed; not required for the main target. | 10 min |
| O4 | environment / access | Current TabPFN (9.2.0) requires an account, an accepted license and an API token to download weights. The author's April 2025 run did not need this (only an `HF_TOKEN` warning). A teammate repeating the run will have to do the same setup. | 10 min |
| O5 | environment | Unpinned `tabpfn` installs a newer model generation (weights `tabpfn-v3.5-20260909`); the result changes from 0.8997 (v2 series) to 0.9191. | not recorded |
| O6-O10 | method | Data leakage and protocol issues, see section 5. | n/a |
| O11 | data | The notebook that creates the SMOTE file is not identified yet; CTGAN has no seed, so the final CSV cannot be regenerated identically. | not recorded |
| O12 | code | The notebook called "dataset preparation" does not prepare the dataset. | not recorded |

## 4. Runs

**Environment of our runs:** Google Colab, CPU runtime, Python 3.13.18, torch 2.14.1+cpu, scikit-learn 1.6.1, pandas 2.2.3, numpy 2.4.6, pytorch-tabnet 4.1.0. The author's run (2025-04-19) used Python 3.11 and numpy 1.26.4.
**Split used in every run:** stratified 80/20, `random_state=123`, 309 test rows (125 without disease, 184 with disease).

### R1 - TabPFN, current version (A1)
- `tabpfn 9.2.0`, weights `tabpfn-v3.5-20260909.safetensors`, `random_state=123`.
- Accuracy **0.9191** (284/309), F1 **0.9337**. Delta vs. paper (0.90): 0.0191. The author's stored output is 0.9029.
- Two runs gave identical results.
- Time to reproduce: 4 hours.

### R2 - TabPFN, v2 series (A2) *(added)*
- `tabpfn 2.0.8`, chosen as the estimated version of the author's run (criterion: latest PyPI release on or before 2025-04-19; **[confirm]**). Checkpoint file name: **[to be recorded]**.
- Accuracy **0.8997** (278/309), F1 **0.9178**. Delta vs. paper: 0.0003; vs. the author's stored output: one row.

### R3 - TabNet with the author's hyperparameters (Optuna not re-run) *(added)*
- Hyperparameters taken from the hard-coded final cell of the author's notebook.
- Runs 1, 2 and 3 gave identical results: accuracy **0.8900** (275/309), F1 **0.9066**. The author's stored output is 0.8867 (one row difference).
- Run 1 report:

```
              precision    recall  f1-score   support
           0       0.85      0.88      0.87       125
           1       0.92      0.90      0.91       184
    accuracy                           0.89       309
```

### R4 - CGEVC (causal-guided voting ensemble) *(added)*
- Features: exang, slope, fbs, cp, sex, oldpeak (hard-coded by the author). The ATE cells were not re-run; the author's stored values match the paper.
- Accuracy **0.8835** (273/309), precision 0.8854. The author's stored output is 0.8867.

### Preliminary Rashomon observation *(added)*
On the same 309 test rows, TabPFN (v2), TabNet and CGEVC span accuracy 0.8835-0.8997, yet their labels differ for **33 of 309 cases (10.7 %)**; all three agree and are wrong in 17 cases. This was obtained under the author's leaky protocol and must be redone in Level C.

## 5. Findings about the paper and its code

1. **Data leakage.** SMOTE and CTGAN generate synthetic rows before the train/test split, so a synthetic row interpolated from a test patient can end up in training, and vice versa. The reported 0.90 is probably inflated. (O6)
2. **Synthetic patients in the test set.** Evaluation uses 309 rows, a substantial share of which are CTGAN rows, not real patients. (O6)
3. **Size mismatch.** 309 rows is 20 %, so the dataset has about 1,540 rows; Table 1 of the paper says 1016. Confirmed: 1542. (O8)
4. **Feature mismatch.** The notebook uses all 11 features; Table 1 says the clinical dataset has 4. (O8)
5. **TabNet is tuned against the test set.** Optuna maximises test accuracy (best trial 0.9061) and early stopping also uses the test set; there is no validation set. (O7)
6. **Final TabNet is not the Optuna model.** The code comment says "full precision to replicate best score", but the configuration differs: `batch_size` 48 vs 1024, `virtual_batch_size` 32 vs 128, no scheduler, no explicit seed, early stopping on AUC instead of accuracy. Result 0.8867 vs 0.9061. (O9)
7. **CGEVC feature selection leaks.** ATE estimates and the choice of 6 features are computed on the full dataset, including future test rows, synthetic rows and the target; the split comes afterwards. (O10)
8. **CGEVC threshold is inconsistent.** The paper says ATE >= 0.1; the notebook text says "|ATE| >= 1"; the selection includes `oldpeak` (ATE 0.0926, below 0.1) and excludes `restecg` (0.0375). The final model uses 6 of the 11 variables. (O10)
9. **Unweighted voting.** Equation 7 of the paper describes a weighted average with weights `w_m`; the code uses soft voting without weights. (O10)
10. **Unseeded CTGAN.** `CTGANSynthesizer(metadata)` is created without a seed, so the final CSV cannot be regenerated identically; the author's CSV is required. (O11)

## 6. Decisions

- FT-Transformer postponed (O3); BNN not attempted (0.03 below the best model, outside the 0.02 similarity rule).
- The author's published augmented CSV is used for the faithful reproduction (Level A). A rebuild of SMOTE -> CTGAN would be labelled a reconstruction (Level B); a leakage-free redesign (Level C) is a separate controlled experiment, not part of the reproduction status.
- Proposed status: reproduced with caveats; final status to be decided by the team after cross-review.

## 7. Corrections to the original notes

| Original note (translated) | Correction |
|---|---|
| "No seeds. CTGAN has none, and neither do TabPFN or FT-Transformer. Do not expect the same exact number." | Only the CTGAN claim stands (verified in the notebook). TabPFN gave identical results across repeated runs, so it is deterministic in our environment. FT-Transformer was not run. |
| "The final run has no seed. Your result will vary slightly between runs." | There is no explicit seed, but three runs gave identical results, probably because the library default seed is used (not verified in the library source). |
| "TabPFN is probably v2, not v1." | Updated: the author's run used a v2-series model; v2 (2.0.8) gave 0.8997. |
| Run time "1 hour" for R1 | Conflicts with the manifest (240 min); to be reconciled. |
| Entries written as instructions ("note the time", "sha256sum") | Rewritten as log entries. |

## 8. Open items

- [ ] Identify the notebook that produces the SMOTE file (modeling 1 or 2).

