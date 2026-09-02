# 06 — Design System
### Strata — Automation Run Reporting System

> The complete visual language. **The design team uses only what is defined here.** No off-spec colours, typefaces, radii, shadows, or motion. Two themes are first-class: **Light**, **Dark** (Auto = follow OS, the runtime default). Where a value below is a token, the build exposes it as a CSS custom property of that exact name.

---

## 1. Design concept (the one memorable idea)

**Strata = layers you can see through.** The product's essence is depth: run → suite/feature → test → attempt → step → evidence. The identity makes that literal with a **stratigraphic cross-section motif** — thin stacked layer-bands that encode depth and status. This is the single place we spend boldness; everything else stays quiet and data-first.

- The motif appears in exactly three places: the wordmark, the run-header "core sample" strip (a horizontal cross-section summarising the run's status composition), and the Explorer tree's depth rail. Nowhere else — it is structure, not decoration.
- Deliberately **not** an instrument-panel/telemetry look and **not** a generic, templated default (see §11).

**Principles:** (1) Evidence and data are the hero, chrome recedes. (2) Status is read by **shape + colour**, never colour alone. (3) Density is a setting, not an accident — engineers get compact tables, stakeholders get breathing room. (4) One structural motif, used consistently, carries the brand.

---

## 2. Colour — base palette (6 named values per theme)

| Token | Role | Light | Dark |
|-------|------|-------|------|
| `--base` | app background | `#F6F7F9` | `#0F131A` |
| `--surface` | panels, cards, rows | `#FFFFFF` | `#161B23` |
| `--surface-raised` | popovers, side panels, modals | `#FFFFFF` (+shadow) | `#1D232D` |
| `--ink` | primary text | `#1A1D24` | `#E7EAF0` |
| `--ink-muted` | secondary text, labels | `#5B6472` | `#9BA5B4` |
| `--line` | borders, dividers, gridlines | `#E4E7EC` | `#262D38` |

Notes: light base is a **cool blue-grey paper**, not cream; dark base is a **blue-slate**, not tinted black. Text darks are true slate (`#1A1D24`), not `#111`.

**Brand accent (interaction only):**

| Token | Role | Light | Dark |
|-------|------|-------|------|
| `--accent` | links, selection, focus ring, active nav, primary button | `#2B57E6` (lapis) | `#5C86FF` |
| `--accent-weak` | hover wash, selected-row tint | `#EAF0FE` | `#1B2740` |
| `--accent-ink` | text on `--accent` | `#FFFFFF` | `#0F131A` |

`--accent` (lapis) is reserved for interaction and is **never** used to mean a status. It is intentionally outside the status hue set.

---

## 3. Colour — status & overlay palette (semantic, fixed)

**Status colour tokens** map to the model in `03 §2`/`02 §7`: five **raw statuses** (`passed`, `failed`, `broken`, `skipped`, `blocked`) plus `known` (an **effective-status** grouping, not a raw status). `flaky` is **not a status** — it is an **overlay badge**. Each token has `-fg` (text/icon on base), `-bg` (chip fill), and a **shape** (icon) so it reads without colour.

| Token | Kind | Shape (Lucide) | Light `-fg` / `-bg` | Dark `-fg` / `-bg` |
|-------|------|----------------|---------------------|--------------------|
| `--status-passed` | raw status | `check` | `#12855A` / `#E3F5EB` | `#42D695` / `#10281E` |
| `--status-failed` | raw status | `x` | `#C6362E` / `#FBE7E5` | `#F26A61` / `#2E1614` |
| `--status-broken` | raw status | `alert-triangle` | `#B4530E` / `#FBEEDF` | `#F0913E` / `#2C1E0D` |
| `--status-skipped` | raw status | `minus-circle` | `#5B6472` / `#EDEFF3` | `#8A93A3` / `#1C212A` |
| `--status-blocked` | raw status | `slash` | `#8A3A46` / `#F6E7E9` | `#D08A94` / `#281519` |
| `--status-known` | effective grouping | `bookmark` | `#8A6D00` / `#FBF3D6` | `#E3C24B` / `#26200C` |
| `--overlay-flaky` | **overlay badge** | `waves` | `#7A45D1` / `#F1E9FC` | `#B389F5` / `#1F1630` |

Rules: every `-fg`/base pairing meets WCAG AA (4.5:1 text, 3:1 large/icon) — the build **verifies** and nudges values if a pairing misses, keeping hue identity. `failed` (red) and `broken` (rust-orange) must stay visually separable at chip size. The **flaky overlay badge** co-exists with any status (commonly `passed`); in charts it is an overlay band/slice, never a terminal-status slice.

**Transient (live):** `running` = `--accent` with a soft pulse; `not-run` = `--ink-muted` outline.

---

## 4. Colour — defect-type chips

Defect types are **outline chips** (border + `--surface` fill + a leading dot), so they never compete with status fills:

| Defect | Dot colour (light / dark) |
|--------|---------------------------|
| Product Bug | `#C6362E` / `#F26A61` |
| Automation Bug | `#2B57E6` / `#5C86FF` |
| System / Infra | `#5B6472` / `#8A93A3` |
| Environment / Data | `#7A5B2E` / `#C9A667` |
| To Investigate | dashed neutral border, no dot |
| Known Issue | `#8A6D00` / `#E3C24B` |
| No Defect | `#12855A` / `#42D695` |

---

## 5. Typography

Two families, chosen deliberately (not the common Space Grotesk / JetBrains Mono defaults):

- **UI / display — `Schibsted Grotesk`** (fallback: `Hanken Grotesk`, system-ui). Carries headings, nav, body, buttons. Weights: 400/500/600/700.
- **Data / metrics / code — `IBM Plex Mono`** (fallback: `ui-monospace`, `SFMono-Regular`). Used for **all** numbers-as-data (counts, %, durations, deltas), IDs, file paths, stack traces, request/response bodies, and code. Tabular by nature → columns align.

**Type scale** (1.20 minor-third, base 14px for dense UI; 15px comfortable):

| Token | px (compact/comfortable) | Use |
|-------|--------------------------|-----|
| `--t-display` | 30 / 34 | run verdict number, hero metric |
| `--t-h1` | 22 / 24 | section titles |
| `--t-h2` | 18 / 19 | panel titles |
| `--t-h3` | 15 / 16 | card/row headers |
| `--t-body` | 14 / 15 | body, table cells |
| `--t-meta` | 12.5 / 13 | secondary labels, captions |
| `--t-mono` | 13 / 13.5 | metrics, code, IDs (IBM Plex Mono) |

Rules: **sentence case** everywhere (no ALL-CAPS eyebrows). Line length ≤ 80ch for prose. Don't accent a single word in a heading for effect. Labels are plain words, not `WORD — fragment`. Metrics pair value + unit + delta on one line (`93.6% ↘2.1`), value in `--t-mono`.

**Font loading:** reports run **local/offline**. Fonts are **bundled with the viewer** (self-hosted woff2) with system fallbacks (`ui-sans-serif`, `ui-monospace`); **no remote/CDN font requests** — the report must render fully with no network (`10 §4`, `08 §2`).

---

## 6. Layout & spacing

- **Grid:** 4px base unit; spacing tokens `--sp-1`=4 … `--sp-8`=32. 12-col fluid content; the app is a **fixed-shell + scrolling content** (left rail + top bar persistent).
- **Density modes:** `comfortable` (default for stakeholder preset) and `compact` (default for engineer presets) — a global toggle changing row height (36/28px), padding, and `--t-body` base.
- **Radii (purposeful, not uniform):** `--r-control`=6 (buttons, inputs, chips-as-buttons), `--r-panel`=10 (panels, cards, modals), `--r-pill`=999 (status/defect chips). Never one radius on everything.
- **Alignment:** data left-aligned; numeric columns right-aligned with tabular figures; headers align to their column data.

**Explorer wireframe (compact):**
```
┌ top bar ────────────────────────────────────────────────────────────┐
│ ▚ Strata  | Run #42 ▾ | Compare | ⌕ | Gate: PASS | ◑ |               │
├ rail ┬ lens+facets ─┬ tree / list ───────────────┬ test detail ──────┤
│ Ovv  │ Lens: Suite ▾│ ▸ Web        470/500  ▓▓▓░ │ Guest checkout  ✕ │
│ Exp◈ │ ── facets ── │  ▸ Commerce   88/96   ▓▓░░ │ failed · web · p1 │
│ Cmp  │ [status]     │   ▾ Checkout  52/60   ▓░░░ │ ── steps ──       │
│ Trd  │ [platform]   │     ✕ Guest checkout  12.8s│ ▾ Place order  ✕  │
│ Flk  │ [defect]     │     ✓ Coupon apply    3.1s │ ── evidence ──    │
│ Trg  │ [flaky] …    │     ~ Search (flaky)  5.2s │ [shot][video][…]  │
│ Runs │ saved ▾      │                            │                   │
└──────┴──────────────┴────────────────────────────┴───────────────────┘
```

---

## 7. Elevation & borders

Avoid the "same soft grey shadow under every card" tell. Hierarchy is built with **borders + surface tint** first; shadow is reserved for things that truly float.

| Token | Use | Value (light / dark) |
|-------|-----|----------------------|
| `--elev-0` | inline panels, rows, cards | border `--line`, no shadow |
| `--elev-1` | popovers, dropdowns, side panels | `0 4px 16px rgba(16,22,34,.10)` / `0 4px 16px rgba(0,0,0,.5)` |
| `--elev-2` | modals, command palette | `0 12px 40px rgba(16,22,34,.16)` / `0 12px 40px rgba(0,0,0,.6)` |

---

## 8. Components (canonical set)

The design team composes screens from these; no bespoke one-offs.

- **Status chip** — pill, `-bg` fill + `-fg` icon(shape)+label. Compact variant = icon only with tooltip.
- **Defect chip** — outline + dot (§4), clickable to classify.
- **Stat card** — label (`--t-meta`), value (`--t-mono`, `--t-display`/`--t-h1`), delta (arrow + mono), optional sparkline; whole card is a filter link.
- **Rollup bar** — a thin horizontal stacked bar of status composition (the "core sample"), used on tree nodes and the run header.
- **Tree node** — disclosure + name + rollup bar + counts; depth shown by the stratigraphic depth rail.
- **Data table** — virtualized, sortable headers, sticky header, right-aligned numerics, row-select, density-aware.
- **Facet panel** — grouped checkboxes/toggles with live counts; clear-all.
- **Evidence tabs** — screenshot gallery, video player (step-synced scrubber), trace embed/launch, network table, console list, request/response viewer, visual-diff slider, log viewer (chunked, copyable).
- **Steps tree** — nested, per-step status shape + duration + evidence affordance; failed step auto-expanded.
- **Comparison matrix** — sticky first column (test), one column per run, status cells (shape+colour+duration), change badge; grouped by lens.
- **Verdict band** — the gate result; largest type on Overview; names failing rules.
- **Chart frame** — title, legend (status shapes), tooltip, table-toggle, click-through.
- **Side panel / drawer** — for test detail and side-by-side compare; keeps context.
- **Command palette** *(optional)* — `⌘K`.
- **Toast / inline confirm** — for classify/mute/delete; action names match their buttons.

**Buttons:** primary (accent fill), secondary (outline), ghost (text). Verb labels state the outcome ("Delete run", "Classify", "Export CSV"). **No `→` appended.** No ALL-CAPS.

---

## 9. Charts (styling)

- Colour strictly by status role (§3); non-status series use `--accent` then a defined neutral ramp.
- Gridlines `--line`, thin; axis labels `--ink-muted`; value labels `--t-mono`.
- Every chart: shape-coded legend, exact-value tooltips (count + %), an accessible **table toggle**, and click-through to a filtered view. Both themes verified.
- Donuts show the total in the centre (mono). Sparklines are unlabeled but tooltip on hover. Matrix/heatmap cells use status fills + shape at larger sizes.
- No gradient washes as decoration.

---

## 10. Motion

- **One orchestrated moment:** on opening a run, the run-header rollup "core sample" settles into its segments once (≤ 400ms). That is the signature motion.
- Everything else is **action-response** only: drawers slide in on open, rows expand on click, chips confirm on classify. Durations 120–200ms, ease-out.
- No fade-slide-up on every section/card. `prefers-reduced-motion` disables all non-essential motion (rollup settles instantly).

---

## 11. Anti-patterns (do NOT produce)

To keep the identity distinct and non-generic, avoid:
- Cream background + high-contrast serif + terracotta accent (esp. `#D97757`).
- Near-black + acid-green/vermilion.
- Broadsheet hairline/newspaper columns as the theme.
- The SaaS-card kit: identical rounded cards everywhere, one radius on everything, the same soft grey shadow under each, gradient washes.
- Template chrome: ALL-CAPS eyebrow labels, `A · B · C` middle-dot meta strings, `WORD — fragment` spaced-em-dash labels, `#0B0B0B`/`#111` standing in for black, monospace used decoratively (mono here is for **data**, not labels), `→` appended to buttons/links.
- A telemetry/instrument-panel look with phosphor accents and a Space Grotesk + JetBrains Mono pairing — Strata has its own layered identity.

---

## 12. Accessibility (required)

WCAG 2.2 AA: text ≥ 4.5:1, large/icon ≥ 3:1 (verified per §3); status conveyed by **shape + colour**; full keyboard operation with visible `--accent` focus rings; hit targets ≥ 24px (compact) / 32px (comfortable); `prefers-reduced-motion` and `prefers-color-scheme` respected; charts have table equivalents; drawers/modals trap focus and restore it; live regions announce run-progress updates.

---

## 13. Copy voice

Plain, active, sentence case, no filler. Buttons name the outcome and keep that name through the flow ("Publish" → "Published"). Errors say what happened and how to fix it, in the interface's voice, never apologetic or vague. Empty states are invitations to act ("No runs yet — run your suite and Strata records it here."). Name things by what the user understands (a user "classifies a failure", not "sets defectType enum").

---

## 14. Brand assets

- **Wordmark:** "Strata" set in Schibsted Grotesk 600, with the cross-section motif as a compact glyph (stacked layer-bands) to its left. Monochrome (`--ink`) by default; accent only on active/brand surfaces.
- **App icon:** the layer-band glyph on `--surface`.
- *(Working name — may be renamed in one place per `00 §0.4`.)*

---

## 15. Responsive component rules (dense tables, matrices, charts)

Breakpoints: **320 / 390 (phone) · tablet · desktop** (`03 §11`, acceptance `05 RES`).

- **Data table (`08`):** desktop = full columns; tablet = hide low-priority columns behind a "more" toggle; **phone = one card per row** (primary fields stacked: name, status chip, key metric, evidence icons) — never a squeezed desktop table.
- **Comparison matrix:** desktop = full grid; tablet/phone = **sticky test column** + horizontal scroll of run columns, with a scroll affordance; optionally a per-test expandable card on phone. Zero page-level horizontal overflow (scroll is contained to the matrix).
- **Charts:** below a min width a chart falls back to its **accessible table** form automatically; donuts collapse to a labelled legend+values list; sparklines keep tooltips.
- **Nav:** left rail → icon rail (tablet) → bottom/hamburger (phone), content padded so the bottom bar never overlaps.
- **Drawers** (Test Detail, side-by-side): full-screen on phone with a clear back affordance.
- **Long tokens** (ids, urls, stacks): wrap or middle-truncate with expand/copy.

## 16. Chart & table fallback

Every chart has a **table equivalent** toggle (also the a11y path, `§9/§12`); when data is insufficient for a chart (e.g. <2 runs for a trend), show the honest empty state, not an empty axis. When a metric is unavailable, show "—", never a fabricated value.
