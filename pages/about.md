---
layout: page
title: About
description: 技术服务社会
keywords: ssgao, 高硕
comments: true
menu: 关于
permalink: /about/
---

我是矿工，数据+算法提升生产力，探索数据财富。
仰慕「优雅编码的艺术」。
坚信熟能生巧，努力改变人生。

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'mazhuang.org' %}

{% endif %}
</ul>


## Skill Keywords


{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
