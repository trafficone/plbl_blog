---
layout: navi
title: "Plant Blog"
date: 2020-01-30 00:00:00 -0000
categories: botany navigation
---

The Plant Blog is a blog about horticulture, bioinformatics, botany, and 
whatever else I grow in my garden.

Latest posts:
{% for post in site.posts limit:5 %}
* [{{ post.title }}]({{ post.url }})
{% endfor %}

For all posts go [here](/blog).

For more information about me, the blog, and contact info, go to the [about page](/about).

For contact information, [contact me](/contact).
