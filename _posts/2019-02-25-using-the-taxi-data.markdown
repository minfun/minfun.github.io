---
layout: post
title:  "Using the Taxi Data Streams"
date:   2019-02-25 11:37:13 +0800
categories: apache flink
---

## Using the Taxi Data Streams

纽约市得出租车和豪华轿车委员会提供了一份关于2009年至2015年纽约市出租车的公共数据集。我们使用编辑好的子集来生成有关出租车的事件流。

## 1. 下载出租车数据文件

通过以下命令来下载出租车数据

{% highlight shell %}
wget http://training.ververica.com/trainingData/nycTaxiRides.gz

wget http://training.ververica.com/trainingData/nycTaxiFares.gz
{% endhighlight %}

您可以使用wget或者其它来得到这些文件，不要解压缩或重命名.gz文件。

## 2. 出租车乘车事件的模式

我们的出租车数据集包含有关纽约市各个出租车的信息。每次乘车由两个事件代表：旅行开始和旅行结束事件。每个事件由11个字段组成。

{% highlight shell %}
rideId         : Long      // a unique id for each ride
taxiId         : Long      // a unique id for each taxi
driverId       : Long      // a unique id for each driver
isStart        : Boolean   // TRUE for ride start events, FALSE for ride end events
startTime      : DateTime  // the start time of a ride
endTime        : DateTime  // the end time of a ride,
                           //   "1970-01-01 00:00:00" for start events
startLon       : Float     // the longitude of the ride start location
startLat       : Float     // the latitude of the ride start location
endLon         : Float     // the longitude of the ride end location
endLat         : Float     // the latitude of the ride end location
passengerCnt   : Short     // number of passengers on the ride

{% endhighlight %}

注意：数据集包含坐标信息无效或缺失的记录（经度和纬度为0.0）

还有一个包含出租车费用数据的相关数据集，包含以下字段：

{% highlight shell %}
rideId         : Long      // a unique id for each ride
taxiId         : Long      // a unique id for each taxi
driverId       : Long      // a unique id for each driver
startTime      : DateTime  // the start time of a ride
paymentType    : String    // CSH or CRD
tip            : Float     // tip for this ride
tolls          : Float     // tolls for this ride
totalFare      : Float     // total fare collected
{% endhighlight %}

## 3. 在Flink程序中生成一个出租车数据流

注意：许多练习已经提供了使用这些出租车数据流的代码

我们提供了一个Flink源函数（TaxiRideSource），它可以读取带有出租车记录的.gz文件并发出一系列TaxiRide事件。源在事件事件运行。TaxiFare事件有类似的源功能（TaxiFareSource）。

为了真实的生成流，事件的发布与其时间戳成比例。实际上相隔十分钟后发生的两个事件也相继十分钟后提供服务。可以指定加速因子来快进流。如果加速因子为60，则在一分钟内发生的事件在一秒钟内完成。此外，可以指定最大服务延迟，这导致每个事件在指定范围内随机延迟。这产生了无序流，这在许多实际应用中很常见。

对于这些练习，加速因子为600或更多（每秒处理10分钟的事件事件）和最大延迟60（秒）都将很好地工作。

所有练习都应使用事件时间特征来实现。事件时间将程序语义与服务速度分离，即使在历史数据或无序传送的数据的情况下也能保证一致的结果。

## Checkpointing

一些练习要求使用CheckpointedTaxiRideSource和/或CheckpointedTaxiFareSource。与TaxiRideSource和TaxiFareSource不同，这些变种能够检查它们的状态。

## Table Sources

TaxiRideTableSource和TaxiFareTableSource表源可用于Table和SQL API。

## 如何使用这些源

Java

{% highlight java %}
// get an ExecutionEnvironment
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// configure event-time processing
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

// get the taxi ride data stream
DataStream<TaxiRide> rides = env.addSource(new TaxiRideSource("/path/to/nycTaxiRides.gz", maxDelay, servingSpeed));
{% endhighlight %}

Scala

{% highlight scala %}
// get an ExecutionEnvironment
val env = StreamExecutionEnvironment.getExecutionEnvironment

// configure event-time processing
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

// get the taxi ride data stream
val rides = env.addSource(new TaxiRideSource("/path/to/nycTaxiRides.gz", maxDelay, servingSpeed))
{% endhighlight %}

还有一个TaxiFareSource，它使用nycTaxiFares.gz文件以类似的方式工作。用这个源来创建TaxiFare事件流。

Java

{% highlight java %}
// get the taxi fare data stream
DataStream<TaxiFare> fares = env.addSource(new TaxiFareSource("/path/to/nycTaxiFares.gz", maxDelay, servingSpeed));
{% endhighlight %}


Scala

{% highlight scala %}
// get the taxi fare data stream
val fares = env.addSource(new TaxiFareSource("/path/to/nycTaxiFares.gz", maxDelay, servingSpeed))
{% endhighlight %}
