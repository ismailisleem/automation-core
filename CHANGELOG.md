# Changelog

## 0.13.1

### Test Lineage enhancements

- **Intersection-based trend** — the Lineage Pass Rate Trend now measures pass
  rate over the lineage's *common core* (the tests present in every run of the
  lineage), labelled "over N shared tests", so a partial run no longer shows a
  misleadingly high full-run rate against full runs. The full per-run rate stays
  available in the point tooltip, and partial-coverage runs keep their marker.
- **Click-through trend points** — each trend point links to that run's report
  overview via a safe relative path, as a keyboard-navigable SVG link with an
  accessible label (run id, date, pass-rate basis). The current run's point
  stays on its own overview, and points with an unknown run path degrade to a
  non-navigating hover only.

## 0.13.0

### Added — Test Lineage

Group and compare runs by the set of test cases they actually executed, so a
run's trend and any side-by-side comparison stay apples-to-apples instead of
mixing unrelated suites.

- **Lineage model** (`reporting/lineage.py`): a run's identity is the set of its
  fully-qualified test ids; runs join the same lineage by containment
  (`|A∩B| / min(|A|,|B|)` ≥ 0.6, configurable), so a smoke subset of a full run
  is recognised as the same suite. Pairwise comparison is over the exact shared
  intersection. Pure, deterministic, fixture-tested.
- **Report Overview**: a Lineage Pass Rate Trend (pass rate across runs of the
  same test set, with full / partial-coverage / current-run markers), an
  expandable added/removed test diff, and a suite identity chip.
- **Compare page**: comparisons are scored over the tests every selected run
  shares — a "compared over N shared tests" note, a `PARTIAL n/m` badge on runs
  that covered fewer tests, pass rate over the shared intersection, and a
  "Delta over shared tests" table with a per-run added/removed Changes column.
  The delta table collapses to per-run cards on small screens.

The lineage signature and passing-id set are exposed in the report-data sidecar
and portfolio entries; history entries carry a stable per-test id for cross-run
matching.

### Notes

- Deferred: make the multi-run trend line itself intersection-based (tracked in
  `docs/lineage-model.md`).
