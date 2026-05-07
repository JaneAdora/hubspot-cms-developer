---
description: Create a new HubSpot CMS module with all required files
allowed-tools: Write, Bash, Read
---

# Create New HubSpot Module

Create a new HubSpot CMS module with the proper structure.

## Steps

1. Ask for the module name and purpose if not provided
2. Determine appropriate field types based on the purpose
3. Create the module directory structure:

```
{module-name}.module/
├── module.html
├── module.css
├── module.js
├── fields.json
└── meta.json
```

4. Create meta.json with:
   - Appropriate label
   - host_template_types based on use case
   - is_available_for_new_content: true
   - Relevant categories

5. Create fields.json with fields appropriate for the module purpose
   - **CRITICAL:** Never use reserved field names: `body`, `name`, `id`, `type`, `label` (causes silent upload failures). Use `body_text`, `item_name`, `item_id`, `item_type`, `item_label` instead.

6. Create module.html with:
   - Semantic HTML structure
   - Proper field access via `module.field_name`
   - Conditional rendering for optional fields
   - Accessibility attributes

7. Create module.css with:
   - BEM naming convention
   - Mobile-first responsive styles
   - Use of CSS custom properties

8. Create module.js if interactive functionality is needed

Provide guidance on:
- How to add the module to templates
- Testing the module locally
- Best practices for the specific module type
