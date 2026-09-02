# 09 — Existing Ecosystem Integration
### Strata — Automation Run Reporting System

> The implementation target is an **existing Python multi-repo product family**, not a new standalone monorepo. This file defines where reporting responsibilities live, the boundary each repo maps across, and how cross-platform/combined reporting works. It defines product/integration **requirements and boundaries**; repository internals belong in the implementation team's technical design document.

---

## 1. Repositories & ownership

| Repo | Language | Reporting responsibility |
|------|----------|--------------------------|
| **automation-core** | Python | The reporting **engine**: neutral schema/models, store writer, **viewer assets**, CLI (`serve`/`export`/`delete`/`migrate`), metrics, history, comparison, quality gates, redaction engine, neutral importers (JUnit-XML / Allure-results), and the shared report test suite. Core defines the **ingestion API** everything else maps into. |
| **web-automation-framework** | Python | Browser automation and **web evidence capture + mapping**: screenshots, video, trace, DOM snapshots, network (HAR), console, visual diff → mapped to the neutral schema via the core ingestion API. Owns web-specific redaction defaults for DOM/network. |
| **api-automation-framework** | Python | **API evidence capture + mapping**: request/response (method, url, headers, body, latency), contract/schema checks, payload assertions → neutral schema. Owns API redaction defaults (headers, bodies, query params). |
| **mobile-automation-framework** | Python | **Mobile evidence capture + mapping**: Appium session, device metadata, screen recording, screenshots, device logs, native/webview/hybrid context switches, mobile-web → neutral schema. Owns mobile redaction defaults (device logs). |
| **full-stack-automation-orchestrator** | Python | **Cross-platform orchestration + combined-journey reporting.** Coordinates multi-framework run plans and aggregates their results into one run/report. **Must not** re-implement web/API/mobile capture or reporting logic — it composes what the frameworks and core already produce. |

**Principle:** capture lives where the platform expertise lives (the framework repos); the engine, storage, and presentation live once in core. No framework re-implements engine logic; the orchestrator re-implements nothing.

## 2. The ingestion boundary (the seam)

There is exactly one seam between "capturing what happened" and "recording/rendering it": the **core ingestion API** (`02 §9`).

```
web-framework  ─┐
api-framework  ─┤   map native events →   ┌──────────────────────────┐
mobile-framework┤  core ingestion API →   │ automation-core          │
orchestrator   ─┘  (startRun/startTest/   │  • neutral schema/models │
                    step/attach/endTest/   │  • store writer (.strata)│
                    endRun)                │  • metrics/history/compare│
                                           │  • viewer assets + CLI    │
                                           │  • redaction engine       │
                                           └──────────────────────────┘
                                                       │
                                            serve / export / static report
```

- Frameworks depend on **core** (for the ingestion API + models); core does **not** depend on any framework. This keeps core neutral and reusable.
- A framework's adapter is a **thin mapper**: translate that platform's native run/test/step/evidence events into ingestion calls and attach platform-specific evidence blocks (`02 §5.2`). No engine logic in adapters.
- Redaction is applied **at capture/ingestion time** by the framework's mapper using core's redaction engine and the framework's platform defaults (`10`), so secrets never reach the store unredacted.

## 3. Schema fields for a multi-repo world (adds to `02`)

The neutral result/run schema carries **source/origin metadata** (data lineage) so multi-repo and orchestrated runs are unambiguous (to be reflected in `02`):

- `run.projectId` — logical project/product under test.
- `run.reportType` — `web | api | mobile | combined` (combined = orchestrated cross-platform run).
- `run.frameworks[]` — the frameworks that contributed (name + version + repo + adapter version).
- `result.platform` — `web | api | mobile` (+ reserved `desktop`).
- `result.source` — `{ framework, repo, adapter }` for each result, so a combined report can attribute every test to its origin and filter/lens by framework or repo.
- `run.lineage` — identifiers that make runs **comparable** (project, suite-set/test-plan id, framework/profile/environment) — used by comparison and "what changed" (`11 §baseline & lineage`).

## 4. Combined-journey (orchestrated) reporting

When the orchestrator runs a cross-platform journey (e.g. API sets up data → web performs the flow → mobile verifies):

- The orchestrator produces **one run** (`reportType: "combined"`) that aggregates results from each framework, preserving each result's `source` (framework/repo).
- The report exposes a **Journey lens** in addition to the standard lenses (`03 §1.1`): steps/tests grouped by journey and stage, showing the hand-off across platforms in sequence, with each stage's platform-specific evidence.
- Per-platform sub-summaries appear alongside the combined summary (e.g. web pass-rate, API pass-rate) so a combined run never hides a platform's health.
- The orchestrator aggregates; it does not re-capture. If a stage's evidence is missing, the report shows an honest unavailable state (owned by the responsible framework).

## 5. Desktop (deferred — DEC-06)

No desktop automation framework exists in the family. Therefore:
- The schema **reserves** the desktop evidence shape (`02 §5.2`) as future-ready.
- **No desktop adapter** is built and **no desktop P0 acceptance** exists in v1.
- If a desktop framework is added later, it plugs into the same ingestion boundary with a desktop mapper — no engine/core change required.
- *(Confirmed deferred by DEC-06; revises the earlier "all platforms first-class" intent.)*

## 6. Viewer & store placement

- The **viewer** is a static web app (built assets) **vendored into automation-core** and served by core's CLI. It reads the store; it is UI technology, decoupled from Python, and does not require a framework to run (`08 §4`).
- The **store** (`STORE_ROOT`, `02 §1`) is created and updated by core during/after a run and persists cumulatively until manually deleted. Final on-disk location and any framework-run defaults are confirmed in `12` (migration) so they align with current report paths.

## 7. Dependency & versioning direction (summary; full plan in `12`)

- **Core is released first**; frameworks and the orchestrator update their pin to the new core; the orchestrator updates last.
- Frameworks never fork or vendor the engine; they consume core's ingestion API and models.
- Backward compatibility: core reads runs produced by older core versions (`02 §13`; migration in `12`).

## 8. Boundary with the technical design document

This file specifies **integration requirements and ownership**. It intentionally does **not** specify: exact Python package/module layout inside each repo, class design, the locking/queue implementation, or CI pipeline YAML. Those belong in the implementation team's technical design document, which must satisfy the requirements here and in `02`, `08`, `10`, `12`.
