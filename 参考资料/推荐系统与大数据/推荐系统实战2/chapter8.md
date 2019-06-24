## 八 埋点日志处理

### 8.1 埋点日志介绍

埋点就是在应用中特定的流程收集一些信息，用来跟踪应用使用的状况，后续用来进一步优化产品或是提供运营的数据支撑。比如用于作为实现个性化推荐的数据支撑。

埋点方式主流的无非两种方式：

- 自行研发：在研发的产品中注入代码进行统计，并搭建起相应的后台查询和处理
- 第三方平台：第三方统计工具，如友盟、百度移动等

埋点日志主要分为类型：

- 曝光日志：商品被展示到页面被称为曝光，曝光日志也就是指商品一旦被展示出来，则记录一条曝光日志
  - 曝光时间
  - 曝光场景
  - 用户唯一标识
  - 商品ID
  - 商品类别ID
- 点击流日志：用户浏览、收藏、加购物车、购买、评论、搜索等行为记录日志
  - 被曝光时间：对应曝光日志的曝光时间(浏览)
  - 被曝光场景：对应曝光日志的曝光场景(浏览)
  - 用户唯一标识
  - 行为时间
  - 行为类型
  - 商品ID
  - 商品类别ID
  - 停留时长(浏览)
  - 评分(评论)
  - 搜索词(搜索)

### 8.2 埋点日志意义

用户行为偏好分析

- 利用点击流日志分析个体/群体用户的行为特征，预测出用户行为的偏好

统计指标分析

- 点击率：被点击的概率
  - 计算公式：点击次数/曝光次数。某商品共曝光/展示了100次，曝光后共被点击10次，点击率=10%
- 跳出率：用户访问一个页面后，之后没有再也没有其他操作，称为跳出
  - 计算公式：访问一次退出的访问量/总的访问量
  - 整体(整个板块/应用)跳出率
  - 单页面的跳出率。如某页面共有100个用户访问，其中有10个用户访问当前页面后就再也没有其他访问了，当前页面跳出率是10%
- 转化率：电商中的转化率计算：商品订单成交量/商品访问量。如某商品累计访问量是100个，最终提交订单的只有5个，那么该商品转化率就是5%

注意：跳出率和转化率中的访问量指的是独立用户访问量。独立用户访问量：首先独立用户访问量并不等价于来访问的总用户个数。独立用户访问量：比如某用户A在1月1日访问了商品1，1月2日又访问了商品1，那么这里商品1的访问量应算作1次，而不是2次。

### 8.3 埋点日志处理

- 埋点日志格式

```shell
cat /home/hadoop/code/meiduoSourceCode/logs/exposure.log
2018/12/30 03:19:41: exposure_timesteamp<1543519181> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>
2018/12/30 13:18:13: exposure_timesteamp<1543555093> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>
2018/12/30 13:18:13: exposure_timesteamp<1543555093> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>
2018/12/31 01:47:59: exposure_timesteamp<1543600079> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>
2018/12/31 01:49:20: exposure_timesteamp<1543600079> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>
2018/12/31 01:52:53: exposure_timesteamp<1543600373> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>
2018/12/31 01:57:47: exposure_timesteamp<1543600666> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>
```

- 埋点日志格式化

``` python
def get_logger(logger_name, path, level):
    # 创建logger
    logger = logging.getLogger(logger_name)
    logger.setLevel(level)

    # 创建formatter
    fmt = "%(asctime)s: %(message)s"
    datefmt = "%Y/%m/%d %H:%M:%S"
    formatter = logging.Formatter(fmt, datefmt)
    
    # 创建handler
    handler = logging.FileHandler(path)
    handler.setLevel(level)
    
    # 添加 handler 和 formatter 到 logger
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    
    return logger
```

- 处理曝光日志

``` python
import time
import logging

exposure_logger = get_logger("exposure", "/home/hadoop/code/meiduoSourceCode/logs/exposure.log", logging.DEBUG)

# 曝光日志
exposure_timesteamp = time.time()
exposure_loc = "detail"
uid = 1
sku_id = 1
cate_id = 1
```

- 创建日志

```python
exposure_logger.info("exposure_timesteamp<%d> exposure_loc<%s> uid<%d> sku_id<%d> cate_id<%d>"%(exposure_timesteamp, exposure_loc, uid, sku_id, cate_id))
```

- 点击日志

```python
import time
import logging

click_trace_logger = get_logger("click_trace", "/root/meiduoSourceCode/logs/click_trace.log", logging.DEBUG)
```

```python
# 模拟产生点击流日志
exposure_timesteamp = exposure_timesteamp
exposure_loc = exposure_loc
timesteamp = time.time()
behavior="pv" # pv fav cart buy share 
uid = 1
sku_id = 1
cate_id = 1
# 假设某点击流日志记录格式如下：
click_trace_logger.info("exposure_timesteamp<%d> exposure_loc<%s> timesteamp<%d> behavior<%s> uid<%d> sku_id<%d> cate_id<%d>"%(exposure_timesteamp, exposure_loc, timesteamp, behavior, uid, sku_id, cate_id))

```

### 8.4 Flume采集日志

#### 8.4.1 Flume概述

**概念**

Flume 是 Cloudera 提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的软件。 

Flume 核心

- 把数据从数据源(source)收集过来
- 将收集到的数据送到指定的目的地(sink)
- 为保证成功，在送到目的地(sink)之前，先缓存数据(channel),待数据到达目的地(sink)后，flume 再删除自己缓存的数据。

Flume优势

- 支持定制各类数据发送方/接受方，用于收集各类型数据/最终存储数据
- 通过简单配置即可实现一般的采集需求
- 特殊场景也具备良好的自定义扩展能力
- 适用于大部分的日常数据采集场景

#### 8.4.2 Flume运行机制

Flume 系统中核心的角色是 **agent**，agent 本身是一个 Java 进程，一般运行在日志收集节点。

![](/img/1540347254561.png)

每一个 agent 相当于一个数据传递员，内部有三个组件：

**Source**：采集源，用于跟数据源对接，以获取数据；

**Sink**：下沉地，采集数据的传送目的，用于往下一级 agent 传递数据或者往

最终存储系统传递数据；

**Channel**：agent 内部的数据传输通道，用于从 source 将数据传递到 sink；在整个数据的传输的过程中，流动的是 event，它是 Flume 内部数据传输的最基本单元。event 将传输的数据进行封装。如果是文本文件，通常是一行记录， event 也是事务的基本单位。event 从 source，流向 channel，再到 sink，本身为一个字节数组，并可携带 headers(头信息)信息。event 代表着一个数据的最小完整单元，从外部数据源来，向外部的目的地去。

一个完整的 event 包括：event headers、event body、event 信息，其中 event 信息就是 flume 收集到的日记记录。

flume的工作过程：先指定Source将数据进行采集，输出到Channel，Channel相当于内存存储，当Channel存储到一定值时，将数据写入到Sink中

**Flume的可配置子组件**

**Source**

- Avro Source 序列化数据源
- ThriftSource 序列化数据源
- Exec Source 执行Linux命令行的数据源
- NETCAT Source 通过指定端口，ip监控的数据源
- Kafka Source 直接对接Kafka的数据源
- 自定义Source

**Channel**

- Memory Channel
- File Channel
- Kafka Channel
- JDBC Channel

**Sink**

- HDFS Sink 写入到HDFS
- Hive Sink 写入到Hive
- Avro Sink 写入到序列化
- HBase Sinks 写入到HBase
  - HBase Sink 同步写入到HBase
  - Async HBase Sink 异步写入到Hbase

#### 8.4.3 采集目录到HDFS

采集需求：**服务器的某特定目录下，会不断产生新的文件，每当有新文件出现，就需要把文件采集到 HDFS 中去**

根据需求，首先定义以下 3 大要素

​	采集源，即 source——监控文件目录 :  **spooldir**

​	下沉目标，即 sink——HDFS 文件系统  :  **hdfs sink**

​	source 和 sink 之间的传递通道——channel，可用 file channel 也可以用内存 channel

配置文件编写：

```
#Name the components on this agent 
spool-memory-hdfs.sources = spool-source
spool-memory-hdfs.sinks = hdfs-sink
spool-memory-hdfs.channels = memory-channel

# Describe/configure the source
#flume的source采用spooldir时,目录下面不允许存放同名的文件，否则报错！
spool-memory-hdfs.sources.spool-source.type = spooldir
spool-memory-hdfs.sources.spool-source.spoolDir = /home/hadoop/tmp/log
spool-memory-hdfs.sources.spool-source.fileHeader = true

# Describe the sink 
spool-memory-hdfs.sinks.hdfs-sink.type = hdfs
spool-memory-hdfs.sinks.hdfs-sink.hdfs.path = /flume/events/%y-%m-%d/%H%M/
spool-memory-hdfs.sinks.hdfs-sink.hdfs.filePrefix = events-
spool-memory-hdfs.sinks.hdfs-sink.hdfs.round = true
spool-memory-hdfs.sinks.hdfs-sink.hdfs.roundValue = 10
spool-memory-hdfs.sinks.hdfs-sink.hdfs.roundUnit = minute
spool-memory-hdfs.sinks.hdfs-sink.hdfs.rollInterval = 3
spool-memory-hdfs.sinks.hdfs-sink.hdfs.rollSize = 20
spool-memory-hdfs.sinks.hdfs-sink.hdfs.rollCount = 5
spool-memory-hdfs.sinks.hdfs-sink.hdfs.batchSize = 1
spool-memory-hdfs.sinks.hdfs-sink.hdfs.useLocalTimeStamp = true
#生成的文件类型，默认是 Sequencefile，可用 DataStream，则为普通文本 
spool-memory-hdfs.sinks.hdfs-sink.hdfs.fileType = DataStream

# Use a channel which buffers events in memory 
spool-memory-hdfs.channels.memory-channel.type = memory
spool-memory-hdfs.channels.memory-channel.capacity = 1000
spool-memory-hdfs.channels.memory-channel.transactionCapacity = 100
 
# Bind the source and sink to the channel
spool-memory-hdfs.sources.spool-source.channels = memory-channel
spool-memory-hdfs.sinks.hdfs-sink.channel = memory-channel
```

启动：

`bin/flume-ng agent -f spool-memory-hdfs.properties -n spool-memory-hdfs -Dflume.root.logger=INFO,console`

Channel 参数解释：

- capacity：默认该通道中最大的可以存储的 event 数量

- trasactionCapacity：每次最大可以从 source 中拿到或者送到 sink 中的 event数量
  sinks参数解析：

- rollInterval：默认值：30

  hdfs sink 间隔多长将临时文件滚动成最终目标文件，单位：秒；

  如果设置成 0，则表示不根据时间来滚动文件；

  注：滚动（roll）指的是，hdfs sink 将临时文件重命名成最终目标文件，并新打开一个临时文件来写入数据；

- rollSize

  默认值：1024

  当临时文件达到该大小（单位：bytes）时，滚动成目标文件；

  如果设置成 0，则表示不根据临时文件大小来滚动文件；

- rollCount

  默认值：10

  当 events 数据达到该数量时候，将临时文件滚动成目标文件；

  如果设置成 0，则表示不根据 events 数据来滚动文件；

- round

  默认值：false

  是否启用时间上的“舍弃”，这里的“舍弃”，类似于“四舍五入”。

- roundValue

  默认值：1

  时间上进行“舍弃”的值；

- roundUnit

  默认值：seconds

  时间上进行“舍弃”的单位，包含：second,minute,hour

  示例：

  a1.sinks.k1.hdfs.path = /flume/events/%y-%m-%d/%H%M/%S

  a1.sinks.k1.hdfs.round = true

  a1.sinks.k1.hdfs.roundValue = 10

  a1.sinks.k1.hdfs.roundUnit = minute

  当时间为 2015-10-16 17:38:59 时候，hdfs.path 依然会被解析为：

  /flume/events/20151016/17:30/00

  因为设置的是舍弃 10 分钟内的时间，因此，该目录每 10 分钟新生成一个。

#### 8.4.4 采集文件到HDFS

采集需求：**业务系统生成的日志，日志内容不断增加，需要把追加到日志文件中的数据实时采集到 hdfs**

根据需求，首先定义以下 3 大要素

​	采集源，即 source——监控文件内容更新 :  exec  ‘tail -F file’

​	下沉目标，即 sink——HDFS 文件系统  :  hdfs sink

​	Source 和 sink 之间的传递通道——channel，可用 file channel 也可以用内存 channel

配置文件编写：

```
#Name the components on this agent 
exec-memory-hdfs.sources = exec-source
exec-memory-hdfs.sinks = hdfs-sink
exec-memory-hdfs.channels = memory-channel

# Describe/configure the source
exec-memory-hdfs.sources.exec-source.type = exec
exec-memory-hdfs.sources.exec-source.command = tail -F /home/hadoop/tmp/logs/test.log

# Describe the sink 
exec-memory-hdfs.sinks.hdfs-sink.type = hdfs
exec-memory-hdfs.sinks.hdfs-sink.hdfs.path = /flume/events/%y-%m-%d/%H%M/
exec-memory-hdfs.sinks.hdfs-sink.hdfs.filePrefix = events-
exec-memory-hdfs.sinks.hdfs-sink.hdfs.round = true
exec-memory-hdfs.sinks.hdfs-sink.hdfs.roundValue = 10
exec-memory-hdfs.sinks.hdfs-sink.hdfs.roundUnit = minute
exec-memory-hdfs.sinks.hdfs-sink.hdfs.rollInterval = 3
exec-memory-hdfs.sinks.hdfs-sink.hdfs.rollSize = 20
exec-memory-hdfs.sinks.hdfs-sink.hdfs.rollCount = 5
exec-memory-hdfs.sinks.hdfs-sink.hdfs.batchSize = 1
exec-memory-hdfs.sinks.hdfs-sink.hdfs.useLocalTimeStamp = true
#生成的文件类型，默认是 Sequencefile，可用 DataStream，则为普通文本 
exec-memory-hdfs.sinks.hdfs-sink.hdfs.fileType = DataStream
#生成的文件类型，默认是 Sequencefile，可用 DataStream，则为普通文本 
exec-memory-hdfs.sinks.hdfs-sink.hdfs.fileType = DataStream

# Use a channel which buffers events in memory 
exec-memory-hdfs.channels.memory-channel.type = memory
exec-memory-hdfs.channels.memory-channel.capacity = 1000
exec-memory-hdfs.channels.memory-channel.transactionCapacity = 100

 
# Bind the source and sink to the channel
exec-memory-hdfs.sources.exec-source.channels = memory-channel
exec-memory-hdfs.sinks.hdfs-sink.channel = memory-channel
```

### 8.5 离线日志处理

- 查看离线日志存储位置

``` shell
!hadoop fs -ls /project2-meiduo-rs/logs/click-trace
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/root/bigdata/hadoop-2.9.1/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/root/bigdata/apache-hive-2.3.4-bin/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
Found 2 items
drwxr-xr-x   - root supergroup          0 2018-12-30 13:28 /project2-meiduo-rs/logs/click-trace/18-12-30
drwxr-xr-x   - root supergroup          0 2018-12-31 02:51 /project2-meiduo-rs/logs/click-trace/18-12-31

```

- 通过spark读取点击日志

```python
date = "18-12-01"
click_trace = spark.read.csv("hdfs://localhost:8020/project2-meiduo-rs/logs/click-trace/%s"%date)
click_trace.show(truncate=False)
```

终端显示:

```shell
+-------------------------------------------------------------------------------------------------------------------------------------------------------+
|_c0                                                                                                                                                    |
+-------------------------------------------------------------------------------------------------------------------------------------------------------+
|2018/11/30 13:21:08: exposure_timesteamp<1543555093> exposure_loc<detail> timesteamp<1543555268> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/11/30 13:21:08: exposure_timesteamp<1543555093> exposure_loc<detail> timesteamp<1543555268> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/11/30 13:28:37: exposure_timesteamp<1543555093> exposure_loc<detail> timesteamp<1543555717> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/11/30 13:28:37: exposure_timesteamp<1543555093> exposure_loc<detail> timesteamp<1543555717> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/11/30 13:28:37: exposure_timesteamp<1543555093> exposure_loc<detail> timesteamp<1543555717> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/11/30 13:28:37: exposure_timesteamp<1543555093> exposure_loc<detail> timesteamp<1543555717> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/11/30 13:28:37: exposure_timesteamp<1543555093> exposure_loc<detail> timesteamp<1543555717> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:34:28: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543602868> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:34:28: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543602868> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:34:29: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543602869> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:34:29: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543602869> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:34:30: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543602870> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:34:30: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543602870> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:34:31: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543602871> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:17:30: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543601850> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:17:30: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543601850> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:17:54: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543601850> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:17:54: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543601850> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:30:54: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543602654> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
|2018/12/01 02:30:54: exposure_timesteamp<1543601846> exposure_loc<detail> timesteamp<1543602654> behavior<pv> uid<1> sku_id<1> cate_id<1> stay_time<60>|
+-------------------------------------------------------------------------------------------------------------------------------------------------------+
only showing top 20 rows
```

- spark读取曝光日志

``` python
date = "18-12-01"
exposure = spark.read.csv("hdfs://localhost:8020/project2-meiduo-rs/logs/exposure/%s"%date)
exposure.show(truncate=False)
```

终端显示

``` shell
+-----------------------------------------------------------------------------------------------------+
|_c0                                                                                                  |
+-----------------------------------------------------------------------------------------------------+
|2018/11/30 03:19:41: exposure_timesteamp<1543519181> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>|
|2018/11/30 13:18:13: exposure_timesteamp<1543555093> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>|
|2018/11/30 13:18:13: exposure_timesteamp<1543555093> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>|
|2018/12/01 01:47:59: exposure_timesteamp<1543600079> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>|
|2018/12/01 01:49:20: exposure_timesteamp<1543600079> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>|
|2018/12/01 01:52:53: exposure_timesteamp<1543600373> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>|
|2018/12/01 01:57:47: exposure_timesteamp<1543600666> exposure_loc<detail> uid<1> sku_id<1> cate_id<1>|
+-----------------------------------------------------------------------------------------------------+
```

- 处理曝光日志

```python
import re
from pyspark.sql import Row

def map(row):
    match = re.search("\
exposure_timesteamp<(?P<exposure_timesteamp>.*?)> \
exposure_loc<(?P<exposure_loc>.*?)> \
timesteamp<(?P<timesteamp>.*?)> \
behavior<(?P<behavior>.*?)> \
uid<(?P<uid>.*?)> \
sku_id<(?P<sku_id>.*?)> \
cate_id<(?P<cate_id>.*?)> \
", row._c0)

    row = Row(exposure_timesteamp=match.group("exposure_timesteamp"),
                exposure_loc=match.group("exposure_loc"),
                timesteamp=match.group("timesteamp"),
                behavior=match.group("behavior"),
                uid=match.group("uid"),
                sku_id=match.group("sku_id"),
                cate_id=match.group("cate_id"),
                )
    
    return row
click_trace.rdd.map(map).toDF().show()
    
```

终端数据显示

``` shell
+--------+-------+------------+-------------------+------+---------+----------+---+
|behavior|cate_id|exposure_loc|exposure_timesteamp|sku_id|stay_time|timesteamp|uid|
+--------+-------+------------+-------------------+------+---------+----------+---+
|      pv|      1|      detail|         1543555093|     1|       60|1543555268|  1|
|      pv|      1|      detail|         1543555093|     1|       60|1543555268|  1|
|      pv|      1|      detail|         1543555093|     1|       60|1543555717|  1|
|      pv|      1|      detail|         1543555093|     1|       60|1543555717|  1|
|      pv|      1|      detail|         1543555093|     1|       60|1543555717|  1|
|      pv|      1|      detail|         1543555093|     1|       60|1543555717|  1|
|      pv|      1|      detail|         1543555093|     1|       60|1543555717|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543602868|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543602868|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543602869|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543602869|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543602870|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543602870|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543602871|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543601850|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543601850|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543601850|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543601850|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543602654|  1|
|      pv|      1|      detail|         1543601846|     1|       60|1543602654|  1|
+--------+-------+------------+-------------------+------+---------+----------+---+
only showing top 20 rows
```

- 处理点击日志

```python
import re
from pyspark.sql import Row

def map(row):
    match = re.search("\
exposure_timesteamp<(?P<exposure_timesteamp>.*?)> \
exposure_loc<(?P<exposure_loc>.*?)> \
uid<(?P<uid>.*?)> \
sku_id<(?P<sku_id>.*?)> \
cate_id<(?P<cate_id>.*?)>", row._c0)

    row = Row(exposure_timesteamp=match.group("exposure_timesteamp"),
                exposure_loc=match.group("exposure_loc"),
                uid=match.group("uid"),
                sku_id=match.group("sku_id"),
                cate_id=match.group("cate_id"))
    
    return row
exposure.rdd.map(map).toDF().show()
```

终端显示

```python
+-------+------------+-------------------+------+---+
|cate_id|exposure_loc|exposure_timesteamp|sku_id|uid|
+-------+------------+-------------------+------+---+
|      1|      detail|         1543519181|     1|  1|
|      1|      detail|         1543555093|     1|  1|
|      1|      detail|         1543555093|     1|  1|
|      1|      detail|         1543600079|     1|  1|
|      1|      detail|         1543600079|     1|  1|
|      1|      detail|         1543600373|     1|  1|
|      1|      detail|         1543600666|     1|  1|
+-------+------------+-------------------+------+---+
```

- 利用点击流日志中行为是"pv"的数据同时`cate_id|exposure_loc|exposure_timesteamp|sku_id|uid`一一对应的数据进行对应，最终就能得出：所有曝光的商品中，哪些商品被用户浏览了，哪些没有被浏览

