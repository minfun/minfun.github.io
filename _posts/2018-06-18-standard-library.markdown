---
layout: post
title:  "Standard Library"
date:   2018-06-18 19:13:21 +0800
categories: scala tutorial
---

## 标准库

## 列表

在函数式编程里，列表是基本的数据结构

一个以 `x1, ..., xn` 为元素的列表通常写成：`List(x1, ..., xn)` ：

{% highlight scala %}
val fruit = List("apples", "oranges", "pears")
val nums = List(1, 2, 3, 4)
val diag3 = List(List(1, 0, 0), List(0, 1, 0), List(0, 0, 1))
val empty = List()
{% endhighlight %}

- 列表是不可变的 -- 元素不能发生变化
- 列表是递归的
- 列表是同类的 -- 列表中的元素类型必须相同

元素类型为 `T` 的列表写作：`List[T]`:

{% highlight scala %}
val fruit: List[String] = List("apples", "oranges", "peers")
val nums: List[Int] = List(1, 2, 3, 4)
val diag3: List[List[Int]] = List(List(1, 0, 0), List(0, 1, 0), List(0, 0, 1))
val empty: List[Nothing] = List()
{% endhighlight %}

**构造列表**

列表都是这样构造的：

- 空的列表 `Nil`
- 构造操作符 `::`, `x :: xs` 第一个元素为`x`， 接下来的元素是`xs`

例如：

{% highlight scala %}
val fruit = "apples" :: ("oranges" :: ("pears" :: Nil))
val nums = 1 :: (2 :: (3 :: (4 :: Nil)))
val empty = Nil
{% endhighlight %}

右结合

规则：操作符 `:` 向右关联。
`A :: B :: C`被释作 `A :: (B :: C)`

可以在定义中省略括号：

{% highlight scala %}
val nums = 1 :: 2 :: 3 :: 4 :: Nil
{% endhighlight %}

操作符 `:` 可以视作右边的方法调用

因此上边的表达式等同于：

{% highlight scala %}
val nums = Nil.::(4).::(3).::(2).::(1)
{% endhighlight %}

**操作列表**

列表可以通过模式匹配来解耦：

- `Nil`: 常量`Nil`
- `p :: ps`: 列表的一个模式 `head` 匹配 `p`, `tail` 匹配 `ps`

{% highlight scala %}
nums match {
  // Lists of `Int` that starts with `1` and then `2`
  case 1 :: 2 :: xs => ...
  // Lists of length 1
  case x :: Nil => ...
  // Same as `x :: Nil`
  case List(x) => ...
  // The empth list, same as `Nil`
  case Lisgt() =>
  // A list that contains as only element another list that starts with `2`
  case List(2 :: xs) => ...``
}
{% endhighlight %}

**练习：列表排序**

升序排列数字组成的列表：

- 排序列表 `List(7, 3, 9, 2)`，先对尾部列表 `List(3, 9 ,2)` 进行排序得到 `List(2, 3, 9)`
- 然后将头部 `7` 插入到对应的位置，得到结果 `List(2, 3, 7, 9)`

*插入排序*

{% highlight scala %}

{% endhighlight %}



