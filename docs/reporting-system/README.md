# Reporting System Requirements

This directory stores frozen requirements packages for the shared reporting system.

## Current Frozen Version

- Version: `v0.2.1`
- Path: `docs/reporting-system/v0.2.1/`
- Status: frozen requirements package for design and implementation planning

## Scope

The package defines the product requirements, data contracts, feature behavior, acceptance criteria, design system, security requirements, metrics, migration plan, and validation fixtures for the next reporting system.

Implementation details such as internal module layout, exact locking mechanism, CI workflow shape, and release PR breakdown belong in the implementation plan, but must satisfy this frozen package.

## Usage

Read the files in this order:

1. `00_README.md`
2. `01_PRD.md`
3. `02_DATA_MODEL.md`
4. `03_INFORMATION_ARCHITECTURE.md`
5. `04_FEATURE_SPEC.md`
6. `05_AC_MATRIX.md`
7. `06_DESIGN_SYSTEM.md`
8. `07_UI_SCREEN_PROMPTS.md`
9. `08_BUILD_HANDOFF.md`
10. `09_EXISTING_ECOSYSTEM_INTEGRATION.md`
11. `10_SECURITY_PRIVACY_REDACTION.md`
12. `11_METRICS_AND_DEFAULTS.md`
13. `12_MIGRATION_AND_ROLLOUT.md`
14. `13_FIXTURES_AND_VALIDATION.md`

## Change Control

Changes to frozen requirements should be made in a new versioned folder, not by editing an existing frozen version in place.
