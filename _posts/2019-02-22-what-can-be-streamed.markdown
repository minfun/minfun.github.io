---
layout: post
title:  "Whay can be Streamed"
date:   2019-02-22 11:20:13 +0800
categories: apache flink
---

## What can be Streamed

Flink用于Java和Scala的DataStream APIs将允许传输他们可以序列化的任何内容。

Flink的序列化器用于：

- 简单类型：String，Long，integer，Boolean，Array
- 复合类型：Tuples，POJOs，Scala case classes

而Flink对于其他类型则回归于Kryo。

## Java

## Tuples
对于Java而言，Flink定义了Tuple1到Tuple25类型。

{% highlight java %}
Tuple2<String, Integer> person = new Tuple2<>("Fred", 35);

// zero based index!
String name = person.f0;
Integer age = person.f1
{% endhighlight %}

## POJOs

一个POJOs（普通的旧Java对象）是任何Java类：

- 有一个空的默认构造函数
- 所有域都是以下之一：
  - public
  - 有一个默认的getter和setter

例如：

{% highlight java %}
public class Person {
    public String name;
    public Integer age;
    public Person() {};
    public Person(String name, Integer age) {
        ...
    };
}

Person person = new Person("Fred Flintstone", 35);
{% endhighlight %}

## Scala tuples和 case classes

这些工作正如您所期望的那样。