# Solution Design

## Model overview

The semantic model will preserve the three source grains separately and avoid synthetic fact-to-fact joins:

| Table | Grain | Source | Purpose | Reqs |
|---|---|---|---|---|
| FactMonthlyMarket | one row per Year-Month | SRC-01 | Monthly market totals and indicator trends | REQ-001, REQ-002, REQ-004, REQ-006, REQ-011, REQ-012 |
| FactMakeSales | one row per Year-Month-Make | SRC-02 | Make sales volume and source share by month | REQ-003, REQ-004, REQ-012, REQ-013 |
| FactModelSales | one row per Year-Month-Make-Model | SRC-03 | Model sales volume and source share by month and make | REQ-003, REQ-004, REQ-012, REQ-013 |
| DimDate | one row per date in the source period | built in M | Shared calendar dimension for month, year, and filtering | REQ-004, REQ-006, REQ-011 |
| DimMake | one row per Make | trimmed source values | Common make dimension for rankings and filters | REQ-003, REQ-013 |
| DimModel | one row per Model | trimmed source values | Model detail and make-to-model navigation | REQ-003, REQ-013 |

The model will follow the BRD requirement to keep source grains separate, to convert `NA` to blank/null, and to validate row counts and uniqueness before use. It will not create fact-to-fact relationships beyond shared Date and Make dimensions.

## KPI list

- KPI-001: New-car sales. Sum of monthly `Quantity`; display as a count of vehicles. Uses the monthly market fact. No target is defined.
- KPI-002: New-car sales change vs prior year. Compares current sales to the same month(s) in the prior year. Percentage is change divided by prior-year quantity when available; blank otherwise.
- KPI-003: Diesel share. Uses source `Diesel_Share` for the selected month; no cross-month summing.
- KPI-004: Electric sales share. Uses `Quantity_Electric / Quantity` for valid months only; missing or invalid periods stay blank.
- KPI-005: Average reported CO2. Uses `Avg_CO2` for the selected month; no weighting across multiple months without an approved rule.
- KPI-006: Make/model sales and share. Uses source `Quantity` and source `Pct` at their recorded grain, without summing percentages across time.

## Report pages

1. Market Overview
   - Executive page focused on sales size, direction, and leading makes.
   - Includes monthly sales trend, prior-year comparison, top makes, and a reconciliation table.
   - Requirements: REQ-001, REQ-003, REQ-004, REQ-005.

2. Powertrain and CO2 Trends
   - Analysis page for diesel, hybrid, electric, and CO2 indicators over time.
   - Includes monthly share trends, CO2 comparisons, and a powertrain quantity view with blanks for missing data.
   - Requirements: REQ-002, REQ-004, REQ-005.

3. Make and Model Performance
   - Detail page for leading makes/models and rank changes by period.
   - Includes top 10 makes, top 10 models, selected make trend, and a make > model matrix.
   - Requirements: REQ-003, REQ-004, REQ-005.

## Security and governance

- RLS is out of scope. The BRD provides no access mapping or user-to-region data.
- The solution will not state security claims that are not supported by source data.
- Missing values and ambiguous source definitions will remain visible to users through annotations, notes, and blank visuals rather than implied precision.

## Performance notes

- Prefer import mode as the default because the BRD does not establish a different latency or refresh pattern.
- Keep date and category filters consistent across pages to reduce visual churn and improve interactivity.
- Use a shared calendar dimension and trimmed text dimensions to support stable grouping and filter behavior.
- Keep visual cardinality constrained by showing top-N makes/models and limiting multi-series comparisons.

## Risks and open gaps

The main non-blocking issues requiring owner confirmation are:

- ambiguous meaning of `Quantity` and geography
- definition of YoY and powertrain metrics
- data-source meaning of import/used/electric metrics
- CO2 units and aggregation method
- acceptable reconciliation tolerance across monthly, make, and model totals
- default date range and refresh ownership

These are recorded as `GAP-001` through `GAP-010` in the requirements file and should be resolved before final sign-off.

## Roadmap

1. Confirm the business meaning of `Quantity`, reporting geography, and KPI targets.
2. Finalize the metric definitions for `YoY`, `Diesel_Share`, `Avg_CO2`, and `Pct`.
3. Validate the data quality checks, trimmed category values, and reconciliation tolerance rules.
4. Build the semantic model and the three report pages using the approved assumptions.
5. Recheck the report against the BRD acceptance criteria before handoff.
