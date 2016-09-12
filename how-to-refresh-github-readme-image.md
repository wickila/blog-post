---
title: github如何刷新README文件中的图片缓存
date: 2016-09-12
category : Coding Tips
published : True
tags : [github, http]
---

github为了保证每次访问README的时候图片都是正常可显示的，会把README文件中的外部图片缓存起来，这样不管外部图片链接是否还有效，README文件中的图片都是有效可以显示的。

但是这种缓存有时候也会带来困扰。当我们希望更新README文件中的图片时会发现，不管我们如何修改我们外部链接的图片，github的README文件中的图片始终不会刷新，这就是这种缓存带来的困扰。

解决办法是，告诉github把这份缓存删除掉。命令如下：

```bash
curl -X PURGE {url}
```

比如：

```shell
wickideMacBook-Pro:~ wicki$ curl -X PURGE https://camo.githubusercontent.com/5ac8595b2389fa12eb6ee47763a5f7382d5dd977/687474703a2f2f6f6462796a337332722e626b742e636c6f7564646e2e636f6d2f6175746f7472616465322e706e67
{"status": "ok", "id": "81-1469760280-9199470"}

```

curl是linux的一个利用URL规则上传下载的工具。

-X的意思是指定用什么方法请求URL。比如常见http方法：GET, POST之类的。

PURGE就是清除的意思。所以上面这句命令的意思是用PURGE方法去访问github的缓存服务器，缓存服务器得到请求后，就清除指定的缓存内容。其中PURGE方法是github的缓存服务器自定义的HTTP方法。

当明白原理以后，就算你不是在Linux环境，没有curl工具。也可以用Postman来完成这个工作。

> 关于自定义HTTP方法，webpy中可以很好地测试。webpy在得到一个请求以后，就寻找URL映射，再执行映射结果中对应的方法。比如用TEST方法访问http://loacalhost:8080/，而'/'映射的处理类是Index，webpy返回的是Index.TEST()的内容。如果Index中不存在TEST方法，就会返回“405 Method Not Allowed”。

以下是测试过程：

```python
class Index:
    def GET(self):
        """index page"""
        return render('index.html', {})

    def TRACE(self):
        return "trace info"

    def TEST(self):
        return "test info"
```

```shell
wickideMacBook-Pro:~ wicki$ curl -X TEST http://localhost:8080
test info 
wickideMacBook-Pro:~ wicki$ curl -X TRACE http://localhost:8080
trace info 
wickideMacBook-Pro:~ wicki$ curl -X GOOD http://localhost:8080
None
wickideMacBook-Pro:~ wicki$ 
```

```
127.0.0.1:53099 - - [12/Sep/2016 11:22:40] "HTTP/1.1 TEST /" - 200 OK
127.0.0.1:53100 - - [12/Sep/2016 11:22:45] "HTTP/1.1 TRACE /" - 200 OK
127.0.0.1:53101 - - [12/Sep/2016 11:22:48] "HTTP/1.1 GOOD /" - 405 Method Not Allowed
```



更多信息请参考：

- [curl命令](https://zh.wikipedia.org/wiki/CURL)
- [HTTP协议的请求方法](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE#.E8.AF.B7.E6.B1.82.E6.96.B9.E6.B3.95)

