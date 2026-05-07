---
description: Create a new HubSpot CMS template (page, blog, or system)
allowed-tools: Write, Bash, Read
---

# Create New HubSpot Template

Create a new HubSpot CMS template file.

## Steps

1. Ask for template details if not provided:
   - Template name
   - Template type (page, blog_post, blog_listing, email, system)
   - Purpose/description

2. For page templates, create with:
   - Proper template annotations
   - Extension from base layout
   - Drag-and-drop areas with sensible defaults

3. For blog templates, include:
   - Appropriate blog variables (content.name, publish_date, etc.)
   - Author information
   - Tag handling
   - Pagination for listing templates

4. For system templates (404, 500, password, search):
   - Appropriate error messaging
   - Navigation back to home
   - Search functionality where relevant

5. Template should include:
   - `templateType` annotation
   - `label` annotation
   - `isAvailableForNewContent: true`
   - Extension from base layout
   - Proper block structure

Provide information about:
- How to assign the template in HubSpot
- Customization options
- Related templates that might be needed
