---
layout: post
title:  "A Complete Example"
date:   2019-02-23 11:37:13 +0800
categories: apache flink
---

## A Complete Example

这个例子将关于人员的记录流作为输入，并将其过滤为只包含成人。

{% highlight java %}
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.api.common.functions.FilterFunction;

public class Example {
    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<Person> flintstones = env.fromElements(
            new Person("Fred", 35),
            new Person("Wilma", 35),
            new Person("Pebbles", 2)
        );

        DataStream<Person> adults = flintstones.filter(new FilterFunction<Person>() {
            @Override
            public boolean filter(Person person) throws Exception {
                return person.age >= 18;
            }
        });

        adults.print();

        env.execute();
    }

    public static class Person {
        public String name;
        public Integer age;
        public Person() {};

        public Person(String name, Integer age) {
            this.name = name;
            this.age = age;
        };

        public String toString() {
            return this.name.toString() + ": age" + this.age.toSrting();
        };
    }
}
{% endhighlight %}

## 流执行环境

每个Flink应用程序都需要一个执行环境，在这个例子中是 `env`。流式应用需要使用 `StreamExecutionEnvironment`。

在你的应用程序中DataStream API的调用会建立一个关联到`StreamExecutionEnvironment`的作业图。当`env.execute()`被调用这个作业图就会被打包并发送给 Job Manager（作业管理器），作业管理器将作业并行化并将其片段分发给Task Manager（任务管理器）用于执行。每个作业的并行切片将会在task slot（任务槽）中执行。

需要注意的是，如果你不调用 execute()你的应用就不会跑。

![processes](/assets/image/apache/flink/training/processes.svg)

此分布式运行时取决于您的应用程序是否可序列化。它还要求集群中的每个节点都可以使用所有依赖项。

## 基本流源

在上面的例子中我们通过`env.fromElements(...)`构建了一个`DataStream<Person>`。这是将简单流集合在一起以便在原型或测试中使用的便捷方式。在`StreamExecutionEnvironment`上同样有一个方法`fromCollection(Collection)`。我们可以这样实现：

{% highlight java %}
List<Person> people = new ArrayList<Person>();

people.add(new Person("Fred", 35));
people.add(new Person("Wilma", 35));
people.add(new Person("Pebbles", 2));

DataStream<Person> flintstones = env.fromCollection(people);
{% endhighlight %}

在原型设计时将一些数据放入流中的另一种简单方式是使用套接字

{% highlight java %}
DataStream<String> lines = env.socketTextStream("localhost", 9999)
{% endhighlight%}

或文件

{% highlight java %}
DataStream<String> lines = env.readTextFile("file:///path")
{% endhighlight %}

在实际应用中，最常用的数据源是那些支持低延迟，高吞吐量并行度去以及倒带和重放的数据源 - 高性能和容错的先决条件 - 例如Apache Kafka, Kinesis 以及各种文件系统。REST APIs和数据库也经常用于丰富流。

## 基本流下沉

上例使用`adults.print()`来显示结果到任务管理器的日志中（如果运行在IDE上则会出现在IDE的控制台中）。这个方法会为流中的每个元素调用`toString()`。

输出看上去是这样的：

{% highlight java %}
1> Fred: age 35
2> Wilma: age 35
{% endhighlight %}

1> 和 2> 指出了产生输出的子任务

你也可以写到文本文件

{% highlight java %}
stream.writeAsText("/path/to/file")
{% endhighlight %}

或者CSV文件

{% highlight java %}
stream.writeAsCsv("/path/to/file")
{% endhighlight %}

或者套接字

{% highlight java %}
stream.writeToSocket(host, port, SerializationSchema)
{% endhighlight %}

在生产中，常用的接收器包括Kafka以及各种数据库和文件系统。

## 调试

在生产中，你将向应用程序运行的远程集群提交应用程序JAR文件。如果失败，远程也会失败。作业管理器和任务管理器日志在调试此类故障时非常游泳，但在IDE内部进行本地调试要容易的多，这是Flink支持的。你也可以设置断点，检查局部变量，并逐步执行代码。你也可以进入Flink代码，如果你想了解Flink的工作原理，这可能是了解更多内部信息的好方法。