# 03 — Information Architecture
### Strata — Automation Run Reporting System

> Governs how information is organized, classified, navigated, searched, and addressed. Consumed by both the implementation team (behavior) and the design team (structure of every screen). Where this file defines a taxonomy or a state, screens and code must match it exactly.

---

## 1. The layer model (multi-lens hierarchy)

The core IA idea: **the same run is viewable through several hierarchies ("lenses").** A user picks the lens that matches how they think, and every lens supports the same drill path:

```
Run → [Lens node…] → Test → Attempt → Step → Sub-step → Evidence
```

### 1.1 Lenses (hierarchy dimensions)

| Lens | Tree shape | Primary audience |
|------|-----------|------------------|
| **Suite** | parent → suite → sub-suite → test | Developer, Automation |
| **Behavior** | epic → feature → story → test | Stakeholder, QA |
| **Package / Path** | folder → file → test | Developer |
| **Platform** | web / api / mobile / desktop → test | QA, Automation, Stakeholder |
| **Tag** | tag → test (a test appears under each of its tags) | QA (smoke/p1/regression) |
| **Owner** | owner → test | Developer, lead |
| **Severity** | blocker → critical → major → minor → trivial → test | Stakeholder, QA |
| **Custom** | any promoted label key (e.g. team, component) → test | configurable per store |

- Lenses are **built from labels already in each result** (`02 §5`), so switching lens is instant and client-side.
- A test can appear under multiple nodes of a lens (e.g. two tags) — this is expected, not a bug.
- The available lenses per run come from `run.json.layers.available`; custom lenses come from `index.json.customLayers`.
- **Every tree node shows rollup stats** (counts by status, pass-rate, flaky, duration) so a user can triage at any level without expanding to leaves.

### 1.2 Drill levels (the "tier" a user can filter/search to)

Search and filter operate at **every** level, not only on flat tests:

`Run-level` → `Lens-node level` (suite/feature/platform/…) → `Test level` → `Attempt level` → `Step level` → `Evidence level`.

Example journeys the IA must make one path each:
- Stakeholder: Run → Overview (no drill).
- QA: Run → Platform=web → Suite=Checkout → filter status=failed → Test → screenshot.
- Developer: Run → Owner=me → status∈{failed,broken} → Test → failing Step → trace.
- Automation: Run → filter flaky=true → sort by history-instability → Test → attempts diff.

---

## 2. Status model (strict — `rawStatus` + `effectiveStatus` + `flaky` overlay)

Two status fields plus a boolean overlay (full computation in `02 §7`). **`flaky` is never a terminal status.**

**`rawStatus`** — what happened on the final attempt. Exactly **five** terminal values:

| rawStatus | Meaning | Colour role (see `06`) |
|-----------|---------|------------------------|
| `passed` | Met all assertions | `--status-passed` (green) |
| `failed` | An **assertion** did not hold (expected ≠ actual) | `--status-failed` (red) |
| `broken` | An **unexpected error** (exception, timeout, selector/infra) — *not* an assertion | `--status-broken` (deep-orange) |
| `skipped` | Not executed by design | `--status-skipped` (slate) |
| `blocked` | Could not run: a prerequisite/setup failed | `--status-blocked` (grey-red) |

**`effectiveStatus`** — display/gate grouping = `rawStatus`, plus one added value:

| effectiveStatus | Meaning | Colour role |
|-----------------|---------|-------------|
| (the five above) | as `rawStatus` | as above |
| `known` | a `failed`/`broken` mapped by a muted/known-issue rule; excluded from gate-fail by default | `--status-known` (amber) |

**`flaky`** — a **boolean overlay**, independent of status: passed-after-retry or history-unstable. A `passed` (or any) result can be `flaky:true`. In charts it appears as an **overlay slice** (violet `--overlay-flaky`), never as a terminal status; in pass-rate a flaky-but-passed test still counts as passed, flagged.

**Transient (live only):** `running` (pulse), `not-run` (muted).

**`failed` vs `broken` must remain distinct everywhere** (chips, charts, filters, counts) — the backbone of bug-vs-not triage. Counts and lens rollups group by **effectiveStatus**; `flaky` is counted as a separate overlay dimension.

---

## 3. Defect taxonomy (triage classification of any non-pass)

Applied by QA/dev to `failed` / `broken` (optionally `known`) results. Powers "is it a bug or not."

| Defect type | Meaning | Default for |
|-------------|---------|-------------|
| **Product Bug** | Real defect in the system under test | — |
| **Automation Bug** | Defect in the test/script/framework | — |
| **System / Infra Issue** | CI, runner, network, container, driver | — |
| **Environment / Data Issue** | Bad test data or environment state | — |
| **To Investigate** | Unclassified — the triage queue | **every new non-pass** |
| **Known Issue** | Linked to a tracked defect; expected | — |
| **No Defect** | False alarm / expected behavior | — |

- New non-pass results default to **To Investigate** so nothing is silently accepted.
- Classification is stored on the result (`02 §5`, mutable via serve).
- Optional `source: auto|ml` reserves for a future **v2** AI-assist module (DEC-07): ML may *suggest* a type; a human confirms. Not present in v1.
- The comparison view uses defect type (e.g. "new Product Bugs vs baseline" as a gate input).

---

## 4. Search & filter model

### 4.1 Global search
One search box, scoped to the current run (or comparison set). Matches across: test name, full name, error message/type, tag, owner, file path, label values, defect category, external issue key. Results are grouped by lens node. Search is **client-side** over `results/index.json` (light index) with on-demand detail.

### 4.2 Facets (filter dimensions)
All combinable (AND across facets, OR within a facet):

- **Status** (7) · **Flaky** (yes/no) · **Defect type** (7) · **Platform** (4) · **Severity** (5)
- **Lens node** (any suite/feature/package/tag/owner/custom node)
- **Duration** (range slider; presets: slow > p95, > Ns)
- **Evidence present** (has video / has trace / has screenshot / has network / has visual-diff)
- **Change vs baseline** (only meaningful in comparison: new-fail / fixed / added / removed / unchanged / status-changed / duration-regressed)
- **Owner** · **Tag** · **Muted/known** · **Has linked issue**
- **Attempts** (retried / first-try)

### 4.3 Behaviors
- Facet chips show live counts; selecting a facet updates other facet counts (faceted, ReportPortal/Allure-style).
- **Saved filters:** name and store a filter set (in `config.json`) for reuse; ship presets per role (§7).
- **Deep-linkable:** the full filter + search state is URL-encoded (§6) so any filtered view is shareable.
- **Bulk actions** on a filtered set: classify defect type, mute, export CSV/JSON, copy links.

---

## 5. Navigation shell (global)

```
┌───────────────────────────────────────────────────────────────────┐
│  Top bar:  PRODUCT_NAME · Run switcher ▾ · Compare (pick ≤5) · ⌕ search · Gate badge · Theme ◑ │
├───────────┬───────────────────────────────────────────────────────┤
│ Left rail │  Main content (per section)                            │
│  Overview │                                                        │
│  Explorer │   Lens selector ▾  |  Facet panel  |  Tree/List  |  Detail │
│  Compare  │                                                        │
│  Trends   │                                                        │
│  Flaky    │                                                        │
│  Triage   │                                                        │
│  Runs     │                                                        │
│  Settings │                                                        │
└───────────┴───────────────────────────────────────────────────────┘
```

**Primary sections**
1. **Overview** — the run dashboard (KPIs, gate, breakdowns, what-changed, trend spark).
2. **Explorer** — lens tree + facets + result list + test detail (the workhorse).
3. **Compare** — 2–5 run comparison (matrix + diff).
4. **Trends** — history across all stored runs.
5. **Flaky** — stability analytics.
6. **Triage** — the To-Investigate queue + defect boards.
7. **Runs** — manage stored runs (label, pin, tag, delete, export).
8. **Settings** — quality gates, custom lenses, theme default, saved filters.

**Run switcher** is always present: pick the active run; "Compare" opens a picker to select up to five. Live runs (`status:"running"`) show a pulse and auto-refresh.

---

## 6. URL / state model (everything addressable)

Every meaningful state is encoded so it can be bookmarked and shared. Canonical query shape:

```
/#/<section>?run=<runId>&runs=<id,id,…>&lens=<suite|behavior|…>
   &node=<lensNodePath>&status=<a,b>&platform=<web,api>&flaky=1
   &defect=<productBug>&owner=<sara>&tag=<smoke>&q=<search>
   &dur=<min-max>&change=<new-fail>&test=<resultId>&attempt=<n>
   &step=<stepId>&evidence=<trace|video|network>&theme=<light|dark|auto>
   &sort=<field:dir>&filterId=<savedFilter>
```

Rules: opening a link reconstructs the exact view (run, lens, filters, selected test/attempt/step, evidence tab, theme). Missing params fall back to defaults from `config.json`. Theme in URL overrides stored default for that view only.

---

## 7. Role view presets (not permissions)

Presets are saved lens+facet+landing combinations. A "role" is a one-click preset; anyone can use any preset. Default presets:

| Preset | Lands on | Lens | Default facets | Emphasis |
|--------|----------|------|----------------|----------|
| **Stakeholder** | Overview | Behavior | — | Gate verdict, pass-rate + delta, top risks, trend |
| **QA** | Explorer | Platform → Suite | status∈{failed,broken}, flaky=yes | Triage + evidence + new-vs-baseline |
| **Developer** | Explorer | Owner / Package | status∈{failed,broken}, owner=self (if set) | Root cause, stack, trace, linked issue |
| **Automation** | Flaky | Suite | flaky=yes OR status=broken | Stability, automation-bugs, duration regressions |

The active preset is remembered per store (in `config.json`) and reflected in the URL.

---

## 8. Empty, loading, live & error states (global rules)

- **Loading:** skeletons for trees/lists/cards; media loads lazily with a spinner-in-place. Never block the whole screen on media.
- **Empty run store:** first-run screen explaining how the framework populates the store + a "how to serve/export" note.
- **No results after filter:** show which facet is excluding everything + one-click clear.
- **Live run:** live badge, auto-refresh (poll `run.json`/`results/index.json`), progress bar (executed/total), partial data clearly marked "run in progress."
- **Missing evidence:** show the evidence tab as disabled with reason (e.g. "trace not captured for passing tests") — never a broken image.
- **Corrupt/partial run:** viewer reads what it can and flags the run as `partial` rather than failing.

---

## 9. Information density & progressive disclosure

- Default views are **glanceable** (rollups, chips, sparklines). Full evidence, stacks, and step trees are **one interaction away** and never lose the surrounding context (side panels / expandable rows, not full navigations).
- Counts and percentages appear together (e.g. `760 (93.6%)`). Deltas always show direction + magnitude (`↘ 2.1%`).
- Long text (stack traces, request bodies, logs) is collapsible, copyable, and syntax-aware where applicable.

---

## 10. Baseline, lineage & comparability

Comparison and "what changed" must not blindly diff the previous sequential run when suites/profiles/environments differ (full formulas in `11 §7`).

- **Baseline** = most recent completed **comparable** run (same project + framework + profile + environment, and same test-plan/suite-set when present); else previous `seq` **with a visible caveat badge**.
- **Comparability badge:** when compared runs differ in framework/profile/environment, or their test sets only partially overlap, the view shows a **partial-coverage** badge.
- **Pass-rate basis label:** always state whether a rate/delta is over the **full run** or **shared tests only**.
- **Roles are presets, not authorization** (`§7`) — there is no access control; any user sees everything.

---

## 11. Responsive & mobile behavior (global rules)

The report is used on desktop, tablet, and phone (shared exports especially). Hard rules (acceptance in `05`):

- **Breakpoints:** 320, 390 (phone), tablet, desktop — every screen is usable at each.
- **Zero horizontal overflow** on all non-matrix pages; text and chrome wrap within the viewport.
- **Tables/matrices:** either controlled horizontal scroll with a **sticky label column**, or transform to **stacked cards** on phone — never a desktop table squeezed to overflow.
- **Navigation:** the left rail collapses to icons (tablet) then to a bottom/hamburger nav (phone); the bottom nav must **not cover content** (content padding accounts for it).
- **Drawers** (Test Detail, side-by-side) become **full-screen** on phone.
- **Long text** — names, run ids, urls, stack traces, tags — wraps or truncates intentionally (with expand/copy), never breaks layout.
