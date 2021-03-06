**1.SpringBoot入门之Hello SpringBoot**

[TOC]

Spring Boot专注于Spring平台和第三方开发库，简化了基于Spring的产品开发。让你在开始Spring应用开发时，不会被各种繁杂的事务困扰。通过Spring Boot，开发时只需少量的Spring配置就可以完成项目结构的初始搭建。

<!-- more -->

Spring Boot提供了很多”开箱即用“的依赖模块，都是以spring-boot-starter-xx作为命名的。下面列举一些常用的模块。

* spring-boot-starter-logging ：使用 Spring Boot 默认的日志框架 Logback。
* spring-boot-starter-log4j ：添加 Log4j 的支持。
* spring-boot-starter-web ：支持 Web 应用开发，包含 Tomcat 和 spring-mvc。
* spring-boot-starter-tomcat ：使用 Spring Boot 默认的 Tomcat 作为应用服务器。
* spring-boot-starter-jetty ：使用 Jetty 而不是默认的 Tomcat 作为应用服务器。
* spring-boot-starter-test ：包含常用的测试所需的依赖，如 JUnit、Hamcrest、Mockito 和 spring-test 等。
* spring-boot-starter-aop ：包含 spring-aop 和 AspectJ 来支持面向切面编程（AOP）。
* spring-boot-starter-security ：包含 spring-security。
* spring-boot-starter-jdbc ：支持使用 JDBC 访问数据库。
* spring-boot-starter-redis ：支持使用 Redis。
* spring-boot-starter-data-mongodb ：包含 spring-data-mongodb 来支持 MongoDB。
* spring-boot-starter-data-jpa ：包含 spring-data-jpa、spring-orm 和 Hibernate 来支持 JPA。
* spring-boot-starter-amqp ：通过 spring-rabbit 支持 AMQP。
* spring-boot-starter-actuator ： 添加适用于生产环境的功能，如性能指标和监测等功能。

# 开发环境

* 操作系统： macOS High Sierra 10.13
* IDE：IntelliJ IDEA 2017.2
* JDK：JDK8
* SpringBoot版本：1.5.8.RELEASE
* 构建工具：Maven

# IntelliJ IDEA创建Spring Boot工程

IntelliJ IDEA是非常流行的IDE，IntelliJ IDEA 14.1已经支持Spring Boot了。
IntelliJ IDEA创建Spring Boot操作步骤如下：

**1. 在File菜单里面选择 New > Project,然后选择Spring Initializr，接着如下图一步步操作即可。**

![](http://7xqch5.com1.z0.glb.clouddn.com/springboot1-2.png)

![](http://7xqch5.com1.z0.glb.clouddn.com/springboot1-3.png)

![](http://7xqch5.com1.z0.glb.clouddn.com/springboot1-4.png)

![](http://7xqch5.com1.z0.glb.clouddn.com/springboot1-5.png)

# Spring Boot工程目录

根据上面的流程就已经创建了一个基本的SpringBoot的开发框架了，如下图所示：

![springboot_project](http://onke0yoit.bkt.clouddn.com/springboot_project.png)

* pom.xml：Maven构建配置文件
* test目录：测试工程
* main目录：开发工程
* main/java目录下的SpringbootApplication.java文件中的main()方法用于启动应用程序
* main/resources目录下的application.properties文件用于配置工程的属性

# 创建REST API服务

SpringbootApplication类是应用的启动类，代码如下：

```java
@RestController
@SpringBootApplication
public class SpringbootApplication {

  @RequestMapping("/index")
  public String index() {
    return "Hello Spring Boot";
  }

  public static void main(String[] args) {
    SpringApplication.run(SpringbootApplication.class, args);
  }
}
```
* @SpringBootApplication是Spring Boot的核心注解，主要目的是开启自动配置。
* main()方法是启动程序的入口。
* @RestController注解表示当前类为REST工程
* @RequestMapping注解标注REST API路径，所以index()方法的外部调用路径为`http://localhost:8080/index/`

# 启动程序

点击“Run”启动SpringBoot项目，

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.2.4.RELEASE)

2015-08-25 22:53:35.484  INFO 554 --- [           main] com.lkl.springboot.Application       : Starting Application on mac.local with PID 554 (/Users/liaokailin/code/github/blog-springboot/target/classes started by lkl in /Users/liaokailin/code/github/blog-springboot)
2015-08-25 22:53:35.561  INFO 554 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@328908cd: startup date [Tue Aug 25 22:53:35 CST 2015]; root of context hierarchy
2015-08-25 22:53:36.395  INFO 554 --- [           main] o.s.b.f.s.DefaultListableBeanFactory     : Overriding bean definition for bean 'beanNameViewResolver': replacing [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration; factoryMethodName=beanNameViewResolver; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/boot/autoconfigure/web/ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration.class]] with [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter; factoryMethodName=beanNameViewResolver; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter.class]]
2015-08-25 22:53:37.404  INFO 554 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
2015-08-25 22:53:37.796  INFO 554 --- [           main] o.apache.catalina.core.StandardService   : Starting service Tomcat
2015-08-25 22:53:37.798  INFO 554 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.0.23
2015-08-25 22:53:37.955  INFO 554 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2015-08-25 22:53:37.955  INFO 554 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2397 ms
2015-08-25 22:53:38.813  INFO 554 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2015-08-25 22:53:38.818  INFO 554 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'characterEncodingFilter' to: [/*]
2015-08-25 22:53:38.819  INFO 554 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2015-08-25 22:53:39.075  INFO 554 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@328908cd: startup date [Tue Aug 25 22:53:35 CST 2015]; root of context hierarchy
2015-08-25 22:53:39.151  INFO 554 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],methods=[],params=[],headers=[],consumes=[],produces=[],custom=[]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2015-08-25 22:53:39.151  INFO 554 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],methods=[],params=[],headers=[],consumes=[],produces=[text/html],custom=[]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest)
2015-08-25 22:53:39.180  INFO 554 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2015-08-25 22:53:39.180  INFO 554 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2015-08-25 22:53:39.232  INFO 554 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2015-08-25 22:53:39.326  INFO 554 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2015-08-25 22:53:39.430  INFO 554 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2015-08-25 22:53:39.433  INFO 554 --- [           main] com.lkl.springboot.Application       : Started Application in 4.829 seconds (JVM running for 5.238)
```

spring boot已经启动，内嵌tomcat容器，监听为8080端口，在浏览器中输入`http://localhost:8080/index/`，将会在浏览器上打印Hello Spring Boot。

