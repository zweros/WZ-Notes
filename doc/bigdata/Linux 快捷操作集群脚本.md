## 前言

OS:  CentOS 6.8 

## ssh 命令

>  ssh 命令具有用来登录远程服务器、执行服务器命令的工具等功能，这里主要介绍 ssh 命令执行远程服务器命令的功能

- ssh 执行远程服务器命令功能使用格式

  ```ssh
  [sudo] ssh 远程服务器地址（IP 地址或者主机名） "远程服务器需要执行的命令"
  ```

- ssh 命令使用示例

  当前登录机器的主机名 hadoop102，并且使用的用户 hadoop ，并不是 root 用户。下面执行 ssh 命令的功能是在机器的主机名为 hadoop103 上执行 jps 命令，将执行结果返回给 hadoop102  的控制台

  ```
  [hadoop@hadoop102 ~]$ ssh hadoop103 jps
  6147 Jps
  1587 NodeManager
  4293 QuorumPeerMain
  1879 ResourceManager
  1497 DataNode
  ```

  说明：由于机器 hadoop102 与  hadoop103 之前做过免密钥，所以这里执行 ssh 命令后没有再要求输入机器 hadoop103 的登录密码

## ssh 脚本

> 说明：以下脚本存放的位置在 /home/hadoop/bin 目录下，便于 hadoop 用户在全局执行，而不用在脚本所在位置执行或者执行使用脚本的全路径

### 多台机器上执行相同的命令脚本

```shell
#!/bin/bash
# 在多台服务器上执行相同的命令
for i in hadoop102 hadoop103 hadoop104
do
   #  打印对应的服务器主机名
   echo "----$i-------" 
   ssh $i "$*"
done
```

### 操作 zk 集群脚本

```shell
#!/bin/bash

case $1 in
"start"){
    for i in hadoop102 hadoop103 hadoop104
    do
       ssh  $i "/opt/module/zookeeper-3.4.6/bin/zkServer.sh start"
    done
};;
"status"){
    for i in hadoop102 hadoop103 hadoop104
    do
       ssh  $i "/opt/module/zookeeper-3.4.6/bin/zkServer.sh status"
    done
};;
"stop"){
    for i in hadoop102 hadoop103 hadoop104
    do
       ssh  $i "/opt/module/zookeeper-3.4.6/bin/zkServer.sh stop"
    done
};;
esac
```

## 文件传输脚本

```shell
#!/bin/bash
#1 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
echo no args;
exit;
fi

#2 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname

#3 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir

#4 获取当前用户名称
user=`whoami`

#5 循环
for((host=103; host<105; host++)); do
        echo ------------------- hadoop$host --------------
        rsync -rvl $pdir/$fname $user@hadoop$host:$pdir
done
```

## Flume 脚本

```shell
#!/bin/bash

case $1 in
"start"){
        for i in hadoop102 hadoop103
        do
                echo "-----启动 $i 日志采集-----"
                ssh $i "nohup /opt/module/flume/bin/flume-ng agent --conf-file /opt/module/flume/conf/file-flume-kafka.conf --name a1 -Dflume.root.logger=INFO,LOGFILE >/dev/null 2>&1 &"
        done
};;
"stop"){
        for i in hadoop102 hadoop103
        do
                echo "-----关闭 $i 日志采集-----"
                ssh $i "ps -ef | grep file-flume-kafka | grep -v grep | awk '{print \$2}' | xargs kill"
        done
};;
esac
```

说明： `nohup` 命令的作用是执行程序并且常驻后台进行，即不会因为控制台关闭导致程序运行结束

## Kakfa 集群脚本

> 在同一台机器上操作多台器上的 Kafka 启动与停止

```shell
#!/bin/bash

case $1 in
"start"){
     for i in hadoop102 hadoop103 hadoop104
     do
        echo "----启动  $i kafka"
        ssh $i "export JMX_PORT=9988 && /opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties"
     done
};;
"stop"){
     for i in hadoop102 hadoop103 hadoop104
     do
        echo "----停止  $i kafka"
        ssh $i "/opt/module/kafka/bin/kafka-server-stop.sh stop"
     done
};;
esac
```

### hadoop 集群脚本

> 在同一台机器上操作多台机器上 Hadoop、zookeeper、flume、kafka 启动与停止

```shell
#!/bin/bash

case $1 in
"start"){
    echo ---------启动 hadoop 集群-----------
    start-dfs.sh
    ssh hadoop103 "start-yarn.sh"

   zk.sh start
    sleep 4s
    # 启动 Flume 采集集群
    f1.sh start
    kf.sh start
    sleep 6s
    # 启动 hadoop 104 上日志采集
    f2.sh start
};;
"stop"){
    # 停止 hadoop 104 上日志采集
    f2.sh stop
    kf.sh stop
    sleep 6s
    # 停止 Flume 采集集群
    f1.sh stop
    zk.sh stop
    echo ---------停止 hadoop 集群-----------
    ssh hadoop103 "stop-yarn.sh"
    stop-dfs.sh
};;
esac
```



