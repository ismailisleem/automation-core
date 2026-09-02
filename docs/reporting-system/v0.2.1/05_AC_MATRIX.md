# 05 — Acceptance Criteria Matrix
### Strata — Automation Run Reporting System

> The **definition of done**. A feature ships only when every P0 row for it passes; P1 required for release, P2 desirable. Each row is written as an independently testable Given/When/Then. IDs map to features in `04`. The implementation team treats this as the QA gate; the design team satisfies the UI-facing rows via `06`/`07`.

**Priority:** P0 = must (blocks), P1 = required for release, P2 = desirable.

---

## Global / cross-cutting

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| G-01 | any view | it renders | the full state (run, lens, filters, selected test/attempt/step, evidence tab, theme) is encoded in the URL and restores exactly on reload/share | P0 |
| G-02 | light and dark themes | a user switches theme | every element, chart, and status colour re-renders correctly with ≥ WCAG 2.2 AA contrast; no hard-coded colours leak | P0 |
| G-03 | `prefers-reduced-motion` | user has it set | non-essential animation is disabled | P1 |
| G-04 | any large run (≥ 5k results) | Explorer/tree/list renders | it stays responsive via virtualization + lazy detail/media (no full-run JSON load) | P0 |
| G-05 | keyboard only | navigating core flows | Overview→Explorer→Test→evidence is fully operable; visible focus throughout | P1 |
| G-06 | any status/defect label | shown anywhere | it uses the exact taxonomy in `03 §2/§3` — no synonyms, `failed`≠`broken` everywhere | P0 |
| G-07 | any chart | displayed | it has a tooltip with exact count + %, an accessible table toggle, and click-through to a filtered view | P1 |
| G-08 | any deep link received | opened on an empty/other store | it fails gracefully with a clear message, not a blank/broken screen | P1 |

## Data & store (from `02`)

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| DATA-01 | the library finishes a run | `endRun` runs | `index.json` gains one run entry and `runs/<runId>/` contains `run.json`, `results/index.json`, per-result JSON, and copied assets | P0 |
| DATA-02 | a large suite | results written | `results/` is sharded (one file per result) and media are referenced by path, never inlined | P0 |
| DATA-03 | a test with retries where final attempt passes | status computed | `rawStatus:"passed"`, `effectiveStatus:"passed"`, `flaky:true, flakyReason:"retry-recovered"`; flaky is never a terminal status | P0 |
| DATA-04 | an assertion failure vs an unexpected error | status computed | assertion→`rawStatus:"failed"`, error/timeout→`rawStatus:"broken"`; a muted/known rule maps to `effectiveStatus:"known"` | P0 |
| DATA-05 | same logical test in two runs | compared | both share identical `testId`/`historyId` and match as one row | P0 |
| DATA-06 | no derivable stable id | run written | `run.json.identity:"name-based"` set and the viewer warns on added/removed | P1 |
| DATA-07 | user deletes a run | `strata delete` | the run folder + its `index.json` entry are removed; no other run affected; nothing auto-deleted ever | P0 |
| DATA-08 | a triage edit while served | user classifies | only mutable fields (`02 §8`) are written back; execution data is untouched | P0 |
| DATA-09 | `file://` open without server | user opens report | a clear prompt to use `strata serve` (or an exported bundle) is shown rather than silent fetch failures | P1 |
| DATA-10 | parallel workers writing one run | run in progress | writes are atomic (temp+rename) and no shared file is corrupted; shards merge at `endRun` | P0 |
| DATA-11 | a run interrupted before `endRun` | store read | it reads as `partial`/`aborted` with finalized data; a corrupt result file is skipped-with-flag, not a crash | P0 |
| DATA-12 | a damaged/missing index | `strata rebuild-index` | `index.json`/`results/index.json` are reconstructed from run folders | P1 |
| DATA-13 | a run produced by an older core | opened | it is read via migrate-on-read; `strata migrate` upgrades in place | P1 |
| DATA-14 | every store file | validated | it conforms to the core JSON Schema | P0 |
| DATA-15 | multi-repo / combined run | written | `projectId`, `reportType`, `frameworks[]`, and per-result `source{framework,repo,adapter}` are present and correct | P0 |

## F1 — Overview

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F1-01 | a completed run | Overview opens | verdict band, KPI row, status donut, platform bar, defect-mix render from `run.summary`/`run.gate` | P0 |
| F1-02 | a previous run exists | Overview opens | pass-rate and duration show Δ vs previous; "What changed" lists new-fail/fixed/added/removed/flaky-Δ, each linking into F5 pre-filtered | P0 |
| F1-03 | no previous run | Overview opens | Δ and What-changed are hidden with "first run in store" | P1 |
| F1-04 | any KPI card/segment | clicked | opens Explorer/Comparison pre-filtered to that slice | P0 |
| F1-05 | stakeholder preset | Overview opens | a Go/No-Go read is achievable without scrolling past the fold (verdict + pass-rate + top risks above fold) | P1 |

## F2 — Explorer

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F2-01 | a run with labels | lens switched (suite↔behavior↔platform↔…) | the tree re-groups instantly from labels; nodes show rollup stats | P0 |
| F2-02 | facets selected | applied | list/tree filter (AND across facets, OR within); facet counts update live | P0 |
| F2-03 | a filtered set | user acts | bulk classify/mute/export/copy-links operate on exactly the filtered rows | P1 |
| F2-04 | any filter/lens/sort | set | it is URL-encoded and shareable (G-01) | P0 |
| F2-05 | filter excludes all | applied | empty state names the excluding facet with one-click clear | P1 |
| F2-06 | a test appears under multiple tags | tag lens | it correctly appears under each tag node without duplication errors in counts | P1 |

## F3 — Test Detail & Evidence

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F3-01 | a non-pass test | detail opens | failure message, error type, failed/broken indicator, stack, and diff render | P0 |
| F3-02 | a web test with trace/network/console | detail opens | Trace, DOM snapshots, Network table, Console, and Visual-diff tabs work; absent ones are disabled with reason | P0 |
| F3-03 | an API test | detail opens | requests panel shows method/url/status/latency, req+res headers/body, and payload assertions | P0 |
| F3-04 | a mobile test | detail opens | device chip, screen recording, screenshots, device logs, and native/webview contexts render | P0 |
| F3-05 | desktop platform | v1 | **reserved/deferred** — schema shape exists, no desktop adapter or acceptance required (DEC-06) | — |
| F3-06 | any captured headers/bodies/logs | detail opens | values render **already redacted** (`10`); no raw secret is ever shown | P0 |
| F3-07 | absent evidence/metadata (no trace/video/owner/visual-diff/issue status) | detail opens | honest disabled/empty state with reason; never blank, broken image, or fabricated value | P0 |
| F3-08 | a test with video + steps | user clicks a step | video scrubs to that step's timestamp (and vice-versa) | P1 |
| F3-09 | a flaky (retried) test | detail opens | attempts strip shows failing + passing attempts with their own evidence side by side | P0 |
| F3-10 | any evidence panel | opened | it is deep-linkable and each artifact is downloadable | P1 |
| F3-11 | a failed step | detail opens | it is auto-expanded and highlighted in the steps tree | P1 |

## F4 — Triage

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F4-01 | a new non-pass | first seen | its defect type defaults to **To Investigate** | P0 |
| F4-02 | a result | user classifies (type/comment/link) | it persists to disk and reflects immediately in board/filters/counts | P0 |
| F4-03 | many non-passes sharing an error | Triage board | they cluster by normalized error signature; classifying the cluster applies to members | P1 |
| F4-04 | an external issue key entered | saved | tracker/key/url/status stored and link-out works (no two-way sync in v1) | P1 |

## F5 — Comparison (headline)

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F5-01 | 2–5 runs selected | Compare opens | a status matrix renders one row per logical test (union), one status cell per run, matched by `testId` | P0 |
| F5-02 | the compared runs | evaluated | every change category (`04 F5` table) is computed and available as a facet with counts: unchanged-pass/fail, new-failure, fixed, status-changed, flaky-changed, added, removed, duration-Δ, defect-type-changed | P0 |
| F5-03 | a regression exists (pass→fail) | Compare opens | it is listed under **New failure** and reachable in one filter click | P0 |
| F5-04 | a test present in one run only | Compare opens | it appears under **Added** or **Removed** correctly | P0 |
| F5-05 | a matched test | selected | the runs' attempts/evidence show side by side (e.g. failing trace vs passing trace) | P1 |
| F5-06 | parametrized test family | Compare opens | param variants group under the family with per-param sub-rows (same-vs-different explicit) | P1 |
| F5-07 | env differs between runs | Compare opens | environment **drift** is flagged (V8) to avoid false regressions | P1 |
| F5-08 | any comparison | exported | matrix + categories export to CSV/JSON and to a deep link | P1 |
| F5-09 | >5 runs attempted | selecting | blocked with a clear message | P2 |
| F5-10 | the union of tests | totalled | every test is accounted for in exactly the right category (completeness: matched∪added∪removed = union) | P0 |

## F6 — Trends & History

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F6-01 | ≥2 stored runs | Trends opens | pass-rate, status-area, flaky, duration, defect-mix, test-count charts render over history with range control | P0 |
| F6-02 | a trend point | clicked | opens that run's Overview | P1 |
| F6-03 | history present | tables shown | most-failed / flakiest / consistently-slow top-50 render; rows open per-test history | P1 |
| F6-04 | <2 runs | Trends opens | a clear "need more runs" state, not an error | P1 |

## F7 — Flaky

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F7-01 | retry-recovered and/or history-unstable tests | Flaky opens | both are surfaced with flaky-rate and last-N dots | P0 |
| F7-02 | a test's history | evaluated | a stability grade A–F is computed and sortable | P1 |
| F7-03 | a flaky test | selected | the failing-vs-passing attempt diff (steps/evidence) is shown | P1 |
| F7-04 | acknowledged flakes | muted | they become `known`/muted and no longer fail gates by default | P1 |

## F8 — Quality Gates

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F8-01 | gates configured | run finalized | `run.json.gate` holds each rule's actual vs threshold and result; verdict = worst rule | P0 |
| F8-02 | a comparison rule (e.g. new productBug) | evaluated | it computes against the baseline run and reports the baseline used | P1 |
| F8-03 | no gates | run opened | verdict = Not-evaluated with a link to configure | P1 |
| F8-04 | rules edited in Settings | previewed | a live preview against the current run shows the resulting verdict before save | P1 |

## F9 — Search & Saved Filters

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F9-01 | a query | typed | matches across name/error/tag/owner/path/label/defect/issue-key, grouped by lens node, each deep-linking | P0 |
| F9-02 | a filter set | saved | it persists in `config.json`, is reusable, and is shareable via `filterId` | P1 |
| F9-03 | role presets | selected | they apply the documented lens+facets+landing (`03 §7`) | P1 |

## F10 — Runs Management

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F10-01 | stored runs | Runs opens | all runs list with seq/label/env/counts/pass-rate/gate/tags/pinned/size | P0 |
| F10-02 | a run | deleted | confirm shows freed space; deletion removes only that run (DATA-07) | P0 |
| F10-03 | runs | selected | pin/tag/label/export/compare(≤5) work; bulk actions work | P1 |
| F10-04 | empty store | Runs opens | onboarding explains how runs are produced and how to serve/export | P1 |

## F11 — Settings

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F11-01 | settings changed | saved | written to `config.json`; changes affecting verdicts preview first | P0 |
| F11-02 | a label key promoted | saved | it becomes a selectable lens in Explorer/Comparison | P1 |
| F11-03 | redaction/capture config changed | saved | applies to future runs; profiles/allow-deny lists honoured (`10`) | P0 |
| F11-04 | a gate rule edited | saved | stored as structured `{metric,operator,threshold,severity,scope}`; no free-form expression; live preview shown | P0 |

## F12 — Live Run

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F12-01 | a running served run | viewed | live badge + progress (executed/total, elapsed) show; results stream as they finalize | P1 |
| F12-02 | `endRun` fires | finalization | summary/gate/rollups finalize; live badge clears; partial markers removed | P1 |
| F12-03 | run aborted | mid-run | state = aborted with partial data preserved | P1 |

## F13 — Export & Data-out

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F13-01 | a run | `strata export <runId>` | a self-contained read-only single-file HTML opens with data + media, no server needed | P0 |
| F13-02 | a filtered set or comparison | exported | correct CSV/JSON is produced; deep-link copy works | P1 |
| F13-03 | an exported bundle | opened | editing/triage is disabled (read-only) | P1 |

## SEC — Security, privacy & redaction (from `10`)

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| SEC-01 | a payload with tokens/cookies/PAN/PII | captured | it is redacted **before** being written to the store; the raw value never lands on disk | P0 |
| SEC-02 | `strata serve` | started | it binds to `127.0.0.1` only; LAN exposure requires an explicit flag + warning | P0 |
| SEC-03 | an asset path with `..`/absolute/symlink-escape | written/read | it is rejected (no traversal outside `runs/<id>/assets/`) | P0 |
| SEC-04 | an attachment | stored | its MIME is validated against an allowlist; unknown/dangerous types are inert (never executed/inlined) | P0 |
| SEC-05 | an oversized log/body | captured | it is truncated with a `truncated` marker; the viewer never inlines huge blobs | P1 |
| SEC-06 | a safe-share export | produced | `strict` redaction applied and a warning shown before media-heavy/sensitive bundles | P0 |
| SEC-07 | any view at read time | rendered | no external/CDN calls; fonts/assets are local; no telemetry/phone-home | P0 |

## RES — Responsive & mobile (from `03 §11`, `06`, `07`)

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| RES-01 | every screen | at 320 / 390 / tablet / desktop, light + dark | it is usable with **zero horizontal overflow** on non-matrix pages | P0 |
| RES-02 | a table/matrix on phone | rendered | controlled horizontal scroll with sticky label column, **or** stacked-card layout — not a squeezed desktop table | P0 |
| RES-03 | bottom/hamburger nav on phone | shown | it does not cover content (content padding accounts for it) | P1 |
| RES-04 | a detail/side-by-side drawer on phone | opened | it is full-screen | P1 |
| RES-05 | long names/ids/urls/stacks/tags | shown | they wrap or truncate intentionally (expand/copy), never breaking layout | P0 |
| RES-06 | every screen in `07` | design QA | screenshots exist at 320/390/tablet/desktop in light + dark (visual-regression baseline) | P1 |

## MIG — Migration & rollout (from `12`)

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| MIG-01 | the current reporter command | run after cutover | a backward-compatible wrapper/alias still works (or legacy behind a flag) | P0 |
| MIG-02 | historical results (JUnit/Allure) | imported | they backfill the store non-destructively (no deletion/mutation of existing outputs) | P1 |
| MIG-03 | the new viewer fails to generate | a run | the framework falls back to writing the raw store + a clear message; the run is not lost | P0 |
| MIG-04 | rollout | executed | order is core → framework pins → orchestrator, each PR validated before the next | P1 |

## EXP — Export round-trip (from `04 F13`, `10`)

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| EXP-01 | portable HTML / CSV / XLSX / JSON / evidence-zip | exported then opened | content is correct; HTML opens without serve; read-only enforced | P0 |
| EXP-02 | any export leaving the machine | produced | redaction/safe-share applied per `10`; redaction profile recorded | P0 |

## PERF — Performance (from `08 §5`)

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| PERF-01 | a ≥5k-result run over serve | opened | interactive < 1.5s; tree/list built from light index only | P0 |
| PERF-02 | a 5-run × 5k comparison | scrolled | 60fps via virtualization | P1 |

## F14 — AI Assist *(deferred v2 — DEC-07; not built in v1)*

| ID | Given | When | Then | Pri |
|----|-------|------|------|-----|
| F14-01 | v1 | any view | the AI module is absent; no failure data leaves the machine | P0 |
| F14-02 | *(if built later)* AI on + key | a failure | suggestion only, labelled AI-suggested, never auto-applied; off by default | v2 |

---

## Release gate (roll-up)

- **Alpha:** all P0 across G, DATA, F1, F2, F3, F5, **SEC** pass.
- **Beta:** all P0 + all P1 across every feature pass; **SEC/RES/MIG/EXP/PERF P0** pass; parity checklist (`08 §9`) met.
- **1.0:** Beta + P2 triaged (done or explicitly deferred), a11y audit (WCAG 2.2 AA) passed, PERF verified on the ≥5k fixture, visual QA at all breakpoints/themes (RES-06) passed, security fixtures (`13`) green.
