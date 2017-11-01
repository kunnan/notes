**2.SpringBoot入门之配置文件解析**

[TOC]

Spring Boot 使用了一个全局的配置文件application.properties，放在 src/main/resources 目录下或者类路径的 `/config` 下。Spring Boot 的全局配置文件的作用是对一些默认配置的配置值进行修改。

# 自定义属性

application.properties 提供了自定义属性的支持，我们可以把一些常量配置写在这个文件中：

<!-- more -->

```xml
username="liuguoquan"
slogan="Hello Spring Boot"
//参数间引用
welcome=${username}+" "+${slogan}

```
那么如何在项目中引用到这些配置了，SpringBoot 提供了两种方法，一种是直接绑定，一种提供 Bean 对象绑定。

# 配置属性绑定

## 直接绑定

在你想要绑定的属性是直接使用注解`@Value("${configname}")`。

```java
@RestController
public class UserController {

  @Value("${username}")
  private String name;
  @Value("${slogan}")
  private String slogan;

  @RequestMapping("/")
  public String index() {
    return name + slogan;
  }
}
```
启动工程在浏览器中输入`http://localhost:8080/`就可以打印`liuguoquan Hello Spring Boot`。

## Bean 类绑定

**1.**创建一个 ConfigBean 的类，并给类指定注解`@ConfigurationProperties`

```java
@ConfigurationProperties
public class ConfigBean {
  private String username;
  private String slogan;

  public String getUsername() {
    return username;
  }

  public void setUsername(String username) {
    this.username = username;
  }

  public String getSlogan() {
    return slogan;
  }

  public void setSlogan(String slogan) {
    this.slogan = slogan;
  }
}
```
**2.**在 SpringBoot 入口类加上注解 @EnableConfigurationProperties，并指明要加载的 bean。

```java
@SpringBootApplication
@EnableConfigurationProperties({ConfigBean.class})
public class SpringbootApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringbootApplication.class, args);
  }
}

```
**3.**在 Controller 中通过 @Autowired 注解引入即可。

```java
@RestController
public class UserController {

  @Autowired ConfigBean configBean;

  @RequestMapping("/config")
  public String getConfig() {
    return configBean.getUsername()
        + " : "
        + configBean.getSlogan();
  }
}
```

# 使用自定义配置文件

有时候我们不把所有配置放在 application.properties 里面，我们把配置写在自己创建的文件中，下面我们创建一个配置文件 test.properties

```xml
name="test"
slogan="hello word"
```
然后创建一个 bean 类如下：

```java
@Configuration
@ConfigurationProperties
@PropertySource("classpath:test.properties")
public class ConfigTestBean {
  private String name;
  private String slogan;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getSlogan() {
    return slogan;
  }

  public void setSlogan(String slogan) {
    this.slogan = slogan;
  }
}
```
>@PropertySource 表示指定配置文件的位置。

测试过程如上一节 Bean 类绑定过程所示，这里不再赘述。

# 随机值配置

配置文件中${random} 可以用来生成各种不同类型的随机值

```xml
secret=${random.value}
number=${random.int}
bignumber=${random.long}
uuid=${random.uuid}
```

本节完。


