---
title: 20190805 Spring Cloud
date: 2019-08-05
---

## 微服务架构介绍 ##

### 单体架构 ###

> 单体架构也称之为单体系统或者是单体应用。
>
> 就是把系统中**所有的功能、模块耦合在一个应用中**的架构方式。

**特点**

1. 打包成一个独立的单元(导成一个唯一的 jar 包或者是 war 包)

2. 会一个进程的方式来运行


| 优点                           | 缺点                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| 项目易于管理<br/>部署简单<br/> | 测试成本高<br/>可伸缩性差<br/>可靠性差<br/> 迭代困难<br/>跨语言程度差<br/>团队协作难<br/> |

![](http://img.zwer.xyz/blog/20190805101517.png)



### 微服务架构 ###

> 一种**架构风格**。
>
> 一个大型的复杂软件应用，由一个或多个微服务组成。
>
> 系统的各个微服务可被**独立部署**，各个微服务之间是**松耦合**的。
>
> 每个微服务**仅关注于完成一件任务并很好的完成该任务**。 

架构风格

> 项目的一种设计模式（非GOF23）。

常见的架构风格：

1. 客户端与服务端  
2. 基于组件模型的架构（EJB）
3. 分层架构（MVC）
4. 面向服务架构（SOA）

| SOA  特点                                                    |
| ------------------------------------------------------------ |
| 系统是由多个服务构成                                         |
| 每个服务可以单独独立部署                                     |
| 每个服务之间是松耦合。服务内部是高内聚，外部是低耦合。<br/>高内聚就是**每个服务只关注完成一个功能** |

| 优点                                                         | 缺点                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 测试容易<br/>可伸缩性强<br/>可靠性强<br/>跨语言程度 更加灵活<br/>团队协作容易<br/>系统迭代容易 | 运维成本过高，部署数量较多<br/>**接口兼容多版本**<br/>分布式系统的复杂性<br/>分布式事务（难点） |

### MVC 、RPC、SOA、微服务架构 ###

![](http://img.zwer.xyz/blog/架构区别.png)

| 架构 | MVC                | RPC                                       | SOA                                                          | 微服务               |
| ---- | ------------------ | ----------------------------------------- | ------------------------------------------------------------ | -------------------- |
| 介绍 | 一个单体架构。     | RPC(Remote procedure Call )：远程过程调用 | Service Oriented Archtitecture：面向服务架构<br/>ESB(Enterparise Service Bus)：企业服务总线，服务中介。主要提供了一个服务与服务之间的交互<br/>ESB 包括的功能如：负载均衡、流量控制，加密处理，服务的监控，异常处理，监控告急等等 | 轻量级的服务治理方案 |
| 技术 | Struts2、SpringMVC | Thrift、Hessian                           | Mule（付费）、WSO2（free 产品）                              | SpringCloud、Dubbo   |

### 设计微服务架构 ###

#### AKF 拆分原则（重点） ####

> 通过加机器就可以解决容量和可用性问题。（如果一台不行那就两台）

《可扩展的艺术》一书提出了一 个更加系统的可扩展模型—— **AKF** **可扩展立方** （Scalability Cube）。这个立方 

体中沿着三个坐标轴设置分别为：X、Y、Z。 

![](http://img.zwer.xyz/blog/20190805102443.png)

- Y 轴数据（功能）

Y 轴扩展会将庞大的整体应用拆分为多个服务。每个服务实现一组相关的功能，如订单管理、客户管理等。在工程上常见的方案是 **服务化架构**(SOA)。比 如对于一个电子商务平台，我们可以拆分成不同的服务，组成下面这样的架构： 

![](http://img.zwer.xyz/blog/20190805103038.png)

但通过观察上图容易发现，当服务数量增多时，服务调用关系变得复杂。为 系统添加一个新功能，要调用的服务数也变得不可控，由此引发了服务管理上的 混乱。所以，一般情况下，需要采用服务注册的机制形成服务网关来进行服务治 理。系统的架构将变成下图所示：

![](http://img.zwer.xyz/blog/20190805103051.png)

- X 轴（水平扩展）

X 轴扩展与我们前面朴素理念是一致的，通过绝对平等地复制服务与数据， 以解决容量和可用性的问题。其实就是将微服务运行多个实例，做集群加负载均 衡的模式。 为了提升单个服务的可用性和容量， **对每一个服务进行** **X** **轴扩展划分** 。

![](http://img.zwer.xyz/blog/20190805103157.png)

- Z 轴数据分区

Z 轴扩展通常是指基于请求者或用户独特的需求，进行系统划分，并使得划 

分出来的子系统是相互隔离但又是完整的。

**单元化架构**

在分布式服务设计领域，一个单元（Cell）就是满足**某个分区所有业务操作的自包含闭环**。如果在此加上 Z 轴扩展，那服务节点的选择将不再是随机的了，而是每个单元自成一体。如下图：

![](http://img.zwer.xyz/blog/20190805102800.png)

**数据分区**

为了性能数据安全上的考虑，我们将一个完整的数据集按一定的维度划分出 不同的子集。 一个分区（Shard），就是是整体数据集的一个子集。比如用尾 号来划分用户，那同样尾号的那部分用户就可以认为是一个分区。数据分区为般包括以下几种数据划分的方式： 

![](http://img.zwer.xyz/blog/20190805102842.png)

#### 前后端分离 ####

![](http://img.zwer.xyz/blog/20190805103822.png)

何为前后端分离？前后端本来不就分离么？这要从尴尬的 jsp 讲起。分工精细化从来都 是蛋糕做大的原则，多个领域工程师最好在不需要接触其他领域知识的情况下合作，才可能 使效率越来越高，维护也会变得简单。jsp 的模板技术融合了 html 和 java 代码，使得传统 MVC 开发中的前后端在这里如胶似漆，前端做好页面，后端转成模板，发现问题再找前端， 前端又看不懂 java 代码......前后端分离的目的就是将这尴尬局面打破。 前后端分离原则，简单来讲就是前端和后端的代码分离，我们推荐的模式是最好采用物 理分离的方式部署，进一步促使更彻底的分离。如果继续直接使用服务端模板技术，如：jsp， 把 java、js、html、css 都堆到一个页面里，稍微复杂一点的页面就无法维护了。

![](http://img.zwer.xyz/blog/20190805103920.png)

这种分离方式有几个好处： 

1) 前后端技术分离，可以由各自的专家来对各自的领域进行优化，这样前段的用户体 验优化效果更好。

 2) 分离模式下，前后端交互界面更清晰，就剩下了接口模型，后端的接口简洁明了， 更容易维护。 

3) 前端多渠道集成场景更容易实现，后端服务无需变更，采用统一的数据和模型，可 以支持多个前端：例如：微信 h5 前端、PC 前端、安卓前端、IOS 前端。 

#### 无状态服务 ####

![](http://img.zwer.xyz/blog/20190805105040.png)

对于无状态服务，首先说一下什么是状态**：如果一个数据需要被多个服务共享，才能完成一笔交易，那么这个数据被称为状态**。进而依赖这个“状态”数据的服务被称为有状态服务，反之称为无状态服务。

 那么这个无状态服务原则并不是说在微服务架构里就不允许存在状态，表达 的真实意思是要把有状态的业务服务改变为无状态的计算类服务，那么状态数据 也就相应的迁移到对应的“有状态数据服务”中。 

场景说明：例如我们以前在本地内存中建立的数据缓存、Session 缓存，到 现在的微服务架构中就应该把这些数据迁移到分布式缓存中存储，让业务服务变 成一个无状态的计算节点。迁移后，就可以做到按需动态伸缩，微服务应用在运 行时动态增删节点，就不再需要考虑缓存数据如何同步的问题。 

#### 无状态通信-Restful API ####

![](http://img.zwer.xyz/blog/20190805105615.png)

作为一个原则来讲本来应该是个“无状态通信原则”，

在这里我们直接推荐一 个实践优选的 Restful 通信风格 ，因为他有很多好处： 

1) 无状态协议 HTTP，具备先天优势，扩展能力很强。例如需要安全加密，有 现成的成熟方案 HTTPS 即可。

 2) JSON 报文序列化，轻量简单，人与机器均可读，学习成本低，搜索引擎友好。

3) 语言无关，各大热门语言都提供成熟的 Restful API 框架，相对其他的一些 RPC 框架生态更完善。

## Spring Cloud 入门 ##

> 一个服务治理平台，提供了一些服务框架。包含了：服务注册 与发现、配置中心、消息中心 、负载均衡、数据监控等等。 

Spring Cloud 是一个微服务框架，相比 Dubbo 等 RPC 框架, **Spring Cloud 提供的全套的分布式系统解决方案**。 Spring Cloud 对微服务基础框架 Netflix 的多个开源组件进行了封装，同时又实现 了和云端平台以及和 Spring Boot 开发框架的集成。 Spring Cloud 为微服务架构开发涉及的**配置管理，服务治理，熔断机制，智能路由，** **微代理，控制总线，一次性** **token****，全局一致性锁，****leader** **选举，分布式** **session****，集** **群状态**管理等操作提供了一种简单的开发方式。 Spring Cloud 为开发者提供了快速构建**分布式系统的工具**，开发者可以快速的启服务或构建应用、同时能够快速和云平台资源进行对接。 

![](http://img.zwer.xyz/blog/20190805113829.png)

3.1Spring Cloud Config：配置管理工具，支持使用 Git 存储配置内容，支持应 用配置的外部化存储，支持客户端配置信息刷新、加解密配置内容等 

3.2 Spring Cloud Bus：事件、消息总线，用于在集群（例如，配置变化事件）中 传播状态变化，可与 Spring Cloud Config 联合实现热部署3.3Spring Cloud Netflix：针对多种 Netflix 组件提供的开发工具包，其中包Eureka、Hystrix、Zuul、Archaius 等。 

3.3.1 Netflix Eureka：一个基于 rest 服务的服务治理组件，包括服务注册中 心、服务注册与服务发现机制的实现，实现了云端负载均衡和中间层服务器 的故障转移。 

3.3.2 Netflix Hystrix：容错管理工具，实现断路器模式，通过控制服务的节点, 从而对延迟和故障提供更强大的容错能力。 

3.3.3 Netflix Ribbon：客户端负载均衡的服务调用组件。 3.3.4Netflix Feign：基于 Ribbon 和 Hystrix 的声明式服务调用组件。 

3.3.5 Netflix Zuul：微服务网关，提供动态路由，访问过滤等服务。

 3.3.6 Netflix Archaius：配置管理 API，包含一系列配置管理 API，提供动 态类型化属性、线程安全配置操作、轮询框架、回调机制等功能。 

3.4 Spring Cloud for Cloud Foundry ： 通 过 Oauth2 协 议 绑 定 服 务 到 CloudFoundry，CloudFoundry 是 VMware 推出的开源 PaaS 云平台。 

3.5 Spring Cloud Sleuth：日志收集工具包，封装了 Dapper,Zipkin 和 HTrac操作。

 3.6 Spring Cloud Data Flow：大数据操作工具，通过命令行方式操作数据流。 

3.7 Spring Cloud Security：安全工具包，为你的应用程序添加安全控制，主是指 OAuth2。

 3.8 Spring Cloud Consul：封装了 Consul 操作，consul 是一个服务发现与配 置工具，与 Docker 容器可以无缝集成。

3.9 Spring Cloud Zookeeper ： 操 作 Zookeeper 的 工 具 包 ， 用 于 使 zookeeper 方式的服务注册和发现。 

3.10 Spring Cloud Stream：数据流操作开发包，封装了与 Redis,RabbitKafka 等发送接收消息。

 3.11 Spring Cloud CLI：基于 Spring Boot CLI，可以让你以命令行方式快速 建立云组件。

### Spring Cloud 与 Dubbo 对比 ###

> 虽然 Spring Cloud 在性能上与 DubBo 有不小差距，但是 Spring Cloud 在功能上却 Dubbo 更加强大。

![](http://img.zwer.xyz/blog/20190805114106.png)

### Spring Cloud 版本说明 ###

> 设计的目的是为了更好的管理每个 Spring Cloud 的子项目的清单。避免子的版本号与子项目的版本号混淆。 

- Spring Cloud 命名规则

采用伦敦的地铁站名称来作为版本号的命名，

根据首字母排序，字母顺序靠后的版本号越大

- 版本发布计划

![](http://img.zwer.xyz/blog/20190805114208.png)

- 常见版本号说明

软件版本号：2.0.2 RELEASE

2：主版本号。当功能模块有较大更新或者整体架构发生变化时，主版本号会更新

0：次版本号。次版本表示只是局部的一些变动

3：修改版本号。一般是 bug 的修复或者小的变动

RELEASE：希腊字母版本号。该版本号标注当前版本的软件处于哪个开发阶段



| 希腊字母版本号 | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| Base           | 设计阶段。只有相应的设计没有具体的功能实现                   |
| Alpha          | 软件的初级版本                                               |
| Bate           | 表示相对 alpha 有了很对的进步，消除了严重的 bug，还存在潜在的 bug |
| Release        | 该版本表示最终版                                             |

### Spring Cloud 与子项目的版本兼容 ###

![](http://img.zwer.xyz/blog/20190805140016.png)



## Spring Boot 实战 ##

Spring Boot 是在Spring  的基础之上产生的（确切的说 Spring 4.0 版本以后的基础上）

其中 `Boot` 的意思是引导，意在简化开发模式，使开发者能够快速的开发基于Spring 的应用。

Spring Boot 含有一个内嵌的 web 容器。我们开发的 web 应用不需要作为 war 包部署到  web容器中

而是作为一个 jar  包，在启动时根据 web 服务器的配置进行加载。

**没有使用 Spring Boot 开发时项目存放的问题？**

- 在项目中存在大量的 xml 文件
- 整合第三方框架时配置问题
- 低效的开发效率与部署效率问题

**Spring Boot 解决了什么？**

- Spring Boot 使配置简单
- Spring Boot 使编码简单
- Spring Boot 使部署简单
- **Spring Boot 使监控简单**

### Spring Boot 快速构建项目 ###

- 打开 Spring Boot 的官网

```http
https://start.spring.io/
```

![](http://img.zwer.xyz/blog/20190805143851.png)



- 使用 Spring Boot 官网构建项目

  - 会自动的帮助我们生成启动类
  - 会自动生成存放静态资源的目录，以及全局配置文件

  - 会自动生成测试代码，当然只是一个结构

### Spring Boot 全局配置文件 ###

1. **修改内嵌容器的端口号**

```properties
server.port=8888
```

2. **自定义属性配置**

```properties
msg=Hello World 
```

```java
@Value("${msg}")
private String msg;
```

3. **配置变量引用**

```properties
trait=123
msg=Hello World ${trait}
```
```java
@Value("${msg}")
private String msg;
```

4. **随机值配置**

```properties
#随机数
# 只会产生一次随机数
num = ${random.int} 
# 随机端口,生成端口的范围是 1024到8888之间
server.port=${random.int[1024,8888]}
#在 Spring Cloud 中不需要记录服务的 IP 与 端口号
#name需要去维护服务的端口号，让它自己生成
```

注意：

### yml 配置文件 ###

> Spring Boot 中新增的一种配置文件格式。特点：具备天然的树状结构

**yml 配置文件 与 properties 文件的区别**

- 配置文件的扩展名有变化

- 配置文件中语法有变化

**yum 配置文件的语法**

- 在 properties 文件以 `.`点进行分割，在 yum 中使用  `:`冒号 分割
- `yml` 的数据格式和` Json` 的格式很像，都是` key-value` 结构的。并且是通过` ：` 分割
- 在 yum 中缩进一定不能使用 `tab `键，否则会报错
- 每个 `key `的 `:`冒号后面**一定要一个空格**

```yml
server:
       port: 8888

msg: hello World Yml!!!

flag: ${random.int}

my:
  msg: helloWorld Yml!!! OKOKOK ${flag}
  msg2: I'm yml configuration files
```

### logback 日志记录 ###

> 功能上与性能上都强与 log4J

- 导入 logback 的 jar 包
- 添加 logback 配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
 <configuration>
<!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->  
    <property name="LOG_HOME" value="${catalina.base}/logs/" />  
    <!-- 控制台输出 -->   
    <appender name="Stdout" class="ch.qos.logback.core.ConsoleAppender">
       <!-- 日志输出编码 -->  
        <layout class="ch.qos.logback.classic.PatternLayout">   
             <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符--> 
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n   
            </pattern>   
        </layout>   
    </appender>   
    <!-- 按照每天生成日志文件 -->   
    <appender name="RollingFile"  class="ch.qos.logback.core.rolling.RollingFileAppender">   
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/server.%d{yyyy-MM-dd}.log</FileNamePattern>   
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>   
        <layout class="ch.qos.logback.classic.PatternLayout">  
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符--> 
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n   
            </pattern>   
       </layout> 
        <!--日志文件最大的大小-->
       <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
         <MaxFileSize>10MB</MaxFileSize>
       </triggeringPolicy>
    </appender>     

    <!-- 日志输出级别 -->
    <root level="info">   
        <appender-ref ref="Stdout" />   
        <appender-ref ref="RollingFile" />   
    </root> 



<!--日志异步到数据库 -->  
<!--     <appender name="DB" class="ch.qos.logback.classic.db.DBAppender">
        日志异步到数据库 
        <connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">
           连接池 
           <dataSource class="com.mchange.v2.c3p0.ComboPooledDataSource">
              <driverClass>com.mysql.jdbc.Driver</driverClass>
              <url>jdbc:mysql://127.0.0.1:3306/databaseName</url>
              <user>root</user>
              <password>root</password>
            </dataSource>
        </connectionSource>
  </appender> -->

</configuration>
```

### Spring Boot 的配置文件-多环境配置 ###

profile:代表的就是一个环境变量

**语法结构**：`application-{profile}.properties`

- **创建配置文件：**

  开发环境的配置文件  application-dev.properties

  测试环境的配置文件 application-test.properties

  生产环境的配置文件 application-prod.properties

- **打包项目**  install

- **运行项目**

```java 
// 命令
java -jar xxx.jar --spring.profiles.active={profile}

// 栗子
java -jar HelloWorld-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
```

### Spring Boot 核心注解 ###

| 注解                     | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| @SpringBootApplication   | 代表是 SpringBoot 的启动类。                                 |
| @SpringBootConfiguration | 通过 bean 对象来获取配置信息                                 |
| @Configuration           | 通过对 bean 对象的操作替代 spring 中 xml 文件                |
| @EnableAutoConfiguration | 完成一些初始化环境的配置                                     |
| @ComponentScan           | 完成 spring 的组件扫描。替代之前我们在 xml 文件中配置组件扫描的配置 `<context:component-scan pacage=”....”>` |
| @RestController          | 1. 表示一个 Controller。<br/>2. 表示当前这个 Controller 下的所有的方法都会以 Json 格式返回 |

### Spring Boot 全局异常处理 ###

`@ControllerAdvice` +` @ExceptionHandle` 注解

- 创建异常处理控制器 

```java
@ControllerAdvice
public class MyExceptionController {

	/**
	 * 若出现异常信息，则调用该方法返回响应
	 * @return
	 */
	@ResponseBody
	@ExceptionHandler(value=java.lang.Exception.class)
	public Map<String,Object> myException1(Exception ex){
		HashMap<String, Object> map = 
				new HashMap<String,Object>();
		map.put("error", "服务器繁忙！！！  =="+ex.getMessage());
		return map;
	}
	
	/**
	 * 拦截特定的异常信息 
	 * @param ex
	 * @return
	 */
	@ResponseBody
	@ExceptionHandler(value=java.lang.NullPointerException.class)
	public Map<String,Object> myException2(Exception ex){
		HashMap<String, Object> map = 
				new HashMap<String,Object>();
		map.put("error", "服务器繁忙！！！500  =="+ex.getMessage());
		return map;
	}
	
	/**
	 * 拦截自定义异常类的 ApplicationException 的异常信息
	 * @param ex
	 * @return
	 */
	@ResponseBody
	@ExceptionHandler(value=com.szxy.HelloWorld.exception.ApplicationException.class)
	public Map<String,Object> myException3(Exception ex){
		HashMap<String, Object> map = 
				new HashMap<String,Object>();
		map.put("error", "服务器繁忙！！1000  =="+ex.getMessage());
		return map;
	}
	
}

```

- 自定义异常类

```java 
public class ApplicationException extends Exception {

	public ApplicationException() {
		
	}
	public ApplicationException(String msg) {
		super(msg);
	}
	public ApplicationException(String msg,Throwable t) {
		super(msg,t);
	}
	
}

```

- 创建测试 HelloController.java 

```java
@RestController
public class HelloController {

	@Value("${my.msg2}")
	private String msg;
	
	@RequestMapping("/hello")
	public String hello() throws ApplicationException {
		/*
		 * String str = null; str.length();
		 * 
		 * int i = 1/0;
		 */
		throw new ApplicationException("出错啦 出错啦 ...");
		
		//return this.msg;
	}
	
}
```

### Spring Boot 监控状态 ###

#### 使用 Actuator 检查与监控 ####

- **在 pom 文件中 添加Actuator 的坐标**

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

- **在全局配置文件中关闭安全限制**

```properties
manager.security.enable=false;
```

#### 使用 可视化的监控报表 Spring  Boot Admin  ####

> codecentric’s Spring Boot Admin is a community project to manage and monitor your [Spring Boot](http://projects.spring.io/spring-boot/) ® applications. The applications register with our Spring Boot Admin Client (via HTTP) or are discovered using Spring Cloud ® (e.g. Eureka, Consul). The UI is just an AngularJs application on top of the Spring Boot Actuator endpoints.

- 搭建服务端，也就是一个 Spring Boot 项目

```http
https://start.spring.io/
```

- 访问 Spring Boot Admin 的首页

```http
https://github.com/codecentric/spring-boot-admin
```

- 修改 `pom.xml `文件 添加 `Spring Boot Admin `坐标

```xml
<dependencies>
    <dependency>
        <groupId>de.codecentric </groupId>
        <artifactId>spring-boot-admin-starter-server</artifactId>
        <version>1.5.7</version>
    </dependency>
</dependencies>
```

- 服务端  Spring Boot 启动类: 添加 `@EnableAdminServer` 注解

```java
@SpringBootApplication
@EnableAdminServer
public class SpringBootServerAdminApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootServerAdminApplication.class, args);
	}

}

```

- 搭建客户端，修改 `pom.xml `文件

```jxml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>1.5.7</version>
</dependency>
```

- 修改客户端 ` application.properties `文件

```properties
spring.boot.admin.url: http://localhost:8080  
management.security.enabled: false
```

http://codecentric.github.io/spring-boot-admin/1.5.7/

![](http://img.zwer.xyz/blog/20190805174251.png)



## RabiitMQ 实战 ##

> 消息中间件，即消息队列

### RabbitMQ 介绍 ###

> **RabbitMQ** 是实现了高级消息队列协议（AMQP）的开源消息代理软件（亦称面向消息的中间件）。RabbitMQ服务器是用[Erlang](https://baike.baidu.com/item/Erlang)语言编写的，而集群和故障转移是构建在[开放电信平台](https://baike.baidu.com/item/开放电信平台)框架上的。所有主要的编程语言均有与代理接口通讯的客户端[库](https://baike.baidu.com/item/库)。

### 安装 RabbitMQ ###

```shell
# 安装 Erlang
wget https://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm


#修改
rpm -Uvh erlang-solutions-1.0-1.noarch.rpm

#修改 primary.xml.gz 的 sha 的加密值 
cd /var/cache/yum/x86_64/6/erlang-solutions 
sha1sum primary.xml.gz
vim repomd.xml

#修改
<data type="primary"> <checksum type="sha">结果为 sha1sum 命令结果</checksum>

#安装 Erlang
yum install erlang

#安装完成后检查是否安装chengg
erl -version

#安装 RabbitMQ Server
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.5.1/rabbitmq-server-3.5.1-1.noarch.rpm

yum install rabbitmq-server-3.5.1-1.noarch.rpm
===============================================================================
#启动 RabbitMQ 6.1配置为守护进程随系统自动启动，root 权限下执行: 
chkconfig rabbitmq-server on 
#启动 rabbitMQ 服务器
/sbin/service rabbitmq-server start
# 
#安装 Web 管理界面插件 
#安装命令 
rabbitmq-plugins enable rabbitmq_management

#安装成功后会显示如下内容 
The following plugins have been enabled: 
mochiweb webmachine rabbitmq_web_dispatch amqp_client rabbitmq_management_agent rabbitmq_management Plugin configuration has changed. 
Restart RabbitMQ for changes to take effect.

#设置 RabbitMQ 远程 ip 登录这里我们以创建个 zwz 帐号，密码 123456 为例，创建一个账号并 支持远程 ip 访问。 
#创建账号
rabbitmqctl add_user zwz 123456 

#设置用户角色 
rabbitmqctl set_user_tags zwz administrator

#设置用户权限 
rabbitmqctl set_permissions -p "/" zwz ".*" ".*" ".*"

#设置完成后可以查看当前用户和角色(需要开启服务rabbitmqctl list_users 浏览器输入：#serverip:15672。其中 serverip 是 RabbitMQ-Server 所 在主机的 ip。
```

[RabbitMq 安装参考](https://juejin.im/post/5b97e9956fb9a05d37617a79)

[RabbitMQ 官网安装帮助](https://www.rabbitmq.com/install-rpm.html#downloads)

[解决安装 epel 源，安装后报错的问题](https://blog.51cto.com/jschu/1750177)

![](http://img.zwer.xyz/blog/20190806114237.png)



### 使用 RabbitMQ 的原因 ###

1. **同步变异步**

   ​	将服务之间由原来的同步改为异步调用

2. **解除紧耦合**

应用场景：流量秒杀

![](http://img.zwer.xyz/blog/20190806102130.png)



### 消息队列（中转站）基础 ###

- **provider** 

  消息生产者，负责发送消息

- **consumer** 

  消息消费者，负责接收消息并处理

- **没有使用消息队列时消息的传递方法**

![](http://img.zwer.xyz/blog/20190806102329.png)

- **使用消息队列后消息传递方法**

![](http://img.zwer.xyz/blog/20190806102342.png)

- **队列**

  队列就像存放了商品的仓库或者商店，是生产商品的工厂和购买商品的用户之间的中转站

- **队列里存储了什么？**

  在 rabbitMQ 中，**信息流**从你的应用程序出发，来到 RabbitMQ 的队列，所有信息可以只存储在一个队列中。队列可以存储很多信息，因为它基本上是一个无限制的缓冲区，前提是你的机器有足够的存储空间。 

- **队列和应用程序的关系？**

  多个生产者可以将消息发送到同一个队列中，多个消息者也可以只从同一个队列接收数据。 

### 编写 RabbitMQ 案例 ###

- 创建 Maven 项目，并修改 pom.xml 

```xml
<!-- Spring Boot RabbitMQ 消息队列启动器 -->
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
</dependency>
```

- 添加全局配置文件 application.properties:添加 RabbitMQ 配置

```properties
spring.application.name=

spring.rabbitmq.host=
spring.rabbitmq.port=
spring.rabbitmq.username=
spring.rabbitmq.password=
```

### RabbitMQ 原理 ###

![](http://img.zwer.xyz/blog/20190806150422.png)



![](http://img.zwer.xyz/blog/20190806154603.png)





### RabbitMQ 交换器 ###

#### Direct 交换器（发布订阅，完全匹配） ####

> 一个服务对应一个消息队列

![](http://img.zwer.xyz/blog/20190806165100.png)

**服务提供者**

- 修改 application.yml

```yml
server:
      port: 8080
      
spring:
      application:
                  name: rabbitmq-consumer
      rabbitmq:
               host: 192.168.170.134
               port: 5672
               username: zwz
               password: 123456
```

- 修改application.properties

```properties
mq.config.exchange=log-direct
mq.config.queue.info.routing.key=log.info.routing.key
mq.config.queue.error.routing.key=log.error.routing.key
```

- Sender.java

```java
/**
 * 消息发送者
 *
 */
@Component
public class Sender {

	@Autowired
	private RabbitTemplate rabbiteTemplate;
	@Value("${mq.config.exchange}")
	private String exchange;
	@Value("${mq.config.queue.error.routing.key}")
	private String key;
	
	/**
	 * 发送消息
	 * @param msg
	 */
	public void sendMsg(String msg) {
		/**
		 * 第一个参数表示 交换器的名称
		 * 第二个参数表示  路由键
		 * 第三个参数表示 消息，将要发送的消息
		 */
		this.rabbiteTemplate.convertAndSend(exchange,key,msg);
	}
	
}

```

**服务消息者**

- 修改 application.properties

```properties
mq.config.exchange=log-direct
mq.config.queue.info=log.info 
mq.config.queue.info.routing.key=log.info.routing.key
mq.config.queue.error=log.error 
mq.config.queue.error.routing.key=log.error.routing.key
```

- 修改 application.yml

```yml
server:
      port: 8080
      
spring:
      application:
                  name: rabbitmq-consumer
      rabbitmq:
               host: 192.168.170.134
               port: 5672
               username: zwz
               password: 123456

```

- InfoConsumer.java

```java
/**
 * 消息接收者 消息接收者
 * 
 * @RabbitListener bindings:绑定队列
 * 
 * @QueueBinding value:绑定队列的名称 
 *               exchange:配置交换器 
 * @Queue value:配置队列名称 
 *        autoDelete:是否是一个可删除的临时队列 
 * 
 * @Exchange value:为交换器起个名称  
 *           type:指定具体的交换器类型
 */
@Component
@RabbitListener(bindings = @QueueBinding(
  exchange = 
    @Exchange(value = "${mq.config.exchange}", type = ExchangeTypes.DIRECT),
	value = @Queue(value = "${mq.config.queue.info}", autoDelete = "true"),
		key = "${mq.config.queue.info.routing.key}")
)
public class InfoReceiver {

	/**
	 * 监听指定的消息队列
	 * 
	 * @param msg
	 */
	@RabbitHandler
	public void receiveMsg(String msg) {
		System.out.println("Info------"+msg);
	}
}

```

- ErrorConsumer.java

```java
/**
 * 消息接收者 消息接收者
 * 
 * @RabbitListener bindings:绑定队列
 * 
 * @QueueBinding value:绑定队列的名称 
 *               exchange:配置交换器 
 * @Queue value:配置队列名称 
 *        autoDelete:是否是一个可删除的临时队列 
 * 
 * @Exchange value:为交换器起个名称  
 *           type:指定具体的交换器类型
 */
@Component
@RabbitListener(bindings = @QueueBinding(
		exchange = 
    @Exchange(value = "${mq.config.exchange}", type = ExchangeTypes.DIRECT),
		value = @Queue(value = "${mq.config.queue.error}", autoDelete = "true"),
		key = "${mq.config.queue.error.routing.key}")
)
public class ErrorReceiver {

	/**
	 * 监听指定的消息队列
	 * 
	 * @param msg
	 */
	@RabbitHandler
	public void receiveMsg(String msg) {
		System.out.println("Error------"+msg);
	}
}

```

#### Topic 交换器（规则匹配） ####

> 一个服务对应多个消息队列，反之一个消息队列也可以对应多个服务
>
> 路由键采用 `* `通配符

![](http://img.zwer.xyz/blog/20190806165033.png)

**服务提供者**

- application.yml

```yml
server:
      port: 8080
      
spring:
      application:
                  name: rabbitmq-provider
      rabbitmq:
               host: 192.168.170.134
               port: 5672
               username: zwz
               password: 123456
```

- application.properties

```properties
mq.config.exchange=log-topic
```

- UserSender.java

```java
/**
 * 消息发送者
 *
 */
@Component
public class OrderSender {

	@Autowired
	private RabbitTemplate rabbiteTemplate;
	@Value("${mq.config.exchange}")
	private String exchange;
	
	/**
	 * 发送消息
	 * @param msg
	 */
	public void sendMsg(String msg) {
		/**
		 * 第一个参数表示 交换器的名称
		 * 第二个参数表示  路由键
		 * 第三个参数表示 消息，将要发送的消息
		 */
this.rabbiteTemplate.convertAndSend(
    exchange,"order.log.debug","order debug ..."+msg);
this.rabbiteTemplate.convertAndSend(
    exchange,"order.log.info","order info ..."+msg);
this.rabbiteTemplate.convertAndSend(
    exchange,"order.log.warn","order warn ..."+msg);
this.rabbiteTemplate.convertAndSend(
    exchange,"order.log.error","order error ..."+msg);
	}
	
}

```

- ProductSender.java

```java


/**
 * 消息发送者
 *
 */
@Component
public class ProductSender {

	@Autowired
	private RabbitTemplate rabbiteTemplate;
	@Value("${mq.config.exchange}")
	private String exchange;
	
	/**
	 * 发送消息
	 * @param msg
	 */
	public void sendMsg(String msg) {
		/**
		 * 第一个参数表示 交换器的名称
		 * 第二个参数表示  路由键
		 * 第三个参数表示 消息，将要发送的消息
		 */
		this.rabbiteTemplate.convertAndSend(
            exchange,"product.log.debug","product debug ..."+msg);
		this.rabbiteTemplate.convertAndSend(
            exchange,"product.log.info","product info ..."+msg);
		this.rabbiteTemplate.convertAndSend(
            exchange,"product.log.warn","product warn ..."+msg);
		this.rabbiteTemplate.convertAndSend(
            exchange,"product.log.error","product error..."+msg);
	}
	
}
	
```

- OrderSender.java

```java

/**
 * 消息发送者
 *
 */
@Component
public class OrderSender {

	@Autowired
	private RabbitTemplate rabbiteTemplate;
	@Value("${mq.config.exchange}")
	private String exchange;
	
	/**
	 * 发送消息
	 * @param msg
	 */
	public void sendMsg(String msg) {
		/**
		 * 第一个参数表示 交换器的名称
		 * 第二个参数表示  路由键
		 * 第三个参数表示 消息，将要发送的消息
		 */
		this.rabbiteTemplate.convertAndSend(
            exchange,"order.log.debug","order debug ..."+msg);
		this.rabbiteTemplate.convertAndSend(
            exchange,"order.log.info","order info ..."+msg);
		this.rabbiteTemplate.convertAndSend(
            exchange,"order.log.warn","order warn ..."+msg);
		this.rabbiteTemplate.convertAndSend(
            exchange,"order.log.error","order error ..."+msg);
	}
	
}
	
```

- Test 测试类

```java
RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class QueueTest {

	@Autowired
	private UserSender sender;
	@Autowired
	private ProductSender ProductSender;
	@Autowired
	private OrderSender orderSender;
	
	@Test
	public void test() throws Exception{
		int i = 2;
		while(i > 0) {
			Thread.sleep(1000);
			this.sender.sendMsg("user===== Hello  World RabbitMq!!!");
			this.ProductSender.sendMsg("product ===== Hello World RabbitMq!!!");
			this.orderSender.sendMsg("order ====Hello World RabbitMQ !!!");
			i--;
		}
	}
	
}
```

**服务消费者**

- application.yml

```yml
server:
      port: 8080
      
spring:
      application:
                  name: rabbitmq-provider
      rabbitmq:
               host: 192.168.170.134
               port: 5672
               username: zwz
               password: 123456
```

- application.properties

```properties
# 交换器
mq.config.exchange=log-topic
# 队列
mq.config.queue.info=log.info
mq.config.queue.error=log.error
mq.config.queue.logs=log.all
```

- InfoReceiver.java

```java
/**
 * 消息接收者 消息接收者
 * 
 * @RabbitListener bindings:绑定队列
 * 
 * @QueueBinding value:绑定队列的名称 
 *               exchange:配置交换器 
 				 key:路由键
 * @Queue value:配置队列名称 
 *        autoDelete:是否是一个可删除的临时队列 
 * 
 * @Exchange value:为交换器起个名称  
 *           type:指定具体的交换器类型
 */
@Component
@RabbitListener(bindings = @QueueBinding(
		exchange = @Exchange(value = "${mq.config.exchange}", type = ExchangeTypes.TOPIC),
		value = @Queue(value = "${mq.config.queue.info}", autoDelete = "true"),
		key = "*.log.info")
)
public class InfoReceiver {

	/**
	 * 监听指定的消息队列
	 * 
	 * @param msg
	 */
	@RabbitHandler
	public void receiveMsg(String msg) {
		System.out.println("Info------"+msg);
	}
}

```

- ErrorReceiver.java

```java
/*
 * 消息接收者 消息接收者
 * 
 * @RabbitListener bindings:绑定队列
 * 
 * @QueueBinding value:绑定队列的名称 
 *               exchange:配置交换器 
  				key:路由键	
 * @Queue value:配置队列名称 
 *        autoDelete:是否是一个可删除的临时队列 
 * 
 * @Exchange value:为交换器起个名称  
 *           type:指定具体的交换器类型
 */
@Component
@RabbitListener(bindings = @QueueBinding(
		exchange = @Exchange(value = "${mq.config.exchange}", type = ExchangeTypes.TOPIC),
		value = @Queue(value = "${mq.config.queue.error}", autoDelete = "true"),
		key = "*.log.error")
)
public class ErrorReceiver {

	/**
	 * 监听指定的消息队列
	 * 
	 * @param msg
	 */
	@RabbitHandler
	public void receiveMsg(String msg) {
		System.out.println("Error------"+msg);
	}
}

```

- AllReceiver.java

```java
/**
 * 消息接收者 消息接收者
 * 
 * @RabbitListener bindings:绑定队列
 * 
 * @QueueBinding value:绑定队列的名称 
 *               exchange:配置交换器 
 * @Queue value:配置队列名称 
 *        autoDelete:是否是一个可删除的临时队列 
 * 
 * @Exchange value:为交换器起个名称  
 *           type:指定具体的交换器类型
 */
@Component
@RabbitListener(bindings = @QueueBinding(
		exchange = @Exchange(value = "${mq.config.exchange}", type = ExchangeTypes.TOPIC),
		value = @Queue(value = "${mq.config.queue.logs}", autoDelete = "true"),
		key = "*.log.*")
)
public class AllReceiver {

	/**
	 * 监听指定的消息队列
	 * 
	 * @param msg
	 */
	@RabbitHandler
	public void receiveMsg(String msg) {
		System.out.println("All------"+msg);
	}
}

```

#### Fanout 交换器（广播） ####

> 只有交换器，没有路由键

![](http://img.zwer.xyz/blog/20190806194157.png)

**服务提供者**

- application.properties

```properties
mq.config.exchange=log-fanout
```

- application.yml

```yml
server:
      port: 8080
      
spring:
      application:
                  name: rabbitmq-provider
      rabbitmq:
               host: 192.168.170.134
               port: 5672
               username: zwz
               password: 123456
```

- Sender.java

```java
/**
 * 消息发送者
 *
 */
@Component
public class Sender {

	@Autowired
	private RabbitTemplate rabbiteTemplate;
	@Value("${mq.config.exchange}")
	private String exchange;
	
	
	/**
	 * 发送消息
	 * @param msg
	 */
	public void sendMsg(String msg) {
		/**
		 * 第一个参数表示 交换器的名称
		 * 第二个参数表示  路由键，不能写 null，否则就进入黑洞队列中
		 * 第三个参数表示 消息，将要发送的消息
		 */
		this.rabbiteTemplate.convertAndSend(exchange,"",msg);
	}
	
}

```

- 测试类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class QueueTest {

	@Autowired
	private Sender sender;
	
	@Test
	public void test() throws Exception{
		this.sender.sendMsg("Hello  World RabbitMq!!!");
	}
	
}

```

**服务消息者**

- application.properties

```properties
# 交换器
mq.config.exchange=log-fanout
# 队列
mq.config.queue.sms=log.sms 
mq.config.queue.push=log.push 

```

- application.yml

```yml
server:
      port: 8080
      
spring:
      application:
                  name: rabbitmq-consumer
      rabbitmq:
               host: 192.168.170.134
               port: 5672
               username: zwz
               password: 123456
```

- SmsReceiver.java

```java
/**
 * 消息接收者 消息接收者
 * 
 * @RabbitListener bindings:绑定队列
 * 
 * @QueueBinding value:绑定队列的名称 
 *               exchange:配置交换器 
 *               key: 路由器
 * @Queue value:配置队列名称 
 *        autoDelete:是否是一个可删除的临时队列 
 * 
 * @Exchange value:为交换器起个名称  
 *           type:指定具体的交换器类型
 */
@Component
@RabbitListener(bindings = @QueueBinding(
exchange = @Exchange(value = "${mq.config.exchange}", type = ExchangeTypes.FANOUT),
value = @Queue(value = "${mq.config.queue.sms}", autoDelete = "true")
		)
)
public class SmsReceiver {

	/**
	 * 监听指定的消息队列
	 * 
	 * @param msg
	 */
	@RabbitHandler
	public void receiveMsg(String msg) {
		System.out.println("Sms  ------"+msg);
	}
}

```

- PushReceiver.java

```java
/**
 * 消息接收者 消息接收者
 * 
 * @RabbitListener bindings:绑定队列
 * 
 * @QueueBinding value:绑定队列的名称 
 *               exchange:配置交换器 
 *               key: 路由键
 * @Queue value:配置队列名称 
 *        autoDelete:是否是一个可删除的临时队列 
 * 
 * @Exchange value:为交换器起个名称  
 *           type:指定具体的交换器类型
 */
@Component
@RabbitListener(bindings = @QueueBinding(
exchange = @Exchange(value = "${mq.config.exchange}", type = ExchangeTypes.FANOUT),
value = @Queue(value = "${mq.config.queue.push}", autoDelete = "true")
		)
)
public class PushReceiver {

	/**
	 * 监听指定的消息队列
	 * 
	 * @param msg
	 */
	@RabbitHandler
	public void receiveMsg(String msg) {
		System.out.println("push------"+msg);
	}
}

```

### RabbitMQ 松耦合 ###

![](http://img.zwer.xyz/blog/20190806194417.png)



### RabbitMQ  消息处理 ###

| 注解      | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| @Queue    | autoDelete：当所有消费客户端连接断开后，是否自动删除队列，true：删除，false 不删除 |
| @Exchange | autoDelete：删除交换器，true：删除，false： 不删除           |

### RabbitMQ 消息确证 ACK[重点]  ###

```properties
# 解决 ACK 反馈
#开启重试 
spring.rabbitmq.listener.retry.enabled=true 
#重试次数，默认为 3 次 
spring.rabbitmq.listener.retry.max-attempts=5
```



## Eureka 注册中心 ##

> Eureka 是 Netflix 开发的服务发现组件，本身是一个基于 REST 的服务。
>
> Spring Cloud它集成在其子项目 spring-cloud-netflix 中，以实现 Spring Cloud 的服务注册与服务发现，同时还提供了负载均衡、故障转移等能力。

### 服务注册中心 ###

> 服务注册中心是服务实现服务化管理的核心组件,类似于目录服务的作用,主要用来存储服务信息，
>
> 譬如提供者 url 串、路由信息等。服务注册中心是 SOA 架构中最基础的设施之一。

**作用:**

1. 服务的注册 2. 服务的发现

**常见的服务注册中心：**

Dubbo 的注册中心 Zookeeper

Spring Cloud 的注册中心  Eureka

**服务注册中心解决什么问题？**

1. 服务管理

2. 服务间的依赖关系

### Eureka 注册中心三种角色 ###

| 角色                                     | 作用                                                 |
| ---------------------------------------- | ---------------------------------------------------- |
| Eureka Server                            | 通过 Register 、 Get、Renew 等接口提供服务注册和发现 |
| Application Service （Service Provider） | 服务提供方把自身的服务实例注册到 Euraka Server 中    |
| Application Client（Service Consumer）   | 服务调用方通过 Euraka Server 获取服务列表，消费服务  |

### 搭建 Eureka 单机版 ###

- 创建 Maven 项目（jar），并修改 pom 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<artifactId>SpringBoot-Nureka-Server</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>HelloWorld</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>
		<!-- Spring Boot 监控启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- Spring Boot Test 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<!-- Spring Boot 监控客户端启动类 -->
		<dependency>
			<groupId>de.codecentric</groupId>
			<artifactId>spring-boot-admin-starter-client</artifactId>
			<version>1.5.7</version>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

- 添加 applicationContext.properties：取消 Eureka 自己注册服务作为服务以及获取服务列表

```properties
spring.application.name=eureka-server
server.port=8761

#
eureka.cliento.registerWithEureka=false  

eureka.client.fetchRegistry=false
```

- SpringBoot 启动类: 添加 @EnableEurekaServer 注解

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}

}
```

### 搭建 Euraka 集群版 ###

- 创建 Maven 项目（jar），并修改 pom 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<artifactId>SpringBoot-Nureka-Server</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>HelloWorld</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>
		<!-- Spring Boot 监控启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- Spring Boot Test 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<!-- Spring Boot 监控客户端启动类 -->
		<dependency>
			<groupId>de.codecentric</groupId>
			<artifactId>spring-boot-admin-starter-client</artifactId>
			<version>1.5.7</version>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

- 添加 application-nureka1.properties

```properties
spring.application.name=eureka-server
server.port=8761

#设置 eureka 实例名称，与配置文件的变量为主
eureka.instance.hostname=eureka1

#设置服务注册中心地址，指向另一个注册中心
eureka.client.serviceUrl.defaultZone=http://eureka2:8761/eureka/
```

- 添加 application-nureka2.properties

```properties
spring.application.name=eureka-server
server.port=8761

#设置 eureka 实例名称，与配置文件的变量为主
eureka.instance.hostname=eureka2

#设置服务注册中心地址，指向另一个注册中心
eureka.client.serviceUrl.defaultZone=http://eureka1:8761/eureka/
```

- 添加 logback 日志配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
 <configuration>
<!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->  
    <property name="LOG_HOME" value="${catalina.base}/logs/" />  
    <!-- 控制台输出 -->   
    <appender name="Stdout" class="ch.qos.logback.core.ConsoleAppender">
       <!-- 日志输出编码 -->  
        <layout class="ch.qos.logback.classic.PatternLayout">   
             <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符--> 
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n   
            </pattern>   
        </layout>   
    </appender>   
    <!-- 按照每天生成日志文件 -->   
    <appender name="RollingFile"  class="ch.qos.logback.core.rolling.RollingFileAppender">   
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/server.%d{yyyy-MM-dd}.log</FileNamePattern>   
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>   
        <layout class="ch.qos.logback.classic.PatternLayout">  
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符--> 
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n   
            </pattern>   
       </layout> 
        <!--日志文件最大的大小-->
       <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
         <MaxFileSize>10MB</MaxFileSize>
       </triggeringPolicy>
    </appender>     

    <!-- 日志输出级别 -->
    <root level="info">   
        <appender-ref ref="Stdout" />   
        <appender-ref ref="RollingFile" />   
    </root> 



<!--日志异步到数据库 -->  
<!--     <appender name="DB" class="ch.qos.logback.classic.db.DBAppender">
        日志异步到数据库 
        <connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">
           连接池 
           <dataSource class="com.mchange.v2.c3p0.ComboPooledDataSource">
              <driverClass>com.mysql.jdbc.Driver</driverClass>
              <url>jdbc:mysql://127.0.0.1:3306/databaseName</url>
              <user>root</user>
              <password>root</password>
            </dataSource>
        </connectionSource>
  </appender> -->

</configuration>
```

- 将项目打包，将 jar 包上传到 Linux 上

```java
# 创建目录，将 jar 包放在该目录下
mkdir /usr/local/eureka
# 编写 server.sh 脚本，用于启动和停止 eureka 注册中心

#赋予 server.sh 权限
chmod  -R 755 server.sh

# 修改 hosts，添加本地域名
vim /etc/hosts
192.168.170.128 eureka1
192.168.170.129 eureka1


```

[解决 window 下编写 shell 脚本，到Linux 下运行报错](https://blog.csdn.net/m53931422/article/details/42713097)

### 在高可用集群下搭建 Provider ###

- 创建 Maven 项目（jar），并修改 pom 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<artifactId>SpringBoot-Nureka-Server</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>HelloWorld</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- Spring Boot 监控启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- Spring Boot Test 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<!-- Spring Boot 监控客户端启动类 -->
		<dependency>
			<groupId>de.codecentric</groupId>
			<artifactId>spring-boot-admin-starter-client</artifactId>
			<version>1.5.7</version>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

- 添加 application.properties

```properties
spring.application.name=eureka-provider
server.port=9090

eureka.client.serviceUrl.defaultZone=http://eureka1:8761/eureka/,http://eureka2:8761/eureka/

```

- SpringBoot  启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class EurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}

}

```

- UserController.java

```java
@RestController
public class UserController {

	@RequestMapping("/list")
	public Object getUsers() {
		List<User> userList = new ArrayList<>();
		userList.add(new User(1,"tom",20));
		userList.add(new User(2,"jerry",23));
		userList.add(new User(3,"lucy",22));
		return userList;
	}
	
}

```

### 在高可用集群下搭建 Consumer ###

- 创建 Maven 项目（jar）,并修改 pom 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<artifactId>SpringBoot-eureka-consumer</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>SpringBoot-eureka-consumer</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- Spring Boot 监控启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- Spring Boot Test 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<!-- Spring Boot 监控客户端启动类 -->
		<dependency>
			<groupId>de.codecentric</groupId>
			<artifactId>spring-boot-admin-starter-client</artifactId>
			<version>1.5.7</version>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

- 添加 application.properties

```properties
spring.application.name=eureka-consumer
server.port=9091

eureka.client.serviceUrl.defaultZone=http://eureka1:8761/eureka/,http://eureka2:8761/eureka/

```

- UserServiceImpl.java

```java
@Service
public class UserServiceImpl implements UserService{

	// ribbon 负载均衡器
	@Autowired
	private  LoadBalancerClient loadBalancerClient;
	
	@Override
	public List<User> getUsers() {
		// 选择调用服务的名称
		//  ServiceInstance 对象封装了服务的 IP、端口
		ServiceInstance si = 
				this.loadBalancerClient.choose("eureka-provider");
		//拼接服务URL
		StringBuilder sb = new StringBuilder();
		sb.append("http://").append(si.getHost()).
					append(":").append(si.getPort()).append("/list");
		//创建 RestTempate 对象，发送 URL 请求，获取响应结果
		RestTemplate rt = new RestTemplate();
		//请求类型
		ParameterizedTypeReference<List<User>> type = new  ParameterizedTypeReference<List<User>>() {};
		//RequestEntity 封装了请求实体
		ResponseEntity<List<User>> entity = 	rt.exchange(sb.toString(),HttpMethod.GET,null,type);
		List<User> userList = entity.getBody();
		return userList;
	}

}
```

- UserController.java

```java
@RestController
public class UserController {

	@Autowired
	private UserService userService;
	
	@RequestMapping("/list2")
	public Object getUsers() {
		List<User> userList = 
				this.userService.getUsers();
		return userList;
	}
	
}

```

### Eureka 架构图 ###

![](http://img.zwer.xyz/blog/20190807154210.png)

Register(服务注册)：把自己的 IP 和端口注册给 Eureka。

Renew(服务续约)：发送心跳包，每 30 秒发送一次。告诉 Eureka 自己还活着。

Cancel(服务下线)：当 provider 关闭时会向 Eureka 发送消息，把自己从服务列表中删除。防止 consumer 调用到不存在的服务。

Get Registry(获取服务注册列表)：获取其他服务列表。

Replicate(集群中数据同步)：eureka 集群中的数据复制与同步。

Make Remote Call(远程调用)：完成服务的远程调用。

### CAP 定理[重点] ###

> CAP 原则又称 CAP 定理，指的是在一个分布式系统中**，Consistency**（一致性）、**Availability**（可用性）、**Partition tolerance**（分区容错性），三者不可兼得。CAP 由 Eric Brewer 在 2000 年 PODC 会议上提出。该猜想在提出两年后被证明成立，成为我们熟知的 CAP 定理

| 原则                | 解释                                                         |
| ------------------- | ------------------------------------------------------------ |
| Consistency         | 也叫做数据原子性<br/>系统在执行某项操作后仍然处于一致的状态。在分布式系统中，更新操作执行成功后所有的用户都应该读到最新的值，这样的系统被认为是具有强一致性的。等同于所有节点访问 |
| Availability        | 每一个操作总是能够在一定的时间内返回结果，这里需要注意的是"一定时间内"和"返回结果"。一定时间内指的是在可以**容忍的范围内**返回结果，结果可以是成功或者是失败。 |
| Partition tolerance | 在网络分区的情况下，被分隔的节点仍能正常对外提供服务(分布式集群，数据被分布存储在不同的服务器上，无论什么情况，服务器都能正常被访问) |

![](http://img.zwer.xyz/blog/20190807163050.png)

### 	Eureka 与 Zookeeper 注册中心对比 ###

![](http://img.zwer.xyz/blog/20190807163353.png)

### 自我保护 ###

| 自我保护                                                     |
| ------------------------------------------------------------ |
| 1. **自我保护的条件**<br/>一般情况下，微服务在 Eureka 上注册后，会**每 30 秒发送心跳包**，Eureka 通过心跳来判断服务时候健康，同时会定期删除超过 90 秒没有发送心跳服务。<br/>2. 有两种情况会导致 Eureka Server 收不到微服务的心跳<br/>a.是微服务自身的原因<br/>b.是微服务与 Eureka 之间的网络故障<br/>通常(微服务的自身的故障关闭)只会导致个别服务出现故障，一般不会出现大面积故障，而(网络故障)通常会导致 Eureka Server 在短时间内无法收到大批心跳。考虑到这个区别，Eureka 设置了一个阀值，当判断挂掉的服务的数量超过阀值时，Eureka Server 认为很大程度上出现了网络故障，将不再删除心跳过期的服务。<br/>3. 那么这个阀值是多少呢？<br/>15 分钟之内是否低于 85%；<br/>Eureka Server 在运行期间，会统计心跳失败的比例在 15 分钟内是否低于 85%<br/>这种算法叫做 Eureka Server 的自我保护模式。 |
| 自我保护的原因                                               |
| 1. 因为同时保留"好数据"与"坏数据"总比丢掉任何数据要更好，当网络故障恢复后，这个 Eureka 节点会退出"自我保护模式"。<br/> 2. Eureka 还有客户端缓存功能(也就是微服务的缓存功能)。即便 Eureka 集群中所有节点都宕机失效，微服务的 Provider 和 Consumer都能正常通信。<br/>3. 微服务的负载均衡策略会自动剔除死亡的微服务节点。 |

**关闭保护**

```properties
#关闭自我保护:true 为开启自我保护，false 为关闭自我保护
eureka.server.enableSelfPreservation=false
#清理间隔(单位:毫秒，默认是 60*1000)
eureka.server.eviction.interval-timer-in-ms=6000
```

### Eureka 优雅停服 ###

- 不需要在 Eureka Service 中配置关闭自我保护
- 需要在服务中添加 actuator.jar 

```properties
<!-- Eureka 服务端启动类包括 actuator.jar  -- >
<dependency> 
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

- 修改 application.properties

```properties
#启用 shutdown
endpoints.shutdown.enabled=true
#禁用密码验证
endpoints.shutdown.sensitive=false
```

- **发送 Post ，到本地服务**

```http
http://localhost:9090/shutdown
```

注意：这里不能发送 Get 请求

```java
/**
 * 优雅关服
 *
 */
public class ShutdownProvider {
		
	public static void main(String[] args) {
		String url = "http://localhost:9090/shutdown";
		HttpClientUtil.doPost(url);
	}
}
```

HttpClientUtil.java:

```java
package com.szxy.shutdown;

import java.io.IOException;
import java.net.URI;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

import org.apache.http.NameValuePair;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;

public class HttpClientUtil {

	public static String doGet(String url, Map<String, String> param) {

		// 创建Httpclient对象
		CloseableHttpClient httpclient = HttpClients.createDefault();

		String resultString = "";
		CloseableHttpResponse response = null;
		try {
			// 创建uri
			URIBuilder builder = new URIBuilder(url);
			if (param != null) {
				for (String key : param.keySet()) {
					builder.addParameter(key, param.get(key));
				}
			}
			URI uri = builder.build();

			// 创建http GET请求
			HttpGet httpGet = new HttpGet(uri);

			// 执行请求
			response = httpclient.execute(httpGet);
			// 判断返回状态是否为200
			if (response.getStatusLine().getStatusCode() == 200) {
				resultString = EntityUtils.toString(response.getEntity(), "UTF-8");
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				if (response != null) {
					response.close();
				}
				httpclient.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return resultString;
	}

	public static String doGet(String url) {
		return doGet(url, null);
	}

	public static String doPost(String url, Map<String, String> param) {
		// 创建Httpclient对象
		CloseableHttpClient httpClient = HttpClients.createDefault();
		CloseableHttpResponse response = null;
		String resultString = "";
		try {
			// 创建Http Post请求
			HttpPost httpPost = new HttpPost(url);
			// 创建参数列表
			if (param != null) {
				List<NameValuePair> paramList = new ArrayList<>();
				for (String key : param.keySet()) {
					paramList.add(new BasicNameValuePair(key, param.get(key)));
				}
				// 模拟表单
				UrlEncodedFormEntity entity = new UrlEncodedFormEntity(paramList,"utf-8");
				httpPost.setEntity(entity);
			}
			// 执行http请求
			response = httpClient.execute(httpPost);
			resultString = EntityUtils.toString(response.getEntity(), "utf-8");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				response.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}

		return resultString;
	}

	public static String doPost(String url) {
		return doPost(url, null);
	}
	
	public static String doPostJson(String url, String json) {
		// 创建Httpclient对象
		CloseableHttpClient httpClient = HttpClients.createDefault();
		CloseableHttpResponse response = null;
		String resultString = "";
		try {
			// 创建Http Post请求
			HttpPost httpPost = new HttpPost(url);
			// 创建请求内容
			StringEntity entity = new StringEntity(json, ContentType.APPLICATION_JSON);
			httpPost.setEntity(entity);
			// 执行http请求
			response = httpClient.execute(httpPost);
			resultString = EntityUtils.toString(response.getEntity(), "utf-8");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				response.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}

		return resultString;
	}
	
	public static void main(String[] args) {
		String url ="http://127.0.0.1:9090/shutdown";
		//该url必须要使用dopost方式来发送
		HttpClientUtil.doPost(url);
	}
}
```



### Eureka 安全认证 ###

> 即用户 Eureka Server 注册中心，必须要输入用户名和密码，通过认证后，才可以访问 Eureka 注册中

#### Eureka Server 端 ####

- 修改 pom.xml 文件

```xml
<!-- Eureka 安全认证 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- application-eureka1.properties

```properties
spring.application.name=eureka-server
server.port=8761

#设置 eureka 实例名称，与配置文件的变量为主
eureka.instance.hostname=eureka1

#设置服务注册中心地址，指向另一个注册中心
eureka.client.serviceUrl.defaultZone=http://user:123456@eureka2:8761/eureka/

#开启 http basic 的安全认证
security.basic.enabled=true 
security.user.name=user
security.user.password=123456

#关闭自我保护:true 为开启自我保护，false 为关闭自我保护
#eureka.server.enableSelfPreservation=false
#清理间隔(单位:毫秒，默认是 60*1000)
#eureka.server.eviction.interval-timer-in-ms=6000
```

- application-eureka2.properties

```properties
spring.application.name=eureka-server
server.port=8761

#设置 eureka 实例名称，与配置文件的变量为主
eureka.instance.hostname=eureka2

#设置服务注册中心地址，指向另一个注册中心
eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/

#开启 http basic 的安全认证
security.basic.enabled=true 
security.user.name=user
security.user.password=123456

#关闭自我保护:true 为开启自我保护，false 为关闭自我保护
#eureka.server.enableSelfPreservation=false
#清理间隔(单位:毫秒，默认是 60*1000)
#eureka.server.eviction.interval-timer-in-ms=6000
```

修改完以上配置文件，将 Maven 项目重新打包，重新部署到 Linux 服务器上。

#### Eureka Client 端 ####

- application.properties

```properties
spring.application.name=eureka-provider
server.port=9090

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

# 启用 shutdown 
endpoints.shutdown.enabled=true
# 禁用密码验证
endpoints.shutdown.sensitive=false
```

注意： Eureka Server 之间通信以及Eureka Client 与 Euraka Server 之间都需要在 URL 上添加用户名和密码

填写 URL 的用户名和密码的格式如下：

```http
http://用户名:密码@host:port/eureka
```



---------------------

以下的 Spring Cloud 的高级部分

-----

## Ribbon 负载均衡 ##

>  Ribbo 是一个基于 TCP 和 HTTP 的客户端负载均衡工具，内嵌于 Spring Cloud 中，并默认提供了负载均衡算法，如：轮询法、随机法，甚至可以自定义负载均衡算法。
>
> Ribbo 的作用：解决微服务的负载均衡问题

### 负载均衡的分类 ###

1. 集中式负载均衡

   即在 consumer 和 provider 之间使用独立的负载均衡设施(可以是硬件，如 F5, 也可以是软件，如 nginx), 由该设施负责把 访问请求 通过某种策略转发 至 provider；

2. 进程内负载均衡

   将负载均衡逻辑集成到 consumer，consumer 从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的 provider。 

Ribbon 就属于后者，它只是一个类库，集成于 consumer 进程，consumer 通过它来获取到 provider 的地址。

![](http://img.zwer.xyz/blog/20190807195250.png)

### Ribbon 入门案例 ###

> Ribbo 默认采用轮询负载均衡算法寻找 Application Service 服务提供端
>



### 常见的 Ribbo 负载均衡算法 ###

![](http://img.zwer.xyz/blog/20190807203510.png)

![](http://img.zwer.xyz/blog/20190807203531.png)

 	![](http://img.zwer.xyz/blog/20190807203613.png)



### 更换负载均衡策略 ###

1. **通过修改代码更换负载均衡策略**

```java
SpringBootApplication
@EnableEurekaClient
public class ConsumerApplication {
	
	/**
	 * 创建方法，用于更换默认的轮询算法
	 * @param args
	 */
	  @Bean 
	  public RandomRule createRandomRule() {
      	 return new RandomRule(); 
      }
	 
	
	  public static void main(String[] args) {
	 	 SpringApplication.run(ConsumerApplication.class, args);
	  }

}

```

2. **修改配置文件的方式**

```properties
#设置负载均衡策略 eureka-provider 为调用的服务的名称
eureka-provider.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule



```

注意：不同的服务提供者配置的前缀不同，但是都 ` ribbon.NFLoadBalancerRuleClassName ` 要保持一致

### Ribbon 直连 ###

> 用于调试，直接连接指定的服务提供者

- 创建项目，并修改 pom.xml 文件

  剔除掉存在 eureka 启动类和添加及 Robbin 坐标

```xml
<!-- 
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
 -->
<!-- ribbon 坐标 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
```

- 修改 `application.properties`

```properties
#禁用 
eurekaribbon.eureka.enabled=false
#指定具体的服务实例清单，若指定多个 URL用逗号分隔，并采用轮询策略
eureka-provider.ribbon.listOfServers=192.168.170.128:9090
```

## Feign 声明式服务调用 ##

> 声明式服务调用
>
> 是一种声明式、模板化的 HTTP 客户端（仅在 Consumer 中使用）

![](http://img.zwer.xyz/blog/20190808094044.png)

### Feign 项目案例 ###

> 这里以 Ego 项目中商品服务为例,实现商品的基础操作。
>
> 即一个微服务只负责一件任务

#### 创建 Product-Service 项目 ####

- 创建 Maven 工程，并修改 pom 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<artifactId>SpringCloud-Product-Service</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- SpringCloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 客户端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- Product pojo -->
		<dependency>
			<groupId>com.szxy</groupId>
			<artifactId>SpringCloud-Product-pojo</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
	</dependencies>
</project>
```

- ProductService.java：在 Service 接口中也可以使用 @RequestMapping 注解

```java
@RequestMapping("/product")
public interface ProductService {

	/**
	 *   获取所有商品 
	 * @return
	 */
	@RequestMapping(value="/all",method=RequestMethod.GET)
	public List<Product> getAllProducts();
	
	/**
	 *    获取指定商品
	 *   注意：这里的 @RequestParam 注解不可省略，必须为参数起别名 
	 */
	@RequestMapping(value="/getProductById",method=RequestMethod.GET)
	public Product getProductById(@RequestParam("id") Integer id);
	
	/**
	 * 添加商品  方式一：通过 Get 方式
	 * 注意：这里的 @RequestParam 注解不可省略，必须为参数起别名 
	 */
	@RequestMapping(value="/addProduct",method=RequestMethod.GET)
	public Product addProduct(@RequestParam("id") Integer id,@RequestParam("name") String name);
	
	
	/**
	* 添加商品  方式二：通过 POST 方式
	*  这里的 @RequestBody  注解不可省略
	*  因为 Feign 是通过接收 JONS 格式数据  
	* consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
   	*
	*/
	@RequestMapping(value="/addProduct2",method=RequestMethod.POST,consumes=MediaType.APPLICATION_JSON_VALUE)
	public Product addProduct2(@RequestBody Product product);
	
	/**
	 * 添加商品  方式一：通过 Get 方式
	 * 注意：这里的 @RequestParam 注解不可省略，必须为参数起别名 
	 */
	@RequestMapping(value="/addProduct3",method=RequestMethod.GET,consumes=MediaType.APPLICATION_JSON_VALUE)
	public Product addProduct3(@RequestBody Product product);
	
}

```

#### 创建 Product-Pojo 项目 ####

- Product.java

```java
public class Product  implements Serializable{
	private Integer id;
	private String name;
	
    //省略 ...

}
```

#### 创建 Product-Product 项目 ####

- 创建 Maven 工程，并修改 pom 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.szxy</groupId>
	<artifactId>SpringCloud-ego-Product-Provider</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- Ego-Product-Service 服务 API -->
		<dependency>
			<groupId>com.szxy</groupId>
			<artifactId>SpringCloud-Product-Service</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

- ProductContoller.java:实现了 ProductService 接口

```java
/**
 *  实现 ProductService 接口
 *
 */
@RestController
public class ProductController implements ProductService{
	
	/**
	 * 查询所有商品  
	 */
	@Override
	public List<Product> getAllProducts() {
		List<Product> list = new ArrayList<Product>();
		list.add(new Product(1,"笔记本 HP 战66 "));
		list.add(new Product(2,"智能手机 小米 Note3"));
		list.add(new Product(3,"显示屏 AOC"));
		list.add(new Product(4,"鼠标 牧马人"));
		return list;
	}
	
	/**
	 * 查询指定商品
	 */
	@Override
	public Product getProductById(Integer id) {
		return new Product(id,"娃哈哈牌矿泉水"+id);
	}
	
	/**
	 *    添加商品
	 */
	@Override
	public Product addProduct(Integer id, String name) {
		return new Product(id,name);
	}

	@Override
	public Product addProduct2(@RequestBody Product product) {
		return product;
	}

	@Override
	public Product addProduct3(@RequestBody Product product) {
		return product;
	}
	
}

```

- Spring Boot 启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class ProviderApp {
	
	public static void main(String[] args) {
		SpringApplication.run(ProviderApp.class, args);
	}	
	
}

```

#### 创建 Product-Consumer 项目 ####

- 创建 Maven 工程，并修改 pom 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.szxy</groupId>
	<artifactId>SpringCloud-ego-Product-Consumer</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- Feign 启动器 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-feign</artifactId>
		</dependency>
		<!-- Ego-Product-Service 服务 API -->
		<dependency>
			<groupId>com.szxy</groupId>
			<artifactId>SpringCloud-Product-Service</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
		<!-- 使用Apache HttpClient替换Feign原生httpURLConnection -->
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
		</dependency>
		<dependency>
			<groupId>com.netflix.feign</groupId>
			<artifactId>feign-httpclient</artifactId>
			<version>8.17.0</version>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

- ProductConsumerService.java

```java
package com.szxy.ego.product.consumer.service;

import org.springframework.cloud.netflix.feign.FeignClient;

import com.szxy.ego.service.ProductService;

/**
 * @FeignClien 只作用于 Consumer 端
 * 属性 name 指定那个服务的接口实现类 
 */
@FeignClient(name="ego-product-provider")
public interface ProductConsumerService extends ProductService{

}

```

- ProductConsumerProvider.java

```java
@RestController
@RequestMapping("/product")
public class ProductConsumerController {

	@Autowired
	private ProductService productService;
	
	/**
	 * 获取所有商品信息
	 * @return
	 */
	@RequestMapping(value="/getall",method=RequestMethod.GET)
	public List<Product> getAll() {
		return this.productService.getAllProducts();
	}
	
	/**
	 * 获取指定商品
	 * @return
	 */
	@RequestMapping(value="/get/{id}",method=RequestMethod.GET)
	public Product getById(@PathVariable Integer id) {
		return this.productService.getProductById(id);
	}
	
	/**
	 * 添加商品 ，通过 Get 方式
	 * @param product
	 * @return
	 */
	@RequestMapping(value="/add",method=RequestMethod.GET)
	public Product addProduct(Product product) {
		return this.productService.addProduct(product.getId(), product.getName());
	}
	
	/**
	 * 添加商品 ，通过 POST 方式
	 * @param product
	 * @return
	 */
	@RequestMapping(value="/add2",method=RequestMethod.GET)
	public Product addProduct2(Product product) {
		return this.productService.addProduct2(product);
	}
	
	/**
	 * 添加商品  基于 HttpClient 的 Get 方式
	 */
	@RequestMapping(value="/add3",method=RequestMethod.GET)
	public Product addProduct3(Product product) {
		return this.productService.addProduct3(product);
	}
	
}

```

- Spring Boot 启动类

```java
/**
 *  启用 Feign 
 * 需要添加  @EnableFeignClients 和 @EnableDiscoveryClient 注解 
 *
 */
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
public class ConsumerHttpClientApp {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerHttpClientApp.class, args);
	}
	
}

```

#### 传递参数（这里未使用 HttpClient  ） ####

- 传递单一参数

  注意：在 服务端 Service 中接收参数采用 @RequestParam 注解

```java
/**
	 *    获取指定商品
	 *   注意：这里的 @RequestParam 注解不可省略，必须为参数起别名 
	 */
@RequestMapping(value="/getProductById",method=RequestMethod.GET)
public Product getProductById(@RequestParam("id") Integer id);
	
```

- 传递多个参数

  在服务端 Service 中使用 Get 方法接收参数，若是 pojo 类型，需要将属性平铺，一个一个接收。

  在服务端 Service 中使用 POST 方法接收参数，使用 @RequestBody 注解

```java

/**
	 * 添加商品  方式一：通过 Get 方式
	 * 注意：这里的 @RequestParam 注解不可省略，必须为参数起别名 
	 */
@RequestMapping(value="/addProduct",method=RequestMethod.GET)
public Product 
	addProduct(@RequestParam("id") Integer id,@RequestParam("name") String name);

/**
	* 添加商品  方式二：通过 POST 方式
	*  这里的 @RequestBody  注解不可省略
	*  因为 Feign 是通过接收 JONS 格式数据  
	* consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
   	*
	*/
@RequestMapping(value="/addProduct2",method=RequestMethod.POST,
                consumes=MediaType.APPLICATION_JSON_VALUE)
public Product addProduct2(@RequestBody Product product);
	
```


### Feign 性能优化 ###

#### Gzip 介绍 ####

gzip 介绍：gzip 是一种**数据格式**，采用用 deflate 算法压缩 data；gzip 是一种流行的文件压缩算法，应用十分广泛，尤其是在 Linux 平台。gzip 能力：**当 Gzip 压缩到一个纯文本文件时，效果是非常明显的，大约可以减少 70％以上的文件大小**。gzip 作用：网络数据经过压缩后实际上降低了网络传输的字节数，最明显的好处就是可以加快网页加载的速度。网页加载速度加快的好处不言而喻，除了节省流量，改善用户的浏览体验外，另一个潜在的好处是 Gzip 与搜索引擎的抓取工具有着更好的关系。例如 Google就可以通过直接读取 gzip 文件来比普通手工抓取 更快地检索网页。

#### HTTP 协议关于压缩传输的规定 ####

第一：客户端向服务器请求中带有：Accept-Encoding:gzip, deflate 字段，向服务器表示，客户端支持的压缩格式（gzip 或者 deflate)，如果不发送该消息头，服务器是不会压缩的。

第二：服务端在收到请求之后，如果发现请求头中含有 Accept-Encoding 字段，并且支持该类型的压缩，就对响应报文压缩之后返回给客户端，并且携带 Content-Encoding:gzip 消息头，表示响应报文是根据该格式压缩过的。

第三：客户端接收到请求之后，先判断是否有 Content-Encoding 消息头，如果有，按该格式解压报文。否则按正常报文处理。

#### Gzip 优化案例 ####

> 只需修改 application.properties 文件即可
>

1. **只配置 Consumer 通过 Feign 到 Provider 的请求与相应的 Gzip 压缩压缩**

```properties
#-----------------------------feign gzip
#配置请求 GZIP 压缩
feign.compression.request.enabled=true
#配置响应 GZIP 压缩
feign.compression.response.enabled=true
#配置压缩支持的 MIME TYPE
feign.compression.request.mime-types=text/xml,application/xml,application/json
#配置压缩数据大小的最小阀值，默认 2048
feign.compression.request.min-request-size=512
```

2. **对客户端浏览器的请求以及 Consumer 对 provider 的请求与响应做 Gzip 压缩**

```properties
#-----------------------------spring boot gzip
#是否启用压缩
server.compression.enabled=true
server.compression.mimetypes=application/json,application/xml,text/html,text/xml,text/plain
```

![](http://img.zwer.xyz/blog/20190808145113.png)

#### 采用 HTTP 连接池原理 ####

a. 两台服务器建立 http 连接的过程是很复杂的一个过程，涉及到多个数据包的交换，并且也很耗时间。

b. Http 连接需要的 3 次握手 4 次分手开销很大，这一开销对于大量的比较小的 http 消息来说更大。

**优化方案**

a. 如果我们直接采用 http 连接池，节约了大量的 3 次握手 4 次分手；这样能大大提升吞吐率。

b. feign 的 http 客户端支持 3 种框架；HttpURLConnection、httpclient、okhttp；默认是HttpURLConnection。

c. 传统的 HttpURLConnection 是 JDK 自带的，并不支持连接池，如果要实现连接池的机制，还需要自己来管理连接对象。对于网络请求这种底层相对复杂的操作，如果有可用的其他方案，也没有必要自己去管理连接对象。

d. HttpClient 相比传统 JDK 自带的 HttpURLConnection，它封装了访问 http 的请求头，参数，内容体，响应等等；它不仅使客户端发送 HTTP 请求变得容易，而且也方便了开发人员测试接口（基于 Http 协议的），即提高了开发的效率，也方便提高代码的健壮性；另外高并发大量的请求网络的时候，还是用“连接池”提升吞吐量。

#### 采用 HTTPClient   ####

- 在服务消费端 Consumer 项目中修改 pom 文件

```xml
<!-- 使用Apache HttpClient替换Feign原生httpURLConnection -->
<dependency> 
    <groupId>org.apache.httpcomponents</groupId> 
    <artifactId>httpclient</artifactId>
</dependency>
<dependency>
    <groupId>com.netflix.feign</groupId> 
    <artifactId>feign-httpclient</artifactId> 
    <version>8.17.0</version>
</dependency>
```

- 在服务消费端 Consumer 项目中修改 application.properties

```properties
#启用httpclient
feign.httpclient.enabled=true
```

注意：如果使用 HttpClient 作为 Feign 的客户端工具。那么在定义接口上的注解是需要注意，如果传递的是一个自定义的对象（对象会使用 json 格式来专递）。需要制定类型。

#### **在微服务的日志中记录每个接口 URL ，状态码和耗时信息** ####

- 在 Consumer 中添加 logback.xml 日志文件，并修改日志级别为 Debug

```xml
<?xml version="1.0" encoding="UTF-8" ?>
 <configuration>
<!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->  
    <property name="LOG_HOME" value="${catalina.base}/logs/" />  
    <!-- 控制台输出 -->   
    <appender name="Stdout" class="ch.qos.logback.core.ConsoleAppender">
       <!-- 日志输出编码 -->  
        <layout class="ch.qos.logback.classic.PatternLayout">   
             <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符--> 
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n   
            </pattern>   
        </layout>   
    </appender>   
    <!-- 按照每天生成日志文件 -->   
    <appender name="RollingFile"  class="ch.qos.logback.core.rolling.RollingFileAppender">   
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/server.%d{yyyy-MM-dd}.log</FileNamePattern>   
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>   
        <layout class="ch.qos.logback.classic.PatternLayout">  
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符--> 
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n   
            </pattern>   
       </layout> 
        <!--日志文件最大的大小-->
       <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
         <MaxFileSize>10MB</MaxFileSize>
       </triggeringPolicy>
    </appender>     

    <!-- 日志输出级别 -->
    <root level="dubug">   
        <appender-ref ref="Stdout" />   
        <appender-ref ref="RollingFile" />   
    </root> 



<!--日志异步到数据库 -->  
<!--     <appender name="DB" class="ch.qos.logback.classic.db.DBAppender">
        日志异步到数据库 
        <connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">
           连接池 
           <dataSource class="com.mchange.v2.c3p0.ComboPooledDataSource">
              <driverClass>com.mysql.jdbc.Driver</driverClass>
              <url>jdbc:mysql://127.0.0.1:3306/databaseName</url>
              <user>root</user>
              <password>root</password>
            </dataSource>
        </connectionSource>
  </appender> -->

</configuration>
```

- 在 Consumer 中修改 application.properties

```java
/**
 * 启用 Feign 需要添加 @EnableFeignClients 和 @EnableDiscoveryClient 注解
 *
 */
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
public class ConsumerHttpClientApp {

	// NONE:不记录任何信息，默认值
	// BASIC:记录请求方法、请求 URL、状态码和用时
	// HEADERS:在 BASIC基础上再记录一些常用信息
	// FULL:记录请求和相应的所有信息
	@Bean
	public Logger.Level getLog(){
		return Logger.Level.FULL; 
	}

	public static void main(String[] args) {
		SpringApplication.run(ConsumerHttpClientApp.class, args);
	}

}
```

#### 配置 Feign 负载均衡超时时间 ####

- 全局配置

```properties
#全局配置
# 请求连接的超时时间 默认的时间为 1 秒
ribbon.ConnectTimeout=5000
# 请求处理的超时时间
ribbon.ReadTimeout=5000

```

- 根据服务名称进行局部超时配置

```properties
#局部配置# 对所有操作请求都进行重试
ego-product-provider.ribbon.OkToRetryOnAllOperations=true
# 对当前实例的重试次数
ego-product-provider.ribbon.MaxAutoRetries=2
# 切换实例的重试次数
ego-product-provider.ribbon.MaxAutoRetriesNextServer=0
# 请求连接的超时时间
ego-product-provider.ribbon.ConnectTimeout=3000
# 请求处理的超时时间
ego-product-provider.ribbon.ReadTimeout=3000
```

## 服务灾难性雪崩效应 ##

>  在微服务环境中，因为**一个节点的故障而造成的其他节点的不可用**的情况是比较常见的，这也就是我们常说的灾难性雪崩现象，而` Hystrix `给我们提供了解决这种情况的方案。

### 什么灾难性雪崩效应 ###

![](https://ask.qcloudimg.com/http-save/4069933/lv3e8qmzdh.png?imageView2/2/w/1620)

正常情况下各个节点相互配置，完成用户请求的处理工作

![](https://ask.qcloudimg.com/http-save/4069933/xg7fd7ej99.png?imageView2/2/w/1620)

当某种请求增多，造成"服务T"故障的情况时，会延伸的造成"服务U"不可用，及继续扩展，如下

![](https://ask.qcloudimg.com/http-save/4069933/kcw4dips7i.png?imageView2/2/w/1620)

最终造成下面这种所有服务不可用的情况

![](https://ask.qcloudimg.com/http-save/4069933/wnpe9czd0y.png?imageView2/2/w/1620)



### 解决灾难性雪崩效应 ###

> 以下解决雪崩效应都是基于 Ribbon 的形式，用于 Provider 与 Consumer 之间通信。

#### 降级 ####

> 若调用的服务提供者出现异常，则返回托底数据

- 创建项目并添加 hystrix 启动器

```xml
<!-- Spring Boot hystrix 启动器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency> 
```

- 创建 SpringBoot 启动类：这里要开启熔断器（断路器）

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker //开启熔断器 断路器
public class ConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApplication.class, args);
	}
}
```

- UserServiceImpl

```java
@Service
public class UserServiceImpl implements UserService{

	// ribbon 负载均衡器
	@Autowired
	private  LoadBalancerClient loadBalancerClient;
	
	/**
	 * 降级处理
	 */
	@Override
	@HystrixCommand(fallbackMethod="fallback")
	public List<Product> getProducts() {
		// 选择调用服务的名称
		//  ServiceInstance 对象封装了服务的 IP、端口
		ServiceInstance si = 
				this.loadBalancerClient.choose("ego-product-provider");
		//拼接服务URL
		StringBuilder sb = new StringBuilder();
		sb.append("http://").append(si.getHost()).
					append(":").append(si.getPort()).append("/product/all");
		// 输出消息服务的 URL 地址
		System.out.println("URL:"+sb.toString());
		//创建 RestTempate 对象，发送 URL 请求，获取响应结果
		RestTemplate rt = new RestTemplate();
		//请求类型
		ParameterizedTypeReference<List<Product>> type = new  ParameterizedTypeReference<List<Product>>() {};
		//RequestEntity 封装了请求实体
		ResponseEntity<List<Product>> entity = rt.exchange(sb.toString(),HttpMethod.GET,null,type);
		List<Product> ProductList = entity.getBody();
		return ProductList;
	}

	/**
   	* 降级处理 方法
    */
	public List<Product> fallback(){
		List<Product> products = new ArrayList<>();
		products.add(new Product(-1, "我是托底数据哦！！！"));
		return products;
	}
}

```

**以下四种情况将触发getFallback调用**

(1) 方法抛出非 HystrixBadRequestException 异常。

(2) 方法调用超时

(3) 熔断器开启拦截调用

(4) 线程池/队列/信号量是否跑满

#### 请求缓存 ####

Hystrix 为了降低访问服务的频率，支持将一个请求与返回结果做缓存处理。

如果再次请求的 URL 没有变化，那么 Hystrix 不会请求服务，而是直接从缓存中将结果返回。

这样可以大大降低访问服务的压力。

Hystrix 自带缓存。有两个缺点：

1. 是一个本地缓存。在集群情况下缓存是不能同步的。
2. 不支持第三方缓存容器。Redis，memcache 不支持的。可以使用 spring 的 cache。

- 创建项目，并修改 pom 文件

```xml
<!-- Spring Boot hystrix 启动器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency> 
<!-- Spring Boot  redis 启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

- 修改 application.properties

```properties
# Redis
spring.redis.database=0
#Redis 服务器地址
spring.redis.host=192.168.170.128
#Redis 服务器连接端口
spring.redis.port=6379
#Redis 服务器连接密码（默认为空）
spring.redis.password= 
#连接池最大连接数（负值表示没有限制）
spring.redis.pool.max-active=100
#连接池最大阻塞等待时间（负值表示没有限制）
spring.redis.pool.max-wait=3000
#连接池最大空闭连接数
spring.redis.pool.max-idle=200
#连接汉最小空闲连接数
spring.redis.pool.min-idle=50
#连接超时时间（毫秒）
spring.redis.pool.timeout=600

```

- Spring Boot 启动类:添加 @EnableCaching 注解

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCaching
public class ConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApplication.class, args);
	}

}

```

- ProductServiceImpl:

```java


//将当前类中的 Reids 缓存 key 添加前缀
@CacheConfig(cacheNames="com.szxy.ego.product")
@Service
public class ProductServiceImpl implements ProductService{
	
	/**
	 *     将查询到的数据放入 Redis 缓存中   
	 *  #id ，获取方法的实参
	 * @param id
	 * @return
	 */
	@Cacheable(cacheNames="'product'+ #id")
	public Product findProductById(Integer id) {
		System.out.println("== find ===" +id);
		return new Product(id,"哇哈哈哈矿泉水");
	}
	
	/**
	 *   删除指定的商品
	 *  从 Redis 缓存中移除该商品  
	 * @param id
	 * @return
	 */
	@CacheEvict(cacheNames="'product'+ #id")
	public Product removeProduct(Integer id) {
		System.out.println("==== del ==="+id);
		return new Product(id,"删除成功");
	}
	
}

```



#### 请求合并 ####

​				**请求未合并**

![](http://img.zwer.xyz/blog/20190808211526.png)

**什么情况下使用请求合并**

在微服务架构中，我们将一个项目拆分成很多个独立的模块，这些独立的模块通过远程调用来互相配合工作，但是，在高并发情况下，通信次数的增加会导致总的通信时间增加，同时，线程池的资源也是有限的，高并发环境会导致有大量的线程处于等待状态，进而导致响应延迟，为了解决这些问题，我们需要来了解 Hystrix 的请求合并。

**请求合并的缺点**

设置请求合并之后，本来一个请求可能 5ms 就搞定了，但是现在必须再等 10ms 看看还有没有其他的请求一起的，这样一个请求的耗时就从 5ms 增加到 15ms 了，不过，如果我们要发起的命令本身就是一个**高延迟的命令**，那么这个时候就可以使用请求合并了，因为这个时候时间窗的时间消耗就显得微不足道了，另外高并发也是请求合并的一个非常重要的场景。



- ProductServiceImpl.java:

```java
//将当前类中的 Reids 缓存 key 添加前缀
@CacheConfig(cacheNames="com.szxy.ego.product")
@Service
public class ProductServiceImpl implements ProductService{

	//利用hystrix合并请求  
 @HystrixCollapser(batchMethod = "batchProduct", 
 scope = com.netflix.hystrix.HystrixCollapser.Scope.GLOBAL,  
    collapserProperties = {  
    //请求时间间隔在20ms之内的请求会被合并为一个请求,默认为10ms
    @HystrixProperty(name = "timerDelayInMilliseconds", value = "20"),
    //设置触发批处理执行之前，在批处理中允许的最大请求数。
    @HystrixProperty(name = "maxRequestsInBatch", value = "200"),  
    })  
	//consumer的controller调用的方法 该方法返回值必须要返回Future类型	
	@Override
	public Future<Product> findProductById(Integer id) {
		System.out.println("====="+id+"=====");
		return null;
	}

    @HystrixCommand
   	//调用Provider服务的方法
	public List<Product> batchProduct(List<Integer> ids) {
		for (Integer i : ids) {
			System.out.println(i);
		}
		// 假设向 Provider，获取响应结果
		List<Product> list = new ArrayList<>();
		list.add(new Product(1,"手机"));
		list.add(new Product(2,"平板电脑 ipad"));
		list.add(new Product(3,"笔记本电脑 laptop"));
		list.add(new Product(4,"台式电脑 "));
		System.out.println("=======dddd=========");
		return list;
	}
}

```

- ProductController.java:

```java
@RestController
public class ProductController {

	@Autowired
	private ProductService productService;
	
	// 获取商品列表
	@RequestMapping("/list")
	public void getProducts() throws Exception {
		
		Future<Product> p1 = this.productService.findProductById(1);
		Future<Product> p2 = this.productService.findProductById(2);
		Future<Product> p3 = this.productService.findProductById(3);
		Future<Product> p4 = this.productService.findProductById(4);
		System.out.println(p1.get().toString());
		System.out.println(p2.get().toString());
		System.out.println(p3.get().toString());
		System.out.println(p4.get().toString());
		
	}

}

```

##### 请求合并参数 #####

![](http://img.zwer.xyz/blog/20190808212305.png)



#### 服务熔断 ####

![](http://img.zwer.xyz/blog/20190809094956.png)



- 创建项目，并修改 pom.xml

```xml
<!-- Spring Boot hystrix 启动器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency> 
```

- SpringBoot 启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker //开启熔断器 断路器
public class ConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApplication.class, args);
	}

}
```

- ProductServiceImpl.java

```java
@Service
public class UserServiceImpl implements UserService {
	
	@Autowired
	private LoadBalancerClient loadBalancerClient;// ribbon 负载均衡器

	/**
	 *   服务熔断
	 */
	@Override
	@HystrixCommand(fallbackMethod = "fallback", commandProperties = {
			// 默认20个;10s内请求数大于20个时就启动熔断器，当请求符合熔断条件时将触发getFallback()。
			@HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_REQUEST_VOLUME_THRESHOLD,value = "10"),
			// 请求错误率大于50%时就熔断，然后for循环发起请求，当请求符合熔断条件时将触发getFallback()。
			@HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_ERROR_THRESHOLD_PERCENTAGE,value = "50"),
			// 默认5秒;熔断多少秒后去尝试请求
			@HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_SLEEP_WINDOW_IN_MILLISECONDS,value = "5000")
	})
	public List<Product> getProducts(int flag) {
		//假设接收请求后，发生异常
		if(flag == 1) {
			System.out.println("1");
			throw new RuntimeException();
		}
		// 选择调用服务的名称
		// ServiceInstance 对象封装了服务的 IP、端口
		ServiceInstance si = this.loadBalancerClient.choose("ego-product-provider");
		// 拼接服务URL
		StringBuilder sb = new StringBuilder();
		sb.append("http://").append(si.getHost()).append(":").append(si.getPort()).append("/product/all");
		// 输出消息服务的 URL 地址
		System.out.println("URL:" + sb.toString());
		// 创建 RestTempate 对象，发送 URL 请求，获取响应结果
		RestTemplate rt = new RestTemplate();
		// 请求类型
		ParameterizedTypeReference<List<Product>> type = new ParameterizedTypeReference<List<Product>>() {
		};
		// RequestEntity 封装了请求实体
		ResponseEntity<List<Product>> entity = rt.exchange(sb.toString(), HttpMethod.GET, null, type);
		List<Product> ProductList = entity.getBody();
		return ProductList;
	}

	/**
	 * 	服务熔断条件触发后，执行该 fallback 方法
	 */
	public List<Product> fallback(int flag) {
		List<Product> products = new ArrayList<>();
		products.add(new Product(-1, "我是托底数据哦！！！"));
		return products;
	}
}
```

##### **熔断参数** #####

![](http://img.zwer.xyz/blog/20190809095045.png)

#### 隔离 ####

##### 线程池隔离 #####

![](http://img.zwer.xyz/blog/20190809102842.png)

- 创建项目并修改 pom 文件

```xml
<!-- Spring Boot hystrix 启动器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency> 
```

- ProductServiceImpl.java

```java
@Service
public class ProductServiceImpl implements ProductService {

	@Autowired
	private LoadBalancerClient loadBalancerClient; // ribbon 负载均衡器

	/**
	 * 	线程池隔离
	 */
	@Override
	@HystrixCommand(groupKey = "ego-product-provider", commandKey = "getProducts", threadPoolKey = "ego-product-provider", threadPoolProperties = {
			@HystrixProperty(name = "coreSize", value = "30"), // 线程池大小
			@HystrixProperty(name = "maxQueueSize", value = "100"), // 最大队列长度
			@HystrixProperty(name = "keepAliveTimeMinutes", value = "2"), // 线程存活时间
			@HystrixProperty(name = "queueSizeRejectionThreshold", value = "15")// 拒绝请求
	}, fallbackMethod = "fallback")
	public List<Product> getProducts() {
		System.out.println("当前执行线程名: "+Thread.currentThread().getName());
		// 选择调用服务的名称
		// ServiceInstance 对象封装了服务的 IP、端口
		ServiceInstance si = this.loadBalancerClient.choose("ego-product-provider");
		// 拼接服务URL
		StringBuilder sb = new StringBuilder();
		sb.append("http://").append(si.getHost()).append(":").append(si.getPort()).append("/product/all");
		// 输出消息服务的 URL 地址
		System.out.println("URL:" + sb.toString());
		// 创建 RestTempate 对象，发送 URL 请求，获取响应结果
		RestTemplate rt = new RestTemplate();
		// 请求类型
		ParameterizedTypeReference<List<Product>> type = new ParameterizedTypeReference<List<Product>>() {
		};
		// RequestEntity 封装了请求实体
		ResponseEntity<List<Product>> entity = rt.exchange(sb.toString(), HttpMethod.GET, null, type);
		List<Product> ProductList = entity.getBody();
		return ProductList;
	}

	/**
	 * 降级处理 方法
	 */
	public List<Product> fallback() {
		List<Product> products = new ArrayList<>();
		products.add(new Product(-1, "我是托底数据哦！！！"));
		return products;
	}
	
	/**
	 *    显示当前线程名
	*/
	public void showThread() {
		System.out.println(Thread.currentThread().getName());
	}
}

```

**线程池隔离参数**

![](http://img.zwer.xyz/blog/20190809102935.png)

##### 信号量隔离 #####

![](http://img.zwer.xyz/blog/20190809104928.png)

- 创建 Maven 项目，并修改 pom 文件

```xml
<!-- Spring Boot hystrix 启动器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency> 
```

- ProductServiceImpl.java

```java
@Service
public class ProductServiceImpl implements ProductService {

	@Autowired
	private LoadBalancerClient loadBalancerClient; // ribbon 负载均衡器

	@HystrixCommand(fallbackMethod = "fallback", commandProperties = {
			@HystrixProperty(
                name = HystrixPropertiesManager.EXECUTION_ISOLATION_STRATEGY, 
					value = "SEMAPHORE"), // 信号量
			@HystrixProperty(name = HystrixPropertiesManager.EXECUTION_ISOLATION_SEMAPHORE_MAX_CONCURRENT_REQUESTS,
			value = "100")// 信号量最大并度
	})
	public List<Product> getProducts() {
		// 选择调用服务的名称
		// ServiceInstance 对象封装了服务的 IP、端口
		ServiceInstance si = this.loadBalancerClient.choose("ego-product-provider");
		// 拼接服务URL
		StringBuilder sb = new StringBuilder();
		sb.append("http://").append(si.getHost()).append(":").append(si.getPort()).append("/product/all");
		// 输出消息服务的 URL 地址
		System.out.println("URL:" + sb.toString());
		// 创建 RestTempate 对象，发送 URL 请求，获取响应结果
		RestTemplate rt = new RestTemplate();
		// 请求类型
		ParameterizedTypeReference<List<Product>> type = new ParameterizedTypeReference<List<Product>>() {
		};
		// RequestEntity 封装了请求实体
		ResponseEntity<List<Product>> entity = rt.exchange(sb.toString(), HttpMethod.GET, null, type);
		List<Product> ProductList = entity.getBody();
		return ProductList;
	}

	/**
	 * 降级处理 方法
	 */
	public List<Product> fallback() {
		List<Product> products = new ArrayList<>();
		products.add(new Product(-1, "我是托底数据哦！！！"));
		return products;
	}
}

```

![](http://img.zwer.xyz/blog/20190809102621.png)

**信号量参数**

![](http://img.zwer.xyz/blog/20190809104945.png)

##### 线程池隔离与信号量隔离的区别 #####

![](http://img.zwer.xyz/blog/20190809105722.png)

**线程池的应用场景**：请求并发量大，并且耗时长

**信号量的应用场景：**请求并发量大，并且耗时短

### Feign 雪崩效应 ###

> 以下解决雪崩效应都是基于 Feign 的形式，用于 Provider 与 Consumer 之间通信。

#### Fegin 降级处理 ####

- 创建 Maven 项目，并修改 pom 文件

```xml
<!-- Feign 启动器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

- SpringBoot 启动类

```java
/**
 *  启用 Feign 
 * 需要添加  @EnableFeignClients 和 @EnableDiscoveryClient 注解 
 *
 */
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
public class ConsumerApp {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApp.class, args);
	}
	
}
```

- ProductConsumerService.java 

```java
/**
 * @FeignClien 只作用于 Consumer 端
 * 属性 name 指定那个服务的接口实现类 
 */
@FeignClient(
    name="ego-product-provider",fallback=ProductConsumerServiceFallback.class)
public interface ProductConsumerService{

	/**
	 *   获取所有商品 
	 * @return
	 */
	@RequestMapping(value="/product/all",method=RequestMethod.GET)
	public List<Product> getAllProducts();
	
}

```

- ProductConsumerServiceFallback.java

```java
@Component
public class ProductConsumerServiceFallback implements ProductConslogumerService {
	
	/**
	 *  当 调用 Provider 中 getAllProducts 方法出现异常时，执行 fallback 方法
	 */
	public List<Product> getAllProducts() {
		List<Product> list = new ArrayList<>();
		list.add(new Product(-1,"我是托底数据哦 ！！！服务炸了"));
		return list;
	}

}

```

#### Feign 雪崩日志异常记录 ####

- ProductConsumerService.java 

```java
/**
 * @FeignClient 只作用于 Consumer 端
 *    属性 name 指定那个服务的接口实现类 
 *    
 *  @FeignClient 
 *  	fallback 只能返回托底数据
 *  	fallbackFactory 返回托底数据和异常日志记录
 *    
 */
@FeignClient(
    name="ego-productprovider",
    fallbackFactory=ProductConsumerServiceFallbackFactory.class
)
public interface ProductConsumerService{

	/**
	 *   获取所有商品 
	 * @return
	 */
	@RequestMapping(value="/product/all",method=RequestMethod.GET)
	public List<Product> getAllProducts();
	
}
```

- ProductConsumerServiceFallbackFactory.java 

```java
@Component
public class ProductConsumerServiceFallbackFactory 
	implements FallbackFactory<ProductConsumerService>{

	// 日志记录器
	Logger logger = 
		 LoggerFactory.getLogger(ProductConsumerServiceFallbackFactory.class);
	
	@Override
	public ProductConsumerService create(final Throwable t) {
		return new ProductConsumerService() {
			
			//  调用 Provider 中 getAllProducts 方法出现异常时，执行 fallback 方法
			@Override
			public List<Product> getAllProducts() {
				logger.warn("Fallback Exception :",t);
				List<Product> list = new ArrayList<>();
				list.add(new Product(-1,"我是托底数据哦 ！！！服务炸了"));
				return list;
			}
		};
	}
}
		
```

| @Feign 注解属性 | 作用                         |
| --------------- | ---------------------------- |
| name            | 指定 接口 属于哪个服务的名称 |
| fallback        | 返回托底数据                 |
| fallbackFactory | 返回托底数据和异常日志记录   |



### 可视化的数据监控 Hystrix-DashBoard  ###

> 只能监控单节点 Cosumer 

**创建监控项目**

- 创建 Maven 项目，并修改 pom 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath />
	</parent>
	<groupId>com.szxy</groupId>
	<artifactId>SpringCloud-service-hystrix-dashborad</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Spring Cloud config 启动器 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<!-- Spring Cloud eureka 启动器 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- Spring Boot hystrix 启动器 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix</artifactId>
		</dependency>
		<!-- Spring boot actuator 监控坐标 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- Spring Cloud hstrix-dashboard 坐标 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
		</dependency>
	</dependencies>
</project>
```

- application.properties

```java
spring.application.name=hystrix-dashborad
server.port=1001

eureka.client.serviceUrl.defaultZone=
    		http://user:123456@eureka1:8761/eureka,http://user:123456@euraka2/eureka


```

- SpringBoot 启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableHystrix
@EnableHystrixDashboard
public class DashboardApp {

	public static void main(String[] args) {
		SpringApplication.run(DashboardApp.class, args);
	}
	
}
```

![](http://img.zwer.xyz/blog/20190809152643.png)

![ ](http://img.zwer.xyz/blog/20190809152818.png)

**在被监控项目中**

- 修改 pom 文件

```xml
<!-- Spring Boot hystrix 启动器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
<!-- Spring boot actuator 监控坐标 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<!-- Spring Cloud hstrix-dashboard 坐标 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
</dependency>
```

- 修改 SpringBoot 启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker //开启熔断器 断路器
@EnableHystrix
@EnableHystrixDashboard 
public class ConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApplication.class, args);
	}

}
```



### **集群监控 Turbine** ###

![](http://img.zwer.xyz/blog/20190809155519.png)

Turbine 是聚合服务器发送事件流数据的一个工具，hystrix 的监控中，只能监控单个节点，实际生产中都为集群，因此可以通过 turbine 来监控集群服务。

**创建聚合监控项目 trubine**

- 创建 Maven 项目，并修改 pom 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath />
	</parent>
	<groupId>com.szxy</groupId>
	<artifactId>SpringCloud-service-hystrix-dashborad</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Spring Cloud config 启动器 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<!-- Spring Cloud eureka 启动器 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- Spring Boot hystrix 启动器 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix</artifactId>
		</dependency>
		<!-- Spring boot actuator 监控坐标 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- Spring Cloud hstrix-dashboard 坐标 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
		</dependency>
		<!-- 添加 turbine 坐标 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-turbine</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-netflix-turbine</artifactId>
		</dependency>
	</dependencies>
</project>
```

- Application.properties

```properties
spring.application.name=service-turbine
server.port=8765

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka,http://user:123456@euraka2/eureka

#---------------------------------------turbine--------------------------
#配置 Eureka 中的 serviceId 列表，表明监控哪些服务
turbine.appConfig=eureka-consumer-threadpool,ego-product-consumer-feign	
#指定聚合哪些集群，多个使用","分割，默认为 default。可使用http://.../turbine.stream?cluster={clusterConfig 之一}访问
turbine.aggregator.clusterConfig= default
# 1. clusterNameExpression 指定集群名称，默认表达式 appName；此时：turbine.aggregator.clusterConfig 需要配置想要监控的应用名称；
# 2. 当 clusterNameExpression: default 时，turbine.aggregator.clusterConfig 可以不写，因为默认就是 default；
# 3. 当 clusterNameExpression: metadata['cluster']时，假设想要监控的应用配置了 eureka.instance.metadata-map.cluster: ABC，
# 则需要配置，同时 turbine.aggregator.clusterConfig: ABC
turbine.clusterNameExpression="default"
```

- Spring Boot 启动类

```java
@SpringBootApplication
@EnableTurbine
public class DashboardApp {

	public static void main(String[] args) {
		SpringApplication.run(DashboardApp.class, args);
	}
	
}
```

**被监控项目**

- 修改 pom.xml 文件

```xml
<!-- Spring Boot hystrix 启动器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
<!-- Spring boot actuator 监控坐标 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<!-- Spring Cloud hstrix-dashboard 坐标 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
</dependency>
```

- Spring Boot 启动类

```java
/**
 *  启用 Feign 
 * 需要添加  @EnableFeignClients 和 @EnableDiscoveryClient 注解 
 *
 */
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
@EnableCircuitBreaker     //熔断器
@EnableHystrixDashboard 
public class ConsumerApp {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApp.class, args);
	}
	
}
```

![](http://img.zwer.xyz/blog/20190809162853.png)

##### 使用 Turbine 监控集群 #####

![](http://img.zwer.xyz/blog/20190809171306.png)



##### 使用 RabbitMq 结合 turbine  #####

> 没跑起来


## 设计微服务  ##

### 常见的微服务设计模式 ###

> 常见的微服务设计模式有六种

```http
https://www.infoq.cn/article/2015/04/micro-service-architecture
```

### 微服务设计模式实战 ###

> 一个服务既可以当做服务提供者 Provider ，也可以作为服务消费者 Consumer
>
> 举例，如下图所示，Consumer 代理与 Trade 服务之间，Consumer 代理是服务消费者 Consumer，Trade 服务是服务提供者 Provider ，然后在 Order 服务与 Trade 服务之间，Trade 服务是服务消费者 Consumer ，而 Order 服务是服务提供者。

![](http://img.zwer.xyz/blog/20190809201323.png)

注意：pojo 类需实现 Serializable 接口

## 服务网关 ##

> 1. 统一入口，为全部微服务提供唯一入口点，网关起到外部和内部**隔离**，保障了**后台服务**的安全性。
> 2. 鉴权校验：**识别**每个请求的**权限**，拒绝不符合要求的请求
> 3. 动态路由：动态的将请求**路由**到不同的后端集群中
> 4. 减少客户端与服务的**耦合**，服务可以独立发展，通过网关层来做映射

![](http://img.zwer.xyz/blog/20190810163249.png)

### 网关案例 ###

- 创建 Maven jar 项目，并修改 pom.xml 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<artifactId>zuul-gateway</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- zuul 网关 启动类 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-zuul</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

- 创建 application.properties

```properties
spring.application.name=zuul-gateway
server.port=9020

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/



```

- Spring Boot启动类

```java
@SpringBootApplication
@EnableZuulProxy
public class ConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApplication.class, args);
	}

}

```

- 通过 zuul 网关访问服务

```http
http://网关地址:网关端口号/服务名称/服务路径xxx
```

举例：

```http
http://localhost:9020/e-ebook-product-provider/product/findall
```

### 路由指定 ###

#### URL指定 ####

- application.properties

```properties
spring.application.name=zuul-gateway-route
server.port=9030

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/


# 1 ###################### 路由指定:URL 指定
############################## URL 匹配关键字，如果包含关键字就跳转到指定的 URL 中
zuul.routes.e-book-product-provider.path=/e-ebook-product-provider
zuul.routes.e-book-product-provider.url=http://127.0.0.1:9091
```

| 通配符                 |      |                                                              |
| ---------------------- | ---- | ------------------------------------------------------------ |
| 单个特定的路径         | ？   | http://localhost:9020/e-ebook-product-provider/a 或者 http://localhost:9020/e-ebook-product-provider/b |
| 单个任意的路径         | *    | http://localhost:9020/e-ebook-product-provider/product/a,http://localhost:9020/e-ebook-product-provider/product/ab,http://localhost:9020/e-ebook-product-provider/abc |
| 单个或者多个任意的路径 | **   | http://localhost:9020/e-ebook-product-provider/product/findall/xxx |

#### 采用服务名称指定路由方式 ####

```properties
## 2 ###################### 路由指定:服务指定 1 #############################
##将路径的/suibian/引到 eureka 的 e-book-product-provider 服务上
##规则：zuul.routes.路径名.path
##规则：zuul.routes.路径名.serviceId=eureka 的服务名
#zuul.routes.e-book-product-provider.path=/product2/**
#zuul.routes.e-book-product-provider.serviceId=e-ebook-product-provider
## 3 ###################### 路由指定:服务指定 2 ############################
##zuul.routes 后面跟着的是服务名，服务名后面跟着的是路径规则，这种配置方式更简单。
zuul.routes.e-ebook-product-provider.path=/product3/**
```

#### 采用路由排除 ####

```properties
## 4 ###################### 路由排除：排除某几个服务#############################
##排除后，这个地址将为空http://127.0.0.1:9030/e-book-product-provider/product/findAll
## 多个服务逗号隔开
#zuul.ignored-services=e-ebook-product-provider
## 5 ###################### 路由排除：排除所有服务############################
##由于服务太多，不可能手工一个个加，故路由排除所有服务，然后针对要路由的服务进行手工加
#zuul.ignored-services=*
#zuul.routes.e-book-order-provider.path=/e-book-order-provider/**
## 6 ###################### 路由排除：排除指定关键字的路径############################
## 排除所有包括/list/的路径
zuul.ignored-patterns=/**/findAll/**
zuul.routes.e-ebook-product-provider.path=/suibian/*
```

#### 添加前缀 ####

```properties
## 添加前缀
## http://localhost:9030/suibian/product-provider/product/findAll
zuul.prefix=/suibian
zuul.routes.e-ebook-product-provider.path=/product-provider/**
```

### 网关过滤器 ###

> zuul是可以认为是一种API-Gateway。zuul的核心是一系列的filters, 其作用可以类比Servlet框架的Filter，或者AOP。其原理就是在zuul把Request route到源web-service的时候，处理一些逻辑，比如Authentication，Load Shedding等

#### 网关日志输出案例 ####

- 创建项目并修改 pom 文件

```xml
<!-- zuul 网关 启动类 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
```

- application.properties

```properties
spring.application.name=zuul-gateway-filter
server.port=9030

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/
```

- LoggerFilter.java: 网关日志记录器

```java

@Component   //纳入 Spring IOC 容器中
public class LoggerFilter extends ZuulFilter{

	private static final Logger logger = 
			LoggerFactory.getLogger(LoggerFilter.class);
	/**
	 *  过滤内容：在 run 方法编写过滤逻辑
	 */
	@Override
	public Object run() {
		// 获取请求上下文
		RequestContext rc = RequestContext.getCurrentContext();
		HttpServletRequest req = rc.getRequest();
		logger.info("Logger ... :Method={},URL={}",req.getMethod(),req.getRequestURL().toString());
		return null;
	}

	/**
	 *  是否开启该Filter，true 表示开启，false 表示不开启 
	 */
	@Override
	public boolean shouldFilter() {
		return true;
	}
	
	/**
	 * 表示该 Filter 的优先级，通过返回的整数值
	 * 值越小，表示优先级越高
	 */
	@Override
	public int filterOrder() {
		return 0;
	}
	
	/**
	 * 过滤器类型：通过过滤器类型决定了过滤器执行的时间
	 */
	@Override
	public String filterType() {
		return "pre";
	}

}

```

- SpringBoot 启动类

```java
@SpringBootApplication
@EnableZuulProxy
public class ConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApplication.class, args);
	}

}
```

#### 过滤器类型 ####

|             | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| filterType  | pre：可以在请求被路由**之前**调用。一般用于身份权限验证、记录调用日志等等<br/>route:在路由执行**之后**被调用<br/>post：在 routing 和 error 过滤器之后被**调用**。可用于信息收集、统计信息如性能指标，对 response 的结构做些特殊处理<br/>error: 处理请求时发生错误时被调用。用于异常处理封装 |
| filterOrder | 用 int 来定义过滤器的执行顺序，数值越小优先级越高            |
| run         | 逻辑处理，一般2个用于：<br/>第一：请求前拦截，对请求进行验证判断。如果请求无效就直接断路；如果有效可再加工处理<br/>第二：请求结果后处理，即对结果做一些加工处理 |

#### zuul  请求生命周期 ####

![](http://img.zwer.xyz/blog/20190810201530.png)

注意： 过滤器的执行顺序按照同级别的设定的，不同级别的执行顺序按照上图所示。

#### 网关权限管理案例 ####

> 在上面的网关日志项目基础上，添加下面的网关过滤器类，用于用户登录权限校验

- AccessFilter.java

```java
/**
   *     网关
   *    用户登录权限验证
 *
 */
@Component   //纳入 Spring IOC 容器中
public class AccessFilter extends ZuulFilter{

	private static final Logger logger = 
			LoggerFactory.getLogger(AccessFilter.class);
	/**
	 *  过滤内容：在 run 方法编写过滤逻辑
	 */
	@Override
	public Object run() {
		logger.info("=============== pre1 ================");
		// 获取请求上下文
		RequestContext rc = RequestContext.getCurrentContext();
		HttpServletRequest req = rc.getRequest();
		//从表单中获取 token 信息
		String token = req.getParameter("token");
		if(token == null || token.equals("")) {
			logger.warn("token is null：cause by  might user is not null ...");
			rc.setSendZuulResponse(false);// 请求不继续往下传递
			rc.setResponseStatusCode(403); //表示用户未登录
			rc.setResponseBody("{\"result\":\"token is null\"}");//添加响应体内容
			rc.getResponse().setContentType("text/html;charset=utf-8");//添加响应类型
		}else {
			logger.info("=============== token existed ===============");
		}
		return null;
	}

	/**
	 *  是否开启该Filter，true 表示开启，false 表示不开启 
	 */
	@Override
	public boolean shouldFilter() {
		return true;
	}
	
	/**
	 * 表示该 Filter 的优先级，通过返回的整数值
	 * 值越小，表示优先级越高
	 */
	@Override
	public int filterOrder() {
		return 0;
	}
	
	/**
	 * 过滤器类型：通过过滤器类型决定了过滤器执行的时间
	 */
	@Override
	public String filterType() {
		return "pre";
	}

}
```

#### **网关过滤器对系统异常同一处理** ####

- ErrorFilter.java

```java
/**
 *    错误过滤器
 *    当其他过滤器出现异常时，调用该过滤器
 *
 */
@Component   //纳入 Spring IOC 容器中
public class ErrorFilter extends ZuulFilter{

	private static final Logger logger = 
			LoggerFactory.getLogger(ErrorFilter.class);
	/**
	 *  过滤内容：在 run 方法编写过滤逻辑
	 */
	@Override
	public Object run() {
		logger.info("=============== error ================");
		return null;
	}

	/**
	 *  是否开启该Filter，true 表示开启，false 表示不开启 
	 */
	@Override
	public boolean shouldFilter() {
		return true;
	}
	
	/**
	 * 表示该 Filter 的优先级，通过返回的整数值
	 * 值越小，表示优先级越高
	 */
	@Override
	public int filterOrder() {
		return 10;
	}
	
	/**
	 * 过滤器类型：通过过滤器类型决定了过滤器执行的时间
	 */
	@Override
	public String filterType() {
		return "error";
	}

}
```

- ExceptionHandler.java:处理异常信息并返回前台显示

```java
@RestController
public class ExceptionHandler implements ErrorController {

	@Override
	public String getErrorPath() {
		return "/error"; //若出现错误，交给下面的处理器处理
	}
	
	/**
	 *  处理 网关 error 类型过滤器的错误处理
	 * @return
	 */
	@RequestMapping(value="/error")
	public String error() {
		return "{\"result\":\"500 error!!\"}";
	}
	
}
```

### 网关容错 ###

#### zuul 与 hystrix 无缝结合 ####

>  zuul  的启动类包括 hystrix jar 包

![](D:\photo\TyporaPicture\20190811112324.png)

![](http://img.zwer.xyz/blog/20190811113007.png)

#### 网关对服务的降级 ####

> 降级是对服务的保护措施

- 创建 Maven 项目并修改 pom 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<artifactId>zuul-gateway</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- zuul 网关 启动类 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-zuul</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

- ProviderProductServiceFallback.java:实现 ZuulFallbackProvider 接口

```java
/**
 *   对 product-service 服务做降级处理
 *
 */
@Component
public class ProviderProductServiceFallback implements ZuulFallbackProvider{

	@Override
	public String getRoute() {
		return null;
	}

	/**
	 *   product service 发生异常，不可使用
	 *       对客户端做的响应
	 */
	@Override
	public ClientHttpResponse fallbackResponse() {
		
		return new ClientHttpResponse() {
			
			/**
			 * 设置响应头信息
			 */
			@Override
			public HttpHeaders getHeaders() {
				HttpHeaders header = new HttpHeaders();
				MediaType mediaType = new MediaType("application", "json",Charset.forName("utf-8") );
				header.setContentType(mediaType);
				return header;
			}
			
			/**
			 * 设置响应体
			 */
			@Override
			public InputStream getBody() throws IOException {
				String body = "商品服务不可用，请与管理员联系！";
				return new ByteArrayInputStream(body.getBytes());
			}
			
			@Override
			public String getStatusText() throws IOException {
				return this.getStatusCode().toString();
			}
			
			/**
			 * 设置状态码
			 */
			@Override
			public HttpStatus getStatusCode() throws IOException {
				return HttpStatus.OK;//200 
			}
			
			@Override
			public int getRawStatusCode() throws IOException {
				return this.getStatusCode().value();
			}
			
			@Override
			public void close() {
				
			}
		};
	}

}
```

注意：ClientHttpResponse 是一个接口，需要手动实现

### 网关限流 ###

> 在高并发的情况下，网关实现限流达到自我保护

- 创建 Maven 项目，并修改 pom 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<artifactId>zuul-gateway-ratelimit</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- zuul 网关 启动器 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-zuul</artifactId>
		</dependency>
		<!-- zuul 网关限流启动器 -->
		<dependency>
			<groupId>com.marcosbarbero.cloud</groupId>
			<artifactId>spring-cloud-zuul-ratelimit</artifactId>
			<version>1.3.4.RELEASE</version>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

- 修改 application.properties

```properties
spring.application.name=zuul-gateway-ratelimit
server.port=9020

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#根据服务名指定网关路由
zuul.routes.e-ebook-product-provider.path=/product/**
zuul.routes.e-ebook-product-provider.serviceId=e-ebook-product-provider
zuul.routes.e-ebook-order-provider.path=/order/**
zuul.routes.e-ebook-order-provider.serviceId=e-ebook-order-provider

#全局配置限流，默认为 false
#zuul.ratelimit.enabled=true
##60s 内请求超过 3 次，服务端就抛出异常，60s 后可以恢复正常请求
#zuul.ratelimit.default-policy.limit=3
#zuul.ratelimit.default-policy.refresh-interval=60
##针对 IP 进行限流，不影响其他 IP
#zuul.ratelimit.default-policy.type=origin


# 局部限流：针对某个服务进行限流
##开启限流
zuul.ratelimit.enabled=true
##60s 内请求超过 3 次，服务端就抛出异常，60s 后可以恢复正常请求
zuul.ratelimit.policies.e-ebook-product-provider.limit=3
zuul.ratelimit.policies.e-ebook-product-provider.refresh-interval=60
##针对某个 IP 进行限流，不影响其他 IP
zuul.ratelimit.policies.e-ebook-product-provider.type=origin
```

- 测试

![](D:\photo\TyporaPicture\20190811190942.png)

**网关限流参数**

![1565524495143](D:\photo\TyporaPicture\1565524495143.png)



### zuul 性能优化 网关的 2 层超时调优 ###

![](D:\photo\TyporaPicture\20190811200217.png)

- 修改 application.properties

```properties
spring.application.name=zuul-gateway-timeout
server.port=9020

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

zuul.routes.e-ebook-order-provider.path=/book/**

#第一层 hystrix 超时时间设置
#默认情况下是线程池隔离，超时时间 1000ms
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=8000
#第二层 ribbon 超时时间设置：设置比第一层小
# 请求连接的超时时间: 默认 5s
ribbon.ConnectTimeout=5000
# 请求处理的超时时间: 默认 5s
ribbon.ReadTimeout=5000
```

## Spring Cloud config 配置中心 ##

> Spring Cloud Conf 配置中心默认使用 Git ，作为配置中心的管理，可以进行版本控制

### 创建 Config -Server 项目 ###

- 创建 Maven 工程，并修改 pom 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- Spring Cloud Config 启动器 Server 服务器端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<artifactId>config-Server</artifactId>
</project>

```

- application.properties

```properties
spring.application.name=config-server
server.port=9050

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#Git 配置
spring.cloud.config.server.git.uri=https://gitee.com/zwer/config
spring.cloud.config.server.git.username=
spring.cloud.config.server.git.password=

```

 上传配置文件到 Gitee 码云上

- Spring Boot 启动类：

```java
@SpringBootApplication
@EnableEurekaClient
@EnableConfigServer
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}

}
```

- 测试

![](D:\photo\TyporaPicture\20190811211509.png)

**配置文件的前缀：config-client 表示服务的名称**

### 配置文件的命令规则与访问  ###

![](D:\photo\TyporaPicture\20190811211546.png)

### 创建 config-client 项目 ###

- 创建 Maven 项目，并修改 pom 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- Spring Cloud Config 配置中心 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<artifactId>config-Server</artifactId>
</project>
```

- **添加 boostrap.propreties**

```properties
spring.application.name=config-client
server.port=9051

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#默认 false,这里设置 true,表示开启读取配置中心的配置
spring.cloud.config.discovery.enabled=true
#对应 eureka 中的配置中心 serviceId，默认是 configserver
spring.cloud.config.discovery.serviceId=config-server
#指定环境
spring.cloud.config.profile=prod
#git 标签
spring.cloud.config.label=master
```

注意： boostrap.propreties  的名称不能修改，必须叫 boostrap，该配置文件读取优先于 application.properties 配置文件，通过 bootstrap.properties ，设置服务名和端口，以及连接 eureka 注册中心，和从配置中心获取配置信息

- Spring Boot 启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class ConfigClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigClientApplication.class, args);
	}

}

```

### 配置中心原理 ###

![](http://img.zwer.xyz/blog/20190811215333.png)



### 如何让 config-client 项目不重启，加载配置文件 ###

- 在 config-client 项目中修改 pom 配置文件，添加 Actuator 启动器

```xml
<!-- actuator -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

- 修改 bootstrap.properites

```properties
spring.application.name=config-client
server.port=9051

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#默认 false,这里设置 true,表示开启读取配置中心的配置
spring.cloud.config.discovery.enabled=true
#对应 eureka 中的配置中心 serviceId，默认是 configserver
spring.cloud.config.discovery.serviceId=config-server
#指定环境
spring.cloud.config.profile=prod
#git 标签
spring.cloud.config.label=master

#springboot 默认开启了权限拦截 会导致 /refresh 出现 401，拒绝访问
management.security.enabled=false
```

- 修改 ConfigClientController.java：添加注解 @RefreshScope

```java
@RestController
@RefreshScope  //刷新作用域
public class ConfigClientController {
	
	@Value("${e-book}")
	private String version;
	
	@RequestMapping("/show")
	public String show() {
		return version;
	}
	
}
```

注意： Spring Boot 启动时会以单例模式初始化 Controller 类的对象到Spring IOC 容器中，即 @Value 注解中的值注入到 Controller 类的单例对象中，即使配置文件发生变化，也不会重新加载，所以需要 @RefreshScope 刷新作用域，将 @Value 注解中的值重新注入到 Controller类的对象中。

- 刷新请求的 URL
>发送 Post 请求，重新加载 config-client 项目，刷新 application.properities

![](http://img.zwer.xyz/blog/20190812082010.png)

```java
public class HttpClientUtil {

	public static String doGet(String url, Map<String, String> param) {

		// 创建Httpclient对象
		CloseableHttpClient httpclient = HttpClients.createDefault();

		String resultString = "";
		CloseableHttpResponse response = null;
		try {
			// 创建uri
			URIBuilder builder = new URIBuilder(url);
			if (param != null) {
				for (String key : param.keySet()) {
					builder.addParameter(key, param.get(key));
				}
			}
			URI uri = builder.build();

			// 创建http GET请求
			HttpGet httpGet = new HttpGet(uri);

			// 执行请求
			response = httpclient.execute(httpGet);
			// 判断返回状态是否为200
			if (response.getStatusLine().getStatusCode() == 200) {
				resultString = EntityUtils.toString(response.getEntity(), "UTF-8");
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				if (response != null) {
					response.close();
				}
				httpclient.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return resultString;
	}

	public static String doGet(String url) {
		return doGet(url, null);
	}

	public static String doPost(String url, Map<String, String> param) {
		// 创建Httpclient对象
		CloseableHttpClient httpClient = HttpClients.createDefault();
		CloseableHttpResponse response = null;
		String resultString = "";
		try {
			// 创建Http Post请求
			HttpPost httpPost = new HttpPost(url);
			// 创建参数列表
			if (param != null) {
				List<NameValuePair> paramList = new ArrayList<>();
				for (String key : param.keySet()) {
					paramList.add(new BasicNameValuePair(key, param.get(key)));
				}
				// 模拟表单
				UrlEncodedFormEntity entity = new UrlEncodedFormEntity(paramList, "utf-8");
				httpPost.setEntity(entity);
			}
			// 执行http请求
			response = httpClient.execute(httpPost);
			resultString = EntityUtils.toString(response.getEntity(), "utf-8");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				response.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

		return resultString;
	}

	public static String doPost(String url) {
		return doPost(url, null);
	}

	public static String doPostJson(String url, String json) {
		// 创建Httpclient对象
		CloseableHttpClient httpClient = HttpClients.createDefault();
		CloseableHttpResponse response = null;
		String resultString = "";
		try {
			// 创建Http Post请求
			HttpPost httpPost = new HttpPost(url);
			// 创建请求内容
			StringEntity entity = new StringEntity(json, ContentType.APPLICATION_JSON);
			httpPost.setEntity(entity);
			// 执行http请求
			response = httpClient.execute(httpPost);
			resultString = EntityUtils.toString(response.getEntity(), "utf-8");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				response.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}

		return resultString;
	}

	public static void main(String[] args) {
		//刷新 config-client 项目，让配置文件重新加载
		String url = "http://localhost:9051/refresh";
		// 该url必须要使用dopost方式来发送
		HttpClientUtil.doPost(url);
		System.out.println("刷新成功 ....");
	}
}

```

**注意**

config-server 只会在下面两个时机同步 Git 端的远程配置文件到本地仓库中

1. 当 config-server 服务**启动**时，会同步 Git 端的配置文件信息
2. 当有其他客户端服务**请求 cofnig-server 服务**时，confi-server 服务会同步 Git 端的配置文件信息

### 安全加密与解密 ###

#### 对称加密 ####

- 检查加密环境

![](http://img.zwer.xyz/blog/20190812085712.png)

- 修改 application.properties:添加 encrypt.key 

```properties
encrypt.key=oldlu
```

- 添加 JCE  jar 包

下载解压后，把 jar 文件上传到需要安装 jce 机器上 JDK 或 JRE 的 security 目录下，覆盖源文件即可。

JDK：将两个 jar 文件放到%JDK_HOME%\jre\lib\security 下

JRE：将两个 jar 文件放到%JRE_HOME%\lib\security 下

```http
https://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html
```

![](http://img.zwer.xyz/blog/20190812085958.png)

- **切换 Spring Cloud 版本**

> Dalston.SR4、Dalston.SR3、Dalston.SR2 版本不能对配置文件加密，若需要调整到 Dalston.SR1
>
> https://github.com/spring-cloud/spring-cloud-config/issues/767 

```xml
<!-- Spring Cloud -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <!-- 修改 版本  -->
            <version>Dalston.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

- 测试

![](http://img.zwer.xyz/blog/20190812090620.png)

```http
http://localhost:9050/encrypt
```

```http
http://localhost:9050/decrypt
```



- 测试加密与解密

```java
public static void main(String[] args) {
    String url = "http://localhost:9050/encrypt";
    String info = HttpClientUtil.doPostJson(url, "root");
    //f22744e078c79e69bda8483c0657599aefb098af108f75f8a97fcc2abe516944
    System.out.println(info);
}
```

```java
public static void main(String[] args) {
    String url = "http://localhost:9050/decrypt";
    String info = HttpClientUtil.doPostJson(url, "f22744e078c79e69bda8483c0657599aefb098af108f75f8a97fcc2abe516944");
    System.out.println(info);//root
}
```

- 修改  config-e-ebook-product-provider项目的 pom 文件

```xml
<!-- Spring Cloud config 启动器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

- 添加 bootstrap.properties 文件

```properties
spring.application.name=config-e-ebook-product-provider
server.port=9091
 eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#默认 false,这里设置 true,表示开启读取配置中心的配置
spring.cloud.config.discovery.enabled=true
#对应 eureka 中的配置中心 serviceId，默认是 configserver
spring.cloud.config.discovery.serviceId=config-server-entryption-sys
#指定环境,若不指定环境采用 dafault 环境
#spring.cloud.config.profile=prod
#git 标签
spring.cloud.config.label=master
```
- 将 config-e-ebook-product-provider.properites文件上传到 Git 远程仓库中:在加密的密文前加上 `{cipher} `

```properties

#--------------db----------------
mybatis.type-aliases-package=com.book.product.pojo
mybatis.mapper-locations=classpath:com/book/product/mapper/*.xml
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/book-product?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
spring.datasource.username={cipher}01cce37ce9f3140debba61ca5db09788f530fb1f438e81ae5238571423465e91
spring.datasource.password={cipher}01cce37ce9f3140debba61ca5db09788f530fb1f438e81ae5238571423465e91
```

- 测试 

```http
http://127.0.0.1:9050/config-e-ebook-product-provider/default
```

![](http://img.zwer.xyz/blog/20190812093913.png)

#### 非对称加密 ####

> 加密使用公钥，解密使用私钥。即加密与解密的过程，使用不同的密钥处理。



![](https://upload.wikimedia.org/wikipedia/commons/thumb/0/03/Public_key_encryption_alice_to_bob.svg/langzh-1024px-Public_key_encryption_alice_to_bob.svg.png)

##### Java-Keytool 证书-使用说明 #####

> keytool 工具是 JDK 自带的工具,用于管理密钥和证书。

![](http://img.zwer.xyz/blog/20190812100410.png)

![](http://img.zwer.xyz/blog/20190812100631.png)



```
keytool -genkeypair -alias "test1" -keyalg "RSA" -keystore "test.keystore"
```

![](http://img.zwer.xyz/blog/20190812100746.png)

- 创建 config-server-rsa 项目，并修改 pom.xml  配置文件

```xml
<!-- Spring Cloud -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <!-- 修改 版本 ,仅针对 Spring cloud Config 服务端 Server-->
            <version>Dalston.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
</artifactId>
</project>
```

- 修改 application.properties 文件

```properties
spring.application.name=config-Server-rsa
server.port=9050

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#Git 配置
spring.cloud.config.server.git.uri=https://gitee.com/zwer/config
spring.cloud.config.server.git.username=
spring.cloud.config.server.git.password=

#keytool -genkeypair -alias "config-info" -keyalg "RSA" -keystore "encrypt-info.keystore" 
# keystore 文件的路径
encrypt.key-store.location=classpath:encrypt-info.keystore
# alias 指定密钥对的别名，该别名是公开的;
encrypt.key-store.alias=config-info
# storepass 密钥仓库
encrypt.key-store.password=zwz123
# keypass 用来保护所生成密钥对中的私钥
encrypt.key-store.secret=zwz456
```

- 测试

```http
http://127.0.0.1:9050/encrypt/status
```

![](http://img.zwer.xyz/blog/20190812105124.png)

- 创建 config-e-ebook-Product-Provider-rsa 项目，并修改 pom 文件 

```properties
<!-- Spring Cloud config 启动器 -->
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

- 上传 config-e-ebook-Product-Provider-rsa.properties 文件

```properties
#--------------db----------------
mybatis.type-aliases-package=com.book.product.pojo
mybatis.mapper-locations=classpath:com/book/product/mapper/*.xml
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/book-product?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
spring.datasource.username={cipher}AQALwv1qjVozd18qPKQFo9ATOmwyRKL1b82fBnM3vtv0fvZS1B8ZDdb3jY5ur7TDcQ4imMzAu/OM9kBv1BGoLJgxek/2p1bdOcfcdOjC5BhklBnQsNWHCYy0V8lfzJQr6U1tTZh63xkAySWR8XEuKIGOXIWuzCt7hOXl4oA2pbV1dxN+xMn4C6Rkb2kYSLGjN7z7fgA6+WUWEVnsey1a0FhMIP/ocqz9QAs2FVa0j3dZdtZiOh3bzIhnLeSGlGXpavNGGnL5u0EejhdzvFEC3A0+3s3OACD/nmb5Yz/IwimZ+cr9Ru+M8vFRNVOPnORzD+MM1vGH703s/3oZljTw3q8mDehBG/n9ekmJvIahsOZ+eGTU2Ecj0JipzV0J4pZPYvg=

spring.datasource.password={cipher}AQALwv1qjVozd18qPKQFo9ATOmwyRKL1b82fBnM3vtv0fvZS1B8ZDdb3jY5ur7TDcQ4imMzAu/OM9kBv1BGoLJgxek/2p1bdOcfcdOjC5BhklBnQsNWHCYy0V8lfzJQr6U1tTZh63xkAySWR8XEuKIGOXIWuzCt7hOXl4oA2pbV1dxN+xMn4C6Rkb2kYSLGjN7z7fgA6+WUWEVnsey1a0FhMIP/ocqz9QAs2FVa0j3dZdtZiOh3bzIhnLeSGlGXpavNGGnL5u0EejhdzvFEC3A0+3s3OACD/nmb5Yz/IwimZ+cr9Ru+M8vFRNVOPnORzD+MM1vGH703s/3oZljTw3q8mDehBG/n9ekmJvIahsOZ+eGTU2Ecj0JipzV0J4pZPYvg=
```

注意：这里加密的数据库的用户名和密码，根据 HttpClientUtils 工具发送 Post 请求到 `http://localhost:9050/encrypt`,并携带 json 参数 ，获取参数对应的的密文

![ ](http://img.zwer.xyz/blog/20190812105444.png)



- 测试

![](http://img.zwer.xyz/blog/20190812105044.png)

### 配置中心的用户安全认证 ###

- 创建 config-server-rsa-security 项目，并修改pom 文件

```xml
<!-- Spring Boot Security 启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- 修改 application.properties

```properties
# 安全认证#开启基于 http basic 的安全认证
security.basic.enabled=true 
security.user.name=user
security.user.password=123456
```

- 创建 项目 


- 修改 application.properties
```properties
#安全保护
spring.cloud.config.username=user
spring.cloud.config.password=123456
```


- 测试

![](http://img.zwer.xyz/blog/20190812110438.png)


## Spring Cloud Bus 消息总线
> Spring cloud Bus  集成了市面上常用的消息代理（RabbitMQ、Kafka 等2种），连接微服务系统中所有节点，当有数据变更时，可以通过消息代理广播通知微服务及时变更；
> 列如微服务的配置更新。
>bus 的应用场景：**微服务的数据变更，及时同步**

当需要数据变更，同步的时候，都可以使用  Spring cloud Bus .下面以同步客户端配置信息为例

![](http://img.zwer.xyz/blog/20190812140033.png)

#### 基于 Client 客户端刷新项目 ####

- 创建 config-client-bus-refresh 项目，并修改 pom 文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- Spring Cloud Config 配置中心 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<!-- actuator -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- Spring Cloud Bus 消息总线 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-bus-amqp</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<artifactId>config-Server</artifactId>
</project>

```

- 修改 bootstrap.properties
```properties
spring.application.name=config-client
server.port=9051

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#默认 false,这里设置 true,表示开启读取配置中心的配置
spring.cloud.config.discovery.enabled=true
#对应 eureka 中的配置中心 serviceId，默认是 configserver
spring.cloud.config.discovery.serviceId=config-server
#指定环境
spring.cloud.config.profile=dev
#git 标签
spring.cloud.config.label=master


#springboot 默认开启了权限拦截 会导致 /refresh 出现 401，拒绝访问
management.security.enabled=false

# RabbitMQ 配置
spring.rabbitmq.host: 192.168.170.134
spring.rabbitmq.port: 5672
spring.rabbitmq.username: zwz
spring.rabbitmq.password: 123456

```

- 刷新项目
![](http://img.zwer.xyz/blog/20190812114517.png)

```java
public static void main(String[] args) {
    String url = "http://localhost:9051/bus/refresh";
    HttpClientUtil.doPos    HttpClientUtil.doPost(url);
t(url);
}
```



#### 基于 Server 服务端刷新项目 ####

![](http://img.zwer.xyz/blog/20190812143209.png)



- 创建 config-server-bus-refresh  项目，并修改 pom 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- Spring Cloud Config 启动器 Server 服务器端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>
		<!-- Spring Cloud Bus 消息总线 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-bus-amqp</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<artifactId>config-Server</artifactId>
</project>
```

- 修改 application.properties
```properties
spring.application.name=config-server-bus-refresh
server.port=9050

eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#Git 配置
spring.cloud.config.server.git.uri=https://gitee.com/zwer/config
spring.cloud.config.server.git.username=
spring.cloud.config.server.git.password=

#springboot 默认开启了权限拦截 会导致 /refresh 出现 401，拒绝访问
management.security.enabled=false

#RabbitMQ 配置
spring.rabbitmq.host: 192.168.170.134
spring.rabbitmq.port: 5672
spring.rabbitmq.username: zwz
spring.rabbitmq.password: 123456

```


- 刷新 server 服务端项目

![](http://img.zwer.xyz/blog/20190812114517.png)

```java
public static void main(String[] args) {
String url = "http://localhost:9050/bus/refresh";
HttpClientUtil.doPost(url);
}
```

####  局部刷新服务 ####

> 基于 Server 服务端刷新的方式

1 刷新指定服务http://Config-Server/bus/refresh?destination=需要刷新的服务名称:端口

2 刷新指定集群http://Config-Server/bus/refresh?destination=需要刷新的服务名称:**

```java

public class HttpClientUtil {

	public static String doGet(String url, Map<String, String> param) {

		// 创建Httpclient对象
		CloseableHttpClient httpclient = HttpClients.createDefault();

		String resultString = "";
		CloseableHttpResponse response = null;
		try {
			// 创建uri
			URIBuilder builder = new URIBuilder(url);
			if (param != null) {
				for (String key : param.keySet()) {
					builder.addParameter(key, param.get(key));
				}
			}
			URI uri = builder.build();

			// 创建http GET请求
			HttpGet httpGet = new HttpGet(uri);

			// 执行请求
			response = httpclient.execute(httpGet);
			// 判断返回状态是否为200
			if (response.getStatusLine().getStatusCode() == 200) {
				resultString = EntityUtils.toString(response.getEntity(), "UTF-8");
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				if (response != null) {
					response.close();
				}
				httpclient.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return resultString;
	}

	public static String doGet(String url) {
		return doGet(url, null);
	}

	public static String doPost(String url, Map<String, String> param) {
		// 创建Httpclient对象
		CloseableHttpClient httpClient = HttpClients.createDefault();
		CloseableHttpResponse response = null;
		String resultString = "";
		try {
			// 创建Http Post请求
			HttpPost httpPost = new HttpPost(url);
			// 创建参数列表
			if (param != null) {
				List<NameValuePair> paramList = new ArrayList<>();
				for (String key : param.keySet()) {
					paramList.add(new BasicNameValuePair(key, param.get(key)));
				}
				// 模拟表单
				UrlEncodedFormEntity entity = new UrlEncodedFormEntity(paramList, "utf-8");
				httpPost.setEntity(entity);
			}
			// 执行http请求
			response = httpClient.execute(httpPost);
			resultString = EntityUtils.toString(response.getEntity(), "utf-8");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				response.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

		return resultString;
	}

	public static String doPost(String url) {
		return doPost(url, null);
	}

	public static String doPostJson(String url, String json) {
		// 创建Httpclient对象
		CloseableHttpClient httpClient = HttpClients.createDefault();
		CloseableHttpResponse response = null;
		String resultString = "";
		try {
			// 创建Http Post请求
			HttpPost httpPost = new HttpPost(url);
			// 创建请求内容
			StringEntity entity = new StringEntity(json, ContentType.APPLICATION_JSON);
			httpPost.setEntity(entity);
			// 执行http请求
			response = httpClient.execute(httpPost);
			resultString = EntityUtils.toString(response.getEntity(), "utf-8");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				response.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}

		return resultString;
	}

	public static void main(String[] args) {
		//String url = "http://localhost:9050/bus/refresh";
		//String url = "http://localhost:9050/bus/refresh?destination=config-client:9051";
		String url = "http://localhost:9050/bus/refresh?destination=config-client:**";
		HttpClientUtil.doPost(url);
		
		System.out.println("OK...");
	}
}

```

-----



## 消息驱动 Stream

> 解决程序与消息中间件的耦合度，动态地切换中间件

![](http://img.zwer.xyz/blog/20190812154241.png)


### 创建消息发送者
- 创建 stream-sender 项目，并修改 pom 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- test 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
		</dependency>
		<!-- Stream RabbitMQ 消息驱动 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<artifactId>config-Server</artifactId>
</project>
```
-  修改 application.properties 配置文件
```properties
spring.application.name=stream-sender
server.port=9050

#注册中心地址
eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#RabbitMQ 配置
spring.rabbitmq.host: 192.168.170.134
spring.rabbitmq.port: 5672
spring.rabbitmq.username: zwz
spring.rabbitmq.password: 123456


```

- 创建 ISenderService

```java
public interface ISenderService {
	
	@Output("zwz-exchange")
	SubscribableChannel send();
	
}

```
- Spring Boot 启动类
- 
```java
@SpringBootApplication
@EnableEurekaClient
@EnableBinding(value={ISenderService.class})
public class StreamSenderApplication {

	public static void main(String[] args) {
		SpringApplication.run(StreamSenderApplication.class, args);
	}

}

```

### 创建消息接收者 ###

- 创建 stream-receiver 项目，并修改 pom 文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy</groupId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka 启动器 服务提供者 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- test 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
		</dependency>
		<!-- Stream RabbitMQ 消息驱动 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<artifactId>config-Server</artifactId>
</project>

```

- 修改 application.properties 配置文件

```properites
	spring.application.name=stream-receiver
server.port=9051

#注册中心地址
eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#RabbitMQ 配置
spring.rabbitmq.host: 192.168.170.134
spring.rabbitmq.port: 5672
spring.rabbitmq.username: zwz
spring.rabbitmq.password: 123456
	
```

- Spring Boot 启动类
```java
@SpringBootApplication
@EnableEurekaClient
@EnableBinding(value={IReceiverService.class})
public class StreamReceiver {

	public static void main(String[] args) {
		SpringApplication.run(StreamReceiver.class, args);
	}

}

```

- 测试（Stream-sender 端）

```java

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes={StreamSender.class})
public class SenderServiceTest {
	
	@Autowired
	private ISenderService senderService;
	
	@Test
	public void send() {
		String msg = "Hello World  moximoxi";
		Message<byte[]> message = MessageBuilder.withPayload(msg.getBytes()).build();
		this.senderService.send().send(message);
	}
	
}

```

###  消息分组

> 将消息分散发送给不同的服务，并且**一个消息不会发送给不同的服务中，只会发送给一个服务中**，即一个消息对应一个服务，并且消息只发送一次。

- 创建分组中的**消息发送者项目**，并修改 pom 文件
```xml
<!-- Stream RabbitMQ 消息驱动 -->
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

- 修改 application.properites
```properties
spring.application.name=stream-group-receiver2
server.port=9053

#注册中心地址
eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#RabbitMQ 配置
spring.rabbitmq.host: 192.168.170.134
spring.rabbitmq.port: 5672
spring.rabbitmq.username: zwz
spring.rabbitmq.password: 123456

# 对应 MQ 是 exchange
spring.cloud.stream.bindings.outProduct.destination=exchangeProduct

```

- ISenderService.java 接口

```java
package com.szxy.service;

import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.SubscribableChannel;

public interface ISenderService {
	
	String OUTPUT = "outProduct";
	
	@Output(OUTPUT)
	SubscribableChannel send();
	
}

```

- Spring Boot 启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableBinding(value={ISenderService.class})
public class StreamGroupSender {

	public static void main(String[] args) {
		SpringApplication.run(StreamGroupSender.class, args);
	}

}

```

- 创建分组中的**消息接收者项目**，并修改 pom 文件
```xml
<!-- Stream RabbitMQ 消息驱动 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

- 修改 application.properties

```properties
spring.application.name=stream-group-receiver
server.port=9051

#注册中心地址
eureka.client.serviceUrl.defaultZone=http://user:123456@eureka1:8761/eureka/,http://user:123456@eureka2:8761/eureka/

#RabbitMQ 配置
spring.rabbitmq.host: 192.168.170.134
spring.rabbitmq.port: 5672
spring.rabbitmq.username: zwz
spring.rabbitmq.password: 123456


# 对应 MQ 是 exchange
spring.cloud.stream.bindings.inputProduct.destination=exchangeProduct
# 具体分组 对应 MQ 是 队列名称 并且持久化队列
spring.cloud.stream.bindings.inputProduct.group=groupProduct
```

- IReceiverService.java

```java
public interface IReceiverService {
	
	String INPUT =  "inputProduct";
	
	@Input(INPUT)
	SubscribableChannel receive();
	
}

```

- ReceiverService.java

```java
@Service
@EnableBinding(value={IReceiverService.class})
public class ReceiverService {

	@StreamListener(IReceiverService.INPUT)
	public void onReceive(Product product) {
		System.out.println("Receiver :"+product);
	}
	
}

```

- Spring Boot 启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableBinding(value={IReceiverService.class})
public class StreamGroupReceiver {

	public static void main(String[] args) {
		SpringApplication.run(StreamGroupReceiver.class, args);
	}

}
```

- 测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes={StreamGroupSender.class})
public class SenderServiceTest {
	
	@Autowired
	private ISenderService senderService;
	
	@Test
	public void send() {
		Product product = new Product();
		product.setId(1);
		product.setName("娃哈哈");
		for(int i=0; i<10;i++) {
			Message<Product> message = MessageBuilder.withPayload(product).build();
			this.senderService.send().send(message);
		}
	}
	
}
```

### 消息分区 ###

> 相同的消息发送多次，只会发送给相同的服务

- 消息发送端的 application.properties

```properties

```

- 消息接收端的 application.properties

```properties

```

## Spring Cloud Sleuth
> 微服务跟踪工具
![](http://img.zwer.xyz/blog/20190812173946.png)


### sleuth 日志分析
> 加入 logback.xml 日志配置文件,并且日志级别为 DEBUG
![](http://img.zwer.xyz/blog/20190812180716.png)

### ELK（Elasticsearch，Logstash 和 Kibana ）

![](http://img.zwer.xyz/blog/20190813100110.png)

- **安装  ElasticSearch**

```shell
# 查看当前 Linux 的内核
uname -a 
# 更新 内核
rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7
rpm -Uvh http://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-lt -y

vim /etc/grub.conf
#修改文件中内容，保证使用新内核启动。
default=0
#重启系统


-----------------------------安装 elasticsearch--------------------------------------安装 elasticsearch6.2.31.下载压缩
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.3.tar.gztar -zxvf elasticsearch-6.2.3.tar.gz

2.修改 elasticsearch 需要的系统配置。
vi /etc/security/limits.conf
#增加下述内容。
* soft nofile 65536
* hard nofile 65536

3.vi /etc/security/limits.d/90-nproc.conf es 启动时的线程池最低容量
#修改下述内容
* soft nproc 4096
root soft nproc unlimited

4.vi /etc/sysctl.conf 
#新增下述内容
vm.max_map_count=655360

#使用命令，让 sysctl 配置生效
sysctl -p

#修改 elasticsearch 的配置文件，设置可访问的客户端。0.0.0.0 代表任意客户端访问
vi config/elasticsearch.yml
#修改下述内容
network.host: 0.0.0.0
http.port: 9200

---------------------------------创建用户-------------------------------------
从 5.0 开始，ElasticSearch 安全级别提高了，不允许采用 root 帐号启动，所以我们要添加一个用户。
#1.创建 elk 用户组
groupadd elk
#2.创建用户 oldlu
useradd oldlu
passwd oldlu
#3.将 oldlu 用户添加到 elk 组
usermod -G elk oldlu
#4.设置 sudo 权限
visudo
#找到 root ALL=(ALL) ALL 一行，添加 oldlu 用户，如下。
#Allow root to run any commands anywhereroot ALL=(ALL) ALL
#找到后添加如下内容
oldlu ALL=(ALL) ALL
#5.为用户分配权限
chown -R oldlu:elk /usr/local/elasticsearch
------------------------------切换用户 oldlu--------------------------------------
#1.ElasticSearch 启动与停止
#1.启动
./elasticsearch -d
#2.验证
curl http://192.168.70.140:9200
#浏览器直接访问 
http://192.168.70.140:9200/
```

![](http://img.zwer.xyz/blog/20190812193354.png)

![](http://img.zwer.xyz/blog/20190812183624.png)

- **安装  Head 插件**

[npminstall报错：Connecttimeoutfor5000ms踩坑](http://blog.sina.com.cn/s/blog_17689050c0102yhq1.html)

```shell
---------------------------------切换用户 root---------------------------
#2.安装 NodeJS
#要求在 root 下执行
curl -sL https://rpm.nodesource.com/setup_8.x | bash -
yum install -y nodejs
#3.安装 npmnpm install -g cnpm --registry=https://registry.npm.taobao.org
#4.使用 npm 安装 grunt
npm install -g grunt
npm install -g grunt-cli --registry=https://registry.npm.taobao.org --no-proxy
----------------------切换用户 oldlu------------------------------------
#5.查看以上版本node -vnpm -vgrunt -version6.
#下载 head 插件源码
cd /home/oldlumkdir es
wget https://github.com/mobz/elasticsearch-head/archive/master.zip
unzip master.zip 
#7.国内镜像安装
cd elasticsearch-head-master
sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
sudo cnpm install
--------------------------------配置---------------------------------
#1.配置 ElasticSearch，使得 HTTP 对外提供服务
vi config/elasticsearch.yml
#添加如下内容
# 增加新的参数，这样 head 插件可以访问 es。
#设置参数的时候:后面要有空
    http.cors.enabled: true
    http.cors.allow-origin: "*"
#2.修改 Head 插件配置文件
vi Gruntfile.js
找到 connect：server，添加 hostname 一项，如下
connect: {
	server: {
		options: {
				hostname: '0.0.0.0',
				port: 9100,
				base: '.',
				keepalive: true} } 
		}
#六、启动
#1.重启 elasticsearch ，注意以 oldlu 用户启动，不能以 root 用户启动
./elasticsearch -d
#2.启动 head
#elasticsearch-head-master 目录下：
grunt server或 npm run start
3.访问 9100 端口
七、简单应用
1.创建索引
curl -XPUT http://192.168.70.140:9200/applog
2.查看 head 变化http://192.168.70.140:9100/
```

![1565615415446](D:\photo\TyporaPicture\1565615415446.png)

- **安装 Logstach** 

```shell
#1.下载压缩
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.2.3.tar.gz
tar zxvf logstash-6.2.3.tar.gz
mv logstash-6.2.3/ logstash
#2.测试
./bin/logstash -e 'input { stdin { } } output { stdout {} }'
#3.修改配置在 logstash 的主目录下
vim config/log_to_es.conf
内容如下：
# For detail structure of this file 
# Set: https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html 
input { 
    # For detail config for log4j as input, 
    # See: https://www.elastic.co/guide/en/logstash/current/plugins-inputs-log4j.html 
    tcp { 
        mode => "server" 
        host => "192.168.70.140" 
        port => 9250 
} 
}filter { 
	#Only matched data are send to output. 
} 
output { 
 	# For detail config for elasticsearch as output, See:   https://www.elastic.co/guide/en/logstash/current/plugins-outputs-		elasticsearch.html 
elasticsearch { 
        action => "index" #The operation on ES 
        hosts => "192.168.70.140:9200" #ElasticSearch host, can be array. 
        index => "applog" #The index to write data to. 
	} 
} 

#2.启动
./bin/logstash -f config/log_to_es.conf 
或 后台运行守护进程
./bin/logstash -f config/log_to_es.conf &

#3.测试curl 'http://192.168.70.140:9200/_search?pretty'
```

- **安装 Kibana** 

```shell
#1.下载压缩包
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.2.3-linux-x86_64.tar.gz
tar zxvf kibana-6.2.3-linux-x86_64.tar.gzmv kibana-6.2.3-linux-x86_64.tar.gz/ kibana

#2.修改配置vim config/kibana.yml 
#把以下注释放开，使配置起作用。
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.url: http://192.168.70.140:9200
kibana.index: ".kibana"

#3.启动./bin/kibana 
#4.测试http://192.168.70.140:5601/app/kibana
```

![1565620322294](D:\photo\TyporaPicture\1565620322294.png)


- 测试

kibana 界面上不能显示数据，但是 es 可以获取日志数据


### 分布式服务跟踪：Zipkin
> 实时服务链路跟踪
> zipkin 是一款开源的分布式实时数据追踪系统（Distributed Tracking System）
> 器主要功能是聚集来自各个异构系统的实时监控数据

zipkin 原理中文版论文地址
```http
http://bigbully.github.io/Dapper-translation/
```

**搭建 sleuth-zipkin-server 端**

- 创建 maven 项目，并修改 pom 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.szxy</groupId>
	<artifactId>sleuth-Consumer</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Camden.SR4</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>io.zipkin.java</groupId>
			<artifactId>zipkin-autoconfigure-ui</artifactId>
		</dependency>
		<!-- zipkin-server 服务端 -->
		<dependency>
			<groupId>io.zipkin.java</groupId>
			<artifactId>zipkin-server</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

- 修改 application.properties: 服务的端口是 zikpin 官方推荐的

```properties
spring.application.name=sleuth-zipkin-server
server.port=9411


```

- Spring boot 启动类

```java
@SpringBootApplication
@EnableZipkinServer
public class ZipkinServer {

	public static void main(String[] args) {
		SpringApplication.run(ZipkinServer.class,args);
	}
	
}

```

#### Spring Cloud 集成 Zipkin

```xml
 <!-- Sleuth 启动器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<!-- zipkin 坐标 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

- 修改 application.properties

```properties
spring.zipkin.base-url=http://127.0.0.1:9411/
```

- Spring boot 启动类无需添加新的注解


- 测试

![](http://img.zwer.xyz/blog/20190813142242.png)

#### Zikpin 原理

- json 格式数据分析

![](http://img.zwer.xyz/blog/20190813142332.png)

- 事件类型 

|  类型    |  作用    |
| ---- | ---- |
|  cs    |  客户端/消费者发起请求    |
|  cr    |  客户端/消费者接收到应答    |
|  sr    |  服务端/生产者接收到请求    |
|  ss    |  服务端/生产者发送应答    |

![](http://img.zwer.xyz/blog/20190813142644.png)

**ZipKin 执行原理图**

![](http://img.zwer.xyz/blog/20190813142901.png)



### 采用  RabbitMQ  收集 ZipKin 的跟踪数据-创建服务端 ###

- 创建 maven 项目，并修改 pom 文件
更换 Spring Cloud 的版本，换成 Dalston.SR5 版本
```xml
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin-autoconfigure-ui</artifactId>
</dependency>
<!-- zipkip rabbit -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin-stream</artifactId>
</dependency>
```
-  修改 application.properites 

```properties
spring.application.name=sleuth-zipkin-server-mq
server.port=9411

#RabbitMQ 配置
spring.rabbitmq.host: 192.168.170.134
spring.rabbitmq.port: 5672
spring.rabbitmq.username: zwz
spring.rabbitmq.password: 123456

```

-  Spring Boot 启动类

```java

@SpringBootApplication
@EnableZipkinStreamServer
public class ZipkinServer {

	public static void main(String[] args) {
		SpringApplication.run(ZipkinServer.class,args);
	}
	
}

```
-  启动 
![](http://img.zwer.xyz/blog/20190813145030.png)

### 采用  RabbitMQ  收集 ZipKin 的跟踪数据-创建客户端 ###

- 创建 Maven 项目，并修改 pom 文件

```xml
<!-- Sleuth 启动器 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<!-- RabbitMQ Zipkin -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

- 修改 application.properties

```properties
#用于服务与 Zipkin 直接连接
#spring.zipkin.base-url=http://127.0.0.1:9411/

#RabbitMQ 配置
spring.rabbitmq.host: 192.168.170.134
spring.rabbitmq.port: 5672
spring.rabbitmq.username: zwz
spring.rabbitmq.password: 123456
```


### 跟踪数据持久化到 MySQL 中  ###
> Zipkin 的内存是有限，但是服务产生的数据却巨大的，所以需要将 Zipkin 跟踪数据放入数据库中。


- 创建 sleuth-zipkin-server-mq-mysql 项目，并修改 pom 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.szxy</groupId>
	<artifactId>sleuth-zipkin-server-mq</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<!-- Spring Cloud -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- Spring Boot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>io.zipkin.java</groupId>
			<artifactId>zipkin-autoconfigure-ui</artifactId>
		</dependency>
		<!-- zipkip rabbit -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-stream-binder-rabbit</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-sleuth-zipkin-stream</artifactId>
		</dependency>
		<!--持久化服务跟踪数据到 MySQL 中 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>

```
- 修改 application.properties

```properties
spring.application.name=sleuth-zipkin-server-mq
server.port=9411

#RabbitMQ 配置
spring.rabbitmq.host: 192.168.170.134
spring.rabbitmq.port: 5672
spring.rabbitmq.username: zwz
spring.rabbitmq.password: 123456

#zipkin数据保存到数据库中需要进行如下配置
#表示当前程序不使用sleuth
spring.sleuth.enabled=false

#表示zipkin数据存储方式是mysql
zipkin.storage.type=mysql

#数据库脚本创建地址
spring.datasource.schema=classpath:/mysql.sql

#spring boot数据源配置
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/zipkin?autoReconnect=true&useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&useSSL=false
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.initialize=true
spring.datasrouce.continueOnError=true


```











---


Spring Boot 启动类大全

```http
https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters
```

```
jps -- Java Virtual Machine Process Status Tool 
# 可以列出本机所有java进程的pid 
```



