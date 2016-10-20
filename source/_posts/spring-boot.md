---
title: spring-boot是什么？
date: 2016-10-21 01:35:58
tags:
	- spring-boot
	- spring
categories: java
toc: true
---



# 1. Getting started

## 1.1 Introducing

Spring Boot用于构建卓越的、工业级Spring应用，使用spring和第三方库，以及很少的配置就可完成。

Spring Boot可以使你使用`java -jar`这种启动javase项目的模式来启动应用，或者仍然选择传统的`war`

包的方式。同时，spring官方提供了一个命令行工具`spring scripts`。

spring-boot的官方愿景

* 打造一个彻底的快速的，被广泛接受的上手体验
* 开箱即用 快速开发 （感谢way的翻译）
* 为大型项目提供一些常用的非功能性拓展，比如内嵌的web容器、安全、性能监控、健康检查、额外配置等
* 尽量少的配置文件

## 1.2 环境要求

官方已经发布`2.0.0.BUILD-SNAPSHOT`，目前该版本的Maven资源存在问题无法下载。

2.0需要最低Java 7和spring 5，如果使用6需要特殊配置

http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-use-java-6

Maven 3.2+

Gradle 1.12 or 2.x，不支持3，对2.7和之前的版本是过期状态，可以看出对Gradle支持不是很好

当然，官方很建议使用java 8来上手。

Servlet 3.0+	个人建议

<!--more-->

## 1.3 一个spring boot实例

### 1.3.1 Maven配置项

spring-boot提供了一系列的starter，名称类似`spring-boot-*.jar`，具体下面会讲。

官方建议使用构建工具譬如Maven来管理项目，首先确认Java和maven环境

下面是一个典型的`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <!-- Inherit defaults from Spring Boot -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.0.BUILD-SNAPSHOT</version>
    </parent>

    <!-- Add typical dependencies for a web application -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <!-- Package as an executable jar -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <!-- Add Spring repositories -->
    <!-- (you don't need this if you are using a .RELEASE version) -->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
            <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```

上面通过`spring-boot-starter-parent`来使用spring-boot，但是在项目中，业务模块往往需要继承自己的parent模块，这种方式就不可取了，因此，官方提供了另外的方式

通过`dependencyManagement`来使用spring-boot

```xml
<dependencyManagement>
     <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.0.0.BUILD-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

上面的做法中并不能覆盖spring-boot提供的依赖版本，如果想要覆盖个别依赖版本，需要下面这样的额外配置

```xml
<dependencyManagement>
    <dependencies>
        <!-- Override Spring Data release train provided by Spring Boot -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-releasetrain</artifactId>
            <version>Fowler-SR2</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.0.0.BUILD-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 1.3.2 使用starter 

spring-boot提供了一系列starters，可以兼容很多开发环境，我们已经使用过`spring-boot-starter-parent`，这是一个maven默认的指定的starter，其它不同的starters分别服务于为不同的开发环境，譬如开发一个web应用可以使用`spring-boot-starter-web`。

在开始之前可以使用`mvn dependency:tree`来观察项目结构，可以看到starter-parent没有提供任何依赖。

现在加入starter-web

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

之后再使用tree来看，会发现多了很多依赖。

```
[INFO] com.xx.fun:hbase-client-test:jar:1.0-SNAPSHOT
[INFO] \- org.springframework.boot:spring-boot-starter-web:jar:1.3.1.RELEASE:compile
[INFO]    +- org.springframework.boot:spring-boot-starter:jar:1.3.1.RELEASE:compile
[INFO]    |  +- org.springframework.boot:spring-boot:jar:1.3.1.RELEASE:compile
[INFO]    |  +- org.springframework.boot:spring-boot-autoconfigure:jar:1.3.1.RELEASE:compile
[INFO]    |  +- org.springframework.boot:spring-boot-starter-logging:jar:1.3.1.RELEASE:compile
[INFO]    |  |  +- ch.qos.logback:logback-classic:jar:1.1.3:compile
[INFO]    |  |  |  +- ch.qos.logback:logback-core:jar:1.1.3:compile
[INFO]    |  |  |  \- org.slf4j:slf4j-api:jar:1.7.13:compile
[INFO]    |  |  +- org.slf4j:jcl-over-slf4j:jar:1.7.13:compile
[INFO]    |  |  +- org.slf4j:jul-to-slf4j:jar:1.7.13:compile
[INFO]    |  |  \- org.slf4j:log4j-over-slf4j:jar:1.7.13:compile
[INFO]    |  +- org.springframework:spring-core:jar:4.2.4.RELEASE:compile
[INFO]    |  \- org.yaml:snakeyaml:jar:1.16:runtime
[INFO]    +- org.springframework.boot:spring-boot-starter-tomcat:jar:1.3.1.RELEASE:compile
[INFO]    |  +- org.apache.tomcat.embed:tomcat-embed-core:jar:8.0.30:compile
[INFO]    |  +- org.apache.tomcat.embed:tomcat-embed-el:jar:8.0.30:compile
[INFO]    |  +- org.apache.tomcat.embed:tomcat-embed-logging-juli:jar:8.0.30:compile
[INFO]    |  \- org.apache.tomcat.embed:tomcat-embed-websocket:jar:8.0.30:compile
[INFO]    +- org.springframework.boot:spring-boot-starter-validation:jar:1.3.1.RELEASE:compile
[INFO]    |  \- org.hibernate:hibernate-validator:jar:5.2.2.Final:compile
[INFO]    |     +- javax.validation:validation-api:jar:1.1.0.Final:compile
[INFO]    |     +- org.jboss.logging:jboss-logging:jar:3.3.0.Final:compile
[INFO]    |     \- com.fasterxml:classmate:jar:1.1.0:compile
[INFO]    +- com.fasterxml.jackson.core:jackson-databind:jar:2.6.4:compile
[INFO]    |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.6.4:compile
[INFO]    |  \- com.fasterxml.jackson.core:jackson-core:jar:2.6.4:compile
[INFO]    +- org.springframework:spring-web:jar:4.2.4.RELEASE:compile
[INFO]    |  +- org.springframework:spring-aop:jar:4.2.4.RELEASE:compile
[INFO]    |  |  \- aopalliance:aopalliance:jar:1.0:compile
[INFO]    |  +- org.springframework:spring-beans:jar:4.2.4.RELEASE:compile
[INFO]    |  \- org.springframework:spring-context:jar:4.2.4.RELEASE:compile
[INFO]    \- org.springframework:spring-webmvc:jar:4.2.4.RELEASE:compile
[INFO]       \- org.springframework:spring-expression:jar:4.2.4.RELEASE:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
```

### 1.3.3 编写代码

编写一个Java类，作为入口

```java
@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Example.class, args);
    }

}
```

`@RestController`是一个约定的写法，告诉人们这个类扮演的角色，spring会把这个类当成一个处理请求的web controller。

`@RequestMapping`提供一个路由信息，告诉spring路径为`/`的请求会被该方法处理。

这两个都是spring mvc的注解。

`@EnableAutoConfiguration`告诉spring怎样配置应用，starter-web加入了tomcat和spring mvc的依赖，主要是针对这两个的配置项。

> 自动化配置的设计在starter中运行良好，但是并不是说必须选择自动化配置，用户仍然可以使用传统方式配置指定的依赖包

在main方法中委托spring boot的`SpringApplication`类通过run来启动应用。

启动成功后可以通过tomcat默认端口来访问。

## 1.4 创建一个可执行的jar包

在上面我们已经创建了一个spring-boot项目，并且运行成功，下面通过Maven plugin来将应用打成jar包。这个独立的jar包应该包含所有运行环境需要的依赖包。

java原生并不提供执行一个嵌套的jar包，为了解决这个问题

> Über-jar，Über来自于德语above或者over，一个Über-jar是指一个jar包内嵌了所有的依赖包。

在dependences下面增加plugin节点

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

starter-parent中的pom已经定义了`<executions>`的配置，如果不是使用starter-parent的话需要自己额外配置！

执行`mvn package`会在target生成jar包，执行

```shell
$ jar tvf target/xxx-0.0.1-SNAPSHOT.jar		//查看jar内部  个人推荐jd-gui来看
```

同时target下面也会有一个小点的jar包，`xxx-0.0.1-SNAPSHOT.jar.original`这个是maven生成的不包含lib的普通jar包。



