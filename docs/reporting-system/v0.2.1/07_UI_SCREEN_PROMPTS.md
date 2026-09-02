# 07 — UI Screen Prompts (design governance)
### Strata — Automation Run Reporting System

> These prompts **fully govern the design work**. The design team produces **only** the screens defined here, using **only** tokens/components from `06_DESIGN_SYSTEM.md`, showing **only** data defined in `02`/`04`. No new screens, no invented data, no off-spec styling. Every screen must be delivered in **Light and Dark**. Each prompt lists: Purpose · Regions · Components (from `06`) · Data (from `02`/`04`) · States · Responsive · Do-not.

**Global rules for every screen (apply to all S-#):**
- Use the fixed shell (top bar + left rail) from S-00 unless the screen is a full-screen overlay.
- Status = shape + colour (`06 §3`). Metrics/IDs/durations in `IBM Plex Mono`. Sentence case. No `→` on buttons.
- Provide **loading (skeleton)**, **empty**, **error**, and where noted **live** states — not just the happy path.
- **Do not invent data.** Any field/metric/evidence that may be absent (owner, severity, visual-diff, video, trace, mobile logs, external issue status) must have an explicit **unavailable/disabled state with a reason** — never blank, broken, or fabricated.
- **Redaction:** show captured headers/bodies/logs as already redacted (`10`); never a raw-secret layout.
- Everything is addressable: show the URL-state affordances (selected run/lens/filter/test reflected).
- Deliver both themes; verify AA contrast; visible focus; reduced-motion variant.
- **Deliver explicit mobile layouts**, not just breakpoints: each screen defines its **320 / 390 / tablet / desktop** layout, with tables→cards, sticky-column matrices, full-screen drawers, and bottom-nav that never covers content (`06 §15`). Provide **screenshots at 320/390/tablet/desktop in light + dark** per screen (`05 RES-06`).
- Density: design **compact** and **comfortable** variants for data-dense screens (S-02, S-05).

---

## S-00 — App shell (top bar + left rail + theme)

**Purpose:** persistent navigation and global controls.
**Regions:** top bar [wordmark+motif · Run switcher ▾ · Compare button · global search ⌕ · Gate badge · density toggle · theme toggle (Light/Dark/Auto)]; left rail [Overview, Explorer, Compare, Trends, Flaky, Triage, Runs, Settings] with icons + labels, active item in `--accent`.
**Components:** wordmark (`06 §14`), run switcher dropdown, gate badge (verdict colour), theme toggle, icon-nav.
**Data:** active `runId`, `run.gate.verdict`, store run list for the switcher.
**States:** live run → run switcher shows a pulse on running runs; empty store → rail present but sections show first-run empty states.
**Responsive:** < 1024px the left rail collapses to icons; < 720px it becomes a bottom/hamburger nav.
**Do-not:** no ALL-CAPS nav, no middle-dot meta, no decorative gradient in the bar.

## S-01 — Run Overview (F1)

**Purpose:** fastest true read of one run; stakeholder home.
**Regions (top→bottom):** run header with **core-sample rollup bar** + env fingerprint chip; **verdict band** (largest); KPI stat-card row (Total, Pass-rate +Δ, Failed, Broken, Flaky, Skipped, Known, Blocked, Duration +Δ); two-up charts (status donut, platform stacked bar); **What-changed** panel; **Top risks** list; defect-mix donut; trend spark row; slowest-tests strip.
**Components:** stat card, rollup bar, verdict band, chart frame (donut/bar/spark), chips.
**Data:** `run.summary`, `run.gate`, Δ vs previous run, F5 what-changed lines, severity-weighted top risks.
**States:** live (progress bar executed/total, partial marks, auto-refresh); no-previous-run (hide Δ + what-changed); gate not configured (verdict = "Not evaluated" + configure link).
**Responsive:** cards wrap 4→2→1; charts stack.
**Do-not:** don't bury the verdict; don't make every KPI a same-size card grid with identical shadows — verdict and pass-rate dominate.

## S-02 — Explorer (F2) — tree/list + facets + detail

**Purpose:** navigate one run by any lens; filter/search to a precise slice; open a test.
**Regions:** left [lens selector, facet panel with live counts, saved-filter dropdown, role-preset chips]; middle [Tree⇄List toggle; tree nodes with rollup bars + counts + stratigraphic depth rail; list = virtualized data table]; right [Test Detail drawer S-03, context preserved].
**Components:** facet panel, tree node, data table, status/defect chips, saved-filter dropdown, bulk-action bar.
**Data:** `results/index.json` (light) for tree/list/facets; full result on selection.
**States:** loading skeleton tree; empty-after-filter (name excluding facet + clear); live (rows stream in); large-run compact density.
**Responsive:** three panes → detail becomes a full-height drawer < 1200px; facets collapse to a filter button < 900px.
**Do-not:** don't load all results up front; don't lose the tree when a test is open.

## S-03 — Test Detail & Evidence (F3) — drawer

**Purpose:** everything to understand one test and decide bug-vs-not.
**Regions:** header (name/path/status/flaky/platform/severity/owner/tags/duration/attempt selector/issue links); **triage bar** (defect chip + classify + comment + link); failure block (message, failed/broken indicator, stack, diff) for non-pass; attempts strip; **steps tree**; **evidence tabs** (polymorphic — render only present blocks).
**Components:** status/defect chips, steps tree, evidence tabs, diff viewer, attempts strip.
**Data:** full `<resultId>.json` (`02 §5`), media by path.
**Evidence sub-prompts (design each):**
- **S-03a Screenshot gallery** — failure-shot first; before/after pairs; zoom; caption.
- **S-03b Video** — player with a scrubber that snaps to step timestamps; step click ⇄ video seek.
- **S-03c Trace (web)** — embedded trace viewer or a clear "Open trace" launch; explain if trace only on failure.
- **S-03d Network** — request table (method/url/status/latency), row → headers+body; status-coloured rows.
- **S-03e Console/logs** — level-coloured, chunked, copyable, searchable.
- **S-03f Requests (API)** — method/url/status/latency; req+res headers/body; payload assertions (path/expected/actual/pass).
- **S-03g Mobile** — device chip, screen recording, screenshots, device logs, native/webview context timeline.
- **S-03h Desktop** — recording, screenshots, window/aX tree, OS logs, resource peaks.
- **S-03i Visual diff** — baseline/actual/diff slider + pixel/layout delta.
**States:** absent evidence → tab disabled with reason; passing test → collapse failure/diff; flaky → failing+passing attempts side by side; large log → lazy.
**Responsive:** drawer full-screen < 900px; evidence tabs become a select.
**Do-not:** never a broken image for missing evidence; never colour-only status.

## S-04 — Triage board (F4)

**Purpose:** classify every non-pass with evidence.
**Regions:** To-Investigate queue + columns per defect type (kanban, classify-only); filters (platform/suite/owner/severity); **error-signature clusters** panel.
**Components:** defect chips, cluster card (member count, affected suites/platforms, representative evidence), bulk-move bar.
**Data:** non-pass results + `defect`, normalized error signatures.
**States:** all-classified empty state; a cluster expands to members.
**Do-not:** no execution controls (Strata doesn't run tests).

## S-05 — Comparison (F5) — matrix + side-by-side

**Purpose:** what changed across 2–5 runs, completely.
**Regions:** run-picker header (choose ≤5, set baseline, arrange order) + env **drift** banner; **status matrix** (sticky test column, one column per run, status cells = shape+colour+duration, change badge), grouped by lens; change-category facet chips with counts; summary strip (pass-rate per run + deltas, new-fail/fixed/added/removed counts); **side-by-side** panel when a matched test is selected (attempts/evidence of each run next to each other).
**Components:** comparison matrix, change facets, rollup summary, side-by-side evidence, drift banner.
**Data:** matched by `testId`; all change categories (`04 F5`); env per run.
**States:** name-based-identity warning; >5 blocked; param family → sub-rows.
**Responsive:** matrix horizontally scrolls with sticky test column; side-by-side stacks < 1100px.
**Do-not:** don't hide added/removed; don't imply a regression when only env drifted (surface drift).

## S-06 — Trends & History (F6)

**Purpose:** long-view quality across all stored runs.
**Regions:** range control; chart grid (pass-rate line, status stacked-area, flaky line, duration p50/p95, defect-mix area, test-count line); tables (most-failed / flakiest / consistently-slow top-50); per-test history view.
**Components:** chart frames, data tables, sparkline dots.
**Data:** `index.runs[]` series; per-`testId` history.
**States:** <2 runs → "need more runs"; brush-to-zoom; point → run Overview.
**Do-not:** no gradient decoration; keep status colours consistent with §3.

## S-07 — Flaky (F7)

**Purpose:** surface and quantify instability honestly.
**Regions:** flaky list (grade A–F, flaky-rate, last-N dots, retry count, affected suites/platforms); flaky-rate trend; per-test failing-vs-passing attempt diff; bulk-mute action.
**Components:** stability-grade badge, dot history, attempt-diff, mute control.
**Data:** retry-recovered + history-unstable signals.
**States:** no-retries note; muted items marked.

## S-08 — Quality Gates & Settings (F8/F11)

**Purpose:** configure verdicts and the store without editing files.
**Regions:** gate rule editor (rows of **metric / operator / threshold / severity / scope** — structured, no free-form expression) with **live preview** verdict against current run; custom-lens manager; theme default; default preset/lens; saved filters; mute/known registry; metadata-inference mappings; duration-regression thresholds; **redaction & capture config** (profiles, allow/deny lists, capture modes, PII toggle); export/safe-share defaults.
**Components:** rule rows (metric/operator/threshold/severity/scope), preview verdict band, toggles, allow/deny list editors.
**Data:** `config.json`.
**States:** preview-before-save for verdict-affecting changes; redaction changes noted as applying to future runs.
**Do-not:** no free-form/`eval` rule expressions; **no AI key/toggle in v1** (AI assist is v2 roadmap — DEC-07).

## S-09 — Runs management (F10)

**Purpose:** own the store; manual retention.
**Regions:** runs table (seq/label/trigger/env/counts/pass-rate/gate/tags/pinned/size); actions (rename, tag, pin, delete-with-freed-space, export, compare-select ≤5); store-health summary (total size, counts, largest/oldest, delete hint).
**Components:** data table, confirm dialog (shows freed MB), bulk-action bar.
**Data:** `index.json`.
**States:** empty store → onboarding (how runs are produced, how to serve/export); delete confirm.
**Do-not:** never auto-delete; confirm every delete.

## S-10 — Live run (F12) — overlay behaviour on S-01/S-02

**Purpose:** watch a run in progress.
**Regions:** live badge + progress (executed/total, elapsed, ETA) in header; results stream into Overview/Explorer; transient running/not-run states.
**States:** aborted (partial preserved); reconnecting (poll failure) without losing view.
**Do-not:** don't show comparison/trends as if final for a running run.

## S-11 — First-run / empty store

**Purpose:** the state before any run exists.
**Regions:** brief explainer of how the framework populates the store, and the two ways to view (serve / export), with the exact commands. One primary action.
**Components:** empty-state card, code snippet (mono), primary button.
**Do-not:** no marketing copy; direct and instructional (`06 §13`).

## S-12 — Exported single-run bundle (F13)

**Purpose:** the read-only portable HTML for people who won't run a server.
**Constraints:** identical visual language; editing/triage disabled (hidden, not broken); a small "exported · read-only · <timestamp> · <env>" marker; all media embedded; no live/compare-across-store features (single run only).
**Do-not:** don't show controls that require the store or serve process.

---

## Delivery checklist for the design team (per screen)

- [ ] Light + Dark, AA-verified.
- [ ] Loading / empty / error (and live where applicable) states.
- [ ] **Unavailable-data states** for every optional field/metric/evidence (reason shown, nothing fabricated).
- [ ] **Redacted** display of captured headers/bodies/logs (`10`).
- [ ] Only `06` tokens/components; no new colours/type/radii/shadows.
- [ ] Only `02`/`04` data fields; no invented content.
- [ ] Status by shape + colour; metrics in mono; sentence case; no `→`.
- [ ] **Explicit layouts + screenshots at 320 / 390 / tablet / desktop, light + dark** (tables→cards, sticky-column matrices, full-screen drawers, bottom-nav clearance).
- [ ] Keyboard focus + reduced-motion; local fonts (no remote).
- [ ] Traces to a feature in `04` and satisfies its UI-facing rows in `05` (incl. RES).
