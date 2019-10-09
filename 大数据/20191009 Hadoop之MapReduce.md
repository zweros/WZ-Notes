---
title: 20191009 Hadoop-MapReduce
date: 2019-10-09
---

# MapReduce #

> MapTask & ReduceTask

一个切片对应一个 Map，也就是说切片的数量决定了 Map 的数量

split 切片指逻辑上概念，用于指定 Map 处理数据的大小

切片用于将 HDFS中的块与 Map 之间解耦

Reduce 的数量由人来决定，根据前面的组的推导

![](http://img.zwer.xyz/blog/20191009101915.png)

### <font color='green'>MR  原语</font> ###

输入(格式化k,v)数据集 -> map映射成一个中间数据集(k,v)  -> reduce
<font color='red'>“相同”的 key 为一组，调用一次 reduce 方法，方法内迭代这一组数据进行计算</font>

| 关系/对应比例 | block > split | split > map | map > reduce | group(key)>partition                | partition > outputfile |
| ------------- | ------------- | ----------- | ------------ | ----------------------------------- | ---------------------- |
| 1:1           | *             | *           | *            | *                                   |                        |
| 1:N           | *             |             | *            | <font color='red'>违背了原语</font> |                        |
| N:1           | *             |             | *            | *                                   |                        |
| N:M           |               |             | *            | *                                   |                        |

### Shuffler<洗牌> ###

> 框架内部实现机制
> 分布式计算节点数据流转：连接 MapTask 与 ReduceTask

![](http://img.zwer.xyz/blog/20191009110701.png)

### 计算框架 ###

![](http://img.zwer.xyz/blog/20191009111332.png)

|                            | 作用                                                         |
| -------------------------- | ------------------------------------------------------------ |
| Map                        | 读懂数据<br/>映射为KV模型<br/>并行分布式<br/><font color='red'>计算向数据移动</font> |
| Reduce                     | 数据全量/分量加工<br/>Reduce中可以包含不同的key<br/>相同的Key汇聚到一个Reduce中<br/>相同的Key调用一次reduce方法<br/>排序实现key的汇聚<br/> |
| K,V使用自定义数据类型<br/> | 作为参数传发成本，提高程序自由度<br/>- Writable 序列化<br/> - Comparable 比较器<br/>实现具体排序（字典序，数值序等） |

### MapReduce 1.x ###

> 计算向数据移动

![](http://img.zwer.xyz/blog/20191009113152.png)

#### 计算框架 Mapper ####

![](http://img.zwer.xyz/blog/20191009113223.png)

#### 计算框架 Reducer ####

![](http://img.zwer.xyz/blog/20191009113300.png)



| MRv1角色：  | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| JobTracker  | 核心，<font color='red'>主</font>，单点<br/>调度所有的作业<br/>监控整个集群的资源负载 |
| TaskTracker | <font color='red'>从</font>，自身节点资源管理<br/>和 JobTracker 心跳，汇报资源，获取Task |
| Client      | 作业为单位<br/>规划作业计算分布<br/>提交作业资源到HDFS<br/>最终提交作业到 JobTracker |
| 弊端：      | JobTracker：负载过重，单点故障<br/>资源管理与计算调度强耦合，其他计算框架需要重复实现资源管理<br/>不同框架对资源不能全局管理 |

### MRV2 之 YARN ###

>  YARN：解耦资源与计算

![](http://img.zwer.xyz/blog/20191009140653.png)


|                 | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| ResourceManager | 主，核心<br/>集群节点资源管理                                |
| NodeManager     | 与RM汇报资源<br/>管理Container生命周期<br/>计算框架中的角色都以Container表示 |
| Container       | 【节点NM，CPU,MEM,I/O大小，启动命令】<br/>默认NodeManager启动线程监控Container大小，超出申请资源额度，kill<br/>支持Linux内核的Cgroup |
| MR              | - MR-ApplicationMaster-Container<br/>x作业为单位，避免单点故障，负载到不同的节点<br/>创建Task需要和RM申请资源（Container）<br/>- Task-Container |
| Client          | RM-Client：请求资源创建AM<br/>AM-Client：与AM交互            |

## 搭建 yarn ##

### 准备 ###

|        | NN-1 | NN-2 | DN   | ZK   | ZKFC | JNN  | RM   | NM   |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| node01 | *    |      |      |      | *    | *    |      |      |
| node02 |      | *    | *    | *    | *    | *    |      | *    |
| node03 |      |      | *    | *    |      | *    | *    | *    |
| node04 |      |      | *    | *    |      |      | *    | *    |

说明： 

	1. HA 高可用 HDFS 
 	2. RM 资源管理器采用主从架构，使用 Zookeeper 做分布式协调
 	3. NM 的数量与 DN 的数量相同

### 修改配置文件 ###

- mapred-site.xml 

```xml
<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
  </property>
```

- yarn-site.xml

```xml
<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
</property>
<property>
    <name>yarn.resourcemanager.ha.enabled</name>
    <value>true</value>
</property>
<property>
    <name>yarn.resourcemanager.cluster-id</name>
    <value>cluster1</value>
</property>
<property>
    <name>yarn.resourcemanager.ha.rm-ids</name>
    <value>rm1,rm2</value>
</property>
<property>
    <name>yarn.resourcemanager.hostname.rm1</name>
    <value>node03</value>
</property>
<property>
    <name>yarn.resourcemanager.hostname.rm2</name>
    <value>node04</value>
</property>
<property>
    <name>yarn.resourcemanager.zk-address</name>
    <value>node02:2181,node03:2181,node04:2181</value>
</property>
```

- 分发

```shell
# 将 node01 修改的配置文件分发给 node02、node03、node04
scp mapred-site.xml yarn-site.xml root@node02:`pwd`
scp mapred-site.xml yarn-site.xml root@node03:`pwd`
scp mapred-site.xml yarn-site.xml root@node04:`pwd`
```

### 部署 yarn ###

```shell
# 在 node01。 
# node01 可以免秘钥直接访问其他三个节点 node02、node03、node04
# 这样 node01 上 hadoop 管理脚本可以直接操纵其他其他机器上的 hadoop
start-yarn.sh 
# 在 node03、node04
yarn-daemon.sh start resourcemanager
```

### 访问 yarn web 界面 ###

```http
http://node03:8088
```

![](http://img.zwer.xyz/blog/20191009153119.png)

直接访问 http://node04:8088 ,会自动重定向到  http://node03:8088

![](http://img.zwer.xyz/blog/20191009153209.png)

### 测试-运行 wordCount 程序 ###

```shell
# 进入 hadoop-2.6.5/share/hadoop/mapreduce 目录下
hadoop jar hadoop-mapreduce-examples-2.6.5.jar wordcount /user/root/test.txt /data/wc/output
```

## 手写 wordcount 程序 ##

### 客户端 ###

```java
public class MainClient {

	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration(true);
		Job job = Job.getInstance(conf);

		// Create a new Job
		// Job job = Job.getInstance();
		job.setJarByClass(MainClient.class);

		// Specify various job-specific parameters
		job.setJobName("myjob");

		// job.setInputPath(new Path("in"));
		// job.setOutputPath(new Path("out"));
		// import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
		Path path = new Path("/user/root/test.txt");
		FileInputFormat.addInputPath(job, path);
		
		Path output = new Path("/data/wc/output");
		if(output.getFileSystem(conf).exists(output)){
			output.getFileSystem(conf).delete(output, true);
		}
		FileOutputFormat.setOutputPath(job, output );
		
		job.setMapperClass(MyMapper.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		
		
		job.setReducerClass(MyReducer.class);

		// Submit the job, then poll for progress until the job is complete
		job.waitForCompletion(true);

	}

}

```

### Mapper ###

```java
/**
 * 自定义 Mapper
 * 
 * @author zwer Hadoop 对基本数据类型进行了包装
 * 
 *
 */
public class MyMapper extends Mapper<Object, Text, Text, IntWritable> {

	private final static IntWritable one = new IntWritable(1);
	private Text word = new Text();

	public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
		// StringTokenizer 对单词数字进行分割
		StringTokenizer itr = new StringTokenizer(value.toString());
		while (itr.hasMoreTokens()) {
			word.set(itr.nextToken());
			context.write(word, one);
		}
	}

}
```

### Reducer ###

```java
public class MyReducer
    extends Reducer<Text, IntWritable, Text, IntWritable> {

	private IntWritable result = new IntWritable();
	//相同的 key 为一组，调用一次 reduce 方法，方法内迭代这一组数据进行计算 (sum count max min)
	public void reduce(Text key, Iterable<IntWritable> values, Context context)
			throws IOException, InterruptedException {
		int sum = 0;
		for (IntWritable val : values) {
			sum += val.get();
		}
		result.set(sum);
		context.write(key, result);
	}

}
```

注意： 导出 jar 的 JDK 版本与 Linux 上 JDK 版本（大版本号）一致



