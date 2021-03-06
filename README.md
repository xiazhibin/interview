<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**
- [Python语言特性](#python语言特性)
  - [1 生成器和迭代器](#1-生成器和迭代器)
  - [2 自己实现一个xrange](#2-自己生成一个xrange)
  - [3 LEGB 变量搜索](#3-legb)
  - [4 python unicode&str](#4-python-unicodestr)
  - [5 python 如何寻找属性](#5-python-attribute)
  - [6 cache property](#6-cache-property)
  - [7 buid list of list](#7-build-list-of-list)
  - [8 init set](#8-init-set)
  - [9 简单说说类的创建过程](#9-简单说说类的创建过程)
  - [10 简单说说这个段代码(文件写入)](#10-简单说说这个段代码)
  - [11 简单说说这段代码（decorator）](#11-简单说说这段代码)
  - [12 写出这个代码输出(copy array)](#12-写出这个代码输出)
  - [13 AttributeDict](#13-attributedict)
  

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
  - [3 flask sessionmiddleware实现](#3-flask-sessionmiddleware实现)
  - [4 django 一次处理请求过程](#4-django-一次处理请求过程)
      
- [网络](#网络)
  - [1 水平触发和边缘触发](#1-水平触发和边缘触发)
  - [2 简单解释select和epoll](#2-简单解释select和epoll)

- [redis](#redis)
  - [1 限制ip访问次数](#1-限制ip访问次数)
  - [2 redis集群方式](#2-redis集群方式)
  
- [linux](#linux)
  - [1 fork机制](#1-fork机制)
  
- [算法](#算法)
  - [1 删除列表中特定元素](#1-remove-list)
  - [2 快排](#2-快排)
  - [3 全排列](#3-全排列)
  - [4 排列](#4-排列)
  - [5 反转链表](#5-反转链表)
  - [6 堆排序](#6-堆排序)
  - [7 反转二叉树](#7-反转二叉树)
  

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
还可以使用步长的方式
```python
def my_range(start, stop, step):
    total_step = int(stop - start) / step
    i = 0
    while total_step >= i:
        yield start + step * i
        i += 1
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
from dis import dis
会分析反汇编

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

## 7 build-list-of-list
 - `[['_']*3]*3` the same as 
 
 ``` 
 row = ['_']*3 
 board = [] 
 for _ in range(3):
    board.append(row)
 ```
 
 - `[['_']*3 for _ in range(3)]` the same as  
 ``` 
 board = [] 
 for _ in range(3):
    board.append(['_']*3)
 ```

## 8 init-set

说说set([1])和{1}都是初始化一个set，哪个比较好，为什么？

{1} ``` LOAD_CONST BUILD_SET RETURN_VALUE```

set([1]) ```LOAD_NAME LOAD_CONST BUILD_LIST CALL_FUNCTION RETURN_VALUE ```


## 9 简单说说类的创建过程
``` 
class A():
    pass
a = A()
```
对象的创建只是调用可调用对象的一种特例 因为A是type对象的一个实例, 所以调用了type的__call__方法，调用object的__new__方法，返回一个A的实例
继续调用object的__init__方法

## 10 简单说说这个段代码
``` python
with open('a.txt', 'wb+') as f:
    f.write('nihao')
```
open方法有个buffering参数， 默认为-1，使用系统的buffersize，充满了就自动flush到文件

write方法,Write a string to the file. There is no return value. Due to buffering, the string may not actually show up in the file until the flush() or close() method is called.

[what exactly the python's file.flush() is doing?](https://stackoverflow.com/questions/7127075/what-exactly-the-pythons-file-flush-is-doing)

## 11 简单说说这段代码
``` python
# test.py
def A(func):
    return func

@A
def B():
    pass

if __name__ == '__main__':
    B()
    
python test.py
```

 - `__name__` is a built-in variable which evaluate to the name of the current module。However, if a module is being run directly then `__name__` instead is set to the string "__main__"
 - `@`这个只是语法糖，上述等价于 B = A(B) A已经被执行一次了

## 12 写出这个代码输出
``` python
l1 = [3, [66, 55, 44], (7, 8, 9)]
l2 = list(l1)
print(l1 == l2)
print(l1 is l2)
l1.append(100)
l1[1].remove(55)
print('l1:', l1)
print('l2:', l2)
l2[1] += [33, 22]
l2[2] += (10, 11)
print('l1:', l1)
print('l2:', l2)
```
True

False

l1: [3, [66, 44], (7, 8, 9), 100]

l2: [3, [66, 44], (7, 8, 9)]

l1: [3, [66, 44, 33, 22], (7, 8, 9), 100]

l2: [3, [66, 44, 33, 22], (7, 8, 9, 10, 11)]

这个是浅复制。l1,l2不是同一个list对象，所以他们自身的增改是不会影响对方的。但是，里面的元素就不一定了。`l1[1].remove(55)`因为l2,l1的第二个元素是list，他们的对象是一样的,mutable的+=,remove这样的操作是在自身身上进行，并不会返回新的对象。而第三个元素是一个`tuple` 是一个immutable，对自身的操作会返回一个新的对象,跟string一个道理。

浅复制，mutable和immutable修改自身元素特点

## 13 AttributeDict
```python
class AttributeDict(dict):
    __getattr__ = dict.__getitem__
    __setattr__ = dict.__setitem__


class AttributeDict2(dict):
    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(key)

    def __setattr__(self, key, value):
        self[key] = value


class AttributeDict3(dict):
    def __init__(self, **kwargs):
        super(AttributeDict3, self).__init__(**kwargs)
        self.__dict__ = self

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

## 3 flask sessionmiddleware实现
继承SessionInterface， 实现`open_session,save_session`方法

## 4 django 一次处理请求过程
WSGIHandler
  - BaseServer->serve_forever()->select.select->self.RequestHandlerClass(request, client_address, self)
  - 如果没有middleware为空，load middleware（每个middleware可能会有，`process_request, process_view,process_template_response等`）
  - 初始化WSGIRequest
  - urlresolvers
  - for 一次所有response = _request_middleware(request)
  - 看url match不
  - for 一次所有的response ＝ _view_middleware(request)
  - view_func(request) 视图处理函数
  - for一次response ＝ _template_response_middleware(request)
  - urlresolvers.set_urlconf(None)
  - for一次response ＝ _response_middleware(request)
  - return response
  
# 网络
## 1 水平触发和边缘触发
- 水平触发 只要满足条件就会不断触发该事件
- 边缘触发 只有在条件发现变化的时候，才会触发事件

## 2 简单解释select和epoll
- select
通过系统调用来监视着一个由多个文件描述符（file descriptor）组成的数组，当select()返回后，数组中就绪的文件描述符会被内核修改标记位(其实就是一个整数)，使得进程可以获得这些文件描述符从而进行后续的读写操作。select饰通过遍历来监视整个数组的，而且每次遍历都是线性的
- 缺点
 - 每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多的时候会很大
 - 单个进程能够监视的fd数量存在最大限制，在linux上默认为1024（可以通过修改宏定义或者重新编译内核的方式提升这个限制）
 - 并且由于select的fd是放在数组中，并且每次都要线性遍历整个数组，当fd很多的时候，开销也很大
- [实例代码](https://gist.github.com/xiazhibin/a1b8fcde3d84c610277914cf6849a09c)

- epoll
 - epoll的解决方法是每次注册新的事件到epoll中，会把所有的fd拷贝进内核，而不是在等待的时候重复拷贝，保证了每个fd在整个过程中只会拷贝1次。
 - epoll没有这个限制，它所支持的fd上限是最大可以打开文件的数目，具体数目可以cat /proc/sys/fs/file-max查看，一般来说这个数目和系统内存关系比较大。
 - epoll的解决方法不像select和poll每次对所有fd进行遍历轮询所有fd集合，而是在注册新的事件时，为每个fd指定一个回调函数，当设备就绪的时候，调用这个回调函数，这个回调函数就会把就绪的fd加入一个就绪表中。（所以epoll实际只需要遍历就绪表）。
- [实例代码](https://gist.github.com/xiazhibin/e564c6eab9f727bb5b95e71f8a66fa9a)

# redis
## 1 限制ip访问次数
- [tokenbucket](https://github.com/xiazhibin/blog/blob/master/%E9%99%90%E6%B5%81.md)

## 2 redis集群方式
 - 客户端分片 根据自己的规则对多台redis实例进行访问。可运维和可开发性差
 - 代理分片 通过代理，客户端对redis服务器进行访问
 - redis cluster 将所有Key映射到16384个Slot中，集群中每个Redis实例负责一部分
 - Twemproxy 分布式中间件
 - codis

# 算法
## 1 remove list
题目：在一个数组里面移除指定的value,并返回新的数组的长度,例如[1,23,4,None,None,3,45]删掉None ，返回5
- 方法一：
```python
def remove_element(array):
    for i in xrange(len(array) - 1, 0, -1):
        if array[i] is None:
            del array[i]
    return len(array)
```
- 方法二：
```python
def remove_element(array):
    j = 0
    for i in range(len(array)):
        if array[i] is not None: #符合条件的元素就往上移动
            array[j] = array[i]
            j += 1
    del array[j:]
    return len(array)
```
- 方法三:
```python
def filter_in_place(array, filter_):
    j = 0
    for i in range(0, len(array)):
        if filter_(array[i]):
            array[j] = array[i]
            j += 1
    del array[j:]
    return len(array)
```

## 2 快排
```python
def quick_sort(arr):
    if arr == []: return []
    return quick_sort([i for i in arr[1:] if i >= arr[0]]) + [arr[0]] + quick_sort([i for i in arr[1:] if i < arr[0]])

```
## 3 全排列
 - [全排列算法思路解析](http://blog.csdn.net/summerxiachen/article/details/60579623)
 - [full_permutation.py](https://gist.github.com/xiazhibin/fe925e8bba637960df2ada5ac1f6c038)

## 4 排列
 - 取一个出来，剩下的n-1里面取m-1个，或者剩下的n-1取m个。
 - [combie.py](https://gist.github.com/xiazhibin/5650acd8d00ba355df00e072362c3235)

## 5 反转链表
```go
func reverseLN(head *LinkNode) *LinkNode{
	temp := head
	var cur *LinkNode
	head = nil
	for ; temp != nil; {
		cur = temp.next
		temp.next = head
		head = temp
		temp = cur
	}
	return head
}
```

## 6 堆排序
这个是满的二叉树所以有以下性质
- 父节点i的左子节点在位置(2i+1)
- 父节点i的右子节点在位置(2i+2)
- 子节点i的父节点在位置floor((i-1)/2)

步骤：
- 创建最大值堆
- 堆排序（移除位在第一个数据的根节点，放到末尾，并做最大堆调整的递归运算）

```go
func HeapSort(arr []int) {
	l := len(arr)
	m := l / 2
	for i := m; i > -1; i-- {
		siftDown(arr, i, l-1)
	}

	for i := l - 1; i > 0; i-- {
		arr[i], arr[0] = arr[0], arr[i]
		siftDown(arr, 0, i-1)
	}
}

func siftDown(arr []int, start, end int) {
	left := 2*start + 1
	if left > end {
		return
	}
	whichOne := left
	right := 2*start + 2
	if right <= end && arr[left] < arr[right] {
		whichOne = right
	}
	if arr[start] > arr[whichOne]{
		return
	}
	arr[whichOne], arr[start] = arr[start], arr[whichOne]
	siftDown(arr, whichOne, end)
}
```

## 7 反转二叉树
思路：将node的左右节点反转即可（因为对于一个节点来说，反转了自己，自己的孙子也跟反转的了）
```go
func invert_tree2(node *tree) {
	if node == nil {
		return
	}
	if (node.left == nil) && (node.right == nil) {
		return
	}
        node.left, node.right = node.right, node.left
	invert_tree2(node.left)
	invert_tree2(node.right)
}
```

```go
func invert_tree(root *tree) {
	if root == nil {
		return
	}
	stack := NewStack()
	stack.Push(root)
	for stack.len > 0 {
		node, ok := stack.Pop()
		if ok != nil {
			return
		}
		node.left, node.right = node.right, node.left
		if node.left != nil {
			stack.Push(node.left)
		}
		if node.right != nil {
			stack.Push(node.right)
		}
	}
}
```

# linux
## 1 fork机制
```python
import os
pid1 = os.fork()
pid2 = os.fork()
print pid1, pid2
```
如果一个输出是(1001,1002) 其他输出是什么(0,1003),(1001,0),(0,0)

```python
fork()&&fork()||fork()
```
1.一共产生几个进程  2.返回值为1的概率为多少？(5,3/5)

[fork面试题](https://blog.csdn.net/chdhust/article/details/10579001)
