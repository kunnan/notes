**3.SpringBoot之 MySql 配置**

[TOC]

SpringBoot 接入 mysql 数据库，需要首先添加 spring-boot-starter-jdbc 依赖和 mysql 依赖，spring-boot-starter-jdbc 默认使用 tomcat-jdbc 数据源，其次需要
在 application.properties 中写入数据库配置。

# pom.xml

首先在 pom.xml 文件中添加 mysql 依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
 <groupId>mysql</groupId>
 <artifactId>mysql-connector-java</artifactId>
 <version>5.1.21</version>
</dependency>
```

# application.properties

其次，在 application.properties 文件中配置 datasource 信息。

```xml
#################
###datasource
#################
spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8
spring.datasource.username=root
spring.datasource.password=2436363
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
pring.datasource.max-active=20
spring.datasource.max-idle=8
spring.datasource.min-idle=8
spring.datasource.initial-size=10
```

* spring.datasource.url：指定JDBC URL
* spring.datasource.driver-class-name：指定driver的类名
* spring.datasource.username：指定数据库用户名
* spring.datasource.password：指定数据库密码
* spring.datasource.max-active：指定连接池中最大的活跃连接数
* spring.datasource.max-idle：指定连接池最大的空闲连接数量
* spring.datasource.min-idle：指定必须保持连接的最小值
* spring.datasource.initial-size：指定启动连接池时，初始建立的连接数量


# 参考资料

* [SpringBoot配置属性之DataSource](https://segmentfault.com/a/1190000004316491)
* [Spring Boot 揭秘与实战（二） http://blog.720ui.com/2016/springboot_02_data_jdbc/](http://blog.720ui.com/2016/springboot_02_data_jdbc/)

