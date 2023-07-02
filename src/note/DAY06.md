# 9. 关于异常

在Java语言中，异常的继承结构大致是：

```
Throwable
-- Error
-- -- OutOfMemoryError（OOM）
-- Exception
-- -- IOException
-- -- -- FileNotFoundException
-- -- RuntimeException
-- -- -- NullPointerException（NPE）
-- -- -- IllegalArgumentException
-- -- -- ClassNotFoundException
-- -- -- ClassCastException
-- -- -- ArithmeticException
-- -- -- IndexOutOfBoundsException
-- -- -- -- ArrayIndexOutOfBoundsException
-- -- -- -- StringIndexOutOfBoundsException
```

如果调用的某个方法抛出了非`RuntimeException`，则必须在源代码中使用`try...catch`或`throws`语法，否则，源代码将报错！而`RuntimeException`不会受到这类语法的约束！

在项目中，如果需要通过抛出异常来表示某种“错误”，应该使用自定义的异常类型，否则，可能与框架或其它方法抛出的异常相同，在处理时，会模糊不清（不清楚异常到底是显式的抛出的，还是调用其它方法时由那些方法抛出的）！同时，为了避免抛出异常时有非常多复杂的语法约束，通常，自定义的异常都会是`RuntimeException`的子孙类异常。

另外，抛出异常时，应该对出现异常的原因进行描述，所以，在自定义异常类中，应该添加带`String message`参数的构造方法，且此构造方法需要调用父类的带`String message`参数的构造方法。

则在项目的根包下创建`ex.ServiceException`异常类，继承自`RuntimeException`，例如：

```java
package cn.tedu.csmall.product.ex;

/**
 * 业务异常
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
public class ServiceException extends RuntimeException {

    public ServiceException(String message) {
        super(message);
    }

}
```

然后，在Service中，就抛出此类异常，并添加对于错误的描述文本，例如：

```java
if (count != 0) {
    // 是：名称已存在，不允许创建，抛出异常
    throw new ServiceException("添加相册失败，尝试添加的相册名称已经被占用！");
}
```

在Controller中，将调用Service中的方法，可以使用`try..catch`包裹这段代码，对异常进行捕获并处理，例如：

```java
try {
    albumService.addNew(albumAddNewDTO);
    return "添加相册成功！";
} catch (ServiceException e) {
    return e.getMessage();
}
```

# 10. Spring MVC框架的统一处理异常机制

由于Service在处理业务，如果视为”失败“，将抛出异常，并且，抛出时还会封装”失败“的描述文本，而Controller每次调用Service的任何方法时，都会使用`try..catch`进行捕获并处理，并且，处理的代码都是相同的（暂时是`return e.getMessage();`），这样的做法是非常固定的，导致在Controller中存在大量的`try...catch`（处理任何请求，调用Service时都是这样的代码）。

Spring MVC提供了统一处理异常的机制，它可以使得Controller不再处理异常，改为抛出异常，而Spring MVC在调用Controller处理请求时，会捕获Controller抛出的异常并尝试处理。

关于处理异常的方法：

- 访问权限：应该是`public`
- 返回值类型：参考处理请求的方法
- 方法名称：自定义
- 参数列表：至少需要添加1个异常类型的参数，表示你希望处理的异常，也是Spring MVC框架调用Controller的方法时捕获到的异常
- 注解：`@ExceptionHandler`
- 如果将处理异常的方法定义在某个Controller中，仅作用于当前Controller中所有处理请求的方法，对别的Controller中处理请求的方法是**不生效**的！Spring MVC建议将处理异常的代码写在专门的类中，并且，在类上添加`@RestControllerAdvice`注解，当添加此注解后，此类中处理异常的代码将作用于**整个项目每次处理请求的过程**中
- 允许存在多个处理异常的方法，只要这些方法处理的异常类型**不直接冲突**即可
  - 即：不允许2个处理异常的方法都处理同一种异常
  - 即：允许多个处理异常的方法中处理的异常存在继承关系，例如A方法处理`NullPointerException`，B方法处理`RuntimeException`

在实际处理时，推荐添加一下对`Throwable`处理的方法，以避免某些异常没有被处理，导致响应500错误。

关于处理异常的类，暂定为：

```java
package cn.tedu.csmall.product.ex.handler;

import cn.tedu.csmall.product.ex.ServiceException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

/**
 * 全局异常处理器
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    public GlobalExceptionHandler() {
        System.out.println("创建全局异常处理器对象：GlobalExceptionHandler");
    }

    @ExceptionHandler
    public String handleServiceException(ServiceException e) {
        log.debug("捕获到ServiceException：{}", e.getMessage());
        return e.getMessage();
    }

    @ExceptionHandler
    public String handleThrowable(Throwable e) {
        log.debug("捕获到Throwable：{}", e.getMessage());
        e.printStackTrace(); // 强烈建议
        return "服务器运行过程中出现未知错误，请联系系统管理员！";
    }

}
```

# 11. 关于控制器类Controller

在Spring MVC框架，使用控制器（`Controller`）来接收请求、响应结果。

在根包下的任何一个类，添加了`@Controller`注解，就会被视为控制器类。

在默认情况下，控制器类中处理请求的方法，响应的结果是”视图组件的名称“，即：控制器对请求处理后，将返回视图名称，Spring MVC还会根据视图名称来确定视图组件，并且，由此视图组件来响应！这**不是**前后端分离的做法！

> 提示：如果需要了解传统的Spring MVC不使用前后端分离的做法，可以参考扩展视频教程《基于XML配置的Spring MVC框架》。

可以在处理请求的方法上添加`@ResponseBody`注解，则此方法处理请求后，返回的值就是响应到客户端的数据！这种做法通常称之为”响应正文“。

`@ResponseBody`注解还可以添加在控制器类上，则此控制器类中所有处理请求的方法都将是”响应正文“的！

另外，还可以使用`@RestController`取代`@Controller`和`@ResponseBody`，关于`@RestController`的源代码：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {
    @AliasFor(
        annotation = Controller.class
    )
    String value() default "";
}
```

可以看到，在`@RestController`的源代码上，添加了`@Controller`和`@ResponseBody`，所以，可以把`@RestController`称之为”组合注解“，而`@Controller`和`@ResponseBody`可以称之为`@RestController`的”元注解“。

与之类似的，在Spring MVC框架中，添加了`@ControllerAdvice`注解的类中的特定方法，将可以作用于每次处理请求的过程中，但是，仅仅只使用`@ControllerAdvice`时，并不是前后端分离的做法，还应该结合`@ResponseBody`一起使用，或，直接改为使用`@RestControllerAdvice`，关于`@RestControllerAdvice`的源代码片段：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ControllerAdvice
@ResponseBody
public @interface RestControllerAdvice {
	// 暂不关心内部源代码   
}
```

# 12. 关于控制器类中处理请求的方法

关于处理请求的方法：

- 访问权限：应该使用`public`
- 返回值类型：暂时使用`String`
- 方法名称：自定义
- 参数列表：按需设计，即需要客户端提交哪些请求参数，在此方法的参数列表中就设计哪些参数，如果参数的数量有多个，并且多个参数具有相关性，则可以封装，并使用封装的类型作为方法的参数，另外，可以按需添加Spring容器中的其它相关数据作为参数，例如`HttpServletRequest`、`HttpServletResponse`、`HttpSession`等
- 异常：如果有，全部抛出
- 注解：需要通过`@RequestMapping`系列注解配置请求路径

# 13. 关于`@RequestMapping`

在Spring MVC框架中，`@RequestMapping`的主要作用是：配置**请求路径**与**处理请求的方法**的映射关系。

此注解可以添加在控制类上，也可以添加在处理请求的方法上。

通常，会在控制器类和处理请求的方法上都配置此注解，例如：

```java
@RestController
@RequestMapping("/albums")
public class AlbumController {

    @RequestMapping("/add-new")
    public String addNew(AlbumAddNewDTO albumAddNewDTO) {
        // 暂不关心方法内部代码
    }

}
```

以上配置的路径将是：`http://主机名:端口号/类上配置路径/方法上配置的路径`，即：http://localhost:8080/albums/add-new

并且，在使用``@RequestMapping`配置路径时，路径值两端多余的 `/` 是会被自动处理的，在类上的配置值和方法上的配置值中间的 `/` 也是自动处理的，例如，以下配置是等效的：

| 类上的配置值 | 方法上的配置值 |
| ------------ | -------------- |
| /albums      | /add-new       |
| /albums      | add-new        |
| /albums/     | /add-new       |
| /albums/     | add-new        |
| albums       | /add-new       |
| albums       | add-new        |
| albums/      | /add-new       |
| albums/      | add-new        |

尽管以上8种组合配置是等效的，但仍推荐使用第1种。

在`@RequestMapping`注解的源代码中，有：

```java
@AliasFor("path")
String[] value() default {};
```

以上源代码表示在此注解中存在名为`value`的属性，并且，此属性的值类型是`String[]`，例如，你可以配置`@RequestMapping(value = {"xxx", "zzz"})`，此属性的默认值是`{}`（空数组）。

在所有注解中，`value`是默认的属性，所以，如果你需要配置的注解参数是`value`属性，且只配置这1个属性时，并不需要显式的指定属性名！例如：

```java
@RequestMapping(value = {"xxx", "zzz"})
```

```java
@RequestMapping({"xxx", "zzz"})
```

以上2种配置方式是完全等效的！

在所有注解中，如果某个属性的值是数组类型的，但是，你只提供1个值（也就是数组中只有1个元素），则这个值并不需要使用大括号框住！例如：

```java
@RequestMapping(value = {"xxx"})
```

```java
@RequestMapping(value = "xxx")
```

以上2种配置方式是完全等效的！

在源代码中，关于`value`属性的声明上还有`@AliasFor("path")`，它表示”等效于“的意思，也就是说，`value`属性与另一个名为`path`的属性是完全等效的！

在`@RequestMapping`的源代码中，还有：

```java
RequestMethod[] method() default {};
```

以上属性的作用是**配置并限制请求方式**，例如，配置为：

```java
@RequestMapping(value = "/add-new", method = RequestMethod.POST)
```

按照以上配置，以上请求路径**只允许使用POST方式**提交请求！

**强烈建议在正式运行的代码中，明确的配置并限制各请求路径的请求方式！**

另外，在Spring MVC框架中，还定义了基于`@RequestMapping`的相关注解：

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

**所以，在开发实践中，通常：在控制器类上使用`@RequestMapping`配置请求路径的前缀部分，在处理请求的方法上使用`@GetMapping`、`@PostMapping`这类限制了请求方式的注解。**

# 14. 根据id删除相册

目前，在Mapper层已经实现了此功能！则只需要开发Service层和Controller层。

在`IAlbumService`接口中添加抽象方法：

```java
void delete(Long id);
```

在`AlbumServiceImpl`类中实现以上方法：

```java
@Override
public void delete(Long id) {
    log.debug("开始处理【删除相册】的业务，参数：{}", id);
    // 调用Mapper对象的getDetailsById()方法执行查询
    AlbumStandardVO queryResult = albumMapper.getStandardById(id);
    // 判断查询结果是否为null
    if (queryResult == null) {
        // 是：无此id对应的数据，抛出异常
        String message = "删除相册失败，尝试访问的数据不存在！";
        log.warn(message);
        throw new ServiceException(message);
    }

    // 调用Mapper对象的deleteById()方法执行删除
    log.debug("即将删除相册数据……");
    albumMapper.deleteById(id);
    log.debug("删除相册，完成！");
}
```

在`AlbumServiceTests`类中编写并执行测试：

```java
@Test
void testDelete() {
    Long id = 14L;

    try {
        service.delete(id);
        System.out.println("删除相册成功！");
    } catch (ServiceException e) {
        System.out.println(e.getMessage());
    }
}
```

在`AlbumController`中添加处理请求的方法：

```java
// http://localhost:8080/albums/delete?id=1
@RequestMapping("/delete")
public String delete(Long id) {
    log.debug("开始处理【删除相册】的请求，参数：{}", id);
    albumService.delete(id);
    return "OK";
}
```



# 作业

请完成以下功能的Service层、Controller层，并且，Service层需要完成测试：

- 根据id删除属性模板，业务规则：数据必须存在
- 添加品牌，业务规则：品牌名称必须唯一
- 根据id删除品牌，业务规则：数据必须存在
- 添加类别，业务规则：类别名称必须唯一
- 根据id删除类别，业务规则：数据必须存在



