---
name: hubspot-cli
description: HubSpot CLI v8.0.0 commands, local development workflow, authentication, hs watch, hs upload, hs cms scaffolding, project deployment, and CI/CD. Use when setting up local development, deploying themes, or working with the HubSpot CLI tools.
---

# HubSpot CLI Reference

The HubSpot CLI (v8.0.0+) connects your local development environment to HubSpot. Requires **Node.js 20+**.

## Installation

```bash
# Install globally
npm install -g @hubspot/cli

# Or locally in project
npm install --save-dev @hubspot/cli
```

For local installation, prefix commands with `npx`:
```bash
npx hs upload src dest
```

## Initial Setup

CLI auth, Service Keys, Private App migration, and the 1Password CLI workflow are documented in the `hubspot-auth-and-secrets` skill. Load that skill when setting up a new client or onboarding a teammate. It covers per-client vault layout, `op run` patterns, and CI service accounts.

Quick reference for the impatient:

```bash
# After storing your PAK in 1Password under "<Client Vault>/HubSpot/pak"
echo "client-name" | hs account auth --pak="$(op read 'op://Acme Corp/HubSpot/pak')"
```

This creates global config at `~/.hscli/config.yml`.

**Important command distinctions:**
- `hs account auth` runs first-time setup and creates new config (use this for a new client)
- `hs auth` only works when config already exists (use this to add another account)

### Account Management

```bash
# List configured accounts
hs accounts list

# Switch default account
hs accounts use my-sandbox

# Add another account (config must already exist)
hs auth --pak="$(op read 'op://Acme Corp/HubSpot/pak')"
```

### Config File Locations

- **Global (recommended):** `~/.hscli/config.yml`
- **Project-level (legacy):** `./hubspot.config.yml`

When using `op run` with `.env.op`, the CLI picks up `HUBSPOT_PERSONAL_ACCESS_KEY` from the environment via `--use-env` and the config file is not strictly required. See `hubspot-auth-and-secrets` for the full pattern.

### Environment Variables

For CI/CD and automated workflows, use environment variables instead of config files:

| Variable | Description |
|----------|-------------|
| `HUBSPOT_ACCOUNT_ID` | Target account/portal ID |
| `HUBSPOT_PERSONAL_ACCESS_KEY` | Authentication key (PAK) |
| `HUBSPOT_SERVICE_KEY` | Service Key for direct API calls (not used by `hs` itself) |

**NOTE:** `HUBSPOT_PORTAL_ID` was removed in CLI v8.0.0. Use `HUBSPOT_ACCOUNT_ID` instead.

## Core Commands

### hs upload

Upload files to HubSpot. **Changes are live immediately.**

> **Before any `hs upload`**, run through the pre-flight checklist and diff-before-push protocol in `hubspot-safe-development`. The "live immediately" property means there is no soft delete or undo at the CLI level.

```bash
# Upload directory
hs upload src/theme my-theme

# Upload to specific account
hs upload src/theme my-theme --account=production

# Upload single file
hs upload src/theme/css/main.css my-theme/css/main.css

# Upload using environment variables (recommended for CI/CD)
hs upload src/theme my-theme --use-env
```

### hs watch

Watch for local changes and auto-upload.

```bash
# Watch a directory
hs watch src/theme my-theme

# Watch with specific account
hs watch src/theme my-theme --account=DEV

# Initial upload then watch (recommended)
hs watch src/theme my-theme --initial-upload

# Remove deleted files from HubSpot
hs watch src/theme my-theme --remove
```

**Important notes:**
- `hs watch` no longer uploads on start - use `--initial-upload` or run `hs upload` first
- Deleting files locally does NOT delete from HubSpot unless using `--remove`
- Renaming a folder creates a new folder; old folder remains in HubSpot

### hs cms

Create new assets from templates. Replaces the removed `hs create` command.

```bash
# Create a new module
hs cms module my-module

# Create a template
hs cms template my-template

# Create a theme (React-based)
hs cms react-theme my-theme

# Create a website theme (traditional)
hs cms website-theme my-theme
```

**NOTE:** `hs create` was removed in CLI v8.0.0. Use `hs cms` instead.

### Removed Commands (v8.0.0)

The following commands were removed and have no direct replacement:

| Removed Command | Alternative |
|----------------|-------------|
| `hs create` | `hs cms` |
| `hs fetch` | Download from Design Manager UI. **Always commit or stash local changes first** to avoid `hs fetch` overwriting WIP. See `hubspot-safe-development`. |
| `hs theme preview` | Use `hs watch --initial-upload` and preview on HubSpot directly |
| `hs init` | `hs account auth` |

## Theme Commands

### hs theme generate-selectors

Generate CSS selectors from fields.json.

```bash
hs theme generate-selectors my-theme
```

## Project Commands (React/Developer Projects)

> **As of April 14, 2026:** HubSpot Developer Platform `2026.03` is GA (Spring 2026 Spotlight). New projects use this platform version by default. See the `hubspot-react-themes` skill for project structure changes that came with the GA.

### hs project

Manage HubSpot developer projects.

```bash
# Initialize new project
hs project init

# Start development server
hs project dev

# Upload project
hs project upload

# Deploy project
hs project deploy

# List builds
hs project builds

# View logs
hs project logs
```

### Development Server

```bash
# Start dev server with auto-upload
hs project dev

# Specific account
hs project dev --account=sandbox

# Skip initial upload
hs project dev --skip-initial-upload
```

## File Operations

### hs mv

Move or rename files in HubSpot.

```bash
hs mv old-path new-path
```

### hs remove

Delete files from HubSpot.

```bash
hs remove my-theme/old-file.css

# Remove directory
hs remove my-theme/old-folder
```

### hs filemanager upload

Upload to File Manager.

```bash
hs filemanager upload ./image.png /my-folder/image.png
```

## Secrets & Environment

### hs secrets

Manage secrets for serverless functions.

```bash
# Add secret
hs secrets add MY_API_KEY

# List secrets
hs secrets list

# Delete secret
hs secrets delete MY_API_KEY

# Update secret
hs secrets update MY_API_KEY
```

## HubDB Commands

### hs hubdb

Manage HubDB tables.

```bash
# Create table from JSON
hs hubdb create my-table.json

# Fetch table
hs hubdb fetch TABLE_ID output.json

# Clear table rows
hs hubdb clear TABLE_ID

# Delete table
hs hubdb delete TABLE_ID
```

## Sandboxes

```bash
# Create sandbox
hs sandbox create

# List sandboxes
hs sandbox list

# Delete sandbox
hs sandbox delete SANDBOX_ID
```

## Useful Options

### Global Flags

| Flag | Description |
|------|-------------|
| `--account=NAME` | Use specific account |
| `--use-env` | Use environment variables for auth (recommended for CI/CD) |
| `--help` | Show help |
| `--debug` | Enable debug output |

## Deployment Best Practices

### Always Use `--use-env` for Automated Deployments

Config file-based approaches frequently fail in CI/CD contexts. Always use `--use-env` with environment variables:

```bash
# Set environment variables
export HUBSPOT_ACCOUNT_ID=12345678
export HUBSPOT_PERSONAL_ACCESS_KEY=your_key_here

# Deploy with --use-env
hs upload src/theme my-theme --use-env
```

### Local Preview Workflow

Since `hs theme preview` was removed in v8, the recommended preview workflow is:

```bash
# Upload and watch for changes
hs watch src/theme my-theme --initial-upload --account=sandbox

# Then preview at:
# https://app.hubspot.com/content/PORTAL_ID/preview
```

## .hsignore File

Exclude files from upload (like .gitignore):

```
# .hsignore
node_modules/
.git/
*.log
*.map
.DS_Store
*.scss
src/
README.md
package.json
package-lock.json
webpack.config.js
tsconfig.json
```

## hubspot.config.yml Reference

```yaml
# Default account for commands
defaultPortal: development

# Account configurations
portals:
  - name: development
    portalId: 12345678
    authType: personalaccesskey
    personalAccessKey: YOUR_KEY

  - name: staging
    portalId: 23456789
    authType: personalaccesskey
    personalAccessKey: STAGING_KEY

  - name: production
    portalId: 34567890
    authType: personalaccesskey
    personalAccessKey: PROD_KEY

# Allow usage tracking (optional)
allowUsageTracking: true
```

### Auth Types

- `personalaccesskey` - Personal access key (recommended)
- `oauth2` - OAuth 2.0 flow

## Project Versions

| Version | Status |
|---------|--------|
| 2023.2 | **REMOVED** (support ended Oct 2025) |
| 2025.1 | Deprecated (support ends June 2026) |
| 2025.2 | Stable |
| 2026.03 | **GA** (Spring 2026 Spotlight, April 14, 2026). Brings serverless functions back. |

Use **2026.03** for new projects. If you have a project already on 2025.2, plan a migration when convenient (no urgency until June 2026 when 2025.1 support ends).

## Serverless Functions

Serverless functions are **NOT available** in v2025.2 projects. They returned in **v2026.03** (GA April 14, 2026).

If you are still on 2025.2 or earlier, host functions externally (e.g., Google Cloud Functions, AWS Lambda) and use `hubspot.fetch()` for API calls from HubSpot until you migrate.

## Development Workflow

### Traditional Theme

```bash
# 1. Authenticate (PAK injected from 1Password; see hubspot-auth-and-secrets)
echo "my-account" | hs account auth --pak="$(op read 'op://Acme Corp/HubSpot/pak')"

# 2. Create theme scaffold
hs cms website-theme my-theme

# 3. Watch for changes (uploads on start)
hs watch my-theme my-theme --initial-upload

# 4. Or manual upload
hs upload my-theme my-theme
```

### React Theme

```bash
# 1. Create project
npx @hubspot/create-cms-theme@latest

# 2. Configure account
hs account auth

# 3. Start development
hs project dev

# 4. Deploy
hs project upload
```

### Multi-Environment Deployment

```bash
# Deploy to staging
hs upload src/theme my-theme --account=staging

# Test on staging...

# Deploy to production
hs upload src/theme my-theme --account=production
```

## Troubleshooting

### Common Issues

**"The directory was not uploaded"**
- `hs watch` no longer uploads initially
- Run `hs upload` first or use `--initial-upload`

**Files not syncing**
- Check `.hsignore` isn't excluding files
- Verify correct account with `hs accounts list`

**Authentication failed**
- See `hubspot-auth-and-secrets` for the full auth workflow.
- Quick check: `op read 'op://<Client Vault>/HubSpot/pak'` returns the PAK.
- If using `--use-env`, confirm `HUBSPOT_PERSONAL_ACCESS_KEY` is set in the environment (e.g., via `op run --env-file=.env.op`).
- If the key was auto-deactivated by HubSpot's exposed-token monitoring, see the "Auto-Deactivation of Exposed Tokens" section in `hubspot-auth-and-secrets`.

**Rate limiting**
- Slow down watch frequency
- Batch uploads instead of individual files

**`--use-env` not working**
- Verify both `HUBSPOT_ACCOUNT_ID` and `HUBSPOT_PERSONAL_ACCESS_KEY` are set
- The old `HUBSPOT_PORTAL_ID` env var no longer works in v8

## Command Quick Reference

| Task | Command |
|------|---------|
| **New account auth** | `echo "name" \| hs account auth --pak='KEY'` |
| Add account (existing config) | `hs auth --pak='KEY'` |
| Upload | `hs upload src dest` |
| Upload (CI/CD) | `hs upload src dest --use-env` |
| Watch | `hs watch src dest --initial-upload` |
| Create module | `hs cms module name` |
| Create theme | `hs cms website-theme name` |
| Create React theme | `hs cms react-theme name` |
| React project dev | `hs project dev` |
| Switch account | `hs accounts use name` |
| List accounts | `hs accounts list` |

## Official Documentation

- [CLI Reference](https://developers.hubspot.com/docs/developer-tooling/local-development/hubspot-cli/reference)
- [Local Development Guide](https://developers.hubspot.com/docs/cms/developer-reference/local-development-cli)
- [Project CLI](https://developers.hubspot.com/docs/platform/project-cli-commands)
- [CLI v8.0.0 Changelog](https://developers.hubspot.com/changelog/introducing-hubspot-cli-v8.0.0)
