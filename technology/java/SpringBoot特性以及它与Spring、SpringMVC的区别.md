# 前言
> Java开发人员几乎都使用过Spring，Spring为Java程序提供了非常实用的功能，如：Spring JDBC
Spring MVC、Spring Security、Spring AOP、Spring ORM、Spring Test等，这些开发模块的出现，不仅缩短了应用程序的开发时间，而且提高了应用开发的效率。而现在SpringBoot往往更受欢迎，本文主要介绍一下SpringBoot的特性，以及它与Spring、SpringMVC的区别。
# SpringBoot的特性和Spring的区别
Spring Boot本质是Spring框架的扩展和延伸，它消除了设置Spring应用程序所需的复杂例行配置，提供了更快、更高效的开发生态系统。SpringBoot特性如下：
## 提供了更快的构建能力
Spring Boot提供了很多Starters用于快速构建业务框架以及和中间件连接，如spring-boot-starter、spring-boot-starter-web、spring-boot-starter-amqp等。它可以一站式集成Spring以及其他的技术，而不需要用户到处需要依赖包。在这里，我们可以将它提供的starter看成一个启动器。

在Spring中我们创建Web应用程序的最小依赖是：
```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>xxxE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>xxx</version>
</dependency>
```
而在Spring Boot中只需要一个依赖项就可以启动和运行Web应用程序，如下：
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
同理，我们在做单元测试的时候，不用再引入Spring中其他许多依赖包了，而只用引入，spring-boot-starter-test即可。
除此之外，Spring Boot还提供了其他许多Starter，如：

 - spring-boot-starter-test
 - spring-boot-starter-web
 - spring-boot-starter-thymeleaf
 - spring-boot-starter-security
 - spring-boot-starter-data-jpa

## 内嵌容器支持
Spring Boot内嵌了Tomcat，当然除了Tomcat之外，它还有Jetty、Undertow三种容器。默认启动的是Tomcat，这个可以进行修改。比如通过修改pom.xm就可以切换容器，我们修改完之后，可以在后台控制台上看到。配置如下：
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- 移除 Tomcat -->
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- 加入 jetty 容器依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```
##  安全监控机制
Spring Boot提供了Actuator监控功能，用于服务监控，如监控应用程序运行状况，服务接口是否通。内存、线程池、Http请求统计等，它提供了19个接口，接口的请求方式以及含义如下：

# SpringBoot与SpringMVC的区别

# 小结