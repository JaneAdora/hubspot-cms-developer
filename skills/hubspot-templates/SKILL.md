---
name: hubspot-templates
description: HubSpot CMS template types including page templates, blog templates, email templates, system templates, layouts, and partials. Use when creating templates, implementing layouts, or working with template inheritance.
---

# HubSpot Templates

## Template Types

| Type | Purpose | Annotation Value |
|------|---------|------------------|
| Page | Website & landing pages | `page` |
| Blog Post | Individual blog posts | `blog_post` |
| Blog Listing | Blog index/archive | `blog_listing` |
| Email | Marketing emails | `email` |
| System | Error, password, search | Various |

## Template Annotations

Add at the top of every template:

```hubl
<!--
  templateType: page
  label: Home Page Template
  isAvailableForNewContent: true
  screenshotPath: ./images/template-preview.png
-->
```

### Annotation Properties

| Property | Description |
|----------|-------------|
| `templateType` | Template type (required) |
| `label` | Display name in HubSpot |
| `isAvailableForNewContent` | Show in template picker |
| `screenshotPath` | Preview image path |

## Page Templates

### Basic Page Template

```hubl
<!--
  templateType: page
  label: Standard Page
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
            path="@hubspot/rich_text",
            html="<h1>Welcome</h1><p>Your content here.</p>"
          %}
          {% end_dnd_module %}
        {% end_dnd_row %}
      {% end_dnd_column %}
    {% end_dnd_section %}
  {% end_dnd_area %}
{% endblock %}
```

### Landing Page Template

```hubl
<!--
  templateType: page
  label: Landing Page
  isAvailableForNewContent: true
-->

{% extends "layouts/base.html" %}

{% block body %}
  {# Hero Section #}
  {% dnd_area "hero" label="Hero" %}
    {% dnd_section
      background_color={
        "color": "#0066cc"
      },
      padding={
        "top": 80,
        "bottom": 80
      }
    %}
      {% dnd_column %}
        {% dnd_row %}
          {% dnd_module path="@hubspot/rich_text" %}
          {% end_dnd_module %}
        {% end_dnd_row %}
      {% end_dnd_column %}
    {% end_dnd_section %}
  {% end_dnd_area %}

  {# Content Section #}
  {% dnd_area "content" label="Content" %}
    {% dnd_section %}
      {% dnd_column width=8 %}
        {% dnd_row %}
          {% dnd_module path="@hubspot/rich_text" %}
          {% end_dnd_module %}
        {% end_dnd_row %}
      {% end_dnd_column %}
      {% dnd_column width=4 %}
        {% dnd_row %}
          {% dnd_module path="@hubspot/form" %}
          {% end_dnd_module %}
        {% end_dnd_row %}
      {% end_dnd_column %}
    {% end_dnd_section %}
  {% end_dnd_area %}
{% endblock %}
```

## Blog Templates

### Blog Post Template

```hubl
<!--
  templateType: blog_post
  label: Blog Post
  isAvailableForNewContent: true
-->

{% extends "layouts/base.html" %}

{% block body %}
<article class="blog-post">
  <header class="blog-post__header">
    <h1>{{ content.name }}</h1>

    <div class="blog-post__meta">
      <span class="blog-post__author">
        By {{ content.blog_author.display_name }}
      </span>

      <time class="blog-post__date" datetime="{{ content.publish_date }}">
        {{ content.publish_date_localized }}
      </time>

      {# Custom date format #}
      <time>
        {{ content.publish_date|datetimeformat("%B %d, %Y") }}
      </time>
    </div>

    {% if content.tag_list %}
      <div class="blog-post__tags">
        {% for tag in content.tag_list %}
          <a href="{{ blog_tag_url(group.id, tag.slug) }}">
            {{ tag.name }}
          </a>
        {% endfor %}
      </div>
    {% endif %}
  </header>

  {% if content.featured_image %}
    <div class="blog-post__featured-image">
      <img
        src="{{ content.featured_image }}"
        alt="{{ content.featured_image_alt_text }}"
      >
    </div>
  {% endif %}

  <div class="blog-post__content">
    {{ content.post_body }}
  </div>

  <footer class="blog-post__footer">
    {# Author bio #}
    {% if content.blog_author.bio %}
      <div class="author-bio">
        {% if content.blog_author.avatar %}
          <img src="{{ content.blog_author.avatar }}" alt="">
        {% endif %}
        <div>
          <strong>{{ content.blog_author.display_name }}</strong>
          <p>{{ content.blog_author.bio }}</p>
        </div>
      </div>
    {% endif %}

    {# Social sharing #}
    {% module "social_sharing"
      path="@hubspot/social_sharing"
    %}

    {# Comments #}
    {% if blog_settings.allow_comments %}
      {% module "blog_comments"
        path="@hubspot/blog_comments"
      %}
    {% endif %}
  </footer>
</article>

{# Related posts #}
{% set related = blog_recent_tag_posts(group.id, content.tag_list[0].slug, 3) %}
{% if related %}
  <aside class="related-posts">
    <h2>Related Posts</h2>
    {% for post in related %}
      {% if post.id != content.id %}
        <a href="{{ post.absolute_url }}">{{ post.name }}</a>
      {% endif %}
    {% endfor %}
  </aside>
{% endif %}
{% endblock %}
```

### Blog Listing Template

```hubl
<!--
  templateType: blog_listing
  label: Blog Listing
  isAvailableForNewContent: true
-->

{% extends "layouts/base.html" %}

{% block body %}
<div class="blog-listing">
  <header class="blog-listing__header">
    <h1>{{ group.public_title }}</h1>

    {# Show tag/author if filtered #}
    {% if tag %}
      <p>Posts tagged: {{ tag }}</p>
    {% endif %}

    {% if blog_author %}
      <p>Posts by: {{ blog_author.display_name }}</p>
    {% endif %}
  </header>

  <div class="blog-listing__posts">
    {% for post in contents %}
      <article class="post-card">
        {% if post.featured_image %}
          <a href="{{ post.absolute_url }}">
            <img src="{{ post.featured_image }}" alt="">
          </a>
        {% endif %}

        <div class="post-card__content">
          <h2>
            <a href="{{ post.absolute_url }}">{{ post.name }}</a>
          </h2>

          <time datetime="{{ post.publish_date }}">
            {{ post.publish_date_localized }}
          </time>

          <p>{{ post.post_summary|truncate(150) }}</p>

          <a href="{{ post.absolute_url }}" class="read-more">
            Read More
          </a>
        </div>
      </article>
    {% endfor %}
  </div>

  {# Pagination #}
  {% if contents.total_page_count > 1 %}
    <nav class="blog-pagination">
      {% if last_page_num %}
        <a href="{{ blog_page_link(last_page_num) }}">Previous</a>
      {% endif %}

      <span>Page {{ current_page_num }} of {{ contents.total_page_count }}</span>

      {% if next_page_num %}
        <a href="{{ blog_page_link(next_page_num) }}">Next</a>
      {% endif %}
    </nav>
  {% endif %}
</div>

{# Sidebar #}
<aside class="blog-sidebar">
  {# Recent posts #}
  <div class="widget">
    <h3>Recent Posts</h3>
    {% set recent = blog_recent_posts(group.id, 5) %}
    <ul>
      {% for post in recent %}
        <li><a href="{{ post.absolute_url }}">{{ post.name }}</a></li>
      {% endfor %}
    </ul>
  </div>

  {# Tags #}
  <div class="widget">
    <h3>Topics</h3>
    {% set topics = blog_topics(group.id, 20) %}
    <ul>
      {% for topic in topics %}
        <li>
          <a href="{{ blog_tag_url(group.id, topic.slug) }}">
            {{ topic.name }}
          </a>
        </li>
      {% endfor %}
    </ul>
  </div>
</aside>
{% endblock %}
```

## System Templates

### 404 Error Template

```hubl
<!--
  templateType: error_page
  label: 404 Page
-->

{% extends "layouts/base.html" %}

{% block body %}
<div class="error-page">
  <h1>Page Not Found</h1>
  <p>The page you're looking for doesn't exist.</p>
  <a href="/" class="btn">Go Home</a>

  {# Site search #}
  {% module "search" path="@hubspot/search_input" %}
</div>
{% endblock %}
```

### 500 Error Template

```hubl
<!--
  templateType: error_page
  label: 500 Page
-->

{% extends "layouts/base.html" %}

{% block body %}
<div class="error-page">
  <h1>Something Went Wrong</h1>
  <p>We're sorry, but something went wrong. Please try again later.</p>
  <a href="/" class="btn">Go Home</a>
</div>
{% endblock %}
```

### Password Prompt Template

```hubl
<!--
  templateType: password_prompt
  label: Password Prompt
-->

{% extends "layouts/base.html" %}

{% block body %}
<div class="password-page">
  <h1>Password Required</h1>
  <p>This page is password protected.</p>

  {% password_prompt "password_form" %}
</div>
{% endblock %}
```

### Search Results Template

```hubl
<!--
  templateType: search_results
  label: Search Results
-->

{% extends "layouts/base.html" %}

{% block body %}
<div class="search-results">
  <h1>Search Results</h1>

  {% module "search_input" path="@hubspot/search_input" %}

  {% if search_results %}
    <p>Found {{ search_results.total }} results for "{{ search_results.query }}"</p>

    {% for result in search_results.results %}
      <article class="search-result">
        <h2><a href="{{ result.url }}">{{ result.title }}</a></h2>
        <p>{{ result.description|truncate(200) }}</p>
      </article>
    {% endfor %}

    {# Pagination #}
    {% if search_results.pages > 1 %}
      <nav class="search-pagination">
        {% for page in range(1, search_results.pages + 1) %}
          <a href="{{ search_results.page_url(page) }}"
             {% if page == search_results.page %}class="active"{% endif %}>
            {{ page }}
          </a>
        {% endfor %}
      </nav>
    {% endif %}
  {% else %}
    <p>No results found.</p>
  {% endif %}
</div>
{% endblock %}
```

### Subscription Preferences

```hubl
<!--
  templateType: subscription_preferences
  label: Email Preferences
-->

{% extends "layouts/base.html" %}

{% block body %}
<div class="subscription-page">
  <h1>Email Preferences</h1>
  {% subscription_preferences "subscription_form" %}
</div>
{% endblock %}
```

## Layouts

### Base Layout

`templates/layouts/base.html`:

```hubl
<!DOCTYPE html>
<html lang="{{ html_lang }}" {{ html_lang_dir }}>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{{ page_meta.html_title }}</title>
  <meta name="description" content="{{ page_meta.meta_description }}">

  {{ standard_header_includes }}

  {{ require_css(get_asset_url("css/main.css")) }}

  {% block head %}{% endblock %}
</head>
<body class="{{ builtin_body_classes }}">
  {% block header %}
    {% include "partials/header.html" %}
  {% endblock %}

  <main id="main-content">
    {% block body %}{% endblock %}
  </main>

  {% block footer %}
    {% include "partials/footer.html" %}
  {% endblock %}

  {{ require_js(get_asset_url("js/main.js"), "footer") }}

  {{ standard_footer_includes }}

  {% block footer_scripts %}{% endblock %}
</body>
</html>
```

### Extending Layouts

```hubl
{% extends "layouts/base.html" %}

{% block head %}
  {# Page-specific head content #}
  <link rel="stylesheet" href="{{ get_asset_url('css/home.css') }}">
{% endblock %}

{% block body %}
  {# Page content #}
{% endblock %}

{% block footer_scripts %}
  {# Page-specific scripts #}
  <script src="{{ get_asset_url('js/home.js') }}"></script>
{% endblock %}
```

### Using super()

Include parent block content:

```hubl
{% block body %}
  {{ super() }}  {# Includes parent block content #}
  <p>Additional content</p>
{% endblock %}
```

## Partials

### Header Partial

`templates/partials/header.html`:

```hubl
<header class="site-header">
  <div class="container">
    {% module "logo" path="@hubspot/logo" %}

    <nav class="main-nav">
      {% module "menu"
        path="@hubspot/menu",
        menu="default"
      %}
    </nav>

    {% module "cta"
      path="@hubspot/cta",
      label="Header CTA"
    %}
  </div>
</header>
```

### Footer Partial

`templates/partials/footer.html`:

```hubl
<footer class="site-footer">
  <div class="container">
    <div class="footer-columns">
      <div class="footer-column">
        {% module "footer_logo" path="@hubspot/logo" %}
        {% module "footer_text" path="@hubspot/rich_text" %}
      </div>

      <div class="footer-column">
        {% module "footer_menu"
          path="@hubspot/menu",
          menu="footer"
        %}
      </div>
    </div>

    <div class="footer-bottom">
      <p>&copy; {{ now()|datetimeformat("%Y") }} Company Name</p>
    </div>
  </div>
</footer>
```

### Including Partials

```hubl
{% include "partials/header.html" %}
{% include "partials/footer.html" %}

{# Without context (isolated) #}
{% include "partials/component.html" without context %}

{# With additional context #}
{% include "partials/card.html" with context %}
```

## Drag and Drop Areas

### Basic DnD Area

```hubl
{% dnd_area "main_content" label="Main Content" %}
  {# Default content #}
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

### DnD Section with Styling

```hubl
{% dnd_section
  background_color={"color": "#f5f5f5"},
  padding={
    "top": 60,
    "bottom": 60,
    "left": 20,
    "right": 20
  },
  max_width=1200,
  margin={
    "top": 0,
    "bottom": 0
  }
%}
```

### Multi-Column Layout

```hubl
{% dnd_section %}
  {% dnd_column width=4 %}
    {% dnd_row %}
      {% dnd_module path="@hubspot/rich_text" %}{% end_dnd_module %}
    {% end_dnd_row %}
  {% end_dnd_column %}

  {% dnd_column width=4 %}
    {% dnd_row %}
      {% dnd_module path="@hubspot/rich_text" %}{% end_dnd_module %}
    {% end_dnd_row %}
  {% end_dnd_column %}

  {% dnd_column width=4 %}
    {% dnd_row %}
      {% dnd_module path="@hubspot/rich_text" %}{% end_dnd_module %}
    {% end_dnd_row %}
  {% end_dnd_column %}
{% end_dnd_section %}
```

## Template Variables

### Page Variables

| Variable | Description |
|----------|-------------|
| `content.name` | Page title |
| `content.absolute_url` | Full URL |
| `content.meta_description` | Meta description |
| `content.featured_image` | Featured image URL |
| `page_meta.html_title` | HTML title tag |

### Blog Variables

| Variable | Description |
|----------|-------------|
| `content.publish_date` | Publish timestamp |
| `content.publish_date_localized` | Formatted date |
| `content.blog_author` | Author object |
| `content.tag_list` | Post tags |
| `content.post_body` | Post content |
| `content.post_summary` | Excerpt |
| `group.id` | Blog ID |
| `contents` | Posts in listing |
