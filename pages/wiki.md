---
layout: page
title: Wiki
description: 人越学越觉得自己无知
keywords: 维基, Wiki
comments: false
menu: 维基
permalink: /wiki/
---

> 此wiki中的全部文章均Fork自[mzlogin](https://github.com/mzlogin/mzlogin.github.io/tree/master/_wiki)，因为感觉写得非常好，Fork主题的时候就没有删除这些文章。如果这样侵权了，请联系我立即删除。

<ul class="listing">
{% for wiki in site.wiki %}
{% if wiki.title != "Wiki Template" %}
<li class="listing-item"><a href="{{ site.url }}{{ wiki.url }}">{{ wiki.title }}</a></li>
{% endif %}
{% endfor %}
</ul>
