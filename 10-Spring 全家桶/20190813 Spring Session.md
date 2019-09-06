# Spring Session #

> Spring Session provides an API and implementations for managing a user’s session information.
>
> 解决 Session  共享问题。

准备工作：安装 redis 服务器

## Spring Session ##

#### 创建  Spring-session 项目 ####

- pom 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.szxy</groupId>
    <artifactId>Spring-session</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>session_service1</module>
        <module>session_service2</module>
        <module>commons</module>
    </modules>

    <name>Spring-session</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
    </properties>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
    </parent>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.session</groupId>
                <artifactId>spring-session-bom</artifactId>
                <version>Bean-SR3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- web starter  -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--Spring session data redis-->
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
        </dependency>
        <!--Lettuce 是一个基于Netty的NIO方式处理Redis的技术-->
        <dependency>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### session-service1 子模块 ####

- WebController.java:用于设置 session 数据

```java
@RestController
@RequestMapping("service1")
public class WebController {

    @RequestMapping(value="/set",method= RequestMethod.GET)
    public String addSession(String msg, HttpSession session){
        session.setAttribute("msg",msg);
        return "ok";
    }

    @RequestMapping(value="/setUser",method = RequestMethod.POST)
    public String addUserSession(Users user,HttpSession session){
        session.setAttribute("user",user);
        return "user ok";
    }

}

```

- Spring Boot 启动类

```java
@SpringBootApplication
@EnableRedisHttpSession
public class SessionService1 {
    public static void main(String[] args) {
        SpringApplication.run(SessionService1.class,args);
    }
}

```

#### session-service2 子模块 ####

- WebController.java:用于获取 session 数据

```java
@RestController
@RequestMapping("service2")
public class WebController {

    @RequestMapping("/get")
    public String getSession(HttpSession session){
        String msg = (String) session.getAttribute("msg");
        return msg;
    }

    @RequestMapping("/getUser")
    public Users getUserSession(HttpSession session){
        Users user= (Users) session.getAttribute("user");
        System.out.println(user);
        return user;
    }
}

```

- Spring boot 启动类

```java
@SpringBootApplication
@EnableRedisHttpSession
public class SessionService2 {
    public static void main(String[] args) {
        SpringApplication.run(SessionService2.class,args);
    }
}

```



### Spring-Session 中 Redis 存储结构 ###

| Key                                           | 作用                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| spring:session:expirations:(Set 结构)         | 用户 ttl 过期时间记录 这个 k 中的值是一个时间戳，根据这个 Session 过期时刻滚动至下一分钟而计算得出。 <br/>这个 k 的过期时间为 Session 的最大过期时间 + 5 分钟。 |
| spring:session:sessions:(Hash 结构)           | maxInactiveInterval：过期时间间隔 <br/>creationTime：创建时间<br/>lastAccessedTime：最后访问时间 <br/>sessionAttr：Attributes 中的数据 存储 Session 的详细信息，包括 Session 的过期时间间隔、最后的访问时间、attributes 的值。<br/>这个 k 的过期时间为 Session 的最大过期时间 + 5 分钟。 |
| spring:session:sessions:expires:(String 结构) | 过期时间记录 这个 k-v 不存储任何有用数据，只是表示 Session 过期而设置。 这个 k 在 Redis 中的过期时间即为 Session 的过期时间间隔。 |



### 设置 Session 失效时间 ###

> 注意：若要设置 session 的失效时间，必须把该 session 相关所有服务的 session 失效时间都设置为一致的

```java
SpringBootApplication
/**
 *   maxInactiveIntervalInSeconds 设置 session 失效时间，默认以秒为单位
 */
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 20)
public class SessionService1 {
    public static void main(String[] args) {
        SpringApplication.run(SessionService1.class,args);
    }
} 
```

### **@EnableRedisHttpSession** **注解** ###

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(RedisHttpSessionConfiguration.class)
@Configuration
public @interface EnableRedisHttpSession {

	int maxInactiveIntervalInSeconds()
        default MapSession.DEFAULT_MAX_INACTIVE_INTERVAL_SECONDS;//1800

	String redisNamespace() 
        default RedisOperationsSessionRepository.DEFAULT_NAMESPACE;

	RedisFlushMode redisFlushMode() 
        default RedisFlushMode.ON_SAVE;

	String cleanupCron() 
        default RedisHttpSessionConfiguration.DEFAULT_CLEANUP_CRON;

}

```

|                              |                                                              |
| ---------------------------- | ------------------------------------------------------------ |
| maxInactiveIntervalInSeconds | 设置 session 的失效时间，单位为秒。默认 30分钟（1800s）      |
| redisNamespace               | 为键定义唯一的命名空间。该值用于通过更改前缀与默认的 spring：session 隔离 |
| redisFlushMode               | Redis 会话的刷新模式，默认值为 “保存”                        |
| cleanupCron                  | 过期会话清理作业的 cron 表达式。默认值("0 * * * * *")每分钟运行一次。 |

### 更换 Spring Session 序列化器 ###

> Spring  Session 默认采用 JDK 序列器，其特点是占用内存较大，效率低下。这里更换为其他序列器

```java
@Configuration
public class SpringSessionConfig  {


    /**
     * 设置默认的序列化器
     * @return
     */
    @Bean("springSessionDefaultRedisSerializer")
    public RedisSerializer defaultRedisSerializer(){
        return getRedisSerializer();
    }

    /**
     * 获取  GenericJackson2JsonRedisSerializer 序列化器
     * @return
     */
    public RedisSerializer getRedisSerializer(){
        return new GenericJackson2JsonRedisSerializer();
    }

}
```

## Spring Session MongoDB ##

> Spring Session MongoDB 是 Spring Session 的二级项目。其功能与 Spring Session 是相同 的。Spring Session MongoDB 提供了一个 API 和实现，用于通过利用 Spring Data MongoDB 来管理存储在 MongoDB 中的用户会话信息。 

### 安装 MongoDB ###

上传 MongoDB 压缩包到 Linux 中

解压 tar 压缩包

```shell
#解压
tar -zxf  tar -zxvf mongodb-linux-x86_64-4.0.9.tgz 

#解压后的文件夹放到指定位置
 cp -rf mongodb-linux-x86_64-4.0.9 /usr/local/mongodb/
#进入 mongodb 目录下
cd /usr/local/mongodb
#创建目录
mkdir -p etc/data/db
#进入 etc 目录中
cd etc
#创建 mongodb 日志文件
touch mongodb.log
# vim 编辑 mongodb.conf,加入以下内容
dbpath=/usr/local/mongodb/etc/data/db
logpath=/usr/local/mongodb/etc/mongodb.log
prot=27017
fork=true
bind_ip=0.0.0.0  #表示任何 IP 都可以访问	
#进入 bin 目录中，执行下面的命名，后置启动 mongodb 
./mongod --config /usr/local/mongodb/etc/mongodb.conf

#关闭 mongodb 命令
./mongod --shutdown --config /usr/local/mongodb/etc/mongodb.conf

#创建 mongodb 的库
> use szxy;
switched to db szxy

```

### MongoDB 的启动与停止 ###

```shell
#进入 bin 目录中，执行下面的命名，后置启动 mongodb 
./mongod --config /usr/local/mongodb/etc/mongodb.conf

#关闭 mongodb 命令
./mongod --shutdown --config /usr/local/mongodb/etc/mongodb.conf
```

### MongoDB 操作 ###

> 使用  `./mongo` 连接到 MongoDB 数据库的客户端

| 命令           | 作用                 |
| -------------- | -------------------- |
| use 库名       | 创建或者切换到该库下 |
| show tables    | 显示库下的表         |
| db.表名.find() | 查看表中的内容       |
| db.表名.drop() | 删除表中的内容       |



### 创建 Spring-session-mongodB 项目 ###

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.szxy</groupId>
    <artifactId>Spring-session-mongodb</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>session_mongo1</module>
        <module>session_mongo2</module>
    </modules>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
    </parent>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.session</groupId>
                <artifactId>spring-session-bom</artifactId>
                <version>Bean-SR3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-mongodb</artifactId>
        </dependency>
    </dependencies>

</project>
```

#### session_mongo1 子模块 ####

- controller

```java
@RestController
@RequestMapping("service1")
public class WebController {

    @RequestMapping("/set")
    public String sendMsg(String msg, HttpSession session) {
        session.setAttribute("msg", msg);
        return "ok";
    }
    
    @RequestMapping("/setUser")
    public String sendMsg(Integer userId,String userName, HttpSession session) {
    	Users user = new Users();
    	user.setUserId(userId);
    	user.setUserName(userName);
    	session.setAttribute("user", user);
    	return "ok User";
    }
}
```

- Spring Boot 启动类

```java
@SpringBootApplication
@EnableMongoHttpSession
public class Mongo1App {
    public static void main(String[] args) {
        SpringApplication.run(Mongo1App.class,args);
    }
}

```

#### session_mongo2 子模块 ####

- controller

```java
@RestController
@RequestMapping("/service2")
public class WebController {
    @RequestMapping("/get")
    public String getMsg(HttpSession session){
        String msg = (String) session.getAttribute("msg");
        return msg;
    }
    @RequestMapping("/getUser")
    public Users getUser(HttpSession session){
    	Users user = (Users) session.getAttribute("user");
    	return user;
    }
}
```

- Spring Boot 启动类

```java
@SpringBootApplication
@EnableMongoHttpSession
public class Mongo2App {
    public static void main(String[] args) {
        SpringApplication.run(Mongo2App.class,args);
    }
}
```



### Spring session mongoDB 存储结构 ###



```shell
> use szxy
switched to db szxy
> show tables
sessions
> db.sessions.find()
{ 
"_id" : "8115216d-d130-4904-a3e8-a4afa2dd2b23",
"created" : ISODate("2019-08-13T13:50:36.664Z"),
"accessed" : ISODate("2019-08-13T13:57:04.235Z"), 
"interval" : "PT30M", "principal" : null,
"expireAt" : ISODate("2019-08-13T14:27:04.235Z"),
"attr":BinData(0,"rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmDRAwACRgAKbG9hZEZhY3RvckkACXRocmVzaG9sZHhwP0AAAAAAAAx3CAAAABAAAAACdAADbXNndAAKaGVsbG8xMTExMXQABHVzZXJzcgATY29tLnN6eHkucG9qby5Vc2VycyvvyrciydKfAgACTAAGdXNlcklkdAATTGphdmEvbGFuZy9JbnRlZ2VyO0wACHVzZXJOYW1ldAASTGphdmEvbGFuZy9TdHJpbmc7eHBzcgARamF2YS5sYW5nLkludGVnZXIS4qCk94GHOAIAAUkABXZhbHVleHIAEGphdmEubGFuZy5OdW1iZXKGrJUdC5TgiwIAAHhwAAAAAXQACOW8oOS4iTExeA==") 
}
```



### @EnableMongoHttpSession 注解 ###

> 注意：若要设置 session 的失效时间，必须把该 session 相关所有服务的 session 失效时间都设置为一致的

| 属性                           | 作用                               |
| ------------------------------ | ---------------------------------- |
| maxInactiveIntervalInSeconds： | 设置 Session 失效时间              |
| collectionName：               | 设置 MongoDB 的 Collections 的名称 |

```java
@SpringBootApplication
@EnableMongoHttpSession(collectionName="szxycollection")
public class Mongo2App {
    public static void main(String[] args) {
        SpringApplication.run(Mongo2App.class,args);
    }
}
```



### 更换序列化器 ###

Spring Session MongoDB 默认使用的是 JDK 序列化器。 

#### **创建配置类添加** **Jackson** **序列化器** ####

```java
@Configuration
public class SessionMongoConfig {
	
	/**
	 * 获取  JacksonMongoSessionConverter 序列器
	 * @return
	 */
	@Bean
	JacksonMongoSessionConverter mongoSessionConverter() {
		return new JacksonMongoSessionConverter();
	}

}
```

#### 在实体类上添加注解 ####

```java
@JsonAutoDetect(fieldVisibility=JsonAutoDetect.Visibility .ANY)
public class Users implements Serializable{
	
	private Integer userId;
	private String userName;
	@Override
	public String toString() {
		return "Users [userId=" + userId + ", userName=" + userName + "]";
	}
	public Integer getUserId() {
		return userId;
	}
	public void setUserId(Integer userId) {
		this.userId = userId;
	}
	public String getUserName() {
		return userName;
	}
	public void setUserName(String userName) {
		this.userName = userName;
	}
	
}
```

