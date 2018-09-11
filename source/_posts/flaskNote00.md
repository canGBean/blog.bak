---
title: Flask Web开发笔记-程序的基本结构
date: 2018-09-11 14:58:39
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

## 2.1初始化

```python    
    from flask import Flask
    app = Flask(__name__)
```
    
## 2.2路由和视图函数
使用程序实例提供的app.route修饰器，把修饰的函数注册为路由
    
```python
@app.route('/')
def index():
    return '<h1>Hello World</h1>'
```
在route修饰器中使用特殊的句法，实现动态URL
    
```python
@app.route('/user/<name>')
def user(name):
    return '<h1>Hello ,%s</h1>' % name
```

## 2.3启动服务器
```python
    if __name == '__main__':
        app.run(debug=True)
```
## 2.4完整的程序
```python
    
    from flask import Flask

    app = Flask(__name__)

    @app.route('/')
    def hello_world():
        # app.before_first_request(print(123))
        return 'Hello World!'

    @app.route('/user/<name>')
    def user(name):
        return '<h1>Hello ,%s</h1>' % name

    if __name__ == '__main__':
        app.run(debug=True)
```

## 2.5请求-响应
### 2.5.1程序和请求上下文
|变量名|上下文|说明|
|------|------|----|
|current_app|程序上下文|当前激活程序的程序实例|
|g|程序上下文|处理请求时作用临时存储的对象，每次请求都会重设这个变量|
|request|请求上下文|请求对象，封装了客户端发出的HTTP请求中的内容|
|session|请求上下文|用户会话，用于存储请求之间需要“记住”的值的词典|

Flask在分发请求之前激活(或推送)程序的请求上下文，请求处理完后再将其删除。程序上下文被推送后，就可以在线程中使用current_app和g变量。没激活程序上下文之前就调用current_app.name会导致错误，但推送完上下文之后就可以调用了。注意，在程序实例上调用 app.app_context() 可获得一个程序上下文。例子:
    
```python
    
    from flask import current_app
    
    # print('current_app.name:',current_app.name)
    app_ctx = app.app_context()
    app_ctx.push()
    current_app.name
    app_ctx.pop()
```
### 2.5.2请求调度
/和/user/&lt;name&gt; 路由在程序中使用 app.route 修饰器定义。/static/&lt;filename&gt;路由是Flask 添加的特殊路由，用于访问静态文件。例子：
    
```python
    if __name__ == '__main__':
        app_map = app.url_map
        app.run(debug=True)
    #Map([<Rule '/' (HEAD, OPTIONS, GET) -> index>,
    #<Rule '/static/<filename>' (HEAD, OPTIONS, GET) -> static>,
    #<Rule '/user/<name>' (HEAD, OPTIONS, GET) -> user>])
```
### 2.5.3请求钩子
Flask支持以下4种钩子
- before_first_request:注册一个函数，在处理第一个请求之前执行
- before_request:注册一个函数，在每次请求之前运行 
- after_request:注册一个函数，如果没有微处理的异常抛出，在每次请求之后运行
- teardown_request:注册一个函数，即使有未处理的异常抛出，也在每次请求后运行

在请求钩子函数和视图函数之间共享数据一般使用上下文全局变量g。例如，before_request处理程序可以从数据库中加载已登录用户，并将其保存到 g.user中。随后调用视图函数时，视图函数再使用 g.user 获取用户。例子：

```python
    @app.before_first_request
    def first():
        g.string = '123123'
        print('It is before_first_request')


    @app.before_request
    def before():
        print(g.string)
        print('app.before_request')


    @app.after_request
    def after(test):
        s = g
        print('after %s' % test)
        return test
```

### 2.5.4响应
如果视图返回的响应需要自定义的状态码，可以将状态码做为第二个参数返回。视图函数返回的响应还可接受第三个参数，这是一个由首部（ header）组成的字典，可以
添加到 HTTP 响应中，例子：
    
```python
    from flask import Flask

    app = Flask(__name__)

    @app.route('/')
    def index():
        return '<h1>Bad Request</h1>', 400, {'headertest':123123}

    if __name__ == '__main__':
        app.run(debug=True)
```
如果不想返回由1个、2个或3个值组成的元组，Flask 视图函数还可以返回 Response 对象。make_response() 函数可接受1个、2个或3个参数(和视图函数的返回值一样)，并返回一个Response对象。下例创建了一个响应对象，然后设置了cookie：

```python
    from flask import Flask
    from flask import make_response

    app = Flask(__name__)#type:Flask

    @app.route('/')
    def index():

        # return '<h1>Bad Request</h1>', 400, {'headertest':123123}
        response = make_response('<h1>This document carries a cookie!</h1>', 400)
        response.headers['aaa'] = '111'
        response.headers['bbb'] = '222'
        response.set_cookie('a', '123')
        return response

    if __name__ == '__main__':
        app.run(debug=True)
```
若系统使用重定向(302)，指向的地址由Location首部提供。重定向响应可以使用3个值形式的返回值生成，也可在Response对象中设定不过由于使用频繁Flask提供了 redirect() 辅助函数，用于生成这种响应：

```python
    from flask import Flask
    from flask import redirect
    @app.route('/')
    def index():
        return redirect('http://www.baidu.com')

    if __name__ == '__main__':
        app.run(debug=True)
```
还有一种特殊的响应由abort函数生成，用于处理错误。在下面这个例子中，如果 URL中
动态参数 id 对应的用户不存在,就返回状态码404：

```python
    from flask import abort
    
    @app.route('/user/<id>')
    def get_user(id):
        user = load_user(id)
        if not user:
        abort(404)
        return '<h1>Hello, %s</h1>' % user.name
```
注意， abort 不会把控制权交还给调用它的函数，而是抛出异常把控制权交给 Web 服
务器。例子：
    
```python
    from flask import Flask
    from flask import redirect
    from flask import abort
    
    @app.route('/user/<id>')
    def get_user(id):
        user = load_user(id)
        if not user:
            abort(404)
        return '<h1>Hello, %s</h1>' % user.name

    def load_user(id):
        return None

    if __name__ == '__main__':
        app.run(debug=True)
```
## 2.6Flask扩展
Flask-Script是一个Flask的扩展，为Flask程序添加一个命令行解析器。Flask-Script使用pip安装:
    
    $pip install flask-script
为Flask开发的扩展都在flask.ext命名空间下。Flask-Script输出了一个名为Manager的类。这个扩展可以把程序实例作为参数传给狗在其，初始化主类的实例。创建的对象可以在各个扩展中使用。

```python
    #! usr/bin/env python3
    # -*- coding:utf-8 -*-
    from flask import Flask
    from flask.ext.script import Manager

    app = Flask(__name__)#type:Flask
    manager = Manager(app)

    if __name__=='__main__':
        manager.run()
```
    
运行时可指定命令，例子：

    >python hello.py --help

