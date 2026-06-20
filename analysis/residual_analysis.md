# Residual Analysis

This analysis uses the **final model (Model F — multiple regression)** to calculate predicted monthly sales for every store-month record, compare predictions against actual sales, and examine where the model is most wrong. Full calculations live in `analysis/regression_workbook.xlsx`, sheet `Predictions_Residuals`. A screenshot is provided at `screenshots/residuals_preview.png`.

**Definitions used throughout:**
- `predicted_sales` = Intercept + (coefficient × value) summed across all seven predictors in the final model.
- `residual` = `monthly_sales` (actual) − `predicted_sales`.
- A **positive residual** means the model under-predicted — the store actually sold more than the model expected.
- A **negative residual** means the model over-predicted — the store actually sold less than the model expected.

---

## Top 5 Largest Positive Residuals (Model Under-Predicts)

| Rank | Store ID | Month | Region | Store Type | Actual Sales | Predicted Sales | Residual |
|---|---|---|---|---|---|---|---|
| 1 | STR-1030 | 2025-02 | West | Residential | 820,519 | 714,620 | +105,899 |
| 2 | STR-1073 | 2025-03 | East | Residential | 813,317 | 713,693 | +99,623 |
| 3 | STR-1032 | 2025-01 | South | High Street | 914,544 | 815,773 | +98,771 |
| 4 | STR-1050 | 2025-04 | North | Residential | 735,787 | 638,540 | +97,246 |
| 5 | STR-1028 | 2025-04 | East | Mall | 713,611 | 618,466 | +95,145 |

## Top 5 Largest Negative Residuals (Model Over-Predicts)

| Rank | Store ID | Month | Region | Store Type | Actual Sales | Predicted Sales | Residual |
|---|---|---|---|---|---|---|---|
| 1 | STR-1017 | 2025-03 | West | High Street | 685,379 | 844,620 | −159,241 |
| 2 | STR-1023 | 2025-02 | South | Mall | 627,172 | 755,437 | −128,265 |
| 3 | STR-1012 | 2025-01 | West | Residential | 595,468 | 716,546 | −121,079 |
| 4 | STR-1007 | 2025-02 | West | Mall | 800,452 | 900,272 | −99,821 |
| 5 | STR-1009 | 2025-01 | North | Residential | 641,865 | 740,315 | −98,450 |

---

## What These Residuals Mean in Business Terms

A residual is the part of a store's monthly sales that the model's formula **cannot explain** using footfall, marketing spend, inventory availability, discounting, and store type alone. Large residuals — in either direction — point to stores where something else is going on that the model doesn't capture:

- **Large positive residuals** (e.g., STR-1030, STR-1073, STR-1032) are stores significantly outperforming what their footfall, marketing spend, and inventory levels would predict. Plausible explanations include strong local management, a particularly effective local promotion not captured in `avg_discount_pct`, a nearby competitor closing, or a product mix skewed toward higher-value items. These stores are worth a closer qualitative look — there may be a replicable practice here.
- **Large negative residuals** (e.g., STR-1017, STR-1023, STR-1012) are stores underperforming relative to what the model expects given their inputs. Plausible explanations include local competitive pressure not captured by `competitor_distance_km` alone, operational issues (stockouts not reflected in the monthly average, staffing problems, poor in-store execution), or a one-off disruption during that month.

## Is the Model Systematically Under- or Over-Predicting Certain Store Types?

Checking the average residual by **store_type** (a variable the model already includes): averages are effectively zero for every store type (Airport, High Street, Mall, Residential), which is expected — the regression is built to balance its errors across the categories it explicitly models. This is a basic sanity check, not a finding.

The more informative pattern shows up in **region**, a variable the final model does **not** include:

| Region | Average Residual |
|---|---|
| East | −10,321 (over-predicted) |
| North | −2,012 (slightly over-predicted) |
| South | +7,948 (under-predicted) |
| West | +7,883 (under-predicted) |

This is a meaningful signal: the model, as built, **systematically over-predicts East region stores and under-predicts South and West region stores**, on average. Region was deliberately left out of the final model (see `analysis/model_comparison.md`), and this residual pattern is direct evidence that region carries information about sales beyond what footfall, marketing, inventory, discount, and store type already capture. If leadership wants a model tuned specifically for regional planning, region dummies should be added back in as a next iteration.

## Practical Takeaway

The model is a good general-purpose explainer of sales (R² = 0.821) but is not equally accurate everywhere. Before using it to set regional targets or evaluate individual store managers, leadership should treat:
- **Region** as a known gap — current predictions will run optimistic in the East and conservative in the South/West.
- **Individual large-residual stores** as flags for manual review, not automatic praise or blame — a single month's residual could reflect a real operational story or simply noise in a four-month dataset.
