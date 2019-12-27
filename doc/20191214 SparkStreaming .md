## sparkStream

Spark Streaming is an extension of the core Spark API that enables scalable, high-throughput, fault-tolerant stream processing of live data streams.

翻译过来的意思是：Spark Streaming是核心Spark API的扩展，可实现实时数据流的可伸缩，高吞吐量，容错流处理。

![](https://spark.apache.org/docs/2.3.1/img/streaming-arch.png)



## SparkStreaming & Storm  比较

1. **Storm 是纯实时处理数据， SparkStreaming 准实时处理数据**
   Storm 延时少，数据量少； SparkStreaming 延时高，数据量微处理

2. **Storm 擅长处理汇总型业务数据， SparkStreaming 擅长处理复杂业务， SparkStreaming 不仅可以使用 Spark Core， Spark SQL**

3. Storm 事务相对完善， SparkStreaming 也比较完善

4. Storm 支持动态资源调度， Sparing 1.2+也是支持	

   注意: SparkStreaming 支持动态资源调度，但是不建议开启

Storm ：水龙头 ，SparkStreaming： 水闸， Flink ： 流



## SparkStreaming 运行机制

SparkStreaming 7*24 不间断运行， SparkStreaming 启动之后首先启动一个 job 接收数据， 将 *一段时间*  内接收的数据放入一个 batch 中， 这里*一段时间*  就是 batchInterval， batch 又被封装到 RDD 中， RDD 又被封装到 DStream 中。 DStream 有自己的 Transformaction 类算子，**懒执行**，需要 DStream 的 OutputOperator 类算子触发执行。

- 假设 batchInterval = 5s， 集群处理一批次数据的时间是 3s

  启动集群， 0-5s 只是接收数据， 5-8s 一边接收数据， 一边处理数据， 8-10s 只是接收数据 .....

- 假设 batchInterval = 5s ，集群处理一批次数据的数据是 8s 

  启动集群， 0-5s 只是接收数据， 5-10s 一边处理数据， 10-13s 一边接收数据一边处理数据 13-15s 一边接收数据一边处理数据 ....

结论： 最好 batchInterval 生成一批次， batchInterval 时间处理完一批次

![](http://img.zwer.xyz/blog/20191214143223.png)



## SparkStreaming 入门案例

这里的数据来源是 socket 产生的，通过在虚拟机 Linux 执行 `nc -lk 9999`

下面代码执行的任务时： 接收 socket 中数据，统计单词词频

```scala

/**
  * SparkStreaming 注意：
  * 1.需要设置local[2]，因为一个线程是读取数据，一个线程是处理数据
  * 2.创建StreamingContext两种方式，如果采用的是StreamingContext(SparkConf/SparkContext,Durations.seconds(5))这种方式
  * 3.Durations 批次间隔时间的设置需要根据集群的资源情况以及监控每一个job的执行时间来调节出最佳时间。
  * 4.SparkStreaming所有业务处理完成之后需要有一个output operato操作
  * 5.StreamingContext.start()straming框架启动之后是不能在次添加业务逻辑
  * 6.StreamingContext.stop()无参的stop方法会将sparkContext一同关闭，stop(false) ,默认为true，会一同关闭
  * 7.StreamingContext.stop()停止之后是不能在调用start
  */
object WordCountFromSocket {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setAppName("WordCountOnLine").setMaster("local[2]")
    val ssc = new StreamingContext(conf,Durations.seconds(5))
    ssc.sparkContext.setLogLevel("ERROR")
    //使用new StreamingContext(conf,Durations.seconds(5)) 这种方式默认会创建SparkContext
//    val sc = new SparkContext(conf)
    //从ssc中获取SparkContext()
//    val context: SparkContext = ssc.sparkContext

    val lines: ReceiverInputDStream[String] = ssc.socketTextStream("mynode5",9999)
    val words: DStream[String] = lines.flatMap(line=>{line.split(" ")})
    val pairWords: DStream[(String, Int)] = words.map(word=>{(word,1)})
    val result: DStream[(String, Int)] = pairWords.reduceByKey((v1, v2)=>{v1+v2})

    result.print()

    /**
      * foreachRDD 注意事项：
      * 1.foreachRDD中可以拿到DStream中的RDD，对RDD进行操作，但是一点要使用RDD的action算子触发执行，不然DStream的逻辑也不会执行
      * 2.froeachRDD算子内，拿到的RDD算子操作外，这段代码是在Driver端执行的，可以利用这点做到动态的改变广播变量
      *
      */
//    result.foreachRDD(wordCountRDD=>{
//
//      println("******* produce in Driver *******")
//
//      val sortRDD: RDD[(String, Int)] = wordCountRDD.sortByKey(false)
//      val result: RDD[(String, Int)] = sortRDD.filter(tp => {
//        println("******* produce in Executor *******")
//        true
//      })
//      result.foreach(println)
//    })

    ssc.start()
    ssc.awaitTermination()
    ssc.stop(false)

  }
}

```



## SparkStreaming 中的算子

### foreachRDD 算子

> 可以获取 DStream 中的  RDD ，对 RDD 进行转换，最后需要返回 RDD 使用 Action 算子触发操作 

利用 foreachRDD 做到动态的改变广播变量，可用于查找指定数据相关

```scala
resultUpdate.foreachRDD(rdd => {
    println("*************") // 在 Driver 端执行
    rdd.foreach(println)    // 在 Executor 端执行
})
```

### updateStateByKey 算子

统计总的数据

```scala
/**
 *  测试 算子 updateStateByKey
 *  用于统计自 SparkStreaming 以来的数据总和
 */
object UpdateStateByKeyDemo {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf()
    conf.setMaster("local[2]")
    conf.setAppName("updateStateByKeyDemo")

    val smc: StreamingContext = new StreamingContext(conf, Durations.seconds(5))
    // 通过 StreamingContext 获取 SparkContext 上下文
    smc.sparkContext.setLogLevel("error")
    // The checkpoint directory has not been set.
    // Please set it by StreamingContext.checkpoint().
    /**
     *  所有 key 默认保存在内存中， 最终保存在 checkpoint 目录中
     *  多久将内存中的状态保存到 checkpoint 中一次？
     *  1. 当 batchInterval <= 10s 时，10s 保存一次
     *  2. 当 batchInterval > 10s 时， batchInterval 保存一次
     */
    smc.sparkContext.setCheckpointDir("./data/ck")

    val lines: ReceiverInputDStream[String] = smc.socketTextStream("node05", 9999)
    val words: DStream[String] = lines.flatMap(_.split(" "))
    val wordPairs: DStream[(String, Int)] = words.map((_, 1))
    val result: DStream[(String, Int)] = wordPairs.reduceByKey((_ + _))
    /**
     *  使用 updateStateByKey 算子
     */
    val resultUpdate: DStream[(String, Int)] = result.updateStateByKey((seqs: Seq[Int], option: Option[Int]) => {
      var state: Int = option.getOrElse(0)
      for (seq <- seqs) {
        state += seq
      }
      Option(state)
    })
    resultUpdate.print()
    /*
    resultUpdate.foreachRDD(rdd => {
      println("*************") // 在 Driver 端执行
      rdd.foreach(println)    // 在 Executor 端执行
    })
    */
    smc.start()
    smc.awaitTermination()

    smc.stop()

  }
}

```

### reduceByKeyAndWindow 算子

统计每过一定时间最近一段时间内的数据

```scala
/**
  * UpdateStateByKey 根据key更新状态
  *   1、为Spark Streaming中每一个Key维护一份state状态，state类型可以是任意类型的， 可以是一个自定义的对象，那么更新函数也可以是自定义的。
  *   2、通过更新函数对该key的状态不断更新，对于每个新的batch而言，Spark Streaming会在使用updateStateByKey的时候为已经存在的key进行state的状态更新
  */
object UpdateStateByKey {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setMaster("local[2]")
    conf.setAppName("UpdateStateByKey")
    val ssc = new StreamingContext(conf,Durations.seconds(5))
    //设置日志级别
    ssc.sparkContext.setLogLevel("ERROR")
    val lines: ReceiverInputDStream[String] = ssc.socketTextStream("node5",9999)
    val words: DStream[String] = lines.flatMap(line=>{line.split(" ")})
    val pairWords: DStream[(String, Int)] = words.map(word => {(word, 1)})

    /**
      * 根据key更新状态，需要设置 checkpoint来保存状态
      * 默认key的状态在内存中 有一份，在checkpoint目录中有一份。
      *
      *    多久会将内存中的数据（每一个key所对应的状态）写入到磁盘上一份呢？
      * 	      如果你的batchInterval小于10s  那么10s会将内存中的数据写入到磁盘一份
      * 	      如果bacthInterval 大于10s，那么就以bacthInterval为准
      *
      *    这样做是为了防止频繁的写HDFS
      */
    ssc.checkpoint("./data/streamingCheckpoint")
//    ssc.sparkContext.setCheckpointDir("./data/streamingCheckpoint")
    /**
      * currentValues :当前批次某个 key 对应所有的value 组成的一个集合
      * preValue : 以往批次当前key 对应的总状态值
      */
    val result: DStream[(String, Int)] = pairWords.updateStateByKey((currentValues: Seq[Int], preValue: Option[Int]) => {
      var totalValues = 0
      if (!preValue.isEmpty) {
        totalValues += preValue.get
      }
      for(value <- currentValues){
        totalValues += value
      }

      Option(totalValues)
    })
    result.print()
    ssc.start()
    ssc.awaitTermination()
    ssc.stop()

  }
}

```

###  transform 算子

可以获取 DStream 中的  RDD ，对 RDD 进行转换，最后需要返回 RDD

```scala	
object TransformBlackList {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setAppName("transform")
    conf.setMaster("local[2]")
    val ssc = new StreamingContext(conf,Durations.seconds(5))
//    ssc.sparkContext.setLogLevel("Error")
    /**
      * 广播黑名单
      */
    val blackList: Broadcast[List[String]] = ssc.sparkContext.broadcast(List[String]("zhangsan","lisi"))

    /**
      * 从实时数据【"hello zhangsan","hello lisi"】中发现 数据的第二位是黑名单人员，过滤掉
      */
    val lines: ReceiverInputDStream[String] = ssc.socketTextStream("node5",9999)
    val pairLines: DStream[(String, String)] = lines.map(line=>{(line.split(" ")(1),line)})
    /**
      * transform 算子可以拿到DStream中的RDD，对RDD使用RDD的算子操作，但是最后要返回RDD，返回的RDD又被封装到一个DStream
      *  transform中拿到的RDD的算子外，代码是在Driver端执行的。可以做到动态的改变广播变量
      */
    val resultDStream: DStream[String] = pairLines.transform((pairRDD:RDD[(String,String)]) => {
      println("++++++ Driver Code +++++++")
      val filterRDD: RDD[(String, String)] = pairRDD.filter(tp => {
        val nameList: List[String] = blackList.value
        !nameList.contains(tp._1)
      })
      val returnRDD: RDD[String] = filterRDD.map(tp => tp._2)
      returnRDD
    })

    resultDStream.print()

    ssc.start()
    ssc.awaitTermination()
    ssc.stop()


  }
}

```

### 监控某个目录

```scala
/**
 *  SparkStreaming 测试
 *  在 node05 上使用  nc -lk 9999
 *  任务：输入的数据，统计单词词频
 */
object SparkStreamingDemo {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setMaster("local")// 表示在本地模拟 2 个线程
    conf.setAppName("sparkStreamingDemo")

    //val sc: SparkContext = new SparkContext(conf)
    // 创建 SparkStreaming 上下文第一种方式
    //val smc: StreamingContext = new StreamingContext(sc, Durations.seconds(5))
    // 创建 SparkStreaming 上下文的第二种方式, 注意：这种方式前面不需要创建 Spark
    val smc: StreamingContext = new StreamingContext(conf, Durations.seconds(5))
    smc.sparkContext.setLogLevel("Error")

    /*val lines: ReceiverInputDStream[String] =
            smc.socketTextStream("node05", 9999)*/
    val lines = smc.textFileStream("./data/copyedfiledir")

    val words: DStream[String] = lines.flatMap(line => line.split(" "))
    val wordPairs: DStream[(String, Int)] = words.map(word => (word, 1))
    val result: DStream[(String, Int)] = wordPairs.reduceByKey((v1, v2) => {v1 + v2})
    // 测试打印输出
    result.print()

    smc.start()
    smc.awaitTermination()

  }
}
```

## SparkStreaming Driver HA 

**实现 Driver 高可用的两种方式**

1. 在提交 application 的时候， 添加 --supervise 选项， 如果 Driver 挂掉，会自动启动一个 Driver
2. 代码层面恢复 Driver（StreamingContext）

**下面代码用于演示在代码层面上实现恢复**

```scala
/**
  * Driver HA ：
  * 1.在提交application的时候  添加 --supervise 选项  如果Driver挂掉 会自动启动一个Driver
  * 2.代码层面恢复Driver(StreamingContext)
  *
  */
object SparkStreamingDriverHA {
  //设置checkpoint目录
  val ckDir = "./data/streamingCheckpoint"
  def main(args: Array[String]): Unit = {
    /**
      * StreamingContext.getOrCreate(ckDir,CreateStreamingContext)
      *   这个方法首先会从ckDir目录中获取StreamingContext【 因为StreamingContext是序列化存储在Checkpoint目录中，恢复时会尝试反序列化这些objects。
      *   如果用修改过的class可能会导致错误，此时需要更换checkpoint目录或者删除checkpoint目录中的数据，程序才能起来。】
      *
      *   若能获取回来StreamingContext,就不会执行CreateStreamingContext这个方法创建，否则就会创建
      */
    val ssc: StreamingContext = StreamingContext.getOrCreate(ckDir,CreateStreamingContext)
    ssc.start()
    ssc.awaitTermination()
    ssc.stop()

  }

  def CreateStreamingContext() = {
    println("=======Create new StreamingContext =======")
    val conf = new SparkConf()
    conf.setMaster("local")
    conf.setAppName("DriverHA")
    val ssc: StreamingContext = new StreamingContext(conf,Durations.seconds(5))
    ssc.sparkContext.setLogLevel("Error")

    /**
      *   默认checkpoint 存储：
      *     1.配置信息
      *   	2.DStream操作逻辑
      *   	3.batch执行的进度 或者【offset】
      */
    ssc.checkpoint(ckDir)
    val lines: DStream[String] = ssc.textFileStream("./data/streamingCopyFile")
    val words: DStream[String] = lines.flatMap(line=>{line.trim.split(" ")})
    val pairWords: DStream[(String, Int)] = words.map(word=>{(word,1)})
    val result: DStream[(String, Int)] = pairWords.reduceByKey((v1:Int, v2:Int)=>{v1+v2})

    // result.print()

    /**
      * 更改逻辑
      */
    result.foreachRDD(pairRDD=>{
      pairRDD.filter(one=>{
        println("*********** filter *********")
        true
      }).foreach(println)
    })

    ssc  //返回值
  }
}

```



## Kafka 消息中间件

### Kafka 的好处

- 解耦
- 缓冲

### 面试题

- 怎么保证消息者消息的顺序与生产者生产的顺序相同？

  生产者生产消息将消息放入同一个 partition 中，可以根据指定分区号或者指定相同 key 的方式

- 一个 topic 有多个 partition 的好处是什么？ 

  提高 kafka 集群的并发度

### kafka 架构图

<img src="http://img.zwer.xyz/blog/20191215154225.png" style="zoom:50%;" />

Kafka 分布式消息队列，默认将数据存储在磁盘，默认保存 7 天

**Producer**：  消息的生产者，两种生产模式： 1. 轮询 2.基于 key 的 hash

**Broker**： 组成 kafka 集群的节点，由 zookeeper 协调管理，Broker 负责消息的存储和读写，一个 broker 可以管理多个 Partition

**Topic**： 一类消息，消息队列，Topic 是由 Partition 组成的，多个 Partition 是为了做并行，一个 Topic 是由几个 Partition组成可以在创建 Topic 时指定

**Partition**：Partition 是组成 Topic 单元， Partition 对应磁盘目录，一个 Partition 中的消息是强有序的， Partition 有副本，可以在创建 Topic 的时指定。每个 Partition 是多个由多个消费者组内的一个消费者同时消费。Partition 是由Broker 管理，每个Partition 只能由一个 Broker 来管理

**Consumer**： 每个消费者都有自己的消费者组，不同的消费者组同一个 topic 互不影响。同一个消费者组的不同消费者同一个 Topic 时，这个 Topic 中相同的数据只能被消费一次。

**Zookeeper**: 协调 Broker,存储元数据： Broker，Topic，Partition

在 Zookeeper 0.8.2 之前，Zookeeper 还可以存储消费者 offset 



## SparkStream 与 Kafka 整合

### 从 Kafka 中消费消息

```scala
package com.bjsxt.scalaspark.streaming.streamingOnKafka

import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.{DStream, InputDStream}
import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe
import org.apache.spark.streaming.kafka010.{CanCommitOffsets, HasOffsetRanges, KafkaUtils, OffsetRange}
import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent
import org.apache.spark.streaming.{Durations, StreamingContext}

/**
  * SparkStreaming2.3版本 读取kafka 中数据 ：
  *  1.采用了新的消费者api实现，类似于1.6中SparkStreaming 读取 kafka Direct模式。并行度 一样。
  *  2.因为采用了新的消费者api实现，所有相对于1.6的Direct模式【simple api实现】 ，api使用上有很大差别。未来这种api有可能继续变化
  *  3.kafka中有两个参数：
  *      heartbeat.interval.ms：这个值代表 kafka集群与消费者之间的心跳间隔时间，kafka 集群确保消费者保持连接的心跳通信时间间隔。这个时间默认是3s.
  *             这个值必须设置的比session.timeout.ms 小，一般设置不大于 session.timeout.ms  的1/3
  *      session.timeout.ms ：
  *             这个值代表消费者与kafka之间的session 会话超时时间，如果在这个时间内，kafka 没有接收到消费者的心跳【heartbeat.interval.ms 控制】，
  *             那么kafka将移除当前的消费者。这个时间默认是30s。
  *             这个时间位于配置 group.min.session.timeout.ms【6s】 和 group.max.session.timeout.ms【300s】之间的一个参数,
  *             如果SparkSteaming 批次间隔时间大于5分钟，也就是大于300s,那么就要相应的调大group.max.session.timeout.ms 这个值。
  *  4.大多数情况下，SparkStreaming读取数据使用 LocationStrategies.PreferConsistent 这种策略，这种策略会将分区均匀的分布在集群的Executor之间。
  *    如果Executor在kafka 集群中的某些节点上，可以使用 LocationStrategies.PreferBrokers 这种策略，那么当前这个Executor 中的数据会来自当前broker节点。
  *    如果节点之间的分区有明显的分布不均，可以使用 LocationStrategies.PreferFixed 这种策略,可以通过一个map 指定将topic分区分布在哪些节点中。
  *
  *  5.新的消费者api 可以将kafka 中的消息预读取到缓存区中，默认大小为64k。默认缓存区在 Executor 中，加快处理数据速度。
  *     可以通过参数 spark.streaming.kafka.consumer.cache.maxCapacity 来增大，也可以通过spark.streaming.kafka.consumer.cache.enabled 设置成false 关闭缓存机制。
  *     "注意：官网中描述这里建议关闭，在读取kafka时如果开启会有重复读取同一个topic partition 消息的问题，报错：KafkaConsumer is not safe for multi-threaded access"
  *
  *  6.关于消费者offset
  *    1).如果设置了checkpoint ,那么offset 将会存储在checkpoint中。
  *     这种有缺点: 第一，当从checkpoint中恢复数据时，有可能造成重复的消费。
  *                第二，当代码逻辑改变时，无法从checkpoint中来恢复offset.
  *    2).依靠kafka 来存储消费者offset,kafka 中有一个特殊的topic 来存储消费者offset。新的消费者api中，会定期自动提交offset。这种情况有可能也不是我们想要的，
  *       因为有可能消费者自动提交了offset,但是后期SparkStreaming 没有将接收来的数据及时处理保存。这里也就是为什么会在配置中将enable.auto.commit 设置成false的原因。
  *       这种消费模式也称最多消费一次，默认sparkStreaming 拉取到数据之后就可以更新offset,无论是否消费成功。自动提交offset的频率由参数auto.commit.interval.ms 决定，默认5s。
  *       *如果我们能保证完全处理完业务之后，可以后期异步的手动提交消费者offset.
  *       注意：这种模式也有弊端，这种将offset存储在kafka中方式，参数offsets.retention.minutes=1440控制offset是否过期删除，默认将offset信息保存一天，
  *       如果停机没有消费达到时长，存储在kafka中的消费者组会被清空，offset也就被清除了。
  *    3).自己存储offset,这样在处理逻辑时，保证数据处理的事务，如果处理数据失败，就不保存offset，处理数据成功则保存offset.这样可以做到精准的处理一次处理数据。
  *
  */
object SparkStreamingOnKafkaDirect {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setMaster("local")
    conf.setAppName("SparkStreamingOnKafkaDirect")
    val ssc = new StreamingContext(conf,Durations.seconds(5))
    //设置日志级别
//    ssc.sparkContext.setLogLevel("ERROR")

    val kafkaParams = Map[String, Object](
      "bootstrap.servers" -> "node02:9092,nod03:9092,node04:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "MyGroupId",//
      /**
        *
        *  earliest ：当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始
        *  latest：自动重置偏移量为最大偏移量【默认】*
        *  none:没有找到以前的offset,抛出异常
        */
      "auto.offset.reset" -> "earliest",
      /**
        * 当设置 enable.auto.commit为false时，不会自动向kafka中保存消费者offset.需要异步的处理完数据之后手动提交
        */
      "enable.auto.commit" -> (false: java.lang.Boolean)//默认是true
    )

    val topics = Array[String]("mytopic")
    val stream: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream[String, String](
      ssc,
      PreferConsistent,//消费策略
      Subscribe[String, String](topics, kafkaParams)
    )

    val transStrem: DStream[String] = stream.map((record:ConsumerRecord[String, String]) => {
      val key_value = (record.key, record.value)
      println("receive message key = "+key_value._1)
      println("receive message value = "+key_value._2)
      key_value._2
    })
    val wordsDS: DStream[String] = transStrem.flatMap(line=>{line.split("\t")})
    val result: DStream[(String, Int)] = wordsDS.map((_,1)).reduceByKey(_+_)
    result.print()

    /**
      * 以上业务处理完成之后，异步的提交消费者offset,这里将 enable.auto.commit 设置成false,就是使用kafka 自己来管理消费者offset
      * 注意这里，获取 offsetRanges: Array[OffsetRange] 每一批次topic 中的offset时，必须从 源头读取过来的 stream中获取，不能从经过stream转换之后的DStream中获取。
      */
    stream.foreachRDD { rdd =>
      val offsetRanges: Array[OffsetRange] = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
      // some time later, after outputs have completed
      for(or <- offsetRanges){
        println(s"current topic = ${or.topic},partition = ${or.partition},fromoffset = ${or.fromOffset},untiloffset=${or.untilOffset}")
      }
      stream.asInstanceOf[CanCommitOffsets].commitAsync(offsetRanges)
    }
    ssc.start()
    ssc.awaitTermination()
    ssc.stop()
  }
}

```

### 使用 kafka 生产消息

```scala
package com.bjsxt.scalaspark.streaming.streamingOnKafka

import java.text.SimpleDateFormat
import java.util.{Date, Properties}

import org.apache.kafka.clients.producer.{KafkaProducer, ProducerRecord}

import scala.util.Random

/**
  * 向 kafka 中生产数据
  */
object ProduceDataToKafka {
  def main(args: Array[String]): Unit = {
    val props = new Properties()
    props.put("bootstrap.servers", "node02:9092,node03:9092,node04:9092")
    props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer")
    props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer")

    val producer = new KafkaProducer[String,String](props)
    var counter = 0
    var keyFlag = 0
    while(true){
      counter +=1
      keyFlag +=1
      val content: String = userlogs()
      producer.send(new ProducerRecord[String, String]("mytopic", s"key-$keyFlag", content))
      if(0 == counter%100){
        counter = 0
        Thread.sleep(5000)
      }
    }

    producer.close()
  }

  def userlogs()={
    val userLogBuffer = new StringBuffer("")
    val timestamp = new Date().getTime();
    var userID = 0L
    var pageID = 0L

    //随机生成的用户ID
    userID = Random.nextInt(2000)

    //随机生成的页面ID
    pageID =  Random.nextInt(2000);

    //随机生成Channel
    val channelNames = Array[String]("Spark","Scala","Kafka","Flink","Hadoop","Storm","Hive","Impala","HBase","ML")
    val channel = channelNames(Random.nextInt(10))

    val actionNames = Array[String]("View", "Register")
    //随机生成action行为
    val action = actionNames(Random.nextInt(2))

    val dateToday = new SimpleDateFormat("yyyy-MM-dd").format(new Date())
    userLogBuffer.append(dateToday)
      .append("\t")
      .append(timestamp)
      .append("\t")
      .append(userID)
      .append("\t")
      .append(pageID)
      .append("\t")
      .append(channel)
      .append("\t")
      .append(action)
    System.out.println(userLogBuffer.toString())
    userLogBuffer.toString()
  }
}
```

