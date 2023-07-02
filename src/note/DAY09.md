# 25. 关于Knife4j框架（续）

如果处理请求时，参数是封装的POJO类型，需要对各请求参数进行说明时，应该在此POJO类型的各属性上使用`@ApiModelProperty`注解进行配置，通过此注解的`value`属性配置请求参数的名称，通过`requeired`属性配置是否必须提交此请求参数（并不具备检查功能），例如：

```java
@Data
public class AlbumAddNewDTO implements Serializable {

    /**
     * 相册名称
     */
    @ApiModelProperty(value = "相册名称", example = "小米10的相册", required = true)
    private String name;

    /**
     * 相册简介
     */
    @ApiModelProperty(value = "相册简介", example = "小米10的相册的简介", required = true)
    private String description;

    /**
     * 排序序号
     */
    @ApiModelProperty(value = "排序序号", example = "98", required = true)
    private Integer sort;

}
```

需要注意，`@ApiModelProperty`还可以用于配置响应时的各数据！

对于处理请求的方法的参数列表中那些未封装的参数（例如`String`、`Long`），需要在处理请求的方法上使用`@ApiImplicitParam`注解来配置参数的说明，并且，必须配置`name`属性，此属性的值就是方法的参数名称，使得此注解的配置与参数对应上，然后，再通过`value`属性对参数进行说明，还要注意，此属性的`required`属性表示是否必须提交此参数，默认为`false`，即使是用于配置路径上的占位符参数，一旦使用此注解，`required`默认也会是`false`，则需要显式的配置为`true`，另外，还可以通过`dataType`配置参数的数据类型，如果未配置此属性，在API文档中默认显示为`string`，可以按需修改为`int`、`long`等。例如：

```java
@ApiImplicitParam(name = "id", value = "相册id", required = true, dataType = "long")
@PostMapping("/{id:[0-9]+}/delete")
public JsonResult delete(@PathVariable Long id) {
    // 暂不关心方法内部的代码
}
```

如果处理请求的方法上有多个未封装的参数，则需要使用多个`@ApiImplicitParam`注解进行配置，并且，这多个`@ApiImplicitParam`注解需要作为`@ApiImplicitParams`注解的参数，例如：

```java
@ApiImplicitParams({
    @ApiImplicitParam(xxx),
    @ApiImplicitParam(xxx),
    @ApiImplicitParam(xxx)
})
```

# 26. 关于检查请求参数

在编写服务器端项目时，当接收到请求参数时，必须第一时间对各请求参数的基本格式进行检查！

需要注意：既然客户端（例如网页）已经检查了请求参数，服务器端应该再次检查，因为：

- 客户端的程序是运行在用户的设备上的，存在程序被篡改的可能性，所以，提交的数据或执行的检查是不可信的
- 在前后端分离的开发模式下，客户端的种类可能较多，例如网页端、手机端、电视端，可能存在某些客户端没有检查
- 升级了某些检查规则，但是，用户的设备上，客户端软件没有升级（例如手机APP还是此前的版本）

以上原因都可能导致客户端没有提交必要的数据，或客户端的检查不完全符合服务器端的要求！所以，对于服务器端而言，客户端提交的所有数据都是不可信的！则服务器端需要对请求参数进行检查！

即使服务器端已经对所有请求参数进行了检查，各个客户端仍应该检查请求参数，因为：

- 能更早的发现明显错误的数据，对应的请求将不会提交到服务器端，能够减轻服务器端的压力
- 客户端的检查不需要与服务器端交互，当出现错误时，能及时得到反馈，对于用户的体验更好

# 27. 通过Validation框架检查请求参数的基本格式

## 27.1. 添加依赖

Spring Validation框架可用于在服务器端检查请求参数的基本格式（例如是否提交了请求参数、字符串的长度是否正确、数字的大小是否在允许的区间等）。

首先，添加依赖项：

```xml
<!-- Spring Boot Validation，用于检查请求参数的基本格式 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

## 27.2. 检查封装在POJO中的请求参数

如果请求参数使用自定义的POJO类型进行封装，当需要检查这些请求参数的基本格式时，需要：

- 在处理请求的方法的参数列表中，在POJO类型前添加`@Validated`或`@Valid`注解，表示需要通过Spring Validation框架对此POJO类型封装的请求参数进行检查
- 在POJO类型的属性上，使用检查注解来配置检查规则，例如`@NotNull`注解就表示“不允许为`null`”，即客户端必须提交此请求参数

所有检查注解都有`message`属性，配置此属性，可用于向客户端响应相关的错误信息。

由于Spring Validation验证请求参数格式不通过时，会抛出异常，所以，可以在全局异常处理器中对此类异常进行处理！

先在`ServiceCode`中添加对应的枚举值：

```java
ERR_BAD_REQUEST(40000)
```

然后，在全局异常处理器中添加对`org.springframework.validation.BindException`的处理：

```java
@ExceptionHandler
public JsonResult handleBindException(BindException e) {
    log.debug("捕获到BindException：{}", e.getMessage());
    // 以下2行代码，如果有多种错误时，将随机获取其中1种错误的信息，并响应
    // String message = e.getFieldError().getDefaultMessage();
    // return JsonResult.fail(ServiceCode.ERR_BAD_REQUEST, message);
    // ===============================
    // 以下代码，如果有多种错误时，将获取所有错误信息，并响应
    StringBuilder stringBuilder = new StringBuilder();
    List<FieldError> fieldErrors = e.getFieldErrors();
    for (FieldError fieldError : fieldErrors) {
        stringBuilder.append(fieldError.getDefaultMessage());
    }
    return JsonResult.fail(ServiceCode.ERR_BAD_REQUEST, stringBuilder.toString());
}
```

## 27.3. 检查时快速失败

可以发现，Spring Validation在检查请求参数格式时，如果检查不通过，会记录下相关的错误，然后，继续进行其它检查，直到所有检查全部完成，才会返回错误信息！

检查全部的错误，相对更加消耗服务器资源，可以通过配置，使得检查出错时直接结束并返回错误！

```java
package cn.tedu.csmall.product.config;

import lombok.extern.slf4j.Slf4j;
import org.hibernate.validator.HibernateValidator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.validation.Validation;

/**
 * Spring Validation的配置类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Slf4j
@Configuration
public class ValidationConfiguration {

    public ValidationConfiguration() {
        log.debug("创建配置类：ValidationConfiguration");
    }

    @Bean
    public javax.validation.Validator validator() {
        return Validation.byProvider(HibernateValidator.class)
                .configure() // 开始配置Validator
                .failFast(true) // 快速失败，即检查请求参数发现错误时直接视为失败，并不向后继续检查
                .buildValidatorFactory()
                .getValidator();
    }

}
```

## 27.4. 常用检查注解

关于检查注解，常用的有：

- `@NotNull`：不允许为`null`，适用于所有类型的请求参数
- `@NotEmpty`：不允许为空字符串（长度为0的字符串），仅适用于字符串类型的请求参数
  - 此注解不检查是否为`null`，即请求参数为`null`将通过检查
  - 此注解可以与`@NotNull`同时使用
- `@NotBlank`：不允许为空白（形成空白的主要有：空格、TAB制表位、换行等），仅适用于字符串类型的请求参数
  - 此注解不检查是否为`null`，即请求参数为`null`将通过检查
  - 此注解可以与`@NotNull`同时使用

- `@Pattern`：要求被检查的请求参数必须匹配某个正则表达式，通过此注解的`regexp`属性可以配置正则表达式，仅适用于字符串类型的请求参数
  - 此注解不检查是否为`null`，即请求参数为`null`将通过检查
  - 此注解可以与`@NotNull`同时使用
- `@Range`：要求被检查的数值型请求参数必须在某个数值区间范围内，通过此注解的`min`属性可以配置最小值，通过此注解的`max`属性可以配置最大值，仅适用于数值类型的请求参数
  - 此注解不检查是否为`null`，即请求参数为`null`将通过检查
  - 此注解可以与`@NotNull`同时使用

另外，在`org.hibernate.validator.constraints`和`javax.validation.constraints`包还有其它检查注解。

## 27.5. 检查基本值的请求参数

如果请求参数是一些基本值，没有封装（例如`String`、`Integer`、`Long`），则需要将检查注解添加在请求参数上，例如：

```java
@Deprecated
@ApiOperation("删除相册【测试2】")
@ApiOperationSupport(order = 910)
@ApiImplicitParam(name = "id", value = "相册id", paramType = "query")
@PostMapping("/delete/test2")
// ===== 重点关注以下方法参数上的注解 =====
public String deleteTest(@Range(min = 1, message = "测试删除相册失败，id值必须是1或更大的有效整数！") Long id) {
    log.debug("【测试】开始处理【删除相册】的请求，这只是一个测试，没有实质功能！");
    return "OK";
}
```

然后，还需要在控制器类上添加`@Validated`注解，以上方法参数前的检查注解才会生效！如果后续运行时没有通过此检查，Spring Validation框架将抛出`ConstraintViolationException`类型的异常，例如：

```
javax.validation.ConstraintViolationException: deleteTest.id: 测试删除相册失败，id值必须是1或更大的有效整数！
```

则在全局异常处理器中添加处理以上异常的方法：

```java
@ExceptionHandler
public JsonResult handleConstraintViolationException(ConstraintViolationException e) {
    log.debug("捕获到ConstraintViolationException：{}", e.getMessage());
    StringBuilder stringBuilder = new StringBuilder();
    Set<ConstraintViolation<?>> constraintViolations = e.getConstraintViolations();
    for (ConstraintViolation<?> constraintViolation : constraintViolations) {
        stringBuilder.append(constraintViolation.getMessage());
    }
    return JsonResult.fail(ServiceCode.ERR_BAD_REQUEST, stringBuilder.toString());
}
```

# 28. 显示相册列表--Mapper层

查询相册列表的功能此前已经实现！

# 29. 显示相册列表--Service层

在`IAlbumService`接口中添加抽象方法：

```java
/**
 * 查询相册列表
 *
 * @return 相册列表
 */
List<AlbumListItemVO> list();
```

在`AlbumServiceImpl`类中实现以上方法：

```java
@Override
public List<AlbumListItemVO> list() {
    log.debug("开始处理【查询相册列表】的业务");
    return albumMapper.list();
}
```

在`AlbumServiceTests`中编写并执行测试：

```java
@Test
void testList() {
    List<?> list = service.list();
    System.out.println("查询列表完成，列表中的数据的数量=" + list.size());
    for (Object item : list) {
        System.out.println(item);
    }
}
```

# 30. 显示相册列表--Controller层

服务器端处理请求后响应的都是`JsonResult`对象，但是，目前，此类型中并不足以表示“响应到客户端的数据”，需要在类中补充新的属性，用于封装响应到客户端的数据！

则先调整`JsonResult`类：

```java
package cn.tedu.csmall.product.web;

import cn.tedu.csmall.product.ex.ServiceException;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import java.io.Serializable;

@Data
public class JsonResult<T> implements Serializable {

    /**
     * 业务状态码
     */
    @ApiModelProperty("业务状态码")
    private Integer state;
    /**
     * 操作失败时的提示文本
     */
    @ApiModelProperty("操作失败时的提示文本")
    private String message;
    /**
     * 操作成功时的响应数据
     */
    @ApiModelProperty("操作成功时的响应数据")
    private T data;

    public static JsonResult<Void> ok() {
        // JsonResult jsonResult = new JsonResult();
        // jsonResult.state = ServiceCode.OK.getValue();
        // jsonResult.message = null;
        // jsonResult.data = null;
        // return jsonResult;
        return ok(null);
    }

    public static <T> JsonResult<T> ok(T data) {
        JsonResult jsonResult = new JsonResult();
        jsonResult.state = ServiceCode.OK.getValue();
        jsonResult.message = null;
        jsonResult.data = data;
        return jsonResult;
    }

    public static JsonResult<Void> fail(ServiceException e) {
        // JsonResult jsonResult = new JsonResult();
        // jsonResult.state = e.getServiceCode().getValue();
        // jsonResult.message = e.getMessage();
        // return jsonResult;
        return fail(e.getServiceCode(), e.getMessage());
    }

    public static JsonResult<Void> fail(ServiceCode serviceCode, String message) {
        JsonResult jsonResult = new JsonResult();
        jsonResult.state = serviceCode.getValue();
        jsonResult.message = message;
        return jsonResult;
    }

}
```

然后，在`AlbumController`中添加处理请求的方法：

```java
// http://localhost:9080/albums
@ApiOperation("查询相册列表")
@ApiOperationSupport(order = 420)
@GetMapping("")
public JsonResult<List<AlbumListItemVO>> list() {
    log.debug("开始处理【查询相册列表】的请求");
    List<AlbumListItemVO> list = albumService.list();
    return JsonResult.ok(list);
}
```

完成后，重启项目，通过在线API文档的调试功能可以测试访问。



# 作业

完成以下功能，最终通过在线API文档的调试功能可以测试访问即可：

- 显示品牌列表
- 显示属性模板列表
- 显示管理员列表（在`passport`项目中）



