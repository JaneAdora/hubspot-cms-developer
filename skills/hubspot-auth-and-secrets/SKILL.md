---
name: hubspot-auth-and-secrets
description: HubSpot authentication (Service Keys, CLI Personal Access Keys, legacy Private App Tokens) and 1Password CLI workflow for storing and injecting client credentials. Use when setting up CLI access for a new client, migrating tokens, or sharing credentials with teammates. Assumes 1Password CLI is the secret store.
---

# HubSpot Auth and Secrets

This skill covers all HubSpot authentication flows and the 1Password CLI workflow used to store and inject those secrets. **1Password CLI (`op`) is required**, not optional. If `op` is not installed, install it before continuing (see "Installing the 1Password CLI" below).

## Decision Tree: Which Auth Do I Need?

| What you're doing | Auth to use |
|---|---|
| Running `hs auth`, `hs upload`, theme/module development | **CLI Personal Access Key (PAK)** |
| Server-side script that hits HubDB API or CRM API (no webhooks) | **Service Key** |
| Existing integration that uses webhooks | Stay on **Private App Access Token** for now (legacy, but still required for webhooks) |
| Building a public/multi-account app | OAuth (out of scope for this plugin) |

## CLI Personal Access Key (PAK)

The PAK is what `hs auth` uses. One PAK per HubSpot portal per developer.

### Create a PAK

1. Go to https://app.hubspot.com/l/personal-access-key (make sure you are logged into the correct portal)
2. Click "Generate personal access key"
3. Select scopes (CMS access, file manager, HubDB if you need it)
4. Copy the key (shown once)
5. Store it in 1Password (see "1Password Workflow" below)

### Authenticate the CLI

```bash
echo "client-name" | hs account auth --pak="$(op read 'op://Acme Corp/HubSpot/pak')"
```

Always inject via `op read` rather than pasting the key on the command line. The shell history and any active terminal recording will both capture pasted secrets.

## Service Keys

Public beta as of February 2026. Service Keys are the modern replacement for legacy Private App Access Tokens for system-to-system integrations.

### When to use a Service Key

- You need to call the HubSpot API from a server-side script (HubDB writes, CRM data sync, etc.)
- You do NOT need webhooks
- You want one credential per client per scope (e.g., CMS-only or CRM-only)

### When NOT to use a Service Key

- You need webhooks (use a legacy Private App for now)
- You are deploying themes via the CLI (use a PAK instead)
- You need OAuth-style multi-account access

### Create a Service Key

1. Open the HubSpot portal as a user with the scopes you want the key to have
2. Settings → Integrations → Service Keys (or follow the in-product onboarding)
3. Name the key after the scope, e.g. `hubdb-write-only` or `crm-read`
4. Select scopes (a key can only be granted scopes the creating user already has)
5. Copy the key (shown once)
6. Store it in 1Password

### Use a Service Key

Service Keys are sent as a Bearer token in the `Authorization` header:

```bash
curl -H "Authorization: Bearer $(op read 'op://Acme Corp/HubSpot/service-key-cms')" \
  "https://api.hubapi.com/cms/2026-03/hubdb/tables/12345/rows"
```

Note the date-based API version (`/2026-03/`) replacing the older `/v3/` style. See the `hubspot-hubdb` skill for HubDB-specific examples and the `hubspot-ci-cd` skill for CI/CD usage.

## Private App Access Tokens (Legacy)

HubSpot now treats UI-created Private App tokens as legacy. New work should use Service Keys. Existing integrations can migrate when convenient.

### Migrating from Private App Token to Service Key

1. Create a Service Key with the same scopes as the existing Private App
2. Update the integration's secret reference (in 1Password, in code, in env vars) to point at the new Service Key
3. Test the integration end-to-end
4. Revoke the old Private App Token in the HubSpot UI (Settings → Integrations → Private Apps → ⋯ → Deactivate)

### Exception: Webhooks

Service Keys do not support webhooks. If your integration uses webhooks, keep using the Private App Token until HubSpot adds webhook support to Service Keys.

## 1Password Workflow (Required)

This plugin assumes you store HubSpot credentials in 1Password and inject them into commands using the `op` CLI. Other secret stores are not documented here.

### Installing the 1Password CLI

If `op --version` errors with "command not found", install it. Installation steps drift across OSes and package managers, so we don't duplicate them here. Use one of:

- **Official docs:** https://developer.1password.com/docs/cli/get-started
- **Context7:** ask Claude to query Context7 for the current 1Password CLI install commands for your OS or package manager

After install, sign in:

```bash
op signin
```

Verify:

```bash
op whoami
```

If `op whoami` errors on macOS with the desktop app integration enabled, you may have hit a known bug in older `op` versions. Update to the latest CLI before debugging further.

### Vault Layout: One Vault Per Client

Each client gets its own 1Password vault. This matches a common per-client secrets pattern used by agencies and lets you grant teammates access on a per-client basis. Only people working on Acme can read Acme's HubSpot keys.

Inside each client vault, create a single item named `HubSpot` with one field per credential type:

```
Acme Corp/                       (1Password vault, named for the client)
  HubSpot/                       (item)
    pak                  (string, secret)         CLI Personal Access Key
    service-key-cms      (string, secret)         CMS-scoped Service Key
    service-key-crm      (string, secret)         CRM-scoped Service Key (only if needed)
    portal-id            (string, plain)          For reference
    notes                (notes, plain)           Scope list, who created the keys, when
```

The path pattern is always `op://<Client Vault>/HubSpot/<field>`. The item name stays `HubSpot` across all clients, so scripts and templates can be parameterized by vault name alone.

### Reading Secrets

```bash
# Use $(op read ...) anywhere a token is needed
op read 'op://Acme Corp/HubSpot/pak'
op read 'op://Acme Corp/HubSpot/service-key-cms'
```

### Running Commands With Secrets Injected

For one-off commands, use `op run` with an environment file:

Create `.env.op` (committable to git, since it contains references not values):

```
HUBSPOT_PERSONAL_ACCESS_KEY=op://Acme Corp/HubSpot/pak
HUBSPOT_SERVICE_KEY=op://Acme Corp/HubSpot/service-key-cms
HUBSPOT_PORTAL_ID=op://Acme Corp/HubSpot/portal-id
```

Run any command with the secrets injected as env vars:

```bash
op run --env-file=.env.op -- hs upload src/theme my-theme --use-env
```

If you find yourself repeating the same `op run` invocation for a CLI tool, write a thin wrapper script that invokes `op run` for you. That way day-to-day usage stays clean (`hs upload ...`) and the credential injection is handled invisibly.

### Service Account for CI/CD

For CI (GitHub Actions, etc.), do not use your personal 1Password login. Create a service account scoped only to the relevant client vault:

1. 1Password admin console → Integrations → Service Accounts → Create
2. Name it descriptively, tied to the client (example: `acme-corp-hubspot-deploy`)
3. Grant **read-only** access to the client's vault (e.g., `Acme Corp`). Do not grant access to other clients' vaults.
4. Save the service account token to GitHub Secrets in the client's repo as `OP_SERVICE_ACCOUNT_TOKEN`

In the GitHub Actions workflow, use `1password/load-secrets-action`:

```yaml
- uses: 1password/load-secrets-action@v2
  with:
    export-env: true
  env:
    OP_SERVICE_ACCOUNT_TOKEN: ${{ secrets.OP_SERVICE_ACCOUNT_TOKEN }}
    HUBSPOT_PERSONAL_ACCESS_KEY: op://Acme Corp/HubSpot/pak
    HUBSPOT_PORTAL_ID: op://Acme Corp/HubSpot/portal-id

- run: hs upload src/theme my-theme --use-env
```

Service account tokens are immutable in scope. If you need to change which vaults the account can read, you must delete and recreate the service account.

### Sharing Credentials With a Teammate

To grant a teammate access (e.g., onboarding Abby):

1. In the 1Password admin console, add them to the **specific client vault(s)** they will be working on, as a member with read access. Do not give blanket access to all client vaults.
2. They run `op signin`
3. They verify with `op read 'op://<Client Vault>/HubSpot/pak'`

You never paste the secret to them directly. They read it from the vault. Per-client access also means a teammate who rolls off Acme can have access revoked from just that vault without touching other client work.

## Auto-Deactivation of Exposed Tokens

HubSpot now monitors public sources (GitHub, npm, common log services) and **automatically deactivates** any token it finds exposed. This applies to PAKs, Service Keys, and Private App tokens.

### Recovery if a token is auto-deactivated

1. You will get a notification email from HubSpot
2. Generate a new key (PAK or Service Key, whichever was deactivated)
3. Update the corresponding 1Password item field
4. Anywhere the old key was used (CI secrets, local configs), re-inject from 1Password
5. Audit how the leak happened (committed `.env` file, log spillage, etc.) and fix the root cause

### Prevention

- Never paste a token directly into a command, config, or commit. Always use `op read` or `op run`.
- Add `.env` and any local config files containing real secrets to `.gitignore`.
- Use `.env.op` (which contains 1Password references, not real secrets) for shareable team configs.

## Official Documentation

- [Service Keys announcement](https://developers.hubspot.com/changelog/service-keys)
- [HubSpot CLI auth docs](https://developers.hubspot.com/docs/cms/developer-reference/local-development-cli)
- [1Password CLI docs](https://developer.1password.com/docs/cli)
- [1Password load-secrets GitHub Action](https://github.com/1Password/load-secrets-action)
