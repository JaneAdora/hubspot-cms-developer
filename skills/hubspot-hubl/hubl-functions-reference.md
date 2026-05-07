# HubL Functions Reference

Functions are called using `{{ function_name() }}` syntax.

## Content Functions

### blog_all_posts_url(blog_id)
Get URL for all posts page.
```hubl
{{ blog_all_posts_url(blog.id) }}
```

### blog_author_url(blog_id, author_slug)
Get author archive URL.
```hubl
{{ blog_author_url(blog.id, author.slug) }}
```

### blog_page_link(page)
Get paginated blog URL.
```hubl
{{ blog_page_link(2) }}
```

### blog_popular_posts(blog_id, limit, tag_slug, time_frame)
Get popular posts.
```hubl
{% set popular = blog_popular_posts(blog.id, 5) %}
{% set popular_this_month = blog_popular_posts(blog.id, 5, "", "popular_this_month") %}
```

### blog_post_archive_url(blog_id, year, month)
Get archive URL.
```hubl
{{ blog_post_archive_url(blog.id, 2024, 1) }}
```

### blog_recent_author_posts(blog_id, author_slug, limit)
Get recent posts by author.
```hubl
{% set posts = blog_recent_author_posts(blog.id, author.slug, 5) %}
```

### blog_recent_posts(blog_id, limit)
Get recent blog posts.
```hubl
{% set recent = blog_recent_posts(blog.id, 10) %}
{% for post in recent %}
  <a href="{{ post.absolute_url }}">{{ post.name }}</a>
{% endfor %}
```

### blog_recent_tag_posts(blog_id, tag_slug, limit)
Get recent posts with tag.
```hubl
{% set tagged = blog_recent_tag_posts(blog.id, "news", 5) %}
```

### blog_tag_url(blog_id, tag_slug)
Get tag archive URL.
```hubl
{{ blog_tag_url(blog.id, tag.slug) }}
```

### blog_topics(blog_id, limit)
Get blog topics/tags.
```hubl
{% set topics = blog_topics(blog.id, 50) %}
```

### blog_total_post_count(blog_id)
Get total post count.
```hubl
{{ blog_total_post_count(blog.id) }}
```

### content_by_id(content_id)
Get content by ID.
```hubl
{% set page = content_by_id(12345) %}
{{ page.name }}
```

### content_by_ids(content_ids)
Get multiple content items.
```hubl
{% set pages = content_by_ids([123, 456, 789]) %}
```

### crm_associations(object_id, association_type, query)
Get CRM associations.
```hubl
{% set contacts = crm_associations(deal.id, "deal_to_contact") %}
```

### crm_object(object_type, id, properties)
Get CRM object.
```hubl
{% set contact = crm_object("contact", vid, "firstname,lastname,email") %}
{{ contact.firstname }} {{ contact.lastname }}
```

### crm_objects(object_type, query)
Query CRM objects.
```hubl
{% set deals = crm_objects("deal", "limit=10&status=open") %}
```

### hubdb_table(table_id)
Get HubDB table info.
```hubl
{% set table = hubdb_table(123456) %}
```

### hubdb_table_column(table_id, column_id)
Get HubDB column.
```hubl
{% set column = hubdb_table_column(123456, 1) %}
```

### hubdb_table_row(table_id, row_id)
Get single HubDB row.
```hubl
{% set row = hubdb_table_row(123456, 1) %}
```

### hubdb_table_rows(table_id, query)
Query HubDB table.
```hubl
{% set rows = hubdb_table_rows(123456, "category=news&orderBy=date") %}
{% for row in rows %}
  {{ row.title }}
{% endfor %}
```

### menu(menu_id)
Get menu data.
```hubl
{% set nav = menu("main-navigation") %}
{% for item in nav.children %}
  <a href="{{ item.url }}">{{ item.label }}</a>
{% endfor %}
```

### module_asset_url(path)
Get module asset URL.
```hubl
{{ module_asset_url("images/icon.png") }}
```

### personalization_token(token, default)
Get personalization token value.
```hubl
{{ personalization_token("contact.firstname", "there") }}
```

## URL Functions

### get_asset_url(path)
Get theme asset URL.
```hubl
<link rel="stylesheet" href="{{ get_asset_url('css/main.css') }}">
<script src="{{ get_asset_url('js/app.js') }}"></script>
```

### get_public_template_url(path)
Get template URL.
```hubl
{{ get_public_template_url("templates/page.html") }}
```

### get_public_template_url_by_id(template_id)
Get template URL by ID.
```hubl
{{ get_public_template_url_by_id(12345) }}
```

### page_by_id(page_id)
Get page data.
```hubl
{% set page = page_by_id(12345) %}
<a href="{{ page.absolute_url }}">{{ page.name }}</a>
```

### resize_image_url(url, width, height, length)
Resize image URL.
```hubl
{{ resize_image_url(image.src, 800, 600) }}
{{ resize_image_url(image.src, 0, 400) }}  {# height only #}
{{ resize_image_url(image.src, 800, 0) }}  {# width only #}
```

### site_url(path)
Build site-relative URL.
```hubl
{{ site_url("/contact") }}
```

## Utility Functions

### cta(cta_id, align)
Render CTA.
```hubl
{{ cta("uuid-of-cta") }}
{{ cta("uuid-of-cta", "center") }}
```

### datetimeformat(value, format, timezone)
Format datetime (function version).
```hubl
{{ datetimeformat(content.publish_date, "%Y-%m-%d") }}
```

### format_date(value, format)
Format date.
```hubl
{{ format_date(date, "MMMM d, yyyy") }}
```

### geo_distance(lat1, lon1, lat2, lon2, unit)
Calculate distance.
```hubl
{{ geo_distance(40.7128, -74.0060, 34.0522, -118.2437, "mi") }}
```

### include_css(path)
Include CSS file.
```hubl
{{ include_css(get_asset_url("css/module.css")) }}
```

### include_javascript(path)
Include JavaScript file.
```hubl
{{ include_javascript(get_asset_url("js/module.js")) }}
```

### locale_name(locale_code)
Get locale display name.
```hubl
{{ locale_name("en-us") }}  {# English (United States) #}
```

### now()
Get current timestamp.
```hubl
{% set current_year = now()|datetimeformat("%Y") %}
```

### oembed(url)
Get oEmbed data.
```hubl
{% set embed = oembed("https://youtube.com/watch?v=...") %}
{{ embed.html }}
```

### postal_location(postal_code, country)
Get location from postal code.
```hubl
{% set location = postal_location("10001", "US") %}
```

### range(start, end, step)
Generate number sequence.
```hubl
{% for i in range(1, 11) %}
  {{ i }}
{% endfor %}

{% for i in range(0, 10, 2) %}
  {{ i }}  {# 0, 2, 4, 6, 8 #}
{% endfor %}
```

### render_form(form_id)
Render HubSpot form.
```hubl
{{ render_form("form-uuid") }}
```

### require_css(url)
Require CSS (adds to head).
```hubl
{{ require_css(get_asset_url("css/styles.css")) }}
```

### require_js(url, position)
Require JavaScript.
```hubl
{{ require_js(get_asset_url("js/script.js"), "footer") }}
```

### super()
Call parent block content.
```hubl
{% block content %}
  {{ super() }}
  <p>Additional content</p>
{% endblock %}
```

### today()
Get current date.
```hubl
{% set current_date = today() %}
```

### to_json(object)
Convert to JSON.
```hubl
<script>
  var config = {{ to_json(module.config) }};
</script>
```

### truncate(text, length, end)
Truncate text (function version).
```hubl
{{ truncate(content.post_body, 200, "...") }}
```

### type(value)
Get value type.
```hubl
{{ type(variable) }}  {# string, number, list, dict, etc. #}
```

### unixtimestamp()
Get current Unix timestamp.
```hubl
{{ unixtimestamp() }}
```

### year(timestamp)
Get year from timestamp.
```hubl
{{ year(content.publish_date) }}
```

## Color Functions

### color_variant(color, amount)
Lighten or darken color.
```hubl
{{ color_variant("#ff0000", 20) }}   {# lighter #}
{{ color_variant("#ff0000", -20) }}  {# darker #}
```

## Request/Context Functions

### request_contact()
Get current contact (if known).
```hubl
{% set contact = request_contact() %}
{% if contact %}
  Welcome, {{ contact.firstname }}!
{% endif %}
```

### request.cookies
Access cookies.
```hubl
{{ request.cookies.cookie_name }}
```

### request.headers
Access request headers.
```hubl
{{ request.headers.user_agent }}
```

### request.path
Get current path.
```hubl
{% if request.path == "/contact" %}
  Contact page
{% endif %}
```

### request.query_dict
Access query parameters.
```hubl
{{ request.query_dict.utm_source }}
```

### request.referrer
Get referrer URL.
```hubl
{{ request.referrer }}
```

## Standard Library Functions

### abs(number)
Absolute value.
```hubl
{{ abs(-5) }}  {# 5 #}
```

### dict()
Create dictionary.
```hubl
{% set data = dict(name="John", age=30) %}
```

### float(value)
Convert to float.
```hubl
{{ float("3.14") }}
```

### int(value)
Convert to integer.
```hubl
{{ int("42") }}
```

### list()
Create list.
```hubl
{% set items = list() %}
```

### max(values)
Get maximum.
```hubl
{{ max(1, 5, 3) }}  {# 5 #}
{{ max(items) }}
```

### min(values)
Get minimum.
```hubl
{{ min(1, 5, 3) }}  {# 1 #}
```

### str(value)
Convert to string.
```hubl
{{ str(42) }}
```
