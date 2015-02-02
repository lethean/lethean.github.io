---
layout: default
title: 전체글
permalink: /archive/
---

<article>
	<ul>
	  {% for post in site.posts %}
	    <li data-index="{{ post.title | downcase }}">
	      <a href="{{ post.url }}">{{ post.title }}</a>
	      - <span class="date">{{ post.date | date: "%Y년 %m월 %d일" }}</span>
	    </li>
	  {% endfor %}
	</ul>
</article>

