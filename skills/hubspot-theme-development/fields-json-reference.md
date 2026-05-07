# fields.json Reference

Complete reference for theme and module field types.

## Field Structure

Every field has these common properties:

```json
{
  "type": "text",
  "name": "field_name",
  "label": "Field Label",
  "required": false,
  "locked": false,
  "default": "Default value",
  "help_text": "Help text for editors",
  "inline_help_text": "Inline hint",
  "visibility": {
    "controlling_field_path": "other_field",
    "operator": "EQUAL",
    "controlling_value_regex": "true"
  }
}
```

## Reserved Field Names (DO NOT USE)

These names conflict with HubSpot internals and cause silent upload failures:
`body`, `name`, `id`, `type`, `label`

Use alternatives: `body_text`, `item_name`, `item_id`, `item_type`, `item_label`.

## Text Fields

### text
Single-line text input.
```json
{
  "type": "text",
  "name": "title",
  "label": "Title",
  "default": "Default Title",
  "validation_regex": "^[A-Za-z0-9 ]+$",
  "allow_new_line": false,
  "show_emoji_picker": false,
  "placeholder": "Enter title..."
}
```

### textarea
Multi-line text input.
```json
{
  "type": "textarea",
  "name": "description",
  "label": "Description",
  "default": "",
  "rows": 4
}
```

### richtext
WYSIWYG HTML editor.
```json
{
  "type": "richtext",
  "name": "content",
  "label": "Content",
  "default": "<p>Default content</p>",
  "enabled_features": [
    "bold",
    "italic",
    "underline",
    "link",
    "image",
    "lists",
    "headings",
    "source_code"
  ]
}
```

## Number Fields

### number
Numeric input.
```json
{
  "type": "number",
  "name": "count",
  "label": "Count",
  "default": 10,
  "min": 1,
  "max": 100,
  "step": 1,
  "suffix": "items",
  "prefix": "$",
  "display": "text"
}
```

With slider display:
```json
{
  "type": "number",
  "name": "opacity",
  "label": "Opacity",
  "default": 100,
  "min": 0,
  "max": 100,
  "step": 1,
  "suffix": "%",
  "display": "slider"
}
```

## Boolean Fields

### boolean
Toggle/checkbox.
```json
{
  "type": "boolean",
  "name": "show_title",
  "label": "Show Title",
  "default": true,
  "display": "toggle"
}
```

Checkbox display:
```json
{
  "type": "boolean",
  "name": "is_featured",
  "label": "Featured",
  "default": false,
  "display": "checkbox"
}
```

## Choice Fields

### choice
Dropdown, radio, checkbox, or button group.
```json
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
```

Radio buttons:
```json
{
  "type": "choice",
  "name": "size",
  "label": "Size",
  "display": "radio",
  "default": "medium",
  "choices": [
    ["small", "Small"],
    ["medium", "Medium"],
    ["large", "Large"]
  ]
}
```

Button group with icons:
```json
{
  "type": "choice",
  "name": "layout",
  "label": "Layout",
  "display": "button_group",
  "default": "grid",
  "choices": [
    {
      "value": "grid",
      "label": "Grid",
      "icon": "grid"
    },
    {
      "value": "list",
      "label": "List",
      "icon": "list"
    }
  ]
}
```

Multiple selection:
```json
{
  "type": "choice",
  "name": "categories",
  "label": "Categories",
  "display": "checkbox",
  "multiple": true,
  "default": ["news"],
  "choices": [
    ["news", "News"],
    ["updates", "Updates"],
    ["events", "Events"]
  ]
}
```

## Media Fields

### image
Image picker.
```json
{
  "type": "image",
  "name": "hero_image",
  "label": "Hero Image",
  "responsive": true,
  "resizable": true,
  "show_loading": true,
  "default": {
    "src": "",
    "alt": "",
    "width": 1200,
    "height": 600,
    "loading": "lazy"
  }
}
```

### file
File picker.
```json
{
  "type": "file",
  "name": "download",
  "label": "Download File",
  "picker": "file",
  "default": null
}
```

### video
Video picker.
```json
{
  "type": "videoplayer",
  "name": "video",
  "label": "Video",
  "default": {
    "player_id": null,
    "height": null,
    "width": null
  }
}
```

### icon
Icon picker.
```json
{
  "type": "icon",
  "name": "icon",
  "label": "Icon",
  "default": {
    "name": "star",
    "unicode": "f005",
    "type": "SOLID"
  }
}
```

## Color Fields

### color
Color picker with opacity.
```json
{
  "type": "color",
  "name": "background_color",
  "label": "Background Color",
  "default": {
    "color": "#ffffff",
    "opacity": 100
  }
}
```

### gradient
Gradient picker.
```json
{
  "type": "gradient",
  "name": "background_gradient",
  "label": "Background Gradient",
  "default": {
    "side_or_corner": {
      "verticalSide": "TOP",
      "horizontalSide": null
    },
    "colors": [
      {
        "color": {
          "r": 0,
          "g": 102,
          "b": 204,
          "a": 1
        }
      },
      {
        "color": {
          "r": 102,
          "g": 51,
          "b": 153,
          "a": 1
        }
      }
    ]
  }
}
```

## Typography Fields

### font
Font picker.
```json
{
  "type": "font",
  "name": "body_font",
  "label": "Body Font",
  "load_external_fonts": true,
  "default": {
    "font": "Lato",
    "font_set": "GOOGLE",
    "size": 16,
    "size_unit": "px",
    "color": "#333333",
    "styles": {}
  }
}
```

### textalignment
Text alignment picker.
```json
{
  "type": "textalignment",
  "name": "text_align",
  "label": "Text Alignment",
  "default": {
    "text_align": "LEFT"
  }
}
```

## Link Fields

### link
Link/URL picker.
```json
{
  "type": "link",
  "name": "cta_link",
  "label": "CTA Link",
  "supported_types": ["EXTERNAL", "CONTENT", "FILE", "EMAIL", "BLOG"],
  "default": {
    "url": {
      "content_id": null,
      "type": "EXTERNAL",
      "href": ""
    },
    "open_in_new_tab": false,
    "no_follow": false
  }
}
```

### url
Simple URL input.
```json
{
  "type": "url",
  "name": "external_url",
  "label": "External URL",
  "default": {
    "content_id": null,
    "type": "EXTERNAL",
    "href": ""
  },
  "supported_types": ["EXTERNAL"]
}
```

## Layout Fields

### spacing
Margin/padding control.
```json
{
  "type": "spacing",
  "name": "margin",
  "label": "Margin",
  "default": {
    "top": {
      "value": 0,
      "units": "px"
    },
    "right": {
      "value": 0,
      "units": "px"
    },
    "bottom": {
      "value": 0,
      "units": "px"
    },
    "left": {
      "value": 0,
      "units": "px"
    }
  }
}
```

### alignment
Horizontal/vertical alignment.
```json
{
  "type": "alignment",
  "name": "content_alignment",
  "label": "Content Alignment",
  "alignment_direction": "BOTH",
  "default": {
    "horizontal_align": "CENTER",
    "vertical_align": "MIDDLE"
  }
}
```

### border
Border control.
```json
{
  "type": "border",
  "name": "border",
  "label": "Border",
  "default": {
    "top": {
      "width": {
        "value": 1,
        "units": "px"
      },
      "style": "solid",
      "color": "#cccccc",
      "opacity": 100
    },
    "right": null,
    "bottom": null,
    "left": null
  }
}
```

### backgroundimage
Background image control.
```json
{
  "type": "backgroundimage",
  "name": "background",
  "label": "Background Image",
  "default": {
    "src": "",
    "background_position": "MIDDLE_CENTER",
    "background_size": "cover"
  }
}
```

## Content Fields

### form
HubSpot form picker.
```json
{
  "type": "form",
  "name": "contact_form",
  "label": "Contact Form",
  "default": {
    "response_type": "inline",
    "message": "Thank you for submitting!",
    "form_id": null
  }
}
```

### cta
CTA picker.
```json
{
  "type": "cta",
  "name": "cta",
  "label": "Call to Action",
  "default": {
    "guid": null
  }
}
```

### blog
Blog picker.
```json
{
  "type": "blog",
  "name": "blog",
  "label": "Select Blog",
  "default": null
}
```

### hubdbrow
HubDB row picker.
```json
{
  "type": "hubdbrow",
  "name": "row",
  "label": "Select Row",
  "table_name_or_id": "my_table",
  "columns_to_fetch": ["name", "description"],
  "default": null
}
```

### hubdbtable
HubDB table picker.
```json
{
  "type": "hubdbtable",
  "name": "table",
  "label": "Select Table",
  "default": null
}
```

### menu
Menu picker.
```json
{
  "type": "menu",
  "name": "navigation",
  "label": "Navigation Menu",
  "default": null
}
```

### tag
Blog tag picker.
```json
{
  "type": "tag",
  "name": "filter_tag",
  "label": "Filter by Tag",
  "tag_value": "slug",
  "default": null
}
```

### followupemail
Follow-up email picker.
```json
{
  "type": "followupemail",
  "name": "confirmation_email",
  "label": "Confirmation Email",
  "default": null
}
```

## CRM Fields

### crmobject
CRM object picker.
```json
{
  "type": "crmobject",
  "name": "contact",
  "label": "Select Contact",
  "object_type": "CONTACT",
  "properties_to_fetch": ["firstname", "lastname", "email"],
  "default": null
}
```

### crmobjectproperty
CRM property picker.
```json
{
  "type": "crmobjectproperty",
  "name": "property",
  "label": "Select Property",
  "object_types": ["CONTACT"],
  "default": null
}
```

## Group Fields

### group (field_group)
Group related fields.
```json
{
  "type": "group",
  "name": "button_settings",
  "label": "Button Settings",
  "expanded": true,
  "children": [
    {
      "type": "text",
      "name": "text",
      "label": "Button Text",
      "default": "Click Here"
    },
    {
      "type": "link",
      "name": "link",
      "label": "Button Link"
    },
    {
      "type": "choice",
      "name": "style",
      "label": "Button Style",
      "choices": [
        ["primary", "Primary"],
        ["secondary", "Secondary"]
      ],
      "default": "primary"
    }
  ]
}
```

### Repeater group
```json
{
  "type": "group",
  "name": "slides",
  "label": "Slides",
  "occurrence": {
    "min": 1,
    "max": 10,
    "default": 3,
    "sorting_label_field": "title"
  },
  "children": [
    {
      "type": "image",
      "name": "image",
      "label": "Slide Image"
    },
    {
      "type": "text",
      "name": "title",
      "label": "Slide Title"
    }
  ]
}
```

## Visibility Conditions

Control field visibility based on other fields:

```json
{
  "type": "text",
  "name": "custom_url",
  "label": "Custom URL",
  "visibility": {
    "controlling_field_path": "link_type",
    "operator": "EQUAL",
    "controlling_value_regex": "custom"
  }
}
```

Operators:
- `EQUAL` - Value matches regex
- `NOT_EQUAL` - Value doesn't match
- `EMPTY` - Field is empty
- `NOT_EMPTY` - Field has value

Multiple conditions:
```json
{
  "visibility": {
    "operator": "AND",
    "criteria": [
      {
        "controlling_field_path": "show_advanced",
        "operator": "EQUAL",
        "controlling_value_regex": "true"
      },
      {
        "controlling_field_path": "type",
        "operator": "EQUAL",
        "controlling_value_regex": "custom"
      }
    ]
  }
}
```

## Special Properties

### locked
Prevent editing in content editor:
```json
{
  "locked": true
}
```

### required
Make field required:
```json
{
  "required": true
}
```

### inherited_value
Inherit from theme settings:
```json
{
  "inherited_value": {
    "property_value_paths": {
      "color": "theme.colors.primary_color.color"
    }
  }
}
```
