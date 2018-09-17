---
title: Flask Web开发笔记-第3章模板（3.3-3.6）
date: 2018-09-17 16:06:49
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

**Flask笔记内容**
- 3.3 自定义错误页面
- 3.4 链接
- 3.5 静态文件
- 3.6 使用Flask-Moment本地化日期和时间

<!--more-->


## 3.3 自定义错误页面
Flask允许程序使用基于模板的自定义错误页面。
```python
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404

@app.errorhandler(500)
def page_not_found(e):
    return render_template('500.html'), 500
```
404页面如下
```HTML
{% extends 'extend.html' %}
{% block title %}404 Page Not Found{% endblock %}

{% block body %}
<P>404 PAGE NOT FOUND</P>
{% endblock %}
```
上面虽然用了实现了页面，但自定义错误页面要分别修改页面头部元素，这种方式比较麻烦。Jinja2的模板继承机制中，变自定义一个带有样式的模板。自定义个base.html模板，继承bootstrap/base.html。修改后，代码如下

自定义的base.html
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
        {% block page_content %}{% endblock %}
    </div>
{% endblock %}
```
修改后的404.html页面
```HTML
{% extends 'base.html' %}
{% block title %}404 Page Not Found{% endblock %}
{% block page_content %}
    <div class="page-header">
        <h1>404 Page Not Found</h1>
    </div>
{% endblock %}
```

修改后的user.html页面
```HTML
{% extends 'base.html' %}
{% block page_content %}
    <div class="page-header">
        <h1>Hello {{ username }}</h1>
    </div>
{% endblock %}
```

## 3.4 链接
对于包含可变部分的动态路由，在模板中构建正确的URL就很困难。而且，直接编写 URL 会对代码中定义的路由产生不必要的依赖关系。如果重新定义路由，模板中的链接可能会失效。为了避免这些问题，Flask提供了url_for()辅助函数，它可以使用程序 URL 映射中保存的信息生成 URL。

url_for()函数最简单的方法是以试图函数命名（或者 app.add_url_route() 定义路由时使用
的端点名）作为参数，返回对应的URL，例子如下：
```python
#! usr/bin/env python3
# -*- coding:utf-8 -*-

from flask import Flask,url_for
from flask import render_template
from flask import redirect
from flask_bootstrap import Bootstrap

app = Flask(__name__)#type:Flask
bootstrap = Bootstrap(app)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/user/<name>')
def user(name):
    return render_template('user.html', username=name)

@app.route('/login')
def login():
    # return redirect(url_for('index', _external=True))
    # return redirect(url_for('index'))
    # return redirect(url_for('user', name='test'))#http://127.0.0.1:5000/user/test
    return redirect(url_for('index', name='who'))#http://127.0.0.1:5000/?name=who


@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404

@app.errorhandler(500)
def page_not_found(e):
    return render_template('500.html'), 500

if __name__=='__main__':
    app.run(debug=True)
```
在login()方法中，调用了url_for()来实现动态路由，其中_external=True表示返回绝对地址。以下是带有参数跳转的两种形式。
    
    return redirect(url_for('user', name='test'))#http://127.0.0.1:5000/user/test
    return redirect(url_for('index', name='who'))#http://127.0.0.1:5000/?name=who
## 3.5 静态文件
略
## 3.6 使用Flask-Moment本地化日期和时间
略
