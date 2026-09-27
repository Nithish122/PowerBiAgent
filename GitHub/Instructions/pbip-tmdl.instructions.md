---
applyTo: "Output/PBIP/**"
description: How to write PBIP / TMDL / PBIR files that open in Power BI Desktop
---
# PBIP / TMDL Build Rules

## Template-first (mandatory)
Do not invent JSON schemas or file layouts. Start from the Desktop-generated template in `Templates/PBIP/`:
1. Copy `Templates/PBIP/` to `Output/PBIP/`, then rename the `.pbip` file, the `.Report` and `.SemanticModel` folders, and the path inside `definition.pbir` to `<Name>`.
2. Keep `definition.pbism`, `definition.pbir`, `.platform`, `database.tmdl`, `version.json`, `report.json` and theme resources exactly as generated. Change only names and paths.
3. For every new visual, page or table file, mirror the `$schema` and structure of the matching template file.
If `Templates/PBIP/` is missing, stop and ask the user to create it (see README).

## Semantic model layout
```
<Name>.SemanticModel/
  definition.pbism          (from template)
  definition/
    database.tmdl           (from template)
    model.tmdl              (template; add a `ref table <Name>` line per table if the file uses ref lines)
    relationships.tmdl
    tables/<TableName>.tmdl (one file per table)
    roles/<RoleName>.tmdl   (only if RLS)
```
In `model.tmdl`, keep `discourageImplicitMeasures`. Keep `annotation __PBI_TimeIntelligenceEnabled = 0` so auto date/time stays off.

## TMDL syntax
- Indent with **tabs**: object at 1 tab, its properties at 2 tabs, and multi-line expressions at 3 tabs (one level deeper than the properties).
- Quote names that contain spaces or special characters with single quotes: `measure 'Mention Count' =`.
- A description is a `///` line directly above the object, at the same indentation.
- A boolean property is written as a bare keyword (`isHidden`, `isKey`), never `isHidden: true`.

```
table FactMention

	column MentionKey
		dataType: int64
		isHidden
		summarizeBy: none
		sourceColumn: MentionKey

	column SentimentScore
		dataType: double
		isHidden
		summarizeBy: none
		sourceColumn: SentimentScore

	partition FactMention = m
		mode: import
		source =
				let
					Source = #table(type table [MentionKey = Int64.Type, DateKey = Int64.Type, SentimentScore = number], {})
				in
					Source
```

```
table Measures

	/// Count of mentions in the current filter context. Depends on FactMention.
	measure 'Mention Count' = COUNTROWS ( 'FactMention' )
		formatString: #,0
		displayFolder: Measures\Base

	measure 'Mention Count YoY %' =
			VAR CurrentValue = [Mention Count]
			VAR PreviousValue = [Mention Count Previous Year]
			RETURN
				DIVIDE ( CurrentValue - PreviousValue, PreviousValue )
		formatString: 0.0%
		displayFolder: Measures\Time Intelligence

	column Placeholder
		dataType: string
		isHidden
		sourceColumn: Placeholder

	partition Measures = m
		mode: import
		source = #table(type table [Placeholder = text], {})
```

Date table (built in M; `dataCategory: Time` plus `isKey` on `Date` marks it as the date table):
```
table DimDate
	dataCategory: Time

	column Date
		dataType: dateTime
		isKey
		formatString: dd-mmm-yyyy
		summarizeBy: none
		sourceColumn: Date

	column MonthName
		dataType: string
		sortByColumn: MonthNumber
		sourceColumn: MonthName

	hierarchy 'Calendar Hierarchy'
		level Year
			column: Year
		level Month
			column: MonthName
```

Relationships (`relationships.tmdl`). The default is many-to-one, single direction, active.
```
relationship 3f2a9c1e-5b7d-4e8a-9c0f-1a2b3c4d5e6f
	fromColumn: FactMention.DateKey
	toColumn: DimDate.DateKey
```
Optional properties: `isActive: false`, `crossFilteringBehavior: bothDirections` (needs an `ASM`). Use a new random GUID per relationship, and quote table names that contain spaces (`'Dim Date'.DateKey`).

Role (`roles/RegionalManager.tmdl`):
```
role 'Regional Manager'
	modelPermission: read

	tablePermission DimRegion = [ManagerEmail] = USERPRINCIPALNAME()
```

## Report (PBIR folder format)
```
<Name>.Report/
  definition.pbir                      (from template; path ../<Name>.SemanticModel)
  definition/
    version.json, report.json          (from template)
    pages/pages.json                   (pageOrder lists the page folder names)
    pages/<pageName>/page.json         (name = folder name, displayName = page title, width 1280, height 720)
    pages/<pageName>/visuals/<visualName>/visual.json   (name = folder name, e.g. KPI_MentionCount)
```
- Copy `visual.json` structure from the template visual. Set `position` (x, y, z, width, height, tabOrder) from `report.yaml`.
- Field binding pattern inside `query.queryState.<Role>.projections[]`:
  - Measure: `{"field":{"Measure":{"Expression":{"SourceRef":{"Entity":"Measures"}},"Property":"Mention Count"}},"queryRef":"Measures.Mention Count"}`
  - Column: `{"field":{"Column":{"Expression":{"SourceRef":{"Entity":"DimDate"}},"Property":"MonthName"}},"queryRef":"DimDate.MonthName"}`
- Roles: `card` → `Values`; `lineChart` / `clusteredBarChart` / `donutChart` → `Category` + `Y`; `tableEx` / `slicer` → `Values`. If a visual type isn't in the template, copy its role names from a Desktop-generated example and don't guess.
- `Entity` and `Property` must match the TMDL table and object names exactly, including case.

## Never
- Credentials, server passwords or tokens in M. Use M parameters in `expressions.tmdl` with placeholder values only.
- Rewriting or reformatting files that don't need to change.
- Git operations (commit, push, branch) unless explicitly asked.
