---
title: 20191009 Hadoop-MapReduce
date: 2019-10-09
---

# MapReduce #

## MR 理论

> MapTask & ReduceTask

一个切片对应一个 Map，也就是说切片的数量决定了 Map 的数量

split 切片指逻辑上概念，用于指定 Map 处理数据的大小

切片用于将 HDFS 中的块与 Map 之间解耦

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

### <font color='#aa000'>Shuffler<洗牌></font> ###

> 框架内部实现机制
> 分布式计算节点数据流转：连接 MapTask 与 ReduceTask

![](http://img.zwer.xyz/blog/20191009110701.png)

注意： Map 阶段上，kv —> kvp ，增加分区的目的： 防止数据倾斜，导致数据都集中到一个 reduce 中，而其他 reduce 没有数据。

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

- **计算框架 Mapper**

![](http://img.zwer.xyz/blog/20191009113223.png)

- **计算框架 Reducer**

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
        
        // 设置切片大小
		// FileInputFormat.setMinInputSplitSize(job, size);
		// FileInputFormat.setMaxInputSplitSize(job, size);
		FileInputFormat.addInputPath(job, path);
		
		Path output = new Path("/data/wc/output");
		if(output.getFileSystem(conf).exists(output)){
			output.getFileSystem(conf).delete(output, true);
		}
		FileOutputFormat.setOutputPath(job, output );
		
		job.setMapperClass(MyMapper.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		
		// 设置 Reduce 的数量, Reduce 的数量由 group(key) 的数量决定
		// job.setNumReduceTasks(tasks);
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

## 源码分析

### Client 源码分析 ###

#### 开始 ####

```java
// Submit the job, then poll for progress until the job is complete
job.waitForCompletion(true);
```

#### submit()

```java
/**
   * Submit the job to the cluster and return immediately.
   * @throws IOException
   */
public void submit() 
    throws IOException, InterruptedException, ClassNotFoundException {
    ensureState(JobState.DEFINE);
    setUseNewAPI();
    connect();
    final JobSubmitter submitter = 
        getJobSubmitter(cluster.getFileSystem(), cluster.getClient());
    status = ugi.doAs(new PrivilegedExceptionAction<JobStatus>() {
        public JobStatus run() throws IOException, InterruptedException, 
        ClassNotFoundException {
            return submitter.submitJobInternal(Job.this, cluster);
        }
    });
    state = JobState.RUNNING;
    LOG.info("The url to track the job: " + getTrackingURL());
}
```

#### submitJobInternal(...) 方法的作用 ####

The job submission process involves:  

1.    Checking the input and output specifications of the job.    
2.    Computing the `InputSplit`s for the job.    
3.    Setup the requisite accounting information for the     `DistributedCache` of the job, if necessary.    
4.    Copying the job's jar and configuration to the map-reduce system   directory on the distributed file-system.     
5.    Submitting the job to the `JobTracker` and optionally   monitoring it's status. 

```java
  JobStatus submitJobInternal(Job job, Cluster cluster) 
  throws ClassNotFoundException, InterruptedException, IOException {

    //validate the jobs output specs 
    checkSpecs(job);
	 //....

      copyAndConfigureFiles(job, submitJobDir);

      Path submitJobFile = JobSubmissionFiles.getJobConfPath(submitJobDir);
      
      // Create the splits for the job
      LOG.debug("Creating splits at " + jtFs.makeQualified(submitJobDir));
      int maps = writeSplits(job, submitJobDir);
	  //...
 }
      
private int writeSplits(org.apache.hadoop.mapreduce.JobContext job,
                        Path jobSubmitDir) throws IOException,
InterruptedException, ClassNotFoundException {
    JobConf jConf = (JobConf)job.getConfiguration();
    int maps;
    if (jConf.getUseNewMapper()) {
        maps = writeNewSplits(job, jobSubmitDir);
    } else {
        maps = writeOldSplits(jConf, jobSubmitDir);
    }
    return maps;
}      


  @SuppressWarnings("unchecked")
  private <T extends InputSplit>
  int writeNewSplits(JobContext job, Path jobSubmitDir) throws IOException,
      InterruptedException, ClassNotFoundException {
    Configuration conf = job.getConfiguration();
    InputFormat<?, ?> input =
      ReflectionUtils.newInstance(job.getInputFormatClass(), conf);

    List<InputSplit> splits = input.getSplits(job);
    T[] array = (T[]) splits.toArray(new InputSplit[splits.size()]);

    // sort the splits into order based on size, so that the biggest
    // go first
    Arrays.sort(array, new SplitComparator());
    JobSplitWriter.createSplitFiles(jobSubmitDir, conf, 
        jobSubmitDir.getFileSystem(conf), array);
    return array.length;
  }


```

#### <font color="pinkyellow">FileInputFormat -getSplits(...) </font>* ####

```java
/** 
   * Generate the list of files and make them into FileSplits.
   * @param job the job context
   * @throws IOException
   */
  public List<InputSplit> getSplits(JobContext job) throws IOException {
    Stopwatch sw = new Stopwatch().start();
    long minSize = Math.max(getFormatMinSplitSize(), getMinSplitSize(job));
    long maxSize = getMaxSplitSize(job);

    // generate splits
    List<InputSplit> splits = new ArrayList<InputSplit>();
    List<FileStatus> files = listStatus(job);
    for (FileStatus file: files) {
      Path path = file.getPath();
      long length = file.getLen();
      if (length != 0) {
        BlockLocation[] blkLocations;
        if (file instanceof LocatedFileStatus) {
          blkLocations = ((LocatedFileStatus) file).getBlockLocations();
        } else {
          FileSystem fs = path.getFileSystem(job.getConfiguration());
          blkLocations = fs.getFileBlockLocations(file, 0, length);
        }
        if (isSplitable(job, path)) {
          long blockSize = file.getBlockSize();
          // 若要切片比块小， 调整 maxsize 大小
          // 若要切片比块大， 调整 minszie 大小
          long splitSize = computeSplitSize(blockSize, minSize, maxSize);

          long bytesRemaining = length;
          while (((double) bytesRemaining)/splitSize > SPLIT_SLOP) {
            int blkIndex = getBlockIndex(blkLocations, length-bytesRemaining);
            splits.add(makeSplit(path, length-bytesRemaining, splitSize,
                        blkLocations[blkIndex].getHosts(),
                        blkLocations[blkIndex].getCachedHosts()));
            bytesRemaining -= splitSize;
          }

          if (bytesRemaining != 0) {
            int blkIndex = getBlockIndex(blkLocations, length-bytesRemaining);
            splits.add(makeSplit(path, length-bytesRemaining, bytesRemaining,
                       blkLocations[blkIndex].getHosts(),
                       blkLocations[blkIndex].getCachedHosts()));
          }
        } else { // not splitable
          splits.add(makeSplit(path, 0, length, blkLocations[0].getHosts(),
                      blkLocations[0].getCachedHosts()));
        }
      } else { 
        //Create empty hosts array for zero length files
        splits.add(makeSplit(path, 0, length, new String[0]));
      }
    }
    // Save the number of input files for metrics/loadgen
    job.getConfiguration().setLong(NUM_INPUT_FILES, files.size());
    sw.stop();
    if (LOG.isDebugEnabled()) {
      LOG.debug("Total # of splits generated by getSplits: " + splits.size()
          + ", TimeTaken: " + sw.elapsedMillis());
    }
    return splits;
  }

  protected long computeSplitSize(long blockSize, long minSize,
                                  long maxSize) {
    return Math.max(minSize, Math.min(maxSize, blockSize));
  }

  protected int getBlockIndex(BlockLocation[] blkLocations, 
                              long offset) {
    for (int i = 0 ; i < blkLocations.length; i++) {
      // is the offset inside this block?
      if ((blkLocations[i].getOffset() <= offset) &&
          (offset < blkLocations[i].getOffset() + blkLocations[i].getLength())){
        return i;
      }
    }
    BlockLocation last = blkLocations[blkLocations.length -1];
    long fileLength = last.getOffset() + last.getLength() -1;
    throw new IllegalArgumentException("Offset " + offset + 
                                       " is outside of file (0.." +
                                       fileLength + ")");
  }
```

注意：

若一个切片包括多个块，mapper 程序只会移动到一个块中，其他块通过网络读取。

反之若一个块包括多个切片，并且块的副本数对应切片数,mapper 程序移动到每个块的副本位置上，这样就对一个块进行并行计算。即切片越小，计算的并行度越高

<font color='red' size='4px'>切片生成</font>

```java
long bytesRemaining = length;
while (((double) bytesRemaining)/splitSize > SPLIT_SLOP) {
    int blkIndex = getBlockIndex(blkLocations, length-bytesRemaining);
    splits.add(makeSplit(path, length-bytesRemaining, splitSize,
                         blkLocations[blkIndex].getHosts(),
                         blkLocations[blkIndex].getCachedHosts()));
    bytesRemaining -= splitSize;
}
```

<font color='red' size='4px'>切片的四要素: </font>

| 切片   | 说明                  |
| ------ | --------------------- |
| file   | 切片属于的文件        |
| start  | 切片的起始位置,偏移量 |
| length | 切片的大小            |
| host   | 块所在的物理位置      |

   

### Map-input

#### MapTask.java

```java
@Override
public void run(final JobConf job, final TaskUmbilicalProtocol umbilical)
    throws IOException, ClassNotFoundException, InterruptedException {
    this.umbilical = umbilical;

    if (isMapTask()) {
        // If there are no reducers then there won't be any sort. Hence the map 
        // phase will govern the   attempt's progress.
        if (conf.getNumReduceTasks() == 0) {
            mapPhase = getProgress().addPhase("map", 1.0f);
        } else {
            // If there are reducers then the entire attempt's progress will be 
            // split between the map phase (67%) and the sort phase (33%).
            mapPhase = getProgress().addPhase("map", 0.667f);
            sortPhase  = getProgress().addPhase("sort", 0.333f);
        }
    }
    TaskReporter reporter = startReporter(umbilical);

    boolean useNewApi = job.getUseNewMapper();
    initialize(job, getJobID(), reporter, useNewApi);

    // check if it is a cleanupJobTask
    if (jobCleanup) {
        runJobCleanupTask(umbilical, reporter);
        return;
    }
    if (jobSetup) {
        runJobSetupTask(umbilical, reporter);
        return;
    }
    if (taskCleanup) {
        runTaskCleanupTask(umbilical, reporter);
        return;
    }

    if (useNewApi) {
        runNewMapper(job, splitMetaInfo, umbilical, reporter);
    } else {
        runOldMapper(job, splitMetaInfo, umbilical, reporter);
    }
    done(umbilical, reporter);
}

 @SuppressWarnings("unchecked")
  private <INKEY,INVALUE,OUTKEY,OUTVALUE>
  void runNewMapper(final JobConf job,
                    final TaskSplitIndex splitIndex,
                    final TaskUmbilicalProtocol umbilical,
                    TaskReporter reporter
                    ) throws IOException, ClassNotFoundException,
                             InterruptedException {
    // make a task context so we can get the classes
    org.apache.hadoop.mapreduce.TaskAttemptContext taskContext =
      new org.apache.hadoop.mapreduce.task.TaskAttemptContextImpl(job, 
                                                                  getTaskID(),
                                                                  reporter);
    // make a mapper
    org.apache.hadoop.mapreduce.Mapper<INKEY,INVALUE,OUTKEY,OUTVALUE> mapper =
      (org.apache.hadoop.mapreduce.Mapper<INKEY,INVALUE,OUTKEY,OUTVALUE>)
        ReflectionUtils.newInstance(taskContext.getMapperClass(), job);
    // make the input format
    org.apache.hadoop.mapreduce.InputFormat<INKEY,INVALUE> inputFormat =
      (org.apache.hadoop.mapreduce.InputFormat<INKEY,INVALUE>)
        ReflectionUtils.newInstance(taskContext.getInputFormatClass(), job);
    // rebuild the input split
    org.apache.hadoop.mapreduce.InputSplit split = null;
    split = getSplitDetails(new Path(splitIndex.getSplitLocation()),
        splitIndex.getStartOffset());
    LOG.info("Processing split: " + split);

    org.apache.hadoop.mapreduce.RecordReader<INKEY,INVALUE> input =
      new NewTrackingRecordReader<INKEY,INVALUE>
        (split, inputFormat, reporter, taskContext);
    
    job.setBoolean(JobContext.SKIP_RECORDS, isSkipping());
    org.apache.hadoop.mapreduce.RecordWriter output = null;
    
    // get an output object
    if (job.getNumReduceTasks() == 0) {
      output = 
        new NewDirectOutputCollector(taskContext, job, umbilical, reporter);
    } else {
      output = new NewOutputCollector(taskContext, job, umbilical, reporter);
    }

    org.apache.hadoop.mapreduce.MapContext<INKEY, INVALUE, OUTKEY, OUTVALUE> 
    mapContext = 
      new MapContextImpl<INKEY, INVALUE, OUTKEY, OUTVALUE>(job, getTaskID(), 
          input, output, 
          committer, 
          reporter, split);

    org.apache.hadoop.mapreduce.Mapper<INKEY,INVALUE,OUTKEY,OUTVALUE>.Context 
        mapperContext = 
          new WrappedMapper<INKEY, INVALUE, OUTKEY, OUTVALUE>().getMapContext(
              mapContext);

    try {
      input.initialize(split, mapperContext);
      mapper.run(mapperContext);
      mapPhase.complete();
      setPhase(TaskStatus.Phase.SORT);
      statusUpdate(umbilical);
      input.close();
      input = null;
      output.close(mapperContext);
      output = null;
    } finally {
      closeQuietly(input);
      closeQuietly(output, mapperContext);
    }
  }
```

#### <font color='pingyellow'>LineRecordReader之initialize(...)</font>*

```java
public void initialize(InputSplit genericSplit,
                         TaskAttemptContext context) throws IOException {
    FileSplit split = (FileSplit) genericSplit;
    Configuration job = context.getConfiguration();
    this.maxLineLength = job.getInt(MAX_LINE_LENGTH, Integer.MAX_VALUE);
    start = split.getStart();
    end = start + split.getLength();
    final Path file = split.getPath();

    // open the file and seek to the start of the split
    final FileSystem fs = file.getFileSystem(job);
    fileIn = fs.open(file);
    
    CompressionCodec codec = new CompressionCodecFactory(job).getCodec(file);
    if (null!=codec) {
      isCompressedInput = true;	
      decompressor = CodecPool.getDecompressor(codec);
      if (codec instanceof SplittableCompressionCodec) {
        final SplitCompressionInputStream cIn =
          ((SplittableCompressionCodec)codec).createInputStream(
            fileIn, decompressor, start, end,
            SplittableCompressionCodec.READ_MODE.BYBLOCK);
        in = new CompressedSplitLineReader(cIn, job,
            this.recordDelimiterBytes);
        start = cIn.getAdjustedStart();
        end = cIn.getAdjustedEnd();
        filePosition = cIn;
      } else {
        in = new SplitLineReader(codec.createInputStream(fileIn,
            decompressor), job, this.recordDelimiterBytes);
        filePosition = fileIn;
      }
    } else {
      fileIn.seek(start);
      in = new UncompressedSplitLineReader(
          fileIn, job, this.recordDelimiterBytes, split.getLength());
      filePosition = fileIn;
    }
    // If this is not the first split, we always throw away first record
    // because we always (except the last split) read one extra line in
    // next() method.
    if (start != 0) {
      start += in.readLine(new Text(), 0, maxBytesToConsume(start));
    }
    this.pos = start;
  }
  
```

#### 格式化 InputFormat 的作用

| InputFormat    |                      |
| -------------- | -------------------- |
| 在客户端 phase | 生成 splits 切片清单 |
| 在 mapper      | 生成行读取器         |

<font color='red' size='4px'>Mapper 读取行的过程</font>

假设： 一个切片对应一个块

第一个切片会读取当前切片上全部行以及第二切片的第一行。

从第二个切片开始，每次从当前切片的第二行开始读取，直接跳过第一行（读取第一行只算字节数，不操作）。

这样做的目的：保持 HDFS 数据的完整性

```java
if (start != 0) {
    start += in.readLine(new Text(), 0, maxBytesToConsume(start));
}
```

#### Map-input 的工作



<div style='width:100%;height:20px;background:#ee0a0a'></div>
### Map-output

```java
//NewOutputCollector

    @SuppressWarnings("unchecked")
    NewOutputCollector(org.apache.hadoop.mapreduce.JobContext jobContext,
                       JobConf job,
                       TaskUmbilicalProtocol umbilical,
                       TaskReporter reporter
                       ) throws IOException, ClassNotFoundException {
      collector = createSortingCollector(job, reporter);
      partitions = jobContext.getNumReduceTasks();
      if (partitions > 1) {
        partitioner = (org.apache.hadoop.mapreduce.Partitioner<K,V>)
          ReflectionUtils.newInstance(jobContext.getPartitionerClass(), job);
      } else {
        partitioner = new org.apache.hadoop.mapreduce.Partitioner<K,V>() {
          @Override
          public int getPartition(K key, V value, int numPartitions) {
            return partitions - 1;
          }
        };
      }
    }
    // 输出 key value 以及分区号
    @Override
    public void write(K key, V value) throws IOException, InterruptedException {
      collector.collect(key, value,
                        partitioner.getPartition(key, value, partitions));
    }


// 默认的分区器
public class HashPartitioner<K, V> extends Partitioner<K, V> {

  /** Use {@link Object#hashCode()} to partition. */
  public int getPartition(K key, V value,
                          int numReduceTasks) {
    return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
  }
```

#### createSortingCollector()

```java
  private <KEY, VALUE> MapOutputCollector<KEY, VALUE>
          createSortingCollector(JobConf job, TaskReporter reporter)
    throws IOException, ClassNotFoundException {
    MapOutputCollector.Context context =
      new MapOutputCollector.Context(this, job, reporter);

    Class<?>[] collectorClasses = job.getClasses(
      JobContext.MAP_OUTPUT_COLLECTOR_CLASS_ATTR, MapOutputBuffer.class);
    int remainingCollectors = collectorClasses.length;
    for (Class clazz : collectorClasses) {
      try {
        if (!MapOutputCollector.class.isAssignableFrom(clazz)) {
          throw
              new IOException("Invalid output collector class: " + clazz.getName() +
            " (does not implement MapOutputCollector)");
        }
        Class<? extends MapOutputCollector> subclazz =
          clazz.asSubclass(MapOutputCollector.class);
        LOG.debug("Trying map output collector class: " + subclazz.getName());
        MapOutputCollector<KEY, VALUE> collector =
          ReflectionUtils.newInstance(subclazz, job);
        collector.init(context);
        LOG.info("Map output collector class = " + collector.getClass().getName());
        return collector;
      } catch (Exception e) {
        String msg = "Unable to initialize MapOutputCollector " + clazz.getName();
        if (--remainingCollectors > 0) {
          msg += " (" + remainingCollectors + " more collector(s) to try)";
        }
        LOG.warn(msg, e);
      }
    }
    throw new IOException("Unable to initialize any output collector");
  }
```

####  collector.init(context)	

```java
public void init(MapOutputCollector.Context context
                    ) throws IOException, ClassNotFoundException {
      job = context.getJobConf();
      reporter = context.getReporter();
      mapTask = context.getMapTask();
      mapOutputFile = mapTask.getMapOutputFile();
      sortPhase = mapTask.getSortPhase();
      spilledRecordsCounter = reporter.getCounter(TaskCounter.SPILLED_RECORDS);
      partitions = job.getNumReduceTasks();
      rfs = ((LocalFileSystem)FileSystem.getLocal(job)).getRaw();

      //sanity checks
      final float spillper =
        job.getFloat(JobContext.MAP_SORT_SPILL_PERCENT, (float)0.8);
      final int sortmb = job.getInt(JobContext.IO_SORT_MB, 100);
      indexCacheMemoryLimit = job.getInt(JobContext.INDEX_CACHE_MEMORY_LIMIT,
                                         INDEX_CACHE_MEMORY_LIMIT_DEFAULT);
      if (spillper > (float)1.0 || spillper <= (float)0.0) {
        throw new IOException("Invalid \"" + JobContext.MAP_SORT_SPILL_PERCENT +
            "\": " + spillper);
      }
      if ((sortmb & 0x7FF) != sortmb) {
        throw new IOException(
            "Invalid \"" + JobContext.IO_SORT_MB + "\": " + sortmb);
      }
      sorter = ReflectionUtils.newInstance(job.getClass("map.sort.class",
            QuickSort.class, IndexedSorter.class), job);
      // buffers and accounting
      int maxMemUsage = sortmb << 20;
      maxMemUsage -= maxMemUsage % METASIZE;
      kvbuffer = new byte[maxMemUsage];
      bufvoid = kvbuffer.length;
      kvmeta = ByteBuffer.wrap(kvbuffer)
         .order(ByteOrder.nativeOrder())
         .asIntBuffer();
      setEquator(0);
      bufstart = bufend = bufindex = equator;
      kvstart = kvend = kvindex;

      maxRec = kvmeta.capacity() / NMETA;
      softLimit = (int)(kvbuffer.length * spillper);
      bufferRemaining = softLimit;
      LOG.info(JobContext.IO_SORT_MB + ": " + sortmb);
      LOG.info("soft limit at " + softLimit);
      LOG.info("bufstart = " + bufstart + "; bufvoid = " + bufvoid);
      LOG.info("kvstart = " + kvstart + "; length = " + maxRec);

      // k/v serialization
      comparator = job.getOutputKeyComparator();
      keyClass = (Class<K>)job.getMapOutputKeyClass();
      valClass = (Class<V>)job.getMapOutputValueClass();
      serializationFactory = new SerializationFactory(job);
      keySerializer = serializationFactory.getSerializer(keyClass);
      keySerializer.open(bb);
      valSerializer = serializationFactory.getSerializer(valClass);
      valSerializer.open(bb);

      // output counters
      mapOutputByteCounter = reporter.getCounter(TaskCounter.MAP_OUTPUT_BYTES);
      mapOutputRecordCounter =
        reporter.getCounter(TaskCounter.MAP_OUTPUT_RECORDS);
      fileOutputByteCounter = reporter
          .getCounter(TaskCounter.MAP_OUTPUT_MATERIALIZED_BYTES);

      // compression
      if (job.getCompressMapOutput()) {
        Class<? extends CompressionCodec> codecClass =
          job.getMapOutputCompressorClass(DefaultCodec.class);
        codec = ReflectionUtils.newInstance(codecClass, job);
      } else {
        codec = null;
      }

      // combiner
      final Counters.Counter combineInputCounter =
        reporter.getCounter(TaskCounter.COMBINE_INPUT_RECORDS);
      combinerRunner = CombinerRunner.create(job, getTaskID(), 
                                             combineInputCounter,
                                             reporter, null);
      if (combinerRunner != null) {
        final Counters.Counter combineOutputCounter =
          reporter.getCounter(TaskCounter.COMBINE_OUTPUT_RECORDS);
        combineCollector= new CombineOutputCollector<K,V>(combineOutputCounter, reporter, job);
      } else {
        combineCollector = null;
      }
      spillInProgress = false;
      minSpillsForCombine = job.getInt(JobContext.MAP_COMBINE_MIN_SPILLS, 3);
      spillThread.setDaemon(true);
      spillThread.setName("SpillThread");
      spillLock.lock();
      try {
        spillThread.start();
        while (!spillThreadRunning) {
          spillDone.await();
        }
      } catch (InterruptedException e) {
        throw new IOException("Spill thread failed to initialize", e);
      } finally {
        spillLock.unlock();
      }
      if (sortSpillException != null) {
        throw new IOException("Spill thread failed to initialize",
            sortSpillException);
      }
    }
```

#### Map-output 的工作

1. 将 kv 转为 kvp（partition）
2. 将 kvp 放入内存缓冲区(环形缓冲区)。
3. 若达到内存缓冲区阈值，则会先做排序（QS快排），若有 combiner，则进行combiner
4. 将内存缓冲区写出为小文件，并清空缓冲区之前写出的 kvp
5. 当数据全部处理完毕后，将之前产生的小文件合并成大文件

注意：排序不对原kvp排序，只对kvp 对应的元数据排序

### Reduce

#### ReduceTask.java

#### ReduceContextImpl.java

```java
 /** Start processing next unique key. */
  public boolean nextKey() throws IOException,InterruptedException {
    while (hasMore && nextKeyIsSame) {
      nextKeyValue();
    }
    if (hasMore) {
      if (inputKeyCounter != null) {
        inputKeyCounter.increment(1);
      }
      return nextKeyValue();
    } else {
      return false;
    }
  }

@Override
  public boolean nextKeyValue() throws IOException, InterruptedException {
    if (!hasMore) {
      key = null;
      value = null;
      return false;
    }
    firstValue = !nextKeyIsSame;
    DataInputBuffer nextKey = input.getKey();
    currentRawKey.set(nextKey.getData(), nextKey.getPosition(), 
                      nextKey.getLength() - nextKey.getPosition());
    buffer.reset(currentRawKey.getBytes(), 0, currentRawKey.getLength());
    key = keyDeserializer.deserialize(key);
    DataInputBuffer nextVal = input.getValue();
    buffer.reset(nextVal.getData(), nextVal.getPosition(), nextVal.getLength()
        - nextVal.getPosition());
    value = valueDeserializer.deserialize(value);

    currentKeyLength = nextKey.getLength() - nextKey.getPosition();
    currentValueLength = nextVal.getLength() - nextVal.getPosition();

    if (isMarked) {
      backupStore.write(nextKey, nextVal);
    }

    hasMore = input.next();
    if (hasMore) {
      nextKey = input.getKey();
      nextKeyIsSame = comparator.compare(currentRawKey.getBytes(), 0, 
                                     currentRawKey.getLength(),
                                     nextKey.getData(),
                                     nextKey.getPosition(),
                                     nextKey.getLength() - nextKey.getPosition()
                                         ) == 0;
    } else {
      nextKeyIsSame = false;
    }
    inputValueCounter.increment(1);
    return true;
  }

```



#### 分组比较器的获取顺序

1. 从用户配置那，获取分组比较器结束。若没有，走 第二步。
2. 从 Mapper 端，获取用户是否设置排序比较器。若没有，则采用默认 key 比较器

<font color='#ff0033'>注意： mapper 端和 reducer 端的比较器可以相同，也可以不相同</font>

#### <font color='green'>reduce 方法迭代的原理</font>*

真迭代器：存储从 Mapper  获取全部 keyvalue 数据

假迭代器：存储一组 key 对应的 values ，符合<font color='#00FF33'>原语</font>

<font color='red'>nextKeyIsSame</font>

## <font color='blue'>MapReduce 案例</font>

### 天气

- 需求

  找出每个月气温最高的 2 天

- 实现

1. 客户端代码

```java
/**
 *   客户端配置
 */
public class MyWeather {
	public static void main(String[] args) throws Exception {
		
		Configuration conf = new Configuration(true);
		Job job = Job.getInstance(conf);
		
		job.setJarByClass(MyWeather.class);
		job.setJobName("MyWeather");
		
		/*************conf******************/
		Path input = new Path("/data/weather/weather.txt");
		FileInputFormat.addInputPath(job, input);
		Path output = new Path("/data/tq/out");
		if(output.getFileSystem(conf).exists(output)){
			output.getFileSystem(conf).delete(output, true);
		}
		FileOutputFormat.setOutputPath(job, output );
		
		//-------map --START
		job.setMapperClass(TMapper.class);
		job.setMapOutputKeyClass(TQ.class); //年月
		job.setMapOutputValueClass(IntWritable.class);// 气温
		
		job.setPartitionerClass(TPartitioner.class);
		job.setSortComparatorClass(TSortComparator.class);
//		job.setCombinerKeyGroupingComparatorClass(TComibner.class);
		//-------map --END
		
		job.setGroupingComparatorClass(TGroupingComparator.class);
		job.setReducerClass(TReducer.class);
		
		job.setNumReduceTasks(2); // 设置 Reduce 的数量
		/*************conf END******************/
		
		job.waitForCompletion(true);
		
	}
}

```

2. Mapper 代码

```java
/**
 * mapper 映射 kv
 */
public class TMapper extends Mapper<LongWritable, Text, TQ, IntWritable>{
	
	TQ weather = new TQ();  // keyout
	IntWritable temperature = new IntWritable(); //valueout 
	
	@Override
	protected void map(LongWritable key, Text value, Context context)
			throws IOException, InterruptedException {
					
			Date date = null;
			try {
				// 1949-10-02 14:01:02	36c
				String[] str = StringUtils.split(value.toString(),'\t');
				SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
				date = sdf.parse(str[0]);
				Calendar calendar = Calendar.getInstance();
				calendar.setTime(date);
				// 设置  keyout
				this.weather.setYear(calendar.get(Calendar.YEAR));
				this.weather.setMonth(calendar.get(Calendar.MONTH)+1); // 月份+1，
				this.weather.setDay(calendar.get(calendar.DAY_OF_MONTH));
				// 设置 valueout
				int temperatureValue = Integer.valueOf(str[1].substring(0, str[1].length()-1));
				this.weather.setTemperature(temperatureValue);
				this.temperature.set(temperatureValue);
				// 输出
				context.write(weather, temperature);
			} catch (ParseException e) {
				e.printStackTrace();
			}
	}
}

```

3. Redcer 代码

```java

public class TReducer extends Reducer<TQ, IntWritable, Text, IntWritable> {

	Text rkey = new Text();
	IntWritable rvalue = new IntWritable();

	@Override
	protected void reduce(TQ key, Iterable<IntWritable> values, Context context)
			throws IOException, InterruptedException {
		int flag = 0;
		int day = 0;
		// 相同的 key 为一组，调用一次 reduce 方法，方法内迭代这一组数据
		for (IntWritable val : values) {
			// 1949-10-11 34 20
			// 1949-10-12 33 21
			// 1949-10-14 20 20
			if (flag == 0) {
				StringBuilder builder = new StringBuilder();
				builder.append(key.getYear());
				builder.append("-");
				builder.append(key.getMonth());
				builder.append("-");
				builder.append(key.getDay());
				flag++;
				day = key.getDay();
				rkey.set(builder.toString());
				rvalue.set(val.get());
				context.write(rkey, rvalue);
			}
			
			if(flag != 0 && day != key.getDay()){
				StringBuilder builder = new StringBuilder();
				builder.append(key.getYear());
				builder.append("-");
				builder.append(key.getMonth());
				builder.append("-");
				builder.append(key.getDay());
				rkey.set(builder.toString());
				rvalue.set(val.get());
				context.write(rkey, rvalue);
				break;
			}
		}
	}
}

```

### 好友推荐

- 需求

  为 hadoop 进行好友推荐

- 好友数据分析
  - 好友数据

    ```
    tom hello hadoop cat
    world hadoop hello hive
    cat tom hive
    mr hive hello
    hive cat hadoop world hello mr
    hadoop tom hive world
    hello tom world hive mr
    ```

  - 分析

    每行的首个用户与其所在行的其他用户是直接好友关系。

    而每个用户的好友之间则存在间接好友关系或者直接好友关系。

    这样通过每个用户的好友之间的关系总和除去直接好友关系，剩下的就是间接好友关系。

    由间接好友关系的数量，就能推出共同好友的数量，由此做好友推荐的功能。

    

- 实现

  Client 程序

```java
/**
 * 好友推荐客户端 
 */
public class MyRecommendedFriend {
	
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		
		job.setJarByClass(MyRecommendedFriend.class);
		job.setJobName("MyRecommendedFriend");
		
		/********** conf-START *****************/
		// input and output
		Path input = new Path("/data/recommendedfriend/input/recommendedfriend.txt");
		FileInputFormat.addInputPath(job, input);
		Path output = new Path("/data/recommendedfriend/output");
		if(output.getFileSystem(conf).exists(output)){
			output.getFileSystem(conf).delete(output, true);
		}
		FileOutputFormat.setOutputPath(job, output );
		// mapper
		job.setMapperClass(RecommendedFriendMapper.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		// 采用默认的分区器  hash
		// 采用默认的排序器 
		
		// reducer
		job.setReducerClass(RecommendedFriendReducer.class);
		// 采用默认的分组器
		
		/********** conf-END *****************/
		
		job.waitForCompletion(true);
	}
	
}

```

​	Mapper 程序

```java
/*
 *  mapper  读取每条记录，从中找去直接好友关系和间接好友关系
 *  key 的格式为 用户名:用户名（用户名的先后按字典序排列）
 *  value 0 或者 1  （直接好友关系为 0 ，间接好友关系为 1）
 */
public class RecommendedFriendMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	
	Text mkey = new Text();
	IntWritable mvalue = new IntWritable(); // 直接好友关系为 0 ，间接好友关系为 1
	
	@Override
	protected void map(LongWritable key, Text value, Context context)
			throws IOException, InterruptedException {
			// tom hello hadoop cat
			String[] strs = StringUtils.split(value.toString(), ' ');
			for (int i = 1; i < strs.length; i++) {
				// strs[0]+strs[i]
				mkey.set(this.getRecommendedFriendKeyValue(strs[0],strs[i]));
				mvalue.set(0); //直接好友
				context.write(mkey, mvalue);
				for (int j = i+1; j < strs.length; j++) {
					mkey.set(this.getRecommendedFriendKeyValue(strs[i],strs[j]));
					mvalue.set(1); //间接好友
					context.write(mkey, mvalue);
				}
			}
	}
	
	// 按照字典序排列两个好友的先后顺序
	private String getRecommendedFriendKeyValue(String f1, String f2){
		if(f1.compareTo(f2) < 0 ){
			return f2+":"+f1;
		} 
		return f1+":"+f2;
	}
}

```

​	Reducer 程序

```java
/*
 *  Reducer
 *  读取一组相同的 key 
 *  根据 value 的值判断，若 0 则直接跳出，若 1 则统计次数
 */
public class RecommendedFriendReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

	IntWritable rvalue = new IntWritable();

	@Override
	protected void reduce(Text key, Iterable<IntWritable> values, Context context)
			throws IOException, InterruptedException {
		// hadoop:hello 1
		// hadoop:hello 1
		// tom:hello 0
		boolean flag = false;
		int sum = 0;
		for (IntWritable val : values) {

			if (val.get() == 0) {
				flag = true;
				return;
			}

			sum += val.get();
		}

		if (!flag) {
			rvalue.set(sum);
			context.write(key, rvalue);
		}
	}
}
```

<font color='pinkblanck'>运行结果文件内容</font>

```
hadoop:cat	2
hello:cat	2
hello:hadoop	3
mr:cat	1
mr:hadoop	1
tom:hive	3
tom:mr	1
world:cat	1
world:mr	2
world:tom	2
```

### <font color='green'>PageRank*</font>

> - PageRank是Google提出的算法，用于衡量特定网页相对于<font color='red'>搜索引擎</font>索引中的其他网页而言的重要程度。
> - 是Google创始人拉里·佩奇和谢尔盖·布林于1997年创造的
> - PageRank实现了将链接价值概念作为排名因素。

- 算法原理

  入链 ==== 投票
  PageRank让链接来“投票“，到一个页面的超链接相当于对该页投一票。
  入链数量
  如果一个页面节点接收到的其他网页指向的入链数量越多，那么这个页面越重要。
  入链质量
  指向页面A的入链质量不同，质量高的页面会通过链接向其他页面传递更多的权重。所以越是质量高的页面指向页面A，则页面A越重要。

- 算法原理2 

  初始值
  Google的每个页面设置相同的PR值
  pagerank算法给每个页面的PR初始值为1。
  迭代计算（收敛）
  Google不断的重复计算每个页面的PageRank。那么经过不断的重复计算，这些页面的PR值会趋向于稳定，也就是收敛的状态。
  在具体企业应用中怎么样确定收敛标准？
  1、每个页面的PR值和上一次计算的PR相等
  2、设定一个差值指标（0.0001）。当所有页面和上一次计算的PR差值平均小于该标准时，则收敛。
  3、设定一个百分比（99%），当99%的页面和上一次计算的PR相等

- 算法原理3

  站在互联网的角度：
  只出，不入：PR会为0
  只入，不出：PR会很高
  直接访问网页
  修正PageRank计算公式：增加阻尼系数
  在简单公式的基础上增加了阻尼系数（damping factor）d
  一般取值d=0.85。
  完整PageRank计算公式
  $d$：阻尼系数
  $M(i)$：指向i的页面集合
  $L(j)$：页面的出链数
  $PR(p_j)$：j页面的PR值
  $n$：所有页面数
  $$
  PR(p_i) = \frac{1 - d}{n}+d\sum\limits_{p_j\in M(i)}\frac{PR(p_j)}{L(j)}
  $$
  
- 网络上各个页面的链接图

  ![](http://img.zwer.xyz/blog/20191012101131.png)

- PR 计算
  PR需要迭代计算 其PR值会趋于稳定

- 实现思路

  ```
  解需求思路
  **MR原语不被破坏
  PR计算是一个迭代的过程，首先考虑一次计算
  思考：
  页面包含超链接
  每次迭代将pr值除以链接数后得到的值传递给所链接的页面
  so：每次迭代都要包含页面链接关系和该页面的pr值
  mr：相同的key为一组的特征
  map：
  1，读懂数据：第一次附加初始pr值
  2，映射k:v
  1，传递页面链接关系，key为该页面，value为页面链接关系
  2，计算链接的pr值，key为所链接的页面，value为pr值
  reduce：
  *，按页面分组
  1，两类value分别处理
  2，最终合并为一条数据输出：key为页面&新的pr值，value为链接关系
  ```

  


### <font color='green'>TFIDF*</font>

> TF-IDF（term frequency–inverse document frequency）
>
> 是一种用于资讯检索与资讯探勘的常用加权技术。
>
> TF-IDF是一种统计方法，用以评估一字词对于一个文件集或一个语料库中的其中一份文件的重要程度。
> <font color='pingkbrown'>字词的重要性随着它在文件中出现的次数成正比增加
> 但同时会随着它在语料库中出现的频率成反比下降</font>
> TF-IDF加权的各种形式常被搜寻引擎应用
> 作为文件与用户查询之间相关程度的度量或评级。
> 除了TF-IDF以外，
>
> 因特网上的搜寻引擎还会使用**基于链接分析的评级方法**，以确定文件在搜寻结果中出现的顺序：PR。

- 解释

  用户通过调整字词来缩小范围
  每个字词都有对应出现的页面
  通过字词数量缩小范围
  最终通过字词对于页面的权重来进行排序

- 词频 (term frequency, TF) 指的是**某一个给定的词语在一份给定的文件中出现的次数**。这个数字通常会被归一化（分子一般小于分母 区别于IDF），以防止它偏向长的文件。（同一个词语在长文件里可能会比短文件有更高的词频，而不管该词语重要与否。）
  公式中：
  $n_i,_j$是该词在文件 $d_j$ 中的出现次数，而分母则是在文件 $d_j$ 中所有字词的出现次数之和。

$$
tf_i,_j = \frac{n_i,_j}{\sum_k {n_k,_j}}
$$

- 逆向文件频率 (inverse document frequency, IDF) 是一个词语普遍重要性的度量。

  某一特定词语的IDF，可以由总文件数目除以包含该词语之文件的数目，再将得到的商取对数得到。
  $|D|$：语料库中的文件总数
  $|j:t_i \in d_j|$ 包含 $t_i$ 文件的数目
  $$
  idf_i = log\frac{|D|}{|\{j:t_i \in d_j\}|}
  $$
  
- TF-IDF

    $$
    tf_idf_i,_j = tf_i,_j   * f_i
    $$

    某一特定文件内的高词语频率，以及该词语在整个文件集合中的低文件频率，可以产生出高权重的TF-IDF。

    因此，TF-IDF倾向于过滤掉常见的词语，保留重要的词语。

    TFIDF的主要思想是：

    **如果某个词或短语在一篇文章中出现的频率 TF 高，并且在其他文章中很少出现，则认为此词或者短语具有很好的类别区分能力，适合用来分类。**

- MR

  ```
  第一次：词频统计+文本总数统计
  map：
  词频：key：字词+文本，value：1
  文本总数：key：count，value：1
  partition：4个reduce
  0~2号reduce并行计算词频
  3号reduce计算文本总数
  reduce：
  0~2：sum
  3：count：sum
  第二次：字词集合统计：逆向文件频率
  map：
  key：字词，value：1
  reduce：
  sum
  第三次：取1，2次结果最终计算出字词的 TF-IDF
  map：输入数据为第一步的 tf
  setup：加载：a，DF；b，文本总数
  计算TF-IDF
  key：文本，value：字词+TF-IDF
  reduce：
  按文本（key）生成该文本的字词+TF-IDF值列表
  ```

  

### <font color='#cc3333'>ItemCF*</font>

> 用户商品推荐

- 推荐系统——协同过滤(Collaborative Filtering)算法

  1. UserCF
     基于用户的协同过滤，通过不同用户对物品的评分来评测用户之间的相似性

     基于用户之间的相似性做出推荐

     简单来讲就是：给用户推荐和他兴趣相似的其他用户喜欢的物品。
     
     <img src="http://img.zwer.xyz/blog/20191012170254.png" style="zoom:80%;" />
     
  2. **ItemCF**
  
     基于item的协同过滤，通过用户对不同item的评分来评测item之间的相似性
  
     基于item之间的相似性做出推荐。
  
     简单来讲就是：给用户推荐和他之前喜欢的物品相似的物品。
  
     <img src="http://img.zwer.xyz/blog/20191012170539.png" style="zoom: 80%;" />



- **Co-occurrence Matrix(同现矩阵)和User Preference Vector(用户评分向量)相乘得到的这个Recommended Vector(推荐向量)**
  基于全量数据的统计，产生同现矩阵
  体现商品间的关联性
  每件商品都有自己对其他全部商品的关联性（每件商品的特征）
  用户评分向量体现的是用户对一些商品的评分
  任一商品需要：
  用户评分向量乘以基于该商品的其他商品关联值
  求和得出针对该商品的推荐向量
  排序取TopN即可

  ![](http://img.zwer.xyz/blog/20191012170741.png)

- MR 实现

```
去除重复数据
计算用户评分向量
计算同现矩阵
计算乘机
计算求和
计算取TopN
```



### 正则使用

1. 按照制表符和逗号切割给定的字符串

```java
@Test
public void testSplit(){
    String test = "u12	i123,11";
    String[] strs = test.split("[\t,]");
    for (String str : strs) {
   		 System.out.println(str);
	}
}
```

2. 按多个空格作为切割条件

```java
String[] str2 = str[1].split("\\s+");
```

