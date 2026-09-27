---
applyTo: "Output/Implementation/model.yaml,Output/PBIP/**/*.tmdl"
description: DAX measure patterns, formatting, format strings and folders
---
# DAX Rules

## Measure layers and folders
| Layer | Folder (TMDL uses `\`) | Depends on | Hidden |
|---|---|---|---|
| Base: aggregates one column | `Measures\Base` | columns only | no |
| Intermediate: combines measures, time intelligence | `Measures\Time Intelligence` or `Measures\Base` | base | hide if technical |
| KPI: actual vs target, variance, status | `Measures\KPI`, `Measures\Targets`, `Measures\Variance` | base, intermediate | no |
| Presentation: labels, colors, text | `Measures\Presentation` | any above | no |

- Base measures never reference other measures. KPI measures never reference presentation measures. No circular references.
- One definition, one measure. Never create two measures with the same logic (for example, `Revenue` and `Total Revenue`).

## Coding style
- Multi-step logic uses `VAR` with business names and a single `RETURN`. Simple one-line base measures may skip `VAR`.
- Quote table names in DAX: `'DimDate'[Date]`, `'FactMention'[SentimentScore]`. Reference measures without a table: `[Mention Count]`.
- Put a space inside parentheses: `CALCULATE ( [Mention Count], ... )`. Indent by 4 spaces per nesting level, one argument per line for complex calls.
- Write complete, valid DAX only. No `...`, comments that replace code, or pseudo-functions.

## Patterns (always use `'DimDate'[Date]`)
```DAX
[Mention Count] = COUNTROWS ( 'FactMention' )

[Mention Count YTD] =
CALCULATE ( [Mention Count], DATESYTD ( 'DimDate'[Date] ) )

[Mention Count Previous Year] =
CALCULATE ( [Mention Count], SAMEPERIODLASTYEAR ( 'DimDate'[Date] ) )

[Mention Count YoY %] =
VAR CurrentValue = [Mention Count]
VAR PreviousValue = [Mention Count Previous Year]
RETURN
    DIVIDE ( CurrentValue - PreviousValue, PreviousValue )

[Mention Count Rolling 12 Months] =
VAR AnchorDate = MAX ( 'DimDate'[Date] )
RETURN
    CALCULATE (
        [Mention Count],
        DATESINPERIOD ( 'DimDate'[Date], AnchorDate, -12, MONTH )
    )

[Positive Mention Count] =
CALCULATE (
    [Mention Count],
    KEEPFILTERS ( 'DimSentiment'[SentimentLabel] = "Positive" )
)
```
Also available: `DATESMTD`, `DATESQTD`, and `DATEADD ( 'DimDate'[Date], -1, MONTH )` for the previous month. Name comparisons explicitly (`Previous Month`, `Previous Year`), never just `Previous Period`.

## Error and blank handling
- Ratios always use `DIVIDE`. Never use `/` with a measure denominator.
- `BLANK()` means no data; 0 means a known zero. Don't use `COALESCE ( ..., 0 )` unless the BRD needs zeros, and state why in the description.
- Use `SWITCH ( TRUE (), ... )` instead of nested `IF`s.
- Never hardcode targets, thresholds, dates or lists. Use a target or lookup table. If none exists, create a stub table and an `ASM`.

## Performance
- Prefer `SUM`, `COUNTROWS` and `DISTINCTCOUNT` over iterators. Use `SUMX` only for true row-by-row logic, and never `SUMX` over a fact just to filter.
- Filter with Boolean predicates on dimension columns inside `CALCULATE` (wrapped in `KEEPFILTERS`), not `FILTER ( 'Fact', ... )`.
- Use `REMOVEFILTERS` to clear filters, and `USERELATIONSHIP` only for inactive date roles.
- Avoid `CROSSJOIN`, `SUMMARIZE` or `ADDCOLUMNS` over fact rows, and `VALUES` on high-cardinality IDs in visual measures.
- Use calculation groups only if the BRD asks for a time or scale switcher across 3 or more measures. Otherwise write explicit measures.

## Format strings (explicit on every measure)
| Type | Format |
|---|---|
| Whole number | `#,0` |
| Decimal | `#,0.00` |
| Percent | `0.0%` |
| Currency | `<symbol>#,0` using the BRD's currency; log an `ASM` if not stated |
| Scaled (K / M) | `#,0.0,"K"` / `#,0.0,,"M"` |

Use `FORMAT()` only in presentation measures.

## Description (one or two lines)
`<Definition>. Uses <date basis>. Blank when <condition>. Depends on [A], [B].` Include the owner only if the BRD names one.
