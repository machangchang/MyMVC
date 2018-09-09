### Welcome to my simple mvc.:smiley:

我觉得学习一个框架有两个方式，一是直接去一层层剖析源码，二是通过别人的学习总结，滤清思路，有的放矢的去学习，并参考已经研究的部分自己尝试去写个类似的实现。

我个人更喜欢第二种方式。所以这是一个简单的MVC实现的文档。

:japanese_goblin: 首先我们先搭建一个 Maven Web 项目，在 pom.xml 中加入下列依赖：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.machangchang</groupId>
  <artifactId>MyMVC</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>
  
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
		<java.version>1.8</java.version>
	</properties>
	
	<dependencies>
	     <dependency>
  		   <groupId>javax.servlet</groupId> 
		   <artifactId>javax.servlet-api</artifactId> 
		   <version>3.0.1</version> 
		   <scope>provided</scope>
		</dependency>
    </dependencies>
</project>
```

:hear_no_evil: 接着我们的Web.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">
	<servlet>
		<servlet-name>MySpringMVC</servlet-name>
		<servlet-class>com.machangchang.servlet.MyDispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>application.properties</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>MySpringMVC</servlet-name>
		<url-pattern>/*</url-pattern>
	</servlet-mapping>
</web-app>
```



