---
layout: post
title:  "Python 'is None' with '== None'"
date:   2019-03-20 11:37:13 +0800
categories: python performance
---

## python比较 is None和 == None

PEP8说：比较实例是否为None时应使用`is`或者`is not`，不要使用`==`运算符

为了弄清楚这些内容，现在创建一些类似于0的实例或者特殊的类，来看看哪些实例的`is None`和`== None`为`True`

{% highlight python %}
class Zero():
    def __nonzero__(self):
        return False


class Len0():
    def __len__(self):
        return 0


class Equal():
    def __eq__(self, other):
        return True


stuff = [None, False, 0, 0L, 0.0, 0j, (), [], {}, set(), '', float('NaN'), float('inf'), Zero(), Len0(), Equal()]

for x in stuff:
    if x is None:
        print("%r is None" % x)
    if x == None:
        print("%r == None" % x)
{% endhighlight %}

输出结果为：
{% highlight python %}
None is None
None == None
<__main__.Equal instance at 0x105262830> == None
{% endhighlight %}

那么为什么`Equal == None` 而不是 `Equal is None`，这是因为`==`是比较两个对象的内容是否相等，即两个对象的值是否相等，而不区分两个对象在内存中的引用地址是否一样。在`==`判别式中调用的是`Equal`类中的`__eq__`方法，使得判别式返回`True`。而在做`is`判断的时候，相当于判断`id(Equal()) == id(None)`

分析一下这些例子：

{% highlight python %}
>>> a = 'hello'
>>> b = 'hello'
>>> print(id(a))
4498943360
>>> print(id(b))
4498943360
>>> print(a is b)
True
>>> print(a == b)
True
>>> a = 'hello world'
>>> b = 'hello world'
>>> print(id(a))
4498943264
>>> print(id(b))
4498943552
>>> print(a is b)
False
>>> print(a == b)
True
>>> a = [1, 2, 3]
>>> b = [1, 2, 3]
>>> print(id(a))
4498953568
>>> print(id(b))
4498832992
>>> print(a is b)
False
>>> print(a == b)
True
>>> a = [1, 2, 3]
>>> b = a
>>> print(id(a))
4498953424
>>> print(id(b))
4498953424
>>> print(a is b)
True
>>> print(a == b)
True
{% endhighlight %}

如果`id(a)`和`id(b)`的值一致则返回`True`，如果`a`的值和`b`的值一致则`a == b`。
那么为什么`a`和`b`都是`hello`的时候，`a is b`返回`True`，而`a`和`b`都是`hello world`的时候，`a is b`返回`False`呢？
这是因为python字符串有个驻留机制，对于比较小的字符串，为了提高系统性能python会保留其值的副本，在创建新的字符串的时候直接指向该副本，所以`hello`在内存中只有一个副本，而`hello world`是长字符串，在内存中没有副本，所以python分别为a和b创建了对象。

在进行比较的时候，`is`比`==`更快，运行如下示例：

{% highlight python %}
import timeit
print timeit.timeit("1 is None", number=10000000)
print timeit.timeit("1 == None", number=10000000)
0.247735023499
0.437688112259
{% endhighlight %}

这是为什么呢？因为`is`进行的是快速的`id`比较，而`==`需要同时在`dict`中查找`1`和`None`两个对象然后进行比较。
