---
name: hubspot-hubl
description: HubL (HubSpot Markup Language) templating syntax, tags, filters, functions, operators, loops, and conditionals. Use when writing HubL templates, debugging template logic, or implementing dynamic content in HubSpot CMS.
---

# HubL Templating Language

HubL is HubSpot's Jinja-inspired templating language for building templates and modules on the HubSpot CMS.

## Syntax Delimiters

| Delimiter | Purpose | Example |
|-----------|---------|---------|
| `{{ }}` | Expression/output | `{{ content.name }}` |
| `{% %}` | Statement/logic | `{% if condition %}` |
| `{# #}` | Comments | `{# This is a comment #}` |

## Variables

Access data using dot notation:
```hubl
{{ content.name }}
{{ module.field_name }}
{{ request.query_dict.param }}
```

### Setting Variables
```hubl
{% set my_var = "value" %}
{% set items = [1, 2, 3] %}
{% set data = {"key": "value"} %}
```

## Control Flow

### If Statements
```hubl
{% if condition %}
  <!-- content -->
{% elif other_condition %}
  <!-- other content -->
{% else %}
  <!-- fallback -->
{% endif %}
```

### Inline If (Ternary)
```hubl
{{ "Yes" if is_active else "No" }}
{{ value if value else default_value }}
```

### For Loops
```hubl
{% for item in items %}
  {{ item }}
{% endfor %}
```

#### Loop Variables
| Variable | Description |
|----------|-------------|
| `loop.index` | Current iteration (1-indexed) |
| `loop.index0` | Current iteration (0-indexed) |
| `loop.first` | True if first iteration |
| `loop.last` | True if last iteration |
| `loop.length` | Total number of items |
| `loop.cycle()` | Cycle through values |

```hubl
{% for item in items %}
  <div class="{{ loop.cycle('odd', 'even') }}">
    {{ loop.index }}. {{ item }}
  </div>
{% endfor %}
```

### Loop with Else
```hubl
{% for item in items %}
  {{ item }}
{% else %}
  No items found.
{% endfor %}
```

## Operators

### Comparison
| Operator | Description |
|----------|-------------|
| `==` | Equal |
| `!=` | Not equal |
| `>`, `>=` | Greater than |
| `<`, `<=` | Less than |
| `is` | Identity test |
| `in` | Containment |

### Logical
| Operator | Description |
|----------|-------------|
| `and` | Logical AND |
| `or` | Logical OR |
| `not` | Logical NOT |

### Math
| Operator | Description |
|----------|-------------|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `//` | Floor division |
| `%` | Modulo |
| `**` | Power |

## Expression Tests

```hubl
{% if value is defined %}
{% if value is none %}
{% if value is string %}
{% if value is number %}
{% if value is iterable %}
{% if value is mapping %}
{% if value is sequence %}
{% if number is divisibleby(3) %}
{% if number is even %}
{% if number is odd %}
```

For complete reference, see [hubl-tags-reference.md](hubl-tags-reference.md), [hubl-filters-reference.md](hubl-filters-reference.md), and [hubl-functions-reference.md](hubl-functions-reference.md).

## Common Patterns

### Null/Empty Checks
```hubl
{% if content.featured_image %}
  <img src="{{ content.featured_image }}">
{% endif %}

{# With default fallback #}
{{ content.title|default("Untitled") }}
```

### Including Partials
```hubl
{% include "partials/header.html" %}
{% include "partials/footer.html" %}
```

### Extending Layouts
```hubl
{% extends "layouts/base.html" %}

{% block content %}
  <!-- Page-specific content -->
{% endblock %}
```

### Macros (Reusable Functions)
```hubl
{% macro button(text, url, style="primary") %}
  <a href="{{ url }}" class="btn btn-{{ style }}">{{ text }}</a>
{% endmacro %}

{{ button("Click Me", "/contact") }}
{{ button("Learn More", "/about", "secondary") }}
```

### Importing Macros
```hubl
{% from "macros/buttons.html" import button %}
{{ button("Click", "/link") }}
```

## Locale and Formatting (CLDR)

As of April 1, 2026, HubSpot's content rendering uses CLDR (Unicode Common Locale Data Repository) instead of legacy Java Runtime Environment locale data. This affects HubL filters that format dates, times, and numbers.

### What changed

Output from these filters may differ slightly from pre-April 2026 renderings:

- `format_datetime`, `format_date`, `format_time`
- `format_currency`, `format_number`

The differences are typically subtle: digit grouping separators, weekday abbreviations, currency symbol placement. They can still affect cached pages and visual regression snapshots.

### What to do

- After April 2026, re-render any cached pages that depend on locale-specific output and update visual regression baselines if you have them.
- If a specific format is critical, pin it explicitly rather than relying on locale defaults:

```hubl
{# Pinned format, locale-independent #}
{{ content.publish_date|format_datetime("yyyy-MM-dd") }}

{# Locale-dependent (CLDR will determine the format) #}
{{ content.publish_date|format_datetime("medium", "en-US") }}
```

- For multilingual sites, test rendering for each locale you support after the migration.

### Reference

- HubSpot CLDR migration announcement (April 1, 2026 entry on the [Developer Changelog](https://developers.hubspot.com/changelog))
- [Unicode CLDR project](https://cldr.unicode.org/)

## Debug Tips

```hubl
{# Print variable for debugging #}
{{ variable|pprint }}

{# Check variable type #}
{{ variable.__class__.__name__ }}

{# List all available properties #}
{% for key in content|keys %}
  {{ key }}: {{ content[key] }}<br>
{% endfor %}
```
