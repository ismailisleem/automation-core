# Strata — Automation Run Reporting System
### Business & Requirements Package (specification)

> **Working product name:** **Strata** *(placeholder — see decision DEC-05; if renamed, change the `PRODUCT_NAME` token in one place across the package).*
> **Package version:** **v0.2.1** — realigned to the existing product family (v0.2), plus a consistency cleanup (v0.2.1): split DEC-*/DATA-* IDs, status-model and gate-schema consistency, health-score/known-debt, and corrected cross-references.
> **Status:** requirements/spec package intended to become the frozen source of truth before design and implementation.
> **Consumers:** the implementation team (build), the design team (screens), and reviewers (approval).

---

## 0.1 What this package is

Strata is a **run-based automation reporting system**, delivered as part of the existing automation product family. After (or during) a run, the automation framework hands results to the reporting engine in **automation-core**; the engine writes each run to a persistent on-disk store and renders one report application spanning all stored runs — a single run in depth, up to five runs compared, and long-term trends. It **replaces the current built-in reporter**.

This package is the **single source of truth** for what gets built. Deep implementation design (repository internals, concurrency mechanisms, CI wiring) lives in the implementation team's own technical design document; this package defines product requirements and contracts, not internal implementation design.

## 0.2 The product family this targets (see `09`)

The reporting system is **not** a new standalone monorepo. It lives inside an existing Python multi-repo family:

| Repo | Owns (for reporting) |
|------|----------------------|
| **automation-core** | reporting engine, neutral schema/models, store writer, viewer assets, CLI (`serve`/`export`/`delete`/`migrate`), metrics, history, comparison, shared report tests, neutral importers |
| **web-automation-framework** | browser automation + web-specific evidence capture and mapping into core |
| **api-automation-framework** | API request/response, contract/schema evidence, API redaction defaults, mapping into core |
| **mobile-automation-framework** | Appium/device/context/native/hybrid/mobile-web evidence capture and mapping into core |
| **full-stack-automation-orchestrator** | cross-platform orchestration + combined-journey reporting; coordinates and aggregates, does **not** duplicate framework logic |

Ownership boundaries, the ingestion boundary, and cross-platform/combined reporting are specified in `09_EXISTING_ECOSYSTEM_INTEGRATION.md`.

## 0.3 Read order

| # | File | Purpose | Primary consumer |
|---|------|---------|------------------|
| 00 | `00_README.md` | This index, rules, decision log | All |
| 01 | `01_PRD.md` | Vision, personas, scope, goals, glossary | All |
| 02 | `02_DATA_MODEL.md` | Neutral run/result schema, store layout, ingestion + integrity contract | Implementation |
| 03 | `03_INFORMATION_ARCHITECTURE.md` | Layer model, status/defect taxonomy, comparability/lineage, search, navigation, state | Implementation + Design |
| 04 | `04_FEATURE_SPEC.md` | Every page, chart, function (incl. comparison, exports) | Implementation + Design |
| 05 | `05_AC_MATRIX.md` | Acceptance criteria — the definition of done | Implementation (QA gate) |
| 06 | `06_DESIGN_SYSTEM.md` | Visual identity, tokens (light + dark), components, charts, a11y, responsive | Design |
| 07 | `07_UI_SCREEN_PROMPTS.md` | Screen-by-screen governance incl. mobile + unavailable-data states | Design |
| 08 | `08_BUILD_HANDOFF.md` | Ecosystem-aligned stack, ownership, rollout order, validation | Implementation |
| 09 | `09_EXISTING_ECOSYSTEM_INTEGRATION.md` | Repo ownership, ingestion boundary, orchestrator/combined reporting | Implementation |
| 10 | `10_SECURITY_PRIVACY_REDACTION.md` | Secret/PII redaction, safe-share, loopback, artifact safety | Implementation |
| 11 | `11_METRICS_AND_DEFAULTS.md` | Exact formulas + defaults: health, gates, risk, flaky grade, regression, baseline, lineage | Implementation |
| 12 | `12_MIGRATION_AND_ROLLOUT.md` | Replacing the current reporter across the repos | Implementation |
| 13 | `13_FIXTURES_AND_VALIDATION.md` | Concrete fixtures + validation matrix for QA | Implementation |

> Files 09–13 were added in v0.2; files 01–07 fully absorb the v0.2 decisions and the v0.2.1 cleanup (ecosystem alignment, status model, security, metrics, exports, responsive, desktop deferral). This README's decision log is the quick reference.

## 0.4 Rules for implementation and design

1. **Do not invent scope.** If a requirement is absent, raise it — do not fill gaps with assumptions. Every screen, field, chart, and state must trace to `04`/`05`.
2. **The data model is a contract.** `02` defines the exact JSON written and read. Do not rename, restructure, or "improve" fields without an approved edit to that file.
3. **Design obeys `06`/`07` literally.** Only the screens defined, only the tokens/components defined. No off-spec colours, fonts, layouts, or liberties.
4. **Every feature ships light + dark** and meets the responsive + accessibility bar (`06`, `05` global rows).
5. **Acceptance is `05`.** A feature is done only when its P0 rows pass.
6. **Security is a requirement.** Redaction and safe-share defaults in `10` are P0, not optional.
7. **No filler in shipped UI or docs.** Copy is functional and specific.
8. **Public-ready docs.** Copy is neutral and product-focused. AI appears only as an optional product capability, never as implementation attribution.

## 0.5 Token glossary

- `PRODUCT_NAME` → `Strata` (placeholder; DEC-05).
- `STORE_ROOT` → the fixed on-disk report-store path owned by the framework run (default `<framework-root>/.strata`; final path confirmed in `09`/`12`).
- `CORE` → **automation-core**, which owns the reporting engine, store writer, viewer assets, and CLI.

## 0.6 Decision log

| ID | Decision | Resolution |
|----|----------|-----------|
| DEC-01 | Reporting-engine location | **automation-core** owns engine, schema, store, viewer assets, CLI, metrics, history, comparison, importers (`09`) |
| DEC-02 | Evidence-capture ownership | Each framework repo owns its platform's capture + mapping into core (`09`) |
| DEC-03 | Orchestrator role | Coordinates cross-platform runs + combined-journey reports; no duplication of framework logic (`09`) |
| DEC-04 | Viewer technology | A **static web app** whose built assets are vendored in and served by core; SPA-framework choice is the implementation team's, subject to `08` constraints |
| DEC-05 | Product name | `Strata` working name — **confirm or change before design/build** to avoid rename drift |
| DEC-06 | Desktop platform | **Deferred**: schema keeps a reserved desktop evidence shape; no desktop adapter or desktop P0 acceptance in v1 (revises the earlier "all platforms first-class" intent, since no desktop framework exists) |
| DEC-07 | AI assist (F14) | **Deferred to v2/backlog**; if ever shipped, off by default, bring-your-own-key only |
| DEC-08 | Zero-ops vs serve | Zero-ops = **no hosted backend, no database, no accounts**. A run still **auto-opens a useful latest report** (parity with current UX); `serve` (loopback-only) powers the full store/live/large history; `export` powers portable sharing |
| DEC-09 | Status model | `rawStatus` (execution) + `effectiveStatus` (display/gate grouping incl. `known`) + `flaky` boolean overlay — flaky is **never** a terminal status (`02`/`03`) |
| DEC-10 | Security/redaction | Default-on secret/PII redaction, loopback-only serve, safe-share exports, artifact safety (`10`) — P0 |
| DEC-11 | Metrics/gates | Exact defaults + a **safe structured rule schema** (metric/operator/threshold/severity/scope), no expression evaluation (`11`) |
| DEC-12 | Rollout | Core first → release tag → framework pin updates → orchestrator update → post-merge CI + local report validation (`12`) |
| DEC-13 | Metadata burden | Owner/severity/feature/component/team/case/issue labels are **optional** and inferable (path maps, markers, tags, config, naming, CI); the report is useful without heavy per-test metadata |

Locked: framework-embedded engine in the existing Python family; on-disk cumulative store; no server/DB/accounts; the layered multi-lens model; light/dark; independent visual identity.
