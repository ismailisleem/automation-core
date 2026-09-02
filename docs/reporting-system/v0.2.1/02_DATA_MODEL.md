# 02 — Data Model & Storage Contract
### Strata — Automation Run Reporting System

> A **contract**. Core (`09`) writes exactly this; the viewer reads exactly this; frameworks map into it via the ingestion API. No field may be renamed, removed, or restructured without an approved edit here. Identifiers are strings unless noted; timestamps ISO-8601 UTC; durations integer ms. `schemaVersion` starts at `1`. A machine-readable JSON Schema for every file lives in core and is the validation source of truth.

---

## 1. On-disk store layout

`STORE_ROOT` default: `<framework-root>/.strata` (final path confirmed in `12` to align with current report paths).

```
.strata/
├── index.json                 # store manifest — light index of every run
├── config.json                # gates, custom lenses, theme default, saved filters, redaction config, thresholds
├── report/                    # the viewer app (static assets, vendored from core)
├── runs/
│   └── <runId>/
│       ├── run.json           # run metadata + summary + gate + lineage
│       ├── results/           # per-test results, sharded when large
│       │   ├── index.json     # light per-result rows (build trees/lists/facets without loading full results)
│       │   └── <resultId>.json
│       ├── steps/             # optional shard for large step trees
│       ├── assets/
│       │   ├── screenshots/
│       │   ├── video/
│       │   ├── traces/
│       │   ├── network/
│       │   └── logs/
│       └── .writing/          # temp area for atomic writes + per-worker shards (see §10)
```

Rules: `index.json` and `results/index.json` are lightweight; full result + media load on demand. Large runs shard `results/` (one file per result) and heavy `steps/`. Media are files referenced by relative path (never inlined, except `export`). Deletion is **manual only** (`strata delete <runId>` removes the folder + its index entry; nothing auto-prunes).

---

## 2. `index.json` (store manifest)

```jsonc
{
  "schemaVersion": 1,
  "product": "Strata", "toolVersion": "0.2.0",
  "createdAt": "…", "updatedAt": "…",
  "retention": { "policy": "manual" },
  "customLayers": ["team", "component"],
  "runs": [
    {
      "runId": "r_2026_09_02_1240_a1b2", "seq": 42, "label": "Nightly regression",
      "projectId": "checkout-web", "reportType": "web",   // web | api | mobile | combined
      "trigger": "ci", "startedAt": "…", "finishedAt": "…", "durationMs": 2400000,
      "status": "completed",                               // running | completed | aborted | error | partial
      "gate": { "verdict": "pass" },
      "pinned": false, "tags": ["nightly","release-4.3"],
      "env": { "envName": "staging", "appVersion": "4.3.0-rc1", "branch": "release/4.3", "commit": "9f2c1a", "ci": "github" },
      "frameworks": [{ "name": "web-automation-framework", "version": "x.y.z", "repo": "web", "adapter": "a.b.c" }],
      "lineage": { "projectId": "checkout-web", "testPlanId": "regression-full", "profile": "staging-chromium" },
      "counts": { "total": 812, "passed": 760, "failed": 22, "broken": 6, "skipped": 14, "known": 4, "blocked": 2, "flaky": 9 },
      "passRate": 0.936, "flakyRate": 0.011,
      "byPlatform": { "web": 812 }
    }
  ]
}
```
`counts` are by **effectiveStatus** (§7); `flaky` is an **overlay count** (tests where `flaky:true`), not part of the terminal totals.

---

## 3. `run.json` (per run)

```jsonc
{
  "schemaVersion": 1, "runId": "…", "seq": 42, "label": "…", "description": "…",
  "projectId": "checkout-web",
  "reportType": "web",                                   // web | api | mobile | combined
  "trigger": "ci", "status": "completed",
  "startedAt": "…", "finishedAt": "…", "durationMs": 2400000, "sumDurationMs": 8100000,
  "execution": { "workers": 6, "shards": 2, "parallel": true },
  "frameworks": [{ "name": "web-automation-framework", "version": "x.y.z", "repo": "web", "adapter": "a.b.c" }],
  "environment": {
    "envName": "staging", "appVersion": "4.3.0-rc1", "baseUrl": "https://staging.example.com",
    "os": "Ubuntu 24.04", "runtime": "python 3.12",
    "browsers": [{ "name": "chromium", "version": "127" }], "devices": [],
    "locale": "en-US", "viewport": "1280x720",
    "ci": { "provider": "github", "buildNumber": "1487", "buildUrl": "…", "branch": "…", "commit": "…", "pr": "…", "actor": "…" }
  },
  "lineage": {                                           // makes runs comparable (see 11 §baseline)
    "projectId": "checkout-web", "testPlanId": "regression-full",
    "suiteSetHash": "…",                                 // hash of the set of testIds intended to run
    "profile": "staging-chromium", "framework": "web", "environment": "staging"
  },
  "layers": { "available": ["suite","behavior","package","platform","tag","owner","severity","team"], "default": "suite" },
  "summary": {
    "counts": { "total": 812, "passed": 760, "failed": 22, "broken": 6, "skipped": 14, "known": 4, "blocked": 2, "flaky": 9 },
    "passRate": 0.936, "flakyRate": 0.011, "healthScore": 0.91,   // see 11
    "defectCounts": { "productBug": 12, "automationBug": 7, "system": 3, "environment": 2, "toInvestigate": 4, "knownIssue": 4, "noDefect": 0 },
    "duration": { "p50": 4200, "p95": 26000, "max": 91000, "slowestResultId": "t_…" },
    "byPlatform": { "web": { "total": 812, "passed": 760, "failed": 22, "broken": 6, "flaky": 9 } },
    "byLayer": { "suite": { "Checkout": { "total": 60, "failed": 8 } } }
  },
  "gate": {                                              // evaluated safe rules (11), stored result
    "verdict": "pass",
    "rules": [
      { "id": "min-pass-rate", "metric": "passRate", "operator": ">=", "threshold": 0.95, "scope": "run", "actual": 0.936, "result": "warn", "severity": "warn" },
      { "id": "no-new-product-bugs", "metric": "new.productBug", "operator": "==", "threshold": 0, "scope": "vs-baseline", "baselineRunId": "r_…", "actual": 3, "result": "fail", "severity": "fail" }
    ]
  },
  "redaction": { "applied": true, "profile": "default", "rulesVersion": "1" },   // see 10
  "attachments": [{ "id": "a_ci", "name": "CI log", "type": "text/plain", "path": "assets/logs/ci.log", "redacted": true }],
  "notes": []
}
```
For `reportType: "combined"` (orchestrated), `frameworks[]` lists every contributor and `summary.byPlatform` carries per-platform sub-summaries (`09 §4`).

---

## 4. `results/index.json` (light per-run index)

```jsonc
{
  "schemaVersion": 1, "runId": "…",
  "results": [{
    "resultId": "t_00a1", "testId": "h_9c1f…",
    "name": "Guest can complete checkout",
    "rawStatus": "failed", "effectiveStatus": "failed", "flaky": false,
    "platform": "web", "source": { "framework": "web", "repo": "web", "adapter": "a.b.c" },
    "durationMs": 12800, "defectType": "productBug",
    "labels": { "suite": ["Web","Commerce","Checkout"], "feature": "Checkout", "tags": ["smoke","p1"], "owner": "sara", "severity": "critical" },
    "hasEvidence": { "screenshot": true, "video": true, "trace": true, "network": true, "console": true },
    "attempts": 1
  }]
}
```

---

## 5. `<resultId>.json` (full result)

```jsonc
{
  "schemaVersion": 1, "resultId": "t_00a1", "testId": "h_9c1f…", "historyId": "h_9c1f…",
  "caseId": "TR-1042",
  "name": "Guest can complete checkout", "fullName": "Web > Commerce > Checkout > Guest can complete checkout",
  "description": "…",
  "platform": "web",                                    // web | api | mobile ( | desktop reserved )
  "source": { "framework": "web-automation-framework", "repo": "web", "adapter": "a.b.c" },
  "location": { "file": "tests/checkout/guest.py", "line": 42, "project": "chromium" },

  "labels": {                                            // ALL optional & inferable (see 11 §metadata inference)
    "suite": { "parent": "Web", "suite": "Commerce", "sub": "Checkout" },
    "behavior": { "epic": "Commerce", "feature": "Checkout", "story": "Guest checkout" },
    "package": "tests.checkout.guest", "platform": "web",
    "tags": ["smoke","p1","regression"], "owner": "sara", "severity": "critical",
    "custom": { "team": "payments", "component": "cart" }
  },
  "parameters": [{ "name": "currency", "value": "USD" }],

  "rawStatus": "failed",         // execution outcome of final attempt: passed | failed | broken | skipped | blocked
  "effectiveStatus": "failed",   // display/gate grouping after known/muted rules: passed | failed | broken | skipped | blocked | known
  "flaky": false,                // OVERLAY, independent of status
  "flakyReason": null,           // "retry-recovered" | "history-unstable" | "both" | null
  "muted": false,

  "defect": {
    "type": "productBug", "category": "Payment declined on valid card", "comment": "Gateway returns 402",
    "source": "manual", "assignedBy": "sara",
    "links": [{ "tracker": "jira", "key": "PAY-771", "url": "…", "status": "Open" }]
  },

  "failure": {
    "message": "expect(page).toHaveURL(/\\/success/) — timed out 10000ms",
    "type": "AssertionError",                            // assertion → failed; other error/timeout → broken
    "stack": "…", "diff": { "expected": "/success", "actual": "/checkout?error=402", "kind": "text" }
  },

  "timings": { "startedAt": "…", "stoppedAt": "…", "durationMs": 12800, "setupMs": 900, "teardownMs": 300 },

  "attempts": [{
    "index": 0, "rawStatus": "failed", "startedAt": "…", "durationMs": 12800,
    "failure": { "message": "…", "type": "AssertionError", "stack": "…" },
    "steps": [ /* §5.1 */ ], "evidence": { /* §5.2 */ }
  }],

  "history": { "last": ["passed","passed","failed","passed"], "computedByViewer": true },
  "evidence": { /* roll-up of final attempt, §5.2 */ },
  "redaction": { "applied": true, "fields": ["network.headers.authorization","request.body.$.card"] }  // see 10
}
```

### 5.1 Step tree (nestable)

```jsonc
{ "id":"s1", "name":"Click 'Place order'", "type":"action",   // action|assertion|hook|api|navigation|attach
  "status":"passed",                                           // passed|failed|broken|skipped
  "startedAt":"…", "durationMs":420, "params":[{ "name":"selector","value":"#place-order" }],
  "evidence": { "screenshot":"…", "snapshot":"…", "network":["…"], "console":[{ "level":"error","text":"…" }] },
  "subSteps": [] }
```

### 5.2 Evidence — polymorphic by platform

Common (all): `screenshots[]`, `video`, `logs[]`, `attachments[]` (each attachment carries `type`(MIME), `bytes`, `redacted`, `truncated`).
- **Web:** `trace{path,viewer}`, `domSnapshots[]`, `network[]` (HAR, redacted), `console[]`, `visualDiff{baseline,actual,diff,pixelDelta,layoutDiff}`.
- **API:** `requests[]{ method,url,reqHeaders,reqBody,status,resHeaders,resBody,latencyMs,assertions[] }` (headers/bodies redacted).
- **Mobile:** `device{name,os,osVersion,udid}`, `app{package,build}`, `screenRecording`, `screenshots[]`, `deviceLogs[]`, `contexts[]{native|webview}`, `appiumSessionId`.
- **Desktop (reserved/future):** shape defined but not produced in v1 (`09 §5`).

The viewer renders only present blocks; absent evidence shows an honest disabled state with a reason (`03 §8`, `07`).

---

## 6. Cross-run identity (`testId`/`historyId`)

`testId = hash(normalize(fullName) + "|" + platform + "|" + sortedParams)`. `historyId = testId` (parametrized families may collapse params; the mapper documents its choice). A native stable id (e.g. `caseId`) takes precedence. If no stable id is derivable, match on `normalize(fullName)+platform` and set `run.identity:"name-based"` so the viewer warns on added/removed. `normalize()` is one shared function in core, used identically on write and in compare (spec in `08`/`11`).

---

## 7. Status model (strict — resolves the terminal/overlay question)

Computed by **core** at finalize; the viewer never recomputes.

- **`rawStatus`** — what happened on the final attempt. Exactly five: `passed | failed | broken | skipped | blocked`.
  1. not executed by design → `skipped`; 2. prerequisite/setup failed so it couldn't run → `blocked`; 3. final attempt all-pass → `passed`; 4. an **assertion** failed → `failed`; 5. a non-assertion error/timeout → `broken`.
- **`effectiveStatus`** — display/gate grouping = `rawStatus`, except a **muted/known** rule (`11`) maps a `failed`/`broken` to `known`. Adds one value: `known`. This is what `counts`, charts, and gates group by (known excluded from gate-fail by default).
- **`flaky`** — a **boolean overlay**, independent of status. `true` when an earlier attempt was non-pass and the final attempt passed (`retry-recovered`), or when history is unstable (`history-unstable`), or `both`. A `passed` test can be `flaky:true`. Charts may show a flaky slice, but it is documented as an **overlay slice**, not a terminal status.

`failed` (assertion) vs `broken` (error) is a hard distinction and must never be conflated. There is **no** terminal `flaky` status anywhere.

---

## 8. Mutable metadata (edited after the run via serve)

Only these are writable post-run: `defect`, `muted`, `notes`, `index.runs[].{label,pinned,tags}`. Everything describing execution (attempts, steps, evidence, timings, raw failure, statuses) is immutable. The serve process owns writes; exported bundles are read-only.

---

## 9. Ingestion API (write side)

```
startRun(meta) → RunHandle              # creates runs/<id>/, run.json status:"running"
startTest(run, test) → TestHandle
step(test|step, step) → StepHandle      # nestable
attach(target, attachment) → relPath    # copies media into assets/ (redaction applied first, §11 + `10`)
endTest(test, outcome)                  # writes <resultId>.json + results/index row (atomic, §10)
endRun(run)                             # computes summary/gate/rollups/lineage; status:"completed"
abortRun(run, reason)                   # status:"aborted", partial preserved
```
Frameworks map native events → these calls (`09 §2`). Incremental writes during the run enable live view; `endRun` finalizes.

---

## 10. Store integrity & concurrency (required guarantees; mechanism owned by the technical design doc)

The store must stay valid under parallel workers and interrupted runs. **Required guarantees:**
- **Atomic writes:** every file is written to a temp path and `rename`d into place; readers never see partial files.
- **Single-writer discipline:** concurrent workers do not corrupt shared files — via a single writer/queue or per-worker shard files under `runs/<id>/.writing/` merged at `endRun`. No lost updates to `results/index.json`.
- **Run-id uniqueness:** ids are collision-free (timestamp + random/host component); a colliding id is rejected, not overwritten.
- **Partial/aborted/corrupt runs:** a run interrupted before `endRun` is readable as `status:"partial"`/`"aborted"` with whatever finalized; a corrupt result file is skipped with a flag, never crashing the viewer.
- **Recovery:** `strata rebuild-index` reconstructs `index.json` / `results/index.json` from the run folders.
- **Migration/compat:** `schemaVersion` on every file; core reads older-version runs (migrate-on-read; `strata migrate` for in-place). Migration has tests (`13`).

## 11. Redaction metadata & artifact safety (contract; policy in `10`)

- Redaction is applied **before** any secret/PII value is written to the store (at capture/ingestion). `run.redaction` and `result.redaction` record that it ran, the profile, and which fields were redacted (values themselves are never stored).
- Every attachment records `type` (validated MIME), `bytes`, `redacted`, and `truncated`. Oversized logs/bodies are truncated per `10` with a marker.
- Asset paths are validated to prevent traversal outside `runs/<id>/assets/` (`10`).
- No field ever stores a raw secret; redaction is default-on (`10`, DEC-10).

## 12. Export & portability

`strata export <runId>` → self-contained read-only single-file HTML (media embedded); `--data` → zip of raw JSON; viewer CSV/JSON/XLSX export of filtered sets/comparisons; safe-share/redacted export mode (`10`). Full export catalogue and attributes (editable/read-only/media/serve-free) in `04 F13`.

## 13. Versioning

`schemaVersion` per file; viewer reads `<=` its own; multiple tool versions may coexist in one store; migrate-on-read; breaking changes bump the version with a migration note in `12`.
