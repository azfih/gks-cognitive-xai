# Findings notes (working draft, October 2026)

These are working notes, not the final report. Explainability (SHAP) is planned but not done yet.

## Agenda

To analyze the models' performance with and without MMSE, and then, for the best-performing models, to see which features contribute most to the predictions (SHAP, planned next).

## Baseline experiment

First I built a "dumb" model that always predicts the most common class. It sets the benchmark that every other model has to beat.

| Metric | Baseline (always majority) |
|---|---|
| Accuracy | 0.55 |
| Precision | 0.00 |
| Recall | 0.00 |
| F1 | 0.00 |
| ROC-AUC | 0.50 |

A first run with 5-fold person-level validation (Logistic Regression, Random Forest, XGBoost, with and without MMSE) is saved in `report/results_v1.csv`. It showed the same overall pattern as the repeated validation below: MMSE helps a lot.

## Repeated validation (main results)

I repeated the person-level validation 10 times with different random groupings of people (50 test rounds in total). Values are mean ± standard deviation. The full table is in `report/results_v2_repeated.csv`.

| Experiment | Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|---|
| B (no MMSE) | Baseline | 0.55 ± 0.01 | 0.00 ± 0.00 | 0.00 ± 0.00 | 0.00 ± 0.00 | 0.50 ± 0.00 |
| B (no MMSE) | Logistic Regression | 0.70 ± 0.07 | 0.69 ± 0.09 | 0.64 ± 0.12 | 0.65 ± 0.09 | 0.76 ± 0.09 |
| B (no MMSE) | Random Forest | 0.63 ± 0.08 | 0.61 ± 0.11 | 0.56 ± 0.11 | 0.57 ± 0.09 | 0.69 ± 0.09 |
| B (no MMSE) | XGBoost | 0.64 ± 0.08 | 0.61 ± 0.10 | 0.59 ± 0.13 | 0.59 ± 0.10 | 0.69 ± 0.08 |
| A (with MMSE) | Baseline | 0.55 ± 0.01 | 0.00 ± 0.00 | 0.00 ± 0.00 | 0.00 ± 0.00 | 0.50 ± 0.00 |
| A (with MMSE) | Logistic Regression | 0.81 ± 0.05 | 0.84 ± 0.08 | 0.73 ± 0.09 | 0.77 ± 0.07 | 0.91 ± 0.04 |
| A (with MMSE) | Random Forest | 0.80 ± 0.05 | 0.85 ± 0.07 | 0.68 ± 0.09 | 0.75 ± 0.06 | 0.87 ± 0.04 |
| A (with MMSE) | XGBoost | 0.78 ± 0.05 | 0.81 ± 0.08 | 0.69 ± 0.10 | 0.74 ± 0.07 | 0.86 ± 0.04 |

Repeating the validation 10 times (50 test rounds) gave more stable estimates than a single 5-round run. Logistic Regression remained the best or tied-best model in both experiments, but the differences between models were mostly within the spread.

## Findings

1. **MMSE effect (RQ2):** For Logistic Regression, removing MMSE reduced ROC-AUC from 0.91 to 0.76 and recall from 0.73 to 0.64. Random Forest and XGBoost showed the same pattern. This is a clear sign that MMSE carries a large share of the predictive signal for dementia status. MMSE is a cognitive test that is close to what CDR measures, so Experiment A shows how much the models lean on it, and not that the model is better in a medical sense.

2. **Model comparison (RQ1):** In the repeated validation, the best model was Logistic Regression, with ROC-AUC 0.91 in Experiment A and 0.76 in Experiment B. The baseline got 0.50. The gaps between the three models were smaller than the spread between test rounds, so no model can be called clearly better. A simple, easy-to-interpret model performed at least as well as the more complex ones.

3. **Missed cases:** With Logistic Regression (one run, 5-fold person-level validation), 61 visits with dementia signs were missed without MMSE and 46 with MMSE, out of 167. Without MMSE there were also 50 false alarms, and with MMSE 24. These counts are visits (rows), not people.

4. **Where the misses are:** Almost all the misses were at CDR 0.5, which means very mild impairment. Without MMSE, 50 of the 61 misses were CDR 0.5 (41% of those 123 visits were missed). With MMSE, 45 of the 46 misses were CDR 0.5 (37% missed), while CDR 1.0 was almost always caught (1 of 41 missed). MMSE therefore helped most with clearer cases, and very mild cases stayed hard. (CDR 2.0 has only 3 visits, so it says nothing.) My hypothesis is that people with very mild impairment look similar to healthy people on these features, so there is little signal to separate them. This is a hypothesis, not a proven fact. Since very mild cases are where early detection matters most, a future study could model severity levels (for example CDR 0, 0.5, 1 or above) instead of a yes/no label. I am not doing this in this project because there are too few visits at CDR 1 or above.

5. **Limitations:** The data contain only 150 people (373 visits) from a single study cohort, with no external validation, so the results show associations in this dataset and are not a clinical tool. The label also includes very mild impairment (CDR 0.5), so it is not a confirmed diagnosis.

## Next steps

- Run SHAP on Logistic Regression and Random Forest and compare which features they rely on (RQ3).
- Compare the important features with published medical findings.