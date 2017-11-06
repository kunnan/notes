**6.SpringBoot之GET和POST请求方式**

[TOC]

开发中常用的 HTTP 请求方式是 GET 和 POST，下面介绍下 SpringBoot 中如何处理 GTE 和 POST 请求。

# GET 请求

## 带参数

```java
  /**
   * GET 不带参数
   */
  @RequestMapping(method = RequestMethod.GET)
  public Response<List<Student>> testGet(HttpServletRequest request) {
    mLogger.info(request.getQueryString());
    mLogger.info(request.getHeader("userId"));
    Response<List<Student>> response = new Response<>();
    response.data = mMapper.getStudents();
    response.code = 200;
    return response;
  }

```

## 不带参数

```java
  /**
   * GET带参数
   */
  @RequestMapping(method = RequestMethod.GET, path = "/stu")
  public Response tesGetParam(HttpServletRequest request,
      @RequestParam(name = "userId") String id) {
    Student student = mMapper.queryById(id);
    Response<Student> response = new Response<>();
    if (student != null) {
      response.code = 200;
      response.data = student;
    } else {
      response.code = -1;
    }
    return response;
  }
```

# POST 请求

## form 表单

```java
  /**
   * POST 表单传参
   */
  @RequestMapping(method = RequestMethod.POST, path = "/post")
  public Response testPost(HttpServletRequest request,
      @RequestParam(name = "userId") String id) {
    Student student = mMapper.queryById(id);
    Response<Student> response = new Response<>();
    if (student != null) {
      response.code = 200;
      response.data = student;
    } else {
      response.code = -1;
    }
    return response;
  }
```

## Json 结构体

```java
  /**
   * POST Body传参
   */
  @RequestMapping(method = RequestMethod.POST, path = "/insert")
  public Response testPostBody(HttpServletRequest request,
      @RequestBody Student student) {
    Response response = new Response();
    try {
      mMapper.insert(student);
      response.code = 200;
      response.data = mMapper.getStudents();
    } catch (Exception e) {
      response.msg = e.getMessage();
      response.code = -1;
      return response;
    }
    return response;
  }
```

# 完整示例

```java
/**
 * Descriptions: GET、POST 请求示例
 *
 * Created by liuguoquan on 2017/10/31.
 */

@RestController
@RequestMapping("api/rest")
public class RestfulController {

  private final Logger mLogger =
      LogManager.getLogger(RestfulController.class);

  @Autowired StudentMapper mMapper;

  /**
   * GET 不带参数
   */
  @RequestMapping(method = RequestMethod.GET)
  public Response<List<Student>> testGet(HttpServletRequest request) {
    mLogger.info(request.getQueryString());
    mLogger.info(request.getHeader("userId"));
    Response<List<Student>> response = new Response<>();
    response.data = mMapper.getStudents();
    response.code = 200;
    return response;
  }

  /**
   * GET带参数
   */
  @RequestMapping(method = RequestMethod.GET, path = "/stu")
  public Response tesGetParam(HttpServletRequest request,
      @RequestParam(name = "userId") String id) {
    Student student = mMapper.queryById(id);
    Response<Student> response = new Response<>();
    if (student != null) {
      response.code = 200;
      response.data = student;
    } else {
      response.code = -1;
    }
    return response;
  }

  /**
   * POST 表单传参
   */
  @RequestMapping(method = RequestMethod.POST, path = "/post")
  public Response testPost(HttpServletRequest request,
      @RequestParam(name = "userId") String id) {
    Student student = mMapper.queryById(id);
    Response<Student> response = new Response<>();
    if (student != null) {
      response.code = 200;
      response.data = student;
    } else {
      response.code = -1;
    }
    return response;
  }

  /**
   * POST Body传参
   */
  @RequestMapping(method = RequestMethod.POST, path = "/insert")
  public Response testPostBody(HttpServletRequest request,
      @RequestBody Student student) {
    Response response = new Response();
    try {
      mMapper.insert(student);
      response.code = 200;
      response.data = mMapper.getStudents();
    } catch (Exception e) {
      response.msg = e.getMessage();
      response.code = -1;
      return response;
    }
    return response;
  }
}
```

启动程序后，在浏览器中输入测试链接进行测试。


