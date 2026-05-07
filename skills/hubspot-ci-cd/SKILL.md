---
name: hubspot-ci-cd
description: GitHub Actions auto-deploy workflow for HubSpot CMS themes, environment variables, multi-environment deployment, and CI/CD best practices. Use when setting up automated deployment pipelines or configuring GitHub Actions for HubSpot.
---

# HubSpot CI/CD with GitHub Actions

## Why Use CI/CD?

- Automated deployment on push (no manual `hs upload`)
- Multi-environment support (staging, production)
- Consistent, repeatable deployments
- No credentials in config files

## Environment Variables

HubSpot CLI v8.0.0 uses these environment variables with the `--use-env` flag:

| Variable | Description |
|----------|-------------|
| `HUBSPOT_ACCOUNT_ID` | Target HubSpot account (portal) ID |
| `HUBSPOT_PERSONAL_ACCESS_KEY` | Personal Access Key for CLI deployment (themes, modules, files) |
| `HUBSPOT_SERVICE_KEY` | Service Key for direct API calls in custom CI scripts (not used by `hs` itself). Public beta as of Feb 2026. |

**NOTE:** `HUBSPOT_PORTAL_ID` was removed in CLI v8.0.0. Use `HUBSPOT_ACCOUNT_ID`.

### Getting Your Credentials

See the `hubspot-auth-and-secrets` skill for the full credential workflow. For CI specifically:

1. Store the client's PAK and any Service Keys in a 1Password item named `HubSpot` inside the client's vault (e.g., `Acme Corp/HubSpot`).
2. Create a 1Password service account with read-only access scoped only to that client's vault.
3. Add the service account token to GitHub Secrets in the client's repo as `OP_SERVICE_ACCOUNT_TOKEN`.
4. Use `1password/load-secrets-action` in your workflow to inject credentials at deploy time. See "Recommended: 1Password-Injected Deploy" below.

This pattern means each client repo holds exactly one secret (`OP_SERVICE_ACCOUNT_TOKEN`); the actual HubSpot keys live in 1Password and rotate independently.

## Important: Always Use `--use-env`

Config file-based approaches (`hubspot.config.yml`) frequently fail in CI/CD contexts. Always use the `--use-env` flag:

```bash
# This works reliably in CI/CD
hs upload src/theme my-theme --use-env

# This often fails in CI/CD
hs upload src/theme my-theme  # relies on config file
```

## GitHub Actions Workflow

### Basic: Deploy on Push to Main

```yaml
# .github/workflows/deploy.yml
name: Deploy HubSpot Theme

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install HubSpot CLI
        run: npm install -g @hubspot/cli

      - name: Deploy theme
        env:
          HUBSPOT_ACCOUNT_ID: ${{ secrets.HUBSPOT_ACCOUNT_ID }}
          HUBSPOT_PERSONAL_ACCESS_KEY: ${{ secrets.HUBSPOT_PERSONAL_ACCESS_KEY }}
        run: hs upload src/theme my-theme --use-env
```

### Recommended: 1Password-Injected Deploy

This is the preferred pattern. GitHub Secrets stores only the 1Password service account token; the actual HubSpot credentials live in 1Password and can be rotated without touching GitHub.

```yaml
# .github/workflows/deploy.yml
name: Deploy to HubSpot

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install HubSpot CLI
        run: npm install -g @hubspot/cli@latest

      - name: Load secrets from 1Password
        uses: 1password/load-secrets-action@v2
        with:
          export-env: true
        env:
          OP_SERVICE_ACCOUNT_TOKEN: ${{ secrets.OP_SERVICE_ACCOUNT_TOKEN }}
          HUBSPOT_PERSONAL_ACCESS_KEY: op://Acme Corp/HubSpot/pak
          HUBSPOT_ACCOUNT_ID: op://Acme Corp/HubSpot/portal-id

      - name: Deploy theme
        run: hs upload src/theme my-theme --use-env
```

Replace `Acme Corp` with the client's actual 1Password vault name. The path pattern is always `op://<Client Vault>/HubSpot/<field>`.

### Multi-Environment: Staging + Production

```yaml
# .github/workflows/deploy.yml
name: Deploy HubSpot Theme

on:
  push:
    branches: [main, staging]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install HubSpot CLI
        run: npm install -g @hubspot/cli

      - name: Deploy to staging
        if: github.ref == 'refs/heads/staging'
        env:
          HUBSPOT_ACCOUNT_ID: ${{ secrets.HUBSPOT_STAGING_ACCOUNT_ID }}
          HUBSPOT_PERSONAL_ACCESS_KEY: ${{ secrets.HUBSPOT_STAGING_ACCESS_KEY }}
        run: hs upload src/theme my-theme --use-env

      - name: Deploy to production
        if: github.ref == 'refs/heads/main'
        env:
          HUBSPOT_ACCOUNT_ID: ${{ secrets.HUBSPOT_PROD_ACCOUNT_ID }}
          HUBSPOT_PERSONAL_ACCESS_KEY: ${{ secrets.HUBSPOT_PROD_ACCESS_KEY }}
        run: hs upload src/theme my-theme --use-env
```

### With Build Step (React Themes)

```yaml
name: Build and Deploy React Theme

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build theme
        run: npm run build

      - name: Install HubSpot CLI
        run: npm install -g @hubspot/cli

      - name: Deploy
        env:
          HUBSPOT_ACCOUNT_ID: ${{ secrets.HUBSPOT_ACCOUNT_ID }}
          HUBSPOT_PERSONAL_ACCESS_KEY: ${{ secrets.HUBSPOT_PERSONAL_ACCESS_KEY }}
        run: hs upload dist my-theme --use-env
```

## Setting Up GitHub Secrets

1. Go to your repo > Settings > Secrets and variables > Actions
2. Add these repository secrets:
   - `HUBSPOT_ACCOUNT_ID` (or `HUBSPOT_PROD_ACCOUNT_ID` / `HUBSPOT_STAGING_ACCOUNT_ID` for multi-env)
   - `HUBSPOT_PERSONAL_ACCESS_KEY` (or per-environment variants)

## Branch Protection

For production safety, configure branch protection on `main`:

1. Require pull request reviews before merging
2. Require status checks to pass (if you add linting/testing steps)
3. Prevent force pushes

This ensures all production deployments go through code review.

## Rollback

If a deployment breaks the site:

```bash
# Option 1: Revert the commit and push
git revert HEAD
git push origin main  # triggers auto-deploy of reverted code

# Option 2: Manual deploy of a known-good commit
git checkout GOOD_COMMIT_SHA -- src/theme
hs upload src/theme my-theme --use-env
```

## Troubleshooting

**Deployment fails with auth error:**
- Verify secrets are set correctly in GitHub
- For 1Password-injected workflows, confirm `OP_SERVICE_ACCOUNT_TOKEN` is current and the service account still has read access to the client vault
- Check the Personal Access Key hasn't expired
- Ensure required scopes (Content, Design Manager) are granted
- See the `hubspot-auth-and-secrets` skill for the full auth workflow

**Token suddenly stops working / "auth failed" after a recent commit:**

HubSpot auto-deactivates tokens it detects in public sources (GitHub, npm, common log services). Check:
- Was a `.env` or config file with the raw key accidentally committed?
- Is the key visible in any build log uploaded to a public CI?
- Did you paste the key into a chat or issue tracker?

Recovery: generate a new key, update the field in 1Password (the GitHub workflow will pick it up on next run), then redeploy. See the "Auto-Deactivation of Exposed Tokens" section in `hubspot-auth-and-secrets` for the full recovery checklist.

**`--use-env` not picking up variables:**
- Both `HUBSPOT_ACCOUNT_ID` and `HUBSPOT_PERSONAL_ACCESS_KEY` must be set
- For the 1Password pattern, confirm `load-secrets-action` ran successfully (check the step's logs for "Loaded ... secrets")
- Check for typos in secret names in the workflow YAML
- The old `HUBSPOT_PORTAL_ID` no longer works in CLI v8
