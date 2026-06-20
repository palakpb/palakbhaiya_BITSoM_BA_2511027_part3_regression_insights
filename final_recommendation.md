# Final Recommendation

**Prepared for:** Retail chain leadership team
**Basis:** Multiple regression analysis of 320 store-months across 80 stores (Jan–Apr 2025). Final model R² = 0.821. Full supporting detail in `analysis/regression_workbook.xlsx`, `analysis/model_comparison.md`, `analysis/residual_analysis.md`, and `outputs/model_equations.md`.

---

## Which Factors Appear Most Strongly Associated with Monthly Sales?

In order of statistical strength and practical size, holding other factors constant:

1. **Footfall** — by far the strongest driver (p < 0.0001). Each additional monthly visitor is associated with roughly ₹33.66 in additional monthly sales.
2. **Inventory availability** — strong and significant (p < 0.0001). Each 1 percentage-point improvement in product availability is associated with roughly ₹3,001 in additional monthly sales.
3. **Marketing spend** — significant (p < 0.0001) but with a smaller direct coefficient (≈ ₹1.17 per ₹1 spent) once footfall is already in the model — much of marketing's effect likely flows *through* driving footfall rather than acting independently of it.
4. **Store type** — Airport stores significantly outsell otherwise-identical High Street stores (+₹21,906/month, p = 0.020); Residential stores significantly undersell them (−₹16,576/month, p = 0.007).

## Which Variables Should Leadership Focus On?

**Footfall and inventory availability** are the two variables leadership can act on with the most confidence — both are large in magnitude and statistically robust. Practically:
- Anything that reliably drives foot traffic (location decisions, local marketing, store visibility) has a strong, well-evidenced payoff in this data.
- Inventory availability is both significant and directly operational — it's a lever store operations can pull without needing a marketing budget increase.

**Marketing spend** still matters, but leadership should size expectations modestly: the *direct* return once footfall is accounted for is smaller than a simple "marketing spend vs. sales" view would suggest (Model A's simple R² of 0.167 overstates marketing's standalone importance because some of that apparent effect is really footfall's effect, correlated with marketing spend).

**Store type** is informative for portfolio decisions — Airport locations are out-performing, Residential locations are under-performing, holding other factors equal. This is useful input for expansion or downsizing discussions, though it describes an association with store *type*, not a controllable lever in the same sense as marketing or inventory.

## Which Variables Should Not Be Over-Interpreted?

- **avg_discount_pct** (p = 0.110) — the direction is negative (more discounting, lower sales, other things equal) but is not statistically significant. There is not enough evidence here to conclude that discounting hurts sales, and definitely not enough to size a specific "stop discounting" recommendation.
- **storetype_Mall** (p = 0.083) — directionally positive relative to High Street but only marginally short of conventional significance. Treat as a weak signal worth monitoring, not a confirmed effect.
- **customer_rating** (simple regression, p = 0.638) — showed no detectable relationship with *this month's* sales. This does not mean customer satisfaction doesn't matter — it likely affects longer-term loyalty and repeat visits in ways a four-month sales snapshot can't capture — but the current data doesn't support using it as a monthly sales predictor.

## What Business Action Would We Recommend?

1. **Prioritize footfall-driving investments** (visibility, local marketing, location strategy) — this is the most reliably supported lever in the data.
2. **Invest in inventory availability operationally.** It is both statistically strong and directly within store operations' control, independent of marketing budget.
3. **Treat marketing spend increases as a secondary lever**, not the primary one — its effect is real but smaller than footfall's once footfall itself is controlled for. Consider testing whether marketing budget is better spent specifically on footfall-driving channels.
4. **Use store-type findings for portfolio planning**, not blame. Airport's advantage and Residential's disadvantage are observed patterns to plan around (e.g., when prioritizing new locations) — but see the causation caveat below before treating "convert Residential stores to another format" as a proven fix.
5. **Hold off on a firm discounting recommendation** until more data is collected — the current evidence is too weak to support a confident change in discounting policy either way.
6. **Investigate the regional gap.** The residual analysis (`analysis/residual_analysis.md`) shows the model systematically over-predicts East region stores and under-predicts South/West region stores. This means region carries information not yet captured by the model — worth a follow-up analysis before using these predictions for regional target-setting.

## What Risks or Limitations Should Leadership Keep in Mind?

- **Short time window.** Only four months of data (Jan–Apr 2025) were available. This limits the ability to detect seasonal effects and means any single month's unusual result (e.g., a promotion, a competitor event) can carry outsized weight.
- **Repeated stores, not independent observations.** The same 80 stores appear four times each. A store with consistently strong (or weak) management will show up four times in the data, which can inflate the apparent precision of some estimates if not accounted for with store-level modeling — a refinement not included in this version.
- **Omitted variables.** Region, local competitive intensity beyond a single distance measure, staff experience/quality, and product-mix differences are not in the final model and may explain some of the unexplained 18% of variation (residuals).
- **A few missing values were imputed.** `competitor_distance_km` (6 of 320 rows) and `customer_rating` (8 of 320 rows) had missing values, filled in with the column median for the affected rows. This is a reasonable default but means a small number of records don't reflect their true original values for those two columns.

## Why Does Regression Show Association, Not Causation?

The regression results show that footfall, marketing spend, inventory availability, and store type **move together with** monthly sales in statistically reliable ways. They do not, by themselves, prove that changing one of these variables **causes** a corresponding change in sales. Three reasons this matters here specifically:

1. **Reverse causation is plausible.** A store doing well for other reasons might justify a *higher* marketing budget or *better* inventory allocation from headquarters — meaning sales could be driving marketing spend and inventory priority, not just the other way around.
2. **Confounding variables.** Store type, location quality, and management strength likely influence both the predictors (footfall, marketing spend, inventory) and sales simultaneously. The regression controls for the variables included in the model, but cannot control for factors not in the dataset (e.g., manager tenure, local economic conditions).
3. **Observational, not experimental, data.** No variable here was randomly assigned to stores. Without a controlled experiment (e.g., randomly increasing marketing spend at some stores and not others and comparing outcomes), the relationships found are best read as **"strongly associated with"** rather than **"will cause."**

**Practical implication:** these results are strong enough to prioritize where to investigate and experiment next (e.g., a controlled pilot increasing marketing spend or inventory availability at a sample of stores), but should not be treated as a guarantee of the exact sales increase a specific action will produce.
