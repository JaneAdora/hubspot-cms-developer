# Public Repo Prep Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Take the existing `hubspot-cms-developer` plugin (currently at `~/.claude/plugins/local-marketplace/hubspot-cms-developer/`, not in git) and publish it as a public GitHub repo at `JaneAdora/hubspot-cms-developer`, set up as both a Claude Code plugin and a single-plugin marketplace named `rep-cap-plugins`.

**Architecture:** Add five root-level files (`marketplace.json`, `README.md`, `LICENSE`, `.gitignore`, `CHANGELOG.md`), scrub three internal-brand mentions, then `git init` and push. The plugin's existing structure (`commands/`, `skills/`, `docs/`, `.claude-plugin/plugin.json`) is unchanged.

**Tech Stack:** Markdown, JSON, git, gh CLI. No build, no tests beyond manual verification.

**Verification approach:** Manual. After push, view the repo on github.com and confirm README renders, LICENSE is detected as MIT, all files committed.

---

## File Structure

**New files at plugin root:**
- `.claude-plugin/marketplace.json` (single-plugin marketplace catalog)
- `README.md` (public entry point)
- `LICENSE` (MIT, copyright "Rep Cap")
- `.gitignore` (standard exclusions)
- `CHANGELOG.md` (Keep a Changelog format, starting with 2.1.0)

**Files modified (sensitive scrub):**
- `skills/hubspot-auth-and-secrets/SKILL.md` (one line)
- `docs/superpowers/specs/2026-05-06-2026-docs-update-design.md` (one line)
- `docs/superpowers/plans/2026-05-06-2026-docs-update.md` (one line)

**Unchanged:** Everything else (`.claude-plugin/plugin.json`, `commands/`, all 11 skills, all spec/plan docs except the brand mentions).

---

## Task 1: Sensitive content scrub (internal brand mention → generic)

**Files:**
- Modify: `skills/hubspot-auth-and-secrets/SKILL.md`
- Modify: `docs/superpowers/specs/2026-05-06-2026-docs-update-design.md`
- Modify: `docs/superpowers/plans/2026-05-06-2026-docs-update.md`

Three source files reference an internal company name when explaining the per-client vault pattern. The scrub replaces each with audience-neutral phrasing.

- [ ] **Step 1: Update the auth skill**

In `skills/hubspot-auth-and-secrets/SKILL.md`, locate the paragraph that begins "Each client gets its own 1Password vault." and replace the company-name reference with "a common per-client secrets pattern used by agencies." The rest of the paragraph (the description of who can read which keys) is unchanged.

- [ ] **Step 2: Update the docs-update spec**

In `docs/superpowers/specs/2026-05-06-2026-docs-update-design.md`, locate the bullet point under "1Password integration" that ends with a reference to an existing internal pattern. Replace "matches the existing [brand] pattern for other client secrets" with "matches a common per-client secrets pattern."

- [ ] **Step 3: Update the docs-update plan**

In `docs/superpowers/plans/2026-05-06-2026-docs-update.md`, locate the paragraph that begins "Each client gets its own 1Password vault." and replace the company-name reference with "a common per-client secrets pattern." The rest of the paragraph is unchanged.

- [ ] **Step 4: Verify no internal brand mentions remain in any plugin file**

Run from the plugin root, replacing `<INTERNAL_BRAND>` with the actual one-word string being scrubbed:

```bash
grep -rn "<INTERNAL_BRAND>" . 2>/dev/null
```

Expected: zero output (no matches). If any line still appears, fix it before continuing.

---

## Task 2: Create marketplace.json

**Files:**
- Create: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Write the file**

Create `.claude-plugin/marketplace.json` with:

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

- [ ] **Step 2: Validate JSON**

```bash
python3 -c "import json; d=json.load(open('.claude-plugin/marketplace.json')); assert d['name']=='rep-cap-plugins'; assert d['plugins'][0]['source']=='./'; print('OK')"
```

Expected: `OK`

---

## Task 3: Create README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write the README**

Create `README.md` with:

```markdown
# HubSpot CMS Developer (Claude Code Plugin)

Opinionated guidance for HubSpot CMS development with Service Keys, the 1Password CLI, and CI/CD. Built for agencies that manage CMS work across multiple HubSpot portals.

## What's in here

- **11 skills** covering CLI, HubL, themes, modules, HubDB, forms, React themes, CI/CD, MCP server integration, and auth/secrets management
- **4 slash commands** for scaffolding themes, templates, and modules
- **Opinionated 1Password workflow** with per-client vault isolation

## Requirements

- **Claude Code** (latest)
- **HubSpot CLI v8.0.0+** (`npm install -g @hubspot/cli@latest`)
- **1Password CLI (`op`)**: required, not optional. See [1Password CLI install docs](https://developer.1password.com/docs/cli/get-started)
- **Node.js 20+**

## Install

### Option 1: Via Claude Code marketplace (recommended)

```
/plugin marketplace add JaneAdora/hubspot-cms-developer
/plugin install hubspot-cms-developer@rep-cap-plugins
```

### Option 2: Direct git clone

```bash
git clone https://github.com/JaneAdora/hubspot-cms-developer.git \
  ~/.claude/plugins/local-marketplace/hubspot-cms-developer
```

Then restart Claude Code so the plugin loads.

## Skills

| Skill | What it covers |
|---|---|
| [`hubspot-auth-and-secrets`](skills/hubspot-auth-and-secrets/SKILL.md) | Service Keys, CLI Personal Access Keys, Private App Token migration, 1Password vault patterns, CI service accounts |
| [`hubspot-cli`](skills/hubspot-cli/SKILL.md) | CLI v8.0.0 commands, project versions, deployment workflow |
| [`hubspot-hubl`](skills/hubspot-hubl/SKILL.md) | HubL templating syntax, filters, control flow, CLDR locale formatting |
| [`hubspot-templates`](skills/hubspot-templates/SKILL.md) | Page, blog, email, system templates and layouts |
| [`hubspot-theme-development`](skills/hubspot-theme-development/SKILL.md) | Theme structure, `theme.json`, style settings |
| [`hubspot-module-development`](skills/hubspot-module-development/SKILL.md) | Module fields, meta config, reusable component patterns |
| [`hubspot-react-themes`](skills/hubspot-react-themes/SKILL.md) | React + HubL hybrid themes, JS partials, TypeScript, webpack |
| [`hubspot-hubdb`](skills/hubspot-hubdb/SKILL.md) | Dynamic data tables, HubL queries, dynamic pages |
| [`hubspot-form-styling`](skills/hubspot-form-styling/SKILL.md) | CSS for embedded forms, modern form selectors |
| [`hubspot-ci-cd`](skills/hubspot-ci-cd/SKILL.md) | GitHub Actions deployment, 1Password-injected secrets, multi-environment |
| [`hubspot-mcp-server`](skills/hubspot-mcp-server/SKILL.md) | Local Developer MCP server (vs. the separate Remote CRM MCP) |

## Commands

| Command | What it does |
|---|---|
| `/new-theme` | Scaffold a new HubSpot theme |
| `/new-template` | Create a new template (page, blog, system) |
| `/new-module` | Create a new module with fields.json |
| `/deploy-theme` | Deploy a theme via the CLI |

## Auth and secrets

This plugin assumes credentials live in 1Password and are injected via the `op` CLI. Per-client vault model:

- One 1Password vault per client (e.g., `Acme Corp`)
- One item per vault, named `HubSpot`, with fields `pak`, `service-key-cms`, `service-key-crm`, `portal-id`
- Path pattern: `op://<Client Vault>/HubSpot/<field>`

Teammates get added to specific client vaults rather than a shared dev vault. This means access can be granted and revoked per client without touching other clients' credentials.

For the full workflow including Service Keys, Private App Token migration, and CI service accounts, see [`hubspot-auth-and-secrets`](skills/hubspot-auth-and-secrets/SKILL.md).

## Versioning

See [CHANGELOG.md](CHANGELOG.md).

## How this plugin evolved

Design specs and implementation plans live in [`docs/superpowers/`](docs/superpowers/) for transparency. They were generated using the [Superpowers](https://github.com/obra/superpowers) brainstorming and writing-plans skills.

## License

MIT. See [LICENSE](LICENSE).
```

- [ ] **Step 2: Verify README has no em dashes and is reasonable size**

```bash
grep -c "—" README.md
wc -l README.md
```

Expected: em dashes = 0, line count ~80 to 100.

---

## Task 4: Create LICENSE (MIT)

**Files:**
- Create: `LICENSE`

- [ ] **Step 1: Write the MIT license**

Create `LICENSE` with the standard MIT text and "Rep Cap" as copyright holder:

```
MIT License

Copyright (c) 2026 Rep Cap

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 2: Verify**

```bash
head -3 LICENSE
```

Expected first three lines:

```
MIT License

Copyright (c) 2026 Rep Cap
```

---

## Task 5: Create .gitignore

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: Write the file**

Create `.gitignore` with:

```
# OS
.DS_Store
Thumbs.db

# Editors
*.swp
*.swo
*~
.vscode/
.idea/

# Logs and temp
*.log
*.bak

# Node (in case future tooling)
node_modules/

# Local secrets (defense in depth; this plugin should never commit secrets)
.env
.env.local
*.key
*.pem
```

- [ ] **Step 2: Verify**

```bash
test -f .gitignore && echo "OK" || echo "MISSING"
```

Expected: `OK`

---

## Task 6: Create CHANGELOG.md

**Files:**
- Create: `CHANGELOG.md`

- [ ] **Step 1: Write the file**

Create `CHANGELOG.md` with:

```markdown
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
```

- [ ] **Step 2: Verify no em dashes**

```bash
grep -c "—" CHANGELOG.md
```

Expected: `0`

---

## Task 7: Initialize git and create initial commit

**Files:**
- (creates `.git/` directory in plugin root)

- [ ] **Step 1: Verify we're at the plugin root**

```bash
pwd
ls .claude-plugin/plugin.json README.md LICENSE .gitignore CHANGELOG.md .claude-plugin/marketplace.json
```

Expected: `pwd` shows `/home/jane/.claude/plugins/local-marketplace/hubspot-cms-developer`. All listed files exist.

- [ ] **Step 2: Initialize git**

```bash
git init -b main
```

Expected output: `Initialized empty Git repository in ...`

- [ ] **Step 3: Stage all files**

```bash
git add .
git status
```

Expected: all the new files (README, LICENSE, etc.) plus the existing plugin files appear under "Changes to be committed."

- [ ] **Step 4: One last sensitive check before commit**

```bash
git diff --cached | grep -E "CiR[a-zA-Z0-9]{20,}|/home/jane|<INTERNAL_BRAND>" | head -20
```

Expected: zero output. If anything appears, stop and address before committing.

- [ ] **Step 5: Create the initial commit**

```bash
git commit -m "$(cat <<'EOF'
Initial public release v2.1.0

Includes:
- 11 skills covering HubSpot CMS development end-to-end
- 4 slash commands for theme/template/module scaffolding
- Opinionated auth workflow using Service Keys and 1Password CLI
- Single-plugin marketplace at .claude-plugin/marketplace.json
- Design and implementation docs in docs/superpowers/

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

Expected: a commit created with the listed files.

- [ ] **Step 6: Confirm commit**

```bash
git log --oneline -1
```

Expected: one line showing the commit SHA and "Initial public release v2.1.0".

---

## Task 8: Create GitHub repo and push

**Files:**
- (no local files; creates remote repo)

- [ ] **Step 1: Verify gh CLI is authenticated**

```bash
gh auth status 2>&1 | head -5
```

Expected: shows logged-in account `JaneAdora`.

- [ ] **Step 2: Create the public repo and push**

```bash
gh repo create JaneAdora/hubspot-cms-developer \
  --public \
  --source=. \
  --description "HubSpot CMS development plugin for Claude Code, with opinionated Service Keys + 1Password workflow" \
  --push
```

Expected output: confirmation that the repo was created and the push succeeded. The command sets the remote and pushes in one step.

- [ ] **Step 3: Confirm the remote and push**

```bash
git remote -v
git log --oneline -1
gh repo view JaneAdora/hubspot-cms-developer --json url,visibility,description --jq '{url, visibility, description}'
```

Expected:
- Remote `origin` set to `https://github.com/JaneAdora/hubspot-cms-developer.git` (or `git@...`)
- Commit visible
- `gh repo view` shows visibility = `PUBLIC`, description matches

---

## Task 9: Final verification

**Files:**
- (none; manual verification on github.com)

- [ ] **Step 1: Open the repo in a browser**

```bash
gh repo view JaneAdora/hubspot-cms-developer --web
```

This opens https://github.com/JaneAdora/hubspot-cms-developer in the default browser.

- [ ] **Step 2: Manually verify on github.com**

Confirm:
- README.md renders cleanly (skill table, install instructions visible)
- LICENSE shows the GitHub badge "MIT" in the right sidebar
- Repository description is set
- File tree shows: `commands/`, `skills/`, `docs/`, `.claude-plugin/`, `README.md`, `LICENSE`, `.gitignore`, `CHANGELOG.md`
- No file contains real credentials

- [ ] **Step 3: Test the marketplace install path (optional, recommended)**

In a fresh test or by removing the local plugin first (only do this if you want to verify the public install works):

```bash
# This would test the install. Skip if you don't want to disrupt the local copy.
# /plugin marketplace add JaneAdora/hubspot-cms-developer
# /plugin install hubspot-cms-developer@rep-cap-plugins
```

If you skip this step, plan to verify with Abby or a teammate when you share the repo.

- [ ] **Step 4: Update memory or notes if anything surprising came up**

If the install path didn't work, the marketplace.json schema needed adjustment, or anything else was non-obvious, capture it as a memory or in CHANGELOG for future you.

---

## Out of scope

- `CONTRIBUTING.md`, issue templates, PR templates
- GitHub Actions for CI/linting
- `CODE_OF_CONDUCT.md`
- Adding more plugins to the marketplace
- Verifying the install with Abby (her actual onboarding happens after the repo is live)
- Restructuring plugin internals
- Migrating any client credentials (operational, not docs)

---

## Success criteria recap

- Repo is live at `https://github.com/JaneAdora/hubspot-cms-developer`
- README renders cleanly on github.com
- LICENSE recognized as MIT by GitHub
- Both install paths in the README are syntactically correct
- No internal brand mentions in published files
- No real credentials, paths, or emails in any committed file
- Plugin still loads from `~/.claude/plugins/local-marketplace/` (path unchanged)
