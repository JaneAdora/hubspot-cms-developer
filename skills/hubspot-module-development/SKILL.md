---
name: hubspot-module-development
description: HubSpot CMS module creation, fields.json configuration, meta.json settings, and module patterns. Use when building custom modules, configuring module fields, or implementing reusable components.
---

# HubSpot Module Development

## Module Structure

Every module is a folder with `.module` suffix:

```
my-module.module/
├── module.html          # Main HubL template (required)
├── module.css           # Module styles
├── module.js            # Module JavaScript
├── fields.json          # Field definitions (required)
├── meta.json            # Module metadata (required)
└── images/              # Module-specific assets
    └── preview.png
```

## meta.json

Required configuration file:

```json
{
  "label": "My Custom Module",
  "css_assets": [],
  "external_js": [],
  "global": false,
  "help_text": "Description shown to editors",
  "host_template_types": ["PAGE", "BLOG_POST", "BLOG_LISTING"],
  "icon": "module",
  "is_available_for_new_content": true,
  "js_assets": [],
  "other_assets": [],
  "smart_type": "NOT_SMART",
  "tags": [],
  "categories": ["text"]
}
```

### meta.json Properties

| Property | Type | Description |
|----------|------|-------------|
| `label` | string | Display name in editor |
| `global` | boolean | Is this a global module? |
| `host_template_types` | array | Where module can be used |
| `is_available_for_new_content` | boolean | Show in module picker |
| `icon` | string | Module icon name |
| `categories` | array | Module categories |
| `help_text` | string | Editor description |

### host_template_types Values

- `PAGE` - Website pages
- `BLOG_POST` - Blog post templates
- `BLOG_LISTING` - Blog listing templates
- `EMAIL` - Email templates
- `QUOTE` - Quote templates
- `KNOWLEDGE_ARTICLE` - Knowledge base

## fields.json

### CRITICAL: Reserved Field Names

These names **CANNOT** be used as field `name` values in fields.json. Using them causes silent upload failures with no clear error message:

- `body` -> use `body_text` or `body_content`
- `name` -> use `item_name` or `display_name`
- `id` -> use `item_id` or `identifier`
- `type` -> use `item_type` or `content_type`
- `label` -> use `item_label` or `display_label`

Define editable fields:

```json
[
  {
    "type": "text",
    "name": "heading",
    "label": "Heading",
    "required": true,
    "default": "Default Heading"
  },
  {
    "type": "richtext",
    "name": "content",
    "label": "Content",
    "default": "<p>Enter your content here.</p>"
  },
  {
    "type": "image",
    "name": "image",
    "label": "Image",
    "responsive": true,
    "default": {
      "src": "",
      "alt": "",
      "loading": "lazy"
    }
  },
  {
    "type": "link",
    "name": "button_link",
    "label": "Button Link",
    "default": {
      "url": {
        "type": "EXTERNAL",
        "href": ""
      },
      "open_in_new_tab": false
    }
  },
  {
    "type": "choice",
    "name": "alignment",
    "label": "Alignment",
    "display": "select",
    "default": "left",
    "choices": [
      ["left", "Left"],
      ["center", "Center"],
      ["right", "Right"]
    ]
  }
]
```

For complete field reference, see [field-types-reference.md](field-types-reference.md).

## module.html

Access fields via `module` object:

```hubl
{# Access field values #}
<section class="my-module my-module--{{ module.alignment }}">
  {% if module.image.src %}
    <div class="my-module__image">
      <img
        src="{{ module.image.src }}"
        alt="{{ module.image.alt }}"
        loading="{{ module.image.loading }}"
        {% if module.image.width %}width="{{ module.image.width }}"{% endif %}
        {% if module.image.height %}height="{{ module.image.height }}"{% endif %}
      >
    </div>
  {% endif %}

  <div class="my-module__content">
    {% if module.heading %}
      <h2>{{ module.heading }}</h2>
    {% endif %}

    {% if module.content %}
      {{ module.content }}
    {% endif %}

    {% if module.button_link.url.href %}
      <a
        href="{{ module.button_link.url.href }}"
        {% if module.button_link.open_in_new_tab %}target="_blank" rel="noopener"{% endif %}
        class="btn"
      >
        {{ module.button_text|default("Learn More") }}
      </a>
    {% endif %}
  </div>
</section>
```

## module.css

Scoped styles:

```css
/* Module styles - automatically scoped */
.my-module {
  padding: 40px 0;
}

.my-module__image img {
  max-width: 100%;
  height: auto;
}

.my-module__content {
  max-width: 800px;
}

/* Alignment variants */
.my-module--left {
  text-align: left;
}

.my-module--center {
  text-align: center;
  margin: 0 auto;
}

.my-module--right {
  text-align: right;
  margin-left: auto;
}

/* Responsive */
@media (max-width: 767px) {
  .my-module {
    padding: 20px 0;
  }
}
```

## module.js

Module JavaScript:

```javascript
(function() {
  // Module initialization
  const modules = document.querySelectorAll('.my-module');

  modules.forEach(function(module) {
    // Module logic here
  });
})();
```

## Common Module Patterns

### Card Module

```json
// fields.json
[
  {
    "type": "image",
    "name": "image",
    "label": "Card Image",
    "responsive": true
  },
  {
    "type": "text",
    "name": "title",
    "label": "Card Title",
    "required": true
  },
  {
    "type": "textarea",
    "name": "description",
    "label": "Description"
  },
  {
    "type": "link",
    "name": "link",
    "label": "Card Link"
  }
]
```

```hubl
{# module.html #}
<article class="card">
  {% if module.image.src %}
    <div class="card__image">
      <img src="{{ module.image.src }}" alt="{{ module.image.alt }}">
    </div>
  {% endif %}

  <div class="card__body">
    <h3 class="card__title">{{ module.title }}</h3>

    {% if module.description %}
      <p class="card__description">{{ module.description }}</p>
    {% endif %}

    {% if module.link.url.href %}
      <a href="{{ module.link.url.href }}" class="card__link">
        Read More
      </a>
    {% endif %}
  </div>
</article>
```

### Repeater Module (Multiple Items)

```json
// fields.json
[
  {
    "type": "group",
    "name": "items",
    "label": "Items",
    "occurrence": {
      "min": 1,
      "max": 12,
      "default": 3
    },
    "children": [
      {
        "type": "text",
        "name": "title",
        "label": "Title"
      },
      {
        "type": "richtext",
        "name": "content",
        "label": "Content"
      },
      {
        "type": "icon",
        "name": "icon",
        "label": "Icon"
      }
    ]
  },
  {
    "type": "number",
    "name": "columns",
    "label": "Columns",
    "default": 3,
    "min": 1,
    "max": 4
  }
]
```

```hubl
{# module.html #}
<div class="feature-grid feature-grid--{{ module.columns }}-col">
  {% for item in module.items %}
    <div class="feature-grid__item">
      {% if item.icon %}
        {% icon
          name="{{ item.icon.name }}"
          style="{{ item.icon.type }}"
          purpose="decorative"
        %}
      {% endif %}

      {% if item.title %}
        <h3>{{ item.title }}</h3>
      {% endif %}

      {% if item.content %}
        {{ item.content }}
      {% endif %}
    </div>
  {% endfor %}
</div>
```

### Tabs Module

```json
// fields.json
[
  {
    "type": "group",
    "name": "tabs",
    "label": "Tabs",
    "occurrence": {
      "min": 2,
      "max": 8,
      "default": 3,
      "sorting_label_field": "tab_label"
    },
    "children": [
      {
        "type": "text",
        "name": "tab_label",
        "label": "Tab Label",
        "required": true
      },
      {
        "type": "richtext",
        "name": "content",
        "label": "Tab Content"
      }
    ]
  }
]
```

```hubl
{# module.html #}
{% set unique_id = module.module_id %}

<div class="tabs" id="tabs-{{ unique_id }}">
  <div class="tabs__nav" role="tablist">
    {% for tab in module.tabs %}
      <button
        class="tabs__tab{% if loop.first %} is-active{% endif %}"
        role="tab"
        aria-selected="{{ 'true' if loop.first else 'false' }}"
        aria-controls="panel-{{ unique_id }}-{{ loop.index }}"
        id="tab-{{ unique_id }}-{{ loop.index }}"
      >
        {{ tab.tab_label }}
      </button>
    {% endfor %}
  </div>

  <div class="tabs__panels">
    {% for tab in module.tabs %}
      <div
        class="tabs__panel{% if loop.first %} is-active{% endif %}"
        role="tabpanel"
        id="panel-{{ unique_id }}-{{ loop.index }}"
        aria-labelledby="tab-{{ unique_id }}-{{ loop.index }}"
        {% if not loop.first %}hidden{% endif %}
      >
        {{ tab.content }}
      </div>
    {% endfor %}
  </div>
</div>
```

### Accordion Module

```json
// fields.json
[
  {
    "type": "group",
    "name": "items",
    "label": "Accordion Items",
    "occurrence": {
      "min": 1,
      "max": 20,
      "default": 3
    },
    "children": [
      {
        "type": "text",
        "name": "title",
        "label": "Title",
        "required": true
      },
      {
        "type": "richtext",
        "name": "content",
        "label": "Content"
      }
    ]
  },
  {
    "type": "boolean",
    "name": "allow_multiple",
    "label": "Allow multiple open",
    "default": false
  }
]
```

```hubl
{# module.html #}
{% set unique_id = module.module_id %}

<div
  class="accordion"
  data-allow-multiple="{{ module.allow_multiple }}"
>
  {% for item in module.items %}
    <div class="accordion__item">
      <button
        class="accordion__trigger"
        aria-expanded="false"
        aria-controls="accordion-content-{{ unique_id }}-{{ loop.index }}"
        id="accordion-header-{{ unique_id }}-{{ loop.index }}"
      >
        {{ item.title }}
        <span class="accordion__icon" aria-hidden="true"></span>
      </button>

      <div
        class="accordion__content"
        id="accordion-content-{{ unique_id }}-{{ loop.index }}"
        aria-labelledby="accordion-header-{{ unique_id }}-{{ loop.index }}"
        hidden
      >
        {{ item.content }}
      </div>
    </div>
  {% endfor %}
</div>
```

## Global Modules

Global modules share content across all pages.

In `meta.json`:
```json
{
  "global": true
}
```

Access global module data:
```hubl
{% module "header" path="@hubspot/header", global=true %}
```

## Module with Style Options

```json
// fields.json
[
  {
    "type": "group",
    "name": "content",
    "label": "Content",
    "children": [
      {
        "type": "text",
        "name": "heading",
        "label": "Heading"
      },
      {
        "type": "richtext",
        "name": "body_content",
        "label": "Body"
      }
    ]
  },
  {
    "type": "group",
    "name": "styles",
    "label": "Styles",
    "tab": "STYLE",
    "children": [
      {
        "type": "color",
        "name": "background_color",
        "label": "Background Color",
        "default": {
          "color": "#ffffff",
          "opacity": 100
        }
      },
      {
        "type": "color",
        "name": "text_color",
        "label": "Text Color",
        "default": {
          "color": "#333333",
          "opacity": 100
        }
      },
      {
        "type": "spacing",
        "name": "padding",
        "label": "Padding"
      }
    ]
  }
]
```

```hubl
{# module.html #}
{% set bg_color = module.styles.background_color.color %}
{% set text_color = module.styles.text_color.color %}
{% set padding = module.styles.padding %}

<section
  class="content-block"
  style="
    background-color: {{ bg_color }};
    color: {{ text_color }};
    {% if padding %}
      padding-top: {{ padding.top.value }}{{ padding.top.units }};
      padding-right: {{ padding.right.value }}{{ padding.right.units }};
      padding-bottom: {{ padding.bottom.value }}{{ padding.bottom.units }};
      padding-left: {{ padding.left.value }}{{ padding.left.units }};
    {% endif %}
  "
>
  {% if module.content.heading %}
    <h2>{{ module.content.heading }}</h2>
  {% endif %}
  {{ module.content.body_content }}
</section>
```

## Best Practices

1. **Semantic HTML**: Use appropriate elements (article, section, nav, etc.)
2. **Accessibility**: Include ARIA attributes, alt text, proper headings
3. **Default Values**: Always provide sensible defaults
4. **Validation**: Use required fields and visibility conditions
5. **Documentation**: Add help_text to complex fields
6. **Naming**: Use BEM or similar CSS methodology
7. **Performance**: Lazy load images, minimize JS
