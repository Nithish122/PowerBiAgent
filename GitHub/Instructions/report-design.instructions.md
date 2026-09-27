---
applyTo: "Output/Implementation/report.yaml,Output/PBIP/**/*.Report/**"
description: Report page, layout, visual selection and visual performance rules (UI/UX + visual performance)
---
# Report Design Rules

## Pages
Every page in `report.yaml` has a `type`, `persona`, `question` and `reqs`.
| Type | Content | Max visuals (excluding slicers, title, buttons) |
|---|---|---|
| executive | 3–6 KPI cards, 1 trend, 1 driver chart, path to detail | 10 |
| analysis | one question, focused slicers, trend or ranked comparison | 10 |
| detail | table or matrix, record count, back button | 8 |
| drillthrough | shows passed context, Back button, works with no or multiple selections | 8 |
| dataQuality | freshness, missing keys, exceptions | 8 |

## Layout (canvas 1280×720, 12-column grid)
- Margin 24, gutter 16, column width 88. Snap every x and width to the grid.
- Header `TXT_PageTitle`: x24 y16 h48, title top-left, with last refresh on the right if useful.
- Filter band (slicers): y72 h48, left to right. Slicers never go anywhere else.
- KPI row: y132 h110. Four cards are 296 wide each (x = 24, 336, 648, 960). For 3 or 6 cards, divide the 1232 width evenly with 16 gutters.
- Main row: y258 h250. Primary visual 8 columns (x24 w816), secondary 4 columns (x856 w400).
- Detail row: y524 h172, full width (x24 w1232).
- Reading order is summary → trend → drivers → detail. Visuals never overlap, and every visual sits inside 1280×720.

## Visual selection
| Question | Visual (`type`) |
|---|---|
| Is the KPI on target? | `card`, or `kpi` if a target exists |
| Change over time | `lineChart` (max 5 series); actual with a rate → `lineClusteredColumnComboChart` |
| Compare or rank categories | `clusteredBarChart`, sorted, top 10–15 |
| Part of whole (at most 5 slices) | `donutChart`; otherwise a bar chart |
| Exact values or records | `tableEx`; grouped with subtotals → `pivotTable` |
| Correlation or outliers | `scatterChart` |
| Filter | `slicer` (dropdown for long lists, search on for more than 20 items) |
Avoid maps, treemaps, decomposition trees and custom visuals unless the BRD asks for them.

## KPI cards and status
- Each card shows one measure with a clear unit. Add a comparison measure (`vs Target`, `vs Previous Month`) when one exists.
- Status uses a text label (`On Target`, `At Risk`, `Below Target`) from a `[Metric] Status` measure, not color alone.
- Thresholds come from a table, never typed into visual formatting.

## Titles, accessibility, interactions
- Every visual has a business title (`Mentions by Platform`), alt text (one sentence on what it shows), and tab order that follows reading order.
- Body text is at least 12pt. Use the theme's colors only, with no per-visual color overrides. Pair red/green with a label or icon.
- Cross-filter by default. Turn off interactions that add no insight (for example, KPI cards filtering each other).
- Date slicer defaults to the BRD period. If none is given, log an `ASM`.

## Performance
- Stay within the max visuals per page. Use Top N or drillthrough instead of axes with thousands of categories.
- Keep tooltip pages small (at most 4 visuals). Avoid measure-driven conditional formatting on large tables.

## Mobile
List the phone order in `report.yaml` (`mobileOrder: [KPI_..., CHT_...]`) only if the BRD requires mobile. Generate the mobile layout only when asked.

## Theme
Use `Templates/Theme/<Domain>Theme.json` if it exists. Otherwise use the template's default theme and log an `ASM`.
