# Research Protocol

**Project:** Explainable Machine Learning for Identifying Patterns Associated with Dementia Status
**Author:** Hifza
**Version:** 1.0 (October 2026)
**Status:** Written before running any models. Changes after this point will be listed in a "Changes log" at the bottom.

---

## 1. Background and motivation

Machine learning models can predict cognitive impairment from clinical and brain-volume data, but a high score alone is not enough. When digital healthcare products are designed, a model should not only predict well, it should also rely on features that are consistent with established medical research. A model that predicts well for the wrong reasons cannot be trusted.

_In my own words:_ This research question is important because, when we design real digital products, we should design them so that they not only predict cognitive decline using the correct features, but also so that we can check whether those features are consistent with medical research.

This project builds on my undergraduate final year project, CereBloom (a cognitive-health platform), where I worked on the backend/AI side. CereBloom was a software project with limited data and was not clinically validated. This project is a smaller and more careful study of one question: what are the models actually relying on?

## 2. Research question

> Which features (such as age, education, and brain volume) do machine learning models rely on most to predict dementia status in older adults, how does performance change when MMSE is removed, and do the important features match established medical findings?

**Sub-questions**

- **RQ1:** How well can different machine learning models predict dementia status from demographic and brain-volume features?
- **RQ2:** How much does performance change when the MMSE score (a cognitive test) is removed from the features?
- **RQ3:** Which features do the models rely on most (measured with SHAP), and do they match established medical findings?

## 3. Dataset

- **Name:** OASIS Longitudinal (Open Access Series of Imaging Studies), downloaded from Kaggle as `oasis_longitudinal.csv`.
- **Size:** 373 rows (visits) from 150 people, aged 60 to 98. Each person has 2 to 5 visits.
- **Important consequence:** the real number of independent people is 150, not 373. Rows from the same person are related, so they must never be split between training and testing (see Section 7).
- **Data use:** the raw data file is not uploaded to this repository. The README links to the source and tells readers where to place the file.

## 4. Label (what the model predicts)

- The label is **dementia status** per visit: `0` if CDR = 0, `1` if CDR > 0.
- CDR (Clinical Dementia Rating) is the clinician's rating that defines the label, so it is **never** used as a feature.
- Class counts: 206 rows with label 0 and 167 rows with label 1. Classes are fairly balanced, so no oversampling is planned.
- Most label-1 rows have CDR = 0.5, which means _very mild_ impairment. In this report the label is therefore described as "dementia status (including very mild)", not as a confirmed diagnosis.
- The `Group` column (Nondemented / Demented / Converted) is not used, because it is built from the same diagnosis information as the label.

## 5. Features

**Used in Experiment B (main):**

| Feature | Meaning                                                                     |
| ------- | --------------------------------------------------------------------------- |
| Age     | Age in years                                                                |
| Sex     | Male/Female, converted to 1/0                                               |
| EDUC    | Years of education                                                          |
| SES     | Socioeconomic status score                                                  |
| eTIV    | Estimated total intracranial volume (head size)                             |
| nWBV    | Normalized whole-brain volume (how much brain tissue relative to head size) |

**Added in Experiment A:** MMSE (Mini-Mental State Examination, a 30-point cognitive test).

**Excluded, and why**

| Column             | Reason                                                                 |
| ------------------ | ---------------------------------------------------------------------- |
| CDR                | Defines the label (would leak the answer)                              |
| Group              | Built from the diagnosis (would leak the answer)                       |
| Visit, MR Delay    | Track time in the study, not the person's condition                    |
| MRI ID, Subject ID | Identifiers (Subject ID is used only to keep a person's rows together) |
| Hand               | Same value for everyone                                                |
| ASF                | Carries almost the same information as eTIV                            |

## 6. Experiments

- **Experiment A:** Age, Sex, EDUC, SES, eTIV, nWBV **plus MMSE**.
- **Experiment B:** the same features **without MMSE**.

MMSE is a cognitive test that doctors use when judging impairment, so it is very close to the label. Comparing A and B shows how much the models depend on it, and whether the remaining features still carry useful signal (RQ2).

## 7. Methods

**Preprocessing**

- Missing values: SES has 19 missing values and MMSE has 2. They are filled with the median.
- The median and the scaling are learned from the **training part only** in each round, using a scikit-learn Pipeline, so no information from the test part leaks in.
- Scaling is applied for Logistic Regression.

**Models**

1. Baseline: always predict the most common class (the "dumb" model every real result must beat).
2. Logistic Regression
3. Random Forest
4. XGBoost

Hyperparameters are kept small and simple. No large search, because with 150 people a big search would overfit the validation results.

**Fair testing**

- Grouped cross-validation: all rows from the same person (`Subject ID`) stay together, either all in training or all in testing (`GroupKFold`).
- The procedure is repeated with different groupings, and results are reported as mean ± standard deviation, never as one single number.

**Metrics**

- Recall (the most important here: of the people with dementia signs, how many were caught)
- Precision
- F1
- ROC-AUC
- Confusion matrix

All are reported. No single "winner metric" is chosen to make the results look better.

## 8. Explainability (SHAP)

- SHAP is applied to Logistic Regression (best performer) and Random Forest (best tree-based model), and the important features are compared between them.
- SHAP values are computed on **held-out test data only**, not on training data.
- Global view: which features matter most overall (summary plot).
- Local view: 2 to 3 individual cases explained, including at least one case the model got wrong.
- Comparison: the results are compared with established medical findings. For example, lower brain volume (nWBV) and older age are expected to be associated with higher dementia risk, and MMSE with lower scores. Findings that do not match are reported and discussed, not hidden.
- Caution: age, eTIV, and nWBV are related to each other, so SHAP scores can be shared between related features. This is stated as a limitation.

## 9. Error analysis

- Which people are consistently misclassified?
- Are the errors concentrated in borderline cases (CDR = 0.5)?
- How do the errors differ between Experiment A and Experiment B?

## 10. Risks and how they are handled

| Risk                           | How it is handled                                                             |
| ------------------------------ | ----------------------------------------------------------------------------- |
| Label leakage                  | CDR and Group are never features                                              |
| Same person in train and test  | Grouped cross-validation by Subject ID                                        |
| Preprocessing leakage          | Imputation and scaling are done inside each training round only               |
| Very small sample (150 people) | Repeated validation, report spread, avoid claiming small differences are real |
| Overfitting through tuning     | Small, simple hyperparameter choices                                          |
| Over-claiming                  | The study is not clinically validated and makes no diagnostic claim           |

## 11. Expected limitations (to be written fully in the report)

- Small dataset (150 people) from a single study cohort, with no external validation.
- The label includes very mild impairment, so it is not a confirmed diagnosis.
- Related features may share SHAP importance.
- Results show associations in this dataset, not causes, and are not a clinical tool.

## 12. Related work read so far

- Lundberg & Lee (2017), _A Unified Approach to Interpreting Model Predictions_ (SHAP).
- Mikhailova & Mikhailova (2026), _Machine learning approaches for early detection of cognitive decline: A comparative study using the OASIS-3 dataset_. A different, larger dataset. Gradient boosting scored best (weighted F1 about 0.86). The authors removed CDR, MMSE and diagnosis codes to avoid leakage, but used basic feature importance and not SHAP, which is the gap this project addresses. Their feature selection appears to use the full dataset before splitting, which this project avoids.
- To read next: Marcus et al. (2010), the OASIS longitudinal dataset paper.

## 13. Timeline

| Month    | Goal                                             |
| -------- | ------------------------------------------------ |
| October  | Protocol, first look at the data, baseline model |
| November | All 3 models, Experiments A and B, evaluation    |
| December | SHAP and error analysis                          |
| January  | Report, graphs, GitHub cleanup                   |
| February | Polish, CV, personal statement                   |

## Changes log

- v1.0 (October 2026): first version.
- v1.1 (October 2026): SHAP will be applied to Logistic Regression as well as Random Forest, because Logistic Regression performed best in the repeated validation.
