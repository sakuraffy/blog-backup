---
title: maven搭建SSH
date: 2016-09-23 00:07:12
tags:
	- freamwork
---
在实际开发中，我们习惯将项目分为三层，展示层，业务逻辑层，数据持久层。对于展现层我们一般会用到Strust2/Spring MVC，而数据层我们往往会用到Hibernate/Mybatis，这里我将介绍一下SSH(Spring + Struts2 + Hibernate)的搭建，至于SSM的搭建请参考[maven搭建SSM](https://sakuraffy.github.io/freamwork_ssm)

<!-- more -->

首先创建一个maven项目，这里不再赘述了，如果不太清楚，请参考[maven项目搭建](https://sakuraffy.github.io/maven_project_create)
{% qnimg freamwork/ssh/p1.png 'class:class1 class2' normal:yes %}

### 整合Spring

#### 在pom.xml中添加spring相应的依赖
``` xml
<properties>
	<!-- spring版本号 -->    
	<spring.version>4.1.9.RELEASE</spring.version>
</properties>

<dependencies>

	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.12</version>
		<scope>test</scope>
	</dependency>

	<!-- Spring依赖 -->
	<dependency>
		<groupId>org.springframework</groupId>                                        
		<artifactId>spring-core</artifactId>
		<version>${spring.version}</version>
	</dependency>

	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context</artifactId>
		<version>${spring.version}</version>
	</dependency>

	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-test</artifactId>
		<version>${spring.version}</version>
	</dependency>

</dependencies>
```

#### 在src/main/resources下添加applicationContext.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:p="http://www.springframework.org/schema/p"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:mvc="http://www.springframework.org/schema/mvc"  
    xmlns:tx="http://www.springframework.org/schema/tx"  
    xmlns:aop="http://www.springframework.org/schema/aop"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans    
       http://www.springframework.org/schema/beans/spring-beans-4.1.xsd    
       http://www.springframework.org/schema/context    
       http://www.springframework.org/schema/context/spring-context-4.1.xsd                  
       http://www.springframework.org/schema/tx   
       http://www.springframework.org/schema/tx/spring-tx-4.1.xsd    
       http://www.springframework.org/schema/aop   
       http://www.springframework.org/schema/aop/spring-aop-4.1.xsd    
       http://www.springframework.org/schema/mvc    
       http://www.springframework.org/schema/mvc/spring-mvc-4.1.xsd">  
       
     <bean id="date" class="java.util.Date"></bean>
      
</beans>
```

#### 在src/test/java下添加测试类SSHTest
``` java
package cn.sakuraffy.ssh.test;

import java.util.Date;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="classpath:applicationContext.xml")
public class SSHTest {

	@Resource
	private Date date;
	
	@Test
	public void testSpring() {
		System.out.println(date);
	}
}

```
Run As Maven test,如果下图所示，有日期打印，则说明Spring搭建成功
{% qnimg freamwork/ssh/p2.png 'class:class1 class2' normal:yes %}

### 整合Spring + Hibernate

#### 在pom.xml中添加Hibernate和数据库相关依赖
``` xml
<!-- ... 不再显示已有内容-->
<properties>
	<!-- Hibernate版本号 -->    
	<hibernate.version>4.1.9.RELEASE</hibernate.version>                                  
</properties>

<dependencies>
	<!-- Spring与Hibernate整合依赖 -->

    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-jdbc</artifactId>  
        <version>${spring.version}</version>  
    </dependency>

    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-orm</artifactId>  
        <version>${spring.version}</version>  
    </dependency>
     
    <dependency>  
      <groupId>org.aspectj</groupId>  
      <artifactId>aspectjweaver</artifactId>  
      <version>1.8.9</version>  
    </dependency>

	<!-- Hibernate依赖 -->

    <dependency>
	    <groupId>org.hibernate</groupId>
	    <artifactId>hibernate-core</artifactId>
	    <version>${hibernate.version}</version>
    </dependency>
	<!-- mysql驱动依赖 -->
    <dependency>
	    <groupId>mysql</groupId>
	    <artifactId>mysql-connector-java</artifactId>
	    <version>5.1.30</version>
    </dependency>
    <!-- 连接池依赖 -->
    <dependency>
	   <groupId>commons-dbcp</groupId>
	   <artifactId>commons-dbcp</artifactId>
	   <version>1.4</version>
    </dependency>
</dependencies>
```
#### 在applicationContext.xml中添加dataSource与sessionFactory
``` xml
<beans>
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">  
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />  
        <property name="url" value="jdbc:mysql://localhost:3306/ssh" />  
        <property name="username" value="root" />  
        <property name="password" value="root" />    
    </bean>  
    
    <bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">  
        <property name="dataSource" ref="dataSource"/> 
        <property name="packagesToScan">
        	<list>
        		<value>cn.sakuraffy.ssh.model</value>
        	</list>
        </property>
        <property name="hibernateProperties">  
            <props>  
                <prop key="show_sql">true</prop>  
            </props>  
        </property>  
    </bean> 
</beans>
```

#### 添加mysql测试数据库及表
``` sql
	CREATE DATABASE ssh;
	USE ssh;
	CREATE TABLE t_user (
		id int PRIMARY KEY AUTO_INCREMENT,
		name VARCHAR(20)
	);
	INSERT INTO t_user VALUES(1,'sakuraffy');
```

#### 添加实体类
``` java
	package cn.sakuraffy.ssh.model;

	import java.io.Serializable;

	import javax.persistence.Entity;
	import javax.persistence.GeneratedValue;
	import javax.persistence.Id;

	@Entity(name="t_user")
	public class User implements Serializable {

		private static final long serialVersionUID = 9213000568883367937L;                    
		
		@Id
		@GeneratedValue
		private int id;
		private String name;
		
		public User() {
		}
		
		public User(String name) {
			super();
			this.name = name;
		}

		public final int getId() {
			return id;
		}
		public final void setId(int id) {
			this.id = id;
		}
		public final String getName() {
			return name;
		}
		public final void setName(String name) {
			this.name = name;
		}

		@Override
		public String toString() {
			return "User [id=" + id + ", name=" + name + "]";
		}
		
	}
```

#### 添加数据持久层接口UserDao及其实现UserDaoImpl
##### UserDao
``` java
package cn.sakuraffy.ssh.dao;

import cn.sakuraffy.ssh.model.User;

public interface UserDao {
	public User getById(int id);
}
```
##### UserDaoImpl
``` java
package cn.sakuraffy.ssh.dao.impl;
import javax.annotation.Resource;
import org.hibernate.SessionFactory;
import org.springframework.stereotype.Repository;

import cn.sakuraffy.ssh.dao.UserDao;
import cn.sakuraffy.ssh.model.User;

@Repository("userDao")
public class UserDaoImpl implements UserDao {
	@Resource
	private SessionFactory sessionFactory;

	@Override
	public User getById(int id) {
		return (User) sessionFactory.getCurrentSession()
				.get(User.class, id);
	}
}
```

#### 添加业务逻辑层接口UserService及其实现UserServiceImpl
##### UserService
``` java
package cn.sakuraffy.ssh.service;

import cn.sakuraffy.ssh.model.User;

public interface UserService {
	public User getById(int id);
}

```

##### UserServiceImpl
``` java
package cn.sakuraffy.ssh.service.impl;

import org.springframework.stereotype.Service;

import javax.annotation.Resource;

import cn.sakuraffy.ssh.dao.UserDao;
import cn.sakuraffy.ssh.model.User;
import cn.sakuraffy.ssh.service.UserService;

@Service("userService")
public class UserServiceImpl implements UserService {

	@Resource
	private UserDao userDao;
	
	@Override
	public User getById(int id) {
		return userDao.getById(id);
	}
}
```

#### 在applicationContext.xml中配置声明式事务管理
``` xml
<beans>
	<bean id="transactionManager"  
	    class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
	    <property name="dataSource" ref="dataSource" />  
	</bean>  

	<tx:advice id="txadvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="add*" propagation="REQUIRED" />
			<tx:method name="save*" propagation="REQUIRED" />
			<tx:method name="update*" propagation="REQUIRED" />
			<tx:method name="delete*" propagation="REQUIRED" />
			<tx:method name="get*" read-only="true" propagation="SUPPORTS" />
			<tx:method name="find*" read-only="true" propagation="SUPPORTS" />
			<tx:method name="*" read-only="true" />
		</tx:attributes>
	</tx:advice> 

	<aop:config>
		<aop:pointcut id="serviceMethods" expression="execution(* cn.sakuraffy.ssh.service.impl.*.*(..))" />
		<aop:advisor advice-ref="txadvice" pointcut-ref="serviceMethods" />
	</aop:config>
</beans>
```

#### 在applicationContext.xml中配置组件扫描
``` xml
<beans>
	<context:component-scan base-package="cn.sakuraffy.ssh" />
</beans>
```

#### 在SSHTest中添加测试方法
``` java
	@Resource
	private UserService userService;

	@Test
	public void testGetById() {
		System.out.println(userService.getById(1));
	}
```
Run As Maven test,如果下图所示，有User打印，则说明Spring与Hibernate整合成功
{% qnimg freamwork/ssh/p3.png 'class:class1 class2' normal:yes %}

### 整合SSH
在pom.xml中加入Struts2与web相关依赖
``` xml
<properties>
	<!-- Struts2版本 -->
  	<struts2.version>2.3.31</struts2.version>
</properties>
<dependencies>
	<!-- servlet依赖 -->
	<dependency>
	    <groupId>javax.servlet</groupId>
	    <artifactId>javax.servlet-api</artifactId>
	    <version>3.1.0</version>
	</dependency>

	<!-- jstl依赖 -->
	<dependency>
   		<groupId>jstl</groupId>
    	<artifactId>jstl</artifactId>
    	<version>1.2</version>
	</dependency>
	<!-- Struts2依赖 -->
	<dependency>
	    <groupId>org.apache.struts</groupId>
	    <artifactId>struts2-core</artifactId>
	    <version>${struts2.version}</version>
	</dependency>
	
	<dependency>
	    <groupId>org.apache.struts</groupId>
	    <artifactId>struts2-spring-plugin</artifactId>                                    
	    <version>${struts2.version}</version>
	</dependency>   
</dependencies>
```

#### 修改web.xml文件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns="http://java.sun.com/xml/ns/javaee" 
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
	http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" version="3.0">
  <display-name>template</display-name>
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>*.action</url-pattern>
  </filter-mapping>
</web-app>
```
#### 添加Action类
``` java
package cn.sakuraffy.ssh.action;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;

import org.apache.struts2.ServletActionContext;

import com.opensymphony.xwork2.ActionSupport;

import cn.sakuraffy.ssh.model.User;
import cn.sakuraffy.ssh.service.UserService;

@Controller("userAction")
public class UserAction extends ActionSupport {

	private static final long serialVersionUID = 6382018041366588184L;                    
	
	@Resource
	private UserService userService;
	
	private int id;
	
	public final int getId() {
		return id;
	}

	public final void setId(int id) {
		this.id = id;
	}

	public String getById() {
		User user = userService.getById(id);
		System.out.println(user);
		HttpServletRequest request = ServletActionContext.getRequest();
		request.setAttribute("user", user);
		return "index";
	}
}
```
#### 添加struts.xml
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
    "http://struts.apache.org/dtds/struts-2.0.dtd">

<struts>
 	<package name="user" extends="struts-default">
        <action name="user_*" class="userAction" method = "{1}">
            <result name="index">/index.jsp</result>
        </action>
    </package>
</struts>
```

#### 在pom.xml中添加jetty容器插件
``` xml
<build>
  	<finalName>ssh</finalName>
    <plugins> 
        <plugin>
           <groupId>org.mortbay.jetty</groupId>
           <artifactId>jetty-maven-plugin</artifactId>
           <version>8.1.16.v20140903</version>
           <configuration>
               <scanIntervalSeconds>60</scanIntervalSeconds>
               <webApp>
                   <contextPath>/ssh</contextPath>
               </webApp>
               <connectors>
                    <connector implementation="org.eclipse.jetty.server.nio.SelectChannelConnector">
                        <port>8080</port>
                    </connector>
                </connectors>
            </configuration>
         </plugin>
    </plugins>
</build>	
```
#### 修改index.jsp
``` html
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <body>
    <a href="user_getById.action?id=1">Hello</a> ${user.name }
  </body>
</html>
```
Run AS Maven Build,输入jetty：run，如图
{% qnimg freamwork/ssh/p4.png 'class:class1 class2' normal:yes %}

打开浏览器，输入 http://localhost:8080/ssh ,点击Hello，得到下图结果，至此SSH整合完毕
{% qnimg freamwork/ssh/p5.png 'class:class1 class2' normal:yes %}