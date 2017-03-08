---
title: maven搭建SSM
date: 2016-09-27 14:56:29
tags: 
	- freamwork
---
在实际开发中，我们习惯将项目分为三层，展示层，业务逻辑层，数据持久层。对于展现层我们一般会用到Strust2/Spring MVC，而数据层我们往往会用到Hibernate/Mybatis，这里我将介绍一下SSM(Spring + Spring MVC + Mybatis)的搭建，至于SSH的搭建请参考[maven搭建SSH](https://sakuraffy.github.io/freamwork_ssh)

<!-- more -->

首先创建一个maven项目，这里不再赘述了，如果不太清楚，请参考[maven项目搭建](https://sakuraffy.github.io/maven_project_create)
{% qnimg freamwork/ssm/p1.png 'class:class1 class2' normal:yes %}

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

#### 在src/test/java下添加测试类SSMTest
``` java
package cn.sakuraffy.ssm.test;

import java.util.Date;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="classpath:applicationContext.xml")
public class SSMTest {

	@Resource
	private Date date;
	
	@Test
	public void testSpring() {                                                            
		System.out.println(date);
	}

	@Resource
	private Date date;
	
	@Test
	public void testSpring() {
		System.out.println(date);
	}
}

```
Run As Maven test,如果下图所示，有日期打印，则说明Spring搭建成功
{% qnimg freamwork/ssm/p2.png 'class:class1 class2' normal:yes %} 

### 整合Spring+Mybatis
#### 在pom.xml中添加Mybatis和数据库相关依赖
``` xml
<!-- ... 不再显示已有内容-->
<properties>
	 <!-- mybatis版本号 -->  
     <mybatis.version>3.2.6</mybatis.version>
</properties>

<dependencies>
	<!-- Spring与Mybatis整合依赖 -->

    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-jdbc</artifactId>  
        <version>${spring.version}</version>  
    </dependency>
     
    <dependency>  
      <groupId>org.aspectj</groupId>  
      <artifactId>aspectjweaver</artifactId>                                                  
      <version>1.8.9</version>  
    </dependency>

	<!-- mybatis核心包 -->  
    <dependency>  
        <groupId>org.mybatis</groupId>  
        <artifactId>mybatis</artifactId>  
        <version>${mybatis.version}</version>  
    </dependency>  
    <!-- mybatis/spring包 -->  
    <dependency>  
        <groupId>org.mybatis</groupId>  
        <artifactId>mybatis-spring</artifactId>  
        <version>1.2.2</version>  
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

#### 添加mysql测试数据库及表
``` sql
	CREATE DATABASE ssm;
	USE ssm;
	CREATE TABLE t_user (
		id int PRIMARY KEY AUTO_INCREMENT,
		name VARCHAR(20)
	);
	INSERT INTO t_user VALUES(1,'sakuraffy');
```

#### 添加实体类
``` java
	package cn.sakuraffy.ssm.model;

	import java.io.Serializable;

	public class User implements Serializable {

		private static final long serialVersionUID = 9213000568883367937L;             	      
		
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

#### 添加数据持久层接口UserDao及UserMapper.xml
##### UserDao
``` java
package cn.sakuraffy.ssm.dao;

import cn.sakuraffy.ssm.model.User;

public interface UserDao {
	public User getById(int id);
}
```
##### UserMapper.xml
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.sakuraffy.ssm.dao.UserDao">
	<select id="getById" parameterType="int" resultType="User">
		SELECT * FROM t_user WHERE id = #{id}
	</select>
</mapper>
```

#### 添加业务逻辑层接口UserService及其实现UserServiceImpl
##### UserService
``` java
package cn.sakuraffy.ssm.service;

import cn.sakuraffy.ssm.model.User;

public interface UserService {
	public User getById(int id);
}

```

##### UserServiceImpl
``` java
package cn.sakuraffy.ssm.service.impl;

import javax.annotation.Resource;

import org.springframework.stereotype.Service;

import cn.sakuraffy.ssm.dao.UserDao;
import cn.sakuraffy.ssm.model.User;
import cn.sakuraffy.ssm.service.UserService;

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

#### 在applicationContext.xml中添加dataSource与sqlSessionFactory
``` xml
<beans>
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">  
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />  
        <property name="url" value="jdbc:mysql://localhost:3306/ssm" />  
        <property name="username" value="root" />  
        <property name="password" value="root" />    
    </bean>  
     
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
        <property name="dataSource" ref="dataSource" />
        <!-- 为实体配置别名 -->
        <property name="typeAliasesPackage" value="cn.sakuraffy.ssm.model" /> 
        <!-- 自动扫描mapping.xml文件 -->  
        <property name="mapperLocations" value="classpath:cn/sakuraffy/ssm/mapper/*.xml" />  
    </bean> 

    <!-- DAO接口所在包名，Spring会自动查找其下的类 -->  
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">  
        <property name="basePackage" value="cn.sakuraffy.ssm.dao" />  
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />  
    </bean> 
</beans>
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
		<aop:pointcut id="serviceMethods" expression="execution(* cn.sakuraffy.ssm.service.impl.*.*(..))" />
		<aop:advisor advice-ref="txadvice" pointcut-ref="serviceMethods" />
	</aop:config>
</beans>
```

#### 在applicationContext.xml中配置组件扫描
``` xml
<beans>
	<context:component-scan base-package="cn.sakuraffy.ssm" />
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
Run As Maven test,如果下图所示，有User打印，则说明Spring与Mybatis整合成功
{% qnimg freamwork/ssm/p4.png 'class:class1 class2' normal:yes %}
如果不幸抛出下图错误，在pom.xml中加入资源处理
{% qnimg freamwork/ssm/p3.png 'class:class1 class2' normal:yes %}
``` xml
<build>
  <resources>
    <resource>
  	  <directory>src/main/java</directory>
   	  <includes>
  	    <include>**/*.xml</include>
 	  </includes>
    </resource>
  </resources>
</build>
```

### 整合SSM
在pom.xml中加入Spring与web相关依赖
``` xml
<dependencies>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-web</artifactId>
		<version>${spring.version}</version>
	</dependency>
	
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-webmvc</artifactId>
		<version>${spring.version}</version>
	</dependency>
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
</dependencies>
```

#### 修改web.xml文件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns="http://java.sun.com/xml/ns/javaee" 
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
	http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" version="3.0">
  <display-name>ssm</display-name>
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
  
  <servlet>
  	<servlet-name>SpringMVC</servlet-name>
  	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  	<init-param>
  		<param-name>contextConfigLocation</param-name>
    	<param-value>classpath:spring-mvc.xml</param-value>
  	</init-param>
  	<load-on-startup>1</load-on-startup>
  </servlet>
  
  <servlet-mapping>
  	<servlet-name>SpringMVC</servlet-name>
  	<url-pattern>*.do</url-pattern>
  </servlet-mapping>
  
</web-app>
```

#### 添加spring-mvc.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:p="http://www.springframework.org/schema/p"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:mvc="http://www.springframework.org/schema/mvc"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans    
      http://www.springframework.org/schema/beans/spring-beans-3.1.xsd    
      http://www.springframework.org/schema/context    
      http://www.springframework.org/schema/context/spring-context-3.1.xsd    
      http://www.springframework.org/schema/mvc    
      http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">  
    
    <context:component-scan base-package="cn.sakuraffy.ssm.controller" />

    <!-- 配置视图渲染器(也可以不配)-->  
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">          
        <property name="prefix" value="/" />  
        <property name="suffix" value=".jsp" />  
    </bean> 
    
</beans>
```

#### 添加Controller类
``` java
package cn.sakuraffy.ssm.controller;

import javax.annotation.Resource;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;                               

import cn.sakuraffy.ssm.model.User;
import cn.sakuraffy.ssm.service.UserService;

@Controller("userController")
public class UserController {
	
	@Resource
	private UserService userService;
	
	@RequestMapping("/getById")
	public String getById(int id, ModelMap model) {
		User user = userService.getById(id);
		System.out.println(user);
		model.addAttribute("user", user);
		return "index";
	}
}
```

#### 在pom.xml中添加jetty容器插件
``` xml
<build>
  	<finalName>ssm</finalName>
    <plugins> 
        <plugin>
           <groupId>org.mortbay.jetty</groupId>
           <artifactId>jetty-maven-plugin</artifactId>
           <version>8.1.16.v20140903</version>
           <configuration>
               <scanIntervalSeconds>60</scanIntervalSeconds>
               <webApp>
                   <contextPath>/ssm</contextPath>
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
{% qnimg freamwork/ssm/p5.png 'class:class1 class2' normal:yes %}

打开浏览器，输入 http://localhost:8080/ssh ,点击Hello，得到下图结果，至此SSM整合完毕
{% qnimg freamwork/ssm/p6.png 'class:class1 class2' normal:yes %}
