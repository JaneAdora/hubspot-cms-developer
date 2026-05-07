# HubL Filters Reference

Filters transform values using the pipe `|` syntax: `{{ value|filter }}`

## String Filters

### capitalize
Capitalize first character.
```hubl
{{ "hello"|capitalize }}  {# Hello #}
```

### center(width)
Center in a string of given width.
```hubl
{{ "hi"|center(10) }}  {# "    hi    " #}
```

### cut(chars)
Remove specified characters.
```hubl
{{ "hello world"|cut(" ") }}  {# helloworld #}
```

### escape / e
HTML-escape special characters.
```hubl
{{ "<script>"|escape }}  {# &lt;script&gt; #}
```

### escape_html
Convert &, <, >, ', " to HTML-safe sequences.
```hubl
{{ content|escape_html }}
```

### escape_jinjava
Escape Jinja/HubL syntax.
```hubl
{{ "{{ var }}"|escape_jinjava }}
```

### format
Python-style string formatting.
```hubl
{{ "Hello, %s!"|format(name) }}
```

### indent(width, first=false)
Indent lines.
```hubl
{{ text|indent(4) }}
{{ text|indent(4, true) }}  {# indent first line too #}
```

### ipaddr
Validate IP address.
```hubl
{{ ip|ipaddr }}
```

### lower
Convert to lowercase.
```hubl
{{ "HELLO"|lower }}  {# hello #}
```

### md5
Generate MD5 hash.
```hubl
{{ email|lower|md5 }}  {# for Gravatar #}
```

### pprint
Pretty-print for debugging.
```hubl
{{ complex_object|pprint }}
```

### regex_replace(pattern, replacement)
Replace using regex.
```hubl
{{ text|regex_replace("[^a-z]", "") }}
```

### replace(old, new)
Replace substring.
```hubl
{{ "hello"|replace("l", "L") }}  {# heLLo #}
```

### safe
Mark string as safe HTML.
```hubl
{{ html_content|safe }}
```

### split(delimiter)
Split string into list.
```hubl
{{ "a,b,c"|split(",") }}  {# ["a", "b", "c"] #}
```

### striptags
Remove HTML tags.
```hubl
{{ html|striptags }}
```

### title
Title Case.
```hubl
{{ "hello world"|title }}  {# Hello World #}
```

### trim
Remove leading/trailing whitespace.
```hubl
{{ "  hello  "|trim }}  {# "hello" #}
```

### truncate(length, killwords=false, end='...')
Truncate text.
```hubl
{{ text|truncate(100) }}
{{ text|truncate(100, false, '...') }}
{{ text|truncate(100, true) }}  {# cut at exact length #}
```

### truncatehtml(length, end='...')
Truncate HTML safely.
```hubl
{{ html|truncatehtml(200) }}
```

### upper
Convert to uppercase.
```hubl
{{ "hello"|upper }}  {# HELLO #}
```

### urlencode
URL-encode string.
```hubl
{{ "hello world"|urlencode }}  {# hello%20world #}
```

### wordcount
Count words.
```hubl
{{ text|wordcount }}
```

### wordwrap(width)
Wrap text at width.
```hubl
{{ text|wordwrap(80) }}
```

## List Filters

### attr(name)
Get attribute from objects in list.
```hubl
{{ users|attr("name") }}
```

### batch(count, fill=none)
Group items into batches.
```hubl
{% for row in items|batch(3) %}
  {% for item in row %}{{ item }}{% endfor %}
{% endfor %}
```

### difference(list)
Items in first list not in second.
```hubl
{{ [1,2,3]|difference([2,3,4]) }}  {# [1] #}
```

### first
Get first item.
```hubl
{{ items|first }}
```

### groupby(attribute)
Group by attribute.
```hubl
{% for group in posts|groupby("author") %}
  <h2>{{ group.grouper }}</h2>
  {% for post in group.list %}{{ post.title }}{% endfor %}
{% endfor %}
```

### intersect(list)
Common items between lists.
```hubl
{{ [1,2,3]|intersect([2,3,4]) }}  {# [2, 3] #}
```

### join(separator)
Join list into string.
```hubl
{{ ["a", "b", "c"]|join(", ") }}  {# a, b, c #}
```

### last
Get last item.
```hubl
{{ items|last }}
```

### length
Get length.
```hubl
{{ items|length }}
```

### list
Convert to list.
```hubl
{{ "abc"|list }}  {# ["a", "b", "c"] #}
```

### map(attribute)
Extract attribute from each item.
```hubl
{{ users|map("name")|join(", ") }}
```

### random
Get random item.
```hubl
{{ items|random }}
```

### reject(test)
Filter out items matching test.
```hubl
{{ items|reject("none") }}
```

### rejectattr(attr, test)
Filter by attribute test.
```hubl
{{ posts|rejectattr("is_draft", "true") }}
```

### reverse
Reverse order.
```hubl
{{ items|reverse }}
```

### select(test)
Keep items matching test.
```hubl
{{ items|select("number") }}
```

### selectattr(attr, test, value)
Filter by attribute value.
```hubl
{{ posts|selectattr("published", "equalto", true) }}
```

### shuffle
Randomly shuffle list.
```hubl
{{ items|shuffle }}
```

### slice(count)
Slice into n lists.
```hubl
{% for column in items|slice(3) %}
  <div class="col">
    {% for item in column %}{{ item }}{% endfor %}
  </div>
{% endfor %}
```

### sort(reverse=false, attribute=none)
Sort list.
```hubl
{{ items|sort }}
{{ items|sort(true) }}  {# reverse #}
{{ posts|sort(false, "date") }}  {# by attribute #}
```

### sum(attribute=none)
Sum numbers.
```hubl
{{ [1, 2, 3]|sum }}  {# 6 #}
{{ orders|sum("total") }}
```

### symmetric_difference(list)
Items in either list but not both.
```hubl
{{ [1,2,3]|symmetric_difference([2,3,4]) }}  {# [1, 4] #}
```

### union(list)
Combine lists, remove duplicates.
```hubl
{{ [1,2]|union([2,3]) }}  {# [1, 2, 3] #}
```

### unique
Remove duplicates.
```hubl
{{ [1, 1, 2, 2, 3]|unique }}  {# [1, 2, 3] #}
```

## Number Filters

### abs
Absolute value.
```hubl
{{ -5|abs }}  {# 5 #}
```

### divide_by(n)
Divide.
```hubl
{{ 10|divide_by(2) }}  {# 5 #}
```

### filesizeformat
Human-readable file size.
```hubl
{{ 1024|filesizeformat }}  {# 1.0 KB #}
```

### float
Convert to float.
```hubl
{{ "3.14"|float }}
```

### format_currency(locale, currency)
Format as currency.
```hubl
{{ 1234.56|format_currency("en-US", "USD") }}  {# $1,234.56 #}
```

### int
Convert to integer.
```hubl
{{ "42"|int }}
```

### minus(n)
Subtract.
```hubl
{{ 10|minus(3) }}  {# 7 #}
```

### multiply(n)
Multiply.
```hubl
{{ 5|multiply(3) }}  {# 15 #}
```

### plus(n)
Add.
```hubl
{{ 5|plus(3) }}  {# 8 #}
```

### round(precision, method)
Round number.
```hubl
{{ 3.14159|round(2) }}  {# 3.14 #}
{{ 3.5|round(0, "floor") }}  {# 3 #}
{{ 3.5|round(0, "ceil") }}  {# 4 #}
```

## Date Filters

### datetimeformat(format, timezone)
Format date/time.
```hubl
{{ content.publish_date|datetimeformat("%B %d, %Y") }}
{{ content.publish_date|datetimeformat("%Y-%m-%d", "America/New_York") }}
```

**Format codes:**
- `%Y` - 4-digit year
- `%m` - 2-digit month
- `%d` - 2-digit day
- `%B` - Full month name
- `%b` - Abbreviated month name
- `%H` - Hour (24h)
- `%I` - Hour (12h)
- `%M` - Minute
- `%S` - Second
- `%p` - AM/PM

### strtodate(format)
Parse string to date.
```hubl
{{ "2024-01-15"|strtodate("%Y-%m-%d") }}
```

### strtotime(format)
Parse string to timestamp.
```hubl
{{ "14:30"|strtotime("%H:%M") }}
```

### unixtimestamp
Convert to Unix timestamp.
```hubl
{{ content.publish_date|unixtimestamp }}
```

## Object/Dict Filters

### attr(name)
Get attribute by name.
```hubl
{{ object|attr("property_name") }}
```

### dictsort
Sort dictionary by key.
```hubl
{% for key, value in dict|dictsort %}
```

### items
Get dict items for iteration.
```hubl
{% for key, value in dict|items %}
```

### keys
Get dictionary keys.
```hubl
{% for key in dict|keys %}
```

### tojson
Convert to JSON string.
```hubl
<script>var data = {{ object|tojson }};</script>
```

### values
Get dictionary values.
```hubl
{% for value in dict|values %}
```

## Default/Fallback Filters

### default(value)
Provide fallback for undefined/empty.
```hubl
{{ variable|default("fallback") }}
{{ content.meta|default("No description") }}
```

### none
Render nothing if none.
```hubl
{{ variable|none }}
```

## HubSpot-Specific Filters

### between_times(start, end)
Check if time is between range.
```hubl
{{ time|between_times(9, 17) }}
```

### blog_author_url
Get author archive URL.
```hubl
{{ author.slug|blog_author_url(blog.id) }}
```

### blog_tag_url
Get tag archive URL.
```hubl
{{ tag.slug|blog_tag_url(blog.id) }}
```

### convert_rgb(format)
Convert color format.
```hubl
{{ "#ff0000"|convert_rgb }}
```

### fromjson
Parse JSON string to object.
```hubl
{% set data = json_string|fromjson %}
```

### geo_distance(lat1, lon1, lat2, lon2, unit)
Calculate distance between coordinates.
```hubl
{{ geo_distance(lat1, lon1, lat2, lon2, "mi") }}
```

### render
Render a string as HubL.
```hubl
{{ "Hello {{ name }}"|render }}
```

### root_url
Get root URL from full URL.
```hubl
{{ "https://example.com/page"|root_url }}  {# https://example.com #}
```

### sanitize_html
Sanitize HTML content.
```hubl
{{ user_input|sanitize_html }}
```

### to_local_time(timezone)
Convert to local timezone.
```hubl
{{ event_time|to_local_time("America/New_York") }}
```

### url_type
Get HubSpot content type from URL.
```hubl
{{ request.path|url_type }}
```
