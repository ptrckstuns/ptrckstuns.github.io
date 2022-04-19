---
layout: page
title: Blog
subtitle: Inside of my Ryouchi (りょうち).
---

<div>
{% assign postsCategory = site.posts | group_by_exp:"post", "post.categories"  %}
{% for category in postsCategory %}
<h4 class="post-teaser__month">
<strong>
{% if category.name %} 
    {% if category.name contains 'secret' %}
    {% else %}
    - - - - -  {{ category.name }} - - - - - 
    {% endif %}
{% else %} 
{{ Print }} 
{% endif %}
</strong>
</h4>
<ul class="list-posts">
{% for post in category.items %}
{% if category.name contains 'secret' %}
{% else %}
<li class="post-teaser">
<a href="{{ post.url | prepend: site.baseurl }}">
<span class="post-teaser__title">{{ post.title }}</span>
<span class="post-teaser__date">{{ post.date | date: "%d %B %Y" }}</span>
</a>
</li>
{% endif %}
{% endfor %}
</ul>
{% endfor %}
</div>