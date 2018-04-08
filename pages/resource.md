---
layout: page
title: 收藏
description: 人越学越觉得自己无知
keywords: 收藏
comments: false
menu: 收藏
permalink: /resource/
---

<ul class="listing">
{% for resource in site.resources %}
<li class="listing-item"><a href="{{ site.url }}{{ resource.url }}" target="_blank">{{ resource.title }}</a></li>
{% endfor %}
</ul>
