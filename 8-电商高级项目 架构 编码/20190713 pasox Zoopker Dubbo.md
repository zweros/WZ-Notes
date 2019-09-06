## Paxos  数据一致性算法

> 用于分布式系统的 ZooKeeper 数据一致性算法

leader 

follower

## Zookeeper 介绍

> 分布式的、开放源码的分布式程序协调服务
>
> 功能：1. 配置维护、分布式通信、域名服务、组服务等

| 功能       | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| 配置维护   | 在一个集群系统中，每个集群节点有大量的配置文件，每个节点都去单独配置单独维护是不可行的，利用 Zookeeper 将所有**共性的配置文件**进行统一配置，统一维护。 |
| 分布式通信 | 实现生产者和消费者之间的**服务发现**                         |

### 集群搭建

将 Zookeeper 压缩包解压到   /user/local/zookeeperCluster/，并重名名为 zk1

进入cd  zk1 目录，新建目录 data ，并在该目录下新建文件 myid ，内容为 1

进入 cd zk1/conf 目录下，将 zoo.simple.cfg ,复制一件并重命名为 zoo.cfg

编辑 zoo.cfg ,添加 server.id (id 为整数，值为 myid 文件中内容),再添加集群中的服务器地址及端口号（zookeeper集群内部通信的端口号和zookeeper 集群选取领导 leader 的端口号 ）

### Zookeeper 数据模型

![](https://zookeeper.apache.org/doc/r3.5.5/images/zknamespace.jpg)

### 常用命令

### ls

| 功能 | ls 命令获取路径下的节点信息(所有子节点),注意路径为绝对路径 |
| ---- | ---------------------------------------------------------- |
| 作用 | ls  /zookeeper                                             |

### create

| 功能                  | 创建 zookeeper 下的节点                                      |
| --------------------- | ------------------------------------------------------------ |
| create /path null     | 创建 zookeeper 下的空节点                                    |
| create -s /path data  | 创建 zookeeper 下的持续节点                                  |
| craete -e  /path data | 创建 zookeeper 下的瞬时节点，即与 zookeeper 断开连接后，创建的瞬时节点会自动移除。 |

### get 命令

| 功能       | 获取 zookeeper 下节点中的信息 |
| ---------- | ----------------------------- |
| get  /path | 获取指定的节点信息            |

## Zookeeper 服务发现

### 目标：

使用 RMI 注册服务

使用 Zookeeper 的 API 实现服务发现

### 代码实现：

Costants.java:

```java
package com.szxy.service;

public class Constants {
	
	// zookeeper 客户端地址
	public final static String ZK_HOST = "192.168.170.129:2181,192.168.170.129:2182,192.168.170.129:2183";
	
	// 连接 zookeeper 超时时间
	public final static int CONN_TIME_OUT = 5000;
	
	//创建发布服务的 zookeepe 永久节点
	public final static String ZK_REGISTRY = "/provider";
	
	//创建发布服务的 zookeeper 瞬时节点
	public final static String ZK_RMI = ZK_REGISTRY+"/rmi";
	
}
```

ServiceProvider.java:

```java
package com.szxy.service;

import java.rmi.Naming;
import java.rmi.Remote;
import java.rmi.registry.LocateRegistry;
import java.util.concurrent.CountDownLatch;

import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;

public class ServiceProvider {
	
	//用于线程同步
	//参数 1 表示线程挂起
	//参数  0 表示线程执行
	private CountDownLatch latch = new CountDownLatch(1);
	
	//连接  zookeeper 客户端
	public ZooKeeper connZk() {
		ZooKeeper zk = null;
		try {
			zk = new ZooKeeper(Constants.ZK_HOST, Constants.CONN_TIME_OUT, new Watcher() {
				@Override
				public void process(WatchedEvent event) {
					//判断 zookeeper 是否连接上
					if(event.getState() == Event.KeeperState.SyncConnected) {
						latch.countDown(); //唤醒等待状态的线程
					}
				}
			});
			latch.await();//将线程挂起，使当前线程处于等待状态
		} catch (Exception e) {
			e.printStackTrace();
		}
		return zk;
	}
	
	//创建 Node 
	public void createNode(ZooKeeper zk,String urls) {
		try {
			//将 RMI 服务地址转为字节数组
			byte[] data = urls.getBytes();
			//ZooDefs.Ids.OPEN_ACL_UNSAFE 不考虑安全限制
			//CreateMode.EPHEMERAL_SEQUENTIAL 创建瞬时序列节点
			zk.create(Constants.ZK_RMI, data,ZooDefs.Ids.OPEN_ACL_UNSAFE , CreateMode.EPHEMERAL_SEQUENTIAL);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	//发布服务地址
	public String publishService(Remote remote,String host,int port) {
		String url = null;
		try {
			LocateRegistry.createRegistry(port);
			url = "rmi://"+host+":"+port+"/rmiservice";
			Naming.bind(url, remote);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return url;
	}
	
	//发布服务
	public void publish(Remote remote,String host,int port) {
		// RMI 发布远程服务
		String url = publishService(remote,host,port);
		//连接 zookeeper
		ZooKeeper zk = connZk();
		if(zk != null) {
			//创建节点
			createNode(zk, url);
		}
	}
}
```

ZooKeeperClusterProviderAPP.java:

```java
package com.szxy.app;

import com.szxy.service.ServiceProvider;
import com.szxy.service.UserService;
import com.szxy.service.UserServiceImpl;

public class ZooKeeperClusterProviderAPP {
	public static void main(String[] args) {
		ServiceProvider provider = new ServiceProvider();
		try {
			UserService userService = new UserServiceImpl();
			provider.publish(userService, "localhost", 8888);
		} catch (Exception e) {
			e.printStackTrace();
		}
		
	}
}

/**
 * org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /provider/rmi
	at org.apache.zookeeper.KeeperException.create(KeeperException.java:111)
	at org.apache.zookeeper.KeeperException.create(KeeperException.java:51)
	at org.apache.zookeeper.ZooKeeper.create(ZooKeeper.java:783)
	at com.szxy.service.ServiceProvider.createNode(ServiceProvider.java:52)
	at com.szxy.service.ServiceProvider.publish(ServiceProvider.java:79)
	at com.szxy.app.ZooKeeperAPP.main(ZooKeeperAPP.java:12)
	
	原因：没有创建持续节点  provider
	创建空数据的 zookeeper 持续节点的命令 create /provider null
 */

```

## Zookeeper 服务注册

### 目标：

使用 RMI 消费服务

使用 Zookeeper 的 API 实现服务订阅

### 代码实现：

ServiceConsumer.java

```java
package com.szxy.service;

import java.net.ConnectException;
import java.rmi.Naming;
import java.rmi.Remote;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ThreadLocalRandom;

import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;

/**
 *  服务消费者
 *
 */
public class ServiceConsumer {
	
	//消费远程服务地址列表
	// volatile 让线程间共享 urls	
	private volatile List<String> urls = new ArrayList<String>();
	
	private CountDownLatch latch = new CountDownLatch(1);
	
	//取出的服务地址
	private String url = null; 
	
	//连接 zookeeper 客户端
	public ZooKeeper connZK() {
		ZooKeeper zk = null;
		try {
			zk = new ZooKeeper(Constants.ZK_HOST, Constants.CONN_TIME_OUT, new Watcher() {

				@Override
				public void process(WatchedEvent event) {
					//判断是否连接上 ZooKeeper 客户端
					if (event.getState() == Event.KeeperState.SyncConnected) {
						latch.countDown();//唤醒线程
					}
				}
			});
			latch.await();//挂起当前线程
		} catch (Exception e) {
			e.printStackTrace();
		}
		return zk;
	}
	
	//观察节点变化
	public void watchNode(final ZooKeeper zk) {
		try {
			//存放节点集合
			List<String> nodeList = zk.getChildren(Constants.ZK_REGISTRY, new Watcher() {
				@Override
				public void process(WatchedEvent event) {
					//NodeChildrenChanged  若节点发生变化，则重新获取节点数据
					if(event.getType() == Event.EventType.NodeChildrenChanged) {
						System.out.println("========节点发生变化==========");
						watchNode(zk); // 回调
					}
				}
			});
			//存放读取数据的地方
			List<String>  dataList = new ArrayList<>();
		 	//从节点集合获取服务地址数据
		 	for (String node : nodeList) {
		 		try {
		 			byte[] data = zk.getData(Constants.ZK_REGISTRY+"/"+node, false, null);
		 			// 注意这里不能直接添加 urls 中
		 			//  因为 urls 中可能还有旧数据，但是无法使用
		 			dataList.add(new String(data));
		 		} catch (Exception e) {
		 			e.printStackTrace();
		 		}
		 	}
			//将 dataList 复制给 urls
			urls = dataList;
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	//从服务地址列表中获取服务
	public <T>  T  lookupService(String url){
		T remote = null;
		try {
			remote = (T) Naming.lookup(url);
		} catch (Exception e) {
			e.printStackTrace();
			//增强程序的健壮性
			if(e instanceof ConnectException) {
				if(urls.size() != 0) {
					url = urls.get(0);
					return lookupService(url);
				}
			}
		}
		return remote;
	}
	
	//使用随机法，从服务地址列表取出服务地址
	public <T extends Remote> T lookup() {
		T service = null;
		//随机产生的地址列表
		int size = urls.size();
		if(size > 0) {
			String url = null;
			if(size == 1) {
				url = urls.get(0);
			}else {
				//获取随机数
				int randomNum = ThreadLocalRandom.current().nextInt(urls.size());
				url = urls.get(randomNum);
				System.out.println("====="+url);
			}
			service = lookupService(url);
		}
		return service;
	}
	
	public ServiceConsumer() {
		//连接 zookeeper 集群
		ZooKeeper zk = connZK();
		if(zk != null) {
			//从 zookeeper 节点中取出 rmi服务地址 url
			watchNode(zk);
		}
		System.out.println(urls);
	}
}
```

ZookeeperClusterConsumerApp.java:

```java
package com.szxy.app;

import java.rmi.RemoteException;

import com.szxy.service.ServiceConsumer;
import com.szxy.service.UserService;

public class ZookeeperClusterConsumerApp {
	
	public static void main(String[] args) throws InterruptedException, RemoteException {
		
		ServiceConsumer consumer = new ServiceConsumer();
		
		while(true) {
			
			UserService userService  = consumer.lookup();
			
			String result = userService.helloConsumer("zookeeper!!!");
			
			System.out.println("result:"+result);
			
			Thread.sleep(3000);
		}
		
	}
}
```

## Dubbo 介绍

Apache Dubbo |ˈdʌbəʊ| 是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。

![](http://dubbo.apache.org/img/architecture.png)

### 背景

随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需一个治理系统确保架构有条不紊的演进。

![](http://dubbo.apache.org/docs/zh-cn/user/sources/images/dubbo-architecture-roadmap.jpg)

#### 单一应用架构

当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。

#### 垂直应用架构

当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键。

#### 分布式服务架构

当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。

#### 流动计算架构

当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。

### 需求

![](http://dubbo.apache.org/docs/zh-cn/user/sources/images/dubbo-service-governance.jpg)

在大规模服务化之前，应用可能只是通过 RMI 或 Hessian 等工具，简单的暴露和引用远程服务，通过配置服务的URL地址进行调用，通过 F5 等硬件进行负载均衡。

**当服务越来越多时，服务 URL 配置管理变得非常困难，F5 硬件负载均衡器的单点压力也越来越大。** 此时需要一个服务注册中心，动态地注册和发现服务，使服务的位置透明。并通过在消费方获取服务提供方地址列表，实现软负载均衡和 Failover，降低对 F5 硬件负载均衡器的依赖，也能减少部分成本。

**当进一步发展，服务间依赖关系变得错踪复杂，甚至分不清哪个应用要在哪个应用之前启动，架构师都不能完整的描述应用的架构关系。** 这时，需要自动画出应用间的依赖关系图，以帮助架构师理清理关系。

**接着，服务的调用量越来越大，服务的容量问题就暴露出来，这个服务需要多少机器支撑？什么时候该加机器？** 为了解决这些问题，第一步，要将服务现在每天的调用量，响应时间，都统计出来，作为容量规划的参考指标。其次，要可以动态调整权重，在线上，将某台机器的权重一直加大，并在加大的过程中记录响应时间的变化，直到响应时间到达阈值，记录此时的访问量，再以此访问量乘以机器数反推总容量。

### 架构

![](http://dubbo.apache.org/docs/zh-cn/user/sources/images/dubbo-architecture.jpg)

##### 节点角色说明

| 节点        | 角色说明                               |
| ----------- | -------------------------------------- |
| `Provider`  | 暴露服务的服务提供方                   |
| `Consumer`  | 调用远程服务的服务消费方               |
| `Registry`  | 服务注册与发现的注册中心               |
| `Monitor`   | 统计服务的调用次数和调用时间的监控中心 |
| `Container` | 服务运行容器                           |

##### 调用关系说明

1. 服务容器负责启动，加载，运行服务提供者。
2. 服务提供者在启动时，向注册中心注册自己提供的服务。
3. 服务消费者在启动时，向注册中心订阅自己所需的服务。
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于**长连接**推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

### 服务注册

#### 定义接口

UserService:java

```java
package com.szxy.service;


public interface UserService {
	
	public String loadUserInfo(Integer id);
	
}

```

#### 定义接口实现类

userServiceImpl.java

```java
package com.szxy.service.impl;

import com.szxy.service.UserService;

public class UserServiceImpl implements UserService {

	@Override
	public String loadUserInfo(Integer id) {
		return "20880 当前用户的ID:"+id;
	}

}
```

#### dubbo 配置文件

application-dubbo.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:dubbo="http://code.alibabatech.com/schema/dubbo" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans        
    	http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        
    	http://code.alibabatech.com/schema/dubbo 
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
   	
    
	<!-- 提供方应用信息，用于计算依赖关系 -->
	<dubbo:application name="dubbo-application" />

	<!-- 使用 zookeeper 注册中心暴露服务地址 -->
	<dubbo:registry protocol="zookeeper"
		address="192.168.170.129:2181,192.168.170.129:2182,192.168.170.129:2183" />

	<!-- 用dubbo协议在20880端口暴露服务 -->
	<dubbo:protocol name="dubbo" port="20881" />
	
	<bean id="userService" class="com.szxy.service.impl.UserServiceImpl"></bean>

	<!-- 声明需要暴露的服务接口 -->
	<dubbo:service interface="com.szxy.service.UserService"
		ref="userService" />

</beans>
```

#### pom.xml 文件

开始使用 dubbo jar 的版本是 `2.5.4`，但是在书写启动类时报错，将 jar 版本改为  `2.5.3`，即可解决。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.szxy</groupId>
	<artifactId>dubbo-provider</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<dependencies>
		<!-- 添加 Dubbo 依赖 -->
		<!-- https://mvnrepository.com/artifact/com.alibaba/dubbo -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>dubbo</artifactId>
			<version>2.5.3</version>
		</dependency>
		<!-- zkClient 客户端工具 -->
		<dependency>
			<groupId>com.101tec</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.10</version>
		</dependency>
	</dependencies>
</project>
```

#### 启动类

```java
package com.szxy.app;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Provider {
    public static void main(String[] args) throws Exception {
    	
    	ClassPathXmlApplicationContext ac =
        new ClassPathXmlApplicationContext("classpath:application-dubbo.xml");
    	ac.start();
    	
        System.in.read(); // 按任意键退出
    }
}
```

### 服务发现

#### 定义接口

这里的接口与服务注册中的接口一致

#### dubbo 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:dubbo="http://code.alibabatech.com/schema/dubbo" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans        
    	http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        
    	http://code.alibabatech.com/schema/dubbo 
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
   	
	<!-- 提供方应用信息，用于计算依赖关系 -->
	<dubbo:application name="dubbo-application" />

	<!-- 使用 zookeeper 注册中心暴露服务地址 -->
	<dubbo:registry protocol="zookeeper"
		address="192.168.170.129:2181,192.168.170.129:2182,192.168.170.129:2183" />
		
	 <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="userService" interface="com.szxy.service.UserService" />
</beans>
```

#### 启动类

Consumer.java

```java
package com.szxy.service;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Consumer {
	public static void main(String[] args) throws InterruptedException {
		
		ClassPathXmlApplicationContext ac = 
				new ClassPathXmlApplicationContext("classpath:application-dubbo.xml");
		ac.start();
		UserService userService = (UserService) ac.getBean("userService");
		while(true) {
			String result = userService.loadUserInfo(111);
			System.out.println("result:"+result);
			Thread.sleep(3000);
		}
	}
}
```

