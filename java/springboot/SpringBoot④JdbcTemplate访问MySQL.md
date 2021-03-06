**4.SpringBoot之 JdbcTemplate 访问MySQL**

[TOC]

SpringBoot 的 JdbcTemplate 是自动配置的，你可以直接使用@Autowired 来注入到你自己的类中来使用。

**1.**首先创建一个 Student 类

这里 Student 中的属性名称与数据库表中的字段名称是一样的：

```java
public class Student implements Serializable {

  private static final long serialVersionUID = 6816204218437248771L;

  private int id;
  private String name;
  private int age;
  private String sex;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }

  public String getSex() {
    return sex;
  }

  public void setSex(String sex) {
    this.sex = sex;
  }

  public int getId() {
    return id;
  }

  public void setId(int id) {
    this.id = id;
  }
}
```
**2.**创建一个 Service 类 StudentService，并加上注解 @Service

直接使用 @Autowired 注解将 JdbcTemplate 注入到 StudentService 类中使用。

```java
@Service
public class StudentService {

  @Autowired
  private JdbcTemplate jdbcTemplate;

 //查询数据库数据
  public List<Student> getStudents() {
    String sql = "SELECT ID,NAME,AGE,SEX FROM STUDENT";
    return jdbcTemplate.query(sql, new RowMapper<Student>() {
      @Override public Student mapRow(ResultSet resultSet, int i) throws SQLException {
        Student student = new Student();
        student.setId(resultSet.getInt("id"));
        student.setName(resultSet.getString("name"));
        student.setAge(resultSet.getInt("age"));
        student.setSex(resultSet.getString("sex"));
        return student;
      }
    });
  }

//向数据库中插入值
public void insert(Student student) {
    jdbcTemplate.update("insert into student(name,age,sex) values(?,?,?)", student.getName(),
        student.getAge(), student.getSex());
  }
}
```
**3.**最后在 Controller 调用 StudentService 方法。

```java
@RestController
public class UserController {

  @Autowired StudentService mStudentService;

  @RequestMapping("/get")
  public List<Student> getList() {
    return mStudentService.getStudents();
  }
  
@RequestMapping("/insert")
  public void insert() {
    Student student = new Student();
    student.setName("zhou");
    student.setAge(25);
    student.setSex("female");
    mStudentService.insert(student);
  }
}
```

**最后，**启动工程在浏览器中调用`http://localhost:8080/get`，可打印：

```json
[
    {
        "id": 1,
        "name": "liu",
        "age": 28,
        "sex": "male"
    },
    {
        "id": 2,
        "name": "zhang",
        "age": 22,
        "sex": "male"
    },
    {
        "id": 3,
        "name": "lee",
        "age": 25,
        "sex": "male"
    },
    {
        "id": 4,
        "name": "xu",
        "age": 23,
        "sex": "female"
    }
]
```

