# Flume使用总结([Flume 1.9用户手册中文版 — 可能是目前翻译最完整的版本了 (liyifeng.org)](https://flume.liyifeng.org/#))

## 一、Source

### 1、Avro Source

```shell
1.1、Avro Source监听Avro端口，接收从外部Avro客户端发送来的数据流。如果与上一层Agent的 Avro Sink 配合使用就组成了一个分层的拓扑结构。 必需的参数已用粗体标明。
eg:
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = avro
a1.sources.r1.channels = c1
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 4141
```

![avrosource](C:\Users\liwei\Pictures\Screenshots\avrosource.png)

### 2、Thrift Source

```
监听Thrift 端口，从外部的Thrift客户端接收数据流。如果从上一层的Flume Agent的 Thrift Sink 串联后就创建了一个多层级的Flume架构（同 Avro Source 一样，只不过是协议不同而已）。Thrift Source可以通过配置让它以安全模式（kerberos authentication）运行，具体的配置看下表。 必需的参数已用粗体标明。
eg:
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = thrift
a1.sources.r1.channels = c1
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 4141
```

![thriftsource](C:\Users\liwei\Pictures\Screenshots\thriftsource.png)

### 3、Exec Source

```
这个source在启动时运行给定的Unix命令，并期望该进程在标准输出上连续生成数据（stderr 信息会被丢弃，除非属性 logStdErr 设置为 true ）。 如果进程因任何原因退出， 则source也会退出并且不会继续生成数据。 综上来看cat [named pipe]或tail -F [file]这两个命令符合要求可以产生所需的结果，而date这种命令可能不会，因为前两个命令（tail 和 cat）能产生持续的数据流，而后者（date这种命令）只会产生单个Event并退出。
eg:
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /var/log/secure
a1.sources.r1.channels = c1
```

![execsource](C:\Users\liwei\Pictures\Screenshots\execsource.png)

### 4、Spooling Directory Source

```shell
这个source允许你把要收集的文件放在磁盘上的某个指定目录。它会监视这个目录中产生的新文件，并在新文件出现时从新文件中解析数据出来。新文件被完全读入channel后默认会重命名该文件名或者删除该文件（通过设置deletePolicy参数）。spooling directory source是可靠的，及时flume重新启动或者被kill，也不会丢失数据。但是作为这种可靠性的代价，指定目录中被收集的文件必须是不可变的、唯一命名的。
以下情况会抛出异常：
	1、如果文件写入完成后又被再次写入新内容，flume将向其日志文件（flume自己logs目录下的日志文件）打印错误并停止处理
	2、如果重新使用以前的文件名，flume将向其日志文件打印错误并停止处理
eg:
a1.channels = ch-1
a1.sources = src-1

a1.sources.src-1.type = spooldir
a1.sources.src-1.channels = ch-1
a1.sources.src-1.spoolDir = /var/log/apache/flumeSpool
a1.sources.src-1.fileHeader = true
```

![spoolingdirectorysource](C:\Users\liwei\Pictures\Screenshots\spoolingdirectorysource.png)

### 4、Taildir Source

```
taildir source会监控指定的一些文件，并在检测到新的一行数据产生的时候几乎实时地读取他们，如果新的一行数据还没有写完，taildir source会等到这行写完之后再读取。
	taildir source是可靠的，它会定期地一JSON格式在一个专门用于定位的文件上记录每个文件最后读取位置，如果flume挂掉，他可以从文件标记的位置重新开始读取。
	文件按照修改时间顺序来读取，修改时间最早的文件将会被最先读取（先进先出）。taildir source不重命名、删除或修改它监控的文件，不支持读取二进制文件，只能逐行读取文本文件。
eg:
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = TAILDIR
a1.sources.r1.channels = c1
a1.sources.r1.positionFile = /var/log/flume/taildir_position.json
a1.sources.r1.filegroups = f1 f2
a1.sources.r1.filegroups.f1 = /var/log/test1/example.log
a1.sources.r1.headers.f1.headerKey1 = value1
a1.sources.r1.filegroups.f2 = /var/log/test2/.*log.*
a1.sources.r1.headers.f2.headerKey1 = value2
a1.sources.r1.headers.f2.headerKey2 = value2-2
a1.sources.r1.fileHeader = true
a1.sources.ri.maxBatchCount = 1000
```

![taildirsource](C:\Users\liwei\Pictures\Screenshots\taildirsource.png)

### 5、Kafka Source

```
Kafka Source就是一个kafka消费者，他从kafka的topic中读取数据，如果运行了多个kafka source，则可以把他们配置到同一个消费者组中，以便每个source都读取一组唯一的topic分区
eg:
tier1.sources.source1.type = org.apache.flume.source.kafka.KafkaSource
tier1.sources.source1.channels = channel1
tier1.sources.source1.batchSize = 5000
tier1.sources.source1.batchDurationMillis = 2000
tier1.sources.source1.kafka.bootstrap.servers = localhost:9092
tier1.sources.source1.kafka.topics = test1, test2
tier1.sources.source1.kafka.consumer.group.id = custom.g.id
```

![kafkasource](C:\Users\liwei\Pictures\Screenshots\kafkasource.png)

### 6、NetCat Tcp Source

```
这个source与nc -k -l [host] [port]相似，监听一个指定的端口，把从这个端口收到转换为event，它能识别的是带换行符的文本数据，解析成功的数据会发生到channel中
eg:
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = netcat
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 6666
a1.sources.r1.channels = c1
```

![netcat](C:\Users\liwei\Pictures\Screenshots\netcat.png)

## 二、Channel

### 1、Memory Channel

```
这个channel是把Event队列存储到内存上，队列的最大数量就是capacity的设定值。他非常适合对吞吐量有较高要求的场景，但是也有代价，当发生故障的时候会丢失当时内存中的所有Event
eg:
a1.channels = c1
a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 10000
a1.channels.c1.byteCapacityBufferPercentage = 20
a1.channels.c1.byteCapacity = 800000
```

![memorychannel](C:\Users\liwei\Pictures\Screenshots\memorychannel.png)

### 2、Kafka Channel

```
将event存储到kafka集群。kafka提供了高可用和复制机制，因此如果flume实例或者kafka的实例挂掉，能保证event数据随时可用。kafka channel可以用于多种场景：
	1、与source、sink一起：给所有event提供一个可靠、高可用的channel
	2、与source、interceptor一起，没有sink：可以把所有的event写入到kafka的topic中，来给其他应用使用
	3、与sink一起，没有source：提供一种低延迟、容错高的方式将event发送到各种sink上，比如hdfs、hbase
eg:
a1.channels.channel1.type = org.apache.flume.channel.kafka.KafkaChannel
a1.channels.channel1.kafka.bootstrap.servers = kafka-1:9092,kafka-2:9092,kafka-3:9092
a1.channels.channel1.kafka.topic = channel1
a1.channels.channel1.kafka.consumer.group.id = flume-consumer
```

![kafkachannel](C:\Users\liwei\Pictures\Screenshots\kafkachannel.png)

### 3、File Channel（[Flume之 各种 Channel 的介绍及参数解析_flume channel-CSDN博客](https://blog.csdn.net/weixin_40727028/article/details/123998376)）

```
这个channel将event写入磁盘文件，与memory channel相比存储容量大，无数据丢失风险。File Channle 数据存储路径可以配置多磁盘文件路径，通过磁盘并行写入提高FileChannel 性能。Flume 将 Event 顺序写入到 File Channel 文件的末尾，在配置文件中通过设置 maxFileSize 参数配置数据文件大小，当被写入的文件大小达到上限时 Flume 会重新创建新的文件存储写入的 Event。当然数据文件数量也不会无限增长，当一个已关闭的只读数据文件中的 Event 被读取完成，并且 Sink 已经提交读取完成的事务，则 Flume 将删除存储该数据的文件。Flume 通过设置检查点和备份检查点实现在 Agent 重启之后快速将 File Channle 中的数据按顺序回放到内存中，保证在 Agent 失败重启后仍然能够快速安全地提供服务。
eg:
a1.channels = c1
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /mnt/flume/checkpoint
a1.channels.c1.dataDirs = /mnt/flume/data1,/mnt/flume/data2
```

![filechannel](C:\Users\liwei\Pictures\Screenshots\filechannel.png)

## 三、Sink

### 1、HDFS Sink

```
这个Sink将event写入hdfs，目前支持创建文本和序列文件，并且支持这两种文件的压缩。可以根据写入时间、文件大小或event数量定期滚动文件（关闭当前文件并创建新文件）。它还可以根据event自带的时间戳或系统时间等属性对数据进行分区。
eg:
a1.channels = c1
a1.sinks = k1
a1.sinks.k1.type = hdfs
a1.sinks.k1.channel = c1
a1.sinks.k1.hdfs.path = /flume/events/%y-%m-%d/%H%M/%S
a1.sinks.k1.hdfs.filePrefix = events-
a1.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.roundValue = 10
a1.sinks.k1.hdfs.roundUnit = minute
```

![hdfs转义字符](C:\Users\liwei\Pictures\Screenshots\hdfs转义字符.png)![hdfssink](C:\Users\liwei\Pictures\Screenshots\hdfssink.png)

### 2、Hive Sink

```
这个sink将包含分隔文本或JSON数据的event直接流式传输到Hive表或分区上。Event使用Hive事务进行写入，一旦一组event提交给hive，他们就会立即显示给hive查询。即将写入的目标分区既可以预先自己创建，也可以选择让flume创建。写入的event数据中的字段将映射到hive表中的相应列
eg:
a1.channels = c1
a1.channels.c1.type = memory
a1.sinks = k1
a1.sinks.k1.type = hive
a1.sinks.k1.channel = c1
a1.sinks.k1.hive.metastore = thrift://127.0.0.1:9083
a1.sinks.k1.hive.database = logsdb
a1.sinks.k1.hive.table = weblogs
a1.sinks.k1.hive.partition = asia,%{country},%y-%m-%d-%H-%M
a1.sinks.k1.useLocalTimeStamp = false
a1.sinks.k1.round = true
a1.sinks.k1.roundValue = 10
a1.sinks.k1.roundUnit = minute
a1.sinks.k1.serializer = DELIMITED
a1.sinks.k1.serializer.delimiter = "\t"
a1.sinks.k1.serializer.serdeSeparator = '\t'
a1.sinks.k1.serializer.fieldnames =id,,msg
```

![hivesink](C:\Users\liwei\Pictures\Screenshots\hivesink.png)![hivesinkserialize](C:\Users\liwei\Pictures\Screenshots\hivesinkserialize.png)
![](image-20230209112203413.png)