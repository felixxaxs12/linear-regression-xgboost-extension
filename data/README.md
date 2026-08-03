# Data files

`data_raw.csv` is the supplied UCI-format dataset. `clean_model_data.csv` contains the same 37 source columns after column-name normalization and one derived field:

```text
grade_year1 = (curricular_units_1st_sem_grade + curricular_units_2nd_sem_grade) / 2
```

The transformation was verified across all 4,424 rows within floating-point tolerance. The two files have no missing values or duplicate rows. The predictive code reads only the cleaned file and never modifies either CSV.

## Checksums

| File | SHA-256 |
|---|---|
| `data_raw.csv` | `af38afa88f782afab1bfe444c71bfdcaa4884b309f4322e0a1c85b6106c71c9a` |
| `clean_model_data.csv` | `705f3d2e47e539bb68027442c23a2b380354504876f73f081b687b9639d8a6c8` |

## Source and license

Realinho, V., Vieira Martins, M., Machado, J., and Baptista, L. (2021). *Predict Students' Dropout and Academic Success* [Data set]. UCI Machine Learning Repository. https://doi.org/10.24432/C5MC89

The UCI record licenses the dataset under [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/).

