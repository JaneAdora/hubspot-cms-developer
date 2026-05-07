# Changelog

All notable changes to this plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.1.0] - 2026-05-06

First public release.

### Added
- New `hubspot-auth-and-secrets` skill consolidating Service Keys, CLI Personal Access Keys, Private App Token migration, and 1Password CLI workflow guidance.
- "Locale and Formatting (CLDR)" section in `hubspot-hubl` covering the April 2026 CLDR migration.
- 1Password-injected GitHub Actions workflow example in `hubspot-ci-cd`.
- Two-MCP-servers disambiguation block in `hubspot-mcp-server` (Local Developer MCP vs. Remote CRM MCP at mcp.hubspot.com).
- Date-based API versioning callout in `hubspot-hubdb`.
- New keywords on plugin manifest: `auth`, `service-keys`, `1password`.
- Single-plugin marketplace catalog at `.claude-plugin/marketplace.json` so the plugin is installable via `/plugin marketplace add`.

### Changed
- `hubspot-cli` auth section slimmed to a pointer toward `hubspot-auth-and-secrets`.
- Project Versions table in `hubspot-cli` updated to reflect Developer Platform 2026.03 GA (April 14, 2026).
- `hubspot-ci-cd` "Getting Your Credentials" section restructured around the 1Password service account pattern.

### Documentation
- Added `docs/superpowers/specs/` and `docs/superpowers/plans/` capturing the design and implementation history of this update.

## [2.0.0] - Prior baseline

Initial private version of the plugin (pre-public-repo). Covered HubSpot CLI, HubL, themes, modules, React themes, HubDB, CI/CD, form styling, MCP server (Feb 2026 GA), and templates.
