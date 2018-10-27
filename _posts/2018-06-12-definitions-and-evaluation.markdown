---
layout: post
title:  "Definitions And Evaluation"
date:   2018-06-12 19:35:08 +0800
categories: scala tutorial
---

## 定义和求值

## 

思考下列程序来计算半径为10的圆的区域

{% highlight scala %}
3.14159 * 10 * 10
{% endhighlight %}

为了使复杂的表达式更具可读性，我们可以为中间表达式赋予更有意义的名字

{% highlight scala %}
val radius = 10
val pi = 3.14159

pi * radius * radius
{% endhighlight %}

除了使最后一个表达式更具可读性之外，它还允许我们不重复半径的实际值

## 求值

通过其定义从右边开始替换用于对名称求值

*示例*

下列表达式的求值步骤：

{% highlight scala %}
pi * radius * radius
3.14159 * radius * radius
3.14159 * 10 * radius
31.4159 * radius
31.4159 * 10
314.159
{% endhighlight %}

## 方法

定义可以包含参数，例如：

{% highlight scala %}
def square(x: Double) = x * x

square(3.0) shouldBe 9.0
{% endhighlight %}

定义一个计算圆的面积的方法，给定半径：

{% highlight scala %}
def square(x: Double) = x * x

def area(radius: Double): Double = 3.14159 * square(radius)

area(10) shouldBe 314.159
{% endhighlight %}

## 多个参数

多个参数以 `,` 分隔：

{% highlight scala %}
def sumOfSquares(x: Double, y: Double) = square(x) + square(y)
{% endhighlight %}

## 参数和返回类型

函数参数伴随着它的类型，以 `:` 分隔

{% highlight scala %}
def power(x: Double, y: Int): Double = ...
{% endhighlight %}

如果给出了返回类型，则它遵循参数列表

## 变量和定义

每次使用都会对右边的`def`定义求值

`val` 右边 在定义本身的位置进行评估。之后，名称引用的是值。


