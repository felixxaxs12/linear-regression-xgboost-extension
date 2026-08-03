# Final integrity audit: two-stage XGBoost extension

**Audit date:** 2026-08-03  
**Scope:** final extension manuscript and PDF, original group PDF, both supplied CSV files, `Final Project.Rmd`, all published result files, and the proposed public-repository contents.  
**Verdict:** **PASS WITH NOTES.** The extension's data summaries, inferential reconstruction, predictive results, bootstrap comparisons, and SHAP summaries are traceable to the supplied data and executable code. No fabricated observation, performance value, SHAP value, reference, or model run was found. The notes in Section 6 are study limitations, not unresolved integrity failures.

## 1. Data and source preservation

- `data_raw.csv` has 4,424 rows and 37 columns.
- `clean_model_data.csv` has the same 37 source columns after name normalization plus `grade_year1`.
- All source-column values match between the two files after name normalization.
- For all 4,424 rows, `grade_year1` equals the arithmetic mean of the two semester grade fields within floating-point tolerance.
- The cleaned data have no missing values or duplicate rows. They contain 3,748 positive grades and 676 recorded zeros.
- The source and repository copies have matching SHA-256 values:

| Artifact | SHA-256 |
|---|---|
| `data_raw.csv` | `af38afa88f782afab1bfe444c71bfdcaa4884b309f4322e0a1c85b6106c71c9a` |
| `clean_model_data.csv` | `705f3d2e47e539bb68027442c23a2b380354504876f73f081b687b9639d8a6c8` |
| Original group PDF | `2b09785ec62a7a5cf024e18c644587a86bccc4b13243a22b3817b6e820d57f2e` |

The original group PDF remains byte-identical to the uploaded file. It was copied, not edited or re-exported.

## 2. Reproducibility

- `Final Project.Rmd` uses relative paths, reads only `data/clean_model_data.csv`, and writes derived files only to `results/`.
- A fresh render from the repository root completed with exit code 0 under R 4.4.0, xgboost 3.2.1.1, and randomForest 4.7-1.2.
- All 12 predictive CSV files and all three figures were byte-identical to the prior independent rerun.
- `session_info.txt` changed only because the R Markdown rendering process loaded `knitr`, `rmarkdown`, and their namespaces. The attached model-package versions are unchanged.
- The public run also produced `sample_characteristics.csv`, `inferential_model_table.csv`, and `inferential_model_fit.csv` directly from the supplied data.
- The reconstructed reduced linear model reproduced the original report: n = 3,748, R-squared = 0.0752309, adjusted R-squared = 0.0742426, log-scale RMSE = 0.2027615, and partial F-test p = 0.1681418.

## 3. Verified held-out results

The fixed test set contains 886 records, including 750 positive outcomes and 136 zeros.

| Quantity | Verified value |
|---|---:|
| XGBoost classifier ROC-AUC | 0.6763578431 |
| XGBoost classifier average precision | 0.9137012031 |
| XGBoost conditional RMSE, positive outcomes | 2.0299412556 |
| XGBoost full-sample RMSE | 4.6323945196 |
| XGBoost full-sample MAE | 3.5152022668 |
| XGBoost full-sample R-squared | 0.0854462415 |
| Full-sample RMSE difference vs regression | -0.0342731961; 95% interval -0.0888179843 to 0.0250141887 |
| Full-sample RMSE difference vs random forest | -0.2508256843; 95% interval -0.3626817274 to -0.1313899753 |

The manuscript keeps positive-only log-scale inference separate from held-out original-scale prediction. It does not convert SHAP values into causal effects.

## 4. Corrections completed after the first audit

| Earlier issue | Final resolution |
|---|---|
| Zero-grade mechanism was overstated | The report now says that both semester grade-average fields are recorded as zero and that the available variables do not identify one common reason. |
| Original group analysis lacked explicit credit | The extension lists Yi Zhao as sole extension author and explicitly names and cites Emilia Li, Jingxiu Zeng, Felix Zhao, Shimin Zeng, and Siqi Yang for the prior group report. |
| Nonessential background citation had unstable metadata | The unsupported background sentence and its three nonessential education citations were removed. |
| Calibration was overstated | The discussion now says regression was competitive on probability scoring metrics. |
| SHAP wording implied incremental value | The report now describes a nonzero contribution within the fitted model and says this does not establish incremental out-of-sample value. |
| Public R Markdown was not a clean reproduction source | The public file is named `Final Project.Rmd`, has no unrelated crop scaffold or absolute path, and labels code for each manuscript table and figure. |
| Funding and other declarations were unverified or unwanted | Both abstracts and all requested declarations were removed from the extension PDF. The README retains a factual development note without claiming that no AI assistance occurred. |

## 5. Citation, authorship, and document checks

- Every author-year citation in the extension has a reference-list entry, and every reference-list entry is used in the text.
- The six external records are real and match the use made of them. The seventh reference is the supplied original group report.
- The UCI citation and CC BY 4.0 license match the official UCI repository record. The 2022 data-paper DOI is `10.3390/data7110146`.
- The final PDF metadata lists Yi Zhao as author. The visible title page also lists only Yi Zhao.
- Text extraction confirms that the PDF contains no abstract, Chinese abstract, data/code availability statement, ethics statement, author-contribution statement, funding statement, competing-interests statement, or AI-assistance disclosure.
- The final extension PDF has seven pages, parses successfully, has no encryption or JavaScript, and embeds all listed fonts. Visual review of every page found no clipped text, overlap, broken table, missing glyph, or misplaced figure.
- The DOCX accessibility audit returned zero high-, medium-, or low-severity findings. Tables have marked header rows and both figures have descriptive alternative text.

## 6. Remaining limitations and honest interpretation

- The dataset represents one Portuguese institution and lacks an external test institution.
- Recorded zeros may arise through different processes that the available variables do not distinguish.
- The model uses five enrollment-time variables and one held-out split. Development folds also informed tuning.
- XGBoost's ROC-AUC of 0.676 and full-sample R-squared of 0.085 indicate limited individual-level prediction.
- No subgroup calibration or external validation was performed.
- SHAP curves are descriptive model decompositions without uncertainty intervals.
- The original group source code was not available as a clean complete source. The inferential code is explicitly labelled as a reconstruction from the original report.

## 7. Publication decision

The final package is suitable for public release as a transparent course-project extension. This verdict confirms traceability and reproducibility within the checked environment; it is not peer review, external validation, or evidence that the model is appropriate for high-stakes individual decisions.
