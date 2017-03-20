# 快速开始

> 一下步骤均为本人在MAC系统环境下进行操作

### 基础环境
* Java运行环境：`java version "1.8.0_92"`
* Rose开发包
* Tomcat、Resin或其他WEB容器
* Java编辑器IDE，如Eclipse `Version: Neon Release (4.6.0)`
* Maven: `Apache Maven 3.3.9`

### Maven简介
Maven是基于项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。如果你已经有十次输入同样的Ant targets来编译你的代码、jar或者war、生成javadocs，你一定会自问，是否有一个重复性更少却能同样完成该工作的方法。Maven便提供了这样一种选择，将你的注意力从作业层转移到项目管理层。Maven项目已经能够知道如何构建和捆绑代码，运行测试，生成文档并宿主项目网页。

Maven对一个项目定义了固定的默认目录定义：
* src/main/java 写主要的java实现
* src/main/resources 写主要的配置文件
* src/test/java 写test case
* src/test/resources 写test case所需要的配置文件
* src/main/webapp [war项目特有]web项目的对外目录
* src/main/webapp/WEB-INF [war项目特有]web项目配置web.xml目录

### 创建项目
* eclipse -> new -> other -> maven -> maven project -> next
* create a simple project -> next
* Group Id: com.xiaomi
* Artifact Id: rose-example
* Packaging: war
* Finish

### 配置 POM文件
打开上一步建立好的项目，打开pom.xml，添加下面的段落到project中：

```xml
<dependencies>
  	<dependency>
  		<groupId>net.paoding</groupId>
  		<artifactId>paoding-rose</artifactId>
  		<version>1.1.1</version>
  	</dependency>
  	<dependency>
  		<groupId>net.paoding</groupId>
  		<artifactId>paoding-rose-jade</artifactId>
  		<version>1.1.1</version>
  	</dependency>
  	<dependency>
  		<groupId>net.paoding</groupId>
  		<artifactId>paoding-rose-portal</artifactId>
  		<version>1.1.1</version>
  	</dependency>
  	<dependency>
  		<groupId>net.paoding</groupId>
  		<artifactId>paoding-rose-scanning</artifactId>
  		<version>1.1.1</version>
  	</dependency>
</dependencies>
```

上述是rose环境最基础的依赖包。再添加一点常见的编译设置：

```xml
<build>
	<testSourceDirectory>src/main/test</testSourceDirectory>
	<resources>
		<resource>
			<directory>src/main/resources</directory>
			<filtering>true</filtering>
		</resource>
    </resources>
    <plugins>
    	<!-- 编译使用1.6 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.5.1</version>
            <configuration>
                <source>1.6</source>
                <target>1.6</target>
                <fork>true</fork>
                <verbose>true</verbose>
                <encoding>UTF-8</encoding>
                <compilerArguments>
                   war
                	<sourcepath>
                        src/main/java
                    </sourcepath>
                </compilerArguments>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>2.0.2</version>
           	<configuration>
                <webResources>
                    <resource>
                        <filtering>true</filtering>
                        <directory>src/main/webapp/WEB-INF
                        </directory>
                        <includes>
                            <include>**/*.txt</include>
                            <include>**/*.xml</include>
                            <include>**/*.properties</include>
                        </includes>
                        <excludes>
                            <exclude>/static</exclude>
                        </excludes>
                        <targetPath>WEB-INF</targetPath>
                    </resource>
                </webResources>
            </configuration>
        </plugin>
        <plugin>
        	<groupId>org.apache.maven.plugins</groupId>
        	<artifactId>maven-jar-plugin</artifactId>
        	<version>2.5.1</version>
        	<configuration>
        		<archive>
        		<manifestEntries>
        			<Rose>*</Rose>
        		</manifestEntries>
                </archive>
            </configuration>
        </plugin>
    	<plugin>
    		<groupId>org.mortbay.jetty</groupId>
    		<artifactId>jetty-maven-plugin</artifactId>
    		<version>8.1.3.v20120416</version>
    		<configuration>
    			<reload>manual</reload>
    			<war>target/rose-example-0.0.1-SNAPSHOT.war</war>
    			<webAppSourceDirectory>target/rose-example-0.0.1-SNAPSHOT</webAppSourceDirectory>
    			<contextXml>src/main/resources/jetty-context.xml</contextXml>
    			<connectors>
    				<connector 
    					implementation="org.eclipse.jetty.server.nio.SelectChannelConnector">
                        <port>8080</port>
                    </connector>
                </connectors>
                <webAppConfig>
                    <allowDuplicateFragmentNames>true</allowDuplicateFragmentNames>
                </webAppConfig>
            </configuration>
        </plugin>
    </plugins>
</build>
```
### 配置 web.xml 
在src/main/webapp/WEB-INF文件夹下建立web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         id="WebApp_ID" version="2.5">
    <display-name>rose-example</display-name>
    <context-param>
        <param-name>log4jConfigLocation</param-name>
        <param-value>/WEB-INF/log4j.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
    </listener>
    <filter>
        <filter-name>roseFilter</filter-name>
        <filter-class>net.paoding.rose.RoseFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>roseFilter</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>REQUEST</dispatcher>
        <dispatcher>FORWARD</dispatcher>
        <dispatcher>INCLUDE</dispatcher>
    </filter-mapping>
</web-app>
```

### 配置applicationContext.xml 
`src/main/resources/applicationContext.xml`, 所有包的扫描和启动都在这里定义：

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans    
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd    
    http://www.springframework.org/schema/context    
    http://www.springframework.org/schema/context/spring-context-2.5.xsd"  
    default-lazy-init="true">  
  
    <!-- 自动扫描 -->  
    <context:annotation-config />  
    <context:component-scan base-package="com.xiaomi">  
    </context:component-scan>  
       
</beans>
```

### 创建Hello World控制器
* 在src/main/java上右键 -> new -> package -> name: com.xiaomi -> finish
* 在com.xiaomi上右键 -> new -> package -> com.xiaomi.controllers -> finish[controllers是Rose框架默认的加载controller的package name]
* 在com.xiaomi.controllers上右键 -> new -> class -> HelloController -> finish [*Controller是rose框架默认的controller层的class后缀]
* 在public class HelloController添加注解@Path("") [Path注解是rose框架提供的标识每个controller的对外访问时的基础路径]
* 在HelloController中添加方法

```java
package com.xiaomi.controllers;

import net.paoding.rose.web.annotation.Path;
import net.paoding.rose.web.annotation.rest.Get;

@Path("") 
public class HelloController {
  
  @Get("")
  public String index() {
    return "@Hello World!";
  }
}
```

* Get注解是rose框架提供的标识一个http访问是get还是post或者是其他，并且会将path与get中的字符串连接成一个url
* 上述代码可以从浏览器访问：http://localhost/
* 下述代码可以从浏览器访问：http://localhost/hello/world [注意path与get中的参数]

```java
package com.xiaomi.controllers;

import net.paoding.rose.web.annotation.Path;
import net.paoding.rose.web.annotation.rest.Get;

@Path("hello") 
public class HelloController {
  
  @Get("world")
  public String index() {
    return "@Hello World!";
  }
}
```

### 参考
* [rose手册第二章：配置与使用](http://www.54chen.com/java-ee/rose-manual-2.html)