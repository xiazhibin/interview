<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**
- [Python语言特性](#python语言特性)
      - [1 生成器和迭代器](#1-生成器和迭代器)
      - [2 自己实现一个xrange](#2-自己生成一个xrange)
      - [3 LEGB 变量搜索](#3-legb)
      - [4 python unicode&str](#4-python-unicodestr)
      - [5 python 如何寻找属性](#5-python-attribute)
      - [6 cache property](#6-cache-property)

- [http](#http)
      - [1 GET和POST的区别](#1-get和post的区别)
      - [2 HttpOnly是什么](#2-httponly是什么)
      - [3 Connection: Keep-Alive](#3-connection-keep-alive)
      - [4 http是如何判断一条请求已经结束](#4-http是如何判断一条请求已经结束)
      - [5 301和302区别](#5-301和302区别)
      
- [nginx](#nginx)
      - [1 root和alias区别](#1-root和alias区别)
  
- [flask](#flask)
      - [1 flask一次请求过程](#1-flask一次请求过程)
      - [2 flask request,g实现原理](#2-flask-requestg实现原理)
      
- [网络](#网络)
      - [1 select epoll](#1-简单解释select和epoll)

      
- [算法](#算法)
- [综合](#综合)
      - [1 短网址](#1-短网址)
<!-- markdown-toc end -->









# Python语言特性
## 1 生成器和迭代器
迭代器 iterator - 有__iter__方法就是iterable，有__next__()/next()/getitem() 就是iteror

生成器 generator - 带有yield的函数就是了

所以generator是iterator，因为yield本来就有next方法

详细可以看这个问题 http://taizilongxu.gitbooks.io/stackoverflow-about-python/content/1/README.html

## 2 自己生成一个xrange
用yield
```python
def my_xrange(start, stop=None, step=None):
    now = 0
    if stop:
        now = start
    else:
        stop = start
    step = step if step else 1

    op = ge if step > 0 else le
    while True:
        if op(now, stop):
            raise StopIteration
        yield now
        now += step
```


## 3 LEGB
```python
count = 10
def outer():
    count = 100
    print(count)
outer()

count = 10
def outer():
    print(count) #这个报错 UnboundLocalError: local variable 'count' referenced before assignment
    count = 100
    print(count)
```
## 4 python unicode&str

python的str就是字节数组，unicode才是我们认为的字符。str.decode('utf-8')变成字符，unicode.encode('gbk')就变成一个字节数组就是str。所以如果字符就应该使用unicode，如果读入一个文本文件，应该指定解码格式，不然读进来的是str，而不是unicode。所以我们操作的str之前要先decode一下

## 5 python attribute 

针对新式类 python如何访问一个类实例的属性。例如 a.attr 

首先调用`__getattribute__`方法，如果raise`AttributeError`，还会继续调用`__getattr__`

if: 在A类或其基类的__dict__出现，且为data descriptor, 调用其`__get__`方法

elif: 出现A类或其基类的__dict__中，且为普通属性，返回`obj.__dict__['attr']`

else: 出现A类或其基类的__dict__中，且为no-data descriptor,调用其`__get__`方法

end

[官方文档](https://docs.python.org/3/howto/descriptor.html)

[参考博文1](http://www.cnblogs.com/xybaby/p/6270551.html)

[参考博文2](http://www.betterprogramming.com/object-attribute-lookup-in-python.html)

## 6 cache property
```
class MyNone(object):
    def __str__(self):
        return 'no value'

    def __repr__(self):
        return 'none'


_my_none = MyNone()


class cache_property(property):
    def __init__(self, func):
        self.__name__ = func.__name__
        self.func = func

    def __get__(self, instance, type2):
        if instance is None:
            return self
        value = instance.__dict__.get(self.__name__, _my_none)
        if value is _my_none:
            value = self.func(instance)
            instance.__dict__[self.__name__] = value
        return value

    def __set__(self, instance, value):
        instance.__dict__[self.__name__] = value
        
```

# HTTP
## 1 Get和Post的区别
根据HTTP文档来说，GET就是用来取数据的，幂等的，POST是用来修改数据的。

## 2 HttpOnly是什么
HttpOnly是cookie里面的一个属性，它是用来防止XSS（Cross Site Scripting)跨站脚本攻击。加上这个属性，js就不能访问这个cookie里面的内容

## 3 Connection: Keep-Alive
复用同一条TCP链接，减少三次握手的开销

## 4 http是如何判断一条请求已经结束
如果没有keepalive，服务器主动断开链接。有的话，可以通过Content-Length判断，如果没有Content-Length，那么就会有Transfer-Encoding: chunk 
而且每个TCP/HTTP连接上同时只能有一个请求 / 响应，这意味着完成响应之前，这个连接不能用于其他请求

## 5 301和302区别
301永久重定向 302临时重定向

# nginx
## 1 root和alias区别
```python
location /img/ {
    alias /var/www/image/;
}
#若按照上述配置的话，则访问/img/目录里面的文件时，ningx会自动去/var/www/image/目录找文件

location /img/ {
    root /var/www/image;
}
#若按照这种配置的话，则访问/img/目录下的文件时，nginx会去/var/www/image/img/目录下找文件。
```
alias是一个目录别名的定义
root则是最上层目录的定义

# flask
## 1 flask一次请求过程
[v0.1](http://blog.csdn.net/bin381/article/details/57086114)

## 2 flask request,g实现原理

关键字

`LocalProxy`,`_lookup_req_object`, `top = _request_ctx_stack.top`

`__storage__为内部保存的自己，键就是thread.get_ident，也就是根据线程的标示符返回对应的值`

`ctx.push操作将ctx push到_request_ctx_stack，所以当我们调用request.method时将调用_lookup_req_object。top此时就是ctx上下文对象，而getattr(top, "request")将返回ctx的request，而这个request就是在ctx的__init__中根据环境变量创建的。
每一次调用视图函数操作之前，flask会把创建好的ctx放在线程Local中，当使用时根据线程id就可以拿到了。在wsgi_app的finally中会调用ctx.auto_pop(error),会根据情况判断是否清除放_request_ctx_stack中的ctx。`

[参考博客](http://blog.csdn.net/yueguanghaidao/article/details/39533841)


# 网络
## 1 简单解释select和epollselect
- select是通过系统调用来监视着一个由多个文件描述符（file descriptor）组成的数组，当select()返回后，数组中就绪的文件描述符会被内核修改标记位(其实就是一个整数)，使得进程可以获得这些文件描述符从而进行后续的读写操作。select饰通过遍历来监视整个数组的，而且每次遍历都是线性的
