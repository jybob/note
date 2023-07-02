# 15. 在Spring MVC中接收请求参数

如果客户端提交的请求参数数量较少，且参数之间没有相关性，则可以选择将各请求参数声明为处理请求的方法的参数，并且，参数的类型可以按需设计。

如果客户端提交的请求参数略多（达到2个或以上），且参数之间存在相关性，则应该将这些参数封装到自定义的POJO类型中，并且，使用此POJO类型作为处理请求的方法的参数，同样，POJO类中的属性的类型可以按需设计。

关于请求参数的值：

- 如果客户端没有提交对应名称的请求参数，则方法的参数值为`null`

- 如果客户端提交了对应名称的请求参数，但是没有值，则方法的参数值为空字符串（`""`），如果方法的参数是需要将字符串转换为别的格式，但无法转换，则参数值为`null`，例如声明为`Long`类型时

- 如果客户端提交对应名称的请求参数，且参数有正确的值，则方法的参数值为就是请求参数值，如果方法的参数是需要将字符串转换为别的格式，但无法转换，则会抛出异常

另外，还推荐将某些**具有唯一性的（且不涉及隐私）**参数设计到URL中，使得这些参数值是URL的一部分！例如：

```
https://blog.csdn.net/weixin_407563/article/details/854745877
https://blog.csdn.net/qq_3654243299/article/details/847462823
```

Spring MVC框架支持在设计URL时，使用`{名称}`格式的占位符，实际处理请求时，此占位符位置是任意值都可以匹配得到！

例如，将URL设计为：

```java
@RequestMapping("/delete/{id}")
```

在处理请求的方法的参数列表中，用于接收占位符的参数，需要添加`@PathVariable`注解，例如：

```java
public String delete(@PathVariable Long id) {
	// 暂不关心方法内部的实现   
}
```

如果`{}`占位符中的名称，与处理请求的方法的参数名称不匹配，则需要在`@PathVariable`注解上配置占位符中的名称，例如：

```java
@RequestMapping("/delete/{albumId}")
public String delete(@PathVariable("albumId") Long id) {
	// 暂不关心方法内部的实现   
}
```

在配置占位符时，可以在占位符名称的右侧，可以添加冒号，再加上正则表达式，对占位符的值的格式进行限制，例如：

```java
@RequestMapping("/delete/{id:[0-9]+}")
```

如果按照以上配置，仅当占位符位置的值是纯数字才可以匹配到此URL！

并且，**多个不冲突**有正则表达式的占位符配置的URL是可以共存的！例如：

```java
@RequestMapping("/delete/{id:[a-z]+}")
```

以上表示的是“占位符的值是纯字母的”，是可以与以上“占位符的值是纯数字的”共存！

另外，某个URL的设计没有使用占位符，与使用了占位符的，是允许共存的！例如：

```java
@RequestMapping("/delete/test")
```

Spring MVC会优先匹配没有使用占位符的URL，再尝试匹配使用了占位符的URL。

# 16. 关于RESTful

RESTful也可以简称为REST，是一种设计软件的风格。

> RESTful既不是规定，也不是规范。

RESTful风格的典型表现主要有：

- 处理请求后是响应正文的
- 将具有唯一性的参数值设计在URL中
- 根据请求访问数据的方式，使用不用的请求方式
  - 如果尝试添加数据，使用`POST`请求方式
  - 如果尝试删除数据，使用`DELETE`请求方式
  - 如果尝试修改数据，使用`PUT`请求方式
  - 如果尝试查询数据，使用`GET`请求方式
  - 这种做法在复杂的业务系统中并不适用

在设计RESTful风格的URL时，建议的做法是：

- 查询某数据的列表：`/数据类型的复数`，并使用`GET`请求方式，例如`/albums`
- 查询某1个数据（通常根据id）：`/数据类型的复数/id值`，并使用`GET`请求方式，例如`/albums/9527`
- 对某1个数据进行操作（增、删、改）：`/数据类型的复数/id值/操作`，并使用`POST`请求方式，，例如`/albums/9527/delete`

# 17. 关于响应正文的结果

通常，需要使用自定义类，作为处理请求的方法、处理异常的方法的返回值类型。

响应到客户端的数据中，应该包含“业务状态码”，以便于客户端迅速判断操作成功与否，为了规范的管理业务状态码的值，在根包下创建`web.ServiceCode`枚举类型：

```java
package cn.tedu.csmall.product.web;

/**
 * 业务状态码的枚举
 * 
 * @author java@tedu.cn
 * @version 0.0.1
 */
public enum ServiceCode {

    OK(20000),
    ERR_NOT_FOUND(40400),
    ERR_CONFLICT(40900);

    private Integer value;

    ServiceCode(Integer value) {
        this.value = value;
    }

    public Integer getValue() {
        return value;
    }

}
```

并且，修改`ServiceException`异常类的代码，要求此类异常的对象中包含业务状态码，可以通过构造方法进行限制：

```java
package cn.tedu.csmall.product.ex;

import cn.tedu.csmall.product.web.ServiceCode;

/**
 * 业务异常
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
public class ServiceException extends RuntimeException {

    private ServiceCode serviceCode;

    public ServiceException(ServiceCode serviceCode, String message) {
        super(message);
        this.serviceCode = serviceCode;
    }

    public ServiceCode getServiceCode() {
        return serviceCode;
    }
}
```

然后，原本业务逻辑层中抛出异常的代码将报错，需要在创建异常对象时传入业务状态码参数，例如：

```java
throw new ServiceException(ServiceCode.ERR_CONFLICT,
                    "添加相册失败，尝试添加的相册名称已经被占用！");
```

接下来，还需要使用自定义类型，表示响应到客户端的数据，则在根包下创建`web.JsonResult`类：

```java
package cn.tedu.csmall.product.web;

import lombok.Data;

import java.io.Serializable;

@Data
public class JsonResult implements Serializable {

    /**
     * 业务状态码的值
     */
    private Integer state;
    /**
     * 操作失败时的提示文本
     */
    private String message;

    public static JsonResult ok() {
        JsonResult jsonResult = new JsonResult();
        jsonResult.state = ServiceCode.OK.getValue();
        return jsonResult;
    }

    public static JsonResult fail(ServiceCode serviceCode, String message) {
        JsonResult jsonResult = new JsonResult();
        jsonResult.state = serviceCode.getValue();
        jsonResult.message = message;
        return jsonResult;
    }

}
```

在实际处理请求和异常时，Spring MVC框架会将方法返回的`JsonResult`对象转换成JSON格式的字符串。

例如：处理请求的代码：

```java
@RequestMapping("/add-new")
public JsonResult addNew(AlbumAddNewDTO albumAddNewDTO) {
    log.debug("开始处理【添加相册】的请求，参数：{}", albumAddNewDTO);
    albumService.addNew(albumAddNewDTO);
    return JsonResult.ok();
}
```

例如：处理异常的代码：

```java
@ExceptionHandler
public JsonResult handleServiceException(ServiceException e) {
    log.debug("捕获到ServiceException：{}", e.getMessage());
    return JsonResult.fail(e.getServiceCode(), e.getMessage());
}
```

通常，不需要在JSON结果中包含为`null`的属性，所以，可以在`application.properties` / `application.yml`中进行配置：

**application.properties配置示例**

```properties
# JSON结果中将不包含为null的属性
spring.jackson.default-property-inclusion=non_nul
```

**application.yml配置示例**

```yaml
# Spring相关配置
spring:
  # Jackson框架相关配置
  jackson:
    # JSON结果中是否包含为null的属性的默认配置
    default-property-inclusion: non_null
```

# 18. 实现前后端交互

目前，后端（服务器端）项目使用默认端口`8080`，建议调整，可以通过配置文件中的`server.port`属性来指定。

例如，在`application-dev.yml`中添加配置：

```yaml
# 服务端口
server:
  port: 9080
```

再次启动项目，通过启动时的日志，可以看到后端项目启动在`9080`端口。

然后，还需要在后端项目中配置允许跨域访问，则需要在实现了`WebMvcConfigurer`接口的配置类中，通过重写`addCorsMappings()`方法进行配置！

则在根包下创建`config.WebMvcConfiguration`类，实现`WebMvcConfigurer`接口，添加`@Configuration`注解，并重写方法配置允许跨域访问：

```java
package cn.tedu.csmall.product.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * Spring MVC的配置类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Configuration
public class WebMvcConfiguration implements WebMvcConfigurer {

    public WebMvcConfiguration() {
        System.out.println("创建配置类：WebMvcConfiguration");
    }

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOriginPatterns("*")
                .allowedMethods("*")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }

}
```

# 作业

完成前后端的功能结合，实现通过前端界面的操作来添加相关数据：

- 添加属性模板
- 添加品牌
- 添加类别





