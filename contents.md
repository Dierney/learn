---
layout: page
title: 目录
permalink: /contents
---

{% for tag in site.categories %}
  <h2 id="{{ tag[0] }}">{{ tag[0] | capitalize }}</h2>
  <ul>
    {% assign pages_list = tag[1] %}
    {% for post in pages_list %}
      {% if post.title != null %}
      {% if group == null or group == post.group %}
      <li>
        <a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title }}
        <span class="entry-date">
            - <time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date: "%b %d, %Y" }}</time>
        </span></a>
      </li>
      {% endif %}
      {% endif %}
    {% endfor %}
    {% assign pages_list = nil %}
    {% assign group = nil %}
  </ul>
  <br>
{% endfor %}