# Customer churn prediction and retention priority

Which current subscribers look likely to cancel, and who should receive a retention offer first?

This project trains three classifiers on the [IBM Telco Customer Churn](https://www.kaggle.com/datasets/blastchar/telco-customer-churn) snapshot, explains the chosen model with SHAP, and ranks active subscribers by the expected net margin of a retention offer.

The executed notebook is `customer_churn_retention.ipynb`. Charts and printed results are saved in the file, so the analysis can be read on GitHub without rerunning it.

## Result

XGBoost is the champion model. On a held-out 20% of customers it reaches a ROC-AUC of **0.845** and an average precision of **0.658**. At a calibrated probability of **0.37**, precision is **57%** and recall is **71%**. In the highest-risk tenth of the test set, **78%** of customers cancelled. The base rate is **26.5%**.

Of **5,174** current subscribers, **565** clear a positive expected net value under the planning assumptions, worth about **$3,266** in expected margin. The first 200 calls account for about **$2,096** of that total.

| Model | Test ROC-AUC | Test average precision |
| --- | ---: | ---: |
| XGBoost | 0.845 | 0.658 |
| Logistic regression | 0.843 | 0.633 |
| Random forest | 0.840 | 0.640 |
| Month-to-month rule | 0.725 | 0.406 |
| Constant rate | 0.500 | 0.265 |

XGBoost wins on validation average precision. The test column was not used to choose the model. All three trained models beat the month-to-month rule.

## What the notebook does

- Cleans `TotalCharges` and defines cancellation.
- Shows cancellation rates by contract, internet service, payment method, and tenure.
- Fits logistic regression, a random forest, and XGBoost on the same 60/20/20 split.
- Selects the model on the validation slice and reports it once on the test slice.
- Calibrates probabilities and sets the decision threshold on validation.
- Explains the champion with SHAP beeswarm, bar, and waterfall charts.
- Scores every customer with cross-validation, so each score comes from a model that did not train on that customer.
- Prices six retention offers and keeps the one with the highest expected net margin.
- Stress-tests the margin and the offer success rate.
- Assigns the top 80 subscribers to a 90-day offer-versus-holdout pilot.

## Retention ranking

Each current subscriber is scored with:

`expected net = churn probability × offer effectiveness × monthly bill × 6 months × 40% margin − contact cost − uptake × incentive`

The planning inputs are a 40% gross margin, a six-month stay if the customer is saved, and an $8 contact cost. Offer success rates are assumptions. The sensitivity section shows that if offers save only half as many people as assumed, nobody remains worth a paid contact. The names at the top of the list stay mostly the same. The ranking is the stable output. The dollar total is not.

| Tier | Rule | Customers |
| --- | --- | ---: |
| P1 — call this week | expected net of at least $15 | 17 |
| P2 — offer this month | expected net of at least $6 | 213 |
| P3 — low-cost nurture | expected net above $0 | 335 |
| Hold | expected net of $0 or below | 4,609 |

`pilot_sheet.csv` splits the top 80 into 41 offer calls and 39 holdouts. The offer arm covers its planned cost of about $993 if it keeps about 4 more customers than the holdout after 90 days, roughly 11 percentage points of retention.

## Files

| File | Role |
| --- | --- |
| `customer_churn_retention.ipynb` | Full analysis with saved outputs |
| `WA_Fn-UseC_-Telco-Customer-Churn.csv` | Source table, 7,043 customers and 21 columns |
| `retention_priority_list.csv` | All 5,174 current subscribers, best call first |
| `pilot_sheet.csv` | Top 80, split into offer and holdout |
| `requirements.txt` | Package versions used to run the notebook |

