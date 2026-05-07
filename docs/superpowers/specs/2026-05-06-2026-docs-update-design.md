# HubSpot CMS Developer Plugin: 2026 Docs Update

**Date:** 2026-05-06
**Plugin version target:** 2.0.0 → 2.1.0
**Status:** Approved

## Goal

Update the `hubspot-cms-developer` plugin to reflect HubSpot platform changes shipped between Feb 2026 and April 2026, and add internal guidance for managing HubSpot credentials in 1Password. This is preparation for sharing the plugin publicly with teammates (e.g., Abby) and eventually as an open-source repo.

This update is scoped to **CMS-relevant** changes only. CRM-side changes (e.g., the Remote MCP Server at mcp.hubspot.com) get cross-references but no deep coverage.

## In-scope HubSpot platform changes

| Change | Date | Affects |
|---|---|---|
| Service Keys (public beta) | Feb 2026 | Auth flow for system-to-system integrations; replaces legacy Private App Access Tokens |
| Date-based API versioning (`/YYYY-MM/`) | March 30, 2026 | Replaces v1 through v4 URL versioning across HubSpot APIs |
| Developer Platform 2026.03 GA / Spring 2026 Spotlight | April 14, 2026 | CLI projects, react themes, platform features |
| CLDR Locale Migration | April 1, 2026 | HubL date, time, and number formatting |
| Automatic Deactivation of Exposed Tokens | 2026 (public beta) | Security; tokens leaked to git or logs are auto-disabled |
| Remote HubSpot MCP Server GA | April 13, 2026 | CRM only; cross-reference in our local-MCP skill so users don't conflate them |

## Out-of-scope

- ChatGPT connector enhancements (CRM-side)
- Custom Events for Pro
- Third-party app install confirmation dialog
- App developer rollups (data model builder, OAuth install logging, etc.)
- The Remote MCP Server itself (CRM-only, only documented as a disambiguation pointer)

## Design

### New skill: `hubspot-auth-and-secrets`

A single skill covering all HubSpot auth flows and secret-storage patterns. Auth and storage answer the same question ("set up CLI access for a new client"), so they stay together.

Contents:
1. **Decision tree.** "I need to do X, what auth do I need?"
2. **CLI Personal Access Key (PAK).** For `hs auth`, theme, module, and HubDB CLI work.
3. **Service Keys.** For HubDB API writes, custom CRM integrations, server-side scripts (no webhooks).
4. **Private App Access Tokens.** Flagged as legacy, with migration steps to Service Keys.
5. **1Password integration (required, not optional).**
   - Stance: 1Password CLI is the assumed workflow for this plugin's audience. Other secret stores are not documented.
   - Onboarding: detect whether `op` is installed; if not, walk the user through installation (link to official docs and Context7 for current commands). The skill itself should not duplicate `op` install instructions that drift over time.
   - **Per-client vault model.** Each client has its own 1Password vault (e.g., `Acme Corp`). HubSpot credentials live in an item named `HubSpot` inside that vault. Access is granted per-client by adding teammates to specific client vaults. This matches a common per-client secrets pattern.
   - Item naming: `HubSpot` (one item per client vault, predictable path)
   - Standard fields: `pak`, `service-key-cms`, `service-key-crm`, `portal-id`
   - Path pattern: `op://<Client Vault>/HubSpot/<field>`
   - `op run` patterns to inject secrets into `hs` and custom scripts
   - Service account setup for CI/CD (model: `skai-agent-v2`), with read-only access to the relevant client vault(s) only
6. **Auto-deactivation of exposed tokens.** Security note plus recovery steps.

### Existing skill updates

| Skill | Edit |
|---|---|
| `hubspot-cli` | Replace the auth section with a one-paragraph pointer toward `hubspot-auth-and-secrets`. Add Developer Platform 2026.03 GA reference. Do not retain duplicated PAK content. |
| `hubspot-mcp-server` | Add disambiguation block at top: "Two MCP servers exist. Local Developer MCP (this skill) vs. Remote CRM MCP at mcp.hubspot.com (out of scope)." |
| `hubspot-hubdb` | Update API URL examples from `/v3/` to `/2026-03/`. Replace token-creation examples with Service Keys. Point to auth skill. |
| `hubspot-ci-cd` | Update API URL examples to date-based versioning. Add Service Keys plus GitHub Secrets pattern. Add exposed-token security note. Point to auth skill. |
| `hubspot-hubl` | Add subsection on CLDR locale migration impact on date, time, and number filters. |

Skills not touched in this update: `hubspot-form-styling`, `hubspot-templates`, `hubspot-theme-development`, `hubspot-module-development`, `hubspot-react-themes`. We will not modify these unless we discover stale content while editing the others.

### Plugin manifest

`.claude-plugin/plugin.json`:
- Bump version 2.0.0 to 2.1.0
- Update description to mention auth/secrets management and 2026 platform updates
- Add `auth`, `service-keys`, `1password` to keywords

## Execution order

1. Write `hubspot-auth-and-secrets/SKILL.md`
2. Update `hubspot-cli/SKILL.md`
3. Update `hubspot-mcp-server/SKILL.md`
4. Update `hubspot-hubdb/SKILL.md`
5. Update `hubspot-ci-cd/SKILL.md`
6. Update `hubspot-hubl/SKILL.md`
7. Update `.claude-plugin/plugin.json` (version, description, keywords)

Each step is one focused edit. No interdependencies. Order chosen so we write the new skill first, then point existing skills at it.

## Out of scope for this work

- README at plugin root (part of the public-share work in step 3 of the broader plan)
- Git initialization of the plugin
- LICENSE file
- Any deeper rewrites of skills not listed above
- Migrating existing client credentials from PATs to Service Keys (operational task, not docs)

## Success criteria

- All six in-scope changes are reflected in the plugin
- Auth guidance is consolidated in one place
- A teammate (Abby) loading the plugin gets current, non-conflicting guidance on auth
- Plugin version bumped, manifest accurate
- Existing skills with no needed changes are left alone (no busywork rewrites)
