# 10 — Security, Privacy & Redaction
### Strata — Automation Run Reporting System

> The report captures API bodies, headers, screenshots, video, logs, device logs, DOM snapshots, and traces — which can contain secrets and PII. Redaction is **default-on** and applied **before anything is written to the store**. These are P0 business requirements (`05`), not optional hardening.

---

## 1. Principles

1. **Redact at capture, not at display.** Secrets must never reach `STORE_ROOT` in the clear. Frameworks apply redaction in their mappers using core's redaction engine before `attach`/`endTest` (`02 §9/§11`, `09 §2`).
2. **Safe by default, configurable by exception.** Sensible defaults protect users who configure nothing; allow/deny lists let teams tune.
3. **Local only.** No telemetry, no phone-home, no external calls (except the user's own AI provider if F14 is ever enabled — off by default).
4. **Least exposure on share.** Exports default to a safe-share posture and warn before including sensitive or heavy media.

## 2. Default redaction targets (on by default)

- **Headers:** `Authorization`, `Proxy-Authorization`, `Cookie`, `Set-Cookie`, `X-Api-Key`, `X-Auth-Token`, and any header matching `*token*`, `*secret*`, `*key*`, `*password*`.
- **Bearer/API tokens** anywhere in captured text (pattern-based), **passwords**, **OTP/2FA codes**, **credit-card numbers** (PAN, Luhn-checked), **CVV**, and common **secret env var names** (`*_TOKEN`, `*_SECRET`, `*_KEY`, `*_PASSWORD`, `AWS_*`, etc.).
- **Emails / phone numbers** where PII redaction is enabled (configurable; off vs on per profile).
- **Query params & form fields** matching the deny patterns above.
- Redacted values are replaced with a stable mask (e.g. `«redacted:authorization»`) so structure is preserved without the secret.

## 3. Configurable model (in `config.json`)

- **Allowlist / denylist** for: headers, JSON paths (request/response bodies), form fields, query params, and log line patterns (regex).
- **Capture modes** per platform: screenshots (`on|failure|off`), video (`on|retain-on-failure|off`), trace (`on-first-retry|retain-on-failure|off`), DOM snapshots, device logs — so teams can disable capture of sensitive surfaces entirely.
- **PII toggle** (emails/phones/custom patterns) per redaction **profile** (`default`, `strict`, `off-internal`).
- **Screenshot/video redaction:** optional selector-based masking regions (framework-supplied) for known-sensitive UI areas; when unavailable, teams can restrict capture to failure-only.
- Redaction rules are **versioned** (`run.redaction.rulesVersion`) so a run records which rule set produced it.

## 4. Serve & transport safety

- `strata serve` binds to **`127.0.0.1` (loopback) only** by default; no `0.0.0.0`, no LAN exposure without an explicit, documented opt-in flag and warning.
- No auth is implied by serve; it is a local viewer, not a shared service (consistent with no-accounts).
- No external resource loading at view time: fonts and assets are **bundled locally** (`06`, `08`), so an offline/air-gapped machine renders fully and nothing leaks via CDN requests.

## 5. Artifact safety

- **Path-traversal protection:** every asset path is validated to resolve inside `runs/<id>/assets/`; `..`, absolute paths, and symlinks escaping the run folder are rejected.
- **MIME/type validation:** attachments declare and are validated against an allowed type list; unknown/dangerous types are stored as inert downloads, never executed/inlined.
- **Size limits & truncation:** per-artifact max size; logs and request/response bodies exceeding the limit are truncated with a `truncated:true` marker and a note; the viewer never tries to render multi-hundred-MB blobs inline.
- **No active content:** exported/served HTML must not execute attachment content; SVGs/HTML attachments are sandboxed or rendered as inert.

## 6. Safe-share & export

- **Safe-share mode** (default for exports intended to leave the machine): applies the `strict` redaction profile, strips raw network bodies/headers beyond what's needed, and can **exclude media** or downscale it.
- **Explicit warning** before exporting a media-heavy or sensitive bundle (size + "may contain screenshots/logs" notice), with a one-click switch to safe-share.
- Each export records which redaction profile produced it; a safe-share export is labelled as redacted.
- Evidence-bundle zips honour the same redaction as the report.

## 7. Acceptance (P0 rows land in `05`)

Redaction default-on and applied pre-store; loopback-only serve; path-traversal rejected; MIME validated; oversized artifacts truncated; safe-share export redacts and warns; no telemetry/external calls at view time. Each has a fixture in `13`.

## 8. Boundary

This file specifies the security **requirements and defaults**. The exact redaction engine implementation, regex sets, and per-framework hooks live in the implementation team's technical design document and each framework repo, and must satisfy the requirements here.
