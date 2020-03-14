---
title: Zookeeper
date: 2019-10-25
---



[TOC]

## paxos 小岛的故事

**组成：**

- 议员： 管理小岛
- 议员记事本： 记录处理的草案的编号，初始当前编号为 0。
- 草案 (提议)： 由单个议员发起草案，草案的编号必须大于议员记事本上记录的编号，否则不予处理
- 法令： 由单个议员发起，需要有半数以上议员们同意，才可生效

**过程：**

​	比方说，该小岛有三个议员，开始的时候，三个议员a、b、c手中记事本记录草案的编号都是 0 。议员 a 提出草案工业用电 2 元/度 ，提出的草案编号为 1，议员 a 自己的记事本的当前编号设置为 1 ，将该草案告诉其他议员 b、c。议员 b、c 接收到该提议后，首先查看自己记事本的当前编号为 0 ，可以处理该草案，处理（赞成或反对）结束后将自己记事本的当前编号设置为 1 ，最后若赞成草案的票数大于半数，则该草案通过，否则不通过。

**对应关系：**

- 小岛(Island)——ZK Server Cluster 
- 议员(Senator)——ZK Serverabout:reader?url=https://www.douban.com/note/208430424/ 
- 提议(Proposal)——ZNode Change(Create/Delete/SetData…) 
- 提议编号(PID)——Zxid(ZooKeeper Transaction Id) 
- 正式法令——所有ZNode及其数据

## Zookeeper 简介

> Zookeeper 是Google的Chubby一个开源的实现，是Hadoop的分布式协调服务包含一个简单的原语集，分布式应用程序可以基于它实现

特点：

![](http://img.zwer.xyz/blog/20191025215429.png)

### Zookeeper 集群

> zookeeper 集群的数量是单数的

![](http://img.zwer.xyz/blog/20191025212148.png)



![](http://img.zwer.xyz/blog/20191025215327.png)

### 数据模型

```
目录结构
    层次的，目录型结构，便于管理逻辑关系
    节点znode而非文件file
znode信息
    包含最大1MB的数据信息
    记录了zxid等元数据信息
节点类型
    znode有两种类型，瞬时的（ephemeral）和持久的（persistent）
    znode支持序列SEQUENTIAL：leader
    短暂znode的客户端会话结束时，zookeeper会将该短暂znode删除，短暂znode不可以有子节点
    持久znode不依赖于客户端会话，只有当客户端明确要删除该持久znode时才会被删除
    znode的类型在创建时确定并且之后不能再修改
    有序znode节点被分配唯一单调递增的整数。
            比如：客户端创建有序znode，路径为/task/task-，则zookeeper为其分配序号1，并追加到znode节点：
    /task/task-1。有序znode节点唯一，同时也可根据该序号查看znode创建顺序。
    znode有四种形式的目录节点
    PERSISTENT
    EPHEMERAL
    PERSISTENT_SEQUENTIAL
    EPHEMERAL_SEQUENTIAL
```

![](http://img.zwer.xyz/blog/20191025215641.png)

### 事件监听

- **基于客户端轮询机制**

  缺陷：客户端轮询指定节点下的数据，通过网络轮询，代价很大

  ![](http://img.zwer.xyz/blog/20191025220242.png)

- **基于事件监听机制**

  客户端向 ZooKeeper 注册需要接收通知的 znode，通过对 znode 设置监视点（watch）来接收通知。

  监视点是一个**单次触发**的操作，意即监视点会触发一个通知。

  为了接收多个通知，客户端必须在每次通知后设置一个新的监视点。

![](http://img.zwer.xyz/blog/20191025220536.png)



- **事件监听 Watcher**
	Watcher 在 ZooKeeper 是一个核心功能，Watcher 可以监控目录节点的数据变化以及子目录的变化，一旦这些状态发生变化，服务器就会通知所有设置在这个目录节点上的Watcher，从而每个客户端都很快知道它所关注的目录节点的状态发生变化，而做出相应的反应

可以设置观察的操作：`exists`,`getChildren`,`getData`
	可以触发观察的操作：`create`,`delete`,`setData`
	
	回调client方法业务核心代码在哪里？ `client`

### 节点模式-原子广播

Zookeeper的核心是**原子广播**，这个机制保证了各个server之间的同步。实现这个机制的协议叫做Zab协议。

```
Zab协议有两种模式：
恢复模式
广播模式
当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，
当领导者被选举出来，且大多数server的完成了和leader的状态同步以后，恢复模式就结束了。
状态同步保证了leader和server具有相同的系统状态
```

| 恢复模式     | 广播模式                     |
| ------------ | ---------------------------- |
| 无主，无服务 | 主从模式                     |
| 选举leader   | leader维护事物的唯一和有序性 |
|              | 队列机制                     |

下面详细介绍广播模式和恢复模式

- **广播模式**

  广播模式需要保证 proposal 被按顺序处理，因此zk采用了递增的事务id号(zxid)来保证。

  所有的提议 (proposal) 都在被提出的时候加上了 zxid。

  实现中 zxid 是一个 64 为的数字，它高32位是 epoch 用来标识 leader 关系是否改变，每次一个leader被选出来，它都会有一个新的epoch，低32位是个递增计数。

  两阶段提交：

  ![](http://img.zwer.xyz/blog/20191025221442.png)

- **恢复模式-选举**
  
  1，判定（zxid，myid）；2，投票传递
  
  ```
  zxid   <从paxos 到 zookeeper>
  myid
  
  首先选举zxid最大的，如果zxid相同，则选举myid最大的
  ```
  
  **Leader 选举**
  
  选举过程耗时在 200ms 之内，一般情况下zookeeper恢复服务时间间隔不超过 200ms
  
  server Si (myid,zxid)
  
  ![](http://img.zwer.xyz/blog/20191025222248.png)
  
  ### 总结
  
  ![](http://img.zwer.xyz/blog/20191025222727.png)
  
  



##   Zookeeper 安装与配置

- 安装

  ```
  1.3节点 java 安装 
  
  2.所有集群节点创建目录: mkdir opt/sxt  
  
  3.zk压缩包解压在其他路径下:：
  	#	tar xf zookeeper-3.4.6.tar.gz -C /opt/sxt/
  	
  4.进入conf目录，拷贝zoo_sample.cfg zoo.cfg 并配置
     dataDir，集群节点。
  5.单节点配置环境变量、并分发 ZOOKEEPER_PREFIX，共享模式读取profile 
  6. 共享创建 /var/sxt/zk目录，进入各自目录 分别输出1,2，3 至文件 myid
  	echo 1 > /var/sxt/zk/myid
  	...
  7. 共享启动zkServer.sh start 集群
  8.启动客户端 help命令查看
  ```

  

- 配置

    在conf目录下创建一个配置文件 zoo.cfg，

    ```
    tickTime=2000  
    dataDir=/Users/zdandljb/zookeeper/data
    dataLogDir=/Users/zdandljb/zookeeper/dataLog
    clientPort=2181
    initLimit=5
    syncLimit=2
    server.1=server1:2888:3888
    server.2=server2:2888:3888
    server.3=server3:2888:3888  observer（表示对应节点不参与投票）
    ```

    创建myid文件

    tickTime：发送心跳的间隔时间，单位：毫秒
    dataDir：zookeeper保存数据的目录。
    clientPort：客户端连接 Zookeeper 服务器的端口，Zookeeper  会监听这个端口，接受客户端的访问请求。
    initLimit： 这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接Zookeeper服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader的Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 5 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 5*2000=10秒
    syncLimit：这个配置项标识  Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个tickTime 的时间长度，总的时间长度就是 2*2000=4 秒
    server.A=B：C：D：其 中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的ip地址；C 表示的是这个服务器与集群中的Leader服务器交换信息的端口；D表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于B都是一样，所以不同的Zookeeper实例通信端口号不能一样，所以要给它们分配不同的端口号。

  

## Zookeeper 操作

### 常用命令

- `ls` 命令

    | 功能 | ls 命令获取路径下的节点信息(所有子节点),注意路径为绝对路径 |
    | ---- | ---------------------------------------------------------- |
    | 作用 | `ls  /zookeeper`                                           |

- `create` 命令

    | 功能                    | 创建 zookeeper 下的节点                                      |
    | ----------------------- | ------------------------------------------------------------ |
    | `create /path null`     | 创建 zookeeper 下的空节点                                    |
    | `create -s /path data`  | 创建 zookeeper 下的持续节点                                  |
    | `craete -e  /path data` | 创建 zookeeper 下的瞬时节点后<br/>即与 zookeeper 断开连接后，创建的瞬时节点会自动移除。 |

- `get` 命令

    | 功能         | 获取 zookeeper 下节点中的信息 |
    | ------------ | ----------------------------- |
    | `get  /path` | 获取指定的节点信息            |

- `delete 命令`

    | 功能           | 删除 Zookeeper 中节点                                        |
    | -------------- | ------------------------------------------------------------ |
    | `delete /path` | 删除 `/path` 节点，注意 `/path` 下面必须没有子节点，不然不能删除 |


## Zookeeper API  使用

使用  RMI （Remote Method Innvocation），远程调用方法。

使用 zookeeper 作为注册中心

### RMI 案例

#### 服务提供端

- **ServiceProvider.java**：连接 zk，发布服务到 zk 上

  ```java
  public class ServiceProvider {
   
      private static final Logger LOGGER = LoggerFactory.getLogger(ServiceProvider.class);
   
      // 用于等待 SyncConnected 事件触发后继续执行当前线程
      private CountDownLatch latch = new CountDownLatch(1);
   
      // 发布 RMI 服务并注册 RMI 地址到 ZooKeeper 中
      public void publish(Remote remote, String host, int port) {
          String url = publishService(remote, host, port); // 发布 RMI 服务并返回 RMI 地址
          if (url != null) {
              ZooKeeper zk = connectServer(); // 连接 ZooKeeper 服务器并获取 ZooKeeper 对象
              if (zk != null) {
                  createNode(zk, url); // 创建 ZNode 并将 RMI 地址放入 ZNode 上
              }
          }
      }
   
      // 发布 RMI 服务
      private String publishService(Remote remote, String host, int port) {
          String url = null;
          try {
              // rmi://localhost:1099/demo.zookeeper.remoting.server.HelloServiceImpl
              url = String.format("rmi://%s:%d/%s", host, port, remote.getClass().getName());
              LocateRegistry.createRegistry(port);
              Naming.rebind(url, remote);
              LOGGER.debug("publish rmi service (url: {})", url);
          } catch (RemoteException | MalformedURLException e) {
              LOGGER.error("", e);
          }
          return url;
      }
   
      // 连接 ZooKeeper 服务器
      private ZooKeeper connectServer() {
          ZooKeeper zk = null;
          try {
              zk = new ZooKeeper(Constant.ZK_CONNECTION_STRING, Constant.ZK_SESSION_TIMEOUT, new Watcher() {
                  @Override
                  public void process(WatchedEvent event) {
                      if (event.getState() == Event.KeeperState.SyncConnected) {
                          latch.countDown(); // 唤醒当前正在执行的线程
                      }
                  }
              });
              latch.await(); // 使当前线程处于等待状态
          } catch (IOException | InterruptedException e) {
              LOGGER.error("", e);
          }
          return zk;
      }
   
      // 创建 ZNode
      private void createNode(ZooKeeper zk, String url) {
          try {
              byte[] data = url.getBytes();
              // /registry/provider
              // /registry/provider000000003
              String path = zk.create(Constant.ZK_PROVIDER_PATH, data, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL); // 创建一个临时性且有序的 ZNode
              LOGGER.debug("create zookeeper node ({} => {})", path, url);
          } catch (KeeperException | InterruptedException e) {
              LOGGER.error("", e);
          }
      }
      
      
  }
  ```

- **HelloServiceImpl.java**：服务具体实现类

  ```java
  public class HelloServiceImpl extends UnicastRemoteObject implements HelloService {
   
      protected HelloServiceImpl() throws RemoteException {
      }
   
      @Override
      public String sayHello(String name) throws RemoteException {
          return String.format("Hello %s", name);
      }
  }
  ```

- **Server.java** ：服务提供者 

  ```java
  public class Server {
   
      public static void main(String[] args) throws Exception {
          String host = "localhost";
          int port = Integer.parseInt("11117");
          ServiceProvider provider = new ServiceProvider();
   
          HelloService helloService = new HelloServiceImpl();
          provider.publish(helloService, host, port);
   
          Thread.sleep(Long.MAX_VALUE);
      }
  }
  ```

#### 服务消费端

- **ServiceConsumer.java**:连接 zk ，获取服务列表

  ```java
  public class ServiceConsumer {
   
      private static final Logger LOGGER = LoggerFactory.getLogger(ServiceConsumer.class);
   
      // 用于等待 SyncConnected 事件触发后继续执行当前线程
      private CountDownLatch latch = new CountDownLatch(1);
   
      // 定义一个 volatile 成员变量，用于保存最新的 RMI 地址（考虑到该变量或许会被其它线程所修改，一旦修改后，该变量的值会影响到所有线程）
      private volatile List<String> urlList = new ArrayList<>(); 
   
      // 构造器
      public ServiceConsumer() {
          ZooKeeper zk = connectServer(); // 连接 ZooKeeper 服务器并获取 ZooKeeper 对象
          if (zk != null) {
              watchNode(zk); // 观察 /registry 节点的所有子节点并更新 urlList 成员变量
          }
      }
   
      // 查找 RMI 服务
      public <T extends Remote> T lookup() {
          T service = null;
          int size = urlList.size();
          if (size > 0) {
              String url;
              if (size == 1) {
                  url = urlList.get(0); // 若 urlList 中只有一个元素，则直接获取该元素
                  LOGGER.debug("using only url: {}", url);
              } else {
                  url = urlList.get(ThreadLocalRandom.current().nextInt(size)); // 若 urlList 中存在多个元素，则随机获取一个元素
                  LOGGER.debug("using random url: {}", url);
              }
              System.out.println(url);
              service = lookupService(url); // 从 JNDI 中查找 RMI 服务
          }
          return service;
      }
   
      // 连接 ZooKeeper 服务器
      private ZooKeeper connectServer() {
          ZooKeeper zk = null;
          try {
              zk = new ZooKeeper(Constant.ZK_CONNECTION_STRING, Constant.ZK_SESSION_TIMEOUT, new Watcher() {
                  @Override
                  public void process(WatchedEvent event) {
                      if (event.getState() == Event.KeeperState.SyncConnected) {
                          latch.countDown(); // 唤醒当前正在执行的线程
                      }
                  }
              });
              latch.await(); // 使当前线程处于等待状态
          } catch (IOException | InterruptedException e) {
              LOGGER.error("", e);
          }
          return zk;
      }
   
      // 观察 /registry 节点下所有子节点是否有变化
      private void watchNode(final ZooKeeper zk) {
          try {
              List<String> nodeList = zk.getChildren(Constant.ZK_REGISTRY_PATH, new Watcher() {
                  @Override
                  public void process(WatchedEvent event) {
                      if (event.getType() == Event.EventType.NodeChildrenChanged) {
                          watchNode(zk); // 若子节点有变化，则重新调用该方法（为了获取最新子节点中的数据）
                      }
                  }
              });
              List<String> dataList = new ArrayList<>(); // 用于存放 /registry 所有子节点中的数据
              for (String node : nodeList) {     //   /registry/provider00000000003
                  byte[] data = zk.getData(Constant.ZK_REGISTRY_PATH + "/" + node, false, null); // 获取 /registry 的子节点中的数据
                  dataList.add(new String(data));
              }
              LOGGER.debug("node data: {}", dataList);
              urlList = dataList; // 更新最新的 RMI 地址
          } catch (KeeperException | InterruptedException e) {
              LOGGER.error("", e);
          }
      }
   
      // 在 JNDI 中查找 RMI 远程服务对象
      @SuppressWarnings("unchecked")
      private <T> T lookupService(String url) {
          T remote = null;
          try {
              remote = (T) Naming.lookup(url);
          } catch (NotBoundException | MalformedURLException | RemoteException e) {
              if (e instanceof ConnectException) {
                  // 若连接中断，则使用 urlList 中第一个 RMI 地址来查找（这是一种简单的重试方式，确保不会抛出异常）
                  LOGGER.error("ConnectException -> url: {}", url);
                  if (urlList.size() != 0) {
                      url = urlList.get(0);
                      return lookupService(url);
                  }
              }
              LOGGER.error("", e);
          }
          return remote;
      }
  }
  ```

- **RmiClient.java**

  ```java
  public class Client {
   
      public static void main(String[] args) throws Exception {
          ServiceConsumer consumer = new ServiceConsumer();
          // zookeeper测试
          while (true) {
              HelloService helloService = consumer.lookup();
              String result = helloService.sayHello("Jack");
              System.out.println(result);
              Thread.sleep(2000);
          }
      }
  }
  ```

#### 共有

- **HelloService.java**

  ```java
  public interface HelloService extends Remote {
   
      String sayHello(String name) throws RemoteException;
  }
  ```

- **Constant.java**：

  ```java
  public interface Constant {
  
  	String ZK_CONNECTION_STRING = "node02:2181,node03:2181,node04:2181";
  	int ZK_SESSION_TIMEOUT = 5000;
  	String ZK_REGISTRY_PATH = "/registry";
  	String ZK_PROVIDER_PATH = ZK_REGISTRY_PATH + "/provider";
  }
  ```

### Socket 案例

Socket 服务端向 zk 注册服务，Socket 客户端从 zk 获取服务列表，从而连接服务器

- Server.java

  ```java
  public class Server {
  
      private ZooKeeper zk = null;
      private String zkHost = "node02:2181,node03:2181,node04:2181";
      private int SESSION_OUT = 3000;
  
      private CountDownLatch latch = new CountDownLatch(1);
  
      public static void main(String[] args) throws Exception {
          for(int i = 0; i < 3; i++){
              Thread t = new Thread(new Runnable() {
                  @Override
                  public void run() {
                      try {
                          String host = "localhost";
                          /*Random random = new Random();
                          String num = String.format("2%4d",random.nextInt(9999));*/
                          int port = (int)((Math.random()*9+1)*10000);
                          String address = host + ":" + String.valueOf(port);
                          // 创建对象
                          Server s = new Server();
                          // 连接 zk 集群
                          s.connectZk();
                          // 开启服务端
                          ServerSocket server = new ServerSocket(port);
                          // 注册服务
                          s.registryService(address);
                          while (true) {
                              Socket client = server.accept();
                              handleReq(client);
                          }
                      } catch (Exception e) {
                          e.printStackTrace();
                      }
                  }
              });
              t.start();
          }
      }
  
      /**
       * 连接 zk 集群
       *
       * @throws Exception
       */
      private void connectZk() throws Exception {
          zk = new ZooKeeper(zkHost, SESSION_OUT, new Watcher() {
              @Override
              public void process(WatchedEvent event) {
                  if (event.getState() == Event.KeeperState.SyncConnected) {
                      latch.countDown();
                  }
              }
          });
          latch.await();
      }
  
      /**
       * 注册服务
       *
       * @param address
       */
      private void registryService(String address) throws Exception {
          String path = "/myRegistry/service";
          zk.create(path,address.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
      }
  
      /**
       * 处理客户端请求
       *
       * @param client
       * @throws Exception
       */
      private static void handleReq(Socket client) throws Exception {
          OutputStream os = client.getOutputStream();
          OutputStreamWriter osw = new OutputStreamWriter(os, "utf-8");
          BufferedWriter writer = new BufferedWriter(osw);
          writer.write("Hello Client");
          writer.newLine();
          writer.flush();
          client.close();
      }
  
  }
  
  ```

- Client.java

  ```java
  public class Client {
  
      private ZooKeeper zk = null;
      private String zkHost = "node02:2181,node03:2181,node04:2181";
      private int SESSION_OUT = 3000;
  
      private CountDownLatch latch = new CountDownLatch(1);
  
      private List<String> urlList = null;
  
      private Socket client = null;
  
      public static void main(String[] args) throws Exception {
          Client c = new Client();
          // 连接 zk 集群
          c.connectZk();
          // 获取注册服务列表
          c.getRegistryList();
  
          while (true) {
              // 连接服务器
              c.connectServer();
          }
      }
  
      /**
       *  连接 zk 集群
       */
      private void connectZk() {
          try {
              zk = new ZooKeeper(zkHost, SESSION_OUT, new Watcher() {
                  @Override
                  public void process(WatchedEvent event) {
                      if (event.getState() == Event.KeeperState.SyncConnected) {
                          latch.countDown();
                      }
                  }
              });
              latch.await();
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  
      /**
       * 获取服务列表
       */
      private void getRegistryList() {
          List<String> nodeList = null;
          try {
              nodeList = zk.getChildren("/myRegistry", new Watcher() {
                  @Override
                  public void process(WatchedEvent event) {
                      if (event.getType() == Event.EventType.NodeChildrenChanged) {
                          System.out.println("节点发生变化。。。");
                          getRegistryList(); // 调用自身方法
                      }
                  }
              });
              urlList = new ArrayList<>();
              for (String node : nodeList) {
                  byte[] data = zk.getData("/myRegistry/" + node, null, null);
                  String url = new String(data);
                  urlList.add(url);
              }
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  
      /**
       * 连接服务器
       *
       * @throws Exception
       */
      private void connectServer() throws Exception {
          String url = null;
          String host = null;
          if (this.urlList.size() == 0) {
              System.out.println("没有可用的服务器....");
              Thread.sleep(2000);
              return;
          } else if (this.urlList.size() == 1) {
              url = this.urlList.get(0);
          } else {
              Random random = new Random();
              url = this.urlList.get(random.nextInt(urlList.size()));
          }
  
          host = url.substring(0, url.indexOf(":"));
          int port = Integer.valueOf(url.substring(url.indexOf(":") + 1));
  
          client = new Socket(host, port);
          InputStream is = client.getInputStream();
          InputStreamReader isr = new InputStreamReader(is, "utf-8");
          BufferedReader reader = new BufferedReader(isr);
          String line = reader.readLine();
          System.out.println(host + ":" + port + "  服务器响应消息: " + line);
          Thread.sleep(2000);
  
          client.close();
      }
  
  
  }
  
  ```

  









  

  

  

