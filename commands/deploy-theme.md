---
description: Deploy a HubSpot CMS theme to a target environment using the CLI
allowed-tools: Bash, Read
---

# Deploy HubSpot Theme

Deploy a HubSpot CMS theme using CLI v8.0.0+ with proper authentication.

## Steps

1. Verify HubSpot CLI v8+ is installed:
   ```bash
   hs --version
   ```
   If not installed or below v8: `npm install -g @hubspot/cli@latest`

2. Verify authentication is configured:
   ```bash
   hs accounts list
   ```
   If no accounts: `echo "account-name" | hs account auth --pak='YOUR_KEY'`

3. Ask for the source directory and destination path if not provided
   - Source: local directory containing the theme (e.g., `src/theme`)
   - Destination: path on HubSpot (e.g., `my-theme`)

4. Ask for the target account if multiple accounts are configured

5. Deploy using `--use-env` when environment variables are set, or `--account` flag:
   ```bash
   # With environment variables (recommended for CI/CD)
   hs upload src/theme my-theme --use-env

   # With account flag (for local development)
   hs upload src/theme my-theme --account=production
   ```

6. Report deployment status and provide the preview URL:
   ```
   Preview: https://app.hubspot.com/content/PORTAL_ID/preview
   ```
