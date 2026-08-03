# Two-stage XGBoost prediction of first-year university grades: an extension of a linear regression analysis

Yi Zhao

August 3, 2026

# 1. Introduction

Enrollment records can be used to study adjusted associations with first-year academic performance and to assess out-of-sample prediction. These aims are distinct: an interpretable association model does not establish predictive accuracy on unseen records.

The original STA302 group project by Emilia Li, Jingxiu Zeng, Felix Zhao, Shimin Zeng, and Siqi Yang analyzed first-year grade with multiple linear regression (Li et al., 2026). Diagnostics showed that a single Gaussian model could not represent the exact-zero mass in the outcome. The final inferential model was therefore estimated only among students with positive grades and used a log-transformed response. That restriction improved model adequacy, but it changed the target from all entrants to students with positive outcomes. The present extension is authored by Yi Zhao and treats the group report's inferential analysis as prior work.

The present analysis retains the inferential result and evaluates how accurately two-stage XGBoost predicts positive-grade status, conditional grade, and full-sample expected grade. Test-set SHAP values describe the fitted nonlinear patterns. Regression and random forest provide benchmarks under the same split and feature set. The analysis evaluates prediction, not intervention effects.

# 2. Data

## 2.1 Source and sample

The data come from the Predict Students' Dropout and Academic Success dataset in the UCI Machine Learning Repository (Realinho et al., 2021). The accompanying data paper reports that the Polytechnic Institute of Portalegre assembled administrative, national admission, and macroeconomic records for 4,424 students in 17 programs from 2008 to 2019 (Realinho et al., 2022). The present analysis used the cleaned project dataset without altering the source files.

The outcome, grade_year1, is the average of first- and second-semester grade variables in the cleaned data. It ranges from 0 to 18.284. For each of the 676 zero outcomes, both semester grade-average fields are recorded as zero. The available variables do not identify one common reason for those records. There are 3,748 positive outcomes and 676 zeros.

Five enrollment-time predictors were fixed before the predictive comparison: admission_grade, previous_qualification_grade, age_at_enrollment, gender, and scholarship_holder. Gender and scholarship status are binary coded categories. Their numeric labels identify dataset categories and should not be assigned meanings beyond the data dictionary.

**Table 1. Sample characteristics.**

| Variable | Summary |
|---|---|
| First-year grade | Mean 10.436; SD 4.819; range 0 to 18.284 |
| Positive first-year grade | 3,748 (84.7%) |
| Zero first-year grade | 676 (15.3%) |
| Admission grade | Mean 126.978; SD 14.482; range 95 to 190 |
| Previous qualification grade | Mean 132.613; SD 13.188; range 95 to 190 |
| Age at enrollment | Mean 23.265; SD 7.588; range 17 to 70 |
| Gender category 1 | 1,556 (35.2%); category 0: 2,868 (64.8%) |
| Scholarship category 1 | 1,099 (24.8%); category 0: 3,325 (75.2%) |

# 3. Methods

## 3.1 Inferential model

The original inferential model was fitted to the 3,748 records with grade_year1 greater than zero:

log(grade_year1) = beta_0 + beta_1 admission_grade + beta_2 log(age_at_enrollment) + beta_3 gender + beta_4 scholarship_holder + error.

Previous qualification grade was removed because it added little after admission grade. The partial F-test comparing the larger and reduced models gave p = 0.168, and information criteria favored the reduced specification. Coefficients describe adjusted associations with log grade among students with positive outcomes. They do not describe the probability of obtaining a positive grade and are not causal effects.

## 3.2 Two-stage prediction

For each student i, the first stage estimates p_i = Pr(Y_i > 0 given X_i). The second stage estimates m_i = E(Y_i given Y_i > 0 and X_i). The full-sample prediction is p_i multiplied by m_i. This construction preserves the zero outcome process and the positive-grade scale.

Three pipelines used the same five predictors. The regression benchmark combined logistic regression for the first stage with linear regression of log grade among positive outcomes. Age was log transformed in both equations, and the positive-grade prediction used Duan's smearing estimator when returning to the original scale (Duan, 1983). Random forest used classification and regression forests (Breiman, 2001). XGBoost used a binary logistic objective for the first stage and squared-error regression for the second (Chen & Guestrin, 2016). Tree models used raw age and grades, while gender and scholarship status were coded 0 or 1.

## 3.3 Split, tuning, and evaluation

A fixed seed of 3022026 produced a stratified 80/20 split by zero versus positive outcome. The training set contained 3,538 records, including 2,998 positive outcomes and 540 zeros. The untouched test set contained 886 records, including 750 positive outcomes and 136 zeros.

Hyperparameters were selected on the training set using five stratified folds. The XGBoost grid crossed learning rates 0.03 and 0.08, tree depths 1 to 3, minimum child weights 1 and 8, and subsampling fractions 0.8 and 1.0. Training allowed up to 700 rounds with 35-round early stopping. The selected classifier used learning rate 0.03, depth 3, minimum child weight 8, subsampling 0.8, and 127 rounds. The selected regressor used learning rate 0.03, depth 1, minimum child weight 8, subsampling 0.8, and 298 rounds. Random forest tuning considered mtry values 1 to 5 and stage-specific node sizes with 400 trees. Final forests used 800 trees; the selected classifier used mtry 3 and node size 5, and the selected regressor used mtry 1 and node size 20.

Out-of-fold training predictions set each classification threshold by maximum balanced accuracy. Final pipelines were then fitted once to the complete training set and evaluated once on the test set. Development cross-validation used the same folds that informed tuning, so those estimates are descriptive and may be optimistic. Test performance is the primary comparison.

Classifier metrics were ROC-AUC, average precision, log loss, Brier score, and balanced accuracy. Positive-outcome regression and full-sample expected-grade prediction were evaluated with RMSE, MAE, and R-squared on their respective scales. A 2,000-replicate paired bootstrap of the test records estimated 95% percentile intervals for XGBoost minus comparator differences in full-sample RMSE and MAE. Negative differences favor XGBoost.

## 3.4 SHAP analysis

SHAP values were calculated from the final XGBoost models on the untouched test set (Lundberg & Lee, 2017). Classifier SHAP values are changes on the log-odds scale relative to the model baseline. Regressor SHAP values are changes in predicted grade points conditional on a positive outcome. Mean absolute SHAP summarizes global predictive importance within each stage. Dependence plots and quantile-bin summaries describe nonlinear patterns for continuous predictors. Group-average SHAP values summarize binary predictors. SHAP decomposes predictions from the fitted model; it does not identify causal effects.

The analysis used R 4.4.0, xgboost 3.2.1.1, and randomForest 4.7-1.2.

# 4. Results

## 4.1 Inferential associations

The reduced positive-outcome model had R-squared 0.0752, adjusted R-squared 0.0742, and in-sample RMSE 0.2028 on the log scale. Admission grade and scholarship category 1 had positive adjusted coefficients. Log age and gender category 1 had negative adjusted coefficients. All four confidence intervals excluded zero (Table 2). These estimates apply only to positive outcomes and cannot be compared numerically with held-out RMSE on the original grade scale.

**Table 2. Final linear-model estimates among positive first-year grades.**

| Term | Estimate | SE | 95% CI | p-value |
|---|---:|---:|---:|---:|
| Intercept | 2.4215 | 0.0514 | 2.3207 to 2.5222 | < 0.001 |
| Admission grade | 0.0027 | 0.0002 | 0.0022 to 0.0032 | < 0.001 |
| Log age at enrollment | -0.0862 | 0.0133 | -0.1123 to -0.0602 | < 0.001 |
| Gender category 1 | -0.0513 | 0.0073 | -0.0656 to -0.0371 | < 0.001 |
| Scholarship category 1 | 0.0439 | 0.0076 | 0.0289 to 0.0589 | < 0.001 |

## 4.2 Development cross-validation

XGBoost had the best mean development ROC-AUC, average precision, classifier log loss, conditional RMSE, full-sample RMSE, and full-sample MAE (Table 3). Differences from regression were small. Because the folds also informed tuning, the values should not be treated as unbiased estimates of new-sample performance.

**Table 3. Five-fold development performance, mean (SD).**

| Metric | Regression | Random forest | XGBoost |
|---|---:|---:|---:|
| Classifier ROC-AUC | 0.677 (0.022) | 0.666 (0.020) | 0.700 (0.026) |
| Classifier average precision | 0.917 (0.008) | 0.913 (0.006) | 0.926 (0.009) |
| Classifier log loss | 0.401 (0.009) | 0.464 (0.022) | 0.394 (0.010) |
| Conditional RMSE | 1.939 (0.116) | 1.942 (0.118) | 1.935 (0.114) |
| Full-sample RMSE | 4.606 (0.080) | 4.750 (0.085) | 4.571 (0.076) |
| Full-sample MAE | 3.407 (0.067) | 3.432 (0.088) | 3.393 (0.064) |

## 4.3 Held-out performance

On the test set, XGBoost had the highest ROC-AUC and average precision, while regression had slightly lower log loss and Brier score and slightly higher balanced accuracy (Table 4). XGBoost had the lowest conditional and full-sample RMSE and MAE. Its full-sample R-squared was 0.085, compared with 0.072 for regression and -0.016 for random forest. The 0.914 average precision should be read against a positive-outcome prevalence of 0.847 in the test set; ROC-AUC 0.676 indicates modest ranking discrimination.

**Table 4. Held-out test performance.**

| Component and metric | Regression | Random forest | XGBoost |
|---|---:|---:|---:|
| Classifier ROC-AUC | 0.668 | 0.632 | 0.676 |
| Classifier average precision | 0.910 | 0.896 | 0.914 |
| Classifier log loss | 0.404 | 0.506 | 0.405 |
| Classifier Brier score | 0.124 | 0.138 | 0.124 |
| Classifier balanced accuracy | 0.640 | 0.594 | 0.636 |
| Conditional RMSE | 2.041 | 2.041 | 2.030 |
| Conditional MAE | 1.349 | 1.328 | 1.321 |
| Conditional R-squared | 0.086 | 0.086 | 0.096 |
| Full-sample RMSE | 4.667 | 4.883 | 4.632 |
| Full-sample MAE | 3.526 | 3.590 | 3.515 |
| Full-sample R-squared | 0.072 | -0.016 | 0.085 |

![Held-out performance for regression, random forest, and XGBoost. The left panel compares full-sample RMSE and MAE. The right panel compares ROC-AUC and average precision for positive-grade classification.](../results/performance_overview.png)

*Figure 1. Held-out predictive performance. Lower values are better for RMSE and MAE; higher values are better for ROC-AUC and average precision.*

The paired bootstrap did not establish a test-set RMSE or MAE advantage over regression: both intervals included zero (Table 5). XGBoost had a lower RMSE than random forest throughout the 95% interval. Its MAE interval against random forest included zero.

**Table 5. Paired bootstrap differences in full-sample test error.**

| Comparator | Metric | XGBoost minus comparator | 95% interval |
|---|---|---:|---:|
| Regression | RMSE | -0.034 | -0.089 to 0.025 |
| Regression | MAE | -0.011 | -0.058 to 0.037 |
| Random forest | RMSE | -0.251 | -0.363 to -0.131 |
| Random forest | MAE | -0.075 | -0.172 to 0.021 |

## 4.4 SHAP patterns

Feature rankings differed by stage (Table 6). Gender, age, and scholarship status had the largest mean absolute contributions to positive-grade classification. Admission grade ranked first for conditional grade, followed by age and gender. The values cannot be compared across stages because the classifier uses log odds and the regressor uses grade points.

**Table 6. Test-set mean absolute SHAP values.**

| Feature | Classifier, log odds | Conditional regressor, grade points |
|---|---:|---:|
| Admission grade | 0.149 | 0.290 |
| Previous qualification grade | 0.171 | 0.120 |
| Age at enrollment | 0.272 | 0.212 |
| Gender | 0.347 | 0.196 |
| Scholarship holder | 0.269 | 0.120 |

Admission grade had different shapes across stages (Figure 2). Its classifier contribution was negative below about 115, positive from roughly 115 to 139, and negative above 139, with the most negative mean contribution in the bin above 149. Its conditional-grade contribution crossed from negative to near zero around 125 to 130 and then increased, reaching a mean of 0.644 grade points in the highest bin. Age contributions were positive at ages 18 to 20, near zero or negative in the early twenties, and negative thereafter. The conditional-grade age contribution became progressively more negative across the observed age bins.

Previous qualification grade showed a threshold-like conditional pattern: mean contributions were negative through approximately 133 and positive above that range. Its classifier pattern was less stable, with positive contributions in several lower and upper bins and a negative region around 130 to 141. Sparse observations in the tails limit the precision of these shapes.

For records coded as scholarship category 1, mean SHAP was 0.654 log-odds units in the classifier and 0.221 grade points in the conditional regressor. For records coded as gender category 1, the corresponding means were -0.423 and -0.305. These are average contributions within the fitted model, not effects of changing category membership.

![Six SHAP dependence panels. The top row shows classifier log-odds contributions for admission grade, previous qualification grade, and age. The bottom row shows conditional grade-point contributions for the same features. Points are test records and red curves are LOWESS summaries.](../results/shap_dependence.png)

*Figure 2. Test-set SHAP dependence for continuous predictors. Horizontal zero lines denote no contribution relative to each model baseline. Curves summarize fitted predictions and do not represent causal effects.*

# 5. Discussion

The two-stage formulation extends the original analysis to every student without forcing zero and positive grades into one Gaussian equation. On the held-out set, XGBoost had the lowest conditional and full-sample error. The gain over regression was small and uncertain under paired bootstrap resampling. The larger RMSE improvement over random forest was supported by the bootstrap interval. Regression remained competitive on probability scoring metrics and thresholded classification.

The predictive patterns partly agree with the inferential model. Higher admission grade and scholarship category 1 were associated with stronger positive-outcome performance, while greater age and gender category 1 were associated with lower performance. SHAP added shape information: admission grade was non-monotone for classification but increased more consistently for conditional grade after about 125 to 130, and age contributions declined after the early twenties. Previous qualification grade was omitted from the reduced linear model because its adjusted linear contribution was weak, yet it had a nonzero conditional predictive contribution in the fitted XGBoost model above roughly 133. This describes how the fitted model used the variable; it does not establish incremental out-of-sample value.

Predictive performance remained limited. Full-sample R-squared was 0.085 for XGBoost, and classifier ROC-AUC was 0.676. Enrollment variables alone therefore leave most individual variation unexplained. The binary gender field also requires care because it encodes administrative categories without the context needed for substantive interpretation. Operational use would require external and subgroup validation. The current outputs should not determine admission, funding, or academic support for an individual.

# 6. Limitations

The data come from one Portuguese institution and have no external test institution. Recorded zero grades occur across multiple observed academic-status labels, and the available variables do not distinguish their underlying mechanisms, so the first stage may blend distinct processes. The feature set contains only five enrollment variables and omits study behavior, program context, course difficulty, work obligations, health, and other potential predictors. One held-out split gives limited information about sampling variability, and the reported development folds also informed hyperparameter selection. Nested cross-validation or repeated external validation would provide a stronger performance estimate.

SHAP values describe the fitted XGBoost functions. They have no uncertainty intervals here, and correlated achievement variables can divide attribution in unstable ways. Binned summaries and LOWESS curves are descriptive, especially in sparse tails. The observational design does not identify causal effects. Subgroup calibration and error analyses were outside the present scope and are required before any operational use.

# 7. Conclusion

A two-stage XGBoost model predicted both positive-grade probability and grade conditional on positivity. It achieved held-out full-sample RMSE 4.632 and conditional RMSE 2.030. Its error was lower than the two benchmarks, but the advantage over regression was too small for a decisive claim on this test set. SHAP identified non-monotone admission-grade contributions to classification, increasing conditional-grade contributions at higher admission grades, and declining age contributions after the early twenties. The extension adds a predictive target to the original association analysis; it does not change the causal interpretation.

# References

Breiman, L. (2001). Random forests. *Machine Learning, 45*(1), 5-32. [doi:10.1023/A:1010933404324](https://doi.org/10.1023/A:1010933404324)

Chen, T., & Guestrin, C. (2016). XGBoost: A scalable tree boosting system. In *Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining* (pp. 785-794). Association for Computing Machinery. [doi:10.1145/2939672.2939785](https://doi.org/10.1145/2939672.2939785)

Duan, N. (1983). Smearing estimate: A nonparametric retransformation method. *Journal of the American Statistical Association, 78*(383), 605-610. [doi:10.1080/01621459.1983.10478017](https://doi.org/10.1080/01621459.1983.10478017)

Li, E., Zeng, J., Zhao, F., Zeng, S., & Yang, S. (2026). *STA302 final project report* [Unpublished course report]. University of Toronto.

Lundberg, S. M., & Lee, S.-I. (2017). A unified approach to interpreting model predictions. In *Advances in Neural Information Processing Systems 30*.

Realinho, V., Vieira Martins, M., Machado, J., & Baptista, L. (2021). *Predict students' dropout and academic success* [Data set]. UCI Machine Learning Repository. [doi:10.24432/C5MC89](https://doi.org/10.24432/C5MC89)

Realinho, V., Machado, J., Baptista, L., & Martins, M. V. (2022). Predicting student dropout and academic success. *Data, 7*(11), 146. [doi:10.3390/data7110146](https://doi.org/10.3390/data7110146)
