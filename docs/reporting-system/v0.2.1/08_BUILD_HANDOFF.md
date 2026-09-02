# 08 — Build Handoff
### Strata — Automation Run Reporting System

> For the implementation team. The target is the **existing Python product family** (`09`), not a greenfield monorepo. This file gives the ecosystem-aligned architecture, **binding constraints**, rollout order, and validation. Deep repository internals (module layout, concurrency mechanism, CI YAML) live in the implementation team's own technical design document, which must satisfy these constraints and the contracts in `02`, `09`, `10`, `12`.

---

## 1. Binding constraints

1. **No server, no database, no accounts.** Zero-ops = no hosted backend/DB/accounts. A local, loopback-only `serve` command is a convenience, not a hosted service (DEC-08).
2. **Reporting engine lives in `automation-core`** (`09`): schema/models, store writer, viewer assets, CLI, metrics, history, comparison, gates, redaction engine, importers, shared tests. Frameworks own only capture + mapping; the orchestrator only coordinates/aggregates.
3. **The neutral schema (`02`) is the contract.** Core writes it; the viewer reads it; frameworks map into it via the ingestion API. No divergence.
4. **Two decoupled layers:** the **viewer** is runtime-agnostic static web assets (vendored in core); the **engine/CLI** is Python in core. The viewer must not depend on Python at read time.
5. **Manual retention only.** Never auto-delete a run.
6. **Security is P0** (`10`): default redaction, loopback-only serve, artifact safety, no telemetry.
7. **Performance:** responsive at ≥ 5k results/run and many stored runs — light indexes, sharding, virtualization, lazy media (`05 G-04`).
8. **Accessibility + theming + responsive** per `06` (WCAG 2.2 AA, light/dark/auto, 320/390/tablet/desktop).
9. **Definition of done = `05_AC_MATRIX.md`.**

## 2. Recommended technical approach (advisory)

- **Engine (core, Python):** a `reporting` package inside automation-core exposing the ingestion API (`02 §9`), the store writer, metrics/history/comparison, the gate evaluator (safe rule schema, `11`), the redaction engine (`10`), importers, and the `strata` CLI (`serve`/`export`/`delete`/`migrate`/`rebuild-index`). No web framework, no DB.
- **Viewer (static web app):** a modern TypeScript SPA (framework choice is the team's — React or Svelte are both reasonable) built by a bundler to **static assets** that are vendored into core and served by `strata serve`. The UI is component- and state-heavy (trees, matrices, drawers, charts, URL-state) and benefits from a mature front-end toolchain; the built output is plain static files.
- **Charts:** a lightweight, themeable library or hand-rolled SVG — full control over status-colour roles, table-toggle, click-through, light/dark. Avoid heavy opinionated kits.
- **Fonts:** **bundle fonts locally** with system fallbacks. Reports are local/offline; **do not depend on remote web fonts** (`06`).
- **Serve:** a tiny static file server bound to `127.0.0.1` (loopback) only (`10`), serving `STORE_ROOT` for lazy per-result/media loading and live polling. `export` produces the portable, serve-free artifacts (`04 F13`).
- **CLI parity:** a framework run auto-generates and can auto-open a useful latest report (parity with current UX, DEC-08).

The team may choose differently; record the choice and show it still meets §1.

## 3. Ownership map (build tasks by repo — see `09`)

- **automation-core:** everything in §2 "Engine" + the vendored viewer build + shared report tests + fixtures (`13`) + migration/rollback (`12`) + redaction engine (`10`) + metrics/gates (`11`).
- **web / api / mobile frameworks:** each implements its **evidence mapper** into the core ingestion API, with platform redaction defaults (`10`). No engine logic.
- **orchestrator:** implements combined-run aggregation + the Journey lens data (`09 §4`). No re-capture.

## 4. Viewer read strategy (serve vs file-open)

- **Auto-open latest:** after a run, core generates the report and can open the latest run view directly (parity). Where that view needs only the latest run's inlined data, it opens without a server; where it needs the full store/live/large history, it uses `serve`.
- **`serve`:** loopback static server over `STORE_ROOT` → lazy result/media loading, live polling, full-store compare/trends.
- **`export`:** self-contained artifacts that open with no server (`04 F13`, `10` safe-share).
- Direct `file://` on a store view that needs lazy fetch shows guidance to run `serve` (`05 DATA-09`); exported single-file HTML always opens directly.
- Preserve compatibility for current `reports/automation-report` consumers (`12`).

## 5. Performance budget

- Build trees/lists/facets from the light `results/index.json` only; never parse full results for the tree.
- Virtualize any list/tree/matrix > 200 rows; lazy-load full result JSON + media on selection (LRU cache).
- Shard `results/` (one file per result) and heavy step trees.
- Target: interactive < 1.5s on a 5k-result run over `serve`; 5-run × 5k matrix scrolls at 60fps.

## 6. Testing & validation (maps to `05`, detailed in `13`)

- Schema conformance against the JSON Schema in core.
- Golden fixtures (`13`): summaries, gate verdicts, flaky flags, and **comparison completeness** (`05 F5-10`).
- Status-logic units: raw vs effective status, assertion→failed, error→broken, retry→flaky, blocked/known (`05 DATA-*`, `02 §7`).
- Identity: stable-id match across runs; name-based degradation flagged.
- Security: redaction unit tests, loopback binding, path-traversal, size/MIME (`10`, `05`).
- Viewer: URL-state round-trip; theme correctness both modes; a11y automated + keyboard; **visual QA at 320/390/tablet/desktop, light/dark**; export round-trip; serve vs file-open modes; performance at 5k/5-runs.
- **Browser matrix:** Chromium, Firefox, WebKit/Safari, Edge where available.

## 7. Rollout order (summary; full plan in `12`)

Core first → release-tag core → update framework pins to new core → update orchestrator → post-merge CI + local report validation. Non-destructive: keep the current reporter available behind a flag/alias until the new report is validated across repos.

## 8. Phasing (mapped to `05` release gates)

- **Phase 0 — Foundation:** core schema + ingestion API + store writer (atomic/concurrent-safe, `02`/`10`) + redaction engine + one framework mapper (web) + `serve` + auto-open. Exit: `05` DATA-* + F1/F2/F3 P0 + SEC P0.
- **Phase 1 — Core report:** Overview, Explorer (all lenses), Test Detail (all present evidence types), Triage; add API + mobile mappers; combined-run aggregation basics.
- **Phase 2 — Comparison + analytics:** Comparison (2–5) with lineage/comparability, Trends, Flaky, Quality Gates (safe rule schema), metrics/defaults (`11`). Exit: Alpha.
- **Phase 3 — Polish + release:** Search/saved-filters, Runs mgmt, Settings, Live, full **Exports** (`04 F13`), responsive/a11y audit, migration (`12`), fixtures/validation (`13`), user docs (§10). Exit: Beta → 1.0.

## 9. Migration & compatibility

Follow `12_MIGRATION_AND_ROLLOUT.md`: current report paths/flags, backward-compatible wrappers/aliases, whether the old `reports/automation-report` output is preserved/redirected/migrated, importer behavior, per-repo PR order + validation commands, and fallback if the new viewer fails to generate.

## 10. Required user-facing documentation (deliverable)

Ship alongside the product: quick-start README, CLI reference, per-framework integration guide, config reference, **redaction guide**, export/share guide, and troubleshooting. Written in plain product language.

## 11. What the implementation must not do

- No backend/DB/accounts, no cloud calls (except the user's own AI provider if F14 is ever enabled), no telemetry/phone-home.
- Don't put web/API/mobile capture logic in core, and don't duplicate framework logic in the orchestrator (`09`).
- Don't change the schema, taxonomy, or IA to simplify implementation — raise it instead.
- Don't auto-delete or mutate execution data.
- Don't ship single-theme, inaccessible, non-responsive, or unredacted-by-default UI.
- Don't invent screens/data beyond `04`/`07`.
