# 11 — Metrics & Defaults
### Strata — Automation Run Reporting System

> Exact definitions and defaults for every computed signal, so implementation is unambiguous and the report never "lies." All defaults are configurable in `config.json` unless noted. Rules use a **safe structured schema**, never evaluated expressions.

---

## 1. Counts & rates

- `total` = results with `effectiveStatus ∈ {passed,failed,broken,skipped,known,blocked}`.
- `passRate` = `passed / (passed + failed + broken + blocked)` — **excludes** skipped and known by default (report the denominator basis in the UI). Configurable to include known.
- `flakyRate` = `flakyTests / executedTests`, where `flaky:true` regardless of final status; `executed = total − skipped`.
- Counts group by **effectiveStatus**; `flaky` is an overlay count (`02 §7`).

## 2. Flaky

- **Retry-recovered:** any earlier attempt non-pass and final attempt passed → `flaky:true`, always (not configurable).
- **History-unstable:** over the flaky window, the test's outcomes alternate (both pass and non-pass occur). **Default window = last 20 comparable runs** (lineage-matched, §7), min 5 runs to judge.
- **Stability grade (A–F)** from the window: `stability = passConsistency × (1 − retryRate)`, where `passConsistency = passes/observations`, `retryRate = retriedRuns/observations`. Bands: A ≥ .97, B ≥ .90, C ≥ .80, D ≥ .65, E ≥ .50, F < .50. < min observations → "N/A (insufficient history)".

## 3. Duration

- **Slow test:** `durationMs > run p95` **or** `> absoluteSlowMs` (default 30000). 
- **Duration regression:** vs the test's historical **median** over the window; regressed when increase `> max(30%, 2000ms)`. Improvement symmetric. Requires ≥ 5 historical points; else "N/A".

## 4. Error-signature normalization (clustering)

`signature = hash(errorType + "|" + normalizeMessage(message))`. `normalizeMessage` strips: numbers, timestamps, uuids/hashes, quoted literals, file paths/line numbers, urls, and memory addresses; collapses whitespace; lowercases. Two failures with the same signature cluster together (`04 F4`).

## 5. Health score (severity-weighted, 0–1) & known debt

Weights by severity: blocker 5, critical 4, major 3, minor 2, trivial 1 (configurable). For each executed test, `weight × outcome`:
- `outcome = 1` for `passed`; `0` for `failed`/`broken`/`blocked`.
- **`known` = 0.5 (partial credit)** — acknowledged, but **not a full pass**, so known issues never produce a false green. (Configurable 0–1; default 0.5.)
- Flaky applies a 0.5 penalty factor to its test's outcome.

`healthScore = Σ(weight × outcome) / Σ(weight)`, reported 0–1 and as a letter grade (§2 bands).

**Known debt is surfaced separately, not hidden:** `knownDebt = count of effectiveStatus == known`, shown on the dashboard as its own figure (and severity-weighted) so acknowledged issues stay visible rather than inflating health. **`passRate` stays honest and independent** — it excludes `known` by default (§1); it is never blended with the health score.

## 6. Risk ranking (top risks)

Per non-pass result: `risk = severityWeight × recencyFactor × ownershipFactor × regressionFactor`.
- `recencyFactor`: newer first-failure ranks higher (1.0 if new this run, decaying by runs-since-first-failure).
- `ownershipFactor`: 1.2 if owned/assigned, 1.0 otherwise (surfaces actionable items).
- `regressionFactor`: 1.5 if it's a new failure vs baseline, 1.0 if persistent.
Top-N by `risk` drive the Overview "Top risks" (`04 F1`).

## 7. Baseline, comparability & lineage

- **"Previous"/baseline** = the most recent **completed, comparable** run, where comparable = same `projectId` + `lineage.framework` + `profile` + `environment` (and same `testPlanId`/`suiteSetHash` when present). If none, fall back to previous `seq` **with a visible caveat badge**.
- **Comparison set (2–5):** any runs the user picks; a **comparability badge** flags when selected runs differ in framework/profile/environment or when their test sets only partially overlap (**partial coverage**).
- **Pass-rate basis label:** always state whether a rate/diff is over the **full run** or **shared tests only** (when runs don't fully overlap).
- `suiteSetHash` = hash of the sorted set of `testId`s intended to run, so lineage detects when the suite changed between runs (avoids blaming added/removed on a suite edit).

## 8. Quality gates (safe rule schema — no expression evaluation)

A gate rule is structured data:
```json
{ "id": "min-pass-rate", "metric": "passRate", "operator": ">=",
  "threshold": 0.95, "severity": "fail", "scope": "run" }
```
- `metric` ∈ a fixed allowlist: `passRate`, `flakyRate`, `counts.failed`, `counts.broken`, `counts.blocked`, `healthScore`, `new.productBug`, `new.failures`, `duration.p95`, … (extensible in code, not by user strings).
- `operator` ∈ `>= | <= | > | < | == | !=`. `scope` ∈ `run | vs-baseline`. `severity` ∈ `warn | fail`.
- Run **verdict** = worst rule result; **known/muted excluded** from fail by default (configurable per rule).
- **No `eval`, no arbitrary expressions** — the evaluator maps `metric`→value and applies `operator`.

**Default gate set (safe, configurable):**
```json
[ { "id":"no-broken","metric":"counts.broken","operator":"==","threshold":0,"severity":"fail","scope":"run" },
  { "id":"no-blocked","metric":"counts.blocked","operator":"==","threshold":0,"severity":"fail","scope":"run" },
  { "id":"min-pass-rate","metric":"passRate","operator":">=","threshold":0.95,"severity":"fail","scope":"run" },
  { "id":"no-new-product-bugs","metric":"new.productBug","operator":"==","threshold":0,"severity":"fail","scope":"vs-baseline" },
  { "id":"max-flaky","metric":"flakyRate","operator":"<=","threshold":0.02,"severity":"warn","scope":"run" } ]
```

## 9. Metadata inference (low burden — DEC-13)

Labels (owner/severity/feature/component/team/case/issue/tags) are **optional**. Core derives them, in priority order, from: explicit test metadata → **file-path mappings** (config: path glob → labels) → **pytest markers** → **tags** → **naming conventions** → **CI metadata**. A test with no metadata still reports fully (labels simply absent; lens nodes show "unlabelled"). The product must be useful with zero per-test annotation.

## 10. Boundary

Defaults and formulas here are the product definition of each signal. Their code, and any tuning UI, live in core; teams tune via `config.json`. Exact numeric defaults may be adjusted before build without changing the model.
