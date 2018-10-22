---
layout: post
title:  "Terms And Types"
date:   2018-06-11 19:23:27 +0800
categories: scala tutorial
---

## SCALA教程

接下来的部分是学习Scala的快速教程

内容基于MOOCS Scala函数式编程原则 和 Scala函数式程序设计

目标人群是已经有编程经验和熟悉JVM的人

## 编程要素

程序语言为程序员提供了表达计算的方式

每种不平凡的编程语言都提供:

- 表达最简元素的原始表达式
- 结合表达式的方法
- 抽象表达式的方法，它引入了一个表达式的名称，然后通过该表达式引用它

## 原始表达式

原始表达式的样例：

- 数字 `1`:

{% highlight scala %}
1
{% endhighlight %}

- 布尔值`true`:

{% highlight scala %}
true
{% endhighlight %}

- 文本 `Hello, Scala!`:

{% highlight scala %}
"Hello, Scala!"
{% endhighlight %}
(注意双引号的用法，`"`)

- 复合表达式

复杂的表达式可以通过运算符结合简单表达式来表达。因此它们可以表达更复杂的计算：

- 1 加 2 等于几？

{% highlight scala %}
1 + 2
{% endhighlight %}

-文本`Hello,`和`Scala!`相连的结果是？

{% highlight scala %}
"Hello, " ++ "Scala!"
{% endhighlight %}

## 评估
非原始表达式是按下列方式评估的:

1. 以最左边的运算符为准
2. 评估操作数（从左到右）
3. 将运算符应用于操作数上

评估过程一旦产生值就会停止

**例**

这是对算术表达式的评估：

{% highlight scala %}
(1 + 2) * 3
3 * 3
9
{% endhighlight %}

{% highlight scala %}
1 + 2 shouldBe 3
"Hello, " ++ "Scala!" shouldBe "Hello, Scala!"
{% endhighlight %}


## 方法调用

用简单表达式创建复杂表达式的另一种方法是在表达式上调用方法

- 文本`Hello, Scala!`的大小是多少？

{% highlight scala %}
"Hello, Scala!".size
{% endhighlight %}

使用 `.` 将表达方法应用于表达式

应用该方法的对象被命名为目标对象

-1到10之间的数字范围是多少？

{% highlight scala %}
1.to(10)
{% endhighlight %}

方法可以通过括号来传递参数

在下面的例子里，`abs` 方法返回数字的绝对值， `toUpperCase` 方法返回目标`字符串`的大写形式

{% highlight scala %}
"Hello, Scala!".toUpperCase shouldBe "HELLO, SCALA!"
-42.abs shouldBe 42
{% endhighlight %}

## 运算符就是方法
实际上，运算符只是具有符号名称的方法：

{% highlight scala %}
3 + 2 == 3。+(2)
{% endhighlight %}

`中缀` 语法允许忽略 `点` 和 `括号`

中缀语法也可以与常规方法一起使用：

{% highlight scala %}
1.to(10) == 1 to 10
{% endhighlight %}

任何带参数的方法都可以像中缀运算符一样使用

## 值和类型
表达式有一个值和一个类型，评估模型定义了如何从表达式中获取值，类型对值进行分类

`0` 和 `1` 都是数字，它们的类型都是 `Int`
`"foo"` 和 `"bar"` 都是文本，它们的类型都是 `String`

## 静态类
Scala 编译器静态检查未组合不兼容的表达式

使用与`Int`类型不同的值填写以下空白：

{% highlight scala %}
1 to 10
{% endhighlight %}

## 常见类型

- `Int`: 32位整型(如：`1` `23` `456`)
- `Double`: 64位浮点数字(如：`1.0`, `2.3`,`4.56`
- `Boolean`:布尔值(`true` 和 `false`)
- `String`:文本(如:`"foo"`,`"bar"`)

注意:类型名称始终以大写字母开头

## 练习

以下是一些标准方法，猜一猜它们做了什么？

{% highlight scala %}
16.toHexString shouldB2 "10"
(0 to 10).contains(10) shouldBe true
(0 until 10).contains(10) should Be false
"foo".drop(1) shouldBe "oo"
"bar".take(2) shouldBe "ba"
{% endhighlight %}
