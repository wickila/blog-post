---
title: web.py中使用PATCH方法
date: 2016-10-27
category : Solve Problems
published : True
tags : [web.py, patch, python]
---

一直以来都是用web.py，因为web.py简洁，轻巧。没想到今天却掉进一个坑，爬了半天才爬出来。在此记录一下，提醒后面的朋友，作为前车之鉴。

#### 问题

事情是这样的。因为要与app通信，所以准备构建一个RESTful的API服务。各种接口都实现以后，用postman测试一切正常。但是在Android APP中用Async-Http采用PATCH方法提交数据的时候，在web.py中的web.input()方法中，却始终接收不到客户端提交的参数数据。

#### 解决过程

本来以为是Async-Http的问题，可是查了一个多小时，都没有看出问题。只好先定位问题所在了，看看到底是客户端没有发送参数数据，还是服务端没有处理参数数据。最好的办法就是用抓包工具看看数据包。

数据包抓取的结果是客户端有发送参数数据，参数数据放在HTTP请求中的消息体中被发送到服务器。但是服务器却没有把数据读取出来。

于是仔细查看了web.py的源代码。web.py中读取HTTP请求的代码都在wsgisever模块的\__init__文件中，关键代码如下：

```python
def parse_request(self):
        """Parse the next HTTP request start-line and message-headers."""
        self.rfile = SizeCheckWrapper(self.conn.rfile,
                                      self.server.max_request_header_size)
        try:
            self.read_request_line()
        except MaxSizeExceeded:
            self.simple_response("414 Request-URI Too Long",
                "The Request-URI sent with the request exceeds the maximum "
                "allowed bytes.")
            return
        
        try:
            success = self.read_request_headers()
        except MaxSizeExceeded:
            self.simple_response("413 Request Entity Too Large",
                "The headers sent with the request exceed the maximum "
                "allowed bytes.")
            return
        else:
            if not success:
                return
        
        self.ready = True
```

从代码中可以看出来，web.py接收到http请求以后，先把请求行读取出来，然后把请求头读取出来。（HTTP协议分为四个部分：__请求行、请求头、空行、消息体__。[参考HTTP协议](https://zh.wikipedia.org/zh-cn/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)）。我们常用的GET方法就是在URL后面带参数，所以在请求行里面会被提取出来。但是消息体里面的数据呢？怎么就丢弃不要了？

但是再仔细想想，POST方法中的参数数据也是放在消息体中的，POST方法却可以从web.input方法中顺利取到参数数据。所以我们要仔细看看web.input到底是怎么取参数的。关键代码在webapi模块中：

```python
def rawinput(method=None):
    """Returns storage object with GET or POST arguments.
    """
    method = method or "both"
    from cStringIO import StringIO

    def dictify(fs): 
        # hack to make web.input work with enctype='text/plain.
        if fs.list is None:
            fs.list = [] 

        return dict([(k, fs[k]) for k in fs.keys()])
    
    e = ctx.env.copy()
    a = b = {}
    
    if method.lower() in ['both', 'post', 'put']:
        if e['REQUEST_METHOD'] in ['POST', 'PUT']:
            if e.get('CONTENT_TYPE', '').lower().startswith('multipart/'):
                # since wsgi.input is directly passed to cgi.FieldStorage, 
                # it can not be called multiple times. Saving the FieldStorage
                # object in ctx to allow calling web.input multiple times.
                a = ctx.get('_fieldstorage')
                if not a:
                    fp = e['wsgi.input']
                    a = cgi.FieldStorage(fp=fp, environ=e, keep_blank_values=1)
                    ctx._fieldstorage = a
            else:
                fp = StringIO(data())
                a = cgi.FieldStorage(fp=fp, environ=e, keep_blank_values=1)
            a = dictify(a)

    if method.lower() in ['both', 'get']:
        e['REQUEST_METHOD'] = 'GET'
        b = dictify(cgi.FieldStorage(environ=e, keep_blank_values=1))

    def process_fieldstorage(fs):
        if isinstance(fs, list):
            return [process_fieldstorage(x) for x in fs]
        elif fs.filename is None:
            return fs.value
        else:
            return fs

    return storage([(k, process_fieldstorage(v)) for k, v in dictadd(b, a).items()])
```

从代码中可以看出来。如果是POST或者是PUT方法，就会去读取HTTP的消息体，把数据从消息体里面取出来（关键代码:`fp = StringIO(data())`，data()方法的作用就是取出消息体中的数据。官方是这样描述data方法的:Sometimes, the client sends a lot of data by the POST method. In webpy, you can handle it like this.

```python
class RequestHandler():
    def POST():
        data = web.data() # you can get data use this method
```

从上面的分析过程来看，web.py在HTTP请求的预处理中，只取出来请求行与请求头中的数据。当需要用的时候，才去读取消息体中的数据，这样做的好处是：消息体中的数据可能很大，不用的时候就不要去处理它，可以提高程序效率。

再回到我们的问题来，原来当参数放在消息体中的时候，只有POST与PUT方法能取出数据来，其它方法是取不到消息体中的数据的。这就是为什么PATCH方法取不到参数数据。

可能因为PATCH方法还比较年轻，而web.py的作者又早早地离开了我们，所以才会留下这个问题。感叹一下天才的离开对这个世界来说是多么大的遗憾。

其实这个问题，有人在[这里](https://github.com/webpy/webpy/pull/259)提交了支持PATCH方法的代码，但是不知道为什么，最后代码中还是没有支持到PATCH方法。只能自己手动支持一下PATCH方法了。github玩得溜的朋友，也可以再去提交一次。造福一下大家。