---
title: webpy中巧用装饰器简化session验证代码
date: 2016-08-08
category : python
published : True
tags : [python, webpy, 装饰器]
---

在web应用中，常常需要记录用户的登录状态。大家常用的做法都是采用session来记录会话状态。在webpy中可以很方便地使用session，大概的原理是这样的：客户端第一次访问服务器的时候，webpy会自动生成一个session，并且将一个session-id返回给客户端。以后每次客户端访问服务器的时候，都会拿着这个session-id给服务器看，证明自己曾经访问过服务器，以维持客户端与服务器之间的会话状态。
具体教程参考[http://webpy.org/cookbook/sessions.zh-cn](http://webpy.org/cookbook/sessions.zh-cn)。

在需要验证用户登录的页面里，我们经常需要先判断用户的登录状态，判断成功后，再去做对应的逻辑，如下：
```python
    def GET(self):
        user = session.user
        if user!=None:
            dosomething()
        else:
            web.seeother('/signin')
```

当页面比较少的时候，这样做并没什么问题。但是当页面多的时候，这种验证用户的代码会大量重复。这时python的装饰器就可以派上用场了。装饰器的解释可以参考这里[http://coolshell.cn/articles/11265.html](http://coolshell.cn/articles/11265.html)，这位老师讲解得非常透彻。在本文中，我们仅讨论装饰器在session验证中的妙用。

首先，我们新写一个装饰器：
```python
	#用于session验证
	def need_check_session(function):
	    def wrap_function(*args):
	        session = web.config._session
	        if session.user:
	            return function(*args,user=session.user)
	        else:
	            return u"you're not login.plase login first."
	    return wrap_function
```

在上面的装饰器函数need_check_session中，我们首先验证用户的登录状态，如果用户已经登录，就执行对应的处理函数。否则直接要求用户登录。session验证的装饰器写好后，我们再来改造一下刚才写的GET函数,在函数的前面一行加上@need_check_session：
```python
	@need_check_session
	def GET(self,user):
	    dosomething(user)
```
是不是瞬间变得非常简洁？！给处理函数GET加上need_check_session这个装饰器以后，每次调用GET前，都会首先调用装饰器函数检查用户的登录状态，而不必在GET函数里面检查用户是否登录了。同理如果有处理函数必须是在用户登录状态下才能执行（比如修改用户资料等），就只用加上这个装饰器就完事大吉啦。

下面提供本示例的完整代码：

[SessionWithDecorator.zip](/files/SessionWithDecorator.zip)

[SessionWithoutDecorator.zip](/files/SessionWithoutDecorator.zip)

Enjoy with python's decorator :)