---
layout: post
title:  "How to do the Labs"
date:   2019-02-26 11:37:13 +0800
categories: apache flink
---

## How to do the Labs

在实践课程中，您将使用各种Flink API实现Flink程序。您将学习如何使用Apache Maven打包Flink程序并在正在运行的Flink实例上执行打包程序。

以下步骤将指导您完成使用提供的数据流，实现第一个Flink流程序，以及在正在运行的Flink实例上打包和执行程序的过程。

我们假设您已根据我们的设置指南设置了您的开发环境，并从github获得了flink-training-exercise repo的本地克隆。

## 1. 了解数据

最初的一系列练习都是基于有关出租车和出租车费用的事件数据流。这些流由源函数生成，源函数从输入文件中读取数据。请阅读这些说明以了解如何使用它们。

## 2. 编辑`ExerciseBase`

下载数据集后在IDE中打开`com.dataartisans.flinktraining.exercises.datastream_java.utils.ExerciseBase`类，并编辑这两行以指向您下载的两个出租车数据文件：

{% highlight java %}
public final static String pathToRideData = "/Users/david/stuff/flink-training/trainingData/nycTaxiRides.gz";
public final statis String pathToFareData = "/Users/david/stuff/flink-training/trainingData/nycTaxiFares.gz";
{% endhighlight %}

## 3. 在IDE中运行和调试Flink程序

可以在IDE中执行和调试Flink程序。这显着简化了开发过程，并提供了类似于处理任何其他Java（或Scala）应用程序的体验。

在IDE中启动Flink程序就像运行main（）方法一样简单。在引擎盖下，执行环境将在同一进程中启动本地Flink实例。因此，也可以在代码中放置断点并对其进行调试。

假设您有一个导入了flink-training-exercises项目的IDE，您可以运行（或调试）一个简单的流式传输作业，如下所示：

- 在IDE中打开`com.dataartisans.flinktraining.examples.datastream_java.basics.RideCount`类 
- 使用IDE运行（或调试）`RideCountExample`类的`main()`方法。

## 4. 练习，测试和解决方案

其中许多练习包括一个练习课，其中包含大部分入门所需的样板代码，以及一个JUnit Test类，其测试将在您实施正确的解决方案之前失败，而Solution类则包含完整的解决方案。

现在您已准备好开始第一次练习。
