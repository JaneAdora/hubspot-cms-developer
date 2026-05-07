# Safe Development Skill Design

**Date:** 2026-05-07
**Plugin version target:** 2.1.0 → 2.2.0
**Status:** Approved

## Goal

Add a `hubspot-safe-development` skill that encodes pre-flight checks, sandbox-first policy, diff-before-push protocol, backup workflow, and incident recovery procedures for HubSpot CMS development. Targeted at preventing the specific incident classes that have hit Rep Cap client work historically: wrong-account pushes, `hs fetch` over uncommitted local work, prod-when-meant-sandbox, and editor-vs-code conflicts.

## Why this is needed

Existing safety coverage in the plugin is thin and scattered. Recent incidents (paraphrased from author):

- Agents have pushed to the wrong client portal
- Agents have pushed to prod instead of sandbox
- During a blog migration, an agent in a hurry overwrote blog post metadata (titles, descriptions, custom fields). Recovered only because the team had a spreadsheet snapshot of the metadata from before the migration. HubSpot itself had no recovery path for blog meta.
- A CSS file was nearly overwritten yesterday because the team diffed the HTML template before editing but forgot to diff the CSS file. The CSS had editor-side changes the local copy didn't have. Caught only because someone re-checked before push.

The author has been able to recover from these because they maintain backups. The skill should make that practice explicit and repeatable for teammates.

## Audience

Primary: Rep Cap teammates who use Claude Code agents to work on HubSpot CMS files. Agents working on behalf of those teammates load this skill before destructive operations.

Secondary: anyone using the public plugin who wants to avoid the same incidents.

## Design

### New skill file

`skills/hubspot-safe-development/SKILL.md`

### Skill contents (in order)

1. **Frontmatter** with description that triggers on destructive-operation intent ("about to push", "before deploy", "before fetch", "production").

2. **Core principle.** Every `hs upload`, `hs fetch`, and `hs watch` is irreversible without a backup. Build the safety net before you start, not after something breaks.

3. **Pre-work checklist (every session).** Bash snippet:
   ```bash
   hs accounts info                                    # which account am I on?
   git status --short                                  # clean working tree?
   ls _backup/$(date +%Y-%m-%d)/ 2>/dev/null          # backup taken today?
   ```
   If any answer surprises you, stop and fix before touching code.

4. **Sandbox-first policy (new team policy).**
   - When the client has Enterprise+ (sandbox tiers): all initial work happens in sandbox. Promote to prod only after review.
   - When the client doesn't have a sandbox: prod is the only env, so every command needs explicit `--account=` and a backup taken first.
   - How to check: `hs sandbox list` shows whether sandboxes exist.

5. **Account verification rituals.**
   - Always pass `--account=<name>` explicitly. Never rely on `defaultPortal`.
   - Read the account name back before pressing enter on any `hs upload` or `hs fetch`.
   - Pre-flight: `hs accounts info` should show the account you intend to target. If it shows the wrong one, run `hs accounts use <correct-name>` first.

6. **Backup workflow.**
   - Convention: `_backup/<YYYY-MM-DD>/` inside the project (gitignored).
   - Before any non-trivial work, fetch the current remote state of the assets you plan to touch:
     ```bash
     mkdir -p _backup/$(date +%Y-%m-%d)
     hs fetch <theme-or-module-path> _backup/$(date +%Y-%m-%d)/<theme-or-module-path> --account=<name>
     ```
   - Recovery: `hs upload _backup/<date>/<path> <remote-path> --account=<name>` restores the snapshot.
   - The skill should remind users to add `_backup/` to `.gitignore` if not already present.

7. **Diff-before-push protocol (the heart).**
   - Required for editable assets: HubL templates, modules, CSS, JS, JSON, fields configs.
   - Not required for new files you are creating fresh (where remote should not have a copy).
   - Procedure:
     1. Before editing: `hs fetch <file> _diff/<file> --account=<name>`
     2. `diff <file> _diff/<file>` (or use a real diff tool)
     3. If the remote has changes you don't have, STOP. Investigate (likely an editor or another dev edited it). Merge those changes into your local before continuing.
     4. Make your edits.
     5. Before pushing: re-fetch and re-diff. Only your intentional changes should appear.
     6. Push.
   - Why: yesterday's CSS incident. The HTML template was diffed; the CSS was not. The CSS had editor-side changes the local didn't have.

8. **HubSpot's built-in revision history.**
   - Design Manager has per-file revisions. Most teams forget this exists.
   - Access: in HubSpot UI, Design Manager → select file → "Revisions" panel.
   - Limitations: timed retention; not available for blog post content (use blog backup tooling instead).
   - This is a free safety net for templates, CSS, JS, and module files. Use it as a recovery option alongside `_backup/`.

9. **Common incidents and recovery procedures.** One subsection per incident class, with the recovery steps:
   - **Wrong account push.** Recovery: redeploy from `_backup/` to the wrong-account, redeploy correct files to the right account.
   - **`hs fetch` overwrote local WIP.** Recovery: `git stash pop` if you stashed first; otherwise pull from `_backup/` if taken; otherwise check Design Manager revisions.
   - **Blog metadata overwritten during migration.** Recovery: external spreadsheet snapshot taken before migration started. Design Manager revision history does NOT cover blog post metadata, so for any blog migration, snapshot all current values to a spreadsheet (or CSV) before the first API write. Treat that snapshot as the recovery target if anything goes wrong.
   - **Editor-vs-code CSS conflict.** Recovery: see the diff-before-push protocol; this incident is what the protocol prevents.

10. **Anti-patterns.**
    - Running `hs watch` against a production account. The skill should explicitly say "don't."
    - Relying on `defaultPortal`. The skill should explicitly say "always use `--account=`."
    - Skipping the diff because "it's a small change." The CSS incident was a small change.

### Cross-skill updates

| Skill | Update |
|---|---|
| `hubspot-cli` | Add a "See `hubspot-safe-development` for the pre-flight checklist and diff-before-push protocol." pointer in the `hs upload` and `hs fetch` sections. |
| `hubspot-ci-cd` | Add a one-line pointer at the top: "For local-development safety patterns, see `hubspot-safe-development`. This skill covers CI/CD only." |
| `hubspot-react-themes` | Brief mention in the deploy section pointing to the safe-development skill. |
| `hubspot-auth-and-secrets` | Add a paragraph under "Auto-Deactivation of Exposed Tokens" noting that token-with-write-access incidents (like blog meta overwrites) are best prevented by combining the auth practices in this skill with the backup/diff protocols in `hubspot-safe-development`. |

### Plugin manifest

`.claude-plugin/plugin.json`:
- Bump version 2.1.0 to 2.2.0
- Update description to mention safe-development workflow
- Add `safety`, `backup`, `diff` to keywords

### Other root files

- **README.md:** Add `hubspot-safe-development` row to the Skills table. Add a brief "Safety" section under "Auth and secrets" that cross-references the new skill.
- **CHANGELOG.md:** Add `## [2.2.0]` entry summarizing the new skill.
- **.gitignore:** Add `_backup/` and `_diff/` patterns so the conventions don't accidentally commit.

## Out of scope

- Actual shell scripts or executables for backup/diff (the skill provides the commands; tooling can come later if needed)
- Blog backup tooling (mention as a gap, but not solving it here)
- Modifying any of the existing scrub-related work
- Changing the marketplace.json structure
- Republishing or version-bumping anything outside this plugin

## Success criteria

- New skill loads in Claude Code and is discoverable via the description
- Pre-flight checklist, sandbox-first policy, backup convention, diff-before-push protocol, and incident recovery are all covered with concrete commands
- The "yesterday's CSS incident" pattern is explicitly addressed in the diff-before-push section
- `_backup/` and `_diff/` are gitignored
- Plugin version bumped to 2.2.0; CHANGELOG and README updated
- Cross-references added to `hubspot-cli`, `hubspot-ci-cd`, `hubspot-react-themes`
- All changes pushed to the public repo with a tagged 2.2.0 commit
