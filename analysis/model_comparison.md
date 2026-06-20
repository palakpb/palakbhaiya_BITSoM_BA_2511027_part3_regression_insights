# Model Comparison

This document compares all six regression models built for this analysis: five simple (single-predictor) models and one multiple regression model. All figures are sourced from `analysis/regression_workbook.xlsx` and cross-checked against an independent Python/statsmodels run (figures match to four decimal places). A clean version of this table is also saved as `outputs/regression_summary.xlsx`, and a screenshot is provided at `screenshots/model_comparison_preview.png`.

---

## Comparison Table

| Model | Dependent Variable | Independent Variable(s) | R-squared | Significant Variables (p < 0.05) | Business Usefulness | Limitations |
|---|---|---|---|---|---|---|
| **A — Simple** | monthly_sales | marketing_spend | 0.167 | marketing_spend (p < 0.0001) | Confirms marketing spend has a real, positive association with sales, but explains only ~17% of the variation — too thin on its own to size a budget decision. | Ignores footfall, inventory, and store mix, all of which also move sales and likely correlate with marketing decisions. |
| **B — Simple** | monthly_sales | footfall | 0.736 | footfall (p < 0.0001) | The strongest single-variable model. Useful as a quick diagnostic ("is this a high-traffic store?") but footfall itself is mostly a *result* of location, marketing, and store type, not a lever leadership can pull directly. | Doesn't explain *why* footfall varies, so it can't directly guide an action plan. |
| **C — Simple** | monthly_sales | avg_discount_pct | 0.008 | None (p = 0.104) | Not useful in isolation — explains under 1% of variation and the relationship is not statistically distinguishable from zero. | Too weak to support any discounting decision on its own. |
| **D — Simple** | monthly_sales | inventory_availability_pct | 0.013 | inventory_availability_pct (p = 0.041) | Statistically significant but explains very little of the variation in sales on its own (R² = 0.013). Directionally useful, but the practical size of the effect is unreliable when other drivers aren't controlled for. | High risk of omitted-variable bias — stores with better inventory availability may also have more staff or marketing spend, inflating this simple correlation. |
| **E — Simple** | monthly_sales | customer_rating | 0.001 | None (p = 0.638) | Not useful — essentially no detectable linear relationship with monthly sales in this dataset. | Customer rating may matter for longer-term loyalty/repeat visits in ways a single month of sales data can't capture. |
| **F — Multiple (FINAL)** | monthly_sales | footfall, marketing_spend, inventory_availability_pct, avg_discount_pct, storetype_Airport, storetype_Mall, storetype_Residential (ref: High Street) | **0.821** | footfall, marketing_spend, inventory_availability_pct, storetype_Airport, storetype_Residential (all p < 0.05) | By far the most useful model for decision-making: it isolates the effect of each lever (marketing, inventory, discounting, store type) while holding the others constant, and explains 82% of the variation in monthly sales. | Discounting and the Mall store-type effect are not statistically significant here — see notes below. Regression shows association, not proof of cause (see `outputs/final_recommendation.md`). |

---

## Why Model F Is the Final Model

Model F (multiple regression) is selected as the final model for three reasons:

1. **It explains far more of the variation in sales.** R² jumps from 0.736 (footfall alone, the best single-variable model) to 0.821 once marketing spend, inventory availability, discounting, and store type are added together.
2. **It separates correlated effects.** Footfall, marketing spend, and inventory availability are all positively related to sales individually, but they also move together across stores (a well-marketed, well-stocked store tends to also have high footfall). The multiple regression isolates the *independent* contribution of each one — which is exactly the question leadership is asking ("which lever should we pull?").
3. **Its core findings are statistically robust.** Five of the seven predictors are significant at the 5% level, and a multicollinearity check (Variance Inflation Factors, all ≈ 1.0–1.3) confirms the coefficients are stable, not artifacts of predictors crowding each other out.

## Variables That Did Not Hold Up in the Multiple Model

Two variables are included in Model F for completeness but should be treated cautiously:

- **avg_discount_pct** (p = 0.110): direction is negative (more discounting associated with lower sales, holding other factors fixed) but not statistically significant. This could mean discounting genuinely has little independent effect in this dataset, or that four months of data simply isn't enough to detect it reliably.
- **storetype_Mall** (p = 0.083): directionally positive relative to High Street, but only marginally short of the conventional significance threshold. Treat as a weak signal, not a confirmed effect.

## What None of These Models Capture

- **Time trends and seasonality beyond the holiday flag.** The dataset spans only four months (Jan–Apr 2025), which limits the ability to detect seasonal patterns.
- **Store-level fixed effects.** The same 80 stores appear four times each. A model with store fixed effects (not built here) could separate "this store is just a strong performer" from "this particular month's discount/marketing changed performance."
- **Interaction effects.** For example, marketing spend might matter more for Mall stores than Residential stores — this model only tests average effects, not interactions.
- **Causality.** All models here show *association*, not proof that changing a variable will change sales by the stated amount. See `outputs/final_recommendation.md` for a full discussion.
