# SpringData #

## 技术介绍 ##

### Hibernate  ###

> Hibernate是一个开放源代码的**对象关系映射框架，**它对JDBC进行了非常*轻量级*的对象封装，它*将POJO与数据库表建立映射关系，是一个全自动的 orm框架，hibernate可以自动生成SQL语句，自动执行*，使得Java程序员可以随心所欲的使用对象编程思维来操纵数据库。 Hibernate可以应用在任何使用JDBC的场合，既可以在Java的客户端程序使用，也可以在Servlet/JSP的Web应用中使用，最具革命意义的是，Hibernate可以在应用EJB的JaveEE架构中取代CMP，完成[数据持久化](https://baike.baidu.com/item/数据持久化/5777076)的重任。

### JPA 标准(规范)  ###

> JPA是Java Persistence API 的简称，中文名Java持久层API，是 JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体[对象持久化](https://baike.baidu.com/item/对象持久化/7316192)到数据库中。 [1] 
>
> Sun引入新的JPA ORM规范出于两个原因：
>
> 其一，简化现有Java EE和Java SE应用开发工作；其二，Sun希望整合ORM技术，实现天下归一。

### Hibernate JPA  ###

Hbernate 在 3.2 以后更具 JPA 规范，提供了一套操作持久层的 API

### Spring Data ###

> Spring Data’s mission is to provide a familiar and consistent, Spring-based programming model for data access while still retaining the special traits of the underlying data store.
>
> It makes it easy to use data access technologies, relational and non-relational databases, map-reduce frameworks, and cloud-based data services. This is an umbrella project which contains many subprojects that are specific to a given database. The projects are developed by working together with many of the companies and developers that are behind these exciting technologies.

### Spring Data JPA  ###

> Spring Data JPA, part of the larger Spring Data family, makes it easy to easily implement JPA based repositories. This module deals with enhanced support for JPA based data access layers. It makes it easier to build Spring-powered applications that use data access technologies.
>
> Implementing a data access layer of an application has been cumbersome for quite a while. Too much boilerplate code has to be written to execute simple queries as well as perform pagination, and auditing. Spring Data JPA aims to significantly improve the implementation of data access layers by reducing the effort to the amount that’s actually needed. As a developer you write your repository interfaces, including custom finder methods, and Spring will provide the implementation automatically.

Spring Data JPA 底层默认使用的是  Hibernate 的 API 

### Spring Data Redis ###

> Spring Data Redis, part of the larger Spring Data family, provides easy configuration and access to Redis from Spring applications. It offers both low-level and high-level abstractions for interacting with the store, freeing the user from infrastructural concerns.

##  Spring 整合 Hibernate  ##

新建 Java Project 项目，添加所依赖的 jar 包

antlr-2.7.7.jar
aopalliance.jar
aspectjrt.jar
aspectjweaver.jar
c3p0-0.9.2.1.jar
commons-logging-1.1.1.jar
dom4j-1.6.1.jar
geronimo-jta_1.1_spec-1.1.1.jar
hibernate-c3p0-5.0.7.Final.jar
hibernate-commons-annotations-5.0.1.Final.jar
hibernate-core-5.0.7.Final.jar
hibernate-jpa-2.1-api-1.0.0.Final.jar
jandex-2.0.0.Final.jar
javassist-3.18.1-GA.jar
jboss-logging-3.3.0.Final.jar
mchange-commons-java-0.2.3.4.jar
mysql-connector-java-5.1.7-bin.jar
spring-aop-4.2.0.RELEASE.jar
spring-aspects-4.2.0.RELEASE.jar
spring-beans-4.2.0.RELEASE.jar
spring-context-4.2.0.RELEASE.jar
spring-core-4.2.0.RELEASE.jar
spring-expression-4.2.0.RELEASE.jar
spring-jdbc-4.2.0.RELEASE.jar
spring-orm-4.2.0.RELEASE.jar
spring-test-4.2.0.RELEASE.jar
spring-tx-4.2.0.RELEASE.jar

- applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/aop
	http://www.springframework.org/schema/aop/spring-aop.xsd
	http://www.springframework.org/schema/tx 
	http://www.springframework.org/schema/tx/spring-tx.xsd">

	<!-- 加载数据库连接配置文件 -->
	<context:property-placeholder location="classpath:db.properties" />

	<!-- 加载 C3p0 数据库连接池 -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="driverClass" value="${jdbc.driver}" />
		<property name="jdbcUrl" value="${jdbc.url}" />
		<property name="user" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
	</bean>

	<!-- 配置 Hibernate 中 SqlSessionFactory -->
	<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="hibernateProperties">
			<props>
				<!-- 显示当前执行的 sql 语句 -->
				<prop key="hibernate.show_sql">true</prop>
				<!-- 开启正向工程,根据 pojo 类，创建数据表 -->
				<prop key="hibernate.hbm2ddl.auto">update</prop>
			</props>
		</property>
		<!-- 扫描实体类所在的包 -->
		<property name="packagesToScan">
			<list>
				<value>com.szxy.pojo</value>
			</list>
		</property>
	</bean>

	<!-- Hibernate 的 HIbernateTemplate  -->
	<bean id="hibernateTemplate" class="org.springframework.orm.hibernate5.HibernateTemplate">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>

	<!-- 配置 HIberante 的事务管理器 -->
	<bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>

	<!-- 配置注解事务处理 -->
	<tx:annotation-driven transaction-manager="transactionManager"/>


	<!--配置 springIOC 的注解扫描 -->
	<context:component-scan base-package="com.szxy" />

</beans>
```

- UserDao.java:

```java
package com.szxy.dao;

import com.szxy.pojo.Users;

public interface UsersDao {

	public void insertUser(Users user);
	
	public void updateUser(Users user);
	
	public void deleteUser(Users user);
	
	public Users selectUserByUserid(Long id);
	
}
```

- UserDaoImpl.java:

```java
package com.szxy.dao.impl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.orm.hibernate5.HibernateTemplate;
import org.springframework.stereotype.Repository;

import com.szxy.dao.UsersDao;
import com.szxy.pojo.Users;

@Repository
public class UsersDaoImpl implements UsersDao {

	@Autowired
	private HibernateTemplate hibernateTemplate;
	
	@Override
	public void insertUser(Users user) {
		this.hibernateTemplate.save(user);
	}

	@Override
	public void updateUser(Users user) {
		this.hibernateTemplate.update(user);
	}

	@Override
	public void deleteUser(Users user) {
		this.hibernateTemplate.delete(user);
	}

	@Override
	public Users selectUserByUserid(Long id) {
		return this.hibernateTemplate.get(Users.class, id);
	}

}

```

#### Spring 整合 Junit4  ####

> 使用 @Runwith  和 @ContextConfiguration 注解 

- UserDaoImplTest.java:

```java
package com.szxy.test;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.annotation.Transactional;

import com.szxy.dao.UsersDao;
import com.szxy.pojo.Users;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class UserDaoImplTest {

	@Autowired
	private UsersDao userDao;
	
	/**
	 * 添加用户
	 */
	@Test
	@Transactional  //在测试类对于事务提交方式默认的是回滚。
	@Rollback(value=false) //取消回滚
	public void testInsertUser(){
		Users user = new Users();
		user.setUserid(1L);
		user.setUsername("张三");
		user.setPassword("12311");
		this.userDao.insertUser(user);
	}
	
	/****
	 * 更新用户
	 */
	@Test
	@Transactional
	@Rollback(value=false)
	public void testupdateUser(){
		Users user = new Users();
		user.setUserid(1L);
		user.setUsername("张1");
		user.setPassword("12311");
		this.userDao.updateUser(user);
	}
	
	/**
	 * 删除用户
	 */
	@Test
	@Transactional
	@Rollback(false)
	public void testdeleteUser(){
		Users user = new Users();
		user.setUserid(1L);
		this.userDao.deleteUser(user);
	}
	
	/**
	 * 查找用户信息
	 */
	@Test
	@Transactional
	@Rollback(false)
	public void testSelectUserByUserid(){
		Users user = this.userDao.selectUserByUserid(2L);
		System.out.println(user);
	}
	
}
```

### HQL 查询 ###

> Hibernate Query Language 
>
> 语法：将原来的表名与字段名用换成对象的名称和属性名称

| 方法              | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| getCurrentSession | 当前 session 必须要有事务边界，且只能处理唯一 的事务         |
| openSession       | 当事务提交或者回滚后 session 自动失效 每次打开都会获取一个新的 sesion 对象，即多次打开后去多个 session 对象，所以我们使用完毕后记得要手动关闭 session |

```java
/**
* 根据用户名查询用户信息
*/
@Override
public List<Users> selectUserByUsername(String username) {
Session session =
this.hibernateTemplate.getSessionFactory().getCurrentSession();
    // HQL 查询
    Query query = session.createQuery("from Users where username = :uname");
    // 根据参数名绑定
    // Query queryTemp = query.setString("uname", username);
    // 根据参数位置绑定,注意与 JDBC 方式不同时，参数位置下标从 0 开始
    // Query queryTemp = query.setString(0, username);
    // 简写
    Query queryTemp = 
	session.createQuery("from Users where username = :uname").
	setString("uname", username);
    // 返回查询结果
    return queryTemp.list();
}
```

### SQL 查询 ###

```java
Override
public List<Users> selectUserByUsernameSQL(String username) {
		Session session =
				this.hibernateTemplate.getSessionFactory().getCurrentSession();
		Query query = session.
			createSQLQuery("select * from t_users where username = ?").
			addEntity(Users.class).setString(0, username);
		return query.list();
	}
```

### QBC 查询 ###

```java
/****
* 
*  QBC 查询  根据用户名查询用户信息   
*/
@Override
public List<Users> selectUserByUsernameQBC(String username) {
    Session session = 
        this.hibernateTemplate.getSessionFactory().getCurrentSession();
    Criteria c = session.createCriteria(Users.class);
    c.add(Restrictions.eq("username",username));
    return c.list();
}
```

## Spring 整合  Hibernate JPA ##

> JPA：由 Sun 公司提供了一对对于持久层操作的标准(**接口+文档**) 
>
> Hibernate:是 Gavin King 开发的一套对于持久层操作的自动的 ORM 框架。 
>
> Hibernate JPA:是在 Hibernate3.2 版本那种提供了对于 JPA 的标准的实现。
>
> 提供了一套按 照 JPA 标准来实现持久层开发的 AP

拷贝工程项目，并添加新的 jar 包  hibernate-entitymanager-5.0.7.Final.jar

修改 applicationContext.xml 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/aop
	http://www.springframework.org/schema/aop/spring-aop.xsd
	http://www.springframework.org/schema/tx 
	http://www.springframework.org/schema/tx/spring-tx.xsd">

	<!-- 加载数据库连接配置文件 -->
	<context:property-placeholder location="classpath:db.properties" />

	<!-- 加载 C3p0 数据库连接池 -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="driverClass" value="${jdbc.driver}" />
		<property name="jdbcUrl" value="${jdbc.url}" />
		<property name="user" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
	</bean>

	<!-- 配置 Hibernate JPA -->
	<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="jpaVendorAdapter">
			<bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
				<!-- hibernate 相关的属性的注入 --> 
				<!-- 配置数据库类型 --> 
				<property name="database" value="MYSQL"/> 
				<!-- 正向工程 自动创建表 --> 
				<property name="generateDdl" value="true"/> 
				<!-- 显示执行的 SQL --> 
				<property name="showSql" value="true"/> 
			</bean>
		</property>
		<!-- 扫描实体类所在的包 -->
		<property name="packagesToScan">
			<list>
				<value>com.szxy.pojo</value>
			</list>
		</property>
	</bean>

	<!-- 配置 Hibernate 的事务管理器 --> 
	<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager"> 
		<property name="entityManagerFactory" ref="entityManagerFactory"/> 
	</bean>

	<!-- 配置注解事务处理 -->
	<tx:annotation-driven transaction-manager="transactionManager"/>

	<!--配置 springIOC 的注解扫描 -->
	<context:component-scan base-package="com.szxy" />

</beans>
```

- UserDaoImpl.java:

```java
@Repository
public class UsersDaoImpl implements UsersDao {

	@PersistenceContext(name="entityManagerFactory")
	private EntityManager entityManager;
	
	@Override
	public void insertUser(Users user) {
		this.entityManager.persist(user);
	}

	@Override
	public void updateUser(Users user) {
		this.entityManager.merge(user);
	}

	@Override
	public void deleteUser(Users user) {
		Users u = selectUserByUserid(user.getUserid());
		this.entityManager.remove(u);
	}

	@Override
	public Users selectUserByUserid(Long id) {
		return this.entityManager.find(Users.class, id);
	}
	
	/**
	 *  HQL 查询
	 */
	@SuppressWarnings("unchecked")
	@Override
	public List<Users> selectUserByUsernameHQL(String username) {
		String hql = "from Users where username = :abc";
		return this.entityManager.createQuery(hql).
				setParameter("abc", username).
				getResultList();
	}

	/**
	 *  SQL 查询
	 */
	@SuppressWarnings("unchecked")
	@Override
	public List<Users> selectUserByUsernameSQL(String username) {
		String sql = "select * from t_users where username = ?";
		return this.entityManager.createNativeQuery(sql,Users.class).
				setParameter(1, username).
				getResultList();
	}

	
	/***
	 *  QBC 查询
	 */
	@Override
	public List<Users> selectUserByUsernameQBC(String username) {
		// 创建  CriteriaBuilder 对象  
		CriteriaBuilder builder = this.entityManager.getCriteriaBuilder();
		// 从 CriteriaBuilder 对象中获取 Query 对象
		// 这相当于  select * from t_users
		CriteriaQuery<Users> query = builder.createQuery(Users.class);
		// 获取要查询数据的实体类对象
		Root<Users> root = query.from(Users.class);
		// 封装查询条件
		Predicate predicate = builder.equal(root.get("username"), username);
		// select  * from t_users where username = ?
		query.where(predicate);
		// 执行查询
		TypedQuery<Users> typedQuery = this.entityManager.createQuery(query);
		return typedQuery.getResultList();
	}
	
	
	
}

```

### HQL 查询 ###

```java
/**
	 *  HQL 查询
	 */
@SuppressWarnings("unchecked")
@Override
public List<Users> selectUserByUsernameHQL(String username) {
    String hql = "from Users where username = :abc";
    return this.entityManager.createQuery(hql).
        setParameter("abc", username).
        getResultList();
}

```

### SQL 查询 ###

```java
/**
	 *  SQL 查询
	 */
@SuppressWarnings("unchecked")
@Override
public List<Users> selectUserByUsernameSQL(String username) {
    String sql = "select * from t_users where username = ?";
    return this.entityManager.createNativeQuery(sql,Users.class).
        setParameter(1, username).
        getResultList();
}


```

### QBC 查询 ###

```java
/***
	 *  QBC 查询
	 */
@Override
public List<Users> selectUserByUsernameQBC(String username) {
    // 创建  CriteriaBuilder 对象  
    CriteriaBuilder builder = this.entityManager.getCriteriaBuilder();
    // 从 CriteriaBuilder 对象中获取 Query 对象
    // 这相当于  select * from t_users
    CriteriaQuery<Users> query = builder.createQuery(Users.class);
    // 获取要查询数据的实体类对象
    Root<Users> root = query.from(Users.class);
    // 封装查询条件
    Predicate predicate = builder.equal(root.get("username"), username);
    // select  * from t_users where username = ?
    query.where(predicate);
    // 执行查询
    TypedQuery<Users> typedQuery = this.entityManager.createQuery(query);
    return typedQuery.getResultList();
}

```

## Spring Data JPA  ##

> Spring Data JPA：Spring Data JPA 是 spring data 项目下的一个模块。提供了一套基于 JPA 标准操作数据库的简化方案。底层默认的是依赖 Hibernate JPA 来实现的。 Spring Data JPA 的技术特点：我们只需要定义接口并集成 Spring Data JPA 中所提供的接 口就可以了。不需要编写接口实现类。 

- 修改 applitionContext.xml 

  在原来 applicationContext.xml 的基础上，添加扫描 dao 层接口包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:jpa="http://www.springframework.org/schema/data/jpa"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/aop
	http://www.springframework.org/schema/aop/spring-aop.xsd
	http://www.springframework.org/schema/data/jpa
	http://www.springframework.org/schema/data/jpa/spring-jpa.xsd
	http://www.springframework.org/schema/tx 
	http://www.springframework.org/schema/tx/spring-tx.xsd">

	<!-- 加载数据库连接配置文件 -->
	<context:property-placeholder location="classpath:db.properties" />

	<!-- 加载 C3p0 数据库连接池 -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="driverClass" value="${jdbc.driver}" />
		<property name="jdbcUrl" value="${jdbc.url}" />
		<property name="user" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
	</bean>

	<!-- 配置 Hibernate JPA -->
	<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="jpaVendorAdapter">
			<bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
				<!-- hibernate 相关的属性的注入 --> 
				<!-- 配置数据库类型 --> 
				<property name="database" value="MYSQL"/> 
				<!-- 正向工程 自动创建表 --> 
				<property name="generateDdl" value="true"/> 
				<!-- 显示执行的 SQL --> 
				<property name="showSql" value="true"/> 
			</bean>
		</property>
		<!-- 扫描实体类所在的包 -->
		<property name="packagesToScan">
			<list>
				<value>com.szxy.pojo</value>
			</list>
		</property>
	</bean>

	<!-- 配置 Hibernate 的事务管理器 --> 
	<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager"> 
		<property name="entityManagerFactory" ref="entityManagerFactory"/> 
	</bean>

	<!-- 配置注解事务处理 -->
	<tx:annotation-driven transaction-manager="transactionManager"/>

	<!--配置 springIOC 的注解扫描 -->
	<context:component-scan base-package="com.szxy" />

	<!-- 扫描 dao 层包 -->
	<jpa:repositories base-package="com.szxy.dao" />
	
</beans>
```

- UserDao.java: **继承 JpaRepository 类**

```java
package com.szxy.dao;

import java.io.Serializable;

import org.springframework.data.jpa.repository.JpaRepository;

import com.szxy.pojo.Users;

public interface UsersDao extends JpaRepository<Users, Serializable>{

	
}
```

## Spring Data JPA 接口继承 ##

![](http://img.zwer.xyz/blog/20190730205215.png)

## Spring Data JPA 运行原理 ##

```java
// 获取 entityManagerFactory 工厂，从该工厂获取 EntityManager 对象，注入到 em 对象中
@PersistenceContext(name="entityManagerFactory")
private EntityManager em;
@Test
public void test1(){
    // org.springframework.data.jpa.repository.support.SimpleJpaRepository@4f186450
    System.out.println(this.userDao);
    // class com.sun.proxy.$Proxy30  
    System.out.println(this.userDao.getClass());

    JpaRepositoryFactory factory = new JpaRepositoryFactory(em); 
    // getRepository(UsersDao.class);可以帮助我们为接口生成实现类。
    // 而这个实现类是 SimpleJpaRepository 的对象 
    // 要求：该接口必须要是继承 Repository 接口 
    UsersDao ud = factory.getRepository(UsersDao.class);
    System.out.println(ud); 
    System.out.println(ud.getClass());

}

```

## Repository 接口 ##

Repository 接口是 Spring Data JPA 中为我们提供的所有接口中的**顶层接口**，也是标志接口

Repository 提供了两种查询方式的支持 

1. 基于方法名称命名规则查询 
2. 基于@Query 注解查询 

#### 方法名称命名规则查询 ####

> 规则：**findBy(关键字)+属性名称(属性名称的*首字母大写*)+查询条件(首字母大写)** 

| 关键字        | 方法命名                          | where 子句                     |
| ------------- | --------------------------------- | ------------------------------ |
| Is，Equal     | findById,findByIdEqual,findByIdIs | where id = ?                   |
| And           | findByUsernameAndAge              | where username = ? and age = ? |
| LessThanEqual | findByIdLessThanEqual             | where id <= ?                  |
| Before        | findByIdBefore                    | where id < ?                   |
| IsNull        | findByNameIsNull                  | where name is null             |

| 查询条件                                                     |
| ------------------------------------------------------------ |
| Is Equal  And LessThanEqual GreatterThanEqual Before After IsNUll Like |

UserDao.java: 接口

```java
public interface UsersDao extends Repository<Users, Serializable>{
	
	/**
	 * 根据用户名查询用户信息
	 * @param string
	 * @return
	 */
	List<Users> findByUsername(String username);
	
	/**
	 * 根据用户名进行模糊查询
	 * @param username
	 * @return
	 */
	List<Users> findByUsernameLike(String username);
	
	/**
	 * 根据用户名和用户年龄查询用户信息
	 * @param string
	 * @param i
	 * @return
	 */
	List<Users> findByUsernameAndUserageGreaterThanEqual(String string, Integer userage);	
}
```

UserDaoImplTest.java:测试类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class UserDaoImplTest {

	@Autowired
	private UsersDao userDao;
	
	/**
	 *  需求：根据用户名查询用户信息
	 */
	@Test
	public void testRepository(){
		List<Users> users = this.userDao.findByUsername("张三");
		for (Users u : users) {
			System.out.println(u);
		}
		
	}
	
	/*****
	 *  需求：根据用户名的姓氏查询用户信息
	 */
	@Test
	public void testRepository2(){
		List<Users> users = this.userDao.findByUsernameLike("赵%");
		for (Users u : users) {
			System.out.println(u);
		}
	}

	
	/****
	 * 需求：根据用户名和年龄查询用户信息
	 */
	@Test
	public void testRepository3(){
		List<Users> users = this.userDao.findByUsernameAndUserageGreaterThanEqual("张三",17);
		for (Users u : users) {
			System.out.println(u);
		}
	}
	
}
```

#### 基于 @Query 注解查询 ####

1. **通过 JPQL 语句查询**

   JPQL：通过 Hibernate 的 HQL 演变过来的。他和 HQL 语法及其相似。 


```java
public interface UsersDao extends Repository<Users, Serializable>{
	
	// 方法命名规则参数查询
	List<Users> findByUsername(String username);
	List<Users> findByUsernameLike(String username);
	List<Users>
        findByUsernameAndUserageGreaterThanEqual(String string, Integer userage);

	// 基于 @Query 注解查询
	@Query(value="from Users where username = ?")
	List<Users> userQueryByUsername(String username);
	@Query(value="from Users where username like ?")
	List<Users> userQueryByUsernameLike(String username);
	@Query("from Users where username = ? and userage >= ?")
	List<Users>
    userQueryByUsernameAndUserageGreatterThanEqual(String username,Integer userage);
	
}
```

2. **通过 SQL 语句查询**

```java
// 基于 @Query 注解 SQL查询
// nativeQuery 默认是 false ，表示不开启。作用是对 value 语句做本地转译
    @Query(value="select * from t_users where username = ?",nativeQuery=true)
    List<Users> userQueryByUsernameUseSQL(String username);
    @Query(value="select * from t_users where username like ?",nativeQuery=true)
    List<Users> userQueryByUserNameLikeUseSQL(String username);
    @Query(value="select * from t_users where username = ?  and userage >= ?",nativeQuery=true)
    List<Users> 
    userQueryByUserNameAndUserageGreatterThanEqualUseSQL(String username,Integer userage);
```

3. **通过 @Query 注解完成数据的更新**

```java
// 基于 @Query 注解完成数据的更新
@Query("update Users set userage = ? where userid = ?")
@Modifying  //表示该条 HQL 语句是更新语句
void updateUserageById(Integer userage,Long userid);
```

由于 HQL 书写错误，导致程序无法正常运行

```java
Caused by: 
org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'usersDao': 
Invocation of init method failed; 
nested exception is java.lang.IllegalArgumentException:
Validation failed for query for method public abstract void com.szxy.dao.UsersDao.updateUserageById(java.lang.Integer,java.lang.Long)!
```

## CrudRepository 接口 ##

本身就提供事务提交的操作

```java
package com.szxy.dao;

import java.io.Serializable;
import java.util.List;

import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.CrudRepository;
import org.springframework.data.repository.Repository;

import com.szxy.pojo.Users;

public interface UsersDao extends CrudRepository<Users, Serializable>{
		
}
```

## PagingAndSortingRepository 接口 ##

### 分页处理 ###

- 新建 UserDao.java  接口

```java
public interface UsersDao extends PagingAndSortingRepository<Users, Serializable>{
		
}
```

- 测试分页

```java
@Test
public void testPage(){
    int page = 3; //当前分页的索引，从 0 开始
    int size = 3; //每页的条数
    Pageable pageable = new PageRequest(page, size);
    Page<Users> pageResult = this.userDao.findAll(pageable);
    long totalElements = pageResult.getTotalElements();
    System.out.println("分页的总记录数"+totalElements);
    int totalPages = pageResult.getTotalPages();
    System.out.println("总的页数："+totalPages);
    List<Users> userList = pageResult.getContent();
    for (Users u : userList) {
    System.out.println(u);
    }
}
```

### 排序处理 ###

```java
/***
 * 多列排序
*/
@Test
public void testManyColumnSort(){
    //Sorr 对象封装了排序的规则和需要排序的属性
    // direction 排序规则（正排还是倒排）
    // properties 排序的属性（pojo类的成员变量）
    Order order1 = new Order(Direction.DESC,"userage");
    // Mysql 按照字符编码排序 
    Order order2 = new Order(Direction.DESC,"username");
    Sort sort = new  Sort(order1,order2);
    Iterable<Users> users = this.userDao.findAll(sort);
    for (Users u : users) {
        System.out.println(u);
    }
}
```

Mysql 对中文排序处理

```sql
select * from t_users order by convert(username using gbk) collate gbk_chinese_ci;
```

## JPARepository 接口 ##

特点：将**其他接口的返回值做适配处理**，方便我们开发使用这些方法，是我们开发使用最多的接口

- 创建接口

```java
public interface UsersDao extends JpaRepository<Users, Serializable>{
	
}
```

- 测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class UserDaoImplTest {

	@Autowired
	private UsersDao userDao;

	@Test
	public void testPage(){
		List<Users> users = userDao.findAll();
		for (Users u : users) {
			System.out.println(u);
		}
    }
	
}
```

## JpaSpecificationExecutor接口 ##

> JpaSpecificationExecutor 接口不可单独使用
>
> 多条件查询加多条件查询分页查询

- 创建接口

```java
/***
 *   JpaSpecificationExecutor 接口不可单独使用
 *   必须配合其他接口使用 
 *   JpaSpecificationExecutor 是一个独立的接口，没有实现 Repository 接口
 *   而只有继承 Repository 接口或者其子接口，才能创建 UsersDao 接口的代理对象
 *  
 */
public interface UsersDao 
	extends JpaRepository<Users, Serializable>,JpaSpecificationExecutor<Users>{
}
```

- 测试方法

#### 多条件查询 ####

```java
/**
	 * 多条件查询一
	 * 根据用户名和用户年龄查询用户信息
	 */
	@Test
	public void testFind2(){
		
		Specification<Users> spec = new Specification<Users>() {

			@Override
			public Predicate toPredicate(Root<Users> root, CriteriaQuery<?> c, CriteriaBuilder builder) {
				List<Predicate> list = new ArrayList<>();
				list.add(builder.equal(root.get("username"), "tom"));
				list.add(builder.equal(root.get("userage"), 67));
				Predicate[] pres = new Predicate[list.size()];
				// and 方法表示条件之间的关系  与
				return builder.and(list.toArray(pres));
			}
		};
		
		List<Users> users = this.userDao.findAll(spec);
		for (Users u : users) {
			System.out.println(u);
		}		
		
	}
	
	/**
	 * 多条件查询二
	 * 根据用户名和用户年龄查询用户信息
	 */
	@Test
	public void testFind3(){
		
		Specification<Users> spec = new Specification<Users>() {
			@Override
			public Predicate toPredicate(Root<Users> root, CriteriaQuery<?> c, CriteriaBuilder builder) {
     // where username = ? or userage = ?
     //return
     //builder.or(builder.equal(root.get("username"),"tom"),
                builder.equal(root.get("userage"),20));
	  // where username = ? or userage = 20 or userage = 20 and password = 12311 
		return builder.or(builder.equal(root.get("username"), "tom"),
builder.equal(root.get("userage"),20),				         	builder.and(builder.equal(root.get("userage"),20),                                                   builder.equal(root.get("password"), "12311")));
			}
		};
		
		List<Users> users = this.userDao.findAll(spec);
		for (Users u : users) {
			System.out.println(u);
		}		
		
	}
```

#### 分页多条件查询 ####

```java
/**
*   分页和多条件查询
*/
@Test
public void testFind4(){

    Specification<Users> spec = new Specification<Users>() {

        @Override
        public Predicate toPredicate(Root<Users> root, CriteriaQuery<?> c, CriteriaBuilder builder) {
            return builder.like(root.get("username").as(String.class), "王%");
        }
    };
    Pageable pageable = new PageRequest(1, 2);
    Page<Users> pageResult =  this.userDao.findAll(spec, pageable);
    System.out.println("总记录数"+pageResult.getTotalElements());
    System.out.println("总页数"+pageResult.getTotalPages());
    List<Users> userList = pageResult.getContent();
    for (Users u : userList) {
        System.out.println(u);
    }		

}
```

#### 排序多条件查询 ####

```java
/**
* 多条件查询+排序
*/
@Test
public void testFind5(){
    Specification<Users> spec = new Specification<Users>() {

        @Override
        public Predicate toPredicate(Root<Users> root, CriteriaQuery<?> arg1, CriteriaBuilder cb) {
            return cb.like(root.get("username"), "王%");
        }
    };
    
    // 根据用户 ID ，做倒序排序
    Sort sort = new Sort(Direction.DESC,"userid");
    List<Users> userList = this.userDao.findAll(spec, sort);
    for (Users u : userList) {
        System.out.println(u);
    }

}
```

#### 排序多条件分页查询 ####

```java
/**
	 * 多条件查询+排序+分页
	 */
	@Test
	public void testFind6(){
		// 根据用户 ID ，倒序排序
		Sort sort = new Sort(Direction.DESC, "userid");
		// 多条件查询
		Specification<Users> spec = new Specification<Users>() {

			@Override
			public Predicate toPredicate(Root<Users> root, CriteriaQuery<?> arg1, CriteriaBuilder cb) {
				return cb.like(root.get("username"), "王%");
			}
			
		};
		// 将排序整合到分页中
		Pageable pageable = new PageRequest(1, 2, sort);
		Page<Users> pageResult = this.userDao.findAll(spec, pageable);
		List<Users> userlist = pageResult.getContent();
		System.out.println("总记录数"+pageResult.getTotalElements());
		System.out.println("总页数"+pageResult.getTotalPages());
		List<Users> userList = pageResult.getContent();
		for (Users u : userList) {
			System.out.println(u);
		}		
	}
```

## 用户自定义 Repository 接口 ##

- 创建接口 UserRepository 

```java
/**
 * 自定义 Repository
 *
 */
public interface UsersRepository {
	public Users findUsersById(Long id);
}

```

- 创建接口实现类 UserDaoImpl

  注意接口实现类的名称，即在接口名称后面加上 `Impl`

```java
public interface UsersDao  extends UsersRepository{
	
}
```

- 测试方法

```java
@Repository
public class UserDaoImpl implements UsersDao {

	@PersistenceContext(name="entityManagerFactory")
	private EntityManager em;
	
	@Override
	public Users findUsersById(Long id) {
		//第一个参数，将查询结果映射到 Users 对象中
		//第二个参数，查询的条件用户 ID
		return em.find(Users.class, id);
	}
}

```

## Spring-Data-Jpa 关联映射 ##

需求：创建用户与角色一对一关联关系

#### 一对一关联映射 ####

- 创建 Role 角色实体

```java
@Entity
@Table(name="t_roles")
public class Roles {
	
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="roleid")
	private Integer roleid;
	@Column(name="rolename")
	private String rolename;
	// 一对一关联映射
	@OneToOne(mappedBy="roles")
	private Users users;
	public Roles() {
		super();
		// TODO Auto-generated constructor stub
	}
	public Roles(Integer roleid, String rolename, Users users) {
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
	public Users getUsers() {
		return users;
	}
	public void setUsers(Users users) {
		this.users = users;
	}
	
	@Override
	public String toString() {
		return "Roles [roleid=" + roleid + ", rolename=" + rolename + "]";
	}
	
}

```

- 创建 Users 用户实体

```java
@Entity
@Table(name="t_users") //1.将 Users 类与数据库中 t_users 关联  2.若数据库中没有该表，则会创建相应的表，表名为 name 属性中的值
public class Users implements  Serializable{
	
	private static final long serialVersionUID = 1L;

	@Id  
	@GeneratedValue(strategy=GenerationType.IDENTITY)//数据库主键自增长
	@Column(name="userid")
	private Long userid;
	
	@Column(name="username")
	private String username;
	
	@Column(name="password")
	private String password;
	
	@Column(name="userage")
	private Integer userage;
	
	//一对一关联映射
	@OneToOne(cascade=CascadeType.PERSIST)
	// 表示由 t_users 表维护一对一关联关系
	// t_roles 表的外键名为 role_id
	@JoinColumn(name="role_id")
	private Roles roles;

	public Users() {
		super();
		// TODO Auto-generated constructor stub
	}

	public Users(Long userid, String username, String password, Integer userage, Roles roles) {
		super();
		this.userid = userid;
		this.username = username;
		this.password = password;
		this.userage = userage;
		this.roles = roles;
	}

	
	public Long getUserid() {
		return userid;
	}

	public void setUserid(Long userid) {
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

	public Roles getRoles() {
		return roles;
	}

	public void setRoles(Roles roles) {
		this.roles = roles;
	}

	public static long getSerialversionuid() {
		return serialVersionUID;
	}

	@Override
	public String toString() {
		return "Users [userid=" + userid + ", username=" + username + ", password=" + password + ", userage=" + userage
				+ ", roles=" + roles + "]";
	}
	
	
}

```

- 测试方法

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class UserDaoTest {

	@Autowired
	private UsersDao userDao;
	
	/*****
	 * 添加用户和角色信息
	 */
	@Test
	public void test1(){
		Roles role = new Roles();
		role.setRolename("管理员");
		Users u = new Users();
		u.setUsername("admin");
		u.setPassword("123");
		u.setUserage(21);
		//设置关联信息
		role.setUsers(u);
		u.setRoles(role);
	
		//先保存角色信息，再保存用户信息
		this.userDao.save(u);
	}
	
	/***
	 * 查询用户信息
	 */
	@Test
	public void test2() {
		Users user = this.userDao.findOne(27L);
		System.out.println(user);
	}

}

```



#### 一对多关联映射 ####

需求：创建一个用户用户拥有多个角色关系

角色：一方     

用户： 多方

- 创建 Role 实体

```java
@Entity
@Table(name="t_roles")
public class Roles {
	
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="roleid")
	private Integer roleid;
	@Column(name="rolename")
	private String rolename;
	
	// 一对多关联映射
	@OneToMany(mappedBy="roles",cascade=CascadeType.PERSIST)
	private Set<Users> users = new HashSet<>();
	
	
	public Set<Users> getUsers() {
		return users;
	}


	public void setUsers(Set<Users> users) {
		this.users = users;
	}

	public Roles(Integer roleid, String rolename, Set<Users> users) {
		super();
		this.roleid = roleid;
		this.rolename = rolename;
		this.users = users;
	}

	public Roles() {
		super();
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

	@Override
	public String toString() {
		return "Roles [roleid=" + roleid + ", rolename=" + rolename+" ]";
	}
	
}

```

- 创建 Users 实体

```java
@Entity
@Table(name="t_users") //1.将 Users 类与数据库中 t_users 关联  2.若数据库中没有该表，则会创建相应的表，表名为 name 属性中的值
public class Users implements  Serializable{
	
	private static final long serialVersionUID = 1L;

	@Id  
	@GeneratedValue(strategy=GenerationType.IDENTITY)//数据库主键自增长
	@Column(name="userid")
	private Long userid;
	
	@Column(name="username")
	private String username;
	
	@Column(name="password")
	private String password;
	
	@Column(name="userage")
	private Integer userage;
	
	//多对一关联映射
	@ManyToOne(cascade=CascadeType.PERSIST)
	@JoinColumn(name="role_id")
	private Roles roles;
	
	public Users() {
		super();
		// TODO Auto-generated constructor stub
	}

	
	public Users(String username, String password, Integer userage) {
		super();
		this.username = username;
		this.password = password;
		this.userage = userage;
	}



	public Users(Long userid, String username, String password, Integer userage, Roles roles) {
		super();
		this.userid = userid;
		this.username = username;
		this.password = password;
		this.userage = userage;
		this.roles = roles;
	}

	public Long getUserid() {
		return userid;
	}

	public void setUserid(Long userid) {
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

	public Roles getRoles() {
		return roles;
	}

	public void setRoles(Roles roles) {
		this.roles = roles;
	}

	public static long getSerialversionuid() {
		return serialVersionUID;
	}

	@Override
	public String toString() {
		return "Users [userid=" + userid + ", username=" + username + ", password=" + password + ", userage=" + userage
				+ ", roles=" + roles + "]";
	}
	
}
```

- 测试方法

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class UserDaoTest {

	@Autowired
	private UsersDao userDao;
	
	/**
	 * 添加用户和角色
	 */
	@Test
	public void test1() {
		//创建角色
		Roles role = new Roles();
		role.setRolename("普通用户");
		//创建用户
		Users user = new Users();
		user.setUsername("李白");
		user.setPassword("123");
		user.setUserage(1000);
		//建立关联
		role.getUsers().add(user);
		user.setRoles(role);
		//保存数据
		this.userDao.save(user);
	}
	
	/***
	 * 查询用户信息
	 */
	@Test
	public void test2(){
		Users user = this.userDao.findOne(31L);
		System.out.println("用户姓名："+user.getUsername());
		Roles role = user.getRoles();
		System.out.println(role);
	}	
}
```

#### 多对多关联映射 ####

需求：菜单与角色之间多对多关联

- 创建菜单实体

```java
@Entity
@Table(name="t_menus")
public class Menus {
	
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="munuid")
	private Integer menuid;
	@Column(name="menuname")
	private String menuname;
	@Column(name="menuurl")
	private String menuurl;
	@Column(name="parent_id")
	private Integer parent_id;
	
	// 多对多关联映射
	@ManyToMany(mappedBy="menus")
	private Set<Roles> roles = new HashSet<>();

	public Menus() {
	}

	public Menus(Integer menuid, String menuname, String menuurl, Integer parent_id, Set<Roles> roles) {
		super();
		this.menuid = menuid;
		this.menuname = menuname;
		this.menuurl = menuurl;
		this.parent_id = parent_id;
		this.roles = roles;
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

	public String getMenuurl() {
		return menuurl;
	}

	public void setMenuurl(String menuurl) {
		this.menuurl = menuurl;
	}

	public Integer getParent_id() {
		return parent_id;
	}

	public void setParent_id(Integer parent_id) {
		this.parent_id = parent_id;
	}

	public Set<Roles> getRoles() {
		return roles;
	}

	public void setRoles(Set<Roles> roles) {
		this.roles = roles;
	}

	@Override
	public String toString() {
		return "Menus [menuid=" + menuid + ", menuname=" + menuname + ", menuurl=" + menuurl + ", parent_id="
				+ parent_id+"]";
	}
	
	
}

```

- 创建角色实体

```java
@Entity
@Table(name="t_roles")
public class Roles {
	
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="roleid")
	private Integer roleid;
	@Column(name="rolename")
	private String rolename;
	
	// 多对多关联映射  
	// fetch 设置查询  FetchType.EAGER 立即加载
	@ManyToMany(cascade=CascadeType.PERSIST,fetch=FetchType.EAGER)
	// @JoinTable 配置多对多关系中间表
	// @JoinColumn 配置中间表中外键字段
	@JoinTable(joinColumns=@JoinColumn(name="role_Id"),inverseJoinColumns=@JoinColumn(name="menu_Id"))
	private Set<Menus> menus = new HashSet<>();

	public Roles() {
		super();
		// TODO Auto-generated constructor stub
	}

	public Roles(Integer roleid, String rolename, Set<Menus> menus) {
		super();
		this.roleid = roleid;
		this.rolename = rolename;
		this.menus = menus;
	}

	@Override
	public String toString() {
		return "Roles [roleid=" + roleid + ", rolename=" + rolename + ", menus=" + menus + "]";
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

	public Set<Menus> getMenus() {
		return menus;
	}

	public void setMenus(Set<Menus> menus) {
		this.menus = menus;
	} 

}
```

- 测试方法

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class RolesDaoTest {

	@Autowired
	private RolesDao rolesDao;
	
	@Test
	public void test1(){
		Roles role = rolesDao.findOne(5);
		System.out.println("用户姓名:"+role.getRolename());
		Set<Menus> menus = role.getMenus();
		for (Menus m : menus) {
			System.out.println(m);
		}
	}	
	
	@Test
	public void test2(){
		//创建角色
		Roles role = new Roles();
		role.setRolename("超级管理员");
		// 创建菜单 xxx 管理平台 --> 用户管理
		Menus menus = new Menus();
		menus.setMenuid(1);
		menus.setMenuname("xxx 管理平台");
		menus.setMenuurl(null);
		menus.setParent_id(-1);
		
		Menus menus2 = new Menus();
		menus.setMenuid(2);
		menus2.setMenuname("用户管理");
		menus2.setMenuurl("/user/all");
		menus2.setParent_id(1);
		
		// 建立关联关系
		role.getMenus().add(menus);
		role.getMenus().add(menus2);
		
		menus.getRoles().add(role);
		menus2.getRoles().add(role);
		
		//保存数据
		this.rolesDao.save(role);
		
		
	}
}
```

#### **总结** ####

| 注解            | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| @Entity         | 表示这个类是实体类                                           |
| @Table          | 一将实体类与表关联起来<br/>二若数据库没有该数据表，则会创建该表，其表名为 name 属性值 |
| @Id             | 用于成员属性上，表示该属性对应的字段为主键                   |
| @GeneratedValue | 用于设置主键的增长类型<br/>strategy 设置 strategy=GenerationType.IDENTITY<br/>IDENTITY 主键由数据库自动生成（主要是自动增长型）<br/>SEQUENCE 根据底层数据库的序列来生成主键，条件是数据库支持序列。 |
| @Column         | 一 将实体类的属性与表的字段相关联<br/>二 若数据库没有该数据表，则会创建该表，其对应的字段名为 name 属性值 |
| @OneToOne       | 表示一对一关联映射                                           |
| @OneToMany      | 表示一对多关联映射                                           |
| @ManyToMany     | 表示多对多关联映射<br/>mappeBy 属性表示与另一个实体类的关联<br/>fetch 设置是否延迟加载，默认是延迟加载，若不想延迟加载，可设置 fetch 属性为 fetch=FetchType.EAGER)<br/>cascade 设置级联操作 cascade=CascadeType.PERSIST |
| @JoinTable      | 表示创建中间表<br/>name 属性表示创建中间表的表名             |
| @JoinColumn     | 表示创建中间表的外键字段<br/>name 属性表示创建中间表中字段名 。eg. @JoinColumn(name="role") |

## Spring Data JPA  Redis  ##

> Spring Data Redis, part of the larger Spring Data family, provides easy configuration and access to Redis from Spring applications. It offers both low-level and high-level abstractions for interacting with the store, freeing the user from infrastructural concerns.

> Java 操作 Redis 的封装

### 准备 ###

**安装 gcc 库**

```shell
yum install gcc-c++
```

**安装 Redis** 

- 上传 Redis 压缩包到 Linux 中
- 解压 Redis 压缩包
- 进入  Redis 压缩包中

```shell
#指安装目录
make PREFIX=/usr/local/redis install

#默认的是前置启动：
./redis-server

#先将 redis.conf 文件拷贝到 redis 的安装目录 
cp redis.conf /usr/local/redis/bin
#编辑 redis.conf 文件修改：daemonize yes 启动： 
./redis-server redis.conf
#查看 redis 进程：
ps aux|grep redis 
#关闭后置启动的 
Redis：./redis-cli shutdown
```

### Spring 与 Redis 整合 ###

**搭建环境**

- **新建 lib 文件夹，添加 jar  包，并发布到类路径上**

- **添加 applicationContext.xml 配置文件**（重点）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd">
	
	<!-- 配置读取properties文件的工具类 -->
	<context:property-placeholder location="classpath:redis.properties"/>
	
	<!-- Jedis连接池 -->
	<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
		<property name="maxTotal" value="${redis.pool.maxtTotal}"/>
        <!---
        最大空闲数，数据库连接的最大空闲时间。超过空闲时间，数据库连
        接将被标记为不可用，然后被释放。设为0表示无限制。
        ->
		<property name="maxIdle" value="${redis.pool.maxtIdle}"/>
		<property name="minIdle" value="${redis.pool.minIdle}"/>
	</bean>
	
	<!-- Jedis连接工厂:创建Jedis对象的工厂 -->
	<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
		<!-- IP地址 -->
		<property name="hostName" value="${redis.hostname}"/>
		<!-- 端口 -->
		<property name="port" value="${redis.port}"/>
		<!-- 连接池 -->
		<property name="poolConfig" ref="poolConfig"/>
	</bean>
	
	<!-- Redis模板对象:是SpringDataRedis提供的用户操作Redis的对象 -->
	<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
		<property name="connectionFactory" ref="jedisConnectionFactory" />
		<!-- 默认的序列化器：序列化器就是根据规则将存储的数据中的key与value做字符串的序列化处理 -->
		<!-- keySerializer、valueSerializer：对应的是Redis中的String类型 -->
		<!-- hashKeySerializer、hashValueSerializer：对应的是Redis中的Hash类型 -->
		<property name="keySerializer">
			<bean class="org.springframework.data.redis.serializer.StringRedisSerializer"></bean>
		</property>
		<property name="valueSerializer">
		    <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"></bean>
		</property>
	</bean>
	
	<context:component-scan base-package="com.szxy" />
	
</beans>
```

- **测试整合**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class RedisTest {

	@Autowired
	private RedisTemplate<String, Object>  redisTemplate;
	
	@Test
	public void testRedisStore() {
		this.redisTemplate.opsForValue().set("key", "Hello World");
	}
	
	@Test
	public void testRedisGet(){
		String value =  (String) this.redisTemplate.opsForValue().get("key");
		System.out.println(value);
	}
	
	@Test
	public void testUsersStore(){
		// 更换序列化器
		// 将原来默认的 String 类型的序列化器替换为 Jdk 的字节型序列化器
		// jdk 序列化器相比 json 格式存储的字符串 转换占用空间更大
		// 注意：替换后的序列化器只在本方法起作用
		this.redisTemplate.setValueSerializer(new JdkSerializationRedisSerializer());
		//创建 Users 对象
		Users user = new Users(1,"张三",28);
		this.redisTemplate.opsForValue().set("users", user);
	}
	
	@Test
	public void testUserGet(){
		// 替换为 JDK 类型的序列化器
		this.redisTemplate.setValueSerializer(new JdkSerializationRedisSerializer());
		Users user = (Users)this.redisTemplate.opsForValue().get("users");
		System.out.println(user);
	}
	
}
```

**使用 Json 格式作序列化处理**

注意：之前要导入 jackson 相关 jar 包

- jackson-annotations-2.8.0.jar
- jackson-core-2.8.10.jar
- jackson-databind-2.8.10.jar

```java
/**
	 *  以 json 格式存储 对象数据
	 */
@Test
public void testUserStoreByJson(){
    // 替换为 Jackson 类型的序列化器
    this.redisTemplate.
        setValueSerializer(new Jackson2JsonRedisSerializer<>(Users.class));
    //创建 Users 对象
    Users user = new Users(1,"张三",28);
    this.redisTemplate.opsForValue().set("user2", user);
}

@Test
public void testUserGetByJson(){
    // 替换为 Jackson 类型的序列化器
    this.redisTemplate.
        setValueSerializer(new Jackson2JsonRedisSerializer<>(Users.class));
    Users user = (Users) this.redisTemplate.opsForValue().get("user2");
    System.out.println(user);
}
```





