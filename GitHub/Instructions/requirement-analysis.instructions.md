---
applyTo: "Requirements/**,Output/Architecture/**"
description: How to analyze the BRD and write 01-requirements.yaml
---
# Requirement Analysis

## Goal
Convert the BRD into `Output/Architecture/01-requirements.yaml`. This file must be complete enough to design the model and report without reading the BRD again.

## Rules
- Read the whole BRD once. Quote source wording in `text` fields (keep it short). Everything else is structured.
- Give each item a stable ID: `REQ-`, `KPI-`, `GAP-`, `ASM-`. Downstream files reference these IDs.
- Classify, don't narrate. No prose sections, no methodology explanation.
- A KPI must link to a business goal and have a definition. A value shown on a chart is a metric, not a KPI.
- Derive **facts** from events (posted, commented, mentioned, scored) and state one grain per fact. Never mix grains in one fact (for example, post-level with daily targets).
- Derive **dimensions** from descriptive nouns (platform, date, brand, campaign, author, region). Reuse one dimension across facts. Always include `DimDate` if time is analyzed.
- Map each BRD question to a report page and its visual intent.
- Missing definition, grain, source, target or security rule → add a `GAP`. If you can proceed safely, add an `ASM` with a conservative choice. If not, set `blocking: true`.
- Ambiguous words (current, active, top, significant, positive sentiment) need a `GAP` unless the BRD defines them.

## Schema
```yaml
document: { title: , version: , owner: , date: }
scope: { in: [ ], out: [ ] }
glossary:
  - { term: Net Sentiment, definition: "<BRD definition>", ref: "BRD §3" }   # only terms the BRD defines; undefined terms → GAP
personas:
  - { id: PER-01, name: Marketing Manager, needs: "track brand sentiment weekly", device: desktop }
requirements:
  - { id: REQ-001, type: goal, text: "<short BRD quote>", ref: "BRD §2.1" }
    # type: goal | question | kpi | metric | fact | dimension | security | visual | frequency | drillthrough | nonfunctional
kpis:
  - id: KPI-001
    name: Net Sentiment Score
    definition: (positive - negative) / total mentions
    numerator: positive mentions
    denominator: total mentions
    grain: mention
    timeBasis: calendar week
    target: null            # null → GAP
    format: "0.0%"
    reqs: [REQ-002]
facts:
  - { name: FactMention, grain: one row per mention, dates: [MentionDate], measures: [SentimentScore], source: SRC-01, reqs: [REQ-004] }
dimensions:
  - { name: DimPlatform, key: PlatformKey, attributes: [PlatformName], hierarchies: [ ], reqs: [REQ-005] }
security:
  - { role: Regional Manager, filterOn: DimRegion, rule: "own region only", reqs: [REQ-009] }
pages:
  - { name: Executive Summary, type: executive, persona: PER-01, question: "Is sentiment improving?", visuals: [KPI cards, weekly trend], reqs: [REQ-010] }
sources:
  - { id: SRC-01, system: <from BRD>, object: <table/file/API>, refresh: daily, status: known }   # known | unknown
gaps:
  - { id: GAP-001, reqs: [KPI-001], issue: "No target defined", severity: major, blocking: false, question: "What is the NSS target?" }
assumptions:
  - { id: ASM-001, gap: GAP-001, decision: "Show KPI without target status", impact: "No red/green status", status: proposed }
```

## Done when
- Every BRD requirement has a `REQ` ID.
- Every KPI, fact, dimension, page and security rule references at least one `REQ`.
- Every `unknown` source, `null` target or undefined term has a `GAP`.
- Every `REQ` maps to at least one KPI, page or security rule, and every KPI maps to a page. Unmapped items are listed as gaps.
