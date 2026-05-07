---
name: hubspot-hubdb
description: HubDB dynamic data tables for HubSpot CMS, querying rows in HubL templates, filtering and sorting, dynamic pages from HubDB, and CLI management. Use when working with structured data, directory pages, team listings, or dynamic content driven by table data.
---

# HubDB - Dynamic Data Tables

HubDB provides structured data tables for HubSpot CMS, similar to a lightweight database. Use it for team directories, product listings, location finders, event calendars, and other structured content.

> **API versioning note (March 2026):** When calling the HubDB API directly (e.g., from a server-side script), HubSpot replaced numeric version paths (`/v3/`) with date-based versions (`/2026-03/`). Use `https://api.hubapi.com/cms/2026-03/hubdb/...` for new code. The `hs hubdb` CLI commands handle versioning internally.

## When to Use HubDB

- **Structured, repeating data** that editors need to manage (team members, locations, products)
- **Dynamic pages** generated from table rows (one page per row)
- **Filterable/sortable content** that goes beyond what modules can handle
- **Shared data** used across multiple templates

## Authentication

`hs hubdb` CLI commands use the HubSpot CLI's existing auth (your PAK). Direct HubDB API calls require a Service Key (preferred) or, for read-only access, a PAK. See the `hubspot-auth-and-secrets` skill for setup.

For API calls, the recommended pattern is to inject the Service Key from 1Password:

```bash
op run --env-file=.env.op -- ./your-script.sh
```

Inside the script, the key is available as `$HUBSPOT_SERVICE_KEY` and goes in the `Authorization: Bearer ...` header.

## Creating Tables

### Via HubSpot UI

1. Marketing > Files and Templates > HubDB
2. Create table with columns (text, number, URL, image, select, etc.)
3. Add rows via the table editor

### Via CLI

```bash
# Create from JSON definition
hs hubdb create table-definition.json

# Fetch existing table
hs hubdb fetch TABLE_ID output.json

# Clear all rows
hs hubdb clear TABLE_ID

# Delete table
hs hubdb delete TABLE_ID
```

### Table Definition JSON

```json
{
  "name": "team_members",
  "label": "Team Members",
  "columns": [
    {
      "name": "member_name",
      "label": "Name",
      "type": "TEXT"
    },
    {
      "name": "role",
      "label": "Role",
      "type": "TEXT"
    },
    {
      "name": "photo",
      "label": "Photo",
      "type": "IMAGE"
    },
    {
      "name": "bio",
      "label": "Bio",
      "type": "RICHTEXT"
    },
    {
      "name": "department",
      "label": "Department",
      "type": "SELECT",
      "options": [
        {"id": 1, "name": "Engineering"},
        {"id": 2, "name": "Marketing"},
        {"id": 3, "name": "Sales"}
      ]
    }
  ]
}
```

### Column Types

| Type | Description |
|------|-------------|
| `TEXT` | Plain text |
| `RICHTEXT` | Rich text / HTML |
| `NUMBER` | Numeric value |
| `CURRENCY` | Currency amount |
| `DATE` | Date value |
| `DATETIME` | Date and time |
| `URL` | URL with link text |
| `IMAGE` | Image with URL, width, height |
| `SELECT` | Dropdown selection |
| `MULTISELECT` | Multiple selections |
| `BOOLEAN` | True/false |
| `LOCATION` | Latitude/longitude |
| `FOREIGN_ID` | Reference to another table row |
| `VIDEO` | Video embed |

## Querying in HubL

### Get All Rows

```hubl
{% set rows = hubdb_table_rows(TABLE_ID) %}
{% for row in rows %}
  <div class="team-member">
    <img src="{{ row.photo.url }}" alt="{{ row.member_name }}">
    <h3>{{ row.member_name }}</h3>
    <p>{{ row.role }}</p>
  </div>
{% endfor %}
```

### Get a Single Row

```hubl
{% set row = hubdb_table_row(TABLE_ID, ROW_ID) %}
<h1>{{ row.member_name }}</h1>
```

### Filter Rows

```hubl
{# Filter by column value #}
{% set engineers = hubdb_table_rows(TABLE_ID, "department=Engineering") %}

{# Filter with multiple conditions #}
{% set senior_eng = hubdb_table_rows(TABLE_ID, "department=Engineering&role__contains=Senior") %}

{# Filter by number range #}
{% set expensive = hubdb_table_rows(TABLE_ID, "price__gt=100") %}
```

### Filter Operators

| Operator | Example | Description |
|----------|---------|-------------|
| `=` | `department=Engineering` | Exact match |
| `__contains` | `role__contains=Senior` | Contains substring |
| `__gt` | `price__gt=100` | Greater than |
| `__gte` | `price__gte=100` | Greater than or equal |
| `__lt` | `price__lt=50` | Less than |
| `__lte` | `price__lte=50` | Less than or equal |
| `__ne` | `status__ne=inactive` | Not equal |
| `__in` | `department__in=Engineering,Marketing` | In list |
| `__isnull` | `photo__isnull=true` | Is null/empty |

### Sort Rows

```hubl
{# Sort ascending #}
{% set rows = hubdb_table_rows(TABLE_ID, "orderBy=member_name") %}

{# Sort descending #}
{% set rows = hubdb_table_rows(TABLE_ID, "orderBy=-member_name") %}

{# Sort by multiple columns #}
{% set rows = hubdb_table_rows(TABLE_ID, "orderBy=department&orderBy=member_name") %}
```

### Limit Results

```hubl
{# Get first 10 rows #}
{% set rows = hubdb_table_rows(TABLE_ID, "limit=10") %}

{# Paginate: skip 10, get next 10 #}
{% set rows = hubdb_table_rows(TABLE_ID, "limit=10&offset=10") %}
```

### Combine Filter + Sort + Limit

```hubl
{% set featured = hubdb_table_rows(TABLE_ID, "department=Engineering&orderBy=member_name&limit=5") %}
```

## Dynamic Pages from HubDB

Create one page per table row automatically.

### Setup

1. Create a HubDB table with a `name` column (used for page titles) and a `path` column of type TEXT (used for URL slugs)
2. Enable "Allow creation of dynamic pages" in table settings
3. Create a template with the `dynamic_page_hubdb_table_id` annotation

### Template

```hubl
<!--
  templateType: page
  label: Team Member Page
  isAvailableForNewContent: true
  dynamic_page_hubdb_table_id: TABLE_ID
-->

{% extends "./layouts/base.html" %}

{% block body %}
  {# `dynamic_page_hubdb_row` is automatically set to the current row #}
  {% set row = dynamic_page_hubdb_row %}

  <article class="team-member-detail">
    <img src="{{ row.photo.url }}" alt="{{ row.member_name }}">
    <h1>{{ row.member_name }}</h1>
    <p class="role">{{ row.role }}</p>
    <div class="bio">{{ row.bio }}</div>
  </article>
{% endblock %}
```

### Listing Page

```hubl
<!--
  templateType: page
  label: Team Directory
  isAvailableForNewContent: true
  dynamic_page_hubdb_table_id: TABLE_ID
-->

{% extends "./layouts/base.html" %}

{% block body %}
  {# On the listing page, dynamic_page_hubdb_rows contains all rows #}
  {% if dynamic_page_hubdb_row %}
    {# Detail page - show single row #}
    <h1>{{ dynamic_page_hubdb_row.member_name }}</h1>
  {% else %}
    {# Listing page - show all rows #}
    <h1>Our Team</h1>
    {% for row in dynamic_page_hubdb_rows %}
      <a href="{{ row.hs_path }}">
        <h3>{{ row.member_name }}</h3>
        <p>{{ row.role }}</p>
      </a>
    {% endfor %}
  {% endif %}
{% endblock %}
```

## Using HubDB in Modules

### Module fields.json

```json
[
  {
    "type": "hubdbtable",
    "name": "data_table",
    "label": "Select Data Table"
  },
  {
    "type": "number",
    "name": "max_rows",
    "label": "Max Rows to Display",
    "default": 10
  }
]
```

### Module template (module.html)

```hubl
{% set rows = hubdb_table_rows(module.data_table, "limit=" ~ module.max_rows) %}
{% for row in rows %}
  <div class="data-row">
    {{ row.hs_name }}
  </div>
{% endfor %}
```

## Passing HubDB Data to React Components

```hubl
{% set rows = hubdb_table_rows(TABLE_ID) %}
{% set serialized = [] %}
{% for row in rows %}
  {% do serialized.append({
    "name": row.member_name,
    "role": row.role,
    "photo": row.photo.url,
    "department": row.department.name
  }) %}
{% endfor %}

{% js_partial
  path="@projects/my-theme/components/partials/TeamGrid.jsx",
  members=serialized
%}
```

## Best Practices

1. **Use meaningful column names** that follow the same conventions as module field names
2. **Avoid reserved names** (`body`, `name`, `id`, `type`, `label`) for column names
3. **Cache considerations**: HubDB data is cached; changes may take a few minutes to appear
4. **Row limits**: Tables support up to 10,000 rows; for larger datasets, use the HubSpot API
5. **Image columns**: Store image URLs, not base64 data
6. **SELECT columns**: Define options upfront; changing options later can break existing rows
