---
layout: page
title: Hall of Fame
menu: main
permalink: /halloffame/
---

{% for category in site.categories %}
{% assign cat_name =  category | first %}
{% if cat_name == "Hall Of Fame" %}
{% for posts in category %}
{% for post in posts %}
<p><h1><a href="{{ post.url }}">{{ post.title }}</a></h1></p>
<p>{{ post.excerpt }}</p>
{% endfor %}
{% endfor %}
{% endif %}
{% endfor %}