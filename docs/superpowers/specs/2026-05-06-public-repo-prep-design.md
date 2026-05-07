# Public Repo Prep: hubspot-cms-developer Plugin

**Date:** 2026-05-06
**Plugin version:** 2.1.0 (no version bump for this work)
**Status:** Approved

## Goal

Take the `hubspot-cms-developer` plugin (currently at `~/.claude/plugins/local-marketplace/hubspot-cms-developer/`, not in git) and publish it as a public GitHub repository at `JaneAdora/hubspot-cms-developer`. Set it up as both a Claude Code plugin AND a single-plugin marketplace, so users can install via either `/plugin marketplace add` or direct git clone.

## Audience

Primary: Rep Cap teammates (Abby first). Secondary: anyone who finds the repo and wants HubSpot CMS development guidance for Claude Code.

## What gets created

### Root-level files

| File | Purpose |
|---|---|
| `.claude-plugin/marketplace.json` | Marketplace catalog. Lists the plugin with `source: "./"`. Marketplace `name`: `rep-cap-plugins`. |
| `README.md` | Public-facing entry point. Install instructions, skill catalog, requirements (1Password CLI), links. |
| `LICENSE` | MIT, copyright "Rep Cap" |
| `.gitignore` | Standard exclusions: `.DS_Store`, editor swap files, `node_modules/`, `*.log` |
| `CHANGELOG.md` | Keep a Changelog format. First entry: `## [2.1.0] - 2026-05-06`, summarizing the work just completed. Reference back to 2.0.0 baseline. |

### No changes to plugin internals

The plugin's existing structure (`.claude-plugin/plugin.json`, `commands/`, `skills/`, `docs/`) stays as-is, with one exception: scrub three internal brand mentions to keep the repo audience-neutral.

## Sensitive content scrub

Three places in the source files (one in `skills/hubspot-auth-and-secrets/SKILL.md`, one in the docs-update spec, one in the docs-update plan) reference an internal brand by name when describing the per-client vault pattern. The scrub replaces those references with audience-neutral phrasing such as "a common per-client secrets pattern used by agencies." See git history for the exact diffs.

Audit confirmed clean for: real API keys, `/home/jane/` paths, real emails, real portal IDs (placeholders only).

## Marketplace.json structure

```json
{
  "name": "rep-cap-plugins",
  "owner": {
    "name": "Rep Cap"
  },
  "description": "Rep Cap's open-source Claude Code plugins, starting with HubSpot CMS development",
  "plugins": [
    {
      "name": "hubspot-cms-developer",
      "source": "./",
      "description": "Expert guidance for HubSpot CMS development with CLI v8.0.0, HubL templating, theme development, module creation, React+HubL themes, CI/CD deployment, HubDB, MCP server integration, form styling, and an opinionated auth workflow using Service Keys plus the 1Password CLI for credential management."
    }
  ]
}
```

The plugin's existing manifest at `.claude-plugin/plugin.json` is unchanged.

## README structure

Sections in order:

1. **Title and tagline.** Title: "HubSpot CMS Developer (Claude Code Plugin)". Tagline: "Opinionated guidance for HubSpot CMS development with Service Keys, 1Password, and CI/CD."
2. **What's in here.** Three-line summary: 11 skills, 4 commands, 1Password-required auth workflow.
3. **Requirements.** Bulleted: Claude Code, HubSpot CLI v8.0.0+, **1Password CLI required (not optional)**, Node.js 20+.
4. **Install.** Two options:
   - Recommended: `/plugin marketplace add JaneAdora/hubspot-cms-developer` then `/plugin install hubspot-cms-developer@rep-cap-plugins`
   - Manual: clone into `~/.claude/plugins/local-marketplace/`
5. **Skills.** Table of 11 skills with one-line descriptions, linked to their `SKILL.md`.
6. **Commands.** Table of 4 slash commands with one-liners.
7. **Auth and secrets (the opinionated part).** Two paragraphs explaining the per-client 1Password vault model and pointing readers to `hubspot-auth-and-secrets`.
8. **Versioning.** Link to CHANGELOG.
9. **How this plugin evolved.** Link to `docs/superpowers/specs/` and `docs/superpowers/plans/` for transparency.
10. **License.** MIT.

## CHANGELOG.md structure

Keep a Changelog format. Initial content:

```markdown
# Changelog

All notable changes to this plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.1.0] - 2026-05-06

### Added
- New `hubspot-auth-and-secrets` skill consolidating Service Keys, CLI Personal Access Keys, Private App Token migration, and 1Password CLI workflow guidance.
- "Locale and Formatting (CLDR)" section in `hubspot-hubl` covering the April 2026 CLDR migration.
- 1Password-injected GitHub Actions workflow example in `hubspot-ci-cd`.
- Two-MCP-servers disambiguation block in `hubspot-mcp-server` (Local Developer MCP vs. Remote CRM MCP at mcp.hubspot.com).
- Date-based API versioning callout in `hubspot-hubdb`.
- New keywords on plugin manifest: `auth`, `service-keys`, `1password`.

### Changed
- `hubspot-cli` auth section slimmed to a pointer toward `hubspot-auth-and-secrets`.
- Project Versions table updated to reflect Developer Platform 2026.03 GA (April 14, 2026).
- `hubspot-ci-cd` "Getting Your Credentials" section restructured around the 1Password service account pattern.

### Documentation
- Added `docs/superpowers/specs/` and `docs/superpowers/plans/` capturing the design and implementation history of this update.

## [2.0.0] - Prior baseline

Initial public version of the plugin (pre-public-repo). Covered HubSpot CLI, HubL, themes, modules, React themes, HubDB, CI/CD, form styling, MCP server (Feb 2026 GA), and templates.
```

## .gitignore content

```
.DS_Store
*.swp
*.swo
*~
node_modules/
*.log
.vscode/
.idea/
```

## Git workflow

In order:

1. Apply the three brand-mention replacements (sensitive scrub)
2. Write `marketplace.json`, `README.md`, `LICENSE`, `.gitignore`, `CHANGELOG.md`
3. `git init` in the plugin root
4. `git add .` and `git commit -m "Initial public release v2.1.0"`
5. `gh repo create JaneAdora/hubspot-cms-developer --public --source=. --description "HubSpot CMS development plugin for Claude Code"`
6. `git push -u origin main`
7. Verify the repo is live and the README renders on github.com

## Out of scope

- Restructuring plugin internals (working file structure stays)
- Adding `CONTRIBUTING.md`, issue templates, PR templates, GitHub Actions for CI/linting
- Adding more plugins to the marketplace
- Anything that requires Abby's input (her actual onboarding happens after the repo is live)
- Marketplace publishing to a broader registry (Claude Code's official marketplace, etc.)

## Success criteria

- Repo is live at `https://github.com/JaneAdora/hubspot-cms-developer`
- README renders cleanly on github.com
- Both install paths in the README work end-to-end (verified manually after push)
- No internal brand mentions remain in the published files
- No sensitive data (real keys, paths, emails) in any committed file
- Plugin still loads correctly from `~/.claude/plugins/local-marketplace/` after `git init` (no broken paths)
- LICENSE file is recognized by GitHub as MIT
