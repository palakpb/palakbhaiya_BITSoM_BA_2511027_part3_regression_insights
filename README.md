# Part 3 — Regression Insights: What Drives Monthly Sales?

## Business Problem Summary

A retail chain's leadership team wants to understand which operational and marketing factors are most strongly associated with monthly store sales, in order to decide where to focus business action — marketing spend, inventory investment, discounting strategy, staff allocation, or store-type prioritization. This analysis uses regression modeling on store-level performance data to identify and quantify those relationships, and ends with a business recommendation grounded in the regression evidence.

## Dataset Description

**Source:** `data/business_regression_data.xlsx`, sheet `store_performance`, with an accompanying `data_dictionary` sheet.

**Size:** 320 rows = 80 unique stores × 4 monthly observations each (January–April 2025).

**Variables:**

| Column | Type | Description | Cleaning needed? |
|---|---|---|---|
| `store_id` | Identifier (text) | Unique store identifier (e.g., STR-1001) | None — used for labeling only, not a regression input |
| `month` | Date | Reporting month | None — used for context, not a regression input |
| `region` | Categorical | East, North, South, West | Convert to dummy variables if used |
| `store_type` | Categorical | Mall, High Street, Residential, Airport | Convert to dummy variables (used in final model) |
| `marketing_spend` | Numerical (continuous) | Monthly marketing spend per store | None |
| `footfall` | Numerical (count) | Monthly visitor count | None |
| `avg_discount_pct` | Numerical (continuous, 0–1) | Average discount percentage for the month | None |
| `staff_count` | Numerical (count) | Number of store staff | None |
| `inventory_availability_pct` | Numerical (continuous) | Average product availability percentage | None |
| `competitor_distance_km` | Numerical (continuous) | Distance to nearest competitor | **6 missing values** — imputed with column median |
| `holiday_flag` | Binary (0/1) | Whether the month had a meaningful holiday effect | None |
| `customer_rating` | Numerical (continuous, 1–5) | Average customer rating | **8 missing values** — imputed with column median |
| `monthly_sales` | Numerical (continuous) | **Dependent variable for this analysis** | None |
| `monthly_profit` | Numerical (continuous) | Secondary business metric, not used as the dependent variable here | None |

## Dependent and Independent Variables

- **Dependent variable:** `monthly_sales`
- **Independent variables tested (simple regressions):** `marketing_spend`, `footfall`, `avg_discount_pct`, `inventory_availability_pct`, `customer_rating`
- **Independent variables in the final multiple regression model:** `footfall`, `marketing_spend`, `inventory_availability_pct`, `avg_discount_pct`, plus `store_type` dummy variables (`storetype_Airport`, `storetype_Mall`, `storetype_Residential`, with **High Street** as the reference category)
- **Variables identified but not used as regression inputs:** `store_id` and `month` are identifiers/labels, not predictors. `monthly_profit` is a related-but-separate business metric; it was deliberately excluded as a predictor of `monthly_sales` since profit is partly *derived from* sales (using it would mean predicting sales using a number that already encodes sales), and was also not analyzed as a second dependent variable in this scope. `staff_count`, `competitor_distance_km`, `holiday_flag`, and `region` were examined but not included in the final model — see `analysis/model_comparison.md` for the reasoning, and `analysis/residual_analysis.md` for evidence that `region` in particular still carries unexplained signal worth a follow-up model.

## Regression Approach

1. Five **simple linear regression** models were run, each using one independent variable at a time against `monthly_sales`, to understand each factor's standalone relationship with sales (Task 4).
2. One **multiple linear regression** model was run combining four numerical predictors and three store-type dummy variables, to isolate each factor's independent contribution to sales while holding the others constant (Task 5).
3. All regressions were computed using Excel's `LINEST` function (live formulas, not hardcoded values) in `analysis/regression_workbook.xlsx`, and independently cross-validated against a Python/statsmodels OLS run of the same dataset — all coefficients, R-squared values, and p-values match to four decimal places.
4. Predicted values and residuals were calculated for every record under the final model, and the largest positive/negative residuals were examined for business meaning (Task 7).

## Dummy Variable Approach

`store_type` (Mall, High Street, Residential, Airport) was converted into three dummy variables — `storetype_Airport`, `storetype_Mall`, `storetype_Residential` — with **High Street** held out as the reference category (it is the most frequent category, 116 of 320 rows). Each remaining dummy's coefficient is interpreted as that store type's sales difference relative to a High Street store, holding all other model variables constant. `region` dummies were also constructed (reference category: **East**, the most frequent, 104 of 320 rows) and are available in the workbook, but were not included in the final model. Full detail and rationale: `outputs/model_equations.md`.

## Model Comparison Summary

| Model | Variable(s) | R-squared |
|---|---|---|
| Model A (Simple) | marketing_spend | 0.167 |
| Model B (Simple) | footfall | 0.736 |
| Model C (Simple) | avg_discount_pct | 0.008 |
| Model D (Simple) | inventory_availability_pct | 0.013 |
| Model E (Simple) | customer_rating | 0.001 |
| **Model F (Multiple) — FINAL** | footfall, marketing_spend, inventory_availability_pct, avg_discount_pct, store_type dummies | **0.821** |

Full comparison with significance, business usefulness, and limitations for every model: `analysis/model_comparison.md`.

## Final Model Selected

**Model F — the multiple regression model** (R² = 0.821). It substantially outperforms every single-variable model, isolates each lever's independent effect (important because footfall, marketing spend, and inventory availability all move together across stores), and its core findings (footfall, marketing spend, inventory availability, and the Airport/Residential store-type effects) are statistically significant with low multicollinearity (VIFs ≈ 1.0–1.3). Full equation and coefficient-by-coefficient interpretation: `outputs/model_equations.md`.

## Business Recommendation

Footfall and inventory availability are the strongest, most reliable drivers of monthly sales in this data and are where leadership should focus first. Marketing spend has a real but more modest direct effect once footfall is controlled for. Store type matters for portfolio planning (Airport outperforms, Residential underperforms, relative to High Street). Discounting strategy and the Mall store-type effect are not statistically conclusive in this dataset and should not be acted on with high confidence yet. Full recommendation, including risks and the association-vs-causation caveat: `outputs/final_recommendation.md`.

## Assumptions and Limitations

- Missing values in `competitor_distance_km` (6 rows) and `customer_rating` (8 rows) were imputed using the column median.
- The dataset spans only four months, limiting the ability to detect seasonal patterns or longer-term trends.
- The same 80 stores appear four times each (repeated observations), which the final model does not explicitly account for with store-level fixed effects.
- Regression results show statistical association, not proof of causation — see `outputs/final_recommendation.md` for a full discussion of why this distinction matters for the recommended actions.
- The final model explains 82% of the variation in monthly sales; the remaining 18% includes factors not captured here (e.g., region, local competitive dynamics, management quality) — see `analysis/residual_analysis.md` for evidence that region in particular is a candidate for a future model iteration.

## Screenshots Included

| File | Shows |
|---|---|
| `screenshots/simple_regression_output.png` | Simple regression output (Model A: marketing_spend) with slope, intercept, R², F-statistic, t-stat, and p-value |
| `screenshots/multiple_regression_output.png` | Multiple regression output: full input table and coefficient table (intercept, all coefficients, standard errors, t-stats, p-values, R², F-statistic) |
| `screenshots/residuals_preview.png` | Predicted sales, residuals, and the top-5 largest positive/negative residual tables |
| `screenshots/model_comparison_preview.png` | Model comparison summary table across all six models |

## Repository Structure

```
part3_regression_insights/
├── data/
│   └── business_regression_data.xlsx       # Original dataset, untouched
├── analysis/
│   ├── regression_workbook.xlsx            # Full working workbook (8 sheets, live formulas)
│   ├── model_comparison.md
│   └── residual_analysis.md
├── outputs/
│   ├── regression_summary.xlsx             # Clean standalone comparison table
│   ├── final_recommendation.md
│   └── model_equations.md
├── screenshots/
│   ├── simple_regression_output.png
│   ├── multiple_regression_output.png
│   ├── residuals_preview.png
│   └── model_comparison_preview.png
└── README.md
```

### `analysis/regression_workbook.xlsx` sheet guide

| Sheet | Contents |
|---|---|
| `Original_Dataset` | Raw data, unmodified, exactly as provided |
| `Cleaned_Data` | Same data with missing values imputed via formula (references `Original_Dataset`, never overwrites it) |
| `Dummy_Variables` | `store_type` and `region` converted to dummy columns |
| `Model_Data` | Consolidated, contiguous table combining cleaned numerical variables and dummy variables, used as the input range for regression formulas |
| `Simple_Regression` | Five simple regression models with full statistics, computed via `LINEST` |
| `Multiple_Regression` | Final multiple regression model: input table, coefficient table, R², F-statistic |
| `Predictions_Residuals` | Predicted sales and residuals for all 320 records, plus top-5 positive/negative residual tables |
| `Model_Comparison` | Side-by-side comparison of all six models' R² and key p-values |
