---
layout: default
---

<div class="home">

  <h1 class="page-heading">{{ page.title | escape }} {{ site.disqus.shortname | escape}}</h1>

  {{ content }}

  <ul class="post-list">
    {% for post in site.posts %}
    	{% if post.tags contains page.tag %}
	      <li>
	        {% assign date_format = site.minima.date_format | default: "%b %-d, %Y" %}
	        <span class="post-meta">{{ post.date | date: date_format }}</span>

	        <h2>
	          <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
	        </h2>
	      </li>
      	{% endif %}
    {% endfor %}
  </ul>

</div>