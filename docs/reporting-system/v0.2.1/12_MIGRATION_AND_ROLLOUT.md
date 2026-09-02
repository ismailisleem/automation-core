# 12 — Migration & Rollout
### Strata — Automation Run Reporting System

> How the new reporting system replaces the current reporter across the product family, non-destructively. Items marked **⟨confirm⟩** need the current reporter's concrete details to finalize; the plan's structure is complete and only these values are pending.

---

## 1. Current-state inventory — implementation-discovery items

These are **implementation-discovery items**: values the implementation team must capture from the current reporter early in Phase 0, before cutover. They are the only open inputs in this package; the plan around them is complete.

- **Current report output path(s)** — e.g. `reports/automation-report/…` — discover exact paths.
- **Current CLI command(s) and flags** that generate/open the report — discover exact commands + flags.
- **Current output formats** (HTML/JSON/JUnit/other) and their consumers (CI, dashboards, humans) — inventory each.
- **CI/doc parity references** — every CI step, artifact, link, or doc that points at the current report — inventory for redirect/parity.

Each discovered value feeds the compatibility strategy (§2) and the parity checklist (`08 §9`).

## 2. Compatibility strategy

- **Backward-compatible wrappers/aliases:** keep the current command names working; internally route to the new engine, or keep both behind a flag during transition (`--reporter=legacy|strata`, default flips after validation).
- **Old output path:** decide per consumer — **preserve** (write both for a deprecation window), **redirect** (old path becomes a pointer/redirect to the new report), or **migrate** (one-time import of historical runs into the store via the importer). Recommended: **preserve both** for one release, then redirect.
- **Importer:** the neutral JUnit-XML / Allure-results importer (core) can backfill historical results into the store so trends/history aren't empty on day one.
- **Auto-open parity:** a framework run still generates and can auto-open a useful latest report (DEC-08), matching current UX so nobody loses the one-command flow.

## 3. Rollout order (dependency-safe)

1. **automation-core** first: land the engine, schema, store, viewer assets, CLI, redaction, metrics, gates, importer, tests, fixtures. Release-tag it.
2. **Framework pins:** update `web-`, `api-`, `mobile-automation-framework` to the new core tag; implement each framework's evidence mapper; validate each produces a valid store.
3. **Orchestrator:** update `full-stack-automation-orchestrator` to the new core; implement combined-run aggregation + Journey lens; validate combined reports.
4. **Flip default** from legacy reporter to Strata once validation passes in every repo; keep legacy behind a flag for one deprecation window.
5. **Remove legacy** after the window, once no consumer references it.

## 4. Per-repo PR order & validation commands

For each repo, a PR must pass before the next repo updates:
- Core PR: schema-validate golden fixtures; unit tests (status, gates, identity, comparison completeness, redaction); viewer build + visual/a11y/perf checks; `serve`/`export`/`file-open` mode tests.
- Framework PR: mapper produces a schema-valid run; evidence present per platform; redaction applied; run opens in the viewer.
- Orchestrator PR: combined run aggregates all frameworks; Journey lens correct; per-platform sub-summaries present.
- **Post-merge CI + local report validation** in each repo: generate a real run and assert the store validates and the report renders.

## 5. Fallback & safety

- If the new viewer fails to generate for a run, the framework **falls back** to writing the raw store (JSON) and surfaces a clear message — a run is never lost because the UI failed.
- Legacy reporter remains available behind a flag until the deprecation window closes.
- Migration/import is **non-destructive**: importing historical runs never deletes or mutates existing outputs.

## 6. Versioning sequence

Core is released first and tagged; frameworks and the orchestrator bump their core pin in the order above; `schemaVersion` is bumped only on breaking changes with a migration note here and migrate-on-read support in core (`02 §10/§13`).

## 7. Deprecation communication

Document the change in each repo's user docs (`08 §10`): what replaced the old reporter, how to run/open the new one, how to opt back temporarily, and when legacy is removed.

## 8. Boundary

This file defines the **rollout requirements and order**. Exact CI YAML, branch/PR mechanics, and repo-internal wiring live in the implementation team's technical design document.
