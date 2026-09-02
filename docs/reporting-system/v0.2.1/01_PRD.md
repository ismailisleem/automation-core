# 01 — Product Requirements Document
### Strata — Automation Run Reporting System

---

## 1. Summary

Strata turns the raw output of an automation run into a precise, layered, role-aware report. Its **engine lives in automation-core**, part of the existing product family (`09`): frameworks produce test results and map them into core, which captures them into a **persistent on-disk store** (one folder per run, kept until the user deletes it) and renders **one report application** that spans every stored run — a single run in depth, up to five runs compared, and long-term trends across all runs.

Strata **replaces** the family's existing built-in reporter. It targets four audiences in one product — stakeholders, QA engineers, developers, and automation engineers — and covers **web, API, and mobile as equals**, with **desktop reserved for a future framework** (DEC-06).

## 2. Problem

The framework's current reporter is inadequate on multiple axes: it does not serve non-engineers with a fast, glanceable read of run health; it does not give engineers enough evidence (steps, screenshots, video, trace, network, logs) to decide **is this a real bug or a test/environment problem**; it does not let a user drill through **layers** (suite → feature → test → attempt → step) or search and filter to a precise slice; it has no serious **run-to-run comparison**; and it has no coherent light/dark experience.

The cost of that gap is measured across four roles on every unstable or failing run: the pipeline burns time, QA re-checks evidence by hand, developers argue over whether a failure is theirs, and stakeholders lose trust in the suite. Strata's job is to answer, quickly and defensibly, **"what is the state of the system after this run, and what changed?"**

## 3. Goals

- **G1 — One run, fully understood.** Any user can open a run and read its outcome at their own depth: a stakeholder in 10 seconds, an engineer down to a single failed step's DOM snapshot.
- **G2 — Bug-vs-not triage.** Every failure can be classified (product bug / automation bug / system / environment / known issue / no defect) with the evidence needed to justify the call attached in one place.
- **G3 — Layered navigation.** The same run is navigable through multiple hierarchies (suite, behavior, package, platform, tag, owner, severity, custom layer), with search and filter at every level.
- **G4 — Comparison across up to 5 runs.** Complete diffing: matched tests, status transitions, newly-failing (regressions), newly-passing (fixes), added, removed, still-failing, flaky deltas, and duration deltas.
- **G5 — Analytics.** Dashboards, charts, and statistics for a single run and across history (pass-rate, trends, flaky, most-failed, duration, defect distribution, quality gates).
- **G6 — Evidence-first.** Screenshots, video, trace, network, console, and logs — per test and per step — captured and displayed, per platform.
- **G7 — Two complete themes.** Light and dark, each fully specified; auto is the runtime default.
- **G8 — Ecosystem-embedded, zero-ops.** The reporting engine lives in **automation-core** (`09`); zero-ops means **no hosted backend, no database, no accounts**. Data lives on disk and persists until the user deletes it. A run still auto-opens a useful latest report (parity with today); a loopback `serve` powers the full store/live/large history; `export` powers portable sharing.
- **G9 — Secure by default.** Secrets and PII in captured evidence (headers, bodies, logs, screenshots, video, traces, device logs) are redacted before anything is written to the store; exports default to safe-share (`10`).
- **G10 — Low metadata burden.** The report is useful with zero per-test annotation; owner/severity/feature/etc. are optional and inferable (`11 §9`).
- **G11 — Buildable under tight control.** The spec is precise enough that the implementation and design teams build it without redesigning it.

## 4. Non-goals (v1)

- **N1** — No multi-user accounts, auth, roles/permissions, or cloud hosting. "Roles" in Strata are **view presets**, not access control.
- **N2** — No test **authoring, execution, or scheduling.** Strata reports on runs; it does not run tests.
- **N3** — No hosted SaaS backend or shared team server. (A future networked mode is out of scope but the data model must not preclude it.)
- **N4** — No write-back to the system under test.
- **N5** — Test-management integration (TestRail/Xray/Zephyr) and bug-tracker sync are **link-out only** in v1 (store URLs/keys; no two-way sync).

## 5. Personas & jobs-to-be-done

Roles are **view presets**, not permissions. Each persona has a default landing view and a set of jobs Strata must make fast.

### 5.1 Stakeholder (PM / release manager / lead)
- **Wants:** the fastest possible read of run health and release readiness; trend direction; top risks; a clear Go/No-Go signal.
- **JTBD:** "Is this build good enough to ship?" · "Is quality trending up or down?" · "What's the one number I report upward?"
- **Needs:** Overview dashboard, quality-gate verdict, pass-rate + delta vs previous, top failing areas, no jargon, shareable link.

### 5.2 QA engineer
- **Wants:** to triage every non-pass quickly and decide bug-vs-not with evidence.
- **JTBD:** "Which failures are real?" · "Give me the screenshot/video/trace/steps for this test." · "Show me only new failures since the last run." · "Filter to this suite / this platform / this tag."
- **Needs:** Explorer with deep filter/search, evidence viewer, defect classification workflow, comparison "new failures" view, flaky list.

### 5.3 Developer
- **Wants:** the failures that belong to their code, with root-cause evidence, minimal noise.
- **JTBD:** "What broke, where, and why?" · "Is it my change or a flaky test?" · "Take me from failure → exact step → stack trace → trace/network."
- **Needs:** failures grouped by suite/owner/package, error + stack + diff, trace/network/console, linked issue, history sparkline for the test.

### 5.4 Automation engineer
- **Wants:** suite health — flakiness, broken tests, automation bugs, duration regressions, coverage of layers.
- **JTBD:** "Which tests are unstable?" · "Which failures are automation defects, not product bugs?" · "What got slower?" · "What's newly added/removed vs last run?"
- **Needs:** flaky analytics (retry + history based), broken vs failed split, automation-bug bucket, duration/performance regression, added/removed tests across runs.

## 6. Product principles

1. **Every screen serves a decision.** If a view doesn't help someone decide something, it doesn't ship.
2. **Depth on demand.** Glanceable by default; every element expands to full evidence without a page reload losing context.
3. **The layer is the unit of navigation.** Users think in suites, features, platforms, tags — not just flat test lists. The report pivots between those lenses.
4. **Everything is addressable.** Any state (a run, a lens, a filter, a test, an attempt, a step, a comparison, a theme) is captured in the URL and shareable.
5. **Evidence over prose.** Show the screenshot/trace/diff; don't describe it.
6. **Honest signals.** Flaky is an overlay, broken is not failed, known issues don't masquerade as green. The status model is strict (see `03`/`02 §7`).
7. **Zero-ops.** No hosted backend, DB, or accounts; a run auto-opens a report, `serve`/`export` cover the rest.
8. **Secure by default.** Secrets/PII are redacted before they touch disk; sharing defaults to safe-share (`10`).
9. **Low metadata burden.** Useful with zero per-test annotation; labels are optional and inferred (`11 §9`).

## 7. Architecture summary (detail in `02`, `08`, `09`)

- **Form factor:** the reporting **engine lives in automation-core** (Python); each framework repo maps its evidence in; the orchestrator aggregates cross-platform runs. The **viewer** is a static web app whose built assets are vendored in and served by core (`09`).
- **Store:** `STORE_ROOT` (default `<framework-root>/.strata`, confirmed in `12`) holds `index.json` + one folder per run + the viewer assets. Runs accumulate; **retention is manual-only** — nothing is auto-deleted; writes are atomic and concurrency-safe (`02 §10`).
- **Write path:** frameworks call core's ingestion API (start-run → report-test/step/attempt/attach → finish-run). Core normalizes into the neutral schema, applies **redaction before writing** (`10`), copies media, and finalizes summary/gate/lineage. Supports incremental writes during the run.
- **Read path:** a run auto-opens a useful latest report (parity). Full store/live/large history use loopback **`strata serve`**; **`strata export`** produces portable, serve-free artifacts. `file://` on a lazy-fetch view shows guidance; exported HTML opens directly.
- **Platforms:** the schema is platform-polymorphic — web, API, mobile first-class; **desktop reserved/deferred** (no desktop framework in the family; DEC-06).
- **No backend, no DB, no accounts.**

## 8. Scope — v1 capability list (detailed in `04`)

1. Run Overview dashboard (KPIs, gate verdict, breakdowns, trend spark).
2. Run Explorer with switchable **hierarchy lenses** and deep **search + faceted filter**.
3. Test Detail with **evidence viewer** (steps tree, screenshots, video, trace, network, console, logs, visual diff) — per platform.
4. **Defect triage** workflow (classify, comment, link issue).
5. **Multi-run Comparison** (2–5 runs): status matrix + full diff (new-fail/fixed/added/removed/unchanged/flaky/duration).
6. **Trends & History** across all stored runs (pass-rate, duration, flaky, defect mix).
7. **Flaky analysis** (retry-based + history-based, stability grade).
8. **Quality Gates** (configurable pass criteria → Go/No-Go verdict).
9. **Global search** and **saved filters**, shareable deep links.
10. **Runs management** (list, label, pin, tag, delete — manual retention).
11. **Light/Dark/Auto** theming, fully specified.
12. **Export** (single-run portable HTML; CSV/JSON data export).

**Roadmap (v2, not built in v1 — DEC-07):** AI root-cause/triage assist — optional, off by default, bring-your-own provider key.

Value-add features (justified in `04`): Ownership/Assignee rollups, Severity-weighted health score, "What changed" auto-summary on every run vs previous, Error-signature clustering (group failures by normalized error), Slowest-tests & duration-regression board, Coverage-of-layers view (which suites/features/platforms ran), Timeline/Gantt of parallel execution, Environment fingerprint & drift detection between runs, Annotations/notes on a run, Muted/known-issue management.

## 9. Success metrics

- **M1 — Time-to-verdict:** a stakeholder reaches a Go/No-Go read in ≤ 10s from opening a run (measured in usability review).
- **M2 — Time-to-evidence:** an engineer goes failure → root-cause evidence in ≤ 3 clicks.
- **M3 — Triage completeness:** 100% of non-pass results are classifiable with evidence attached in-view.
- **M4 — Comparison correctness:** the diff of any two stored runs is provably complete (every matched/added/removed/changed case accounted for; see `05` AC).
- **M5 — Zero-ops:** first report view requires no config beyond running the framework + one open/serve command.
- **M6 — Parity+:** every capability of the replaced reporter is met or exceeded (parity checklist in `08`).

## 10. Constraints & assumptions

- **C1** — No server/DB/accounts. Everything is filesystem-backed and local.
- **C2** — Runs persist until manually deleted; the store can grow large → the viewer must lazy-load and the store must shard large runs (see `02`).
- **C3** — `file://` fetch limits → serve command is the primary path; export is the portable path.
- **A1** — The reporting **engine is Python in automation-core**; the **viewer** is static web assets (a TS/JS SPA build) owned and distributed by core (`09`). The neutral schema is language-agnostic.
- **A2** — Cross-run test identity is derivable deterministically from test metadata (`testId`/`historyId`, see `02`); frameworks that can't provide stable identity degrade to name-based matching with a documented caveat.

## 11. Glossary

- **Run (Launch/Build):** one execution of the suite; the top-level unit Strata stores.
- **Store:** the on-disk collection of all runs at `STORE_ROOT`.
- **Result:** the outcome of one test within a run.
- **Attempt/Retry:** one execution of a test; a test may have several attempts (basis of flaky detection).
- **Step / Sub-step:** an action or assertion inside a test; may nest.
- **Lens / Hierarchy dimension:** a way of grouping the same results (suite, behavior, package, platform, tag, owner, severity, custom).
- **Status model:** `rawStatus` = passed | failed | broken | skipped | blocked; `effectiveStatus` = rawStatus plus `known`; `flaky` = boolean overlay, never a terminal status (see `03`/`02 §7`).
- **Defect type:** triage classification of a non-pass (product bug / automation bug / system / environment / to-investigate / known issue / no defect).
- **Evidence:** screenshots, video, trace, network, console, logs, visual diff attached to a result or step.
- **Quality Gate:** configurable rule set producing a Go/No-Go verdict for a run.
- **Flaky:** a test that passed after retry, or is unstable across history.
