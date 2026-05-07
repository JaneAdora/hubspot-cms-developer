# Safe Development Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a `hubspot-safe-development` skill encoding pre-flight checks, sandbox-first policy, diff-before-push protocol, backup workflow, and incident recovery procedures, then cross-reference it from four existing skills, update README/CHANGELOG/manifest/.gitignore, commit, and push to the public repo as v2.2.0.

**Architecture:** One new skill file at `skills/hubspot-safe-development/SKILL.md`. Targeted edits to four existing skills to add cross-references. Root-level updates to README.md, CHANGELOG.md, .gitignore, and `.claude-plugin/plugin.json`. Commit and push.

**Tech Stack:** Markdown, JSON, git. No build, no tests beyond manual verification.

**Verification approach:** Manual. After each task, verify expected files exist, em dash count is zero, cross-references are in place. After push, view repo on github.com.

---

## File Structure

**New files:**
- `skills/hubspot-safe-development/SKILL.md`

**Files modified:**
- `skills/hubspot-cli/SKILL.md` (cross-ref pointers in `hs upload` and `hs fetch` sections)
- `skills/hubspot-ci-cd/SKILL.md` (one-line cross-ref at top)
- `skills/hubspot-react-themes/SKILL.md` (cross-ref in Development Workflow section)
- `skills/hubspot-auth-and-secrets/SKILL.md` (paragraph in Auto-Deactivation section)
- `README.md` (add safe-development row to Skills table; add Safety section)
- `CHANGELOG.md` (add `## [2.2.0] - 2026-05-07` entry)
- `.claude-plugin/plugin.json` (bump version, add keywords)
- `.gitignore` (add `_backup/` and `_diff/` patterns)

**Files unchanged:** Everything else. The other 6 skills, the 4 commands, the marketplace.json, the spec/plan docs.

---

## Task 1: Create the hubspot-safe-development skill

**Files:**
- Create: `skills/hubspot-safe-development/SKILL.md`

- [ ] **Step 1: Create the directory**

```bash
mkdir -p skills/hubspot-safe-development
```

- [ ] **Step 2: Write SKILL.md**

```markdown
---
name: hubspot-safe-development
description: Pre-flight safety checks, sandbox-first policy, backup workflow, diff-before-push protocol, and incident recovery for HubSpot CMS development. Use BEFORE any `hs upload`, `hs fetch`, `hs watch`, or production deploy. Required reading when an agent or teammate is about to touch a client portal.
---

# HubSpot Safe Development

## Core Principle

Every `hs upload`, `hs fetch`, and `hs watch` is irreversible without a backup. Build the safety net BEFORE you start, not after something breaks.

The HubSpot portal is a shared editing surface. Content editors, other developers, and integrations can all change files at any time. Your local copy is not authoritative. Treat the production portal as the source of truth and verify before you write.

## Pre-Work Checklist (Every Session)

Run this before opening any HubSpot file in your editor. If any answer surprises you, stop and fix it before touching code.

```bash
hs accounts info                                    # which account am I on?
git status --short                                  # clean working tree?
ls _backup/$(date +%Y-%m-%d)/ 2>/dev/null          # backup taken today?
```

If you don't see today's backup, take one before continuing (see "Backup Workflow" below).

## Sandbox-First Policy

**All initial work happens in a sandbox if one exists.** Promote to production only after review.

Check whether the client has sandboxes:

\`\`\`bash
hs sandbox list --account=<client-account>
\`\`\`

- **Has sandboxes (Enterprise+ tier):** Always start in sandbox. Use `--account=<sandbox-name>` for every command. Promote to prod via `hs upload` to the production account only after the change has been reviewed in sandbox.
- **No sandboxes:** Production is the only environment. Every command must use `--account=<prod-name>` explicitly, and a backup must be taken before any non-trivial work. Treat every change as a production change because it is one.

## Account Verification Rituals

Wrong-account pushes are one of the most common incidents. Three rituals:

1. **Always pass `--account=<name>` explicitly.** Never rely on `defaultPortal`. The default can shift if anyone runs `hs accounts use` between your sessions.

2. **Read the account name back before pressing enter.** When you type `hs upload src/theme my-theme --account=acme-prod`, look at the `--account=` value before hitting return. The brain glosses over typos in command lines; making yourself read it back catches them.

3. **Pre-flight `hs accounts info`.** This shows the account that would be used by default. If it doesn't match what you intended, run `hs accounts use <correct-name>` first. Do this even when you're using `--account=` flags, because it confirms your config has the account you think it does.

## Backup Workflow

Convention: `_backup/<YYYY-MM-DD>/` inside the project. Gitignored. Add `_backup/` to `.gitignore` if not already there.

### Take a backup before any non-trivial work

```bash
# Create today's backup directory
mkdir -p _backup/$(date +%Y-%m-%d)

# Fetch the current remote state of the assets you plan to touch
hs fetch <theme-or-module-path> _backup/$(date +%Y-%m-%d)/<theme-or-module-path> --account=<name>
```

### Recovery from backup

If something goes wrong, the backup is your fastest recovery path:

```bash
hs upload _backup/<date>/<path> <remote-path> --account=<name>
```

This restores the snapshot you took at the start of the session.

### Snapshot blog metadata before any blog-touching API call

Blog post metadata (titles, descriptions, custom fields) is NOT covered by Design Manager revision history. If you are about to run a script or migration that writes blog metadata, snapshot all current values to a CSV or spreadsheet first. The snapshot is your only recovery target if the migration goes sideways.

A real incident: during a blog migration, an agent in a hurry overwrote post metadata. Recovery only worked because the team had a spreadsheet snapshot from before the migration started.

## Diff-Before-Push Protocol

This is the heart of the safety workflow. Required for editable assets: HubL templates, modules, CSS, JS, JSON, fields configs. Not required for new files you are creating from scratch (where the remote should not have a copy yet).

### Procedure

1. **Before editing**, fetch the current remote version to a diff staging directory:
   ```bash
   mkdir -p _diff
   hs fetch <file-path> _diff/<file-path> --account=<name>
   ```

2. **Diff your local against the fresh remote:**
   ```bash
   diff <local-path> _diff/<file-path>
   ```
   (Or use a real diff tool like `delta`, `meld`, or your editor's diff view.)

3. **If the remote has changes you don't have, STOP.** Investigate. Likely an editor or another developer modified the file in HubSpot's UI since you last fetched. Merge those changes into your local copy before continuing. Do NOT proceed with your edits until your local matches the remote.

4. **Make your edits.**

5. **Before pushing**, re-fetch and re-diff:
   ```bash
   hs fetch <file-path> _diff/<file-path> --account=<name>
   diff <local-path> _diff/<file-path>
   ```
   Only your intentional changes should appear in the diff. If anything else does (someone edited again while you were working), stop and merge before pushing.

6. **Push.**

### Why this matters

A real incident, paraphrased: "We diffed the HTML template before editing. We forgot to diff the CSS file. The CSS had editor-side changes the local copy didn't have. Caught only because someone re-checked before push."

The lesson: **diff every editable asset you intend to touch, not just the entry-point file.** A template often comes with a CSS, a JS, and a JSON file. All four are editable surfaces. All four can have parallel edits. All four need diffing.

### What counts as an "editable asset"

Required to diff: HubL templates (`.html`), CSS files, JS files, module fields configs (`fields.json`), theme fields, module HTML, partials, modules' meta.json.

Not required to diff: brand-new files you are creating that don't exist on the remote yet.

## HubSpot's Built-in Revision History

A safety net most teams forget about: HubSpot's Design Manager has per-file revision history.

**Where:** HubSpot UI → Design Manager → select file → "Revisions" panel.

**What it covers:** Templates, modules, CSS, JS, partials, anything edited via Design Manager or pushed via the CLI.

**What it does NOT cover:** Blog post content and metadata. Use external snapshots (CSV/spreadsheet) for those.

**Limitations:** Timed retention (the exact window varies by tier; check at the time of need). Not a substitute for `_backup/` because retention may expire and because diffing remote-vs-local is harder through the UI than through fetched files.

Use this as a fallback recovery path alongside `_backup/`, not as a primary safety mechanism.

## Common Incidents and Recovery

### Wrong-account push

What happened: agent ran `hs upload ... --account=client-a-prod` when they meant `client-b-prod`. Files from one client are now live on another.

Recovery:
1. Stop. Don't run more commands until you understand the scope of damage.
2. From the affected (wrong-target) account, run `hs fetch` to a temp directory and inspect what got overwritten.
3. If you have `_backup/` from before the push for the wrong-target account, restore it via `hs upload _backup/<date>/<path> <remote-path> --account=<wrong-target>`.
4. Push the correct files to the correct account.
5. After recovery, document the timeline and check whether any content editors made changes during the incident window.

### `hs fetch` overwrote local WIP

What happened: dev ran `hs fetch` over uncommitted local changes. Editor view now shows remote content, dev's WIP is gone.

Recovery:
1. **Best case:** you committed or stashed before fetching. `git stash pop` or check `git reflog`.
2. **Backup case:** if you took a backup before the session, you can compare `_backup/<date>/` against the freshly fetched remote and reconstruct your edits.
3. **Worst case:** check editor undo history (often goes back further than you'd expect, especially in VS Code). After that, redo from memory.

Prevention: always commit (or at minimum stash) before running `hs fetch`. Make this a session-opening reflex.

### Blog metadata overwritten during migration

What happened: an agent in a hurry ran a blog migration script that overwrote metadata fields (titles, descriptions, custom fields) on production blog posts.

Recovery: external spreadsheet snapshot taken before migration started. Design Manager revision history does NOT cover blog post metadata.

Prevention: for any blog migration, snapshot all current values to a spreadsheet (or CSV via API) before the first API write. Treat that snapshot as the recovery target. Do not run migrations against production without the snapshot.

### Editor-vs-code CSS conflict

What happened: developer was editing template HTML and CSS locally. A content editor or another dev modified the CSS in HubSpot's UI in parallel. Developer pushed; editor's CSS changes got blown away. (Or, in the version that got caught: a re-diff before push surfaced the conflict.)

Recovery (if push already happened):
1. Check Design Manager revision history for the affected CSS file.
2. Restore the editor's version from the revision panel.
3. Manually merge with your changes.
4. Re-push.

Prevention: see the diff-before-push protocol. The "re-diff before push" step is what catches this class of incident.

## Anti-patterns

- **Running `hs watch` against a production account.** Don't. `hs watch` syncs on save, which means every keystroke that triggers a save can land in production. Only run `hs watch` against a sandbox account.
- **Relying on `defaultPortal`.** Always use `--account=<name>` explicitly.
- **Skipping the diff because "it's a small change."** The CSS incident was a small change. Small changes are when people skip safeguards. Don't.
- **Committing `_backup/` or `_diff/` to git.** They're intentionally gitignored. They are local working state, not source of truth.
- **Running migrations against production without a snapshot.** Always snapshot first. Always.

## Quick reference

| Action | Required steps |
|---|---|
| Open a session | `hs accounts info` → `git status` → check today's backup |
| About to edit a file | Diff against fresh `hs fetch` first |
| About to push | Re-fetch, re-diff, then `hs upload --account=<name>` |
| About to fetch | Commit or stash local changes first |
| About to run blog migration | Snapshot blog metadata to spreadsheet first |
| Production deploy | Sandbox first if available, then promote |

## See also

- `hubspot-cli` for the underlying CLI commands referenced here
- `hubspot-ci-cd` for automated deploy patterns (which have their own safety story)
- `hubspot-auth-and-secrets` for credential management (auth incidents and content incidents share recovery patterns)
```

- [ ] **Step 3: Verify the file is in place and has no em dashes**

```bash
ls -la skills/hubspot-safe-development/SKILL.md
grep -c "—" skills/hubspot-safe-development/SKILL.md
head -4 skills/hubspot-safe-development/SKILL.md
```

Expected: file exists; em dash count is 0; frontmatter starts with `---` and `name: hubspot-safe-development`.

---

## Task 2: Add cross-reference to hubspot-cli

**Files:**
- Modify: `skills/hubspot-cli/SKILL.md`

The plan is to add two pointers: one in the `hs upload` section (around line 76), one in the troubleshooting/best-practices area.

- [ ] **Step 1: Add a callout near the top of the `### hs upload` subsection**

Use Edit. Find:

```markdown
### hs upload

Upload files to HubSpot. **Changes are live immediately.**
```

Replace with:

```markdown
### hs upload

Upload files to HubSpot. **Changes are live immediately.**

> **Before any `hs upload`**, run through the pre-flight checklist and diff-before-push protocol in `hubspot-safe-development`. The "live immediately" property means there is no soft delete or undo at the CLI level.
```

- [ ] **Step 2: Add a similar callout for `hs fetch`**

Find the `hs fetch` row in the command table (around line 144):

```markdown
| `hs fetch` | Download from Design Manager UI, or use `hs watch` for sync workflow |
```

Replace with:

```markdown
| `hs fetch` | Download from Design Manager UI. **Always commit or stash local changes first** to avoid `hs fetch` overwriting WIP. See `hubspot-safe-development`. |
```

- [ ] **Step 3: Verify**

```bash
grep -c "hubspot-safe-development" skills/hubspot-cli/SKILL.md
grep -c "—" skills/hubspot-cli/SKILL.md
```

Expected: at least 2 references to the new skill; em dashes count is 0.

---

## Task 3: Add cross-reference to hubspot-ci-cd

**Files:**
- Modify: `skills/hubspot-ci-cd/SKILL.md`

- [ ] **Step 1: Add a one-line callout right after the H1**

Find:

```markdown
# HubSpot CI/CD with GitHub Actions

## Why Use CI/CD?
```

Replace with:

```markdown
# HubSpot CI/CD with GitHub Actions

> **For local-development safety patterns**, see the `hubspot-safe-development` skill (pre-flight checks, sandbox-first policy, backup and diff protocols). This skill covers automated CI/CD only.

## Why Use CI/CD?
```

- [ ] **Step 2: Verify**

```bash
grep -c "hubspot-safe-development" skills/hubspot-ci-cd/SKILL.md
grep -c "—" skills/hubspot-ci-cd/SKILL.md
```

Expected: at least 1 reference; em dashes count is 0.

---

## Task 4: Add cross-reference to hubspot-react-themes

**Files:**
- Modify: `skills/hubspot-react-themes/SKILL.md`

- [ ] **Step 1: Add a callout in the Development Workflow section**

Find:

```markdown
## Development Workflow
```

Replace with:

```markdown
## Development Workflow

> **Before pushing any React theme to a production account**, run through the safety protocols in `hubspot-safe-development`. React themes deploy via `hs project upload` which is just as irreversible as `hs upload`.
```

- [ ] **Step 2: Verify**

```bash
grep -c "hubspot-safe-development" skills/hubspot-react-themes/SKILL.md
grep -c "—" skills/hubspot-react-themes/SKILL.md
```

Expected: at least 1 reference; em dashes count is 0.

---

## Task 5: Add cross-reference to hubspot-auth-and-secrets

**Files:**
- Modify: `skills/hubspot-auth-and-secrets/SKILL.md`

- [ ] **Step 1: Add a paragraph in the Auto-Deactivation section**

Find:

```markdown
## Auto-Deactivation of Exposed Tokens

HubSpot now monitors public sources (GitHub, npm, common log services) and **automatically deactivates** any token it finds exposed. This applies to PAKs, Service Keys, and Private App tokens.
```

Replace with:

```markdown
## Auto-Deactivation of Exposed Tokens

HubSpot now monitors public sources (GitHub, npm, common log services) and **automatically deactivates** any token it finds exposed. This applies to PAKs, Service Keys, and Private App tokens.

> **Token-with-write-access incidents** (like blog metadata being overwritten by a script using a Private App Token) are best prevented by combining the auth practices in this skill with the backup and diff protocols in `hubspot-safe-development`. Auth practices keep the wrong people from getting tokens; safe-development practices keep the right people from causing accidents with the tokens they have.
```

- [ ] **Step 2: Verify**

```bash
grep -c "hubspot-safe-development" skills/hubspot-auth-and-secrets/SKILL.md
grep -c "—" skills/hubspot-auth-and-secrets/SKILL.md
```

Expected: at least 1 reference; em dashes count is 0.

---

## Task 6: Update README.md

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add the new skill to the Skills table**

Find this row in the table:

```markdown
| [`hubspot-cli`](skills/hubspot-cli/SKILL.md) | CLI v8.0.0 commands, project versions, deployment workflow |
```

Insert this row immediately after `hubspot-auth-and-secrets` (before `hubspot-cli`):

```markdown
| [`hubspot-safe-development`](skills/hubspot-safe-development/SKILL.md) | Pre-flight checks, sandbox-first policy, backup and diff protocols, incident recovery |
```

The result is the auth row, then the safe-development row, then the cli row.

- [ ] **Step 2: Update the "What's in here" line**

Find:

```markdown
- **11 skills** covering CLI, HubL, themes, modules, HubDB, forms, React themes, CI/CD, MCP server integration, and auth/secrets management
```

Replace with:

```markdown
- **12 skills** covering CLI, HubL, themes, modules, HubDB, forms, React themes, CI/CD, MCP server integration, auth/secrets management, and safe-development workflows
```

- [ ] **Step 3: Add a Safety section after "Auth and secrets"**

Find:

```markdown
For the full workflow including Service Keys, Private App Token migration, and CI service accounts, see [`hubspot-auth-and-secrets`](skills/hubspot-auth-and-secrets/SKILL.md).

## Versioning
```

Replace with:

```markdown
For the full workflow including Service Keys, Private App Token migration, and CI service accounts, see [`hubspot-auth-and-secrets`](skills/hubspot-auth-and-secrets/SKILL.md).

## Safety

This plugin assumes a sandbox-first development workflow with backups and diff-before-push protocols. Before any `hs upload`, `hs fetch`, or production deploy, work through the pre-flight checklist in [`hubspot-safe-development`](skills/hubspot-safe-development/SKILL.md). The skill covers account verification, backup conventions, the diff-before-push protocol (which prevents editor-vs-code conflicts), and recovery procedures for the most common incident classes.

## Versioning
```

- [ ] **Step 4: Verify**

```bash
grep -c "hubspot-safe-development" README.md
grep -c "12 skills" README.md
grep -c "—" README.md
```

Expected: at least 2 references to safe-development; "12 skills" appears once; em dashes count is 0.

---

## Task 7: Update CHANGELOG.md

**Files:**
- Modify: `CHANGELOG.md`

- [ ] **Step 1: Insert the 2.2.0 entry above the 2.1.0 entry**

Find:

```markdown
## [2.1.0] - 2026-05-06

First public release.
```

Replace with:

```markdown
## [2.2.0] - 2026-05-07

### Added
- New `hubspot-safe-development` skill covering pre-flight checks, sandbox-first policy, account verification rituals, backup workflow (`_backup/<YYYY-MM-DD>/` convention), diff-before-push protocol, HubSpot Design Manager revision history reference, and recovery procedures for four common incident classes (wrong-account push, `hs fetch` overwrite, blog metadata overwrite, editor-vs-code CSS conflict).
- Cross-references to `hubspot-safe-development` from `hubspot-cli`, `hubspot-ci-cd`, `hubspot-react-themes`, and `hubspot-auth-and-secrets`.
- README "Safety" section pointing readers to the new skill.
- `_backup/` and `_diff/` patterns added to `.gitignore` so backup snapshots and diff staging directories don't accidentally get committed.

### Changed
- README skill count updated from 11 to 12.
- Plugin keywords expanded with `safety`, `backup`, `diff`.

## [2.1.0] - 2026-05-06

First public release.
```

- [ ] **Step 2: Verify**

```bash
grep -c "## \[2.2.0\]" CHANGELOG.md
grep -c "—" CHANGELOG.md
```

Expected: 2.2.0 heading present; em dashes count is 0.

---

## Task 8: Update plugin.json manifest

**Files:**
- Modify: `.claude-plugin/plugin.json`

- [ ] **Step 1: Bump version and add keywords**

Read current contents:

```bash
cat .claude-plugin/plugin.json
```

Then write the new version:

```json
{
  "name": "hubspot-cms-developer",
  "version": "2.2.0",
  "description": "Expert guidance for HubSpot CMS development with CLI v8.0.0, HubL templating, theme development, module creation, React+HubL themes, CI/CD deployment, HubDB, MCP server integration, form styling, an opinionated auth workflow using Service Keys plus the 1Password CLI, and safe-development protocols (sandbox-first, backup, diff-before-push, incident recovery).",
  "author": {
    "name": "Jane"
  },
  "keywords": ["hubspot", "cms", "hubl", "theme", "module", "react", "forms", "ci-cd", "hubdb", "mcp-server", "deploy", "auth", "service-keys", "1password", "safety", "backup", "diff"],
  "commands": "./commands/",
  "skills": "./skills/"
}
```

- [ ] **Step 2: Validate**

```bash
python3 -c "import json,sys; d=json.load(open('.claude-plugin/plugin.json')); assert d['version']=='2.2.0', d['version']; assert 'safety' in d['keywords']; assert 'backup' in d['keywords']; assert 'diff' in d['keywords']; print('OK', d['version'], len(d['keywords']), 'keywords')"
```

Expected: `OK 2.2.0 17 keywords`

---

## Task 9: Update .gitignore

**Files:**
- Modify: `.gitignore`

- [ ] **Step 1: Append safe-development patterns**

Find the last line (`*.pem`) and add new patterns after the secrets block:

```bash
cat >> .gitignore <<'EOF'

# Safe-development working directories (per hubspot-safe-development skill)
_backup/
_diff/
EOF
```

- [ ] **Step 2: Verify**

```bash
grep "_backup/" .gitignore
grep "_diff/" .gitignore
```

Expected: both patterns present.

---

## Task 10: Update marketplace.json description

**Files:**
- Modify: `.claude-plugin/marketplace.json`

The marketplace.json's plugin description should match the plugin.json. Update it.

- [ ] **Step 1: Replace the plugin description**

Find:

```json
      "description": "Expert guidance for HubSpot CMS development with CLI v8.0.0, HubL templating, theme development, module creation, React+HubL themes, CI/CD deployment, HubDB, MCP server integration, form styling, and an opinionated auth workflow using Service Keys plus the 1Password CLI for credential management."
```

Replace with:

```json
      "description": "Expert guidance for HubSpot CMS development with CLI v8.0.0, HubL templating, theme development, module creation, React+HubL themes, CI/CD deployment, HubDB, MCP server integration, form styling, an opinionated auth workflow using Service Keys plus the 1Password CLI, and safe-development protocols (sandbox-first, backup, diff-before-push, incident recovery)."
```

- [ ] **Step 2: Validate JSON**

```bash
python3 -c "import json; d=json.load(open('.claude-plugin/marketplace.json')); assert 'safe-development' in d['plugins'][0]['description']; print('OK')"
```

Expected: `OK`

---

## Task 11: Final verification, commit, push

**Files:**
- (creates a new commit, pushes to remote)

- [ ] **Step 1: Confirm all expected files are touched**

```bash
git status --short
```

Expected output (order may vary):

```
 M .claude-plugin/marketplace.json
 M .claude-plugin/plugin.json
 M .gitignore
 M CHANGELOG.md
 M README.md
 M skills/hubspot-auth-and-secrets/SKILL.md
 M skills/hubspot-ci-cd/SKILL.md
 M skills/hubspot-cli/SKILL.md
 M skills/hubspot-react-themes/SKILL.md
?? docs/superpowers/plans/2026-05-07-safe-development.md
?? docs/superpowers/specs/2026-05-07-safe-development-design.md
?? skills/hubspot-safe-development/
```

- [ ] **Step 2: Cross-cutting em dash check on all changed files**

```bash
git diff HEAD --name-only | xargs grep -l "—" 2>/dev/null | grep "\.md$"
git status --porcelain | awk '/^\?\?/ {print $2}' | xargs -I{} find {} -name "*.md" 2>/dev/null | xargs grep -l "—" 2>/dev/null
```

Expected: empty output. If any files appear, fix them before committing.

- [ ] **Step 3: Final RepCap (one-word) audit**

```bash
git diff HEAD --name-only | xargs grep -l "RepCap" 2>/dev/null
```

Expected: empty output.

- [ ] **Step 4: Stage and commit**

```bash
git add .
git commit -m "$(cat <<'EOF'
v2.2.0: add hubspot-safe-development skill

Encodes pre-flight safety checks, sandbox-first policy, backup
workflow (_backup/<date>/), diff-before-push protocol, and recovery
procedures for four common incident classes:

- Wrong-account push
- hs fetch overwriting local WIP
- Blog metadata overwrite during migration
- Editor-vs-code CSS conflict

Cross-references added from hubspot-cli, hubspot-ci-cd,
hubspot-react-themes, and hubspot-auth-and-secrets. README and
CHANGELOG updated. Plugin version bumped to 2.2.0.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

- [ ] **Step 5: Push**

```bash
git push origin main
```

Expected: push succeeds, no errors about non-fast-forward or auth.

- [ ] **Step 6: Verify on github.com**

```bash
gh repo view JaneAdora/hubspot-cms-developer --json url,description --jq '{url, description}'
gh api repos/JaneAdora/hubspot-cms-developer/contents/skills/hubspot-safe-development/SKILL.md --jq '.name, .size'
```

Expected: repo description mentions safe-development; SKILL.md is reachable on remote with non-zero size.

---

## Out of scope

- Shell scripts or executables for backup/diff (the skill provides the commands; tooling can come later)
- Blog backup CLI (mentioned as a gap in the skill but not built here)
- Tagging the commit as `v2.2.0` (the version is in the manifest; tagging is optional and can be added later)
- Touching the 6 skills not in the modify list

## Success criteria recap

- New skill `hubspot-safe-development` exists with all sections from the spec
- Four existing skills cross-reference it
- README, CHANGELOG, plugin.json, marketplace.json, .gitignore all updated
- No em dashes in prose (verification commands excepted)
- No `RepCap` (one-word) mentions
- Commit pushed to public repo
- 2.2.0 visible at https://github.com/JaneAdora/hubspot-cms-developer
