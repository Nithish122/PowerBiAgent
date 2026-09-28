# Phase 1 Solution Design

## Model overview

| Table | Grain | Purpose | Principal reqs |
|---|---|---|---|
| FactPost | One row per public post | Holds the social data facts for sentiment, engagement, and reach | REQ-009 |
| DimDate | One row per date | Supports daily, weekly, monthly, and prior-period analysis | REQ-003, REQ-008 |
| DimPlatform | One row per platform | Groups post activity by social channel | REQ-006 |
| DimTopic | One row per topic | Supports negative sentiment and engagement analysis by theme | REQ-004, REQ-005 |
| DimSentiment | One row per sentiment label | Enables sentiment mix and share calculations | REQ-002, REQ-003 |
| DimLocation | One row per country/region | Filters activity by geography | REQ-004, REQ-006, REQ-007 |
| DimLanguage | One row per language | Filters sentiment views by language | REQ-010 |
| Measures | Hidden calculation table | Holds report measures and KPI formulas | KPI-001 to KPI-012 |

## KPI list

- Post Count — count of posts in the selected filter context
- Positive Sentiment Share — positive posts divided by total posts
- Neutral Sentiment Share — neutral posts divided by total posts
- Negative Sentiment Share — negative posts divided by total posts
- Net Sentiment Score — positive share minus negative share
- Average Sentiment Strength — average sentiment score
- Total Engagement — likes plus shares plus comments
- Audience Reach — sum of follower count
- High-Risk Content Count — negative posts meeting reach or engagement thresholds
- Sentiment Change vs Prior Period — current net sentiment minus prior-period net sentiment
- Rolling 30-Day Net Sentiment — 30-day trailing net sentiment
- Topic Net Sentiment — net sentiment by topic

## Report pages

### Executive Overview
- Audience: Executive Leadership, Brand Management
- Questions: Q1, Q2, Q3, Q5, Q6, Q13, Q16
- Layout: KPI cards, trend, platform performance, negative topics, risk summary
- Filters: Date, Platform, Topic, Location

### Sentiment Intelligence
- Audience: Brand Management, Marketing Teams
- Questions: Q1, Q2, Q3, Q16
- Layout: KPI cards, rolling sentiment, monthly mix, topic ranking, platform matrix
- Filters: Date, Platform, Topic, Location, Sentiment, Language

## Security and privacy

- Public data only; no personal customer data
- No author-level detail in this release
- Access controlled by workspace permissions; no RLS required at this phase

## Performance notes

- Use a star schema with direct dimension-to-fact relationships
- Keep the model import-based unless a latency requirement is confirmed
- Prefer a date table built in M with a single contiguous range covering all dates
- Use slicers with search enabled on Topic and Location and a reset button on every page

## Risks and open decisions

- Source file name and schema need confirmation before final model build
- KPI targets / thresholds are not approved yet and remain provisional
- Reach and engagement thresholds are stored as configuration values, not hard-coded DAX
- Prior period and risk logic are defined in the BRD but still need final business sign-off

## Roadmap

- Phase 2: convert requirements and solution design into model and report specification
- Phase 3: build the semantic model in PBIP/TMDL and validate column names and relationships
- Phase 4: build the two report pages and finalize visual layouts and performance tuning
