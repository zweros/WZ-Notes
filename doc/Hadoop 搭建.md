## 搭建 Hadoop 伪分布式 ##

### 准备 ###

#### 时间同步 ####

```shell
type ntpdate
# 安装 ntpdate
yum install -y ntpdate
# 同步网络时
ntpdate -u ntp.api.bz
```

#### 操作系统环境设置 ####

- 确认主机名一致

![](http://img.zwer.xyz/blog/20190930112247.png)

- ssh 免密钥

```shell
# 在另外一台机器上远程执行 192.168.170.101 机器上的命令(验证)
ssh root@192.168.170.101 'ls ~' 

$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa  # 生成密钥
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys  # 对自己免密钥
```

- 安装 Java 并配置环境变量

```shell
rpm -ivh jdk-7u67-linux-x64.rpm
# 修改 /etc/profile
export JAVA_HOME=/usr/java/jdk1.7.0_67
export PATH=$PATH:$JAVA_HOME
```

#### Hadoop 配置 ####

- 安装 Hadoop 并配置环境变量

```shell
tar xf hadoop-2.6.5.tar.gz 
mkdir /opt/sxt
mv hadoop-2.6.5 /opt/sxt/

#  配置 hadoop 环境变量
export HADOOP_HOME=/opt/sxt/hadoop-2.6.5
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

# cd /opt/sxt/hadoop-2.6.5/
# 进入 etc 目录中
# 修改 Java 环境路径，export JAVA_HOME=/usr/java/jdk1.7.0_67
# 目的是 hadoop 管理脚本操作机器不能读取 /etc/profile 文件，所以需要改 Java 的绝对路径
vi hadoop-env.sh
vi mapred-env.sh
vi yarn-env.sh
```

- 修改  slaves

```
将 localhsot 改为 node01
node01
```

- 修改 core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node01:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/var/sxt/hadoop/local</value>
    </property>
</configuration>
```

- 修改 hdfs-site.xml

```xml
<configuration>
     <property>
        <name>dfs.replication</name>
        <value>1</value>
     </property>
     <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node01:50090</value>
    </property>
</configuration>
```

| 配置文件                 | 作用                      |
| ------------------------ | ------------------------- |
| etc/hadoop/core-site.xml | 决定 NameNode 的启动      |
| etc/hadoop/slaves        | 决定 DataNode 的启动      |
| etc/hadoop/hdfs-site.xml | 决定 SecondaryNode 的启动 |

### 启动 ###

```shell
# 格式化
$hdfs namenode -format

$ls /var/sxt/hodoop/local/dfs/name/current/
fsimage_0000000000000000000  fsimage_0000000000000000000.md5  seen_txid  VERSION

# 启动  NN、DN、SN
$start-dfs.sh
# jps 查看 Java 进程，是否有 NN、DN、SN
$jps
3385 DataNode
3543 SecondaryNameNode
3674 Jps
3306 NameNode

#访问 http://192.168.170.101:50070
```

### 使用 ###

```shell
hdfs dfs -mkdir /user
hdfs dfs -ls /user
hdfs dfs -mkdir /user/root
hdfs dfs -D dfs.blocksize=1048576 -put hadoop-2.6.5.tar.gz 
# 展示当前目录，并显示文件大小 
ll -h
```

## 搭建 Hadoop 全分布式 ##

### 准备 ###

| HOST   | NN   | SNN  | DN   |
| ------ | ---- | ---- | ---- |
| node01 | *    |      |      |
| node02 |      | *    | *    |
| node03 |      |      | *    |
| node04 |      |      | *    |

### 搭建 ###

```shell
# 关闭 Hadoop 伪分布式
stop-dfs.sh

# 配置节点 node02、node03、node04 的 java 环境 和 ssh
cat ~/node01.pub  >> ~/.ssh/authorized_keys
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

# 以下操作针对 node01 ---------------------------------------------
# 修改 Hadoop 配置文件
# 编辑 etc/core-site.xml 文件
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node01:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/var/sxt/hadoop/full</value>
    </property>
</configuration>
# 编辑 etc/hdfs-site.xml 文件
<configuration>
     <property>
        <name>dfs.replication</name>
        <value>2</value>
     </property>
     <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node02:50090</value>
    </property>
</configuration>
# 编辑 etc/slaves 文件
node02
node03
node04
# ------------------------------------------------------------------
# 在 node02、node03、node04 下创建文件夹
mkdir /opt/sxt
# 在 node01 节点的 /opt/sxt 目录下执行
scp -r ./hadoop-2.6.5/ node02:`pwd`
scp -r ./hadoop-2.6.5/ node03:`pwd`
scp -r ./hadoop-2.6.5/ node04:`pwd`

# 在 node01 节点上执行  ---------------------------
hdfs namenode -format
start-dfd.sh
```

在 node01 节点上执行 格式化

![启动 hdfsnamenode -format 截图](http://img.zwer.xyz/blog/20190930163010.png)

在 node01 节点启动 hadoop 

![](http://img.zwer.xyz/blog/20190930163212.png)

## Hadoop 2.x ##

![](http://img.zwer.xyz/blog/20191008194300.png)

- 主备 NameNode
  解决单点故障（属性，位置）
  主NameNode对外提供服务，备NameNode同步主NameNode元数据，以待切换
  所有DataNode同时向两个NameNode汇报数据块信息（位置）
- JNN:集群（属性）
- standby：备，完成了 edits.log 文件的合并产生新的image，推送回 ANN
  两种切换选择
  手动切换：通过命令实现主备之间的切换，可以用HDFS升级等场合
  自动切换：基于Zookeeper实现
  基于Zookeeper自动切换方案
- ZooKeeper Failover Controller：监控NameNode健康状态，并向Zookeeper注册NameNode
  NameNode挂掉后，ZKFC为NameNode竞争锁，获得ZKFC 锁的NameNode变为active



### Hadoop 2.x 联邦 ###

> 联邦：不同应用间是相互隔离

1. 通过多个namenode/namespace把元数据的存储和管理分散到多个节点中，使到namenode/namespace可以通过增加机器来进行水平扩展。
2. 能把单个 namenode 的负载分散到多个节点中，在 HDFS 数据规模较大的时候不会也降低 HDFS 的性能。可以通过多个 namespace 来隔离不同类型的应用，把不同类型应用的HDFS元数据的存储和管理分派到不同的namenode中。

![](http://img.zwer.xyz/blog/20191008194818.png)



## HA 高可用 ##

### 准备 ###

|        | NN-1 | NN-2 | DN   | ZK   | ZKFC | JNN  |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- |
| node01 | *    |      |      |      | *    | *    |
| node02 |      | *    | *    | *    | *    | *    |
| node03 |      |      | *    | *    |      | *    |
| node04 |      |      | *    | *    |      |      |



### 搭建 Zookeeper 集群 ###

```shell
# 解压 Zookeeper 压缩包，并移动到 /opt/sxt 目录下
# 配置 Zookeeper 环境变量
# 到 zookeeper 目录下，进入 conf 目录下
# 复制 zoo_simple.cfg 到 zoo.cfg
# 修改 zoo.cfg 配置文件
dataDir=/var/sxt/hadoop/zk
clientPort=2181
# 配置 zk 集群，第一次启动按 serverid 区分 leader 和  fellower
server.1=192.168.170.102:2888:3888
server.2=192.168.170.103:2888:3888
server.3=192.168.170.104:2888:3888

#创建  /var/sxt/hadoop/zk 目录，将当前机器 zk 的 id 写入 myid 文件中
mkdir /var/sxt/hadoop/zk -p
echo "1">/var/sxt/hadoop/zk/myid
 
# 分发 zookeeper 目录到其他机器上(node03、node04)
scp -r ./zookeeper-3.4.6/ root@node03:`pwd`
scp -r ./zookeeper-3.4.6/ root@node04:`pwd`

# 在 node03、node04 机器上
# 1. 配置 zookeeper 的环境变量 
# 2. 增加 myid 文件，将当前机器 zk 的 serverid 写入
```

### HA 搭建 ###

> 1. 逻辑到物理的映射
> 2. JNode 位置信息相关配置
> 3. 出现故障的处理方式及 SSH 免密钥配置

#### hdfs-site.xml 

```xml
<property>
    <name>dfs.nameservices</name>
    <value>mycluster</value>
</property>
<property>
    <name>dfs.ha.namenodes.mycluster</name>
    <value>nn1,nn2</value>
</property>
<property>
    <name>dfs.namenode.rpc-address.mycluster.nn1</name>
    <value>node01:8020</value>
</property>
<property>
    <name>dfs.namenode.rpc-address.mycluster.nn2</name>
    <value>node02:8020</value>
</property>
<property>
    <name>dfs.namenode.http-address.mycluster.nn1</name>
    <value>node01:50070</value>
</property>
<property>
    <name>dfs.namenode.http-address.mycluster.nn2</name>
    <value>node02:50070</value>
</property>
<property>
  <name>dfs.namenode.shared.edits.dir</name>
  <value>qjournal://node01:8485;node02:8485;node03:8485/mycluster</value>
</property>

<property>
  <name>dfs.journalnode.edits.dir</name>
  <value>/var/sxt/hadoop/ha/jn</value>
</property>

<property>
  <name>dfs.client.failover.proxy.provider.mycluster</name>
  <value>
      org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider
    </value>
</property>
<property>
  <name>dfs.ha.fencing.methods</name>
  <value>sshfence</value>
</property>
<property>
  <name>dfs.ha.fencing.ssh.private-key-files</name>
  <value>/root/.ssh/id_dsa</value>
</property>

<property>
   <name>dfs.ha.automatic-failover.enabled</name>
   <value>true</value>
 </property>
```

### core-site.xml

>注意：hadoop.tmp.dir的配置要变更：/var/sxt/hadoop-2.6/ha ###

```xml
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://mycluster</value>
</property>
<property>
   <name>ha.zookeeper.quorum</name>
   <value>node02:2181,node03:2181,node04:2181</value>
 </property>
```

### HA 部署 ###

```shell
# 第一次部署
# 1.启动 node01、node02、node03 的 journalnode
hadoop-daemon.sh start journalnode
# 2.格式化 node01 dfs(仅第一次)
hdfs namenode –format
# 3.启动 node01 的 NN ，启动 node02 的 standyNN
hadoop-daemon.sh start namenode   # 在 node01
hdfs namenode -bootstrapStandby   # 在 node02
# 4.格式化 zk 
hdfs zkfc -formatZK
# 5.启动 hdfs
start-dfs.sh 
#测试 
kill -9 进程号    强制清除

# 第二次启动 
1，启动zk
2，start-dfs.sh
```

### SSH 免密钥的应用场景 ###

```shell
1. 管理脚本管理服务的开启与关闭
2. 搭建 HA 时，ZKFC 需要控制自己及对方

ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa  # 生成密钥
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys  # 对自己免密钥
```

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



## HDFS YARN 

前提： 不是第一次安装，也不是一次启动

在启动 Hadoop 集群之前，先要启动 ZK 集群，用于  NN 高可用，分布式协调

```shell
start-all.sh  # 启动 HDFS  和 YARN  
# 手动在节点中启动 ResourceManager 
yarn-daemon.sh start resourcemanager 
start—dfs.sh   # 启动 HDFS 
start-yarn.sh  # 启动 YARN 资源管理

hadoop-daemon.sh [start|stop]  [namenode | atanode]

yarn-daemon.sh [start|stop]  [resourcemanager | nodemanager] 
```

