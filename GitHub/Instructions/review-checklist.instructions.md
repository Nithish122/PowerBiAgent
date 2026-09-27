---
applyTo: "Output/**"
description: Self-review the agent runs on its own outputs before reporting a phase as done
---
# Agent Self-Review

Run only the checks for the phase just completed. Fix a failing check in the files before replying. Report only what is still failing, in this format (one line each, max 15 lines):

`[BLOCKER|MAJOR|MINOR] <file> — <object> — <issue> — <fix or GAP/ASM id>`

If everything passes, reply `Review: pass` and list any manual checks.

## Severity
- **BLOCKER**: the file will not parse or load, a binding is broken, a DAX reference is wrong, or there is a secret or personal data in a file.
- **MAJOR**: incorrect semantics, such as wrong grain, an unapproved bi-directional or many-to-many relationship, or a KPI without a definition.
- **MINOR**: naming, a missing description or display folder, or formatting.

## Phase 1 — requirements
- [ ] Every KPI, fact, dimension, page and security rule has `reqs`.
- [ ] Every fact has one grain. No mixed grains.
- [ ] Every `unknown` source, `null` target and ambiguous term has a `GAP`.
- [ ] Every non-blocking `GAP` that the design depends on has an `ASM`.

## Phase 2 — model.yaml / report.yaml
- [ ] Star schema. Relationships go from dimension to fact, single direction, many-to-one.
- [ ] Relationship key data types match on both sides.
- [ ] Every table has `source.m`. Stubs are typed and flagged `status: stub`.
- [ ] Every measure has `expression`, `formatString`, `displayFolder` and `description`.
- [ ] DAX: `DIVIDE` for ratios, `VAR`/`RETURN` for multi-step logic, no circular references, no placeholders.
- [ ] Every `Table[Column]` and `[Measure]` referenced in measures, roles and visuals exists in `model.yaml` with exact case.
- [ ] Every page in `01-requirements.yaml` exists in `report.yaml`. Visuals fit inside the canvas and do not overlap.
- [ ] Names follow `naming.instructions.md`.

## Phase 3 — TMDL
- [ ] One `.tmdl` file per table in `definition/tables/`. `model.tmdl`, `database.tmdl` and `relationships.tmdl` are present. Template files are unchanged apart from names and paths.
- [ ] Indentation is consistent (tabs). Every object from `model.yaml` is present, and there are no extras.
- [ ] Each table has one partition with the M expression copied from `model.yaml`.
- [ ] Names with spaces or special characters are quoted.
- [ ] No credentials in M expressions or `expressions.tmdl`.

## Phase 4 — PBIP report
- [ ] The `.pbip` file and `definition.pbir` point to the correct relative `../<Name>.SemanticModel` path.
- [ ] Every visual from `report.yaml` exists, with field bindings that resolve to model objects.
- [ ] Every page name and visual `id` is unique.

## Manual checks (never mark as passed; list them for the user)
- Project opens in Power BI Desktop and refreshes.
- KPI values reconcile to source totals.
- RLS tested with "View as role".
- Performance Analyzer on the heaviest page.
- Accessibility (tab order, alt text, contrast).
