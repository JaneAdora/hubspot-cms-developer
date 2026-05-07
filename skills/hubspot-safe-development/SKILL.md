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

```bash
hs sandbox list --account=<client-account>
```

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
| Open a session | `hs accounts info` then `git status` then check today's backup |
| About to edit a file | Diff against fresh `hs fetch` first |
| About to push | Re-fetch, re-diff, then `hs upload --account=<name>` |
| About to fetch | Commit or stash local changes first |
| About to run blog migration | Snapshot blog metadata to spreadsheet first |
| Production deploy | Sandbox first if available, then promote |

## See also

- `hubspot-cli` for the underlying CLI commands referenced here
- `hubspot-ci-cd` for automated deploy patterns (which have their own safety story)
- `hubspot-auth-and-secrets` for credential management (auth incidents and content incidents share recovery patterns)
