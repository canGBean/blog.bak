---
title: Flask Web开发笔记-第3章模板（3.1-3.2）
date: 2018-09-13 16:18:39
categories: 
- python
tags: 
- python
- flask
---


**关于Flask笔记**

个人Flask学习笔记，初步学习下Flask。
声明：

- 笔记内容整理自:O‘REILLY系列图书-《Flask Web开发：基于Python的Web应用开发实战》,本书由人民邮电出版社2015年出版。如果感觉有帮助，请购买正版
- 网上查询及搜集
- 笔记内容如有涉及版权问题，请联系我，我将删除相关内容

<!--more-->

## 3.1 Jinja2模板引擎
### 3.1.1渲染模板
默认情况下，Flask在程序文件夹中的templates子文件夹中寻找模板。在此文件夹中建立index.html与user.html页面，例子：

index.html
```HTML
<h1>Hello World!</h1>
```
user.html
```HTML
<h1>Hello, {{name}}!</h1>
```
hello.py
```python
#! usr/bin/env python3
# -*- coding:utf-8 -*-
from flask import Flask, render_template

app = Flask(__name__)#type:Flask

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/user/<name>')
def user(name):
    return render_template('user.html', name=name)

if __name__=='__main__':
    app.run(debug=True)
```

### 3.1.2变量
HTML中模板获取变量值
```HTML
<p>A value from a dictionary: {{ mydict['key'] }}.</p>
<p>A value from a list:{{ mylist[3] }}.</p>
<p>A value from a list, with a variable index: {{mylist[myintvar]}}.</p>
<p>A value from an object's method:{{myobj.somemethod()}}.</p>
<p>
    {% for i in mylist %}
        <h1>Hello,{{ i }}</h1>
    {% endfor %}
</p>
```
可以使用过滤器修改变量值，过滤器在变量名之后，使用竖线分隔
```HTML
<h1>Hello, {{username | capitalize}}!</h1>
```
Jinja2变量过滤器如下：

|过滤器名称|说明|
|-----------|----|
|safe|渲染时不转义|
|capitalize|把值的首字母转换成大写，其他字母转换成小写|
|lower|把值转换成小写|
|upper|把值转换成大写|
|title|把值中每个单词的首字母转换成大写|
|trim|把值的首位空格去掉|
|striptags|渲染前把值中所有的HTML标签都删掉|

需要注意的是，如果变量中存在HTML代码需要使用safe过滤器。完整的过滤器列表可在Jinja2文档
    http://jinja.pocoo.org/docs/templates/#builtin-filters
    
### 3.1.3控制结构
if语句
```HTML
{% if user %}
    Hello, {{ user }}!
{% else %}
    Hello, Stranger!
{% endif %}
```
for循环
```HTML
<ul>
    {% for comment in comments %}
        <li>{{ comment }}</li>
    {% endfor %}
</ul>
```
宏支持(略)：

页面包含
```HTML
{% include 'common.html' %}
```

模板继承，先创建一个base.html的模板
```HTML
<html>
    <head>
    {% block head %}
        <title>{% block title %}{% endblock %} - My Application</title>
    {% endblock %}
    </head>
    <body>
        {% block body %}
        {% endblock %}
    </body>
</html>
```
block标签定义的元素可在继承的模板中修改，定义一个extend.html文件，并配置跳转。
```HTML
{% extends "base.html" %}
{% block title %}Index{% endblock %}
{% block head %}
    {{ super() }}
    <style>
    </style>
{% endblock %}
{% block body %}
    <h1>Hello, World!</h1>
{% endblock %}
```
```python
@app.route('/extend')
def extend():
    return render_template('extend.html')
```
## 3.2使用Flask-Bootstrap集成Bootstrap
安装

    >pip install flask-bootstrap
安装后扩展路径默认为xxx\Python35\Lib\site-packages下。建立user.html页面，并调用bootstrap集成的base.html模板
```HTML

{% extends "bootstrap/base.html" %}

{% block title %}Flasky{% endblock %}
{% block navbar %}
    <div class="navbar navbar-inverse" role="navigation">
        <div class="container">
            <div class="navbar-header">
                <button type="button" class="navbar-toggle"
                        data-toggle="collapse" data-target=".navbar-collapse">
                    <span class="sr-only">Toggle navigation</span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                <a class="navbar-brand" href="/">Flasky</a>
            </div>
            <div class="navbar-collapse collapse">
                <ul class="nav navbar-nav">
                    <li><a href="/">Home</a></li>
                </ul>


            </div>

        </div>
    </div>
{% endblock %}
{% block content %}
    <div class="container">
        <div class="page-header">
            <h1>Hello, {{ username }}!</h1>
        </div>
    </div>
{% endblock %}
```
修改hello.py文件
```python
#! usr/bin/env python3
# -*- coding:utf-8 -*-

from flask import Flask
from flask import render_template
from flask_bootstrap import Bootstrap

app = Flask(__name__)#type:Flask
bootstrap = Bootstrap(app)

@app.route('/user/<name>')
def user(name):
    return render_template('user.html', username=name)

if __name__=='__main__':

    app.run(debug=True)
```
在集成的xxx\Python35\Lib\site-packages\flask_bootstrap\templates下可以查看base.html的内容，如下：
```HTML
{% block doc -%}
<!DOCTYPE html>
<html{% block html_attribs %}{% endblock html_attribs %}>
{%- block html %}
  <head>
    {%- block head %}
    <title>{% block title %}{{title|default}}{% endblock title %}</title>

    {%- block metas %}
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    {%- endblock metas %}

    {%- block styles %}
    <!-- Bootstrap -->
    <link href="{{bootstrap_find_resource('css/bootstrap.css', cdn='bootstrap')}}" rel="stylesheet">
    {%- endblock styles %}
    {%- endblock head %}
  </head>
  <body{% block body_attribs %}{% endblock body_attribs %}>
    {% block body -%}
    {% block navbar %}
    {%- endblock navbar %}
    {% block content -%}
    {%- endblock content %}

    {% block scripts %}
    <script src="{{bootstrap_find_resource('jquery.js', cdn='jquery')}}"></script>
    <script src="{{bootstrap_find_resource('js/bootstrap.js', cdn='bootstrap')}}"></script>
    {%- endblock scripts %}
    {%- endblock body %}
  </body>
{%- endblock html %}
</html>
{% endblock doc -%}
```
如上所示，flask-bootstrap基模板中定义的块如下：

|块名|说明|
|----|----|
|doc|整个HTML文档|
|html_attribs|&lt;html&gt;标签的属性|
|html|&lt;html&gt;标签中的内容|
|head|&lt;head&gt;标签中的内容|
|title|&lt;title&gt;标签中的内容|
|metas|一组&lt;meta&gt;标签|
|styles|层叠样式表定义|
|body_attribs|&lt;body&gt;标签的属性|
|body|&lt;body&gt;标签中的内容|
|navbar|用户定义的导航条|
|content|用户定义的页面内容|
|scripts|文档地步的JavaScript声明|

很多块都是flask-bootstrap自用的，如果直接重定义可能会导致一些问题。例如， Bootstrap所需的文件在styles和scripts块中声明。如果程序需要向已经有内容的块中添加新内容,必须使用 Jinja2 提供的super()函数。例如，如果要在衍生模板中添加新
的JavaScript文件，需要这么定义 scripts 块：
```HTML
{% block scripts %}
{{ super() }}
<script type="text/javascript" src="xxx.js"></script>
{% endblock %}
```


