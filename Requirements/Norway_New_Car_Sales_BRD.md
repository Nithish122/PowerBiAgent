# Business Requirements Document: Norway New-Car Sales

## 1. Document Control
- Version: 1.0
- Status: Draft for report development
- Owner: Business owner to be confirmed
- Audience: Executive, automotive market analyst, brand/product analyst
- Report: Three-page Power BI report
- Geography: Norway (inferred from source file names; confirm)
- Data period available: January 2007-January 2017

## 2. Business Context and Goal
The supplied files describe Norwegian new-car registrations/sales by month, make, and model, with monthly market and emissions/powertrain indicators. Users need a concise way to understand market volume and direction, changes in powertrain mix and reported CO2, and the makes/models contributing to sales.

The source files do not contain factory output, production capacity, or manufacturing events. This report must describe sales/registrations only; it must not label registrations as vehicles manufactured.

### Goals
- REQ-001: Provide an executive overview of new-car sales volume and its change over time.
- REQ-002: Show how available diesel, hybrid, electric, import, used-car, and CO2 indicators change over time, with missing-history caveats.
- REQ-003: Identify leading makes and models and show their relative contribution to recorded new-car sales.
- REQ-004: Let users filter and compare the available reporting period consistently across pages.
- REQ-005: Make data limitations and ambiguous source definitions visible; do not imply precision or coverage that the files do not support.

## 3. Users and Decisions
- PER-01 Executive/market leader: assess overall market size, direction, and notable shifts; desktop.
- PER-02 Market analyst: compare monthly sales and available market indicators; desktop.
- PER-03 Brand/product analyst: identify leading makes/models and changes in ranking; desktop.

## 4. Scope
### In scope
- Monthly new-car sales/registration totals and source-provided monthly indicators.
- Make- and model-level sales quantities and source-provided percentages.
- Calendar-month and calendar-year analysis across the dates present in the CSVs.
- Three report pages, with consistent date filtering and relevant page-specific make/model filters.

### Out of scope
- Vehicle manufacturing/production volume, plant output, capacity, inventory, revenue, price, profitability, customer demographics, dealer performance, geography below Norway, forecasts, targets, and real-time reporting.
- Claims about causes of sales or emissions changes.
- Row-level security: no access-control or user-to-region data was supplied.

## 5. Source Inventory and Observed Shape
| Source ID | File | Observed fields | Observed shape / intended grain |
|---|---|---|---|
| SRC-01 | `Data/norway_new_car_sales_by_month.csv` | Year, Month, Quantity, Quantity_YoY, Import, Import_YoY, Used, Used_YoY, Avg_CO2, Bensin_Co2, Diesel_Co2, Quantity_Diesel, Diesel_Share, Diesel_Share_LY, Quantity_Hybrid, Quantity_Electric, Import_Electric | 121 rows; one row per Year-Month; Jan 2007-Jan 2017 |
| SRC-02 | `Data/norway_new_car_sales_by_make.csv` | Year, Month, Make, Quantity, Pct | 4,377 rows; intended grain one row per Year-Month-Make |
| SRC-03 | `Data/norway_new_car_sales_by_model.csv` | Year, Month, Make, Model, Quantity, Pct | 2,694 rows; intended grain one row per Year-Month-Make-Model |

Observed details:
- Each file contains the date fields Year and Month. The monthly file has all 12 months for 2007-2016 and January only for 2017.
- The monthly file has `NA` in Used (60 rows), Used_YoY (72), Quantity_Hybrid (48), Quantity_Electric (48), and Import_Electric (68). Treat `NA` as missing, not zero.
- Make Pct values sum to 99.5%-100.4% per month, consistent with rounded shares but not yet confirmed by the data owner.
- Make and Model contain inconsistent leading/trailing whitespace in observed values. Trim category text before reporting; preserve source files unchanged.
- CSV numeric values are represented as text and require type conversion. Validate malformed values and duplicate keys during ingestion.

## 6. KPI and Metric Requirements
KPI means a measure used to evaluate a business goal. No targets or thresholds are provided, so show values and comparisons without target-status colors or claims of success/failure.

| ID | KPI / metric | Definition and display | Grain / source | Goal |
|---|---|---|---|---|
| KPI-001 | New-car sales | Sum of monthly `Quantity`; display as vehicle count. Use SRC-01 for market total. | Calendar month; SRC-01 | REQ-001 |
| KPI-002 | New-car sales change vs prior year | For a selected month/period, current `Quantity` minus the same calendar month(s) one year earlier. Display as count and percentage. Percentage = change / prior-year quantity; blank if prior-year value is absent or zero. | Calendar month; SRC-01 | REQ-001 |
| KPI-003 | Diesel share | Use source `Diesel_Share` as a percentage for the selected month; show its unit and do not sum percentages across months. | Calendar month; SRC-01 | REQ-002 |
| KPI-004 | Electric sales share | `Quantity_Electric` / `Quantity` for a month with a valid electric count and positive sales total. Display as percentage; otherwise blank. | Calendar month; SRC-01 | REQ-002 |
| KPI-005 | Average reported CO2 | Use `Avg_CO2` for the selected month; display source unit only after confirmed. Do not average monthly averages across a multi-month period unless a valid weighting rule is approved. | Calendar month; SRC-01 | REQ-002 |
| KPI-006 | Make/model sales and share | Sum `Quantity` in the selected context; show source `Pct` at its recorded Year-Month category grain. Do not sum Pct across time. | Year-Month-Make / Year-Month-Make-Model; SRC-02 / SRC-03 | REQ-003 |

Additional metrics for charts (not independent KPIs): `Import`, `Used`, `Quantity_Diesel`, `Quantity_Hybrid`, `Quantity_Electric`, `Import_Electric`, `Bensin_Co2`, `Diesel_Co2`, `Diesel_Share_LY`, `Quantity_YoY`, `Import_YoY`, and `Used_YoY`, subject to definition and missing-value gaps below.

## 7. Report Pages

### Page 1: Market Overview
- Type/persona: Executive; PER-01 and PER-02.
- Business question: How large is the new-car market, how is it changing, and which makes contribute most?
- Slicers: Calendar date range/year; month selection where needed. Use the same date context as other pages.
- KPI cards: New-car sales (KPI-001), change vs prior year (KPI-002), latest selected month sales, top make by sales. Label the period for context; do not imply a target.
- Visuals:
  - Line chart: monthly new-car sales (Quantity), with prior-year same-month comparison if supported by the date selection. Use a continuous chronological axis.
  - Ranked horizontal bar chart: Top 10 makes by Quantity in the selected period, with sales count and share where valid.
  - Detail table: Year-Month, Quantity, prior-year change, and selected source-provided monthly metrics for reconciliation.
- Interactions: selecting a make filters the detail table and trend only when that interaction is meaningful; date selections filter all page visuals.
- Requirements: REQ-001, REQ-003, REQ-004, REQ-005.

### Page 2: Powertrain and CO2 Trends
- Type/persona: Analysis; PER-01 and PER-02.
- Business question: How have reported diesel/electric/hybrid indicators and average CO2 changed over time?
- Slicers: Calendar date range/year. No make/model filter unless the source supports a reliable join at this grain.
- KPI cards: Diesel share (KPI-003), electric sales share (KPI-004), average reported CO2 (KPI-005), latest selected month new-car sales (KPI-001).
- Visuals:
  - Line chart: monthly diesel share and electric sales share; leave missing values blank, never convert missing to zero.
  - Line chart: monthly `Avg_CO2`, with `Bensin_Co2` and `Diesel_Co2` as comparison series if units/definitions are confirmed.
  - Clustered column chart: monthly diesel, hybrid, and electric quantities where available. Do not use a stacked chart unless categories are confirmed mutually exclusive.
  - Detail table: Year-Month, relevant source values, and visible blanks for unavailable observations.
- Add a concise visual/page note that alternative-fuel history is incomplete and the last year in the data is partial.
- Requirements: REQ-002, REQ-004, REQ-005.

### Page 3: Make and Model Performance
- Type/persona: Analysis/detail; PER-02 and PER-03.
- Business question: Which makes and models lead sales, and how do their volumes and ranks vary over the selected period?
- Slicers: Calendar date range/year, Make, and Model (search enabled for long lists). Model selection must respect its Make.
- KPI cards: selected-period sales, selected make/model share for a single month (otherwise blank or clearly defined), number of makes, number of models in context.
- Visuals:
  - Ranked horizontal bar chart: Top 10 makes by Quantity, with an optional Top N selector only if supported by the report specification.
  - Ranked horizontal bar chart: Top 10 models by Quantity in the current date/make context.
  - Line chart: monthly Quantity for the selected make or selected top makes (limit to five series); do not show every make at once.
  - Matrix: Make > Model rows with Quantity for the selected period; provide drill/expand and sort descending by Quantity.
- Requirements: REQ-003, REQ-004, REQ-005.

## 8. Interaction and Usability Requirements
- REQ-006: Use a consistent calendar date filter across pages; display the selected date range and month/year clearly.
- REQ-007: Cross-filter related visuals by default. Do not allow KPI cards to filter one another. Ensure a make selection does not produce a misleading model comparison.
- REQ-008: Use descriptive titles, units, readable axis labels, chronological month sorting, and accessible contrast. Show missing data as blank/gap and explain it where it affects interpretation.
- REQ-009: Keep the report to exactly three pages. No mobile layout is required by this BRD.
- REQ-010: Provide a page-level reset-to-default action if supported by the implementation; preserve the intended default date range.

## 9. Data and Semantic Model Requirements
- REQ-011: Create a calendar date dimension covering all dates in the source period, with calendar year, month number/name, and sortable Year-Month.
- REQ-012: Preserve source grains separately: monthly market fact (one row per Year-Month), make fact (one row per Year-Month-Make), and model fact (one row per Year-Month-Make-Model). Do not add the three quantity sources together; they represent overlapping aggregations.
- REQ-013: Relate facts through shared Date and Make dimensions; relate Model to Make only if uniqueness and filtering behavior are validated. Avoid fact-to-fact relationships.
- REQ-014: Convert Year, Month, quantities, percentages, and CO2 fields to appropriate numeric/date types. Convert `NA` to null. Trim Make and Model values consistently and document transformations.
- REQ-015: Validate date-key uniqueness, fact grain uniqueness, nonnegative quantities, numeric conversion, and source-to-model row counts. Reconcile monthly make/model totals to monthly Quantity and report exceptions; do not silently force mismatches to balance.
- REQ-016: Use import mode unless a later requirement establishes another refresh/latency need. Source refresh cadence and refresh ownership must be confirmed.
- REQ-017: Do not implement RLS or expose security claims without approved user-access requirements and mapping data.

## 10. Assumptions
- ASM-001 (proposed): “Norway” in the filenames identifies the geography; there is no country column to validate this.
- ASM-002 (proposed): `Quantity` is the count of new-car sales/registrations for the Year-Month. Validate whether “sales” and “registrations” are interchangeable in the source.
- ASM-003 (proposed): Month is a calendar month numbered 1-12; no fiscal calendar is supplied.
- ASM-004 (proposed): `Pct` represents category share within the corresponding month. Preserve as a source percentage, subject to owner confirmation and rounding.
- ASM-005 (proposed): `NA` means unavailable/not reported, not zero. DAX and visuals should retain blanks.
- ASM-006 (proposed): Report the available period only, January 2007-January 2017; do not extrapolate or describe 2017 as a complete year.
- ASM-007 (proposed): Use a default of the full available date range until a business owner supplies a different default.
- ASM-008 (proposed): Source CSVs are static inputs until a refresh process and data owner are provided.

## 11. Gaps Requiring Owner Confirmation
- GAP-001 (major, non-blocking): Confirm business meaning of `Quantity`: new-car sales, first registrations, or another event; confirm the reporting geography.
- GAP-002 (major, non-blocking): Define `Quantity_YoY`, `Import_YoY`, and `Used_YoY`: absolute change or percentage, comparison period, and treatment of absent prior-year data. KPI-002 uses a transparent calculation from Quantity unless confirmed otherwise.
- GAP-003 (major, non-blocking): Define `Import`, `Used`, and `Import_Electric`; confirm whether these refer to vehicles, transactions, or registration status and whether they overlap `Quantity`.
- GAP-004 (major, non-blocking): Confirm whether `Diesel_Share` and `Diesel_Share_LY` are percentages, their denominator, and whether values are already percentage points (e.g. 79.4) rather than fractions.
- GAP-005 (major, non-blocking): Confirm powertrain definitions, completeness dates, and whether diesel/hybrid/electric categories are mutually exclusive. Keep stacked comparisons out until confirmed.
- GAP-006 (major, non-blocking): Confirm `Avg_CO2`, `Bensin_Co2`, and `Diesel_Co2` units, test standard, population, and aggregation/weighting method.
- GAP-007 (major, non-blocking): Confirm whether `Pct` is a within-month share of new registrations and why sums differ slightly from 100% (observed range 99.5%-100.4%).
- GAP-008 (major, non-blocking): Confirm acceptable reconciliation tolerances between monthly, make, and model Quantity totals, including any omitted categories.
- GAP-009 (minor, non-blocking): Provide KPI targets, thresholds, preferred default date, refresh cadence, business owner, and data owner if status indicators or scheduled refresh are required.
- GAP-010 (minor, non-blocking): Confirm whether the report requires Norwegian-language labels, a specific theme, accessibility standard, or mobile layout.

## 12. Acceptance Criteria
- AC-001: The report contains exactly the three pages and business questions defined in Section 7.
- AC-002: All displayed metrics have a visible name, unit, and appropriate date context; no unsupported target status is shown.
- AC-003: Missing source values remain blank, and the report does not present January 2017 as a complete year.
- AC-004: Monthly, make, and model quantities remain at their source grains and are not double-counted across facts.
- AC-005: Date and category filters behave consistently, including Make-to-Model filtering and chronological month sorting.
- AC-006: Data conversion, trimming, uniqueness, and quantity reconciliation checks are documented; exceptions are not silently discarded.
- AC-007: The report describes sales/registrations only and makes no manufacturing-production claim.
- AC-008: All blocking decisions are resolved or recorded as explicit assumptions before implementation; non-blocking gaps are surfaced to the business owner.