---
layout: page
title: Categories
permalink: /categories/
---

Pick a topic, then jump into articles.

## Topic index

- [Java / JDK](#java--jdk)
- [AI / Harness / Add-ons](#ai--harness--add-ons)
- [Tooling / Plugins](#tooling--plugins)
- [All site categories](#all-site-categories)

## Java / JDK

{% for post in site.posts %}
{% assign cats = post.categories %}
{% if cats contains 'java' or cats contains 'spring' or cats contains 'maven' or cats contains 'antlr' or cats contains 'memory' or cats contains 'testing' %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endif %}
{% endfor %}

## AI / Harness / Add-ons

{% for post in site.posts %}
{% assign cats = post.categories %}
{% if cats contains 'ai' or cats contains 'projects' or cats contains 'anki' or cats contains 'security' %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endif %}
{% endfor %}

## Tooling / Plugins

{% for post in site.posts %}
{% assign cats = post.categories %}
{% if cats contains 'tooling' or cats contains 'opencode' or cats contains 'maven' or cats contains 'android' %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endif %}
{% endfor %}

## All site categories

{% assign sorted_categories = site.categories | sort %}
{% for catitem in sorted_categories %}
### {{ catitem[0] }}

{% for post in catitem[1] %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

{% endfor %}
