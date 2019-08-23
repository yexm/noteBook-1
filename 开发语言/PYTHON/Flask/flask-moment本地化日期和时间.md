---
title: 使用Flask-Moment本地化日期和时间 
tags: python,flask
grammar_abbr: true
grammar_table: true
grammar_defList: true
grammar_emoji: true
grammar_footnote: true
grammar_ins: true
grammar_mark: true
grammar_sub: true
grammar_sup: true
grammar_checkbox: true
grammar_mathjax: true
grammar_flow: true
grammar_sequence: true
grammar_plot: true
grammar_code: true
grammar_highlight: true
grammar_html: true
grammar_linkify: true
grammar_typographer: true
grammar_video: true
grammar_audio: true
grammar_attachment: true
grammar_mermaid: true
grammar_classy: true
grammar_cjkEmphasis: true
grammar_cjkRuby: true
grammar_center: true
grammar_align: true
grammar_tableExtra: true
---

# 使用Flask-Moment本地化时间和日期
## 目的

服务器端使用UTC时间，通过浏览器转换为用户本地时间。

## 使用方法

``` bash
pip install flask-moment
```

初始化实例

``` python
from flask.ext.moment import Moment
moment = Moment(app)
```
在模板中引入moment.js

``` jinja
{% block scripts %}
{{ super() }}
{{ moment.include_moment() }}
{% endblock %}


<p>The local date and time is {{ moment(current_time).format('LLL') }}</p>
<p>That was {{ moment(current_time).fromNow(refresh=True) }}</p>
```