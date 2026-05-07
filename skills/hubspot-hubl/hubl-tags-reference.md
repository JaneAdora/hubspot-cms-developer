# HubL Tags Reference

## Standard Tags

### block
Define overridable sections in layouts.
```hubl
{% block content %}
  Default content here
{% endblock %}
```

### call
Call a macro with additional content.
```hubl
{% call modal() %}
  <p>Modal body content</p>
{% endcall %}
```

### do
Execute expressions without output.
```hubl
{% do my_list.append("new item") %}
```

### extends
Inherit from a base template.
```hubl
{% extends "layouts/base.html" %}
```

### filter
Apply filter to block of content.
```hubl
{% filter upper %}
  this will be uppercase
{% endfilter %}
```

### for
Iterate over sequences.
```hubl
{% for item in items %}
  {{ item }}
{% endfor %}
```

### from
Import specific macros from a template.
```hubl
{% from "macros/forms.html" import input, button %}
```

### if
Conditional rendering.
```hubl
{% if condition %}{% elif %}{% else %}{% endif %}
```

### import
Import macros as a namespace.
```hubl
{% import "macros/forms.html" as forms %}
{{ forms.input("email") }}
```

### include
Include another template.
```hubl
{% include "partials/header.html" %}
{% include "partials/nav.html" without context %}
{% include "partials/sidebar.html" with context %}
```

### macro
Define reusable template functions.
```hubl
{% macro input(name, type="text", value="") %}
  <input type="{{ type }}" name="{{ name }}" value="{{ value }}">
{% endmacro %}
```

### raw
Output content without parsing HubL.
```hubl
{% raw %}
  {{ this will not be parsed }}
{% endraw %}
```

### set
Assign values to variables.
```hubl
{% set name = "value" %}
{% set items = ["a", "b", "c"] %}
```

### with
Create a scoped variable context.
```hubl
{% with items = get_items() %}
  {% for item in items %}{{ item }}{% endfor %}
{% endwith %}
```

## HubSpot-Specific Tags

### blog_comments
Render blog comment form and list.
```hubl
{% blog_comments "blog_comments" %}
```

### choice
Create a dropdown field.
```hubl
{% choice "field_name" label="Select Option", choices="opt1,opt2,opt3" %}
```

### content_attribute
Set page-level attributes.
```hubl
{% content_attribute "meta_description" %}
  Page description here
{% end_content_attribute %}
```

### dnd_area
Create drag-and-drop area.
```hubl
{% dnd_area "main_content" %}
  {% dnd_section %}
    {% dnd_column %}
      {% dnd_row %}
        {% dnd_module path="@hubspot/rich_text" %}
        {% end_dnd_module %}
      {% end_dnd_row %}
    {% end_dnd_column %}
  {% end_dnd_section %}
{% end_dnd_area %}
```

### dnd_section
Container for columns.
```hubl
{% dnd_section
  padding={
    "top": 60,
    "bottom": 60
  },
  max_width=1200
%}
```

### dnd_column
Column within a section.
```hubl
{% dnd_column
  width=6,
  offset=0
%}
```

### dnd_row
Row within a column.
```hubl
{% dnd_row %}
```

### dnd_module
Place a module in drag-and-drop area.
```hubl
{% dnd_module
  path="@hubspot/rich_text",
  html="<p>Content</p>"
%}
{% end_dnd_module %}
```

### email_each
Loop for email templates.
```hubl
{% email_each items as item %}
  {{ item }}
{% end_email_each %}
```

### email_flex_area
Flexible area for emails.
```hubl
{% email_flex_area "sidebar" %}
```

### form
Embed a HubSpot form.
```hubl
{% form
  form_to_use="{{ module.form.form_id }}",
  response_response_type="redirect",
  response_redirect_url="https://example.com/thanks"
%}
```

### gallery
Image gallery module.
```hubl
{% gallery "image_gallery" %}
```

### header
Page header tag.
```hubl
{% header "page_header" header_tag="h1" %}
```

### icon
Render an icon.
```hubl
{% icon
  name="hubspot",
  style="solid",
  width="24"
%}
```

### image
Image field.
```hubl
{% image "hero_image" label="Hero Image", responsive=true %}
```

### image_src
Output just the image URL.
```hubl
{% image_src "image" src="{{ module.image.src }}" %}
```

### include_dnd_partial
Include a drag-and-drop partial.
```hubl
{% include_dnd_partial path="partials/hero.html" %}
```

### js_partial (React)
Include a React component.
```hubl
{% js_partial
  path="@projects/my-project/components/partials/Header.jsx",
  pageTitle="My Page"
%}
```

### language_switcher
Multi-language selector.
```hubl
{% language_switcher "lang_switcher" %}
```

### linked_image
Image with a link.
```hubl
{% linked_image "cta_image" link="https://example.com" %}
```

### logo
Site logo.
```hubl
{% logo "company_logo" %}
```

### menu
Navigation menu.
```hubl
{% menu "main_nav" site_map_name="default", root_type="site_root", max_levels=2 %}
```

### module
Embed a module.
```hubl
{% module "my_module" path="/modules/custom-module" %}
```

### module_block
Module with a default.
```hubl
{% module_block module "rich_text" path="@hubspot/rich_text" %}
  {% module_attribute "html" %}
    <p>Default content</p>
  {% end_module_attribute %}
{% end_module_block %}
```

### page_footer
CAN-SPAM footer for emails.
```hubl
{% page_footer "footer" %}
```

### password_prompt
Password protection prompt.
```hubl
{% password_prompt "password_form" %}
```

### post_filter
Blog post filtering.
```hubl
{% post_filter "filter" %}
```

### post_listing
Blog post list.
```hubl
{% post_listing "posts"
  limit=10,
  list_title="Recent Posts"
%}
```

### print
Debug output.
```hubl
{% print variable %}
```

### require_css
Load CSS file.
```hubl
{% require_css %}
  <link rel="stylesheet" href="{{ get_asset_url('css/main.css') }}">
{% end_require_css %}
```

### require_head
Add content to <head>.
```hubl
{% require_head %}
  <meta name="custom" content="value">
{% end_require_head %}
```

### require_js
Load JavaScript file.
```hubl
{% require_js %}
  <script src="{{ get_asset_url('js/main.js') }}"></script>
{% end_require_js %}
```

### require_js_url
Load external JavaScript.
```hubl
{% require_js_url "https://cdn.example.com/lib.js", position="footer" %}
```

### rich_text
WYSIWYG content area.
```hubl
{% rich_text "main_content" label="Content" %}
```

### scope_css
Scope CSS to a container.
```hubl
{% scope_css %}
  <style>
    .my-module { color: blue; }
  </style>
{% end_scope_css %}
```

### section_header
Section heading.
```hubl
{% section_header "section_title" header_tag="h2" %}
```

### simple_menu
Simple navigation menu.
```hubl
{% simple_menu "nav" %}
```

### social_sharing
Social share buttons.
```hubl
{% social_sharing "share_buttons" %}
```

### subscription_preferences
Email subscription manager.
```hubl
{% subscription_preferences "email_prefs" %}
```

### text
Plain text field.
```hubl
{% text "tagline" label="Tagline", value="Default text" %}
```

### textarea
Multi-line text field.
```hubl
{% textarea "description" label="Description" %}
```

### video_player
Video embed.
```hubl
{% video_player "video"
  player_id="{{ module.video.player_id }}",
  width="100%"
%}
```

### widget_block
Legacy module block.
```hubl
{% widget_block widget_type="custom_widget" %}
{% end_widget_block %}
```
