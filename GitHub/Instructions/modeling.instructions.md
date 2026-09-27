---
applyTo: "Output/Implementation/model.yaml,Output/PBIP/**/*.SemanticModel/**"
description: Semantic model design and model-level performance rules (merged Modeling Guidelines + Performance Standards)
---
# Semantic Model Rules

## Schema
- Use a star schema. Facts connect directly to dimensions. No snowflake chains when the attributes can be flattened into the dimension.
- Every fact has a one-sentence grain in `model.yaml` (for example, `one row per mention`). Never mix grains; model targets or daily aggregates as a separate fact.
- Each table has exactly one role: `fact`, `dimension`, `bridge`, `lookup`, `measures`.
- Put measures in one hidden-column table named `Measures` unless a measure clearly belongs to one fact.
- Reuse a dimension across facts. Never create a copy of a dimension per page or per fact.

## Relationships
- Default: dimension (one) → fact (many), single direction, active.
- Never create fact-to-fact relationships.
- Bi-directional or many-to-many only when the BRD requires it. Prefer a `Bridge` table, and log an `ASM`.
- Alternate date roles (for example, a reply date) are inactive relationships to `DimDate`. Measures activate them with `USERELATIONSHIP`.
- Key columns on both sides have the same name and data type. The dimension key is unique and non-blank.

## Keys
- Dimensions use an integer surrogate key `[Entity]Key`. The fact foreign key uses the same name.
- Keep the source ID as `[Entity]BusinessKey` only when it's needed for drillthrough or reconciliation.
- Hide all keys. Set `summarizeBy: none` on keys and numeric IDs.
- Missing fact keys map to an `Unknown` member (key `-1`) in the dimension. Never leave them blank.

## Date table
- Required when there is any time analysis. Name it `DimDate`, build it in Power Query M (contiguous range covering all fact dates), and mark it as the date table.
- Minimum columns: `Date` (dateTime, key), `DateKey` (int64, yyyymmdd), `Year`, `Quarter`, `MonthNumber`, `MonthName` (sort by `MonthNumber`), `YearMonth` (sort by a numeric `YearMonthNumber`), `WeekNumber`, `DayOfWeekName` (sort by `DayOfWeekNumber`).
- Add fiscal columns only if the BRD defines a fiscal calendar.
- Hierarchy: `Calendar Hierarchy` with levels Year → Quarter → Month → Date.
- Auto date/time must be off (see PBIP rules).

## Columns and performance
- Include only columns used by relationships, measures, RLS, slicers, visuals or drillthrough. Leave out free text, URLs, GUIDs and long comments unless a detail page needs them.
- Use the smallest correct type: `int64` for keys and counts, `decimal` for currency, `double` for scores, `dateTime` for dates with the time removed when it isn't needed, `boolean` for flags. Never store numbers, dates or flags as text.
- Hide raw numeric columns that measures aggregate. Measures are the only calculation surface.
- Avoid calculated columns. Do the transformation in M or upstream.
- Storage mode: `import` unless the BRD states a latency or volume need. If it does, log an `ASM` for DirectQuery or Direct Lake.
- Incremental refresh, aggregation tables and composite models only when the BRD states the data volume. Otherwise note them in `02-solution-design.md` as future options.

## Organization
- Every table and measure has a description (one line is enough for tables).
- Apply `dataCategory` to geography and URL columns (`City`, `Country`, `WebUrl`, `ImageUrl`).

## Security
- Design RLS only if the BRD defines who sees what. Use a security dimension or mapping table with a single-direction filter.
- Filter expression pattern: `[UserEmail] = USERPRINCIPALNAME()`.
- If the mapping source is unknown, create a stub mapping table and an `ASM`. Never invent users or regions.
- Hidden is not security. If the BRD requires hiding sensitive columns from some users, log a `GAP` for OLS rather than just hiding them.
