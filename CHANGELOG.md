# Changelog

All notable changes to this project are documented in this file.

The project follows [Semantic Versioning](https://semver.org/).

## [2.0.0] - 2026-08-26

### Changed

- Recast the authenticated user as an independent peer Reviewer alongside
  Claude Reviewer and Codex Reviewer.
- Stop reading, reconciling, quoting, or relying on any other Review state,
  body, finding, approval, request, inline comment, or review thread.
- Replace the reviewer-record-derived reference with repository-owned
  `agentosnext-review-checks.md`.
- Reduce the decision set to exactly `APPROVE` and `REQUEST_CHANGES`.
- Treat missing evidence required for merge safety as a reasoned
  `REQUEST_CHANGES` instead of `HOLD`.
- Publish exactly one top-level formal Review with its reason body; remove
  `HOLD`, inline-comment, comment-only, and retrospective publication paths.
- Keep the two-phase full-SHA user confirmation and post-write readback gate.
- Update the copy-and-paste installation prompt to pin `v2.0.0`.

## [1.0.0] - 2026-08-26

Initial public release.

### Added

- Exact-head AgentOSNext Forgejo PR review workflow.
- Comments-first review and stale-review reconciliation.
- P0–P3 findings with `APPROVE`, `REQUEST_CHANGES`, and `HOLD` verdicts.
- Author self-check evidence matrix without reviewer-side test execution.
- PR1266–PR1290 review-pattern reference distilled from Codex/Claude reviews.
- Two-phase Forgejo publication protocol with full-SHA confirmation and readback.
- Copy-and-paste Codex bootstrap prompt pinned to `v1.0.0`.

[2.0.0]: https://github.com/CamelliaLilium/agentosnext-pr-merge-review/releases/tag/v2.0.0
[1.0.0]: https://github.com/CamelliaLilium/agentosnext-pr-merge-review/releases/tag/v1.0.0
