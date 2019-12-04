# Flink开发批处理应用程序 #
	- 需求：词频统计(word count)
	    - 一个文件，统计文件中每个单词出现的次数
	    - 分隔符是\t
	    - 统计结果我们直接打印在控制台（生产上肯定是Sink到目的地）
	- 实现：   
		Flink + Java
			前置条件： Maven 3.0.4 (or higher) and Java 8.x
			第一种创建项目的方式：
				mvn archetype:generate                               \
				-DarchetypeGroupId=org.apache.flink              \
				-DarchetypeArtifactId=flink-quickstart-java      \
				-DarchetypeVersion=1.7.0 \
				-DarchetypeCatalog=local
	
				out of the box：OOTB 开箱即用
	
			开发流程/开发八股文编程
				1）set up the batch execution environment
				2）read
				3）transform operations  开发的核心所在：开发业务逻辑
				4）execute program
	
			功能拆解
				1）读取数据  
					hello	welcome
				2）每一行的数据按照指定的分隔符拆分
					hello
					welcome
				3）为每一个单词赋上次数为1
					(hello,1)
					(welcome,1)	
				4) 合并操作  groupBy	


		Flink + Scala
			前置条件：Maven 3.0.4 (or higher) and Java 8.x 
	
			mvn archetype:generate                               \
			-DarchetypeGroupId=org.apache.flink              \
			-DarchetypeArtifactId=flink-quickstart-scala     \
			-DarchetypeVersion=1.7.0 \
			-DarchetypeCatalog=local
	
		Flink Java vs Scala
			1) 算子  map  filter  
			2）简洁性
- 大数据处理的流程：  
	- MapReduce：input -> map(reduce) -> output
	- Storm:  input -> Spout/Bolt -> output
	- Spark: input -> transformation/action --> output
	- Flink:  input ->  transformation/sink --> output

- DataSet and DataStream
	- immutable
	- 批处理：DataSet
	- 流处理：DataStream

- Flink编程模型  
	- 1）获取执行环境  
	- 2）获取数据  
	- 3）transformation  
	- 4）sink  
	- 5）触发执行  

- Flink中自定义数据源

	- implementing the `SourceFunction` for non-parallel sources, 
- or by implementing the `ParallelSourceFunction` interface 
	- or extending the `RichParallelSourceFunction` for parallel sources. 

## Flink的Time和Window



 ![img](https://ci.apache.org/projects/flink/flink-docs-release-1.9/fig/times_clocks.svg) 

对于Flink里面的三种时间
	事件时间  10:30
	摄取时间  11:00 
	处理时间  11:30

思考：
对于流处理来说，你们觉得应该是以哪个时间作为基准时间来进行业务逻辑的处理呢？

幂等性

env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
思考：默认的TimeCharacteristic是什么？


窗口分配器：定义如何将数据分配给窗口

A WindowAssigner is responsible for assigning each incoming element to one or more windows
每个传入的数据分配给一个或者多个窗口

Flink对于Window来说有两大类

1）基于时间

- tumbling windows 滚动窗口
  	- have a fixed size and do not overlap  不会重叠

 ![img](https://ci.apache.org/projects/flink/flink-docs-release-1.9/fig/tumbling-windows.svg) 

- sliding windows  滑动窗口
  	overlapping   窗口会重叠

  窗口大小，窗口滑动大小![img](https://ci.apache.org/projects/flink/flink-docs-release-1.9/fig/sliding-windows.svg) 

  

- session windows  会话窗口

- global windows   全局窗口

[start timestamp , end timestamp)

https://blog.csdn.net/lmalds/article/details/52704170



2）基于数量

## Window Functions

`ReduceFunction`, 

`AggregateFunction`,

`FoldFunction`，

`ProcessWindowFunction` 



## Flink  Connectors 与外部系统交互（kafka，Hadoop）



### Bundled Connectors

- [Apache Kafka](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/kafka.html) (source/sink) *
- [Apache Cassandra](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/cassandra.html) (sink)
- [Amazon Kinesis Streams](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/kinesis.html) (source/sink)
- [Elasticsearch](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/elasticsearch.html) (sink) *
- [Hadoop FileSystem](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/filesystem_sink.html) (sink) *
- [RabbitMQ](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/rabbitmq.html) (source/sink)
- [Apache NiFi](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/nifi.html) (source/sink)
- [Twitter Streaming API](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/twitter.html) (source)
- [Google PubSub](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/pubsub.html) (source/sink)

### Connectors in Apache Bahir

Additional streaming connectors for Flink are being released through [Apache Bahir](https://bahir.apache.org/), including:

- [Apache ActiveMQ](https://bahir.apache.org/docs/flink/current/flink-streaming-activemq/) (source/sink)
- [Apache Flume](https://bahir.apache.org/docs/flink/current/flink-streaming-flume/) (sink)
- [Redis](https://bahir.apache.org/docs/flink/current/flink-streaming-redis/) (sink)
- [Akka](https://bahir.apache.org/docs/flink/current/flink-streaming-akka/) (sink)
- [Netty](https://bahir.apache.org/docs/flink/current/flink-streaming-netty/) (source)

## Flink部署及作业提交

- Standalone 单机模式
- Standalone 集群模式
- Yarn模式
- 。。。



