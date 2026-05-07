---
name: hubspot-theme-development
description: HubSpot CMS theme structure, theme.json configuration, fields.json for style settings, and theme organization best practices. Use when creating themes, setting up theme structure, or configuring theme settings.
---

# HubSpot Theme Development

## Theme Structure

```
my-theme/
├── theme.json              # Required: Theme configuration
├── fields.json             # Theme-level style fields
├── templates/
│   ├── layouts/
│   │   └── base.html       # Base layout template
│   ├── pages/
│   │   ├── home.html
│   │   └── landing.html
│   ├── blog/
│   │   ├── blog-listing.html
│   │   └── blog-post.html
│   ├── system/
│   │   ├── 404.html
│   │   ├── 500.html
│   │   ├── password-prompt.html
│   │   └── search-results.html
│   └── partials/
│       ├── header.html
│       └── footer.html
├── modules/
│   └── custom-module.module/
├── css/
│   ├── main.css
│   └── _variables.css
├── js/
│   └── main.js
└── images/
    └── logo.svg
```

## theme.json

Required configuration file for every theme.

```json
{
  "label": "My Theme",
  "preview_path": "./images/theme-preview.png",
  "screenshot_path": "./images/theme-screenshot.png",
  "enable_domain_stylesheets": false,
  "responsive_breakpoints": [
    {
      "name": "mobile",
      "mediaQuery": "(max-width: 767px)"
    },
    {
      "name": "tablet",
      "mediaQuery": "(min-width: 768px) and (max-width: 1024px)"
    },
    {
      "name": "desktop",
      "mediaQuery": "(min-width: 1025px)"
    }
  ]
}
```

### theme.json Properties

| Property | Type | Description |
|----------|------|-------------|
| `label` | string | Display name in HubSpot |
| `preview_path` | string | Path to preview image |
| `screenshot_path` | string | Path to screenshot |
| `enable_domain_stylesheets` | boolean | Enable domain-specific CSS |
| `responsive_breakpoints` | array | Define breakpoints |

## fields.json (Theme Level)

Define theme-wide style settings editors can customize.

```json
[
  {
    "type": "group",
    "name": "typography",
    "label": "Typography",
    "children": [
      {
        "type": "font",
        "name": "primary_font",
        "label": "Primary Font",
        "default": {
          "font": "Lato",
          "font_set": "GOOGLE",
          "size": 16,
          "size_unit": "px"
        }
      },
      {
        "type": "font",
        "name": "heading_font",
        "label": "Heading Font",
        "default": {
          "font": "Montserrat",
          "font_set": "GOOGLE"
        }
      }
    ]
  },
  {
    "type": "group",
    "name": "colors",
    "label": "Colors",
    "children": [
      {
        "type": "color",
        "name": "primary_color",
        "label": "Primary Color",
        "default": {
          "color": "#0066cc",
          "opacity": 100
        }
      },
      {
        "type": "color",
        "name": "secondary_color",
        "label": "Secondary Color",
        "default": {
          "color": "#333333",
          "opacity": 100
        }
      },
      {
        "type": "color",
        "name": "background_color",
        "label": "Background Color",
        "default": {
          "color": "#ffffff",
          "opacity": 100
        }
      }
    ]
  },
  {
    "type": "group",
    "name": "spacing",
    "label": "Spacing",
    "children": [
      {
        "type": "number",
        "name": "section_padding",
        "label": "Section Padding",
        "default": 60,
        "suffix": "px",
        "min": 0,
        "max": 200
      }
    ]
  }
]
```

For complete field type reference, see [fields-json-reference.md](fields-json-reference.md).

## Accessing Theme Settings

In templates and modules:

```hubl
{# Access theme settings #}
{{ theme.typography.primary_font.font }}
{{ theme.colors.primary_color.color }}
{{ theme.spacing.section_padding }}

{# In CSS #}
<style>
  body {
    font-family: {{ theme.typography.primary_font.font }};
    color: {{ theme.colors.primary_color.color }};
  }

  section {
    padding: {{ theme.spacing.section_padding }}px 0;
  }
</style>
```

## Base Layout Template

`templates/layouts/base.html`:

```hubl
<!DOCTYPE html>
<html lang="{{ html_lang }}" {{ html_lang_dir }}>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">

  {# HubSpot standard head #}
  {{ standard_header_includes }}

  {# Theme CSS #}
  {{ require_css(get_asset_url("css/main.css")) }}

  {# Theme settings as CSS variables #}
  <style>
    :root {
      --primary-color: {{ theme.colors.primary_color.color }};
      --secondary-color: {{ theme.colors.secondary_color.color }};
      --primary-font: {{ theme.typography.primary_font.font }};
      --heading-font: {{ theme.typography.heading_font.font }};
    }
  </style>

  {% block head %}{% endblock %}
</head>
<body>
  {% include "partials/header.html" %}

  <main>
    {% block body %}{% endblock %}
  </main>

  {% include "partials/footer.html" %}

  {# Theme JavaScript #}
  {{ require_js(get_asset_url("js/main.js"), "footer") }}

  {{ standard_footer_includes }}

  {% block footer_scripts %}{% endblock %}
</body>
</html>
```

## Page Template

`templates/pages/home.html`:

```hubl
<!--
  templateType: page
  label: Home Page
  isAvailableForNewContent: true
-->

{% extends "layouts/base.html" %}

{% block body %}
  {% dnd_area "main_content"
    label="Main Content"
  %}
    {% dnd_section %}
      {% dnd_column %}
        {% dnd_row %}
          {% dnd_module
            path="@hubspot/rich_text"
          %}
          {% end_dnd_module %}
        {% end_dnd_row %}
      {% end_dnd_column %}
    {% end_dnd_section %}
  {% end_dnd_area %}
{% endblock %}
```

## Template Annotations

Add at the top of template files:

```hubl
<!--
  templateType: page | blog_post | blog_listing | email | error | password_prompt | search_results | subscription
  label: Template Display Name
  isAvailableForNewContent: true | false
  screenshotPath: ./images/template-preview.png
-->
```

## Partials

`templates/partials/header.html`:

```hubl
<header class="site-header">
  <div class="container">
    <div class="header-logo">
      {% if theme.logo.src %}
        <a href="/">
          <img src="{{ theme.logo.src }}" alt="{{ theme.logo.alt }}">
        </a>
      {% else %}
        {% logo "logo" %}
      {% endif %}
    </div>

    <nav class="main-nav">
      {% menu "main_menu"
        site_map_name="default",
        root_type="site_root",
        max_levels=2,
        flow="horizontal"
      %}
    </nav>

    {% module "header_cta"
      path="@hubspot/cta",
      label="Header CTA"
    %}
  </div>
</header>
```

## Global Partials

For site-wide editable content (header, footer):

```hubl
{# In template #}
{% global_partial path="partials/header.html" %}

{# Or include as regular partial #}
{% include "partials/header.html" %}
```

## CSS Organization

`css/main.css`:

```css
/* Import variables */
@import url('./_variables.css');

/* Base styles */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: var(--primary-font), sans-serif;
  color: var(--text-color);
  background: var(--background-color);
}

/* Use HubSpot's CSS variables from theme settings */
h1, h2, h3, h4, h5, h6 {
  font-family: var(--heading-font), sans-serif;
  color: var(--heading-color);
}

a {
  color: var(--primary-color);
}

/* Layout */
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

/* Responsive */
@media (max-width: 767px) {
  .container {
    padding: 0 15px;
  }
}
```

## JavaScript

`js/main.js`:

```javascript
(function() {
  'use strict';

  // Mobile menu toggle
  const menuToggle = document.querySelector('.menu-toggle');
  const mainNav = document.querySelector('.main-nav');

  if (menuToggle && mainNav) {
    menuToggle.addEventListener('click', function() {
      mainNav.classList.toggle('is-open');
      this.setAttribute('aria-expanded',
        this.getAttribute('aria-expanded') === 'true' ? 'false' : 'true'
      );
    });
  }

  // Smooth scroll for anchor links
  document.querySelectorAll('a[href^="#"]').forEach(anchor => {
    anchor.addEventListener('click', function(e) {
      e.preventDefault();
      const target = document.querySelector(this.getAttribute('href'));
      if (target) {
        target.scrollIntoView({ behavior: 'smooth' });
      }
    });
  });
})();
```

## Best Practices

1. **Use CSS Variables**: Define theme colors/fonts as CSS custom properties
2. **Mobile First**: Design for mobile, enhance for desktop
3. **Semantic HTML**: Use proper heading hierarchy, landmarks
4. **Accessibility**: Include ARIA labels, alt text, focus states
5. **Performance**: Optimize images, minimize CSS/JS
6. **DRY Partials**: Extract reusable components to partials
7. **Consistent Naming**: Use BEM or similar CSS methodology

## File Naming Conventions

- Templates: `kebab-case.html`
- Modules: `module-name.module/`
- CSS: `_partial.css` for imports, `main.css` for compiled
- JS: `module-name.js`, `main.js`
- Images: `descriptive-name.{png,jpg,svg}`

## .hsignore File

Exclude files from upload:

```
node_modules/
.git/
*.log
.DS_Store
*.scss
*.map
README.md
```
