---
layout: post
title:  "Python performance compare"
date:   2019-03-08 11:37:13 +0800
categories: python performance
---

## 来自生产实践中的一次性能提升

## 问题描述：在生产环境中，某个任务的生成过程特别漫长，并导致生产上数据的延迟，因此需要调研并定位问题。

需要从一下几个方面开始调研：

1. 任务状态 - 查看任务是否有异常
2. 消息队列（SQS）- 查看任务是否堆积
3. 任务触发器（Launcher）- 查看任务是否触发
4. 工作进程（Celery Worker）- 查看工作进程是否正常运行
5. 源数据 - 查看依赖数据是否完成
6. 入库任务状态 - 查看入库阶段任务是否阻塞或者是否有连接异常

随着经验的积累，我先从任务状态着手调研，任务表是记录各种任务信息的，其中有些字段记录了主机信息（worker machine）和进程信息（process id），根据这两部分内容可以判断基本问题。

登上生产机器查看进程状态，进程状态为R（Running），这个状态不是期望的，同时查看进程消耗的内存等资源，发现消耗特别大，而且整台机器的资源消耗都很高。

似乎找到了问题所在，机器资源不足，那么升级机器是否是最佳的解决方案呢？

## 分两部分入手：
1. 当前机器上部署了很多其它任务，依赖比较大，大量的celery worker在运行（单个celery worker需要200M内存），是否可以优化celery worker并减小内存使用？
2. 当前任务运行时间过长，单个进程内存消耗巨大，是否可以优化这个业务？

对于第一点简单的解决方案就是减少依赖，降低celery worker运行时加载的代码量，这一部分成功释放了一部分内存，本文的重点是对于第二点的优化。

可以想想这几个问题：
这个业务单个进程消耗900M内存，为什么这个进程消耗的内存这么大？
运行时间漫长无比，作为一个触发器任务运行时间将近一个小时，为什么这么慢？
那么，有方法降低和减小内存的使用吗？

首先是增加日志，日志方便获取更多的信息，然后根据这些信息定位问题。在逐行日志的帮助下，分析出程序运行特别慢的地方。

{% highlight python %}
logger.info('Generate data')
id_key = 1000000000
ids = []
for i in range(1000):
    empty_list = []
    for j in range(1000):
        empty_list.append(id_key)
        id_key += 1
    ids.append(a)
logger.info('Load data')
unique_ids = set()
for id_list in ids:
    unique_ids = unique_ids | set(id_list)
logger.info('End Load')
{% endhighlight %}

在 `Load data` 和 `End Load` 之间耗费了很多的时间，看上去这部分代码似乎没什么问题，可是为什么会这么慢呢？

现在来计算一下这段代码的时间复杂度：
来自官网的set操作时间复杂度表
![python-set-time-complexity](/assets/image/python/performance/python-set-time-complexity.jpg)
优化前的时间复杂度：
(0+k)+(k+k)+...+(nk)=k(1+n)/2*n ~ O(kn*n)

问题显而易见，那么就可以开始优化了，优化之后的代码看起来非常简单：

{% highlight python %}
logger.info('Generate data')
id_key = 1000000000
ids = []
for i in range(1000):
    empty_list = []
    for j in range(1000):
        empty_list.append(id_key)
        id_key += 1
    ids.append(a)
logger.info('Load data')
unique_ids = list()
for id_list in ids:
    unique_ids.extend(id_list)
unique_ids = set(unique_ids)
logger.info('End Load')
{% endhighlight %}

![python-list-time-complexity](/assets/image/python/performance/python-list-time-complexity.jpg)
优化后的时间复杂度：
k+k+...+k=nk ~ O(kn)

O(n)的时间复杂度对比O(n*n)的时间复杂度，时间上的损耗减小了很多，足以达到期望的水平。

不妨统计一下真实的时间开销：

{% highlight python %}
def generate_list(lens):
    id_key = 1000000000
    b = []
    for i in range(lens):
        a = []
        for j in range(1000):
            a.append(id_key)
            id_key += 1
        b.append(a)
    return b

import time
t_begin = time.time()
l = generate_list(1000)
c = []
for x in l:
    c.extend(x)
d = set(c)
t_end = time.time()
print 'time cost:'
print t_end - t_begin
{% endhighlight %}

当lens分别为1000，2000，3000，4000，5000的时候，
优化后的方式带来的时间开销小于1秒：
![python-performance-refactor-new-way](/assets/image/python/performance/python-performance-refactor-new-way.jpg)

优化前的方式带来的时间开销远大于优化前：
![python-performance-refactor-old-way](/assets/image/python/performance/python-performance-refactor-old-way.jpg)

