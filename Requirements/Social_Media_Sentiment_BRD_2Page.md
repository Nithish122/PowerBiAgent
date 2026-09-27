# Social Media Sentiment Analytics — BRD (2-Page Release)

| Field | Detail |
|---|---|
| Version | 1.1 (reduced scope of BRD v1.0) |
| Owner | Marketing Analytics Product Owner |
| Sponsor | Chief Marketing Officer |
| Release scope | 2 report pages: Executive Overview, Sentiment Intelligence |
| Purpose | Sole business input for the Power BI agent (first executable release) |

## 1. Objective
Give leadership and brand teams one trusted view of social sentiment, engagement and reputational risk. This release covers the headline and sentiment-driver views only. The remaining pages from BRD v1.0 are listed in section 10.

## 2. Users
| Persona | Use |
|---|---|
| Executive Leadership | Monthly review of conversation health and risk |
| Brand Management | Daily sentiment monitoring; owns sentiment definitions |
| Marketing Teams | Topic and platform performance; owns engagement definitions |

Device: desktop only for this release.

## 3. Business questions in scope
| ID | Question | Page |
|---|---|---|
| Q1 | What is the public sentiment toward our brand and AI-related topics? | 1, 2 |
| Q2 | Is overall sentiment improving or deteriorating over time? | 1, 2 |
| Q3 | Which topics generate the most negative sentiment? | 1, 2 |
| Q5 | Which topics generate the highest engagement? | 1 |
| Q6 | Which platforms drive the most interactions? | 1 |
| Q13 | What content presents reputational risk? | 1 |
| Q16 | How does current-period sentiment compare with the prior period? | 1, 2 |

## 4. Source data
- **Source:** one CSV file, `Data/social_media_sentiment.csv`, with one row per public social post and daily refresh.
- **Grain:** one row per post. Each post has exactly one date, platform, topic, sentiment, location and language.
- **Expected columns** (rename these to match the actual file before Phase 1):

| Column | Type | Description |
|---|---|---|
| post_id | text | Unique post reference |
| post_datetime | datetime | Publish date and time |
| platform | text | Social network |
| topic | text | Topic or theme (includes AI-related themes) |
| sentiment | text | Positive, Neutral or Negative |
| sentiment_score | decimal | Strength of the sentiment classification |
| location | text | Country |
| region | text | Region grouping of the country |
| language | text | Language of the post |
| follower_count | whole number | Author follower count at posting time |
| likes | whole number | Like count |
| shares | whole number | Share or retweet count |
| comments | whole number | Comment or reply count |

Excluded from the model in this release: post content, hashtags, emotion, user ID, user type and verification status.

## 5. Data model expectations
- **Fact:** Social Post (one row per post), holding sentiment score, follower count (reach), likes, shares and comments.
- **Dimensions:** Date, Platform, Topic, Sentiment, Location (country and region), Language.
- **Date:** daily grain covering the full data range, with Year, Quarter, Month and Week. Time of day is not needed in this release.
- **Relationships:** every dimension filters the fact, single direction.

## 6. KPI definitions
All KPIs are calculated over the selected filters. **Prior period** means the period of the same length immediately before the selected date range (for example, the previous 30 days).

| ID | KPI | Definition / calculation | Format | Owner |
|---|---|---|---|---|
| KPI-01 | Post Count | Number of posts | #,0 | Marketing |
| KPI-02 | Positive Sentiment Share | Positive posts ÷ Post Count | 0.0% | Brand |
| KPI-03 | Neutral Sentiment Share | Neutral posts ÷ Post Count | 0.0% | Brand |
| KPI-04 | Negative Sentiment Share | Negative posts ÷ Post Count | 0.0% | Brand |
| KPI-05 | Net Sentiment Score | Positive Sentiment Share − Negative Sentiment Share | 0.0% | Brand |
| KPI-06 | Average Sentiment Strength | Average of sentiment score | 0.00 | Analytics / Brand |
| KPI-07 | Total Engagement | Sum of likes + shares + comments | #,0 | Marketing |
| KPI-08 | Audience Reach | Sum of follower count | #,0.0,,"M" | Marketing |
| KPI-09 | High-Risk Content Count | Negative posts where follower count ≥ Reach Threshold OR engagement ≥ Engagement Threshold | #,0 | Brand |
| KPI-10 | Sentiment Change vs Prior Period | Net Sentiment Score − Net Sentiment Score (prior period) | +0.0%;-0.0% | Brand |
| KPI-11 | Rolling 30-Day Net Sentiment | Net Sentiment Score over the 30 days ending on the last selected date | 0.0% | Brand |
| KPI-12 | Topic Net Sentiment | Net Sentiment Score evaluated per topic (same measure as KPI-05, grouped by topic) | 0.0% | Brand |

**Risk thresholds (provisional, pending Brand Management approval):** Reach Threshold = 100,000 followers; Engagement Threshold = 1,000 interactions. Store both in a threshold table, not in DAX.

## 7. Report pages

### Page 1 — Executive Overview
- **Audience:** Executive Leadership, Brand Management
- **Questions:** Q1, Q2, Q3, Q5, Q6, Q13, Q16
- **Filters (slicers):** Date (default last 30 days), Platform, Topic, Location

| # | Visual | Type | Content |
|---|---|---|---|
| 1 | Net Sentiment Score | KPI card | KPI-05, with KPI-10 shown as "vs prior period" |
| 2 | Positive Sentiment Share | KPI card | KPI-02 |
| 3 | Negative Sentiment Share | KPI card | KPI-04 |
| 4 | Total Engagement | KPI card | KPI-07 |
| 5 | Audience Reach | KPI card | KPI-08 |
| 6 | High-Risk Content Count | KPI card | KPI-09, with status label |
| 7 | Net Sentiment Trend | Line chart | KPI-05 by week |
| 8 | Engagement by Platform | Bar chart (sorted descending) | KPI-07 by platform |
| 9 | Top Negative Topics | Bar chart (top 10) | KPI-04 by topic, sorted descending |
| 10 | Risk Summary | Table | Topic, platform, KPI-09, KPI-04, KPI-08; sorted by KPI-09 descending |

### Page 2 — Sentiment Intelligence
- **Audience:** Brand Management, Marketing Teams
- **Questions:** Q1, Q2, Q3, Q16
- **Filters (slicers):** Date (default last 30 days), Platform, Topic, Location, Sentiment, Language

| # | Visual | Type | Content |
|---|---|---|---|
| 1 | Net Sentiment Score | KPI card | KPI-05, with KPI-10 as "vs prior period" |
| 2 | Positive Sentiment Share | KPI card | KPI-02 |
| 3 | Neutral Sentiment Share | KPI card | KPI-03 |
| 4 | Negative Sentiment Share | KPI card | KPI-04 |
| 5 | Average Sentiment Strength | KPI card | KPI-06 |
| 6 | Rolling 30-Day Net Sentiment | KPI card | KPI-11 |
| 7 | Sentiment Trend vs Rolling 30 Days | Line chart | KPI-05 and KPI-11 by day |
| 8 | Sentiment Mix by Month | 100% stacked bar | Post Count by month, split by sentiment |
| 9 | Topic Net Sentiment | Bar chart (ranked, top 15) | KPI-12 by topic |
| 10 | Sentiment by Platform | Matrix | Rows: platform; values: KPI-02, KPI-03, KPI-04, KPI-05 |

## 8. Report rules
- Every visual has a business title, clear units and alt text.
- Status on KPI-09 uses a text label as well as colour.
- Slicers on Topic and Location have search enabled. Each page has a Reset Filters button.
- Performance target: each page loads in about 5 seconds and slicers respond in about 3 seconds.

## 9. Security
- Public data only; no personal customer data.
- Access is controlled through workspace permissions for the stakeholder groups named above. No RLS is required in this release.
- Author-level detail is not included in this release.

## 10. Out of scope (future releases)
- Pages: Engagement Analytics, Topic Insights, Emotion Analysis, Content Risk Monitoring (full action list), Geographic Analysis (map), User Behavior Analysis.
- Questions: Q4, Q7–Q12, Q14, Q15, Q17.
- KPIs: Average Engagement per Post, Engagement Rate, Amplification Share, Conversation Response Share, author KPIs, topic share and contribution, Negative Engagement Share, Negative Reach Exposure, Escalation Candidate Count, growth KPIs.
- Features: drillthrough pages, decomposition tree, mobile layout, market-based RLS, author masking or OLS.

## 11. Open decisions
| ID | Decision | Owner |
|---|---|---|
| OD-01 | Confirm risk thresholds (100,000 followers / 1,000 interactions) | Brand Management |
| OD-02 | Confirm "prior period" = same-length preceding period | Brand Management |
| OD-03 | Confirm source file name and column names | Analytics Teams |
