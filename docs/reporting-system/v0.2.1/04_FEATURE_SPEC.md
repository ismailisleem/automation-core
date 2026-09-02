# 04 — Feature Specification
### Strata — Automation Run Reporting System

> Every screen, chart, and function, specified to the level the implementation team builds without inventing. Each feature references the schema (`02`) and IA (`03`). Feature IDs (`F#`) anchor the acceptance criteria in `05`. Every **formula/default** (health, gates, flaky grade, duration regression, baseline/lineage, risk) is defined in `11`; **security/redaction** behavior is defined in `10`. All labels are **optional and inferred** (`11 §9`); features degrade to honest empty states when a label/metric/evidence is absent. Nothing here is optional unless marked *(optional)* or *(deferred)*.

**Feature index:** F1 Overview · F2 Explorer · F3 Test Detail & Evidence · F4 Triage · F5 Comparison · F6 Trends & History · F7 Flaky · F8 Quality Gates · F9 Search & Saved Filters · F10 Runs Management · F11 Settings · F12 Live Run · F13 Export & Data-out · F14 AI Assist *(deferred v2)*. Value-add features V1–V10 are embedded in the relevant feature and cross-listed in §V.

---

## F1 — Run Overview (dashboard)

**Purpose:** the fastest true read of one run's health; the stakeholder's home; the entry to everything else.
**Primary users:** Stakeholder (default landing), everyone as a starting point.

**Anatomy (top → bottom):**
1. **Run header:** label, seq (`#42`), trigger, env fingerprint chip (envName · appVersion · branch · commit), start/finish, wall-clock vs summed duration (shows parallel speedup), live badge if running.
2. **Verdict band:** the Quality-Gate verdict (`Pass / Warn / Fail / Not-evaluated`) as the largest element, with the failing/warning rule(s) named inline (F8). This is the "one number reported upward."
3. **KPI row (stat cards):** Total · Pass-rate % (with Δ vs previous run) · Failed · Broken · Flaky · Skipped · Known · Blocked · Duration (wall-clock, with Δ). Each card is a filter shortcut into Explorer.
4. **Status donut** + **platform bar** (stacked by status per platform).
5. **"What changed" panel (V1):** auto-generated diff vs the previous run (by seq): new failures (regressions), newly fixed, added tests, removed tests, flaky delta, pass-rate delta. Each line links into Comparison (F5) pre-filtered. Plain, specific text — no prose padding.
6. **Top risks:** the N most impactful non-passes ranked by severity × recency × ownership (severity-weighted, V4), each a chip → Test Detail.
7. **Defect mix donut:** counts by defect type (product/automation/system/env/to-investigate/known).
8. **Trend spark row:** mini pass-rate + flaky sparklines over the last runs (→ Trends F6).
9. **Slowest tests strip (V6):** top 5 by duration with Δ vs their own history → duration board.

**Charts/stats:** status donut, platform stacked bar, defect-mix donut, pass-rate & flaky sparklines, duration distribution mini-histogram. All defined in §CH.
**Interactions:** every KPI/segment/chip is a deep link into a pre-filtered Explorer/Comparison view (`03 §6`). Header env chip → environment fingerprint popover (V8).
**States:** live (partial data, progress bar executed/total, auto-refresh); no-previous-run (hide Δ + what-changed, show "first run in store"); gate not configured (verdict = Not-evaluated + link to Settings).

---

## F2 — Run Explorer (the workhorse)

**Purpose:** navigate one run through any lens, filter/search to a precise slice, open any test.
**Primary users:** QA, Developer, Automation.

**Layout — three panes:**
- **Left — Lens + Facets:** lens selector (`03 §1.1`); facet panel (`03 §4.2`) with live counts; saved-filter dropdown; role-preset chips.
- **Middle — Tree/List:** toggle between **Tree** (grouped by active lens, each node showing rollup stats + status bar) and **List** (flat, sortable table). Virtualized for large runs. Columns (List): name, status, flaky, platform, suite/feature, owner, severity, defect type, duration (+Δ vs history), attempts, evidence icons.
- **Right — Detail:** the selected Test Detail (F3) as a side panel; the tree/list stays in context (no full-page navigation).

**Behaviors:**
- Switching lens re-groups instantly (client-side from labels).
- Tree nodes expand/collapse; node header is itself filterable ("show only failing under Checkout").
- Sort by: status severity, duration, duration-Δ, name, owner, attempts, recency-of-first-failure.
- Multi-select rows → **bulk actions** (classify defect, mute, export, copy links) (`03 §4.3`).
- Column and lens choices persist per store; everything is URL-encoded.
- Keyboard: `/` focus search, `j/k` move selection, `enter` open, `f` toggle failed-only, `[`/`]` prev/next failure.

**States:** empty-after-filter (name the excluding facet + clear); loading skeleton; live (rows stream in).

---

## F3 — Test Detail & Evidence viewer

**Purpose:** everything needed to understand one test and decide bug-vs-not.
**Primary users:** QA, Developer, Automation.

**Anatomy:**
1. **Header:** name, full path, status chip, flaky badge, platform, severity, owner, tags, duration, `#attempt selector` if retried, `caseId`/issue links.
2. **Verdict & failure block:** for non-pass — failure message, error type, **failed↔broken indicator**, stack trace (collapsible, copyable), **diff** (expected vs actual, text/JSON/visual).
3. **Defect triage bar (F4):** current defect type + classify control + comment + link-issue, inline.
4. **Attempts strip:** each attempt as a tab with its own status/duration/evidence; flaky tests show failed-attempt(s) + passing attempt side by side.
5. **Steps tree:** nested steps (`02 §5.1`) with per-step status, duration, params, and **per-step evidence** (screenshot/snapshot/network/console). Clicking a step scrolls its evidence into view. Failed step is auto-expanded and highlighted.
6. **Evidence tabs — polymorphic by platform (`02 §5.2`):**
   - **All platforms:** Screenshots (gallery, failure-shot first, before/after where present), Video (with a scrubber that can jump to a step's timestamp), Logs, Attachments.
   - **Web:** **Trace** (embed the Playwright-style trace viewer or link to open it), DOM snapshots per step, **Network** (request table: method/url/status/latency; row → headers+body), Console, **Visual diff** (baseline/actual/diff slider + pixel/layout delta).
   - **API:** **Requests** panel — method/url/status/latency, request headers+body, response headers+body, and payload assertions (path/expected/actual/pass).
   - **Mobile:** device chip (name/os/udid), app build, **screen recording**, screenshots, **device logs** (logcat/syslog), native/webview context switches timeline.
   - **Desktop:** *(reserved/future — not produced in v1, DEC-06)*.
7. **History strip:** last N outcomes for this `testId` as a sparkline/dots (→ opens history/flaky view for the test).

**Redaction & unavailable data:** headers, bodies, logs, and captured text are shown **already redacted** (`10`); the viewer never reveals raw secrets. Any evidence, metric, or label that isn't present (no trace, no video, no owner, no visual-diff, no issue status) renders as an **honest disabled/empty state with a reason** — never a broken image, blank, or fabricated value.

**Interactions:** every evidence item deep-linkable (`…&test=&attempt=&step=&evidence=`); copy-link on any panel; download any artifact. Video ⇄ step sync both ways.
**States:** evidence absent → tab disabled with reason ("trace captured on failure only"); large log → lazy chunked; passing test → collapse failure/diff, keep steps + evidence.

---

## F4 — Defect triage workflow

**Purpose:** classify every non-pass as bug/not-bug with evidence, so results are trustworthy.
**Primary users:** QA, Developer.

**Mechanics:**
- Any `failed`/`broken` (optionally `known`) result carries a **defect type** (`03 §3`), defaulting to **To Investigate**.
- Classify inline (Test Detail F3) or in bulk (Explorer F2) or from the **Triage board** (below).
- On classify: set type, optional category + comment, optional **link external issue** (tracker/key/url/status stored, link-out only in v1), record `source` + `assignedBy`. Written to disk via serve (`02 §8`).
- **Triage board:** a dedicated section (`03 §5.6`) = the To-Investigate queue + columns per defect type (kanban-style, read/classify only, no execution). Filter by platform/suite/owner/severity. Bulk move.
- **Error-signature clustering (V2):** group non-passes by **normalized error signature** (message + type with volatile bits stripped) so one root cause covering 30 tests is triaged once and applied to the cluster. Cluster shows member count, affected suites/platforms, representative evidence.

**States:** nothing to triage → "all classified" empty state; classification conflicts across attempts resolved to the result-level type.

---

## F5 — Multi-run Comparison (2–5 runs) — headline feature

**Purpose:** answer "what changed across runs" completely — every case, not just failures.
**Primary users:** all; QA/Automation deepest.

**Selection:** pick 2 to 5 stored runs from the run switcher/Runs list (any runs, not only adjacent). One run is the **baseline** (leftmost by default; user can set). Order is user-arrangeable.

**Match model:** tests are matched across runs by `testId`/`historyId` (`02 §6`). This yields, for the union of all tests across the selected runs, one **row per logical test** with a status **cell per run**.

**Primary view — Status Matrix:**
```
Test (logical)         | Run#40 | Run#41 | Run#42 |  Trend  | Change
Guest checkout         |  pass  |  fail  |  fail  |  ●●●    | New failure→persisting
Apply coupon           |  fail  |  fail  |  pass  |  ●●●    | Fixed
Search suggest (added) |   —    |   —    |  pass  |         | Added in #42
Legacy import (removed)|  pass  |  pass  |   —    |         | Removed after #41
```
- Cell shows status (colour) + flaky overlay + duration; hover → mini evidence/why; click → that run's Test Detail.
- Row grouped by the active **lens** (suites/features/platform…), with node rollups of change categories.

**Change categories (complete set — must all be represented):**
| Category | Definition |
|----------|-----------|
| **Unchanged-pass** | passed in all compared runs |
| **Unchanged-fail** | non-pass with same status across runs (persistent) |
| **New failure (regression)** | pass in baseline → non-pass later |
| **Fixed** | non-pass in baseline → pass later |
| **Status-changed** | any `effectiveStatus` transition not covered above (e.g. failed→broken, broken→passed) |
| **Flaky-changed** | flaky flag or flaky-rate changed for the test |
| **Added** | present in a later run, absent in baseline |
| **Removed** | present in baseline, absent later |
| **Duration-regressed / improved (V6)** | duration Δ beyond threshold |
| **Defect-type-changed** | triage classification differs across runs |

Each category is a **facet** (`03 §4.2 change`) and a chip with counts; selecting filters the matrix. For 3–5 runs, "baseline" is the reference and transitions are computed baseline→each; a per-pair toggle is available.

**Summary strip:** pass-rate per run with deltas; counts of new-failures / fixed / added / removed / still-failing; environment **drift** between runs (V8: appVersion, browser, device, env differences highlighted — a common false-signal source).
**"Similar tests" handling (explicit requirement):** tests that match across runs are the matrix rows; near-matches (same name, differing params) are grouped under the test family via `historyId`, with a sub-row per param set so users see same-vs-different explicitly.
**Side-by-side test compare:** selecting one matched test opens **N attempts/evidence side by side** (e.g. the failing run's trace next to the passing run's) to see exactly what differs.
**Export:** the comparison (matrix + categories) exports to CSV/JSON and to a shareable deep link.

**States:** runs with `identity:"name-based"` → warn that added/removed may be noisy; incompatible env → drift banner; >5 selected → blocked with message.

---

## F6 — Trends & History (across all stored runs)

**Purpose:** long-view quality signal beyond 5-run compare.
**Primary users:** Stakeholder, Automation, lead.

**Charts (over all runs in store, x = run seq/time):** pass-rate line; status stacked-area (passed/failed/broken/skipped/blocked, with a flaky **overlay** band); flaky-rate line; total-duration & p95 line; defect-mix stacked-area; test-count line (growth/removal). Range selector (last N / date range / all). Brush to zoom; click a point → that run's Overview.
**Tables:** **Most-failed tests (top 50)** by failure count across history; **Flakiest tests (top 50)** by history instability; **Consistently slow** tests. Each row → test history.
**Per-test history:** for a chosen `testId`, its status timeline, flaky rate, duration trend across every stored run.
**States:** <2 runs → "need more runs for trends" with what exists.

---

## F7 — Flaky analysis

**Purpose:** surface and quantify instability so it can be fixed or muted honestly.
**Primary users:** Automation, QA.

**Signals combined:** (a) **retry-recovered** within a run (`02 §7`); (b) **history-unstable** across runs (alternating pass/fail for same `testId`).
**Stability grade (A–F):** computed per test from history (pass consistency + retry rate). Displayed as a badge; sortable.
**Views:** flaky list (grade, flaky-rate, last-N dots, retry count, suites/platforms affected); flaky trend (rate over time, F6-linked); per-test attempts diff (what differed between the failing and passing attempt — steps/evidence). Bulk **mute** → sets `known`/`muted` so gates aren't broken by acknowledged flakes (managed in F11).
**States:** no retries configured → note that only history-based flakiness is available.

---

## F8 — Quality Gates (Go/No-Go)

**Purpose:** turn metrics into a release verdict.
**Primary users:** Stakeholder, lead.

**Model:** rules in `config.json` as a **safe structured schema** — each rule is `{ metric, operator, threshold, severity, scope }` (see `11 §8`), **never a free-form expression and no `eval`**. `metric` comes from a fixed allowlist (`passRate`, `counts.broken`, `counts.blocked`, `flakyRate`, `healthScore`, `new.productBug`, …); `operator` ∈ `>= <= > < == !=`; `scope` ∈ `run | vs-baseline`; `severity` ∈ `warn | fail`. Evaluated by core at `endRun`, stored in `run.json.gate` (`02 §3`). Run verdict = worst rule result; known/muted excluded from fail by default (configurable per rule). Default gate set in `11 §8`.
**UI:** verdict band on Overview (F1); rules list with actual vs threshold and pass/warn/fail; editable in Settings (F11) with a live preview against the current run.
**States:** no gates → Not-evaluated; baseline missing for a comparison rule → that rule = not-evaluated with reason.

---

## F9 — Global search & saved filters

**Purpose:** reach any information fast (stakeholder's "quick access", engineer's precision).
**Primary users:** all.

**Search:** one box (`03 §4.1`), scoped to active run or comparison set, matching name/error/tag/owner/path/label/defect/issue-key; grouped by lens node; keyboard-first; each hit deep-links. Recent + suggested queries.
**Saved filters:** persist a facet+search+lens set (named) in `config.json`; role presets ship by default (`03 §7`); shareable via `filterId` in URL.
**Command palette *(optional)*:** `⌘K` to jump to a run, a test, a saved filter, a section, or toggle theme.

---

## F10 — Runs management

**Purpose:** own the store; manual retention.
**Primary users:** all.

**List:** every stored run (from `index.json`) with seq, label, trigger, env, counts, pass-rate, gate, tags, pinned, size-on-disk. Sort/filter/search.
**Actions:** rename label; add tags; **pin** (protect + surface); **delete** run(s) — the only retention mechanism, with confirm + "frees X MB"; **export** (F13); **compare** (select ≤5 → F5). Bulk delete/tag/export.
**Store health:** total size, run count, largest runs, oldest runs; a hint to delete when large (never auto-delete).
**States:** empty store → onboarding (how the framework writes runs, how to serve/export).

---

## F11 — Settings

**Purpose:** configure without touching files.
**Scope (writes `config.json`):** Quality gates (**safe rule schema** editor + live preview, F8/`11 §8`); custom lenses (`03 §1.1`); theme default (light/dark/auto); default role preset & lens; saved filters; mute/known-issue registry (patterns → muted); metadata-inference mappings (path→labels, marker/tag rules, `11 §9`); duration-regression thresholds (`11 §3`); **redaction & capture config** (profiles, allow/deny lists, capture modes, PII toggle — `10 §3`); export/safe-share defaults. *(No AI settings in v1; AI assist is v2 roadmap — DEC-07.)*
**States:** all changes preview before apply where they affect verdicts.

---

## F12 — Live run (during execution)

**Purpose:** watch a run as it happens (report may update during the run, finalized at end).
**Mechanics:** when served, the viewer polls `run.json` + `results/index.json`; results stream into Overview/Explorer as `endTest` writes them. Header shows live badge + progress (executed/total, elapsed, ETA from mean). Statuses appear as they finalize; `running`/`not-run` transient states shown. On `endRun`, summary/gate/rollups finalize and the live badge clears. Partial data is clearly marked; comparison/trends operate on completed runs only.
**States:** aborted run → `aborted` with partial data preserved; poll failure → "reconnecting" without losing view.

---

## F13 — Export & data-out

**Purpose:** portability, stakeholder sharing, and pipeline use.
**Export modes** (each states editable/read-only, media, and whether it needs `serve`):

| Export | Format | Editable | Media | Needs serve | Notes |
|--------|--------|----------|-------|-------------|-------|
| Single-run portable report | HTML (self-contained) | read-only | embedded | no | opens anywhere (`02 §12`) |
| Executive summary | HTML / **PDF** | read-only | key charts only | no | glanceable for management |
| Executive summary (doc) | **DOCX** *(optional / phased)* | editable | none/inline | no | retained only if wanted |
| Filtered test list | **CSV / XLSX** | editable | none | no | current filter set |
| Comparison matrix | **CSV / XLSX / JSON** | editable | none | no | + change categories |
| Raw data | JSON (zip) | n/a | refs | no | pipeline/archival |
| Evidence bundle | ZIP | read-only | included | no | honours redaction |
| Share card | **PNG / SVG** | n/a | n/a | no | verdict + KPIs snapshot |
| Safe-share bundle | HTML/ZIP | read-only | redacted/optional | no | `strict` redaction, warns first (`10 §6`) |

**Rules:** every export applies redaction per `10`; **safe-share** is the default posture for anything leaving the machine, with an explicit warning before media-heavy/sensitive bundles. Exports carry env fingerprint + generation timestamp + redaction profile. **Deep-link copy** anywhere. PDF/DOCX/share-card generation is phased (Phase 3, `08 §8`); HTML/CSV/XLSX/JSON/evidence-zip/safe-share are core v1.

---

## F14 — AI root-cause & triage assist *(deferred to v2/backlog — DEC-07; off by default, bring-your-own-key)*

> Not built in v1. Specified here so v1 leaves room for it. If ever shipped: off by default, the user's own provider key, no data leaves the machine unless explicitly enabled, output is a suggestion only.

**Purpose:** speed triage; never decide it.
**Mechanics (opt-in, key stored locally, consistent with the ecosystem's BYO-key pattern):** for a failure, assemble error + stack + failing step + relevant evidence text (not media) and ask the configured provider for (a) a likely **defect-type suggestion** with reasoning, (b) a probable root-cause summary, (c) a suggested fix area. Output is a **suggestion** the user confirms/edits (`defect.source:"ml"`); never auto-applied. Error-signature clusters (V2) can be explained once per cluster. **Guardrails:** off by default; no data leaves the machine unless enabled; the provider is the user's own account; clearly labeled as AI-suggested.
**States:** no key → module hidden/disabled with a one-line how-to.

---

## §V — Value-add features and where they live

| ID | Feature | Home | Why it strengthens the report |
|----|---------|------|-------------------------------|
| V1 | **What-changed auto-summary** (vs previous run) | F1 Overview → F5 | Instant regression read without opening compare |
| V2 | **Error-signature clustering** | F4 Triage | Triage one root cause for many tests; cut noise |
| V3 | **Ownership rollups** | F2/F6 | Route failures to the right dev; per-owner health |
| V4 | **Severity-weighted health score** | F1 | One honest number that weights blockers over trivia |
| V5 | **Coverage-of-layers view** | F2/F6 | Which suites/features/platforms actually ran (gap detection) |
| V6 | **Slowest & duration-regression board** | F1/F5/F6 | Catch performance regressions in the suite itself |
| V7 | **Execution timeline / Gantt** | F1 (expand) | Visualize parallel execution, workers, long poles |
| V8 | **Environment fingerprint & drift** | F1/F5 | Distinguish real regressions from env changes |
| V9 | **Run annotations / notes** | F1/F10 | Capture human context on a run (mutable, `02 §8`) |
| V10 | **Muted / known-issue registry** | F7/F11 | Acknowledge flakes honestly without breaking gates |

---

## §CH — Charts & statistics catalog (single source for all visualizations)

| Chart | Type | Data source | Used in |
|-------|------|-------------|---------|
| Status breakdown | Donut | `run.summary.counts` | F1 |
| Platform × status | Stacked bar | `run.summary.byPlatform` | F1 |
| Defect mix | Donut | `run.summary.defectCounts` | F1, F4 |
| Pass-rate trend | Line | `index.runs[].passRate` | F1(spark), F6 |
| Flaky-rate trend | Line | `index.runs[].flakyRate` | F1(spark), F6, F7 |
| Status over time | Stacked area | `index.runs[].counts` | F6 |
| Duration distribution | Histogram | result `durationMs` | F1, F6 |
| Duration trend (p50/p95) | Line | per-run duration stats | F6, V6 |
| Test-count trend | Line | `index.runs[].counts.total` | F6, V5 |
| Execution timeline | Gantt | attempt start/duration + worker | V7 |
| Stability grade dist. | Bar | computed A–F | F7 |
| Comparison change mix | Stacked bar | F5 categories | F5 |
| Per-test history | Dot/sparkline | `result.history` | F3, F6, F7 |
| Coverage-of-layers | Treemap/heatmap | label rollups | V5 |

**Global chart rules (styling in `06`):** every chart honours status colour roles; every chart has an accessible table equivalent (toggle) and tooltips with exact counts + %; charts are click-through to a filtered view; charts render correctly in light and dark.
