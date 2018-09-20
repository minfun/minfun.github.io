---
layout: post
title:  "Higher order functions"
date:   2018-06-17 19:56:18 +0800
categories: scala tutorial
---

## 高阶函数

函数式语言将函数视为第一梯队的值。

这就意味着函数如其他值可以像参数一样传递或者当作结果返回。

因此提供了一种更灵活的方式去编写程序。

把函数当作参数进行传递或者当作结果进行返回的都被称作高阶函数。

## 动机

思考以下程序：

将 a 到 b 之间的数值累加：

{% highlight scala %}
def sumInts(a: Int, b: Int: Int =
  if (a > b) 0 else a + sumInts(a + 1, b)
{% endhighlight %}

将 a 到 b 之间的数进行立方累加：

{% highlight scala %}
def cube(x: Int): Int = x * x * x

def sumCubes(a: Int, b: Int): Int =
  if (a > b) 0 else cube(a) + sumCubes(a + 1, b)
{% endhighlight %}

将 a 到 b 之间的数进行阶乘累加：

{% highlight scala %}
def factorial(a: Int): Int =
  if (a == 1) 1 else a * factorial(a - 1)

def sumFacotrials(a: Int, b: Int): Int =
  if (a > b) 0 else factorial(a) + sumFactorials(a + 1, b)
{% endhighlight %}

这些累加方法都很相似，如何从中得到通用的模式？

## 高阶函数求和

定义一个函数：
{% highlight scala %}
def sum(f: Int => Int, a: Int, b: Int): Int =
  if (a > b) 0 else f(a) + sum(f, a + 1, b)
{% endhighlight %}

然后可以这样写：

{% highlight scala %}
def id(x: Int): Int = x
def sumInts(a: Int, b: Int) = sum(id, a, b)
def sumCubes(a: Int, b: Int) = sum(cube, a, b)
def sumFactorials(a: Int, b: Int) = sum(factorial, a, b)
{% endhighlight %}

## 函数类型
类型`A => B`是指一个函数需要输入类型`A`的参数并且返回类型`B`

所以， `Int => Int`是一类将整型映射为整型的函数

## 匿名函数

将函数当作参数从而创建小的函数

如果用`def`定义或者命名这些函数会显得很冗长

因此，我们不需要用`val`定义字符串

如：
{% highlight scala %}
val str = "abc"; println(str)
{% endhighlight %}

直接这样写：
{% highlight scala %}
println('abc')
{% endhighlight %}

因为字符串是常量，类似的我们也可以用函数常量，这样我们就可以在写函数的时候不用给它命名

**匿名函数语法**

示例：值的立方
{% highlight scala %}
(x: Int) => x * x * x
{% endhighlight %}

`(x: Int)`是这个函数的参数， `x * x * x`是函数本身

如果编译器可以从上下文推断出类型，那么参数的类型可以被忽略

**语法糖**

匿名函数 `(x1: T1, ..., xn: Tn) => e` 用 `def` 可以表示成：
{% highlight scala %}
{ def f(x1: T1, ..., xn: Tn) = e; f}
{% endhighlight %}
`f` 是随意的名字，不会在程序中使用，因此可以说匿名函数是语法糖

**总结**

通过匿名函数，我们可以更简单的实现累加方法：

{% highlight scala %}
def sumInts(a: Int, b: Int) = sum(x => x, a, b)
def sumCubes(a: Int, b: Int) = sum(x => x * x * x, a, b)
{% endhighlight %}

## 练习
`sum` 函数用的是线性递归， 下面是尾递归版本：
{% highlight scala %}
def sum(f: Int => Int, a: Int, b: Int): Int = {
  def loop(x: Int, acc: Int): Int = {
    if (x > b) acc
    else loop(x + 1, acc + f(x))
  }
  loop(a, 0)
}
sum(x => x, 1, 10) 值为 55
{% endhighlight %}

## 编者注：

这里的尾递归调用可以写成这样一种模式：

{% highlight scala %}
def sumTailPattern(f: (Int, Int) => Int, x: Int, y: Int, v: Int): Int =
  if (x > y) v else sumTailPattern(f, x + 1, y, f(x, 1) + v)

def sumTailPatternInt(x: Int, y: Int, v: Int): Int = sumTailPattern(int, x, y, v)
println(sumTailPatternInt(1, 3, 0))
// 6
println(sumTailPatternInt(3, 5, 0))
// 12
{% endhighlight %}

如果有兴趣可以实现阶乘累加的递归实现：

{% highlight scala %}
@annotation.tailrec
def tailFactorial(x: Int, v: Int): Int =
  if (x == 1) v else tailFactorial(x - 1, x * v)

def sumTailPatternFactorial(x: Int, y: Int, v: Int): Int = sumTailPattern(tailFactorial, x, y, v)
println(sumTailPatternFactorial(1, 3, 0))
// 9
{% endhighlight %}
