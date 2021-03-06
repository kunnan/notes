**5.SpringBoot 之MyBatis访问MySQL**  

[TOC]

SpringBoot 使用 MyBatis 首先需要添加 pom 依赖。

# pom依赖

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
		
<dependency>
 <groupId>org.mybatis.spring.boot</groupId>
 <artifactId>mybatis-spring-boot-starter</artifactId>
 <version>1.3.0</version>
</dependency>
```

Spring Boot 中集成MyBatis，可以选用基于注解的方式，也可以选择xml文件配置的方式，官方建议使用 XML。下面分别介绍两种方式的使用。

# 注解方式

**1.创建 Student 实体类**

```java
public class Student implements Serializable {

  private static final long serialVersionUID = 6816204218437248771L;

  private String id;
  private String name;
  private String age;
  private String sex;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getAge() {
    return age;
  }

  public void setAge(String age) {
    this.age = age;
  }

  public String getSex() {
    return sex;
  }

  public void setSex(String sex) {
    this.sex = sex;
  }

  public String getId() {
    return id;
  }

  public void setId(String id) {
    this.id = id;
  }
}
```
**2.创建一个Mapper接口类 StudentMapper**

在接口中通过注解定义 CRUD 语句。

```java
@Mapper 
@Component 
public interface StudentMapper {

  @Select("SELECT * FROM STUDENT")
  List<Student> getStudents();

  @Select("select * from student where id=#{id}")
  Student queryById(@Param("id") String id);

  @Insert("insert into student(name,age,sex) values(#{name},#{age},#{sex})")
  void insert(Student stu);

  @Update("update student set name=#{name},age=#{age},sex=#{sex} where id=#{id}")
  void update(Student stu);

  @Delete("delete from student where id=#{id}")
  void delete(@Param("id") String id);

  @DeleteProvider(type = SqlBuilder.class, method = "deleteByids")
  void deleteByIds(@Param("ids") String[] ids);

  class SqlBuilder {
    //删除的方法
    public String deleteByids(@Param("ids") final String[] ids) {
      StringBuffer sql = new StringBuffer();
      sql.append("DELETE FROM STUDENT WHERE id in(");
      for (int i = 0; i < ids.length; i++) {
        if (i == ids.length - 1) {
          sql.append(ids[i]);
        } else {
          sql.append(ids[i]).append(",");
        }
      }
      sql.append(")");
      return sql.toString();
    }
  }
}
```
* 简单的语句使用@Insert、@Update、@Delete、@Select
* 复杂点需要动态SQL语句使用@InsertProvider、@UpdateProvider、@DeleteProvider、@SelectProvider

**3.在启动类中添加 @MapperScan注解**

将创建的 Mapper 接口类路径添加到应用中。

```java
@SpringBootApplication 
@MapperScan("com.liuguoquan.springboot.mapper") 
@EnableConfigurationProperties({ConfigBean.class, ConfigTestBean.class}) 
public class SpringbootApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringbootApplication.class, args);
  }
}
```
**4. Controller使用 Mapper 接口提供服务**

```java
@RequestMapping("api/mybaits") 
@RestController 
public class MyBaitsController {
  @Autowired StudentMapper mStudentMapper;

  @RequestMapping("/get")
  public List<Student> getStudents() {
    return mStudentMapper.getStudents();
  }

  @RequestMapping("/insert")
  public void insert() {
    Student student = new Student();
    student.setName("wang");
    student.setAge("25");
    student.setSex("female");
    mStudentMapper.insert(student);
  }

  @RequestMapping("/update")
  public void update() {
    Student stu = mStudentMapper.queryById("1");
    stu.setAge("30");
    mStudentMapper.update(stu);
  }

  @RequestMapping("/delete")
  public void delete() {
    mStudentMapper.delete("2");
  }

  @RequestMapping("/deleteIds")
  public void deleteIds() {
    mStudentMapper.deleteByIds(new String[] {"4", "5"});
  }
}
```

**最后，在浏览器中调用相应路径如`http://localhost:8080/api/mybaits/get`等进行测试。**

# xml 配置方式

**1. 新建实体类**

```java
public class Student implements Serializable {

  private static final long serialVersionUID = 6816204218437248771L;

  private String id;
  private String name;
  private String age;
  private String sex;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getAge() {
    return age;
  }

  public void setAge(String age) {
    this.age = age;
  }

  public String getSex() {
    return sex;
  }

  public void setSex(String sex) {
    this.sex = sex;
  }

  public String getId() {
    return id;
  }

  public void setId(String id) {
    this.id = id;
  }
}

```

**2. 创建一个Mapper接口类 StudentXMLMapper**

```java
@Component
@Mapper
public interface StudentXMLMapper {

  List<Student> getStudents();

  Student queryById(String id);

  void insert(Student stu);

  void update(Student stu);

  void delete(String id);

  void deleteByIds(String[] ids);
}
```

**3. 启动类上添加 @MapperScan 注解**

```java
@SpringBootApplication
@MapperScan("com.liuguoquan.springboot.mapper")
@EnableConfigurationProperties({ConfigBean.class, ConfigTestBean.class})
public class SpringbootApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringbootApplication.class, args);
  }
}
```

**4. 修改 application.xml 配置文件**

```xml
#指定bean所在包
mybatis.type-aliases-package=com.liuguoquan.springboot.bean
#指定映射文件
mybatis.mapperLocations=classpath:mapper/*.xml
```

**5. 创建建 mapper.xml 文件**

在 src/maim/resources 目录创建一个文件夹 mapper，在mapper 文件夹中创建一个 xml 文件 StudentXMLMapper.xml。

通过mapper标签中的namespace属性指定对应的dao映射，这里指向StudentXMLMapper。**xml 文件中的 id 要与 StudentXMLMapper 类中定义的方法一致**。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.liuguoquan.springboot.mapper.StudentXMLMapper">
  <select id="getStudents" resultType="com.liuguoquan.springboot.bean.Student">
  select * from student
</select>

  <select id="queryById" resultType="com.liuguoquan.springboot.bean.Student">
    select * from student where id=#{id}
  </select>

  <insert id="insert" parameterType="com.liuguoquan.springboot.bean.Student">
    insert into student(name,age,sex) values(#{name},#{age},#{sex})
  </insert>

  <update id="update" parameterType="com.liuguoquan.springboot.bean.Student">
    update student set name=#{name},age=#{age},sex=#{sex} where id=#{id}
  </update>

  <update id="delete" parameterType="com.liuguoquan.springboot.bean.Student">
    delete from student where id=#{id}
  </update>


  <delete id="deleteByIds" parameterType="java.lang.String">
    DELETE FROM student WHERE id in
    <foreach item="id" collection="array" open="(" separator="," close=")">
      #{id}
    </foreach>
  </delete>
</mapper>
```

**6. 启动运行浏览器测试**

# 参考资料
 
 * [mybatis官方中文参考文档 http://www.mybatis.org/mybatis-3/zh/java-api.html](http://www.mybatis.org/mybatis-3/zh/java-api.html)
 * [http://blog.720ui.com/2016/springboot_02_data_mybatis/](http://blog.720ui.com/2016/springboot_02_data_mybatis/)





