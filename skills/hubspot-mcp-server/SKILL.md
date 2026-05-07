---
name: hubspot-mcp-server
description: HubSpot Developer MCP Server setup and usage for AI-assisted CMS development, build log analysis, test account generation, and documentation search. Use when setting up the HubSpot MCP server or leveraging AI tools for HubSpot development.
---

# HubSpot Developer MCP Server

> **Two HubSpot MCP servers exist. Make sure you want this one.**
>
> | Server | Purpose | Endpoint | This skill? |
> |---|---|---|---|
> | **Local Developer MCP** | Build CMS themes, modules, run build log analysis, generate test accounts | `npx @hubspot/mcp-server` (local) | **Yes** |
> | **Remote CRM MCP** (GA April 13, 2026) | Read and write CRM data via natural language (contacts, deals, activities) | `https://mcp.hubspot.com` | No, out of scope for this plugin |
>
> If you want CRM data access, see [HubSpot's Remote MCP docs](https://developers.hubspot.com/docs/apps/developer-platform/build-apps/integrate-with-the-remote-hubspot-mcp-server). This skill is for the local developer tooling only.

The HubSpot Developer MCP Server (GA February 2026) provides AI-assisted development capabilities for HubSpot CMS and app development via the Model Context Protocol.

## Requirements

- **HubSpot CLI v8.0.0 or higher**
- Node.js 20+
- A HubSpot developer account

## What It Can Do

| Capability | Description |
|-----------|-------------|
| **CMS Theme Management** | Create, modify, and manage themes via natural language |
| **Template Operations** | Create and update page, blog, and email templates |
| **Module Development** | Scaffold and configure CMS modules |
| **Build Log Analysis** | Retrieve and analyze project build logs for debugging |
| **Test Account Generation** | Create temporary test accounts with specified hub tiers |
| **Documentation Search** | Query official HubSpot developer docs |
| **CLI Guidance** | Step-by-step CLI command walkthroughs |

## Setup

### 1. Install/Update CLI

```bash
npm install -g @hubspot/cli@latest

# Verify version (must be 8.0.0+)
hs --version
```

### 2. Authenticate

```bash
echo "my-account" | hs account auth --pak="$(op read 'op://Acme Corp/HubSpot/pak')"
```

For the full auth workflow (Service Keys, 1Password CLI patterns, CI service accounts), see the `hubspot-auth-and-secrets` skill.

### 3. Configure MCP

Add the HubSpot MCP server to your AI tool's configuration:

**For Claude Code (`~/.claude/settings.json` or project `.mcp.json`):**

```json
{
  "mcpServers": {
    "hubspot": {
      "command": "npx",
      "args": ["-y", "@hubspot/mcp-server"]
    }
  }
}
```

**For other MCP-compatible tools**, use the same `npx` command as the server entry point.

### 4. Verify Connection

Once configured, the MCP server provides tools that your AI assistant can call to interact with your HubSpot account.

## When to Use MCP Server vs. Direct CLI

| Scenario | Use |
|----------|-----|
| Scaffolding a new theme from scratch | MCP Server (faster iteration with AI) |
| Quick file upload during development | Direct CLI (`hs upload`) |
| Debugging a failed build | MCP Server (can analyze build logs) |
| CI/CD deployment | Direct CLI (with `--use-env`) |
| Exploring what's possible | MCP Server (can search docs) |
| Creating test environments | MCP Server (test account generation) |
| Routine `hs watch` development | Direct CLI |

## Key Features

### Build Log Analysis

The MCP server can retrieve and analyze build logs from your HubSpot projects, helping identify deployment failures, asset compilation errors, and configuration issues.

### Test Account Generation

Generate temporary HubSpot test accounts with specific subscription tiers for testing theme compatibility across different HubSpot plans. Useful for verifying that premium features degrade gracefully.

### Documentation Search

Query the official HubSpot developer documentation through the MCP server. This is especially useful for looking up HubL functions, field types, and API endpoints without leaving your editor.

## Limitations

- Runs locally (not a cloud service)
- Requires active CLI authentication
- Cannot replace CI/CD pipelines for production deployments
- Test accounts are temporary

## Official Documentation

- [MCP Server GA Announcement](https://developers.hubspot.com/changelog/hubspot-developer-mcp-server-for-app-and-cms-development-now-in-ga)
- [HubSpot Developer Docs](https://developers.hubspot.com/)
