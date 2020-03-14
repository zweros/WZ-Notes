---
title: 20190801 Spring Boot
date: 2019-08-01
---

[TOC]



## Spring Boot 介绍 ##

> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".
>
> We take an opinionated view of the Spring platform and third-party  libraries so you can get started with minimum fuss. Most Spring Boot  applications need very little Spring configuration.

Spring Boot 2.00 以上 对 JDK 版本有要求，即 JDK 1.8 以上

Spring  Boot 2.0 一下 ，使用  JDK  1.7 

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.21.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
</parent>
```

## 构建 SpringBoot 项目 Maven 工程 ##

- 继承  SpringBoot 父项目
- 修改 pom.xml 文件
- 注入 SpringBoot 启动器坐标

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.szxy.springboot</groupId>
	<artifactId>01-SpringBoot-HelloWorld</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
	
	<properties>
		<java.version>1.7</java.version>
	</properties>

	<dependencies>
		<!--  SpringBoot Web 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
        </dependency>
	</dependencies>

</project>
```

#### SpringBoot 启动器 ####

> 一些 jar 的集合，SpringBoot 共提供了 44 中 jar 集合

| jar 集合名                | 作用                                                     |
| ------------------------- | -------------------------------------------------------- |
| spring-boot-starter-web   | 支持全栈式的 web 开发，包括了 romcat 和 springMVC 等 jar |
| spring-boot-starter-jdbc  | 支持 spring 以 jdbc 方式操作数据库的 jar 包的集合        |
| spring-boot-starter-redis | 支持 redis 键值存储的数据库操                            |

#### SpringBoot 启动类 ####

```java
package com.szxy.controller;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/***
 * SpringBoot 启动类
 * 
 * 注意：SpringBoot 启动类的位置应该放在 Controller 包下，或者其父包中
 * 不能将 SpringBoot 启动类置于 Controller 包同级包中,否则访问会出现 Whitelabel Error Page 字样
 *
 */
@SpringBootApplication
public class App {
	
	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
}
```

#### SpringBoot  自动配置的位置

这里查看自动配置的信息

![](http://img.zwer.xyz/blog/20191127093059.png)

## SpringBoot 整合 Servlet ##

###  通过注册扫描完成 Servelt 的注册 ###

- FirstServlet .java

```java
/***
 * 基于注解扫描完成 Servlet 注册
 *
 */
@WebServlet(name="firstServlet",urlPatterns="/first")
public class FirstServlet extends HttpServlet{

	private static final long serialVersionUID = 1L;
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		System.out.println("FirstServlet.doGet() ......");
	}

}
```

- App.java：SpringBoot

```java
@SpringBootApplication  
@ServletComponentScan    //扫描 Servlet 组件
public class App {

	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
	
}

```



### 通过方法完成Servlet 的注册 ###

- SecondServlet.java

```java
/**
 * 基于方法完成 Servlet 注册
 *
 */
public class SecondServlet extends HttpServlet{
	
	private static final long serialVersionUID = 1L;

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		System.out.println("SecondServlet.doGet()   ...");
	}
	
	
}

```

- App2.java：SpringBoot 启动类

```java
@SpringBootApplication
public class App2 {

	public static void main(String[] args) {
		SpringApplication.run(App2.class, args);
	}
	
	@Bean
	public  ServletRegistrationBean getServletRegistrationBean(){
		ServletRegistrationBean bean =
            new ServletRegistrationBean(new SecondServlet());
		bean.addUrlMappings("/second");
		return bean;
	}
	
}

```

## SpringBoot 整合 Filter ##

### 通过注册扫描完成 Filter 注册 ###

- FirstFilter.java

```java
/**
 * SpringBoot 整合 filter 方式一
 *
 */
@WebFilter(filterName="firstFilter",urlPatterns="/first")
public class FirstFilter implements Filter{

	@Override
	public void destroy() {
	}

	@Override
	public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
			throws IOException, ServletException {
			System.out.println("拦截执行前...");
			chain.doFilter(req, resp);
			System.out.println("拦截执行后 ...");
	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
		
	}

}
```

- App.java: SpringBoot 启动类

```java
@SpringBootApplication  
@ServletComponentScan    //扫描 Servlet 组件 和 Filter 组件
public class App {

	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
	
}

```

### 通过方法完成 Filter 注册 ###

- SecondFilter.java

```java
/**
 * SpringBoot 整合 Filter 方式二
 *
 */
public class SecondFilter implements Filter {

	@Override
	public void destroy() {
		
	}

	@Override
	public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
			throws IOException, ServletException {
			System.out.println("拦截执行前...");
			chain.doFilter(req, resp);
			System.out.println("拦截执行后 ...");
	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
		
	}

}

```

- App2.java: SpringBoot 启动类

```java
public class App2 {

	public static void main(String[] args) {
		SpringApplication.run(App2.class, args);
	}
	
	/***
	 * 通过方法完成 Filter 注册
	 * @return
	 */
	@Bean
	public FilterRegistrationBean getFilterRegistrationBean(){
		FilterRegistrationBean bean =
				new FilterRegistrationBean(new SecondFilter());
		//bean.addUrlPatterns("/second");
		bean.addUrlPatterns(new String[]{"*.do","*.jsp"});
		return bean;
	}
	
}

```

## SpringBoot 整合 Listener  ##

### 通过注解完成 Listener 注册 ###

- FirstListener.java:

```java
/**
 * 监听 ServletContext 启动
 * @author zwer
 *
 */
@WebListener
public class FirstListener implements ServletContextListener{

	@Override
	public void contextDestroyed(ServletContextEvent arg0) {
		
	}

	@Override
	public void contextInitialized(ServletContextEvent arg0) {
		System.out.println("FirstListener.contextInitialized()  first 初始化 ...");
	}

}

```

- App.java: SpringBoot 启动类

```java
@SpringBootApplication  
@ServletComponentScan    //扫描 Servlet 组件
public class App {

	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
	
}
```

### 通过方法完成 listener 注册 ###

- SecondListener.java 

```java
public class SecondListener implements ServletContextListener{

	@Override
	public void contextDestroyed(ServletContextEvent arg0) {
		
	}

	@Override
	public void contextInitialized(ServletContextEvent arg0) {
		System.out.println("SecondListener.contextInitialized() Second 初始化 ...");
	}

}

```

- App2.java:SpringBoot 启动类

```java
@SpringBootApplication
public class App2 {

	public static void main(String[] args) {
		SpringApplication.run(App2.class, args);
	}
	
	
	/**
	 * 通过方法完成 Listener 注册
	 */
	@Bean 
	public ServletListenerRegistrationBean<SecondListener> 
		getServletListenerRegistrationBean(){
		
		ServletListenerRegistrationBean<SecondListener> bean = 
            
		new ServletListenerRegistrationBean<SecondListener>(new SecondListener());
		
		return bean;
		
		
	}
}
```

## SpringBoot 访问静态资源 ##

####  classpath:static 目录访问静态资源 ####

> 注意：static 目录名称不能修改

![](http://img.zwer.xyz/blog/20190801165722.png)



####  ServletContext 根目录下 ####

> 在 src/main/webapp 目录，注意 webapp 目录不能修改

## SpringBoot 文件上传 ##

- 编写 上传页面

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>上传文件</title>
</head>
<body>
	<form action="/upload" method="post" enctype="multipart/form-data">
			<input type="file"  name="file" />
			<input type="submit" value="upload"/>
	</form>
</body>
</html>
```

- 编写 Controller 

```java
//@Controller
@RestController //将方法的返回值做 json 格式处理
public class FileUploadController {

	@RequestMapping("/upload")
	public Map<String,Object> fileUpload(@RequestParam MultipartFile file) throws Exception{
		Map<String, Object> map = new HashMap<String,Object>();
		if(file != null){
			String uploadFileName = file.getOriginalFilename();
			System.out.println(uploadFileName);
			// 将上传文件保存到服务器本地磁盘中
			file.transferTo(new File("D:/desktop/"+uploadFileName));
			map.put("msg", "ok");
		}else{
			map.put("mgs", "error");
		}
		return map;
	}	
}
```

- 设置文件上传参数

> 添加 `application.properties `文件,位置在 `src/main/resources` 目录下
>
> 注意：`application.properties `名称不能修改

```properties
spring.http.multipart.maxFileSize=200MB
spring.http.multipart.maxRequestSize=200MB
```

## SpringBoot 整合 JSP  ##

- 创建 Users.java 实体类

```java
public class Users {
	private Integer userid;
	private String username;
	private Integer userage;
	public Users() {
		super();
	}
	public Users(Integer userid, String username, Integer userage) {
		super();
		this.userid = userid;
		this.username = username;
		this.userage = userage;
	}
	public Integer getUserid() {
		return userid;
	}
	public void setUserid(Integer userid) {
		this.userid = userid;
	}
	public String getUsername() {
		return username;
	}
	public void setUsername(String username) {
		this.username = username;
	}
	public Integer getUserage() {
		return userage;
	}
	public void setUserage(Integer userage) {
		this.userage = userage;
	}
	@Override
	public String toString() {
		return "Users [userid=" + userid + ", username=" + username + ", userage=" + userage + "]";
	}
	
}

```

- 创建 UsersController.java 控制器

```java
@Controller
public class UsersController {
		
	@RequestMapping("/showUser")
	public String showUser(Model model){
		List<Users> userList = new ArrayList<>();
		userList.add(new Users(1,"张三",20));
		userList.add(new Users(2,"王五",10));
		userList.add(new Users(3,"李四",30));
		model.addAttribute("userList", userList);
		return "showUser";
	}
	
}

```

- **创建 application.properties ，位置放在 src/main/resources 目录下**

```properties
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```

- 创建 showUser.jsp 视图界面

```java
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"  %>

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>展示用户信息</title>
</head>
<body>
	<table border="1px" cellpadding="0">
		<tr>
			<th>ID</th>
			<th>Name</th>
			<th>Age</th>
		</tr>
		<c:forEach items="${userList}" var="user">
			<tr>
				<td>${user.userid}</td>
				<td>${user.username}</td>
				<td>${user.userage}</td>
			</tr>
		</c:forEach>		
	</table>
</body>
</html>
```

- 创建 App.java: SpringBoot 启动类

```java
@SpringBootApplication
public class App {

	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
	
}
```

## SpringBoot 注释总结 ##

- 关于注解

| 注解                   | 作用                                                         |
| ---------------------- | ------------------------------------------------------------ |
| @SpringBootApplication | 用于 SpringBoot 启动类上，且启动类的位置要位于 controller 包的当前包中或者父级包中，不能出现 controller 同级包中 |
| @RestController        | 相当于 注解 @controller 和 @ResponseBody 的组合，将方法的返回值以 json 格式的形式返回 |
| @Bean                  | SpringBoot 整合 Servlet、Filter、Listener  时，使用到，通过方法注册 Servlet、Filter、Listener |
| @WebServlet            | 配置 Servlet 的拦截路径                                      |
| @WebFilter             | 配置 Filter 的拦截路径                                       |
| @WebListener           | 表示该类是个监听类                                           |

- 静态资源存放位置
  1. 放在 src/main/resources/static 目录下，必须 是 static 目录下，**不能修改**
  2. 放在 src/main/webapp 目录下，必须是 webapp 目下，**不能修改**

- SpringBoot 配置文件

  名称必须叫 application.properties,作用如配置上传文件的大小，视图解析器

```properties
spring.http.multipart.maxFileSize=200MB
spring.http.multipart.maxRequestSize=200MB

spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```



## SpringBoot 整合 freemarker  ##

> FreeMarker 是一款 *模板引擎*： 即一种基于模板和要改变的数据， 并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。
>
> 模板编写为FreeMarker Template Language (FTL)。它是简单的，专用的语言， *不是* 像PHP那样成熟的编程语言。 那就意味着要准备数据在真实编程语言中来显示，比如数据库查询和业务运算， 之后模板显示已经准备好的数据。在模板中，你可以专注于如何展现数据， 而在模板之外可以专注于要展示什么数据。

![](http://freemarker.foofun.cn/figures/overview.png)



- 创建项目，添加 pom 文件坐标依赖

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.21.RELEASE</version>
  </parent>
  <groupId>com.szxy.springboot</groupId>
  <artifactId>08-SpringBoot-view-freemarker</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <dependencies>
  	<dependency>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-web</artifactId>
  	</dependency>
  	<dependency>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-freemarker</artifactId>
  	</dependency>
  </dependencies>
</project>
```

- 编写视图

注意：SpringBoot 要求模式形式的视图层技术**必须放到 src/main/resources/templates 目录下**

```html
<html>
<head>
	<meta charset="utf-8">
	<title>展示用户信息</title>
</head>
	<table border="1" width="50%" align="center">
		<tr>
			<th>ID</th>
			<th>Name</th>
			<th>Age</th>
		</tr><br />
		<#list userList as user>
			<tr>
				<td>${user.userid}</td>
				<td>${user.username}</td>
				<td>${user.userage}</td>
			</tr>
		</#list>
	</table>
</html>
```

## SpringBoot 整合 thymeleaf  ##

### Thymeleaf 开始 ###

- 添加 SpringBoot 与 Thymeleaf 整合的启动器

```xml
<!--Spring boot 与 thymeleaef 整合-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

- HTML 网页 thymeleaf 的头部声明

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
   
    
</html>
```

### 创建存放视图的目录 ###

目录位置：**src/main/resources/templates**

> templates : 该目录是安全的。意味着该目录下的内容不允许外界访问。

### Thymeleaf 的基本使用 ###

**Thymeleaf 的特点**

> Thymeleaf 是通过它特定语法对 html 的标记做渲染

**Thymeleaf 的使用**

Thymeleaf 遵守严谨的 HTML 语法

```html
<span th:text="Hellothymeleaf"></span>
<hr/>
<span th:text="${msg}"></span>
```

**出现异常**

![](http://img.zwer.xyz/blog/20190801201753.png)

原因：Thymeleaf 遵守严谨的 HTML 语法，所以导致异常出现，但高版本的 thymeleaf 允许不严谨的 HTML 语法

```xml
<!--  替换 thymeleaf 的版本  -->
<thymeleaf.version>3.0.2.RELEASE</thymeleaf.version>
<thymeleaf-layout-dialect.version>
    2.0.4
</thymeleaf-layout-dialect.version>
```

### 变量输出与字符串操作

Thymeleaf 内置对象

注意语法：

- 调用内置对象一定要用  `#`
- 大部分的内置对象都以 s 结尾 strings、number、dates

| 标记                                                         | 作用                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| th:text                                                      | 在页面中输出值                                               |
| th:value                                                     | 可以将一个值放入到 input 标签的 value 中                     |
| th:filed                                                     | 用于填写表单的 value 属性的值                                |
| th:errors                                                    | 用于回显错误信息                                             |
| ${#strings.isEmpty(key)}                                     | 判断字符串是否为空，如果为空返回 false，否则返回 true        |
| ${#strings.contains(msg,'T')}                                | 判断字符串是否包含指定的字串                                 |
| $(#strings.startsWith(msg,'a'))                              | 判断字符串是否以字串开头                                     |
| $(#strings.endsWith(msg,'b'))                                | 判断字符串是否以字串结尾                                     |
| $(#strings.length(mgs))                                      | 判断字符串的长度                                             |
| $(#strings.indexOf(msg),'h')                                 | 查找字串的位置，并返回该字串你的下标，如果没有找到，则返回 -1 |
| `$(#strings.substring(msg,起始位置))`<br/>$`(#strings.substring(msg,起始位置，结果位置))` | 截取字串，用法与 JDK String 类的用法相同                     |
| $(#strings.toUpperCase(msg))                                 | 将英文单词从小写变为大写                                     |
| $(#strings.toLowerCase(mgs))                                 | 将英文单词从大写变为小写                                     |

### 日期格式化处理

| 标记                                                         | 作用                               |
| ------------------------------------------------------------ | ---------------------------------- |
| $(#dates.format(key))                                        | 格式化日期，按照浏览器默认语言处理 |
| $(#dates.format(key,'yyyy-MM-dd '))                          | 按照自定义的格式作日期格式转换     |
| `$(#dates.year(key))`<br/>`$(#dates.month(key))`<br/>`$(#dates.day(key))`</br> | 获取单个的年、月、日信息           |

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>thymeleaf 学习</title>
</head>
<body>
	<span th:text="Hellothymeleaf"></span>
	<hr/>
	<span th:text="${msg}"></span>
	<hr/>
	<input th:value="${msg}"  type="text" />
	<hr/>
	<h2>字符串操作</h2>
	key:<span th:text="${#strings.isEmpty(key)}"></span><br/>
	msg:<span th:text="${#strings.isEmpty(msg)}"></span>
	<hr/>
	88:<span th:text="${#strings.contains(msg,'88')}"></span>
	th:<span th:text="${#strings.contains(msg,'th')}"></span>
	<hr/>
	th:<span th:text="${#strings.startsWith(msg,'th')}"></span>
	开始：<span th:text="${#strings.endsWith(msg,'开始')}"></span>
	<hr/>
	该字符串的长度<span th:text="${#strings.length(msg)}"></span>
	<hr/>
	将小写转为大写 <span th:text="${#strings.toUpperCase(msg)}"></span>
	将大写转为小写 <span th:text="${#strings.toLowerCase(upperCase)}"></span>
	<hr />
	f 字符的下标的位置：<span th:text="${#strings.indexOf(msg,'f')}"></span>
	<hr />
	2 to ：<span th:text="${#strings.substring(msg,2)}"></span>
	<hr />
	2 to 10 :<span th:text="${#strings.substring(msg,2,10)}"></span>
	<h2>日期格式操作</h2>
	<span th:text="${today}"></span>
	<hr/>
	<span th:text="${#dates.format(today)}"></span>
	<hr/>
	<span th:text="${#dates.format(today,'yyyy-MM-dd HH:mm:ss')}"></span>
	<hr/>
	年：<span th:text="${#dates.year(today)}"></span>&nbsp;
	月：<span th:text="${#dates.month(today)}"></span>&nbsp;
	天：<span th:text="${#dates.day(today)}"></span>
</body>
</html>
```

### 条件判断

| 标记                    | 作用    |
| ----------------------- | ------- |
| th:if="${sex} == ‘男’ " | if 判断 |
| th:switch               |         |
| th:case                 |         |

```html
<h2>If 语句判断</h2>
	<span th:if="${gender} == 'male'">
		性别： 男
	</span>
	<span th:if="${gender} == 'female'">
		性别：女
	</span>
	
	<h2>switch 语句判断</h2>
	ID:<span th:switch="${id}">
		<span th:case="1">1</span>
		<span th:case="2">2</span>
		<span th:case="3">4</span>
	</span>
```

### 迭代遍历

> th:each

- List 集合遍历

```html
<h2> List 集合遍历</h2>
<table border="1px"> 
		<tr>
			<th>ID</th>
			<th>Name</th>
			<th>Age</th>
			<th>first</th>
			<th>last</th>
			<th>even</th>
			<th>odd</th>
			<th>size</th>
			<th>count</th>
			<th>index</th>
		</tr>
		<!-- var 表示状态变量 -->
		<tr th:each="user,var : ${userList}">
			<td th:text="${user.userid}"></td>
			<td th:text="${user.username}"></td>
			<td th:text="${user.userage}"></td>
			<td th:text="${var.first}"></td>
			<td th:text="${var.last}"></td>
			<td th:text="${var.even}"></td>
			<td th:text="${var.odd}"></td>
			<td th:text="${var.size}"></td>
			<td th:text="${var.count}"></td>
			<td th:text="${var.index}"></td>
		</tr>
</table>
```

![](http://img.zwer.xyz/blog/20190802110058.png)



| 状态变量属性 | 作用                                 |
| ------------ | ------------------------------------ |
| index        | 当前迭代器的索引                     |
| count        | 当前迭代器的计算，从 1 开始          |
| size         | 被迭代对象的程度                     |
| even/odd     | 布尔值，判断奇偶数，                 |
| first/last   | 布尔值，判断是否是第一个或者最后一个 |

- Map 集合遍历

```html
<h2> Map 集合遍历</h2>
	<table border="1px">
		<tr>
			<th>key</th>
			<th>ID</th>
			<th>Name</th>
			<th>Age</th>
			<th>ID</th>
			<th>Name</th>
			<th>Age</th>
		</tr>
		<tr th:each="map :${usersMap}">
			<td th:text="${map.key}">
			<td th:text="${map.value.userid}">
			<td th:text="${map.value.username}">
			<td th:text="${map.value.userage}">
			<!-- 第二种方式 -->
			<td th:each="entry : ${map}" th:text="${entry.value.userid}"></td>
			<td th:each="entry : ${map}" th:text="${entry.value.username}"></td>
			<td th:each="entry : ${map}" th:text="${entry.value.userage}"></td>
		</tr>
	</table>
```

​	![](http://img.zwer.xyz/blog/20190802110131.png)

### 域对象操作

```html
<h2>获取作用域中数据</h2>
<!--
	request.setAttribute("request", "HttpServletRequest");
 -->
request:<span th:text="${#httpServletRequest.getAttribute('request')}"></span><br/>
<!--
	request.setAttribute("request", "HttpServletRequest");
 -->
request2:<span th:text="${request}"></span><br/>
<!--
	request.getSession().setAttribute("session", "HttpSession");
 -->
session:<span th:text="${session.session}"></span><br/>
<!--
	request.getServletContext().setAttribute("context", "Application");
 -->
servletContext:<span th:text="${application.context}"></span>
<hr/>
```

### URL 表达式

`@{URI}`

| 标记                                                         | 作用                         |
| ------------------------------------------------------------ | ---------------------------- |
| `<a th:href="@{http://www.baiud.com}">百度</a>`              | 绝对路径                     |
| `<a th:href="@{/show}">点击</a>`                             | 相对于项目上下文的相对路径 / |
| `<a th:href="@{~/project/show}">点击</a>`                    | 相对于服务器根路径  ~        |
| `<a th:href="@{/show(id=1,name=zhangsan)}">`点击</a>`        | 相对路径传参                 |
| `<a th:href="@{/users/deleteUser/{id}(id=${user.userid})}">					删除</a>` | 相对路径传参数—Restful 风格  |

```html
<h2> URL  表达式</h2>
<img alt="图片" src="img/1.jpg" width="200px" height="100px"/>
<img alt="图片" th:src="@{img/1.jpg}" width="200px" height="100px"/>
<hr/>
<a href="http://www.baidu.com">百度一下</a>&nbsp;&nbsp;
<a th:href="@{http://www.baidu.com}">百度一下</a>
<hr/>
<a href="/show">show</a>&nbsp;&nbsp;
<a th:href="@{/show}">show2</a>
<hr/>
<a th:href="@{/show(id=1,name=zhangsan)}">相对路径传参</a>&nbsp;&nbsp;
<a th:href="@{/show/1/zhangsan}">Restful 风格传参</a>

<!-- Restful 风格传参 -->
<a th:href="@{'/user/preUpdateUser/'+${u.userid}}">更新</a> 
<a th:href="@{'/user/deleteUser/'+${u.userid}}">删除</a>

<!-- 相对路径传参 -->
<a th:href="@{/user/preUpdateUser(userid=${u.userid})}">更新</a>
<a th:href="@{/user/deleteUser(userid=${u.userid})}">删除</a>
```

## Spring boot 整合thymleaf 常用配置 

```yml
spring: 
  thymeleaf:
    prefix: classpath:/templates/ # thymeleaf 视图解析器后缀
    suffix: .html  # thymleaf 视图解析器前缀
    mode: HTML  #配置视图模板类型，如果视图是 HTML5 需要配置
    encoding: utf-8
    cache: false
    servlet: 
      content-type: text/html #响应类型

```

## SpringBoot 整合 JDBC 

### 创建 SpringBoot 整合 JDBC 项目 Demo

创建 Maven 项目，并修改 pom.xml 文件，增加  SpringBoot 与 web、thymeleaf 、JDBC 启动器坐标以及 MySQL 数据库驱动

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.6.RELEASE</version>
	</parent>

	<groupId>com.szxy</groupId>
	<artifactId>jdbc_Demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<dependencies>
		<!-- SpringBoot-web 启动类坐标 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- thymeleaf 启动器坐标 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<!-- SpringBoot 整合 jdbc 启动器坐标 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		<!-- MySQL 数据库驱动 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.38</version>
		</dependency>
		<!-- Druid 数据库连接池依赖 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.1.11</version>
		</dependency>
	</dependencies>
	<build />
</project>
```

### 通过 @PropertySource 加载配置信息

通过 Spring 框架中 @PropertySource 注解，加载指定的配置文件信息

- **jdbc.properties 自定义 jdbc 配置文件信息**

  ```properties
  jdbc.driverClassName=com.mysql.jdbc.Driver
  jdbc.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false
  jdbc.username=root
  jdbc.password=root
  ```

- **JDBCConfiguration.java**是JDBC 配置类，加载配置 JDBC 自定义配置文件信息，并获取 DataSource 对象

  ```java
  @Configuration
  @PropertySource("classpath:/jdbc.properties") // 加载指定的 properties 文件
  public class JDBCConfiguration {
  	
  	@Value("${jdbc.driverClassName}")
  	private String driverClassName;
  	@Value("${jdbc.url}")
  	private String url;
  	@Value("${jdbc.username}")
  	private String username;
  	@Value("${jdbc.password}")
  	private String password;
  	
  	@Bean
  	public DataSource getDruidDataSource() {
  		DruidDataSource dataSource = new DruidDataSource();
  		dataSource.setDriverClassName(this.driverClassName);
  		dataSource.setUrl(this.url);
  		dataSource.setUsername(this.username);
  		dataSource.setPassword(this.password);
  		return dataSource;
  	}
  	
  }
  ```

### 通过 @ConfigurationProperties  加载配置信息

注意： @ConfigurationProperties 注解由 SpringBoot 提供，且只能加载 SpringBoot 的配置文件信息（`application[.properties][.yml]`）

- **application.yml** （SpringBoot 配置文件）

  ```yml
  # 自定义 jdbc 配置文件信息，由指定的自定义配置类 JdbcProperties 加载，解析
  jdbc:
    driverClassName: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false
    username: root
    password: root
  ```

- **JdbcProperties.java**  

  将 Jdbc 配置信息单独抽取成一个配置类，可以达到  jdbc 配置信息复用的目的，减少多余代码的书写。

  另外这里使用到 `@Configuration 注解 和 @ConfigurationProperties(prefix=)` 注解， `@ConfigurationProperties(prefix=)`  表示通过加载 SpringBoot 配置信息，解析指定前缀 prefix 的配置信息，然后通过 JdbcProperties 中 setter 方法注入到 JdbcProperties 对象中

  ```java
  /**
   * @author zwer
   * JDBC 自定义配置类
   * 
   */
  @Configuration
  @ConfigurationProperties(prefix="jdbc")  // 加载 jdbc 为前缀的配置信息
  public class JdbcProperties {
  	private String driverClassName;
  	private String url;
  	private String username;
  	private String password;
  
  	// 这里省略  setter 和 getter 方法
  }
  ```

- **JDBCConfiguration.java** 

  1. 使用  JdbcProperties 对象，需要在类上添加    @EnableConfigurationProperties  注解

  2. 这里有三种方式注入 JdbcPropreties 对象

     第一种：通过 @Autowired  注解

     第二种：通过 构造方法注入
  
     第三种： 通过方法参数注入 （最简便）
  
  ```java
    @Configuration
    @EnableConfigurationProperties
    public class JDBCConfiguration {
    	
    	// 1.  @Autowried 注入
    	// @Autowired
    	// private JdbcProperties jdbcProperties; 
    	
    	/**
    	  * 2.  构造注入
    	 * @param jdbcProperties
    	 */
    //	public JDBCConfiguration(JdbcProperties jdbcProperties) {
    //		this.jdbcProperties = jdbcProperties;
    //	}
    	
    	/**
    	 *	3. 通过方法参数注入
    	 *获取数据源  DataSource 对象
    	 *   
    	 * @return DataSource 
    	 */
    	@Bean
    	public DataSource getDruidDataSource(JdbcProperties jdbcProperties) {
    		DruidDataSource dataSource = new DruidDataSource();
    		dataSource.setDriverClassName(jdbcProperties.getDriverClassName());
    		dataSource.setUrl(jdbcProperties.getUrl());
    		dataSource.setUsername(jdbcProperties.getUsername());
    		dataSource.setPassword(jdbcProperties.getPassword());
    		return dataSource;
    	}
    	
    }
    
  ```
  
###  @ConfigurationProperties   注解优雅使用方式

- application.yml

  ```yml
  # 自定义 jdbc 配置文件信息，由指定的自定义配置类 JdbcProperties 加载，解析
  jdbc:
    driverClassName: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false
    username: root
    password: root
  ```

- JDBCConfiguration.java  

  ```java
  @Configuration
  public class JDBCConfiguration {
  	@Bean
  	@ConfigurationProperties(prefix="jdbc") // 优雅注解
  	public DataSource getDruidDataSource() {
  		DruidDataSource dataSource = new DruidDataSource();
  		return dataSource;
  	}
  }
  ```

### 通过 SpringBoot 配置文件配置数据源 ###

SpringBoot 2.x 中 springboot-starter-jdbc 采用 com.zaxxer.hikari.HikariDataSource 作为默认数据源 
当然也可以指定第三方数据源 Druid 

```java
spring:
  datasource:
    driverClassName: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false
    username: root
    password: root
    type: com.alibaba.druid.pool.DruidDataSource
```






## SpringBoot 整合 Mybatis  ##

技术：SpringBoot+MyBatis+Thymeleaf 实现对 users 表的 CRUD 操作

@Mappercan  扫描 Mybatis 的接口配置文件

- 创建 Maven 项目，修改 pom.xml 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.10.RELEASE</version>
	</parent>
	<groupId>com.szxy.springboot</groupId>
	<artifactId>10-SpringBoot-thymeleaf-mybatis</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.7</java.version>
		<thymeleaf.version>3.0.2.RELEASE</thymeleaf.version>
		<thymeleaf-layout-dialect.version>
			2.0.4
		</thymeleaf-layout-dialect.version>
	</properties>
	<dependencies>
		<!-- SpringBoot 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- thymeleaf 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<!-- Mybatis 启动器 -->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.1.1</version>
		</dependency>
		<!-- mysql 数据库驱动 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		 <!-- druid 数据库连接池 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.0.9</version>
		</dependency>
	</dependencies>
</project>
```

- 在 src/main/resources 目录下，创建 application.propreties 文件

```properties
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/ssm
spring.datasource.username=root
spring.datasource.password=root

spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

mybatis.type-aliases-package=com.szxy.pojo
```

- Mapper层 、Service 层、 Controller 层面省略

```html

<span th:if="${user}  == null">
    <h2>添加用户信息</h2>
    <form th:action="@{/user/addUser}" method="post">
        用户姓名：<input type="text" name="username"/><br />
        用户年龄：<input type="number" name="userage"/><br />
        <input type="submit" value="提交" />
    </form>
</span>
<span th:if="${user} != null">
    <h2>更新用户信息</h2>
    <form th:action="@{/user/updUser}" method="post">
  <input type="hidden" name="userid" th:field="${user.userid}">
 用户姓名：<input type="text" name="username" th:field="${user.username}" /><br />
 用户年龄：<input type="number" name="userage" th:field="${user.userage}" /><br />
        <!-- 
<input type="hidden" name="userid" th:value="${user.userid}">
用户姓名：<input type="text" name="username" th:value="${user.username}" /><br />
用户年龄：<input type="number" name="userage" th:value="${user.userage}" /><br />
-->
        <input type="submit" value="提交" />
    </form>
</span>
```



- SpringBoot 启动类

```java
@SpringBootApplication
@MapperScan("com.szxy.mapper")  //用户扫描 Mybatis 的Mapper 接口
public class App {

	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
	
}
```

**出现的异常**

```java
//异常一
SpringBoot 整合 Mybatis 出现的错误：At least one base package must be specified

/****
	由于 ：在主函数上没有加上对mapper层的扫描MapperScan注解
	@MapperScan("com.szxy.mapper") 
****/


//异常二
Description:

Field usersService in com.szxy.controller.UsersController required a bean of type 'com.szxy.service.UsersService' that could not be found.


Action:

Consider defining a bean of type 'com.szxy.service.UsersService' in your configuration.


/****
	这是由于 SpringBoot 启动类放错了位置，放在 com.szxy.controller 包下，导致程序出现异常。
	分析：SpringBoot 启动类放在 com.szxy.controller 包下，只能扫描到 controller 包下的文件，  但是同级包 com.szxy.service ，SpringBoot 启动类是无法扫描到的。
	解决：把 SpringBoot 启动类放在 com.szxy 包下，即controler 包和 service 包的父级包中
	
****/
```

## SpringBoot 数据校验 ##

### SpringBoot 对表单做数据校验 ###

> SpringBoot 采用 Hibernate-validate 校验框架。 效率很差，通常开发时不建议使用
>

- 在实体类上添加校验规则（在成员属性上添加 @NotBlack 注解）

```java
private Integer id;
	@Length(max=4,min=2)
	@NotBlank(message="用户名不能为空")
	private String name;
	@NotEmpty
	private String pwd;
	@Max(300)
	@Min(15)
	private Integer age;
	@Email(message="邮箱格式不对")
	private String email;
	
```

-  Controller 中开启校验（添加 @Valid 注解 、注入 BindingResult 对象 ）

```java
	
	/**
	 * 添加用户操作
	 * @param user   添加用户的信息
	 * @param result  封装了校验信息
	 * @return
	 */
	@RequestMapping("/user/addUser")
	public String addUser(@Valid User user,BindingResult result){
		if(result.hasErrors()){
			return "add";
		}
		user.setId(1);
		System.out.println(user);
		return "ok";
	}
```

- 添加用户界面（使用 `th:errors` 回显错误信息）

```html
<form th:action="@{/user/addUser}" method="post">
    用户姓名：<input type="text" name="name"/>
    <span th:errors="${user.name}" style="color:red;"></span><br/>
    用户密码：<input type="password" name="pwd"/>
    <span th:errors="${user.pwd}" style="color:red;"></span><br/>
    用户年龄：<input type="number" name="age"/><br/>
    <input type="submit"  value="提交"/>
</form>
```

- 解决异常

```java
There was an unexpected error (type=Internal Server Error, status=500).

Error during execution of processor 'org.thymeleaf.spring4.processor.SpringErrorsTagProcessor' 
(template: "add" - line 9, col 46)
    
	/**
	 * 解决异常的方式。可以在跳转页面的方法中注入一个 Uesr 对象。 
	 * 注意：由于 springmvc 会将该对象放入到 Model 中传递。
	 * key 的名称会使用 该对象的驼峰式的命名规则来作为 key。 
	 * 参数的变量名需要与对象的名称相同。将首字母小写
	 * 显示添加用户的界面
	 * 
	 * @ModelAttribute("cc") 自定义 key 的名称，然后将放入 Model 中 
	 * 
	 * @param user  
	 * @return
	 */
    @RequestMapping("/user/add")
    public String Add(User user){
    return "add";
}
```

### 表单数据校验注解总结 ###

| 注解            | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| @NotBlack       | 判断字符串是否 null 或者 空串（去掉首尾空格），即非空校验<br/>message 属性设置校验不通过显示的错误信息 |
| @NotEmpty       | 判断字符串是否 null 或者 空串（不去掉首尾空格），即允许空格作为输入数据 |
| @Length         | 判断字符的长度（最大或者最小）<br/>min 最短长度,max 最长长度 |
| @Min            | 判断数值最小值（可以等于最小值）                             |
| @Max            | 判断数值最大值                                               |
| @Email          | 判断邮箱是否合法                                             |
| @Valid          | 开启对 Users 对象的数据校验<br/>BindingResult:封装了校验的结果 |
| @ModelAttribute | 更改传递对象的名称，作为 Model 中 key                        |

`th:errors`  显示校验错误信息



## SpringBoot 异常处理 ##

#### 自定义错误页面 ####

SpringBoot 默认处理异常的机制，SpringBoot 默认提供了一套处理异常的机制。一旦程序中除了异常，便向 /errror 的 url 发送请求。在 SpringBoot 中提供了一个叫 BasicExceptionController 来处理 /error 请求，然后跳转到默认显示异常的页面来展示异常信息。

如果我们需要将所有的异常统一跳转到自定义的错误页面，需要将 src/main/resources/templates 目录下创建 error.html 页面，注意：名称必叫 error。

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>错误页面</title>
</head>
<body>
	当前访问量较大，请稍后访问！！！
	<br/>
	<span th:text="${exception}"></span>
</body>
</html>
```

#### @ExceptionHandler 注解处理异常 ####

> 只能处理当前 Controller 类中的特定异常，不能处理其他 Controller 类中异常

```java
/**
 * 该方法需要返回一个ModelAndView 
 * 目的是可以让我们封装异常信息以及视图的指定
 * 参数 Exception e：会将产生异常对象注入到方法中
 * 处理除数不能为零  x/0 异常
 * @param e
 * @return
 */
@ExceptionHandler(value=java.lang.NullPointerException.class)
public ModelAndView argumentHandle(Exception e){
    ModelAndView mv = new ModelAndView();
    mv.addObject("error", e);
    mv.setViewName("error1");
    return mv;
}
```

#### @ControllerAdvier+@ExceptionHandler   ####

> 好处：可以处理全局 Controller 类的异常

```java

@ControllerAdvice
public class GlobalController {
    
	@ExceptionHandler(value=java.lang.NullPointerException.class)
	public ModelAndView NullPointerExceptionHandle(Exception e){
		ModelAndView mv = new ModelAndView();
		mv.addObject("error", e);
		mv.setViewName("error1");
		return mv;
	}
	
	@ExceptionHandler(value=java.lang.ArithmeticException.class)
	public ModelAndView ArithmeticExceptionHandleHandle(Exception e){
		ModelAndView mv = new ModelAndView();
		mv.addObject("error", e);
		mv.setViewName("error2");
		return mv;
	}
	
}
```

####  SimpleMappingExceptionResolver ####

> 可以实现全局异常视图跳转，但不能传递异常信息

```java
package com.szxy.controller;

import java.util.Properties;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.handler.SimpleMappingExceptionResolver;

/**
 * 通过 SimpleMappingExceptionResolver 做全局异常处理
 *
 */

@Configuration
public class simpleGlobalExceptionResolver{

	@Bean
	public SimpleMappingExceptionResolver getSimpleMappingExceptionResolver(){
		Properties prop = new Properties();
		
		/*** 
		 * 参数一：异常的类型，注意必须是异常类型的全名 
		 * 参数二：视图名称
		 * 
		 */
		prop.put("java.lang.NullPointerException", "error1");
		prop.put("java.lang.ArithmeticException", "error2");
		
		SimpleMappingExceptionResolver  resolver = new SimpleMappingExceptionResolver();
		
		//设置异常与视图映射信息
		resolver.setExceptionMappings(prop);
		
		return resolver;
		
	}
	
	
}

```

#### HandleExceptionResolver  类异常 ####

> 既可以实现全局异常视图跳转，也可以传递异常信息

```java
package com.szxy.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

@Configuration
public class HandlerExceptionResolverImpl 
	   				implements HandlerExceptionResolver{

	@Override
	public ModelAndView resolveException(HttpServletRequest req, HttpServletResponse resp, Object arg2,
			Exception ex) {
		// 创建 ModelAndView 对象
		ModelAndView mv = new ModelAndView();
		//根据不同的异常类型，做不同页面跳转
		if(ex instanceof NullPointerException){
			mv.setViewName("error1");
		}
		if(ex instanceof ArithmeticException){
			mv.setViewName("error2");
		}
		//将异常信息放入 model 对象中
		mv.addObject("error", ex.toString());
		return mv;
	}

}

```

## Spring 整合 Junit 测试 ##

- 创建 Maven 项目，修改 pom.xml 文件

```ocaml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.10.RELEASE</version>
  </parent>
  <groupId>com.szxy.springboot</groupId>
  <artifactId>13-SpringBoot-Junit</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <properties>
  	<java.version>1.7</java.version>
  </properties>
  <dependencies>
  	<dependency>
  		 <groupId>org.springframework.boot</groupId>
    	 <artifactId>spring-boot-starter-web</artifactId>
  	</dependency>
  	<!--  Junit 启动器 -->
  	<dependency>
  		 <groupId>org.springframework.boot</groupId>
    	 <artifactId>spring-boot-starter-test</artifactId>
  	</dependency>
  </dependencies>
</project>
```

- UserServiceImplTest ：测试类

```java
package com.szxy.service;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.szxy.App;

/****
 * 
 * SpringBoot 测试类
 *  @RunWith:启动器 
 *  SpringJUnit4ClassRunner.class：让 junit 与 spring 环境进行整合
 *  @SpringBootTest(classes={App.class}) 1,当前类为 springBoot 的测试类 
 *  @SpringBootTest(classes={App.class}) 2,加载 SpringBoot 启动类。启动 springBoot
 *
 */
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes={App.class})
public class UserServiceImplTest {
	
	@Autowired
	private UserServiceImpl userServiceImpl;
	
	@Test
	public void saveUserTest(){
		this.userServiceImpl.saveUser();
	}
	
}
```

## SpringBoot 热部署 ##

在服务器不停止的情况下，更新项目代码

### SpringLoader 插件 ###

**方式一：以 Maven 插件的形式**

> SpringLoader 缺点：只能对 Java 代码有热部署能力，但是对页面无能为力
>
> 注意：SpringLoader 热部署程序是在系统后台以进程的形式启动
>
> 即关闭 SpringLoader 插件，需要手动关闭进程，否则再次启动会出现端口抢占

- 创建 Maven 项目，修改 pom.xml 文件
- 添加 Maven 的 SpringLoader 插件配置
- 使用 Maven 命令 `spring-boot:run`启动 SpringBoot 

```xml
<!-- springloader插件 -->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <dependencies>
                <dependency>
                    <groupId>org.springframework</groupId>
                    <artifactId>springloaded</artifactId>
                    <version>1.2.5.RELEASE</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

**方式二：在项目中使用 SpringLoader jar 包**

> 关闭 SpringLoader 更加人性化，直接在 IDE 中关闭即可

- 添加  SpringLoader 的 jar 包
- 启动方式

```
-javaagent:.\lib\springloaded-1.2.5.RELEASE.jar -noverify
```

### DevTools 工具 ###

> SpringLoader 与 DevTools 的区别：
>
> SpringLoader 在部署项目时使用的是热部署的方式
>
> DevTools： DevTools 在部署项目时使用的是重新部署的方式

- **修改 pom.xml 文件，添加 devtoos 坐标**

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <!-- 表示该坐标只作用本项目中，被继承时，不会向下传递 -->
      <optional>true</optional>
  </dependency>
  ```

- **正常使用 SpringBoot 启动类即可**

  当 Java 代码修改保存后，会自动进行重新部署。当页面修改保存后，也可以立即看到修过后的效果

### DevTools 在 IDEA 中的配置

1. 开启 IDEA 中自动编译功能

![](http://img.zwer.xyz/blog/20191130210716.png)

2. 设置 idea 中 Registry

通过 ctrl+Shirt+Alt + /， 打开

![](http://img.zwer.xyz/blog/20191130212343.png)




## SpringBoot 整合 Spring Data JPA  ##

### 搭建项目环境 ###

- 修改 pom.xml 文件

- 新建 applicationContext.properties

- 创建实体类

  ```java
  @Entity
  @Table(name = "t_user")
  public class Users {
  
  	@Id
  	@GeneratedValue(strategy = GenerationType.IDENTITY)
  	@Column(name = "id")
  	private Integer userid;
  	@Column(name = "username")
  	private String username;
  	@Column(name = "password")
  	private String password;
  	@Column(name = "userage")
  	private Integer userage;
  	@Column(name = "address")
  	private String address;
  
  	public Users(Integer userid, String password, Integer userage, String address) {
  		super();
  		this.userid = userid;
  		this.password = password;
  		this.userage = userage;
  		this.address = address;
  	}
  
  	public Users() {
  		super();
  		// TODO Auto-generated constructor stub
  	}
  
  	@Override
  	public String toString() {
  		return "Users [userid=" + userid + ", username=" + username + ", password=" + password + ", userage=" + userage
  				+ ", address=" + address + "]";
  	}
  
  	public Integer getUserid() {
  		return userid;
  	}
  
  	public void setUserid(Integer userid) {
  		this.userid = userid;
  	}
  
  	public String getPassword() {
  		return password;
  	}
  
  	public void setPassword(String password) {
  		this.password = password;
  	}
  
  	public Integer getUserage() {
  		return userage;
  	}
  
  	public void setUserage(Integer userage) {
  		this.userage = userage;
  	}
  
  	public String getAddress() {
  		return address;
  	}
  
  	public void setAddress(String address) {
  		this.address = address;
  	}
  
  	public String getUsername() {
  		return username;
  	}
  
  	public void setUsername(String username) {
  		this.username = username;
  	}
  
  }
  
  ```

- 创建接口

```java
package com.szxy.dao;

import org.springframework.data.jpa.repository.JpaRepository;

import com.szxy.pojo.Users;

/**
 *  JpaRepository<T, ID>
 * 参数一 T 表示当前映射的实体类
 * 参数二 ID 表示:当前映射的实体中的 OID 的类型 
 *
 */
public interface UsersDao extends JpaRepository<Users,Integer> {

}
```

- 测试类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes={App.class})
public class UsersDaoTest {

	@Autowired
	private UsersDao usersDao;
	@Autowired
	private UsersDaoByName usersDaoByName;
	@Autowired
	private UsersDaoByQuery usersDaoByQuery; 
	
	@Test
	public void saveUserTest(){
		Users u = new Users();
		u.setAddress("北京海淀");
		u.setUsername("小猪");
		u.setUserage(11);
		u.setPassword("12311");
		this.usersDao.save(u);
	}

}

```

### Spring JPA 接口的使用 ###

**Repository 接口的使用**

1. **方法名称命名查询**

> 命名规则：findBy(关键字)+查询的属性(首字母大写)+查询条件(And,Or,GreatterThan,Like)

```java
public interface UsersDaoByName extends Repository<Users, Integer>{

	List<Users> findByUsername(String username);
	
	Users findByUsernameAndPassword(String username,String password);
	
	List<Users> findByUsernameLike(String username);
}

```

2. **基于 @Query 注解与更新**

```java

/**
 * 基于 @Query 查询
 *
 */
public interface UsersDaoByQuery extends Repository<Users,Integer> {
	
	//参数绑定，从左到右
	@Query("from Users where username = ?")
	List<Users>  findAllUserUseHQL(String username);
	
	@Query(value="select username,password,userage from t_user where username = ?",nativeQuery=true)
	List<Users>  findAllUserUseSQL(String username);
	
	//更新
	@Query("update Users set username = ? where userid = ?")
	@Modifying 
	void updateUserUseHQL(String username,Integer userid);
	
	/*@Query("update Users set  userage = ? where username = ?")
	@Modifying
	void updateUserUseHQL(Integer userage,String username);*/
	
}

```

**CrudRepository 接口**

> 该接口提供了 CRUD 操作的抽象方法

```java

public interface UsersDaoByCrud extends CrudRepository<Users, Integer> {

}

```

**PagingAndSortingRepository 接口**

> 该接口提供了分页和排序的排序,同时继承了 CrudRepository 接口

```java
public interface UsersDaoByPagingAndSortingResp extends PagingAndSortingRepository<Users,Integer> {

}

```

**JpaRepository 接口**

> 该接口作用将返回值做适配处理

```java
/**
 *  JpaRepository<T, ID>
 * 参数一 T 表示当前映射的实体类
 * 参数二 ID 表示:当前映射的实体中的 OID 的类型 
 *
 */
public interface UsersDao extends JpaRepository<Users,Integer> {

}

```

**JpaSpecificationExecutor 接口**

> 该接口中的功能多条件查询和分页操作
>
> 该接口中单独存在，在使用时，需要配合之前的接口使用

```java
	@Test
	public void  test13(){
		Specification<Users> spec = new Specification<Users>() {
			
			/**
			 *    查询对象的属性的封装
			 *    封装了我们要执行的查询中的各个部分的信息，select from order
			 *    查询条件的构造器。定义不同的查询条件
			 */
			@Override
			public Predicate toPredicate(Root<Users> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
				Predicate pre = cb.equal(root.get("username"),"小猪");
				Predicate pre2 = cb.equal(root.get("userage"),11);
				List<Predicate> list = new ArrayList<>();
				list.add(pre);
				list.add(pre2);
				Predicate[] arr = new Predicate[list.size()];
				// 第一种写法
				//return cb.and(list.toArray(arr));
				// 第二种写法
 				//return cb.and(pre,pre2);
				 
				Predicate and = cb.equal(root.get("userage"),22);
				Predicate and2 = cb.and(cb.equal(root.get("username"), "小猪"),cb.equal(root.get("userage"), 11));
				//where users0_.userage=22 or users0_.username=? and users0_.userage=11
				return cb.or(and,and2);
			}
		};
		int page = 0;
		int size = 2;
		Sort sort = new Sort(new Order(Direction.DESC,"userid"));
		Pageable pageable = new PageRequest(page,size, sort);
		
		Page<Users> pageResult = this.usersDaoBySpecific.findAll(spec,pageable);
		List<Users> userList = pageResult.getContent();
		for (Users users : userList) {
			System.out.println(users);
		}
	}
```

### Spring JPA 一对多关联关系 ###

**Users 实体**

```java
@Entity
@Table(name = "t_user")
public class Users {
    
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "id")
	private Integer userid;
	@Column(name = "username")
	private String username;
	@Column(name = "password")
	private String password;
	@Column(name = "userage")
	private Integer userage;
	@Column(name = "address")
	private String address;

	@ManyToOne(cascade=CascadeType.PERSIST,fetch=FetchType.EAGER)
	@JoinColumn(name="role_id") // @JoinColumn 维护外键
	private Roles role;
	
	public Users(Integer userid, String password, Integer userage, String address) {
		super();
		this.userid = userid;
		this.password = password;
		this.userage = userage;
		this.address = address;
	}
	public Roles getRole() {
		return role;
	}

	public void setRole(Roles role) {
		this.role = role;
	}

	public Users() {
		super();
	}

	@Override
	public String toString() {
		return "Users [userid=" + userid + ", username=" + username + ", password=" + password + ", userage=" + userage
				+ ", address=" + address + ", role=" + role + "]";
	}

	public Integer getUserid() {
		return userid;
	}

	public void setUserid(Integer userid) {
		this.userid = userid;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public Integer getUserage() {
		return userage;
	}

	public void setUserage(Integer userage) {
		this.userage = userage;
	}

	public String getAddress() {
		return address;
	}

	public void setAddress(String address) {
		this.address = address;
	}

	public String getUsername() {
		return username;
	}

	public void setUsername(String username) {
		this.username = username;
	}	
		
}

```

**Roles 实体**

```java
@Entity
@Table(name="t_role")
public class Roles {

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="roleid")
	private Integer roleid;
	@Column(name="rolename")
	private String rolename;
	
	@OneToMany(mappedBy="role",cascade=CascadeType.PERSIST,fetch=FetchType.EAGER)
	private Set<Users> users = new HashSet<>();

	public Roles() {
		super();
	}

	public Roles(Integer roleid, String rolename, Set<Users> users) {
		super();
		this.roleid = roleid;
		this.rolename = rolename;
		this.users = users;
	}

	public Integer getRoleid() {
		return roleid;
	}

	public void setRoleid(Integer roleid) {
		this.roleid = roleid;
	}

	public String getRolename() {
		return rolename;
	}

	public void setRolename(String rolename) {
		this.rolename = rolename;
	}

	public Set<Users> getUsers() {
		return users;
	}

	public void setUsers(Set<Users> users) {
		this.users = users;
	}

	@Override
	public String toString() {
		return "Roles [roleid=" + roleid + ", rolename=" + rolename + "]";
	}

    
/****************************************************************************
	 java.lang.stackflow  异常
	解决：
    当根据用户查询信息，包括用户对应的 role 信息，应该去掉 Roles 实体中的 tostring 方法的 users
	 
****************************************************************************/
/*	@Override
	public String toString() {
		return "Roles [roleid=" + roleid + ", rolename=" + rolename + ", users=" + users + "]";
	}
*/
	
}

```

### SpringJPA 多对多关联关系 ###

**Menus 实体**

```java
@Entity
@Table(name="t_menu")
public class Menus {

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="menu_id")
	private Integer menuid;
	@Column(name="menuname")
	private String menuname;
	@Column(name="parent_id")
	private Integer parentid;
	
	//关联属性
	@ManyToMany(mappedBy="menus")
	private Set<Roles> roles = new HashSet<>();

	public Menus() {
		super();
	}

	public Menus(Integer menuid, String menuname, Set<Roles> roles) {
		super();
		this.menuid = menuid;
		this.menuname = menuname;
		this.roles = roles;
	}
	
	public Integer getParentid() {
		return parentid;
	}

	public void setParentid(Integer parentid) {
		this.parentid = parentid;
	}

	public Integer getMenuid() {
		return menuid;
	}

	public void setMenuid(Integer menuid) {
		this.menuid = menuid;
	}

	public String getMenuname() {
		return menuname;
	}

	public void setMenuname(String menuname) {
		this.menuname = menuname;
	}

	public Set<Roles> getRoles() {
		return roles;
	}

	public void setRoles(Set<Roles> roles) {
		this.roles = roles;
	}

	@Override
	public String toString() {
		return "Menus [menuid=" + menuid + ", menuname=" + menuname + "]";
	}
	
}

```

**Roles 实体**

```java
@Entity
@Table(name="t_role")
public class Roles {

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="roleid")
	private Integer roleid;
	@Column(name="rolename")
	private String rolename;
	
	//关联属性
	// mapperBy的属性值是 选择 Users 实体中角色属性 role 
	@OneToMany(mappedBy="role",cascade=CascadeType.PERSIST,fetch=FetchType.EAGER)
	private Set<Users> users = new HashSet<>();
	
	@ManyToMany(cascade=CascadeType.PERSIST,fetch=FetchType.EAGER)
	@JoinTable(joinColumns=@JoinColumn(name="role_id"),inverseJoinColumns=@JoinColumn(name="menu_id"))
	private Set<Menus> menus = new HashSet<>();
	
	public Roles() {
		super();
	}

	public Roles(Integer roleid, String rolename, Set<Users> users) {
		super();
		this.roleid = roleid;
		this.rolename = rolename;
		this.users = users;
	}

	public Set<Menus> getMenus() {
		return menus;
	}

	public void setMenus(Set<Menus> menus) {
		this.menus = menus;
	}

	public Integer getRoleid() {
		return roleid;
	}

	public void setRoleid(Integer roleid) {
		this.roleid = roleid;
	}

	public String getRolename() {
		return rolename;
	}

	public void setRolename(String rolename) {
		this.rolename = rolename;
	}

	public Set<Users> getUsers() {
		return users;
	}

	public void setUsers(Set<Users> users) {
		this.users = users;
	}

	@Override
	public String toString() {
		return "Roles [roleid=" + roleid + ", rolename=" + rolename + ", users=" + users + ", menus=" + menus + "]";
	}


/*	@Override
	public String toString() {
		return "Roles [roleid=" + roleid + ", rolename=" + rolename + ", users=" + users + "]";
	}
*/

}

```

**多对多关联关系测试**

> 这里通过 Roles 实体操作 Menus 实体，所以在 Menus 实体类中的 toString 方法不要打印 Roles 对象

```java
	/**************************************
	 *	    	多对多关系映射                              *
	 **************************************/
	@Test
	public void test16(){
		//创建 Menus 对象
		Menus m = new Menus();
		m.setMenuname("xxxx 平台管理系统");
		m.setParentid(-1);
		Menus m2 = new Menus();
		m2.setMenuname("用户管理");
		m2.setParentid(1);
		Menus m3 = new Menus();
		m3.setMenuname("栏目管理");
		m3.setParentid(2);
		//创建角色对象
		Roles r = new Roles();
		r.setRolename("超级管理员");
		Roles r2= new Roles();
		r2.setRolename("普通管理员");
		//建立映射关系
		r2.getMenus().add(m);
		r2.getMenus().add(m3);
		r.getMenus().add(m);
		r.getMenus().add(m2);
		r.getMenus().add(m3);
		//保存数据
		List<Roles> list  = new ArrayList<>();
		list.add(r);
		list.add(r2);
		this.rolesDao.save(list);
		
	}	
	
	@Test
	public void test17(){
		Roles role = this.rolesDao.findOne(4);
		Set<Menus> menus = role.getMenus();
		System.out.println(role);
		for (Menus m : menus) {
			System.out.println(m);
		}
	}
```

## Spring Boot 整合 Ehcache 步骤 ##

- 创建 Maven jar 项目，修改 pom.xml 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.10.RELEASE</version>
	</parent>
	<groupId>com.szxy.springboot</groupId>
	<artifactId>16-SpringBoot-echcache</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<java.version>1.7</java.version>
	</properties>

	<dependencies>
		<!-- Springboot 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- thymeleaf 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<!-- SpringData JPA 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<!-- Junit 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
		</dependency>
		<!-- mysql 数据库驱动 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<!-- druid 数据库连接池 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.0.9</version>
		</dependency>
		<!-- Spring Boot 缓存支持 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
		<!-- Ehcache 坐标 -->
		<dependency>
			<groupId>net.sf.ehcache</groupId>
			<artifactId>ehcache</artifactId>
		</dependency>
	</dependencies>
</project>
```

- 添加 application.properties 文件

```properties
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/ssm?useSSL=false
spring.datasource.username=root
spring.datasource.password=root

spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

spring.cache.echcache.config=ehcache.xml
```

- 添加 ehcache.xml  文件

```xml
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">

    <diskStore path="java.io.tmpdir"/>
	
	<!-- defaultCache 默认缓存策略  -->
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxElementsOnDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </defaultCache>
    
    <!--自定义缓存策略 -->
    <cache name="users"
    	   maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxElementsOnDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </cache>
    
</ehcache>

```

- 创建 Users 实体类

```java
@Entity
@Table(name = "t_user")
public class Users implements Serializable{

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "id")
	private Integer userid;
	@Column(name = "username")
	private String username;
	@Column(name = "password")
	private String password;
	@Column(name = "userage")
	private Integer userage;
	@Column(name = "address")
	private String address;

	public Integer getUserid() {
		return userid;
	}

	public void setUserid(Integer userid) {
		this.userid = userid;
	}

	public String getUsername() {
		return username;
	}

	public void setUsername(String username) {
		this.username = username;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public Integer getUserage() {
		return userage;
	}

	public void setUserage(Integer userage) {
		this.userage = userage;
	}

	public String getAddress() {
		return address;
	}

	public void setAddress(String address) {
		this.address = address;
	}

	@Override
	public String toString() {
		return "Users [userid=" + userid + ", username=" + username + ", password=" + password + ", userage=" + userage
				+ ", address=" + address + "]";
	}

}

```

- 创建 UsersDao 接口

```java
package com.szxy.dao;

import org.springframework.data.jpa.repository.JpaRepository;

import com.szxy.pojo.Users;

/**
 *  JpaRepository<T, ID>
 * 参数一 T 表示当前映射的实体类
 * 参数二 ID 表示:当前映射的实体中的 OID 的类型 
 *
 */
public interface UsersDao extends JpaRepository<Users,Integer> {

}

```

- 创建 UsersService 接口

```java
public interface UsersService {
	
	public List<Users> findAllUser();
	
	public Users findUserById(Integer userid);
	
	public Page<Users> findUsersByPage(Pageable pageable);
	
	
}

```

- 创建 UsersService 接口实现类

```java
@Service
public class UsersServiceImpl implements UsersService {
	
	@Autowired
	private UsersDao usersDao;

	@Override
	public List<Users> findAllUser() {
		return this.usersDao.findAll();
	}
	
    /**************************************/
    /* @Cacheable 表示对当前查询对象做缓存处理 */
    /**************************************/
	@Override
	@Cacheable(cacheNames={"users"}) 
	public Users findUserById(Integer userid) {
		return this.usersDao.findOne(userid);
	}

	@Override
	public Page<Users> findUsersByPage(Pageable pageable) {
		return this.usersDao.findAll(pageable);
	}
	
}

```

- 创建 SpringBoot 启动类

```java
/**************************************/
/* @EnableCaching 表示 //允许缓冲*/
/**************************************/
@SpringBootApplication
@EnableCaching 
public class App {
	
	public static void main(String[] args) {
		SpringApplication.run(App.class,args);
	}
	
}

```

- 创建 UserServiceTest 测试类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes={App.class})
public class UsersDaoTest {
	
	@Autowired
	private UsersService usersService;
	
	@Test
	public void  Test(){
		// 第一次查询
		Users users = this.usersService.findUserById(8);
		System.out.println(users);
		// 第二次查询
		Users user2 =  this.usersService.findUserById(8);
		System.out.println(users);
	}
}
```

### @Cacheable 的作用 ###

> 将方法的返回值加入 Ehcache 中做缓存
>
> value 属性：指定一个 Ehcache 配置文件中的缓存策略，如果没有给定 value，则采用默认的缓存策略
>
> key 属性：给存储的值起个名称，若查询时如果有名称，那么则直接从缓存中将数据返回

- 业务层代码

```java
/****************************************************************************/
/*         value 属性表示自定义缓存策略                                         */
/* 		   key 属性表示缓存数据的名称，这里默认是 #pageable 作为默认的名称          */ 
/* 	       也可以选择 Pageable 类的属性作为缓存数据的名称                         */         /****************************************************************************/
@Override
@Cacheable(value={"users"},key="#pageable.pageSize")
public Page<Users> findUsersByPage(Pageable pageable) {
    return this.usersDao.findAll(pageable);
}
```

- 测试代码

```java
@Test
	public void  Test2(){

		/********* 第一次查询 *********/
		Pageable pageable = new PageRequest(0,3);
		Page<Users> pageResult = this.usersService.findUsersByPage(pageable);
		System.out.println(pageResult.getTotalElements());
		List<Users> userList = pageResult.getContent();
		for (Users u : userList) {
			System.out.println(u);
		}
		/********* 第二次查询 *********/
		pageable = new PageRequest(1, 3);
		Page<Users> pageResult2 = this.usersService.findUsersByPage(pageable);
		System.out.println(pageResult2.getTotalElements());
		List<Users> userList2 = pageResult.getContent();
		for (Users u : userList2) {
			System.out.println(u);
		}
	}
```

### @CacheEvict 的作用 ###

> 清除缓存，解决缓冲同步问题
>
> 注意：清除缓存的选择的缓存策略与添加存储的缓存策略相同

- 业务层代码

```java
@Override
	@Cacheable(value="users")
	public List<Users> findAllUser() {
		return this.usersDao.findAll();
	}
	
	@Override
	@CacheEvict(value="users",allEntries=true)  //清除缓存中以 users 缓存策略的对象
	public void addUser(Users user) {
		this.usersDao.save(user);
	}
```

- 测试代码

```java
@Test
public void  Test3(){
    //第一次查询
    List<Users> users = this.usersService.findAllUser();
    System.out.println(users.size());

    Users user = new Users();
    user.setUsername("橙留香");
    user.setPassword("123");
    user.setUserage(12);
    this.usersService.addUser(user);

    //第二次查询
    List<Users> users2 = this.usersService.findAllUser();
    System.out.println(users2.size());

}
```

## Spring Boot 整合 Spring Data Redis ##

> Spring Data 的

### 创建 Maven 项目，修改 pom.xml  ###

```XML
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-parent</artifactId>
  	<version>1.5.10.RELEASE</version>
  </parent>
  <groupId>com.szxy.springboot</groupId>
  <artifactId>17-SpringBoot-Spring-Data-Redis</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <properties>
  	<java.version>1.7</java.version>
	<thymeleaf.version>3.0.2.RELEASE</thymeleaf.version>
	<thymeleaf-layout-dialect.version>2.0.4</thymeleaf-layout-dialect.version>
  </properties>
  <dependencies>
  <!-- Spring Boot 的启动器 -->
  	<dependency>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-web</artifactId>
  	</dependency>
  	<!-- thymeleaf 的启动器 -->
  	<dependency>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-thymeleaf</artifactId>
  	</dependency>
  	<!-- Spring Data Redis 的启动器 -->
  	<dependency>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-data-redis</artifactId>
  	</dependency>
  	<!-- test 启动器 -->
  	<dependency>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-test</artifactId>
  	</dependency>
  </dependencies>
</project>
```

- RedisConfig 配置类

```java
/**
 * Redis 配置
 *
 */
@Configuration
public class RedisConfig {
	
	/**
	 * 1.创建  JedisPoolConfig
	 */
	@Bean
	public JedisPoolConfig JedisPoolConfig(){
		JedisPoolConfig config = new JedisPoolConfig();
		//最大空闲数
		config.setMaxIdle(10);
		 //最小空闲数
		config.setMinIdle(5);
		// 最大连接数
		config.setMaxTotal(30); 
		return config;
	}
	
	/****
	 * 2. 创建 JedisConnection
	 */
	@Bean
	public JedisConnectionFactory jedisConnectionFactory(JedisPoolConfig config){
		JedisConnectionFactory factory = new JedisConnectionFactory();
		
		//关联连接池的配置的对象
		factory.setPoolConfig(config);
		//配置连接 Redis 的信息
		// 主机地址
		factory.setHostName("192.168.170.30");
		// 端口 
		factory.setPort(6379);
		
		return factory;
	}
	
	/***
	 * 3.创建 RedisTemplate ：用于执行 Redis 操作的方法
	 */
	@Bean
	public RedisTemplate<String,Object> redisTemplate(JedisConnectionFactory factory){
		RedisTemplate<String, Object> template = new RedisTemplate<>();
		
		//关联 Jedis 连接工厂
		template.setConnectionFactory(factory);
		//为 key 设置序列化器
		template.setKeySerializer(new StringRedisSerializer());
		//为 value 设置序列化器
		template.setValueSerializer(new StringRedisSerializer());
		
		return template;
	}
}

```

### 提取 Redis 配置信息 ###

**application.properties**

```properties
spring.redis.pool.max-idle=10
spring.redis.pool.min-idle=5
spring.redis.pool.max-total=30

spring.redis.hostName=192.168.170.30
spring.redis.port=6379

```

**RedisConfig.java**

```java
/**
 * Redis 配置
 *
 */
@Configuration
public class RedisConfig {
	
	/**
	 * 1.创建  JedisPoolConfig
	 * 
	 * @ConfigurationProperties 读取 application.properties 文件
	 *  会将前缀相同的内容创建一个实体
	 * 
	 */
	@Bean
	@ConfigurationProperties(prefix="spring.redis.pool")
	public JedisPoolConfig JedisPoolConfig(){
		JedisPoolConfig config = new JedisPoolConfig();
		return config;
	}
	
	/****
	 * 2. 创建 JedisConnection
	 */
	@Bean
	@ConfigurationProperties(prefix="spring.redis")
	public JedisConnectionFactory jedisConnectionFactory(JedisPoolConfig config){
		JedisConnectionFactory factory = new JedisConnectionFactory();
		factory.setPoolConfig(config);
		return factory;
	}
	
	/***
	 * 3.创建 RedisTemplate ：用于执行 Redis 操作的方法
	 */
	@Bean
	public RedisTemplate<String,Object> redisTemplate(JedisConnectionFactory factory){
		RedisTemplate<String, Object> template = new RedisTemplate<>();
		
		//关联 Jedis 连接工厂
		template.setConnectionFactory(factory);
		//为 key 设置序列化器
		template.setKeySerializer(new StringRedisSerializer());
		//为 value 设置序列化器
		template.setValueSerializer(new StringRedisSerializer());
		
		return template;
	}
}
```

### 存储 对象 ###

> JDK 序列器的作用对象转为 json 格式字符串再转为字节数据，即在 Redis 客户端查看是乱码情况。
>
> 或者将字节数据转为 json 格式字符串，再转为 对象返回。
>
> 在存储对象前后，都要使用 JDK 的序列器

```java
@Test
public void test3(){
    Users user = new Users();
    user.setUserage(21);
    user.setUsername("好好");
    //设置序列化器
    this.redisTemplate.setValueSerializer(new JdkSerializationRedisSerializer());
    this.redisTemplate.opsForValue().set("u", user);

}

@Test
public void test4(){
    //设置序列化器
    this.redisTemplate.setValueSerializer(new JdkSerializationRedisSerializer());
    Users user = (Users) this.redisTemplate.opsForValue().get("users");
    System.out.println(user);
}	
```

### 以 Json 格式存储实体对象 ###

> 相比于 JDK 序列化器，存储对象占用空间小,并且在 Redis 客户端可以直接查看 json 格式字符串中内容

```java
/***
	 * 以 json 格式保存 对象实体
	 */
@Test
public void test5(){
    Users user = new Users();
    user.setUserage(21);
    user.setUsername("Json 真好玩");
    //设置序列化器
    this.redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(Users.class));
    this.redisTemplate.opsForValue().set("uu", user);

}
@Test
public void test6(){
    //设置序列化器
    this.redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(Users.class));
    Users user = (Users) this.redisTemplate.opsForValue().get("uu");
    System.out.println(user);
}
```

## Spring Boot 定时任务 ##

### scheduled  ###

- 创建 Maven 的 jar 项目，修改 pom.xml 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.10.RELEASE</version>
	</parent>
	<groupId>com.szxy.springboot</groupId>
	<artifactId>18-SpringBoot-scheduled</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.7</java.version>
		<thymeleaf.version>3.0.2.RELEASE</thymeleaf.version>
		<thymeleaf-layout-dialect.version>2.0.4</thymeleaf-layout-dialect.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<!-- 添加 Scheduled 坐标 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
		</dependency>
	</dependencies>
</project>
```

- 定时类

```java
@Component
public class ScheduledDemo {

	@Scheduled(cron="0/3 * * * * ?")  //指定定时任务
	// @cron 定时任务触发时间时一个字符串表达形式
	public void work(){
		Date date = new Date();
		DateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		System.out.println("定时任务触发:"+sdf.format(date));
	}
	
}

```

- SpringBoot 启动器

```java
package com.szxy;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling  //允许定时任务
public class App {

	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
	
}
```

###  @cron  表达式 ###

> Cron 表达式是一个字符串，分为 **6** 或 7 域，每个代表一个含义
>
> Cron 有如下两种语法格式：
>
> 1. Seconds Minutes Hours Day Mo   nth Week Year
> 2. Seconds MInutes Hours Day Month Week

**结构**

corn 从左到右（用空格隔开）: 秒 分 小时 月份中的日期 月份 星期中的日期 年份

![](http://img.zwer.xyz/blog/20190804191836.png)



●星号(`*`)：可用在所有字段中，表示对应时间域的每一个时刻，例如，*在分钟字段时，表示“每分钟”； *

●问号（?）：该字符只在日期和星期字段中使用，它通常指定为“无意义的值”，相当于占位符； *

●减号(-)：表达一个范围，如在小时字段中使用“10-12”，则表示从 10 到 12 点，即 10,11,12； *

*●逗号(,)：表达一个列表值，如在星期字段中使用“MON,WED,FRI”，则表示星期一，星期三和星期 五；*

*●斜杠(/)：x/y 表达一个等步长序列，x 为起始值，y 为增量步长值。如在分钟字段中使用 0/15，则 表示为 0,15,30 和 45 秒，而 5/15 在分钟字段中表示 5,20,35,50，你也可以使用*/y，它等同于 0/y；

 ●L：该字符只在日期和星期字段中使用，代表“Last”的意思，但它在两个字段中意思不同。L 在日期 字段中，表示这个月份的最后一天，如一月的 31 号，非闰年二月的 28 号；如果 L 用在星期中，则表示星 期六，等同于 7。但是，如果 L 出现在星期字段里，而且在前面有一个数值 X，则表示“这个月的最后 X 天”， 例如，6L 表示该月的最后星期五；

 ●W：该字符只能出现在日期字段里，是对前导日期的修饰，表示离该日期最近的工作日。例如 15W 表示离该月 15 号最近的工作日，如果该月 15 号是星期六，则匹配 14 号星期五；如果 15 日是星期日， 则匹配 16 号星期一；如果 15 号是星期二，那结果就是 15 号星期二。但必须注意关联的匹配日期不能够 跨月，如你指定 1W，如果 1 号是星期六，结果匹配的是 3 号星期一，而非上个月最后的那天。W 字符串 只能指定单一日期，而不能指定日期范围；

 ●LW 组合：在日期字段可以组合使用 LW，它的意思是当月的最后一个工作日；

 ●井号(#)：该字符只能在星期字段中使用，表示当月某个工作日。如 6#3 表示当月的第三个星期五(6 表示星期五，#3 表示当前的第三个)，而 4#5 表示当月的第五个星期三，假设当月没有第五个星期三， 忽略不触发；

 ● C：该字符只在日期和星期字段中使用，代表“Calendar”的意思。它的意思是计划所关联的日期， 如果日期没有被关联，则相当于日历中所有日期。例如 5C 在日期字段中就相当于日历 5 日以后的第一天。 1C 在星期字段中相当于星期日后的第一天。 Cron 表达式对特殊字符的大小写不敏感，对代表星期的缩写英文大小写也不敏感。

 例子:

```java
@Scheduled(cron = "0 0 1 1 1 ?")//每年一月的一号的 1:00:00 执行一次 

@Scheduled(cron = "0 0 1 1 1,6 ?") //一月和六月的一号的 1:00:00 执行一次 

@Scheduled(cron = "0 0 1 1 1,4,7,10 ?") //每个季度的第一个月的一号的 1:00:00 执行一次 

@Scheduled(cron = "0 0 1 1 * ?")//每月一号 1:00:00 执行一次 

@Scheduled(cron="0 0 1 * * *") //每天凌晨 1 点执行一次
```

###  Quartz  ###

> **Quartz**是一个[Java](https://zh.wikipedia.org/wiki/Java)下作业控制的[开源](https://zh.wikipedia.org/wiki/开源)[框架](https://zh.wikipedia.org/wiki/軟體框架)。Quartz用来创建或简单或复杂的调度时间表，执行Java下任意数量的作业。版本1.0发布于2002年9月13日，当前版本2.2.1发布于2013年9月24日。
>
> 可以通过`CronTrigger`定义Quartz的调度时间表（例如`0 0 12 ? * WED`表示“每周三上午12：00”）。
>
> 此外，时间表也可以通过`SimpleTrigger`，由`Date`定义触发的开始时间、毫秒的时间间隔和重复计数（例如“在下周三12：00，然后每隔10秒、执行5次”）。可以使用`Calender`定义例外的日程（例如“没有周末和节假日”）。
>
> 作业可以是实现了Job接口任意的Java类。作业监听器（JobListener）和触发器监听器（TriggerListener）通知作业的执行（和其他事件）。作业及其触发器可以被[持久化](https://zh.wikipedia.org/w/index.php?title=持久化&action=edit&redlink=1)。
>
> Quartz一般用于[企业级应用程序](https://zh.wikipedia.org/wiki/企业级软件)，以支持工作流、[系统管理](https://zh.wikipedia.org/w/index.php?title=系统管理&action=edit&redlink=1)（维护）活动，并在应用程序中提供实时的服务。Quartz还支持集群。
>
> Quartz是[Terracotta公司](https://zh.wikipedia.org/w/index.php?title=Terracotta公司&action=edit&redlink=1)的[开源](https://zh.wikipedia.org/wiki/开源)产品。 [.NET](https://zh.wikipedia.org/wiki/.NET框架)平台下的对应产品叫Quartz.NET。

job -任务-你要做什么事？

Trigger-触发器-你什么时候去做？

Schedulr-任务调度-你需要什么时候去做什么事？

Quartz  单独项目 Demo 

- 创建 Maven 项目，修改 pom.xml 文件

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.szxy</groupId>
	<artifactId>19-quartz</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<dependencies>
			<!-- quartz 坐标 -->
			<dependency>
				<groupId>org.quartz-scheduler</groupId>
				<artifactId>quartz</artifactId>
				<version>2.2.1</version>
			</dependency>
	</dependencies>
</project>
```

- QuartzDemo.java: 

```java
public class QuartzDemo implements Job{

	public void execute(JobExecutionContext arg0) throws JobExecutionException {
		System.out.println("Execute ... 我被执行了？？？"+new Date());
	}

}

```

- QuartzMain.java :测试类

```java
public class QuartMain {
	public static void main(String[] args) throws SchedulerException {
		
		/** 1. 创建 Job 对象 **/
		JobDetail job = JobBuilder.newJob(QuartzDemo.class).build();
		
		/*** 
		 * 2. 简单的 trigger 触发时间：通过 Quartz 提供一个方法来完成简单的重复 调用 
		 * cron * Trigger：按照 Cron 的表达式来给定触发的时间
		 */
		/*SimpleTrigger trigger = 
				TriggerBuilder.newTrigger().
				withSchedule(SimpleScheduleBuilder.repeatSecondlyForever(2)).
				build();*/
		CronTrigger trigger = 
				TriggerBuilder.newTrigger().
				withSchedule(CronScheduleBuilder.cronSchedule("0/3 * * * * ?")).
				build();
		
		/** 3. 创建 Schedule **/
		Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();		
		scheduler.scheduleJob(job, trigger);
		
		//启动
		scheduler.start();
	}
}
```

### SpringBoot 整合 Quartz ###

- 创建 Maven 项目，修改 pom.xml 文件:这里要添加额外坐标

```java
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.10.RELEASE</version>
	</parent>
	<groupId>com.szxy.springboot</groupId>
	<artifactId>09-SpringBoot-view-thymeleaf</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.7</java.version>
		<thymeleaf.version>3.0.2.RELEASE</thymeleaf.version>
		<thymeleaf-layout-dialect.version>2.0.4</thymeleaf-layout-dialect.version>
	</properties>
	<dependencies>
		<!-- SpringBoot 启动类 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- thymeleaf 启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<!-- Quartz 坐标 -->
		<dependency>
			<groupId>org.quartz-scheduler</groupId>
			<artifactId>quartz</artifactId>
			<version>2.2.1</version>
			<!-- 排除 slf4j 依赖 -->
			<exclusions>
				<exclusion>
					<groupId>org.slf4j</groupId>
					<artifactId>slf4j-api</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<!-- 添加 Scheduled 坐标 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
		</dependency>
		 <!-- Sprng tx 坐标 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
		</dependency>
	</dependencies>
</project>
```

- 创建 QuartzDemo.java :执行定时任务类

```java
@Component
public class QuartzDemo implements Job{

	@Autowired
	private UsersService usersService;
	
	@Override
	public void execute(JobExecutionContext arg0) throws JobExecutionException {
		System.out.println("Executing "+new Date());
		usersService.addUser();
	}

}

```

- 创建 QuartzConfig.java: 配置定时任务相关配置

```java
@Configuration
public class QuartzConfig {

	/****
	 * 1. 创建  job
	 * @return
	 */
	@Bean
	public JobDetailFactoryBean getJobDetailFactoryBean(){
		JobDetailFactoryBean factory = new JobDetailFactoryBean();
		// 关联 任务类 Job 
		factory.setJobClass(QuartzDemo.class);
		return factory;
	}
	
	/**
	 * 2. 创建 Trigger 
	 * @param jobDetailFactoryBean
	 * @return
	 */
	/*@Bean
	public SimpleTriggerFactoryBean  
		getSimpleTriggerFactoryBean(JobDetailFactoryBean jobDetailFactoryBean){
		SimpleTriggerFactoryBean factory = new SimpleTriggerFactoryBean();
		//关联 JobDetail 
		factory.setJobDetail(jobDetailFactoryBean.getObject());
		// 设置执行时间  该参数表示一个执行的毫秒数
		factory.setRepeatInterval(2000);
		// 设置执行的次数
		factory.setRepeatCount(5);
		return factory;
	}*/
	
	/**
	 * 2. 创建 Trigger
	 * @param jobDetailFactoryBean
	 * @return
	 */
	@Bean
	public CronTriggerFactoryBean  
        getCronTriggerFactoryBean(JobDetailFactoryBean jobDetailFactoryBean){
        
		CronTriggerFactoryBean factory = new CronTriggerFactoryBean();
		//关联 JobDetail 
		factory.setJobDetail(jobDetailFactoryBean.getObject());
		//设置执行时间
		factory.setCronExpression("0/3 * * * * ?");
		return factory;
	}
	
	
	/**
	 * 3. 创建 scheduler
	 */
	@Bean
	public SchedulerFactoryBean
       	getSchedulerFactoryBean(CronTriggerFactoryBean cronTriggerFactoryBean,
                          MyAdaptableJobFactory myAdaptableJobFactory){
		SchedulerFactoryBean factory = new SchedulerFactoryBean();
		// 关联  trigger
		factory.setTriggers(cronTriggerFactoryBean.getObject());
		// 设置 Job 工厂
		factory.setJobFactory(myAdaptableJobFactory);
		return factory;
	}
	
}

```

- 创建 UsersService.java: 用于定时任务类调用业务层方法

```java
@Service
public class UsersService {
	
	public void  addUser(){
		System.out.println("addUser ...");
	}
	
}

```

- 创建 MyMyAdaptableJobFactory.java: 创建定时任务类对象和将该对象放入 Spring IOC 容器中

```java
/***
 * 解决  QuartzDemo 类注入异常 的问题：
 * 原因： QuartzDemo 对象没有加入到 Spring IOC 容器中，导致 QuartzConfig 的依赖注入发生异常
 * 解决方法； 手动将 QuartzDemo 对象加入到 Spring IOC 容器中
 * @author zwer
 *
 */
@Component("myAdaptableJobFactory")
public class MyAdaptableJobFactory extends AdaptableJobFactory {

	// AutowireCapableBeanFactory 可以将一个对象添加到 SpringIOC 容器中， 
	// 并且完成该对象注入 
	@Autowired
	 private AutowireCapableBeanFactory autowireCapableBeanFactory;

	@Override
	protected Object createJobInstance(TriggerFiredBundle bundle) throws Exception {
		Object obj = super.createJobInstance(bundle);
		//将 obj 对象添加 Spring IOC 容器中，并完成注入
		autowireCapableBeanFactory.autowireBean(obj);
		return obj;
	}

}
```

- SpringBoot 启动类：记得要开启 `@EnableScheduling `  注解

```java
@SpringBootApplication
@EnableScheduling   
public class App {

	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
	
}
```

## 总结 ##

1. 啥是 SpringBoot ，SpringBoot 有啥作用？

   SpringBoot 基于 Spring  开发的高级框架，但是 SpringBoot 配置相比 Spring 框架配置显得非常简洁，许多配置都是默认的，即**约定优于配置**。

   通过 SpringBoot 创建 Web 项目，只需使用 Maven 项目继承 SpringBoot  父项目，再选择依赖注入 spring-boot-web 启动器，最后创建 SpringBoot 启动类，即可 Just Run。

   SpringBoot 可以整合 Servlet 相关、Mybatis 、SpringData JPA以及前端模板freemarker和 thymeleaf

2. SpringBoot 的工作原理，或者执行流程

   参考 [SpringBoot 工作原理](https://blog.csdn.net/Chen_Victor/article/details/81262233)

![](https://img-blog.csdn.net/20180728183555425?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoZW5fVmljdG9y/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)





