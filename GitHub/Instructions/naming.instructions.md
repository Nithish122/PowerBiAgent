---
applyTo: "Output/**"
description: Naming rules for model objects, report objects and PBIP folders
---
# Naming Rules

## General
- Physical objects (tables, columns, PBIP folders) use PascalCase with no spaces.
- Business-facing objects (measures, hierarchies, folders, pages, visual titles) use Title Case with spaces.
- No abbreviations, except `KPI`, `YTD`, `MTD`, `QTD`, `YoY`, `RLS`, `%`, and terms defined in the BRD glossary. Write `Customer`, not `Cust`; `Previous Year`, not `PY`.
- No versions, dates, `Final`, `Copy`, `Test`, `New`, environment names or personal names in any name.

## Tables
| Role | Pattern | Example |
|---|---|---|
| Fact | `Fact[Process]` | `FactMention` |
| Dimension | `Dim[Entity]` | `DimPlatform`, `DimDate` |
| Bridge | `Bridge[EntityA][EntityB]` | `BridgeUserRegion` |
| Lookup | `Lookup[Subject]` | `LookupSentimentThreshold` |
| Aggregation | `Agg[Process][Grain]` | `AggMentionMonth` |
| Measure table | `Measures` | — |

## Columns
- Business columns use full terms: `PlatformName`, `MentionDate`, `SentimentScore`.
- Surrogate key `[Entity]Key`, source ID `[Entity]BusinessKey`. The fact foreign key uses the exact same name as the dimension key.
- Booleans start with `Is`, `Has` or `Can`, such as `IsVerifiedAuthor`.
- Audit columns: `CreatedDateTime`, `UpdatedDateTime`, `LoadDateTime`, `SourceSystem`.

## Measures
| Purpose | Pattern | Example |
|---|---|---|
| Base | `[Metric]` | `Sentiment Score` |
| Count | `[Entity] Count` | `Mention Count` |
| Percent | `[Metric] %` | `Positive Mention %` |
| Prior period | `[Metric] Previous [Period]` | `Mention Count Previous Month` |
| YoY | `[Metric] YoY` / `[Metric] YoY %` | `Mention Count YoY %` |
| Rolling | `[Metric] Rolling N Months` | `Mention Count Rolling 12 Months` |
| Target / variance | `[Metric] Target`, `[Metric] Variance`, `[Metric] Variance %` | `Net Sentiment Variance %` |
| Status / label | `[Metric] Status`, `[Metric] Label` | `Net Sentiment Status` |

Never use prefixes like `m_`, table names or folder names in a measure name.

## Other objects
- Display folders: `Measures\Base`, `Measures\Time Intelligence`, `Measures\KPI`, `Measures\Targets`, `Measures\Variance`, `Measures\Presentation`. Never `Other`, `Misc` or `Temp`.
- Hierarchies: `[Subject] Hierarchy` with business level names (`Year`, `Quarter`, `Month`).
- PBIP: `[Domain][Subject]` in PascalCase, used for the `.pbip` file, `.Report` folder and `.SemanticModel` folder (for example, `SocialSentimentAnalytics`).
- Report title: `[Domain] [Outcome]`, such as `Social Media Sentiment Analysis`.
- Pages: purpose in Title Case, such as `Executive Summary`, `Sentiment Analysis`, `Mention Detail`. Never `Page 1` or `Overview`.
- Visual names (technical, not shown to users): `KPI_`, `CHT_`, `TBL_`, `SLR_`, `BTN_`, `NAV_`, `SHP_` or `TXT_` + PascalCase purpose (for example, `KPI_MentionCount`, `CHT_SentimentTrend`). Unique per page.
- Relationships: in `model.yaml`, add `role: <Order Date | Platform | ...>`. In TMDL, the relationship name is a unique GUID.
