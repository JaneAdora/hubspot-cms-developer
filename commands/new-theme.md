---
description: Scaffold a new HubSpot CMS theme with proper directory structure and configuration files
allowed-tools: Write, Bash, Read
---

# Create New HubSpot Theme

Create a new HubSpot CMS theme with the standard directory structure.

## Steps

1. Ask for the theme name if not provided
2. Create the theme directory structure:

```
{theme-name}/
├── theme.json
├── fields.json
├── templates/
│   ├── layouts/
│   │   └── base.html
│   ├── pages/
│   │   └── home.html
│   ├── blog/
│   │   ├── blog-listing.html
│   │   └── blog-post.html
│   ├── system/
│   │   └── 404.html
│   └── partials/
│       ├── header.html
│       └── footer.html
├── modules/
├── css/
│   └── main.css
├── js/
│   └── main.js
├── images/
└── .hsignore
```

3. Create theme.json with theme metadata
4. Create fields.json with basic theme settings (colors, typography)
5. Create base.html layout template
6. Create home.html page template extending base
7. Create header.html and footer.html partials
8. Create main.css with CSS variables from theme settings
9. Create main.js with basic initialization
10. Create .hsignore file

After scaffolding, provide instructions for:
- Running `hs account auth` to authenticate (NOTE: `hs init` was removed in CLI v8.0.0)
- Using `hs watch --initial-upload` to start development
- Uploading with `hs upload`
- **Reminder:** Never use reserved field names in fields.json: `body`, `name`, `id`, `type`, `label`
