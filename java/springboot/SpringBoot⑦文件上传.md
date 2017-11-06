**7.SpringBoot之文件上传**

[TOC]

SpringBoot 文件上传的主要步骤：

1. 创建一个 SpringBoot 工程，见 SpringBoot① 入门篇。
2. 配置 pom.xml 依赖，IntelliJ IDEA 创建工程后自动配置好。
3. 创建和编写文件上传的 RestController。
4. 运行，使用 Postman 进行测试。

# 单文件上传

```java
  /**
   * 单个文件上传
   */
  @RequestMapping(method = RequestMethod.POST)
  public Response upload(@RequestParam("file") MultipartFile file) {
    Response response = new Response();
    if (!file.isEmpty()) {
      try {
        //保存文件
        BufferedOutputStream out = new BufferedOutputStream(
            new FileOutputStream(new File("123.jpeg")));
        out.write(file.getBytes());
        out.flush();
        out.close();
      } catch (IOException e) {
        response.msg = e.getMessage();
        response.code = -1;
        return response;
      }
    } else {
      response.msg = "文件为空";
      response.code = -1;
      return response;
    }
    response.code = 200;
    return response;
  }
```

# 多文件上传

```java
  /**
   * 多文件上传
   */
  @RequestMapping(path = "/batch", method = RequestMethod.POST)
  public Response uploadBatch(HttpServletRequest request) {
    Response response = new Response();
    List<MultipartFile> multipartFiles = ((MultipartHttpServletRequest) request).getFiles("file");
    BufferedOutputStream out = null;
    for (MultipartFile file : multipartFiles) {
      if (!file.isEmpty()) {
        try {
          out = new BufferedOutputStream(
              new FileOutputStream(new File(System.currentTimeMillis() + ".jpeg")));
          out.write(file.getBytes());
          out.flush();
          out.close();
        } catch (IOException e) {
          response.msg = e.getMessage();
          response.code = -1;
          return response;
        }
      } else {
        response.msg = "文件为空";
        response.code = -1;
        return response;
      }
    }
    response.code = 200;
    response.msg = "上传成功";
    return response;
  }
```

# 示例

这里只给出 RestController 文件的代码。

```java
/**
 * Descriptions:
 *
 * Created by liuguoquan on 2017/11/1.
 */

@RestController
@RequestMapping("api/upload")
public class UploadController {

  /**
   * 单个文件上传
   */
  @RequestMapping(method = RequestMethod.POST)
  public Response upload(@RequestParam("file") MultipartFile file) {
    Response response = new Response();
    if (!file.isEmpty()) {
      try {
        BufferedOutputStream out = new BufferedOutputStream(
            new FileOutputStream(new File("123.jpeg")));
        out.write(file.getBytes());
        out.flush();
        out.close();
      } catch (IOException e) {
        response.msg = e.getMessage();
        response.code = -1;
        return response;
      }
    } else {
      response.msg = "文件为空";
      response.code = -1;
      return response;
    }
    response.code = 200;
    return response;
  }

  /**
   * 多文件上传
   */
  @RequestMapping(path = "/batch", method = RequestMethod.POST)
  public Response uploadBatch(HttpServletRequest request) {
    Response response = new Response();
    List<MultipartFile> multipartFiles = ((MultipartHttpServletRequest) request).getFiles("file");
    BufferedOutputStream out = null;
    for (MultipartFile file : multipartFiles) {
      if (!file.isEmpty()) {
        try {
          out = new BufferedOutputStream(
              new FileOutputStream(new File(System.currentTimeMillis() + ".jpeg")));
          out.write(file.getBytes());
          out.flush();
          out.close();
        } catch (IOException e) {
          response.msg = e.getMessage();
          response.code = -1;
          return response;
        }
      } else {
        response.msg = "文件为空";
        response.code = -1;
        return response;
      }
    }
    response.code = 200;
    response.msg = "上传成功";
    return response;
  }
}
```
启动服务，访问 `http://localhost:8080/upload` 和 `http://localhost:8080/upload/batch` 测试文件上传。


