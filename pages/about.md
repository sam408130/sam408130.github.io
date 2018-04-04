---
layout: page
title: About
description: Sam Blog
keywords: Sam Ding
comments: true
menu: 关于
permalink: /about/
---

2013年毕业于UNSW，Signal Processing专业

2013 ~ 2016年就职于科大讯飞清华大学实验室，从事异构网络模型以及排序算法应用研究

2016至今，front-end developer

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
