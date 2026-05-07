# HubSpot CMS Developer (Claude Code Plugin)

Opinionated guidance for HubSpot CMS development with Service Keys, the 1Password CLI, and CI/CD. Built for agencies that manage CMS work across multiple HubSpot portals.

## What's in here

- **12 skills** covering CLI, HubL, themes, modules, HubDB, forms, React themes, CI/CD, MCP server integration, auth/secrets management, and safe-development workflows
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
| [`hubspot-safe-development`](skills/hubspot-safe-development/SKILL.md) | Pre-flight checks, sandbox-first policy, backup and diff protocols, incident recovery |
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

## Safety

This plugin assumes a sandbox-first development workflow with backups and diff-before-push protocols. Before any `hs upload`, `hs fetch`, or production deploy, work through the pre-flight checklist in [`hubspot-safe-development`](skills/hubspot-safe-development/SKILL.md). The skill covers account verification, backup conventions, the diff-before-push protocol (which prevents editor-vs-code conflicts), and recovery procedures for the most common incident classes.

## Versioning

See [CHANGELOG.md](CHANGELOG.md).

## How this plugin evolved

Design specs and implementation plans live in [`docs/superpowers/`](docs/superpowers/) for transparency. They were generated using the [Superpowers](https://github.com/obra/superpowers) brainstorming and writing-plans skills.

## License

MIT. See [LICENSE](LICENSE).
