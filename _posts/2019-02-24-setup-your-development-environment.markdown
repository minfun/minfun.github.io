---
layout: post
title:  "Setup your Development Environment"
date:   2019-02-24 11:37:13 +0800
categories: apache flink
---

以下说明将指导您完成设置开发环境的过程，以便开发，调试和执行培训练习和示例的解决方案。

## 1. 软件需求

Flink支持Linux，OS X和Windows作为Flink程序和本地执行的开发环境。下列软件需要被安装在你的系统中并用于Flink开发的建立：

- Java JDK 8（JRE是不够的）
- Apache Maven 3.x
- Git
- IDE（推荐IntelliJ）

需要注意的是不支持较旧版本和较新版本的Java。只有Java 8。

## 2.克隆并建立flink-training-exercises项目

`flink-training-exercises`项目包含编程练习，测试和参考解决方案以及大量示例。从Github克隆`flink-training-exercises`项目并建立：

{% highlight shell %}
git clone https://github.com/dataArtisans/flink-training-exercises.git
cd flink-training-exercises
mvn clean package
{% endhighlight %}

如果你之前没有这样做过，那么你最终将下载Flink培训练习项目的所有依赖项。通常需要花费几分钟，取决于网络速度。

如果所有测试都通过并且构建成功，那么您将有一个良好的开端。

## 3. 将flink-training-exercises项目导入到IDE中

这个项目需要被导入到IDE中

- IntelliJ：
 - 1. 因为这个项目混合了Java和Scala代码，所以需要安装Scala插件
  - 打开IntelliJ plugins settings (IntelliJ IDEA -> Preferences -> Plugins) 点击 “Install Jetbrains plugin…”.
  - 选择并安装 Scala插件
  - 重启IntelliJ
 - 2. 选择`pom.xml`来导入本项目
 - 3. 每一步都选择默认值
 - 4. 确保SDK对话框具有JDK的有效路径并将所有其他选项保留为其默认值时即完成项目导入
 - 5. 打开项目结构对话框，并在“全局库”部分中添加Scala 2.11 SDK（即使你不打算使用Scala，你也需要这个）

## 4.下载数据集

通过运行以下命令来下载此培训中使用的出租车数据文件。

{% highlight shell %}
wget http://training.ververica.com/trainingData/nycTaxiRides.gz

wget http://training.ververica.com/trainingData/nycTaxiFares.gz
{% endhighlight %}

您可以使用wget或者其它来得到这些文件，不要解压缩或重命名.gz文件。

要了解有关此数据的更多信息，请参阅使用Taxi数据流。

注意：练习中有这些数据文件的硬连线路径。在尝试执行它们之前，请阅读如何执行实验。

如果还要设置本地群集以在IDE外部执行Flink作业，请参阅设置本地Flink群集。

如果要使用SQL客户端，请参阅设置SQL客户端。