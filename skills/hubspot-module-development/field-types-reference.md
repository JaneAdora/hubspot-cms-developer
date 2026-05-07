# Module Field Types Quick Reference

Quick lookup for common field configurations.

## Reserved Field Names (DO NOT USE)

These names cause silent upload failures when used as field `name` values:
`body`, `name`, `id`, `type`, `label`

Use alternatives: `body_text`, `item_name`, `item_id`, `item_type`, `item_label`.

## Text Input

```json
{
  "type": "text",
  "name": "title",
  "label": "Title",
  "default": "Default value",
  "required": true,
  "placeholder": "Enter title..."
}
```

## Rich Text (WYSIWYG)

```json
{
  "type": "richtext",
  "name": "content",
  "label": "Content",
  "enabled_features": ["bold", "italic", "link", "image", "lists"]
}
```

## Image

```json
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
}
```

## Link/URL

```json
{
  "type": "link",
  "name": "link",
  "label": "Link",
  "supported_types": ["EXTERNAL", "CONTENT", "FILE", "EMAIL", "BLOG"],
  "default": {
    "url": { "type": "EXTERNAL", "href": "" },
    "open_in_new_tab": false
  }
}
```

## Dropdown Select

```json
{
  "type": "choice",
  "name": "style",
  "label": "Style",
  "display": "select",
  "default": "default",
  "choices": [
    ["default", "Default"],
    ["primary", "Primary"],
    ["secondary", "Secondary"]
  ]
}
```

## Color Picker

```json
{
  "type": "color",
  "name": "color",
  "label": "Color",
  "default": {
    "color": "#000000",
    "opacity": 100
  }
}
```

## Number/Slider

```json
{
  "type": "number",
  "name": "count",
  "label": "Count",
  "min": 1,
  "max": 10,
  "default": 3,
  "display": "slider"
}
```

## Toggle/Checkbox

```json
{
  "type": "boolean",
  "name": "show_title",
  "label": "Show Title",
  "default": true,
  "display": "toggle"
}
```

## Repeater Group

```json
{
  "type": "group",
  "name": "items",
  "label": "Items",
  "occurrence": {
    "min": 1,
    "max": 10,
    "default": 3
  },
  "children": [
    { "type": "text", "name": "title", "label": "Title" },
    { "type": "text", "name": "description", "label": "Description" }
  ]
}
```

## HubSpot Form

```json
{
  "type": "form",
  "name": "form",
  "label": "Form",
  "default": {
    "response_type": "inline",
    "message": "Thank you!"
  }
}
```

## Icon

```json
{
  "type": "icon",
  "name": "icon",
  "label": "Icon",
  "default": {
    "name": "star",
    "type": "SOLID"
  }
}
```

## CTA

```json
{
  "type": "cta",
  "name": "cta",
  "label": "Call to Action"
}
```

## Blog Picker

```json
{
  "type": "blog",
  "name": "blog",
  "label": "Select Blog"
}
```

## Menu Picker

```json
{
  "type": "menu",
  "name": "menu",
  "label": "Navigation Menu"
}
```

## Conditional Visibility

Show/hide fields based on other field values.

### Basic: Show when a field equals a value

```json
{
  "type": "text",
  "name": "custom_value",
  "label": "Custom Value",
  "visibility": {
    "controlling_field_path": "content_type",
    "operator": "EQUAL",
    "controlling_value_regex": "custom"
  }
}
```

### Boolean toggle

```json
{
  "type": "color",
  "name": "overlay_color",
  "label": "Overlay Color",
  "visibility": {
    "controlling_field_path": "show_overlay",
    "operator": "EQUAL",
    "controlling_value_regex": "true"
  }
}
```

### Nested field path (dot notation for groups)

```json
{
  "type": "text",
  "name": "custom_width",
  "label": "Custom Width",
  "visibility": {
    "controlling_field_path": "settings.layout_style",
    "operator": "EQUAL",
    "controlling_value_regex": "custom"
  }
}
```

### NOT_EMPTY operator

```json
{
  "type": "text",
  "name": "image_alt",
  "label": "Image Alt Text",
  "visibility": {
    "controlling_field_path": "image.src",
    "operator": "NOT_EMPTY"
  }
}
```

### Available operators

| Operator | Description |
|----------|-------------|
| `EQUAL` | Field matches regex value |
| `NOT_EQUAL` | Field does not match regex value |
| `EMPTY` | Field has no value |
| `NOT_EMPTY` | Field has any value |
| `MATCHES_REGEX` | Full regex match |
