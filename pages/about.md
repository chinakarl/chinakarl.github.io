---
layout: page
title: About
description: 打码改变世界
keywords: Karl, 石头
comments: true
menu: 关于
permalink: /about/
---

我是karl。

仰慕「优雅编码的艺术」。


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
