<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**
- [Python语言特性](#python语言特性)
      - [1 生成器和迭代器](#1-生成器和迭代器)
      - [2 自己实现一个xrange](#2-自己生成一个xrange)
      - [3 LEGB 变量搜索](#3-legb)
      - [4 python unicode&str](#4-python-unicodestr)

- [http](#http)
      - [1 GET和POST的区别](#1-get和post的区别)
      - [2 HttpOnly是什么](#2-httponly是什么)
      - [3 Connection: Keep-Alive](#3-Connection: Keep-Alive)
      - [4 http是如何判断一条请求已经结束](#4-http是如何判断一条请求已经结束)
      - [5 301和302区别](#5-301和302区别)
- [nginx](#nginx)
      - [1 root和alias区别](#1-root和alias区别)
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
## root和alias区别
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
