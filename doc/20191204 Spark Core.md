---
title: 20191204 Spark  学习
date: 2019-12-04 
---

[TOC]



## Spark 简介

- Apache Spark™ is a fast and general engine for large-scale data processing.
- Apache Spark is an open source cluster computing system that aims to make data analytics fast 
-  both fast to run and fast to wrtie

Spark 是基于内存计算的一个分布式计算框架，使用 DAG 有向无环图优化 Job 任务执行。

Spark 官网地址： https://spark.apache.org/

Spark 版本选择：采用 Spark 2.3 版本，要求使用  JDK 1.8 以上版本， Python3.4 以上版本

### Spark 环境搭建 

这里采用 maven 工程创建 Spark 项目，导入 Spark 相关依赖坐标

### pom.xml   

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>20191204 _Spark_Demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <name>20191204 _Spark_Demo</name>

    <!-- 配置以下可以解决 在jdk1.8环境下打包时报错 “-source 1.5 中不支持 lambda 表达式” -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- Spark-core -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>2.3.1</version>
        </dependency>

        <!-- Scala 包-->
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>2.11.7</version>
        </dependency>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-compiler</artifactId>
            <version>2.11.7</version>
        </dependency>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-reflect</artifactId>
            <version>2.11.7</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>
        <dependency>
            <groupId>com.google.collections</groupId>
            <artifactId>google-collections</artifactId>
            <version>1.0</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>

            <!-- 在maven项目中既有java又有scala代码时配置 maven-scala-plugin 插件打包时可以将两类代码一起打包 -->
            <plugin>
                <groupId>org.scala-tools</groupId>
                <artifactId>maven-scala-plugin</artifactId>
                <version>2.15.2</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <!-- maven 打jar包需要插件 -->
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.4</version>
                <configuration>
                    <!-- 设置false后是去掉 MySpark-1.0-SNAPSHOT-jar-with-dependencies.jar 后的 “-jar-with-dependencies” -->
                    <!--<appendAssemblyId>false</appendAssemblyId>-->
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <mainClass>com.bjsxt.xxx</mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>assembly</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

### 项目目录结构

src-> java     java 目录下书写 Java 语言代码

src->scala    scala 目录下书写 Scala 语言代码

## WordCount

### Scala 版本 wordcount

```scala
package com.szxy.myscalacode

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

/**
 * 单词个数统计
 */
object SparkWordCount {
  def main(args: Array[String]): Unit = {
    val config = new SparkConf()
    // 默认设置
    config.setMaster("local")
    config.setAppName("SpringWordApp")
    // 创建 Spark 上下文对象，加载配置类
    val sc = new SparkContext(config)
    // 读取文件
    val words: RDD[String] = sc.textFile("./data/word.txt")
    //words.foreach(println)
    val wordlist = words.flatMap(one => one.split(" "))
    wordlist.foreach(println)
    val wordMap = wordlist.map(one=>new Tuple2(one,1))
    wordMap.foreach(println)
    val result = wordMap.reduceByKey((v1,v2) =>(v1+v2))
    result.foreach(println)
    // 安装单词个数多少倒序，从大到小
    val sortResult = result.sortBy(one => one._2,false)
    sortResult.foreach(println)
    // 交换 key 和 value
    val resultSwap = result.map(pair => (pair._2, pair._1))
    resultSwap.foreach(println)
    // 排序
    resultSwap.sortByKey(false).foreach(println)

    /********** 简化版写法 **********/
   /*
    val config = new SparkConf()
      .setMaster("local")
      .setAppName("SpringWordApp")
    // 创建 Spark 上下文对象，加载配置类
    new SparkContext(config)
      .textFile("./data/word.txt")
      .flatMap(_.split(" "))
      .map((_,1))
      .reduceByKey((v1,v2)=>(v1+v2))
      .sortBy(_._2,false)
      .foreach(println)
    */
  }
}
```

### Java 版本 wordcount

```java
/**
 * @Auther:zwer
 * @Date:2019/12/4 19:30
 * @Description:com.szxy.myjavacode
 * @Version:1.0
 * Java 版本的 wordCount
 **/
public class JavaWordCount {
    public static void main(String[] args) throws Exception {
        SparkConf conf = new SparkConf();
        conf.setMaster("local");
        conf.setAppName("wc");

        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> file = sc.textFile("./data/word.txt");
        /********* 简介版 ***************/
        JavaRDD<String> words = file.flatMap(line -> Arrays.asList(line.split(" ")).iterator());
        //words.map()
        JavaPairRDD<String, Integer> wordPairs = words.mapToPair( word -> new Tuple2(word, 1));
        JavaPairRDD<String, Integer> result = wordPairs.reduceByKey( (v1, v2) -> v1 + v2);
        result.foreach(tuple2 -> System.out.println(tuple2));

        /**************** 排序 ***********/
        JavaPairRDD<Integer, String> resultSwap = result.mapToPair(tuple2 -> new Tuple2<>(tuple2._2, tuple2._1));
        JavaPairRDD<Integer, String> resultSwapSort = resultSwap.sortByKey(false);
        JavaPairRDD<String, Integer> resultSort = resultSwapSort.mapToPair( t -> new Tuple2<>(t._2, t._1));
        resultSort.foreach( tuple2 -> System.out.println(tuple2));


       /* JavaRDD<String> words = file.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public Iterator<String> call(String line) throws Exception {
                return Arrays.asList(line.split(" ")).iterator();
            }
        });
        //words.map()
        JavaPairRDD<String, Integer> wordPairs = words.mapToPair(new PairFunction<String, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(String word) throws Exception {
                return new Tuple2(word, 1);
            }
        });
        JavaPairRDD<String, Integer> result = wordPairs.reduceByKey(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer v1, Integer v2) throws Exception {
                return v1 + v2;
            }
        });
        result.foreach(new VoidFunction<Tuple2<String, Integer>>() {
            @Override
            public void call(Tuple2<String, Integer> tuple2) throws Exception {
                System.out.println(tuple2);
            }
        });

        *//**************** 排序 ***********//*
        JavaPairRDD<Integer, String> resultSwap = result.mapToPair(new PairFunction<Tuple2<String, Integer>, Integer, String>() {
            @Override
            public Tuple2<Integer, String> call(Tuple2<String, Integer> tuple2) throws Exception {
                return new Tuple2<>(tuple2._2, tuple2._1);
            }
        });
        JavaPairRDD<Integer, String> resultSwapSort = resultSwap.sortByKey(false);
        JavaPairRDD<String, Integer> resultSort = resultSwapSort.mapToPair(new PairFunction<Tuple2<Integer, String>, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(Tuple2<Integer, String> t) throws Exception {
                return new Tuple2<>(t._2, t._1);
            }
        });
        resultSort.foreach(new VoidFunction<Tuple2<String, Integer>>() {
            @Override
            public void call(Tuple2<String, Integer> tuple2) throws Exception {
                System.out.println(tuple2);
            }
        });*/

    }
}

```

## Spark 与 MapReduce 的区别

### 回顾 MR

![](http://img.zwer.xyz/blog/20191009110701.png)

### Spark 与 MR 的区别

Spark 计算中间结果存储在内存中，而 MR的中间结果存储在 HDFS 磁盘中，但不论是 Spark 还是 MR，计算最终结果都会落地磁盘存储。

Spark 是内存计算，并且使用 DAG（有向无环图）优化 Job 任务执行

![](http://img.zwer.xyz/blog/20191204210624.png)



## Spark 核心

### Spark 的运行模式

- **local**

  本地运行模式，多用于测试在 Eclipse、 IDEA 环境下使用

- **standalone**

  Spark 自带的资源调度框架，支持分布式搭建。Spark 任务运行在 Standalone 。集群命令提交

- **yarn**

  Yarn 资源调度框架，支持分布式搭建。 Spark 可以在 Yarn 上。集群命令提交

- **Mesos**

  资源调度框架

### Spark RDD 算子特性

> RDD Resilient Distributed Datest ，弹性分布式数据集

**RDD 五大特性** 

A list of partitions

A function for computing each partition

 A list of dependencies on other RDDs

 Optionally, a Partitioner for key-value RDDs

 shuffle的时候  Optionally, a list of preferred locations to compute each split on

task计算的数据本地化

![](http://img.zwer.xyz/blog/day01-1Spark核心RDD.jpg)

RDD 包括了 partition，而partition 存储了计算逻辑。

将 RDD 按照一定的顺序组成起来，最终发送到数据所在位置然后处理数据。

一个 Block 块可以对应一个或多个 Partition 处理



## <font color='red'>Spark 算子</font>

### Spark 算子种类

1. **transformation 算子**

   转换算子， 特点：**懒执行**

   | transaction  算子                                  | 作用                                                         |
   | -------------------------------------------------- | ------------------------------------------------------------ |
   | flatMap                                            | 将一条数据拆分到多条数据                                     |
   | map                                                | 将一个数据映射到一个 K，V 形式的键值对                       |
   | reduceByKey                                        | 按照 key 进行分组，计算 “相同” key 对应的 value 值           |
   | sortByKey                                          | 按照 key 进行排序                                            |
   | sortBy                                             | 按照指定逻辑排序                                             |
   | filter                                             | 将满足过滤条件（return true）的数据抽取出来                  |
   | join、leftOuterJoin、rightOuterJoin、fullOuterJoin | 作用在K,V格式的RDD上。根据K进行连接，对（K,V）join(K,W)返回（K,(V,W)） |
   | cogroup                                            | 当调用类型据上时，返回一个数据集（K，（Iterable<V>,Iterable<W>）），<font color='red'>子RDD的分区与父RDD多的一致。</font> |
   | union                                              | 合并两个数据集。两个数据集的类型要一致。<br/> <font color='red'>返回新的RDD的分区数是合并RDD分区数的总和。</font> |
   | intersection                                       | join后的分区数与父RDD分区数多的那一个相同。<br/>取两个数据集的交集，<font color='red'>返回新的RDD与父RDD分区多的一致</font> |
   | subtract                                           | 取两个数据集的差集，结果RDD的分区数与subtract前面的RDD的分区数一致。 |
   | mapPartitions                                      | 与map类似，遍历的单位是每个partition上的数据。               |
   | mapPartitionsWithIndex                             | 与mapPartitions 类似，携带分区号                             |
   | distinct                                           | map+reduceByKey+map                                          |
   | repartition                                        | 重新分区, 既可以增加分区，也可以减少分区(产生 shuffle) ，一般用于增多分区。<br/>repartition = coalesce(分区数, shuffle=true) |
   | coalesce                                           | 重新分区, 既可以增加分区，也可以减少分区(默认不产生 shuffle)，一般用于减少分区 |
   | groupByKey                                         | 将相同 key 的 value 汇聚在一起                               |
   | zip                                                | 将两个不同的 RDD 中的元素组成 k,v 格式的数据                 |
   | zipWithIndex                                       | 返回每条数据及其所在的分区号                                 |

2. **action 算子**

   行动算子，用于触发 transformation 算子执行

   一个 Spark Applicatin 中有一个 action 算子对应一个 job 

   | Action 算子      | 作用                                                         |
   | ---------------- | ------------------------------------------------------------ |
   | foreach          | 遍历 Results 结果                                            |
   | count            | 返回数据总条数                                               |
   | countByKey       | 统计（k,v）格式中，相同 key 出现的次数                       |
   | countByValue     | 将 （k，v）作为整体，统计 相同（k,v） 出现的次数<br />也统计非 kv 格式的数据 |
   | collection       | 返回所有数据条件，**慎用**（数据过大，会导致内存溢出）       |
   | take(Num : Int)  | 将前 num 条数据抽取出来                                      |
   | first            | first = take(1)，将第一条数据返回                            |
| reduce           | 统计数据总和                                                 |
   | foreachPartition | 遍历的数据是每个partition的数据                              |

3. **持久化算子**

   | RDD 持久化算子 | 作用                                                         |
   | -------------- | ------------------------------------------------------------ |
   | cache          | 默认数据缓存在磁盘                                           |
   | persist        | 可手动指定数据持久化级别                                     |
   | checkpoint     | 将数据直接持久化到外部的存储系统。当application执行完成之后，数据目录不会被清除 |

区分  transformation算子 与 action 算子：

transformation 算子转换前后都是 RDD 类型

action 算子转换后是非 RDD 类型

### Spark 代码执行流程

1. 创建 SparkConf 对象  

   ```scala
   SparkConf conf = new SparkConf()
   ```

2. 创建 SparkContext 上下文对象 

   ```scala
   SparkContext sc = new SparkContext(conf)
   ```

3. 获取 RDD Spark 上下文对象是 Spark 执行的入口

   ```scala
   var rdd = sc.textFile(...)
   ```

4. 对 RDD 使用 transformaction 类算子进行数据转换

5. 对 RDD 使用 Action 类算子触发执行

6. 停止  `sc.stop()`


注意： Spark 中采用 Spark Application 作为整个代码的流程，其中 Spark Application 与 Job 关系是一个 Spark Application 可以包括一个或多个 Job

### Spark 算子使用

使用 Java 语言测试

```java
public class SparkDay01 {
    public static void main(String[] args) {

        SparkConf conf = new SparkConf();
        conf.setMaster("local");
        conf.setAppName("SparkDay01-Hello");

        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> words = sc.textFile("./data/word.txt");

        // filter
//        words.filter(new Function<String, Boolean>() {
//            @Override
//            public Boolean call(String word) throws Exception {
//                return word.contains("Scala");
//            }
//        }).foreach(new VoidFunction<String>() {
//            @Override
//            public void call(String s) throws Exception {
//                System.out.println(s);
//            }
//        });

        // sample
//        words.sample(true,0.1,100L).foreach(new VoidFunction<String>() {
//            @Override
//            public void call(String s) throws Exception {
//                System.out.println(s);
//            }
//        });

        // collect
//        List<String> wordList = words.collect();
//        System.out.println("wordList's length:"+wordList.size());
//        for (String s : wordList) {
//            System.out.println(s);
//        }

        // take
//        List<String> wordsTakeList = words.take(10);
//        for (String s : wordsTakeList) {
//            System.out.println(s);
//        }

        // first
//        String word = words.first();
//        System.out.println(word);

        JavaRDD<Integer> rdd = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
        Integer num = rdd.reduce(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer v1, Integer v2) throws Exception {
                return v1 + v2;
            }
        });
        System.out.println("num="+num);


        sc.stop();

    }
}
```

###  Spark 中 transformaction 、action 算子

- Java 版本

```java
package com.szxy.myjavacode;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.Optional;
import org.apache.spark.api.java.function.*;
import org.omg.CORBA.Any;
import scala.Tuple2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;

/**
 * @Auther:zwer
 * @Date:2019/12/6 16:34
 * @Description:com.szxy.myjavacode
 * @Version:1.0
 **/
public class SparkDay02 {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local"); //运行模式
        conf.setAppName("test_sparktest02");
        JavaSparkContext sc = new JavaSparkContext(conf);

        /*********************  foreachPartition     *************************************/
//        JavaRDD<String> strRdd = sc.parallelize(Arrays.asList("a", "b", "c", "d"), 3);
//        strRdd.foreachPartition(new VoidFunction<Iterator<String>>() {
//            @Override
//            public void call(Iterator<String> iter) throws Exception {
//                System.out.println("获取数据库连接 ...");
//                ArrayList<String> strList = new ArrayList<>();
//                while(iter.hasNext()){
//                    String str = iter.next();
//                    System.out.println("保存数据到数据库 ..." + str);
//                    strList.add(str);
//                }
//                System.out.println("关闭数据库连接 ...");
//            }
//        });

        /*********************  mapPartitions     *************************************/

//        strRdd.mapPartitions(new FlatMapFunction<Iterator<String>, String>() {
//            @Override
//            public Iterator<String> call(Iterator<String> iter) throws Exception {
//                System.out.println("获取数据库连接 ...");
//                ArrayList<String> strList = new ArrayList<>();
//                while(iter.hasNext()){
//                    String str = iter.next();
//                    System.out.println("保存数据到数据库 ..." + str);
//                    strList.add(str);
//                }
//                System.out.println("关闭数据库连接 ...");
//                return strList.iterator();
//            }
//        }).count();

//        strRdd.map(str -> {
//            System.out.println("获取数据库连接 ...");
//            System.out.println("保存数据到数据库 ..." + str);
//            System.out.println("关闭数据库连接 ...");
//            return  str;
//        }).count();
        /*********************  distinct    *************************************/
//        JavaRDD<Integer> num1RDD = sc.parallelize(Arrays.asList(1,1,1,3,4,5,5,7,7), 3);
//        JavaRDD<Integer> nums = num1RDD.distinct();
//        nums.foreach(one-> System.out.println(one));

        // 去重
//        JavaPairRDD<Integer, Integer> numPairs = num1RDD.mapToPair(new PairFunction<Integer, Integer, Integer>() {
//            @Override
//            public Tuple2<Integer, Integer> call(Integer integer) throws Exception {
//                return new Tuple2<>(integer, 1);
//            }
//        });
//        JavaPairRDD<Integer, Integer> numReducePairs = numPairs.reduceByKey(new Function2<Integer, Integer, Integer>() {
//            @Override
//            public Integer call(Integer v1, Integer v2) throws Exception {
//                return v1 + v2;
//            }
//        });
//        JavaRDD<Integer> nums = numReducePairs.map(new Function<Tuple2<Integer, Integer>, Integer>() {
//            @Override
//            public Integer call(Tuple2<Integer, Integer> v1) throws Exception {
//                return v1._1;
//            }
//        });
//        nums.foreach(one-> System.out.println(one));


        /*********************  intersect  subtract   *************************************/
//        JavaRDD<Integer> num1RDD = sc.parallelize(Arrays.asList(1, 2, 3, 4), 3);
//        JavaRDD<Integer> num2RDD = sc.parallelize(Arrays.asList(3, 4, 5, 6, 7, 8), 4);
//        JavaRDD<Integer> integerJavaRDD = num1RDD.intersection(num2RDD);
////        integerJavaRDD.foreach(one -> System.out.println(one));
//
//        JavaRDD<Integer> subtract = num1RDD.subtract(num2RDD);
//        subtract.foreach(one -> System.out.println(one));
//
        /*********************  union   *****************************************/
//        JavaRDD<Integer> num1RDD = sc.parallelize(Arrays.asList(1, 2, 3, 4), 3);
//        JavaRDD<Integer> num2RDD = sc.parallelize(Arrays.asList(5, 6, 7, 8), 4);
//        JavaRDD<Integer> unionResult = num1RDD.union(num2RDD);
//        unionResult.foreach(one-> System.out.println(one));
//        System.out.println("num1RDD numPartitions is "+num1RDD.getNumPartitions());
//        System.out.println("num2RDD numPartitions is "+num2RDD.getNumPartitions());
//        System.out.println("unionResult numPartitions is "+unionResult.getNumPartitions()); // 7

        /********************* join rightOuterJoin, leftOuterJoin, fullOuterJoin, cgroup  *****************************************/
//        JavaPairRDD<String, Integer> nameRDD = sc.parallelizePairs(Arrays.asList(
//                new Tuple2<String, Integer>("zhangsan", 20),
//                new Tuple2<String, Integer>("zhangsan", 21),
//                new Tuple2<String, Integer>("lisi", 20),
//                new Tuple2<String, Integer>("lisi", 20),
//                new Tuple2<String, Integer>("wangwu", 20),
//                new Tuple2<String, Integer>("zhaoliu", 70)
//        ));
//
//        JavaPairRDD<String, Integer> scoreRDD = sc.parallelizePairs(Arrays.asList(
//                new Tuple2<String, Integer>("zhangsan", 80),
//                new Tuple2<String, Integer>("zhangsan", 81),
//                new Tuple2<String, Integer>("lisi", 60),
//                new Tuple2<String, Integer>("lisi", 61),
//                new Tuple2<String, Integer>("wangwu", 100),
//                new Tuple2<String, Integer>("tianqi", 70)
//        ));
//        // cogroup
//        JavaPairRDD<String, Tuple2<Iterable<Integer>, Iterable<Integer>>> result = nameRDD.cogroup(scoreRDD);
//        result.foreach(one-> System.out.println(one));
        // 结果
//        (zhangsan,([20, 21],[80, 81]))
//        (wangwu,([20],[100]))
//        (zhaoliu,([70],[]))
//        (tianqi,([],[70]))
//        (lisi,([20, 20],[60, 61]))


        // fullOuterJoin
//        JavaPairRDD<String, Tuple2<Optional<Integer>, Optional<Integer>>> result = nameRDD.fullOuterJoin(scoreRDD);
//        result.foreach(one-> System.out.println(one));

//        System.out.println("nameRDD partitions is "+nameRDD.getNumPartitions()+
//                        ", scoreRDD partitions is "+scoreRDD.getNumPartitions());
//        System.out.println("result partitions is "+nameRDD.join(scoreRDD).getNumPartitions());
        // 结果， join 的结果：分区数按照原来 RDD 分区数多的算
//        nameRDD partitions is 3, scoreRDD partitions is 4
//        result partitions is 4

        //rightOutJoin
//        JavaPairRDD<String, Tuple2<Optional<Integer>, Integer>> result = nameRDD.rightOuterJoin(scoreRDD);
//        result.foreach(new VoidFunction<Tuple2<String, Tuple2<Optional<Integer>, Integer>>>() {
//            @Override
//            public void call(Tuple2<String, Tuple2<Optional<Integer>, Integer>> tuple) throws Exception {
////                System.out.println(stringTuple2Tuple2);
//                String key = tuple._1;
//                Tuple2<Optional<Integer>, Integer> value = tuple._2;
//
//                if(value._1.isPresent()){
//                    System.out.println("key is "+key+", value is ("+value._1.get()+","+value._2+")");
//                }else{
//                    System.out.println("key is "+key+", value is (null,"+value._2+")");
//                }
//            }
//        });
//        key is zhangsan, value is (20,80)
//        key is wangwu, value is (20,100)
//        key is tianqi, value is (null,70)
//        key is lisi, value is (20,60)


        // leftOutJoin
//        JavaPairRDD<String, Tuple2<Integer, Optional<Integer>>> result = nameRDD.leftOuterJoin(scoreRDD);
//        result.foreach(new VoidFunction<Tuple2<String, Tuple2<Integer, Optional<Integer>>>>() {
//            @Override
//            public void call(Tuple2<String, Tuple2<Integer, Optional<Integer>>> stringTuple2Tuple2) throws Exception {
//                System.out.println(stringTuple2Tuple2);
//            }
//        });
//        (zhangsan,(20,Optional[80]))
//        (wangwu,(20,Optional[100]))
//        (lisi,(20,Optional[60]))


        // join
//        JavaPairRDD<String, Tuple2<Integer, Integer>> resultJoinRDD = nameRDD.join(scoreRDD);
//        resultJoinRDD.foreach(new VoidFunction<Tuple2<String, Tuple2<Integer, Integer>>>() {
//            @Override
//            public void call(Tuple2<String, Tuple2<Integer, Integer>> stringTuple2Tuple2) throws Exception {
//                System.out.println(stringTuple2Tuple2);
//            }
//        });
//        (zhangsan,(20,80))
//        (wangwu,(20,100))
//        (lisi,(20,60))



        sc.stop(); //停止
    }
}

```

- scala 版本

  测试 Spark 算子 1

  ```scala
  package com.szxy.myscalacode
  
  import java.util.Arrays
  
  import org.apache.spark.rdd.RDD
  import org.apache.spark.{SparkConf, SparkContext}
  
  import scala.collection.mutable.ListBuffer
  
  object SparkDay02 {
    def main(args: Array[String]): Unit = {
      val conf = new SparkConf()
      conf.setMaster("local")
      conf.setAppName("spark02")
      val sc: SparkContext = new SparkContext(conf)
  
  
      val num1RDD: RDD[Int] = sc.makeRDD(Array[Int](1, 2, 3, 4, 5),3)
  
      //  mapPartitions
  //    num1RDD.mapPartitions(iter => {
  //      var strList = ListBuffer[String]();
  //      println("建立数据库连接 ...")
  //      while(iter.hasNext){
  //        println("保存数据到数据库中..."+iter.next())
  //        strList.apply(iter.next())
  //      }
  //      println("关闭数据库连接...")
  //      strList.iterator
  //    }).count()
  
      // foreachPartition
      
      num1RDD.foreachPartition(iter=>{
        println("建立数据库连接 ...")
        while(iter.hasNext){
          println("保存数据到数据库中..."+iter.next())
        }
        println("关闭数据库连接...")
      })
  
  //    val num1RDD: RDD[Int] = sc.makeRDD(Array[Int](1, 2, 3, 4, 5))
  //    val num2RDD: RDD[Int] = sc.makeRDD(Array[Int](2, 3, 4, 5))
  //    val num3RDD: RDD[Int] = sc.makeRDD(Array[Int](1, 1, 2, 3, 2,5,6,6))
  //    num1RDD.intersection(num2RDD).foreach(println)
  //    num1RDD.subtract(num2RDD).foreach(println)
  //    num3RDD.distinct().foreach(println)
  
  //    val nameRDD: RDD[(String, Int)] = sc.parallelize(Array[(String,Int)](
  //      ("lisi", 20) ,("zhangsan", 21), ("wangwu", 20), ("zhaoliu", 70)
  //    ));
  //
  //    val scoreRDD: RDD[(String, Int)] = sc.parallelize(Array[(String,Int)](
  //      ("lisi", 50) ,("zhangsan", 81), ("wangwu", 10), ("tianqi", 70)
  //    ));
  //
  //    nameRDD.cogroup(scoreRDD).foreach(println)
  
      // join
  //    val join: RDD[(String, (Int, Int))] = nameRDD.join(scoreRDD)
  //    join.foreach(println)
  //
  //    println("------------------------------------------------------")
  //
  //    val leftjoin: RDD[(String, (Int, Option[Int]))] = nameRDD.leftOuterJoin(scoreRDD)
  //    leftjoin.foreach(println)
  //
  //    println("------------------------------------------------------")
  //
  //    val rightjoin: RDD[(String, (Option[Int], Int))] = nameRDD.rightOuterJoin(scoreRDD)
  //    rightjoin.foreach(println)
  //
  //    println("------------------------------------------------------")
  //
  //    val fullOuterJoin = nameRDD.fullOuterJoin(scoreRDD)
  //    fullOuterJoin.foreach(println)
  
  
    }
  }
  
  ```

  测试 Spark 算子2：

  使用 mapPartitionsWithIndex 、repartition、 coalesce 、 groupByKey、 zip、 zipWithIndex、countByKey、countByValue

  ```scala
  object SparkDay03 {
    def main(args: Array[String]): Unit = {
      val conf: SparkConf = new SparkConf()
      conf.setMaster("local")
      conf.setAppName("spark03")
      val sc: SparkContext = new SparkContext(conf)
  
      /***
       *  countByKey
       */
  //    val rdd1: RDD[(String, Int)] = sc.parallelize(Array[(String, Int)](
  //          ("zhangsan", 1), ("zhangsan", 1), ("lisi", 1), ("lisi", 2), ("zhaoliu", 3), ("zhaoliu", 4)
  //    ))
  //    val result: collection.Map[String, Long] = rdd1.countByKey()
  //    result.foreach(println)
  
      /**
       *   countByValue 将 （k,v） 作为 value ，统计 value 重复的个数
       */
  //    val rdd1: RDD[(String, Int)] = sc.parallelize(Array[(String, Int)](
  //          ("zhangsan", 1), ("zhangsan", 1), ("lisi", 1), ("lisi", 2), ("zhaoliu", 3), ("zhaoliu", 4)
  //    ))
  //    val result: collection.Map[(String, Int), Long] = rdd1.countByValue()
  //    result.foreach(println)
  
  
      /***
       *    zip  将两个不同的 RDD 中的元素组成 k,v 格式的数据
       *    zipWidthIndex  返回每条数据及其所在的分区号
       */
  //    val data1: RDD[Int] = sc.parallelize(Array[Int](1, 2, 3))
  //    val data2: RDD[String] = sc.parallelize(Array[String]("a", "b", "c"))
  //    data1.zip(data2).foreach(println)
  //    val rdd: RDD[(Int, Long)] = data1.zipWithIndex()
  //    rdd.foreach(println)
  
      /**
       *  groupByKey
       *  会产生 shuffle
       */
  //    val rdd1: RDD[(String, Int)] = sc.parallelize(Array[(String, Int)](
  //      ("zhangsan", 1), ("zhangsan", 1), ("lisi", 1), ("lisi", 2), ("zhaoliu", 3), ("zhaoliu", 4)
  //    ))
  //    rdd1.groupByKey().foreach(println)
  
  //    val rdd1: RDD[String] = sc.parallelize(Array[String](
  //      "HelloWorld1", "HelloWorld2", "HelloWorld3",
  //      "HelloWorld4", "HelloWorld5", "HelloWorld6",
  //      "HelloWorld7", "HelloWorld8", "HelloWorld9",
  //      "HelloWorld10", "HelloWorld11", "HelloWorld12"
  //    ),4)
      /**
       *  mapPartitionsWithIndex
       *  与 mapPartitions 类似，遍历每个分区中的数据，并携带分区号 index
       */
      // index 表示分区号，从 0 开始
      // iter  表示迭代器
  //    val rdd2: RDD[String] = rdd1.mapPartitionsWithIndex((index, iter) => {
  //      var list = new ListBuffer[String];
  //      while (iter.hasNext) {
  //        val value: String = iter.next()
  //        list.append(s"[partition index is $index, value is $value]")
  //      }
  //      list.iterator
  //    })
  //    val result: Array[String] = rdd2.collect()
  //    result.foreach(println)
  
      /**
       *  repartition 重新分区, 既可以增加分区，也可以减少分区(产生 shuffle) ，一般用于增多分区
       *  coalesce  重新分区, 既可以增加分区，也可以减少分区(默认不产生 shuffle)，一般用于减少分区
       */
  //    val rdd3: RDD[String] = rdd2.repartition(5)
  //    val rdd4: RDD[String] = rdd2.coalesce(5, true)
  //    rdd4.mapPartitionsWithIndex((index, iter) => {
  //      var list = new ListBuffer[String];
  //      while (iter.hasNext) {
  //        val value: String = iter.next()
  //        list.append(s"[partition index is $index, value is $value]")
  //      }
  //      list.iterator
  //    }).collect().foreach(println)
  
    }
  }
  
  ```

  

## Spark RDD 持久化

> RDD 持久化的单位是 partition
>

### cache

默认缓存数据在内存中， 相当于 `cache() =  persist() = persit(StorageLevel.MEMORY_ONLY)`

### persit

可手动指定持久化级别 

![](http://img.zwer.xyz/blog/20191205151008.png)

**推荐使用的  StorageLevel 级别** 

- MEMORY_ONLY 
- MEMORY_ONLY_SER
- MEMORY_AND_DISK_SER
- MEMORY_AND_DISK

**避免使用的 StorageLevel  级别**

- 不要使用 “_2” 结束的和 “DISK_ONLY”  级别

**测试代码**

```scala

object CacheSparkRDD {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setMaster("local")
    conf.setAppName("cacheSparkRDD")

    val sc = new SparkContext(conf)
    val words = sc.textFile("./data/persistData.txt")

    // 缓存数据
    //words.cache()
    words.persist(StorageLevel.MEMORY_AND_DISK_SER)

    val startTime1 = System.currentTimeMillis()
    val num1 = words.count()
    val endTime1= System.currentTimeMillis()
    println("totoal is "+num1,"time is "+(endTime1-startTime1))

    val startTime2 = System.currentTimeMillis()
    val num2 = words.count()
    val endTime2= System.currentTimeMillis()
    println("totoal is "+num2,"time is "+(endTime2-startTime2))

    sc.stop()
  }
}
```

###  总结

1. cache 和 persit 都是懒执行，需要 action 算子触发
2. 对RDD使用cache() 或者persist() 时，可以将cache()和persist之后的结果赋值给一个变量，下个job使用这个变量就是使用持久化的数据
3. 如果使用②这种方式复制变量，赋值之后不能紧跟action算子
4. cache() 和persist() 持久化的数据当application运行完成之后会自动清除

###  checkPoint 

checkpoint 将数据直接持久化到外部的存储系统。当application执行完成之后，数据目录不会被清除

当RDD的计算逻辑非常复杂时，可以对RDD进行持久化。不要对大量的RDD进行持久化。多数情况下，我们使用checkpoint保存原数据。

优化：对哪个RDD进行持久化，可以先对这个RDD进行cache(),这样可以节省一次计算过程

checkpoint的持久化单位也是partition，checkpoint算子也是懒执行的

**checkPoint 流程**

1. 当RDD job完成之后，Spark会从后往前回溯，找到checkpoint的RDD进行标记

2. 回溯完成之后，Spark会启动一个job，对持久化的RDD进行结果计算，将计算的结果持久化到指定的目录中

**测试代码**

```scala
object CacheSparkRDD {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setMaster("local")
    conf.setAppName("cacheSparkRDD")

    val sc = new SparkContext(conf)
    sc.setCheckpointDir("./data/ck")
    val words = sc.textFile("./data/word.txt")
    words.checkpoint()
    words.count()
  }
}
```

生成的目录和文件

![](http://img.zwer.xyz/blog/20191205154015.png)





## Spark 集群搭建

### Spark 中节点角色

Driver 节点负责发送 tasks 任务到各个 Worker 节点上和接收 Worker 节点上处理数据返回的 Results 结果

Master 是资源管理的主节点， Worker 是资源管理的从节点

若 Driver 节点接收到 Results 结果过大，可能导致 OOM 内存溢出的风险

![](http://img.zwer.xyz/blog/20191205092701.png)



Master(standalone)：资源管理的主节点(进程) • Cluster Manager：在集群上获取资源的外部服务(例如standalone,Mesos,Yarn )
Worker Node(standalone)：资源管理的从节点(进程) 或者说管理本机资源的进程

 Application：基于Spark的⽤用户程序，包含了driver程序和运行在集群上的executor程序

 Driver Program：用来连接工作进程（Worker）的程序

Executor：是在一个worker进程所管理的节点上为某Application启动的⼀一个进程，该进
程负责运行任务，并且负责将数据存在内存或者磁盘上。每个应⽤用都有各自独⽴立的
executors

 Task：被送到某个executor上的工作单元

 Job：包含很多任务(Task)的并行计算，可以看做和action对应

 Stage：⼀个Job会被拆分很多组任务，每组任务被称为Stage(就像Mapreduce分map task
和reduce task一样)

### 更换 JDK 版本

由 JDK 1.7 升级到 JDK 1.8 版本

```shell
# 上传 JDK 1.8 解压安装，移动到指定位置，分发到其他节点
# 修改 vi /etc/profile 文件， 变更  JAVA_HOME 目录

# 替换软链接
# 默认使用解压的jdk安装jdk8,相对于rpm安装来说 不会覆盖默认/usr/bin/java 指向的位置。需要手动改动
# 指向的位置，不然会默认还是执行的旧的jdk1.7
ln -sf /usr/local/jdk/jdk1.8.0_181/bin/java  /usr/bin/java

# 修改 Hadoop 配置文件中 JAVA_HOME 目录
vi yarn-env.sh 
vi hadoop-env.sh 
vi mapred-env.sh 
```

### 安装  Spark

**Spark 角色安排**

| 角色/节点              | Master | Worker |
| ---------------------- | ------ | ------ |
| node01/192.168.170.101 | *      |        |
| node02/192.168.170.102 |        | *      |
| node03/192.168.170.103 |        | *      |

**安装步骤**

```shell
# 解压 Spark 压缩包并移动指定的路径下，然后重命名 
tar -zxvf spark-2.3.1-bin-hadoop2.6.tgz  -C /opt/sxt
cd /opt/sxt
mv spark-2.3.1-bin-hadoop2.6/ spark-2.3.1

# 进入 spark 配置目录
cd /spark-2.3.1/conf 

cp slaves.template slaves
vi slaves
# 以下是 salves 文件添加内容
node02
node03
# -------文件结束
cp spark-env.sh.template spark-env.sh
vi spark-env.sh
# 以下是 spark-env.sh 文件的内容
export SPARK_MASTER_HOST=node01
export SPARK_MASTER_PORT=7077
export SPARK_WORKER_CORES=2
export SPARK_WORKER_MEMORY=2g
# ---- 文件结束

# 分发到 node02,node03 节点上

# 在 node01 节点上启动 Spark 集群  
# 进入 Spark 解压缩目录下 sbin 
./start-all.sh
```

**安装 Spark 客户端**

```shell
# 分发 Spark 解压包到 node04 节点即可 (注意： JDK 版本需要是 1.8 以上)

# 运行 Spark 任务
./spark-submit --master spark://node01:7077 --class org.apache.spark.examples.SparkPi  ../examples/jars/spark-examples_2.11-2.3.1.jar  100
```

## Spark Master HA 高可用

### 前提
已经搭建好 Spark 集群——我这里有 一台 master 节点和 二台 worker 节点

### 准备

这里选择 node01 的 master 作为主节点， node02 的 master 作为备用节点，使用 ZK 集群作为分布式协调

|        | master | master-standby | worker | ZK   |
| ------ | ------ | -------------- | ------ | ---- |
| node01 | *      |                |        |      |
| node02 |        | *              | *      | *    |
| node03 |        |                | *      | *    |
| node04 |        |                |        | *    |

###  具体操作

```shell
1. 选择 node01 上的master节点的 spark
2. 修改配置文件 spark-env.sh 文件
export SPARK_DAEMON_JAVA_OPTS="
    -Dspark.deploy.recoveryMode=ZOOKEEPER 
    -Dspark.deploy.zookeeper.url=node02:2181,node03:2181,node04:2181, 
    -Dspark.deploy.zookeeper.dir=/spark-master"

3. 分开给其他 worker 节点

4. 修改 node02 上 spark 的配置文件  spark-env.sh 
	export SPARK_MASTER_HOST=node02
	
5. 在 node01 上启动 spark 集群
	./$SPARK_HOME/sbin/start-all.sh 
	
6. 在 node02 上单独启动 master 备用节点
	./$SPARK_HOME/sbin/start-master.sh
```
### Master 三种状态
	alive
	recovering
	standy



## Spark 任务提交模式

### Spark standalone 模式

- **基于 Standalone client 模式提交任务（程序测试）**

  1.  Worker 节点向 Master 节点汇报自身资源情况
  2. 客户端节点提交 Spark 任务时启动  Driver 进程，然后客户端节点向 Master 申请资源，Master 找到满足资源的 Worker 节点返回给 Driver 
  3. Driver 发送 task 任务，监控 task 执行，回收结果

  ![](http://img.zwer.xyz/blog/图day02-1 Spark 基于Standalone-client模式提交任务.jpg)

  该模式适用于程序测试，不适用于生产环境。 若客户端提交多个任务，Driver 进程发送 task ，监控 task 任务，回收结果，会占用 大量的网络带宽。

  ```shell
  # deploy-mode 默认采用 client 
  ./spark-submit 
  --master spark://node01:7077
  --deploy-mode client  
  --class 全限定路径名  jar包位置
  ```

  

- **基于 standalone cluster 模式提交任务（生产环境）**

  1. Worker 节点向 Master 节点汇报自身资源情况
  2. 客户端节点向 Master 申请启动 Driver
  3. Master 会在集群中随机选取一台满足资源条件的 Worker 节点启动 Driver 
  4. 被选取的 Worker 节点启动自身 Driver JVM 进程，向 Master 申请资源启动 Spark Application 
  5. Driver JVM 进程会向Worker 节点 发送 task 任务，监控 task 执行，回收结果

![](http://img.zwer.xyz/blog/图day02-2 Spark基于Standalone-cluster模式提交任务.jpg)

该模式适用于生产环境，将网络带宽压力分撒到不同的 Worker 节点上。

```
./spark-submit 
--master spark://node01:7077
--deploy-mode cluster  
--class 全限定路径名  jar包位置
```

测试集群提交

```
./spark-submit --master spark://node01:7077 --deploy-mode cluster --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 1000
```

注意： 以 standalone-cluster 方式提交任务，在客户端 client 不会显示 task 任务执行的具体情况

### Haddop YARN 模式

修改 hadoop 配置，关闭虚拟内存检查

```shell
cd $HADOOP_HOME/etc/haddop
vi yarn-site.xml 
# 添加以下内容 
<property>  
    <name>yarn.nodemanager.vmem-check-enabled</name>  
    <value>false</value>  
</property>
# --------------------
# 注意要将  yarn-site.xml 分发给其他 hadoop 集群中的节点
```

修改 Spark 客户端配置文件，指定 Hadoop 配置文件所在位置

```shell
# 进入 spark 客户端根目录中，即 Spark 解压包中根目录
cd conf
cp spark-env.sh.template spark-env.sh
vi spark-env.sh
# 添加以下内容
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
# --------------------
```

- **基于 Hadoop-yarn-client 模式提交任务**

  1.客户端提交一个 Application，在客户端启动一个Driver进程。
  2.应用程序启动后会向RS(ResourceManager)发送请求，启动AM(ApplicationMaster)的资源。
  3.RS 收到请求，随机选择一台 NM(NodeManager)启动AM。这里的NM相当于Standalone中的Worker节点。
  4.AM启动后，会向RS请求一批container资源，用于启动Executor.
  5.RS会找到一批NM返回给AM,用于启动Executor。
  6.AM会向NM发送命令启动Executor。
  7.Executor启动后，会反向注册给Driver，Driver发送task到Executor,执行情况和结果返回给Driver端。

  ![](http://img.zwer.xyz/blog/图day02-3 Spark基于Yarn-client模式提交任务.jpg)

  总结：Yarn-client模式同样是适用于测试，因为Driver运行在本地，Driver会与yarn集群中的Executor进行大量的通信，会造成客户机网卡流量的大量增加.

  **ApplicationMaster 的作用：**

  1. 为当前的Application申请资源

  2. 给NameNode发送消息启动Executor。

     注意：ApplicationMaster有launchExecutor和申请资源的功能，并没有作业调度的功能。

- **基于 Hadoop-yarn-cluster 模式提交任务**

  1.客户机提交Application应用程序，发送请求到RS(ResourceManager),请求启动AM(ApplicationMaster)。
  2.RS收到请求后随机在一台NM(NodeManager)上启动AM（相当于Driver端）。
  3.AM启动，AM发送请求到RS，请求一批container用于启动Executor。
  4.RS返回一批NM节点给AM。
  5.AM连接到NM,发送请求到NM启动Executor。
  6.Executor反向注册到AM所在的节点的Driver。Driver发送task到Executor。
  
  ![](http://img.zwer.xyz/blog/图day2-4 Spark基于Yarn-cluster模式提交任务.jpg)
  
  **总结**
  Yarn-Cluster主要用于生产环境中，因为Driver运行在Yarn集群中某一台nodeManager中，每次提交任务的Driver所在的机器都是随机的，不会产生某一台机器网卡流量激增的现象，缺点是任务提交后不能看到日志。只能通过yarn查看日志。
  **ApplicationMaster的作用：**
  
  1. 为当前的Application申请资源
  
  2. 给NameNode发送消息启动Excutor。
  
  3. 任务调度。
  
     停止集群任务命令：yarn application -kill applicationID
  
## Spark 术语

![](http://img.zwer.xyz/blog/20191208121242.png)



## Spark 宽窄依赖*

窄依赖 ：

 父 RDD 中 partitions 与 子 RDD 中 parititions 是一对一或多对一的关系

宽依赖 （shuffle） ：

 父 RDD 中 partitions 与 子 RDD 中 partitions 是一对多关系

![](http://img.zwer.xyz/blog/20191206090511.png)

注意： 

针对 join width inputs co—partitioned 情形，之前是已经进行过 groupByKey ，也就是 shuffle 的过程，即相同的 key 都是在同一个分区中。当然一个分区也可以存在多个不同的 key ，这并不矛盾。

stash 是由一组并行的 task 组成

![](http://img.zwer.xyz/blog/图day03-1 RDD宽窄依赖.jpg)

 ## Spark 中  stage*

Spark 任务会根据 RDD 之间的依赖关系，形成一个DAG有向无环图，DAG会提交给 DAGScheduler，DAGScheduler 会把 DAG 划分相互依赖的多个 stage，划分 stage 的依据就是 RDD 之间的宽窄依赖。遇到宽依赖就划分stage,每个stage包含一个或多个task任务。然后将这些task以taskSet的形式提交给TaskScheduler运行。

<font color='red' size='4px'> stage是由一组并行的task组成。</font>

 stage切割规则

**切割规则：从后往前，遇到宽依赖就切割stage。** 

![](http://img.zwer.xyz/blog/20191207131515.png)



## Spark 计算模式*

**采用管道 pipeline 计算模式**

高阶函数展开形式： f3(f2(f1(textFile(...))))

1. stage 的并行度由谁决定？ 

   stage 并行度由 stage 中 finalRDD 的 partition 个数决定。 stage 其他的 RDD Partition 个数可以影响 stage 的并行度

2. 管道中的数据何时落地？

   1. shuffle write 落地  2.对 RDD 进行持久化

3. 如何提高 Stage 的并行度？

   reduceByKey(fun, num) ,join(RDD , num)

![](http://img.zwer.xyz/blog/图day03-2 Spark Stage 计算模式.jpg)

**测试Spark 计算模式为 pipeline 代码**

```scala
object TestPipeline {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf()
    conf.setMaster("local")
    conf.setAppName("app")
    val sc: SparkContext = new SparkContext(conf)
    // 验证 Spark 计算模型是管道模式
    val rdd1: RDD[String] = sc.textFile("./data/word.txt")
    val rdd2: RDD[String] = rdd1.filter(one => {
      println("*** filter ****")
      true
    })
    val rdd3: RDD[(String, String)] = rdd2.map((_, "#"))
    rdd3.foreach(println)
    /********** 截取部分运行结果 **************/
//    *** filter ****
//    (hello World1,#)
//    *** filter ****
//    (hello Scala2,#)
  }
}
```

## Spark 任务调度和资源调度

### Spark 任务调度和资源调度流程

启动集群后，Worker节点会向Master节点汇报资源情况，Master掌握了集群资源情况。

当Spark提交一个Application后，根据RDD之间的依赖关系将Application形成一个DAG有向无环图。任务提交后，Spark会在Driver端创建两个对象：DAGScheduler和TaskScheduler，DAGScheduler是任务调度的高层调度器，是一个对象。DAGScheduler的主要作用就是将DAG根 据RDD之间的宽窄依赖关系划分为一个个的Stage，然后将这些Stage以TaskSet的形式提交给TaskScheduler（TaskScheduler是任务调度的低层调度器，这里TaskSet其实就是一个集合，里面封装的就是一个个的task任务,也就是stage中的并行度task任务），TaskSchedule会遍历TaskSet集合，拿到每个task后会将task发送到计算节点Executor中去执行（其实就是发送到Executor中的线程池ThreadPool去执行）。

task在Executor线程池中的运行情况会向TaskScheduler反馈，当task执行失败时，则由TaskScheduler负责重试，将task重新发送给Executor去执行，默认重试3次。如果重试3次依然失败，那么这个task所在的stage就失败了。stage失败了则由DAGScheduler来负责重试，重新发送TaskSet到TaskSchdeuler，Stage默认重试4次。如果重试4次以后依然失败，那么这个job就失败了。job失败了，Application就失败了。

TaskScheduler不仅能重试失败的task,还会重试straggling（落后，缓慢）task（也就是执行速度比其他task慢太多的task）。如果有运行缓慢的task那么TaskScheduler会启动一个新的task来与这个运行缓慢的task执行相同的处理逻辑。两个task哪个先执行完，就以哪个task的执行结果为准。这就是**Spark的推测执行机制**。在Spark中推测执行默认是关闭的。推测执行可以通过spark.speculation属性来配置。

注意：

- 对于 ETL 类型要入数据库的业务要关闭推测执行机制，这样就不会有重复的数据入库。
- 如果遇到数据倾斜的情况，开启推测执行则有可能导致一直会有task重新启动处理相同的逻辑，任务可能一直¬处理不完的状态。

![](http://img.zwer.xyz/blog/20191207200133.png)

### 图解 Spark 的资源调度和任务调度

下面以 Spark 运行模式中 standalone client 模式为例



![](http://img.zwer.xyz/blog/20191208201724.png)



## Spark 提交任务参数

Spark 中资源指的是 core 和 memory

### Options:

- --master
   MASTER_URL, 可以是spark://host:port, mesos://host:port, yarn,  yarn-cluster,yarn-client, local
- --deploy-mode
  DEPLOY_MODE, Driver程序运行的地方，client或者cluster,默认是client。
- --class
  CLASS_NAME, 主类名称，含包名
- --jars
  逗号分隔的本地JARS, Driver和executor依赖的第三方jar包
- --files
  用逗号隔开的文件列表,会放置在每个executor工作目录中
- --conf
  spark的配置属性
- **--driver-memory**
  Driver程序使用内存大小（例如：1000M，5G），默认1024M
- **--executor-memory**
  每个executor内存大小（如：1000M，2G），默认1G

### Spark standalone with cluster deploy mode only:

- **--driver-cores**
  Driver程序的使用core个数（默认为1），仅限于Spark standalone模式
  Spark standalone or Mesos with cluster deploy mode only:
- --supervise
  失败后是否重启Driver，仅限于Spark  alone或者Mesos模式
  Spark standalone and Mesos only:
- --total-executor-cores
  executor使用的总核数，仅限于SparkStandalone、Spark on Mesos模式

### Spark standalone and YARN only:

- **--executor-cores**
  每个executor使用的core数，Spark on Yarn默认为1，standalone默认为worker上所有可用的core。

### YARN-only:

- --driver-cores
  driver使用的core,仅在cluster模式下，默认为1。
- --queue 
  QUEUE_NAME  指定资源队列的名称,默认：default
- --num-executors
  一共启动的executor数量，默认是2个。

## Spark 源码分析

###  Spark  RPCEnv 通信环境创建

![](http://img.zwer.xyz/blog/Spark RPCEnv 环境准备+Master注册启动.jpg)



### SparkSubmit 资源申请完毕启动 Executor 

![](http://img.zwer.xyz/blog/SparkSubmit .jpg)





### Spark 源码结论

1.  Executor 是在集群中分散启动的， 利于数据处理的本地化
2.  当提交任务时， 什么参数都不指定， 默认集群为当前的 application 每台 Worker 启动 1 个 Executor ， 这个 Executor 使用当前这台 Worker 所有的 core 和 1G 内存
3. 想要在一台 worker 上启动多个 Executor ，提交任务时， 指定 --executor-cores 
4. 如果在集群中提交任务，指定 --total-executor-cores 指定当前 application 最多使用 core 的个数
5. 启动 Executor 不仅和 core 有关还和内存有关

![](http://img.zwer.xyz/blog/Spark源码结论.jpg)

**源码结论验证**

```shell
# 验证 2 
./spark-submit --master spark://node01:7077 --deploy-mode client --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 10000

# 验证结论 3 和 5
./spark-submit --master spark://node01:7077 --deploy-mode client --executor-cores 1 --executor-memory 2g --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 10000

# 验证 1 和 4
 ./spark-submit --master spark://node01:7077 --deploy-mode client --executor-cores 1 --total-executor-cores 2  --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 10000
```

注意： Driver 启动时默认也占用 一个 core 

## Spark 任务调度源码分析

。。

## Spark 二次排序问题

> 二次排序是指大于二列的排序 
>
> 二次排序的思想是**封装对象**，就将一条数据按照一定的规则划分不同的属性，然后将这些属性封装到一个类。在比较对象之间大小时，采用内部比较器进行比较大小。

测试数据

```
1 4
2 17
90 45
90 12
9 56
2 1
2 0
5 7
8 123
8 10
9 111
```

### Java 版本二次排序

> Java 中实现对象之间的比较大小需要实现 Comparable 接口（内部比较器）

```java
/**
 * @Auther:zwer
 * @Date:2019/12/10 15:08
 * @Description:com.szxy.myjavacode
 * @Version:1.0
 * Java 版本二次排序
 **/
public class SecondSort {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local");
        conf.setAppName("secondsort");

        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> rdd1 = sc.textFile("./data/secondsort");
        JavaPairRDD<SecondSort, String> rdd2 =
                rdd1.mapToPair(new PairFunction<String, SecondSort, String>() {
            @Override
            public Tuple2<SecondSort, String> call(String line) throws Exception {
                String[] nums = line.split(" ");
                return new Tuple2(new MySort(Integer.valueOf(nums[0]), Integer.valueOf(nums[1])), line);
            }
        });
        rdd2.sortByKey().foreach(new VoidFunction<Tuple2<SecondSort, String>>() {
            @Override
            public void call(Tuple2<SecondSort, String> tuple) throws Exception {
                System.out.println(tuple._1);
            }
        });

        sc.stop();
    }
}

class MySort implements  Comparable<MySort>, Serializable {
    private int num1;
    private int num2;

    @Override
    public int compareTo(MySort o1) {
        if(this.num1 - o1.num1 == 0){
            return -(this.num2 - o1.num2);
        }else{
            return -(this.num1 - o1.num1);
        }
    }

    public MySort() {
    }

    public MySort(int num1, int num2) {
        this.num1 = num1;
        this.num2 = num2;
    }

	// 省略 getter 和 setter 方法以及 toString 方法
}
```

###  Scala 版本二次排序

Scala 中实现对象之间的比较大小需要继承 Ordered 类

```scala
package com.szxy.myscalacode

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}



case class SecondSort(num1 : Int, num2 :Int) extends Ordered[SecondSort]{
  override def compare(that: SecondSort): Int = {
    if(num1 == that.num1){
      -(num2 - that.num2)
    }else{
       -(num1 - that.num1)
    }
  }
}

/**
 *  Scala 版本二次排序
 */
object SecondSort{
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf()
    conf.setMaster("local")
    conf.setAppName("second_sort")
    val sc: SparkContext = new SparkContext(conf)
    val rdd1: RDD[String] = sc.textFile("./data/secondsort")
    val rdd2: RDD[(SecondSort, String)] =
      rdd1.map(line => (new SecondSort(Integer.valueOf(line.split(" ")(0)), Integer.valueOf(line.split(" ")(1))), line))
    rdd2.sortByKey().foreach(one => println(one._1))

    sc.stop();
  }
}
```

## TopN 问题

>  分组取 topN ，取出每组中排名前 N 条数据

### java 版本

```java
/**
  * 分组取 topN ，取出每组中排名前三的数据
 **/
public class TopN {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local");
        conf.setAppName("testTopN");

        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> rdd1 = sc.textFile("./data/topNdata");
        JavaPairRDD<String, Integer> mapPairs = rdd1.mapToPair(new PairFunction<String, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(String line) throws Exception {
                return new Tuple2<String, Integer>(line.split(" ")[0], Integer.parseInt(line.split(" ")[1]));
            }
        });
        /**************  解法2 采用定长数组的形式 （推荐） ********************/
        mapPairs.groupByKey().foreach(new VoidFunction<Tuple2<String, Iterable<Integer>>>() {
            @Override
            public void call(Tuple2<String, Iterable<Integer>> tuple2) throws Exception {
                String key = tuple2._1;
                Iterator<Integer> iter = tuple2._2.iterator();
                // 声明一个长度为 3 的数组
                Integer[] topN = new Integer[3];
                while(iter.hasNext()){
                    Integer next = iter.next();
                    for(int i = 0; i < topN.length; i++){
                        if(topN[i] == null){
                            topN[i] = next;
                            break;
                        }else if(topN[i] < next){
                            for(int j = 2; j > i; j--){
                                topN[j] = topN[j-1];
                            }
                            topN[i] = next;
                            break;
                        }
                    }
                }
                for(int i = 0; i < topN.length; i++){
                    System.out.println(key+" "+topN[i]);
                }
            }
        });
        /******  解法1： 采用原生集合排序的方式 （不推荐）****************/
//        mapPairs.groupByKey().foreach(new VoidFunction<Tuple2<String, Iterable<Integer>>>() {
//            @Override
//            public void call(Tuple2<String, Iterable<Integer>> tuple2) throws Exception {
//                String key = tuple2._1;
//                Iterator<Integer> iter = tuple2._2.iterator();
//                List<Integer> list = new ArrayList<>();
//                while(iter.hasNext()){
//                    Integer next = iter.next();
//                    list.add(next);
//                }
//                // 排序
//                Collections.sort(list, new Comparator<Integer>() {
//                    @Override
//                    public int compare(Integer o1, Integer o2) {
//                        return -(o1-o2);
//                    }
//                });
//                // 输出到控制台
//                for (Integer integer : list) {
//                    System.out.println(key+" "+integer);
//                }
//            }
//        });

        sc.stop();
    }
}

```

### Scala 版本 TopN 

```scala
package com.szxy.myscalacode


import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.rdd.RDD

import scala.util.control.Breaks


/**
 * 分组取 TopN 问题
 */
object TopN {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setMaster("local")
    conf.setAppName("testTopN")
    val sc: SparkContext = new SparkContext(conf)
    val dataRdd: RDD[String] = sc.textFile("data/topNdata")
    val mapPairs: RDD[(String, Integer)] = dataRdd.map(line =>
      (line.split(" ")(0), Integer.valueOf(line.split(" ")(1))))
    mapPairs.groupByKey().foreach(one => {
      val key: String = one._1
      val iter: Iterator[Integer] = one._2.iterator
      val topN: Array[Int] = new Array[Int](3);

      // 数组越界问题 java.lang.ArrayIndexOutOfBoundsException: 1
      val loop = new Breaks
      while (iter.hasNext) {
        val next: Integer = iter.next()
        loop.breakable {
            for (i <- 0 until topN.size) {
              if (topN(i) == 0) {
                topN(i) = next;
                loop.break()
              } else if (topN(i) < next) {
                for (j <- 2 until(i, -1)) {
                  topN(j) = topN(j - 1)
                }
                topN(i) = next;
                loop.break()
              }
            }
        }
      }
      for(i <- 0 until topN.size){
          println(key+" "+topN(i))
      }
    })
  }
}

```

##  广播变量

当 Driver 端的变量需要在 Executor 端使用的时候， Driver 端的变量在未使用广播变量的时候，Driver 的端的变量将拷贝自身变量到  Executor 端上每个 task 上，这样会导致 Executor 端的内存占用升高，影响 Executor 端执行效率，拖慢 task 执行时间。

由此广播变量应运而生，广播变量会将 Driver 端的变量广播到 Executor 端，而不会导致每执行个 task 任务便会复制一份 Driver端的变量副本。

使用广播变量的好处是 减少 Executor 端的内存开销，同时也提高 executor 端 task 的执行效率，节约时间

解释: 

Driver 端的变量是指 SparkApplication 中声明在算子外的变量

Executor 端执行的代码是指 SparkApplication 中 RDD 使用的算子中回调函数

### 广播变量图解

![](http://img.zwer.xyz/blog/20191210205743.png)

### 广播变量 demo 

```scala
package com.szxy.myscalacode

import org.apache.spark.broadcast.Broadcast
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

/**
 *  测试广播变量
 *  Broadcast
 */
object BroadcastVariableDemo {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf()
    conf.setMaster("local")
    conf.setAppName("broadcastVariable")

    val sc: SparkContext = new SparkContext(conf)
    val list: List[String] = List[String]("hello Spark")
    // 将 list 添加到广播变量中
    val broadcat: Broadcast[List[String]] = sc.broadcast(list)
    val words: RDD[String] = sc.textFile("./data/word.txt")

    words.filter(one => {
      // 使用广播变量
      val bcList: List[String] = broadcat.value
      bcList.contains(one)
    }).foreach(println)

    sc.stop()
  }
}

```

## 累加器

统计分布式上 Executor 端某些值的累加， 累加器不仅可以统计数值类型，也可以统计自定义对象类型

### Spark 累加器版本问题

在 Spark 1.6 之前， 不支持在 Executor 使用累加器的值

在 Spark 2.0 之后， 支持在 Executor 使用累加器的值

### 累加器 demo1- Spark 1.6

```scala 
package com.szxy.myscalacode

import org.apache.spark.{Accumulator, SparkConf, SparkContext}
import org.apache.spark.rdd.RDD
import org.apache.spark.util.DoubleAccumulator

/**
 *  累加器
 *  Spark 1.6
 *
 */
object AccumulateDemo2 {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf()
    conf.setMaster("local")
    conf.setAppName("AccumulateDemo")

    val sc: SparkContext = new SparkContext(conf)
    val words: RDD[String] = sc.textFile("./data/word.txt")
    //累加器 ，  sc.accumulator(0) 在 Spark 1.6 低版本使用
    val accumulator: Accumulator[Int] = sc.accumulator(0)
    words.map(one => {
      accumulator.add(1)
      // Caused by: java.lang.UnsupportedOperationException: Can't read accumulator value in task
      // sc.accumulator 不支持读取累加器在 task 任务中
      println("accumulator:"+ accumulator.value)
      (one, 1)
    }).collect()
    // 打印累加器的结果值
    println("last -- accumulator:"+ accumulator.value)

    sc.stop()

  }
}

```

###  累加器 demo2-Spark 2.0 以上
```scala
package com.szxy.myscalacode

import org.apache.spark.rdd.RDD
import org.apache.spark.util.DoubleAccumulator
import org.apache.spark.{SparkConf, SparkContext}

object AccumulateDemo {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf()
    conf.setMaster("local")
    conf.setAppName("AccumulateDemo")

    val sc: SparkContext = new SparkContext(conf)
    val words: RDD[String] = sc.textFile("./data/word.txt")
     //累加器
    val accumulator: DoubleAccumulator = sc.doubleAccumulator
    words.map(one => {
      accumulator.add(1)
      println("accumulator:"+ accumulator.value)
      (one, 1)
    }).collect()

    //println("i=" + i) // 未使用的累加器的情况下，结果为 0
    println("last -- ;;;lkkioioikaccumulator:"+ accumulator.value)

    sc.stop()
  }
}
```



### 自定义累加器

自定义累加器需要继承` extends AccumulatorV2[T, D]`, 第一个为输入类型，第二个为输出类型

`val myVectorAcc = new VectorAccumulatorV2`

注意： 方法 `zero()` 要和 `reset()` 值保持一致

```scala
package com.bjsxt.scalaspark.core.examples

import org.apache.spark.util.AccumulatorV2
import org.apache.spark.{SparkConf, SparkContext}

/**
  * 自定义累加器:
  * 自定义累加器需要继承extends AccumulatorV2[String, String]，第一个为输入类型，第二个为输出类型
  * val myVectorAcc = new VectorAccumulatorV2
  *  方法 zero() 要和 reset() 值保持一致
  */
case class Info(var totalCount:Int,var totalAge :Int)


class SelfAccumlator extends AccumulatorV2 [Info,Info]{

  /**
    * 初始化累计器的值,这个值是最后要在merge合并的时候累加到最终结果内
    */
  private var result: Info = new Info(0,0)
//  println(s" in first result = $result  end。")

  /**
    * 返回累计器是否是零值。 例如： Int 类型累加器 0 就是零值，对于List 类型数据 Nil 就是零值。
    * 这里判断时，要与方法reset()初始的值一致，初始判断时要返回true. 内部会在每个分区内自动调用判断。
    */
  override def isZero: Boolean = {
//    println("判断 累加器是否是初始值***"+(result.totalAge == 0 && result.totalCount ==0)+" ***end")
    result.totalCount ==0 && result.totalAge == 0
  }

  /**
    * 复制一个新的累加器,在这里就是如果用到了就会复制一个新的累加器。
    */
  override def copy(): AccumulatorV2[Info, Info] = {
    val newAccumulator = new SelfAccumlator()
    newAccumulator.result = this.result
    newAccumulator
  }

  /**
    * 重置AccumulatorV2中的数据，这里初始化的数据是在RDD每个分区内部，每个分区内的初始值。
    */
  override def reset(): Unit = {
//    println("重置累加器中的值")
    result = new Info(0,0)
  }

  /**
    * 每个分区累加数据
    * 这里是拿着初始的result值和每个分区的数据累加
    */
  override def add(v: Info): Unit = {
//    println(s" in add method : v = $v ,v.totalCount = ${v.totalCount},v.totalAge = ${v.totalAge}")
    result.totalCount += v.totalCount
    result.totalAge += v.totalAge
  }

  /**
    * 分区之间总和累加数据
    *
    * 这里拿着初始的result值 和每个分区最终的结果累加
    *
    */
  override def merge(other: AccumulatorV2[Info, Info]): Unit = other match {
    case o : SelfAccumlator => {
//      println(s" in merge method : o = $o ")
      result.totalCount +=o.result.totalCount
      result.totalAge +=o.result.totalAge
    }
    case _ => throw new UnsupportedOperationException(
      s"Cannot merge ${this.getClass.getName} with ${other.getClass.getName}")
  }

  /**
    *  累计器对外返回的最终的结果
    */
  override def value: Info = result
}


object DefindSelfAccumulator {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setMaster("local")
    conf.setAppName("selfAccumulator")
    val sc = new SparkContext(conf)

    val nameList = sc.parallelize(List[String](
      "A 1","B 2","C 3",
      "D 4","E 5","F 6",
      "G 7","H 8","I 9"
    ),3)
    println("nameList RDD partition length = "+nameList.getNumPartitions)
    /**
      * 初始化累加器
      *
      */
    val myAccumulator = new SelfAccumlator()
    // 注册自定义累加器
    sc.register(myAccumulator, "First Accumulator")

    val transInfo = nameList.map(one=>{
      val info = Info(1,one.split(" ")(1).toInt)
      myAccumulator.add(info)
      info
    })

    transInfo.count()

    println(s"accumulator totalCount = ${myAccumulator.value.totalCount}, totalAge = ${myAccumulator.value.totalAge}")


  }
}

```



## SparkUI

###  spark shell命令

命令位置
	在 $SPARK_HOME/bin 下有 spark-shell 命令
作用
	用于 Spark Application 快速开发
命令使用

```shell
./spark-shell --master spark://node01:7077  --name myapp
```

测试使用

```scala
sc.textFile("hdfs://mycluster:8020/spark/data/word.txt")
.flatMap(_.split(" "))
.map((_,1))
.reduceByKey(_+_)
.collect()
```

注意 :	
	在 submit-shell 界面默认已经有 SparkContext 对象 sc ，所以不需要自己重新创建

##   Spark 历史日志服务配置

配置位置： Spark 客户端

开启历史日志服务器配置

```
1. 进入 $SPARK_HOME/conf 目录下 
2. 拷贝 cp  spark-defaults.conf.template spark-defaults.conf
3. vi spark-default.conf  
开启历史日志服务项
	放开  spark.eventLog.enabled true
选择历史日志存放目录
	spark.eventLog.dir  hdfs://myCluster:8020/spark/log
启动历史日志时日志恢复的路径
	spark.history.fs.logDirectory hdfs://myCluster:8020/spark/log
```

开启历史日志服务器配置
1. 进入 $SPARK_HOME/bin 目录下
2.  ./start-history-server.sh
3. jps 查看历史日志服务器是否启动成功
4. 访问 IP：18080 端口

注意： 历史日志存放目录必须手动开启

## Spark 二种 shuffle 机制

### 什么是 Spark Shuffle 

在Spark应用程序中，RDD中某个分区的数据量相对于其他分区数据量大，对应当前处理数据的task执行时间长。在Spark中如果没有shuffle就没有数据倾斜。



reduceByKey会将上一个RDD中的每一个key对应的所有value聚合成一个value，然后生成一个新的RDD，元素类型是<key,value>对的形式，这样每一个key对应一个聚合起来的value。
**问题**：聚合之前，每一个key对应的value不一定都是在一个partition中，也不太可能在同一个节点上，因为RDD是分布式的弹性的数据集，RDD的partition极有可能分布在各个节点上。

### Spark Shuffle 的分类
- Shuffle Write
	在 map task 中，将包含相同 key 的数据写入同一个分区
- Shuffle Read
在 reduce task 中， 将 map 端包含相同 key 的分区拉取到同一个节点上进行聚合操作

### Spark 不同版本对应的 Shuffle 机制

- **Spark 1.6x**

  HashShuffleManager

  1. 内存缓冲区默认 32K ，当内存缓冲区 buffer 满了之后，便会溢写到磁盘中。 溢写磁盘小文件多，即 耗时低效的IO操作多

   2. 读写文件会创建大量的对象，占用 JVM 内存。若 GC 不足，则可能发生 OOM 内存溢出的风险

   3. 将大量的文件需要从 Map task 端发送到 Reduce task 端，需要建立大量的连接。一旦有文件建立连接超过三次都失败了，后面的 Stage 便会重新开始，这样的结果导致 task 执行效率低 
     
  
  优化后的 HashShuffleManager 
          同一个 core 中 task 任务共有同一个内存 buffer 缓冲区
   好处：减少了 Map task 至少一半的磁盘小文件的数目，即在一定程度上减少了磁盘 IO 操作。原来的磁盘小文件的个数从 Map * Reduce 个，到现在  Core * Reduce 个
  
- **Spark 2.3.1** 
  SortShuffleManger（重点）

   运行机制： 普通机制 和 byPass 机制

  ByPass 机制的应用场景: Map task 没有预聚合的过程
  ByPass 机制与普通机制的区别1. 减少排序的过程 2. 速度更快



##  Spark 文件寻址

![](http://img.zwer.xyz/blog/20191211202140.png)

###   shuffle 文件寻址图解

![](http://img.zwer.xyz/blog/Shuffle 文件寻址.jpg)

若 Reduce 端出现 OOM 内存溢出的情况，怎么样处理？

1. 减少拉取数据量
2. **增大 Executor 内存**
3. 增到 Executor Shuffle 内存比例

###  Shuffle 调节

```
设置参数：
1.在代码中设置，硬编码问题
2.在spark-defaluts.conf中设置
3.最好在提交任务设置 ,./spark-submit --conf xxx = xxx


spark.reducer.maxSizeInFlight
默认值：48m 
参数说明：该参数用于设置shuffle read task的buffer缓冲大小，而这个buffer缓冲决定了每次能够拉取多少数据。
调优建议：如果作业可用的内存资源较为充足的话，可以适当增加这个参数的大小（比如96m），从而减少拉取数据的次数，也就可以减少网络传输的次数，进而提升性能。在实践中发现，合理调节该参数，性能会有1%~5%的提升。

spark.shuffle.io.maxRetries
默认值：3
参数说明：shuffle read task从shuffle write task所在节点拉取属于自己的数据时，如果因为网络异常导致拉取失败，是会自动进行重试的。该参数就代表了可以重试的最大次数。如果在指定次数之内拉取还是没有成功，就可能会导致作业执行失败。
调优建议：对于那些包含了特别耗时的shuffle操作的作业，建议增加重试最大次数（比如60次），以避免由于JVM的full gc或者网络不稳定等因素导致的数据拉取失败。在实践中发现，对于针对超大数据量（数十亿~上百亿）的shuffle过程，调节该参数可以大幅度提升稳定性。
shuffle file not find    taskScheduler不负责重试task，由DAGScheduler负责重试stage

spark.shuffle.io.retryWait
默认值：5s
参数说明：具体解释同上，该参数代表了每次重试拉取数据的等待间隔，默认是5s。
调优建议：建议加大间隔时长（比如60s），以增加shuffle操作的稳定性。

spark.shuffle.sort.bypassMergeThreshold----针对SortShuffle
默认值：200
参数说明：当ShuffleManager为SortShuffleManager时，如果shuffle read task的数量小于这个阈值（默认是200），则shuffle write过程中不会进行排序操作，而是直接按照未经优化的HashShuffleManager的方式去写数据，但是最后会将每个task产生的所有临时磁盘文件都合并成一个文件，并会创建单独的索引文件。
调优建议：当你使用SortShuffleManager时，如果的确不需要排序操作，那么建议将这个参数调大一些，大于shuffle read task的数量。那么此时就会自动启用bypass机制，map-side就不会进行排序了，减少了排序的性能开销。但是这种方式下，依然会产生大量的磁盘文件，因此shuffle write性能有待提高。


```

## Spark 内存管理机制

Spark 内存管理机制分为静态内存管理和统一内存管理

在 spark 1.6 之前使用静态内存管理机制，在 Spark 1.6 之后引入统一内存管理。

而在 Spark 2.0+ 版本中，使用统一内存管理机制

***静态内存管理***中存储内存、执行内存和其他内存的大小在 Spark 应用程序运行期间均为固定的，但用户可以应用程序启动前进行配置。

**统一内存管理**与静态内存管理的区别在于存储内存和执行内存共享同一块空间，可以互相借用对方的空间。

Spark1.6以上版本默认使用的是统一内存管理，可以通过参数spark.memory.useLegacyMode 设置为true(默认为false)使用静态内存管理。

![](http://img.zwer.xyz/blog/Spark内存管理.jpg)



