# Model Equations

This file documents the regression equations used in the analysis, explains every coefficient in plain business language, describes the dummy variable approach, and states the final model selected.

All figures below are taken directly from `analysis/regression_workbook.xlsx` (sheets `Simple_Regression`, `Multiple_Regression`) and were verified independently against a Python/statsmodels run of the same dataset — the two match to four decimal places.

---

## 1. Dummy Variable Approach

Two categorical variables were converted into dummy (0/1) variables: `store_type` and `region`.

| Categorical variable | Categories present | Reference category | Dummies created |
|---|---|---|---|
| `store_type` | Airport, High Street, Mall, Residential | **High Street** (most frequent: 116 of 320 rows) | `storetype_Airport`, `storetype_Mall`, `storetype_Residential` |
| `region` | East, North, South, West | **East** (most frequent: 104 of 320 rows) | `region_North`, `region_South`, `region_West` |

**Why a reference category is excluded:** if all four categories of a variable were each given their own dummy column, the four dummy columns would always sum to 1 for every row. Combined with the intercept column (which is also always 1), this creates perfect linear dependence between predictors — the "dummy variable trap." A regression cannot be solved (or is unstable) under this condition. Dropping one category as the reference avoids the redundancy: every coefficient on a remaining dummy is then interpreted **relative to the reference category**, holding all other predictors constant.

`store_type` dummies appear in the final multiple regression model. `region` dummies were created and are available in `analysis/regression_workbook.xlsx` (sheet `Dummy_Variables`) but were not included in the final model — see `analysis/model_comparison.md` for the reasoning.

---

## 2. Simple Regression Equations

Each simple regression uses **monthly_sales** as the dependent variable and a single predictor.

**Model A — marketing_spend**
```
monthly_sales = 560,777.35 + 2.1296 × marketing_spend
```
- Every additional ₹1 of monthly marketing spend is associated with an additional ₹2.13 in monthly sales, on average, holding nothing else constant.
- R² = 0.167 — marketing spend alone explains about 17% of the variation in monthly sales.

**Model B — footfall**
```
monthly_sales = 446,410.58 + 35.678 × footfall
```
- Each additional visitor in a month is associated with an additional ₹35.68 in monthly sales, on average.
- R² = 0.736 — footfall alone explains about 74% of the variation in monthly sales, making it the single strongest individual predictor tested.

**Model C — avg_discount_pct**
```
monthly_sales = 697,835.63 − 138,730.45 × avg_discount_pct
```
- A 1 percentage-point increase in average discounting is associated with a ₹1,387 *decrease* in monthly sales, on average — but this relationship is not statistically significant (p = 0.104), so the negative sign should not be read as reliable evidence of a discounting penalty.

**Model D — inventory_availability_pct**
```
monthly_sales = 484,814.26 + 2,217.83 × inventory_availability_pct
```
- A 1 percentage-point increase in inventory availability is associated with a ₹2,218 increase in monthly sales, on average. Statistically significant (p = 0.041) but explains very little variation on its own (R² = 0.013).

**Model E — customer_rating**
```
monthly_sales = 701,366.86 − 5,284.87 × customer_rating
```
- Not statistically significant (p = 0.638). The negative sign should not be interpreted as "higher-rated stores sell less" — it is statistical noise.

---

## 3. Multiple Regression Equation (Final Model)

```
monthly_sales =
      136,913.65
    + 33.664  × footfall
    + 1.1655  × marketing_spend
    + 3,001.20 × inventory_availability_pct
    − 58,920.81 × avg_discount_pct
    + 21,905.71 × storetype_Airport
    + 11,433.46 × storetype_Mall
    − 16,575.85 × storetype_Residential
```

Reference category for the store-type dummies: **High Street**. R² = 0.8214.

### Coefficient-by-coefficient explanation

| Term | Coefficient | P-value | Business meaning |
|---|---|---|---|
| Intercept | 136,913.65 | 0.0018 | Baseline monthly sales for a High Street store with zero footfall, zero marketing spend, zero inventory availability, and zero discount — not a meaningful real-world scenario on its own, but mathematically required as the equation's anchor point. |
| footfall | +33.66 | <0.0001 | Holding marketing spend, inventory, discounting, and store type constant, each additional monthly visitor adds about ₹33.66 in sales. The strongest and most reliable driver in the model. |
| marketing_spend | +1.17 | <0.0001 | Holding other factors constant, each additional ₹1 of marketing spend adds about ₹1.17 in sales — a positive but modest return once footfall is already accounted for (marketing's effect partly works *through* driving footfall, so this coefficient captures only the remaining direct association). |
| inventory_availability_pct | +3,001.20 | <0.0001 | Each 1 percentage-point improvement in product availability is associated with about ₹3,001 more in monthly sales, holding other factors constant. Statistically strong and economically meaningful. |
| avg_discount_pct | −58,920.81 | 0.110 | Not statistically significant at the 5% level. The negative direction is consistent with discounting not paying for itself in this model, but the evidence is too weak to act on with confidence. |
| storetype_Airport | +21,905.71 | 0.020 | Airport stores sell about ₹21,906 more per month than an otherwise-identical High Street store, holding footfall, marketing, inventory, and discount constant. |
| storetype_Mall | +11,433.46 | 0.083 | Mall stores sell about ₹11,433 more than High Street, but this is only marginally significant (just above the conventional 5% threshold) — directionally positive, not conclusively proven. |
| storetype_Residential | −16,575.85 | 0.007 | Residential stores sell about ₹16,576 less per month than an otherwise-identical High Street store, holding other factors constant. Statistically significant. |

---

## 4. Final Model Selected and Reason

**Final model: Model F — the multiple regression model.**

Reasons:
1. **Explanatory power.** R² = 0.821 versus a maximum of 0.736 for any single-variable model (footfall alone). The multiple model captures meaningfully more of the variation in monthly sales.
2. **Multiple levers, one picture.** Leadership is weighing several possible actions (marketing, inventory, discounting, store-type prioritization). A single-variable model cannot show how these factors compare or interact; the multiple model puts them on the same footing, each holding the others constant.
3. **No multicollinearity problems.** Variance Inflation Factors for all predictors in the final model are close to 1.0 (max ≈ 1.29), meaning the coefficients are stable and individually interpretable — this was checked before finalizing the model.
4. **Statistically defensible core story.** footfall, marketing_spend, inventory_availability_pct, and the Airport/Residential store-type effects are all significant at conventional thresholds (p < 0.05). The model's main conclusions do not rest on a single fragile coefficient.

`avg_discount_pct` and `storetype_Mall` were retained in the model for completeness and transparency (so leadership can see the full picture leadership asked about) even though their p-values do not clear the standard 5% bar — see `outputs/final_recommendation.md` for guidance on how much weight to place on each variable.
