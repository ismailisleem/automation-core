# 13 — Fixtures & Validation
### Strata — Automation Run Reporting System

> Concrete fixtures and the validation matrix that turn `05_AC_MATRIX.md` into runnable QA. Fixtures live in `automation-core` and are the shared substrate for engine tests, viewer tests, and visual/perf/a11y checks.

---

## 1. Required fixtures (sample stores)

Each is a real `STORE_ROOT` the engine and viewer run against:

1. **Empty store** — first-run onboarding path.
2. **One all-passing run.**
3. **Mixed-status run** — passed / failed / broken / skipped / blocked / known present, with correct `rawStatus` vs `effectiveStatus`.
4. **Retry-recovered flaky** — non-pass attempt then pass; `flaky:true, flakyReason:"retry-recovered"`, final `passed`.
5. **History-unstable flaky** — a `testId` alternating across ≥ 5 comparable runs.
6. **Two-run comparison** — contains a new-failure, a fixed, an added, and a removed test.
7. **Five-run comparison** — matrix with all change categories, incl. status-changed and duration-regressed.
8. **Cross-platform / combined run** — web + API + mobile results under one `reportType:"combined"` with `source` attribution and a Journey lens.
9. **Name-based identity degradation** — no stable ids; `identity:"name-based"` + warning.
10. **Long content** — very long test names, run ids, urls, tags, and stack traces (wrap/truncate behavior).
11. **Missing evidence** — tests lacking trace/video/visual-diff/mobile logs (disabled-state rendering).
12. **Corrupt/partial run** — an aborted run + a corrupt result file (skipped-with-flag, `rebuild-index` recovery).
13. **Large store** — a ≥ 5k-result run (performance) and many stored runs (history/trends scale).
14. **Media-heavy store** — large videos/traces (lazy-load + size/truncation behavior).
15. **Redaction/security fixture** — payloads containing tokens, cookies, PANs, PII, oversized bodies, and a path-traversal attempt; asserts redaction + rejection.
16. **Unlabelled tests** — zero per-test metadata (report still useful; inference from path/markers, `11 §9`).

## 2. Validation matrix (maps to `05`)

- **Schema:** every file in every fixture validates against the core JSON Schema.
- **Status logic:** raw vs effective vs flaky per fixture 3/4/5 (`02 §7`).
- **Identity & comparison completeness:** matched ∪ added ∪ removed = union; all change categories present (fixtures 6/7, `05 F5-10`).
- **Metrics:** health score, flaky grade, duration regression, baseline/lineage selection produce expected values (fixtures 3/5/6, `11`).
- **Gates:** default gate set yields expected verdicts (fixture 3); safe rule schema rejects malformed rules.
- **Security:** redaction applied pre-store; loopback binding; path-traversal rejected; MIME validated; oversized truncated; safe-share export redacts + warns (fixture 15, `10`).
- **Integrity:** atomic writes under simulated parallel workers; `rebuild-index` recovers fixture 12; migrate-on-read for an older-schema fixture.
- **Viewer functional:** URL-state round-trip; lens switching; faceted filter counts; evidence tabs per platform; comparison matrix; live streaming (simulated incremental writes).
- **Visual QA:** screenshots at **320 / 390 / tablet / desktop**, **light + dark**, for every screen in `07` (regression baseline).
- **Accessibility:** automated checks + a keyboard-only pass (focus, traps, live regions), reduced-motion.
- **Export:** round-trip for portable HTML / CSV / XLSX / JSON / evidence-zip; exported HTML opens without serve; read-only enforced.
- **Modes:** `serve` vs `file-open` behavior (`05 DATA-09`); auto-open-latest parity.
- **Performance:** interactive < 1.5s on the 5k fixture over serve; 5-run × 5k matrix at 60fps.
- **Browser matrix:** Chromium, Firefox, WebKit/Safari, Edge where available.

## 3. Fixture hygiene

Fixtures are deterministic (fixed ids, timestamps, seeds) so snapshots are stable. Media in fixtures are tiny stand-ins except the media-heavy fixture. Fixtures carry no real secrets — the redaction fixture uses synthetic secret-shaped values.

## 4. Boundary

This file defines **what must be validated and with which fixtures**. The test harness, CI wiring, and snapshot tooling live in the implementation team's technical design document and the repos.
