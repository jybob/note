# 3. SLF4j日志（续）

输出日志的各个方法都被重载了多次，建议使用的方法例如：

```java
void trace(String var1);

void trace(String var1, Object... var2);
```

> 提示：以上是`trace`方法，其它级别的日志也有完全相同参数列表的方法。

以上的第2个方法适用于在输出的日志中添加变量的值，在第1个字符串参数中，各变量均使用`{}`表示，然后，通过第2个参数依次传入各变量对应的值即可，例如：

```java
int x = 1;
int y = 2;
log.trace("{}+{}={}", x, y, x + y);
```

使用以上方式输出时，会将字符串部分进行缓存（是一种预编译的做法），在执行时，并不会出现拼接字符串的情况，所以，在执行效率方面，比传统的`System.out.println()`的要高很多！

# 4. 关于Profile配置

在配置中，许多配置值会因为当前环境不同，需要配置不同的值，例如，在开发环境中，日志的显示级别可以是`trace`这种较低级别的，而在测试环境、生产环境（项目部署上线并运行）中可能需要改为其它值，再例如，连接数据库配置的相关参数，在不同环境中，可能使用不同的值……如果在`application.properties`中反复修改大量配置值，是非常不方便的！

Spring框架支持Profile配置（个性化配置），Spring Boot简化了Profile配置的使用，它支持使用`application-xxx.properties`作为配置文件的名称，其中，`xxx`部分是完全自定义的名称，你可以针对不同的环境，编写一组配置文件，这些配置文件中配置了相同的属性，但是值不同，例如：

**application-dev.properties（暂定为“开发”环境使用的配置文件）**

```properties
# 连接数据库的配置
spring.datasource.url=jdbc:mysql://localhost:3306/mall_pms?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=root

# 日志的显示级别
logging.level.cn.tedu.csmall=trace
```

**application-test.properties（暂定为“测试”环境使用的配置文件）**

```properties
# 连接数据库的配置
spring.datasource.url=jdbc:mysql://192.168.1.100:3306/mall_pms?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
spring.datasource.username=test
spring.datasource.password=test123

# 日志的显示级别
logging.level.cn.tedu.csmall=debug
```

**application-prod.properties（暂定为“生产”环境使用的配置文件）**

```properties
# 连接数据库的配置
spring.datasource.url=jdbc:mysql://192.168.1.255:3306/mall_pms?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
spring.datasource.username=prod
spring.datasource.password=s3ctet@tedu.cn

# 日志的显示级别
logging.level.cn.tedu.csmall=info
```

以上这些不在`application.properties`中的配置，默认是不生效的！需要在`application.properties`中显式的激活后，才会生效！例如：

```properties
# 激活Profile配置
spring.profiles.active=prod
```

小结：

- `appliation.properties`是始终加载的配置
- `application-xxx.properties`是需要通过`application.properties`中的`spring.profiles.active`属性来激活的
- `application-xxx.properties`文件名中的`xxx`是自定义的名称，也是`spring.profiles.active`属性的值
- 需要自行评估哪些配置会因为环境不同而配置不同的值

# 5. 关于YAML配置

YAML是一种使用`.yml`作为扩展名的配置文件，这类配置文件在Spring框架中默认是不支持的，需要添加额外的依赖项，在Spring Boot项目中，默认已经集成了相关依赖项，所以，在Spring Boot项目中可以直接使用。

在Spring Boot项目中，可以使用`.properties`配置，也可以使用`.yml`配置。

相对`.properties`配置，YAML的配置语法为：

- 在`.properties`配置中，属性名使用小数点分隔的，改为使用冒号结束，并从下一行开始，缩进2个空格
- 属性名与属性值之间使用1个冒号和1个空格进行分隔
- 多个不同的配置属性中，如果属性名中有相同的部分，可以不必重复配置，只需要将不同的部分缩进在相同位置即可
- 如果某个属性值是纯数字的，但需要是字符串类型，可以使用一对单引号框住

例如在`.properties`中配置为：

```properties
spring.datasource.username=root
spring.datasource.password=1234
```

在`.yml`中则配置为：

```yaml
spring:
  datasource:
    username: root
    password: '1234'
```

**注意：每换行后，需要缩进2个空格，在IntelliJ IDEA中，编写`.yml`文件时，IntelliJ IDEA会自动将按下的TAB键的字符替换为2个空格。**

**提示：如果`.yml`文件出现乱码（通常是因为复制粘贴文件导致的），则无法解析，项目启动时就会报错，此时，应该保留原代码（将代码复制到别处），删除报错的配置文件，并重新创建新文件，将保留下来的原代码粘贴回新文件即可。**

# 5. 关于业务逻辑

业务逻辑：数据的处理应该有一定的处理流程，并且，在此流程中，可能需要制定某些规则，如果不符合规则，不应该允许执行相关的数据访问！这套流程及相关逻辑则称之为业务逻辑。

例如：当用户尝试注册时，通常要求用户名（或类似的唯一标签，例如手机号码等）需要是“唯一的”，在执行插入数据（将用户信息插入到数据表中）之前，应该先检查用户名是否已经被占用。

**业务逻辑层的主要价值是设计业务流程，及业务逻辑，以保证数据的完整性和安全性。**

在代码的设计方面，业务逻辑层将使用`Service`作为名称的关键词，并且，应该先自定义业务逻辑层接口，再自定义类实现此接口，实现类的名称应该在接口名称的基础上添加`Impl`后缀。例如，处理相册数据的业务逻辑接口的名称应该是`IAlbumService`或`AlbumService`，其实现类的名称则是``AlbumServiceImpl`。

# 6. 业务：添加相册

先在项目的根包下创建`service.IAlbumService`接口：

```java
public interface IAlbumService {
    
}
```

然后，在`service`包下创建`impl.AlbumServiceImpl`类，实现以上接口，并且在类上添加`@Service`注解：

```java
@Service
public class AlbumServiceImpl implements IAlbumService {
    
}
```

提示：在类上添加了`@Autowired`注解后，当启动项目或执行任何一个Spring Boot测试时，都会自动创建以上类的对象，并且，可以使用`@Autowired`实现属性的“自动赋值”。

由于实体类不适合作为业务方法参数，来表示“客户端将提交的数据”，所以，应该使用专门的POJO类型作为业务方法的参数类型！当前项目中使用`DTO`作为此类POJO的后缀！

则在根包下创建`pojo.dto.AlbumAddNewDTO`类：

```java
package cn.tedu.csmall.product.pojo.dto;

import lombok.Data;

import java.io.Serializable;

/**
 * 添加相册的DTO类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Data
public class AlbumAddNewDTO implements Serializable {

    /**
     * 相册名称
     */
    private String name;

    /**
     * 相册简介
     */
    private String description;

    /**
     * 自定义排序序号
     */
    private Integer sort;

}
```

接下来，需要在接口中定义“添加相册”的抽象方法：

```java
/**
 * 添加相册
 *
 * @param albumAddNewDTO 相册数据
 */
void addNew(AlbumAddNewDTO albumAddNewDTO);
```

**提示1：业务方法的参数应该是自定义的POJO类型**

**提示2：业务方法的名称是自定义的**

**提示3：业务方法的返回值类型，是仅以“成功”为前提来设计的，不考虑失败的情况，因为失败将通过抛出异常来表现**

接下来，在`AlbumServiceImpl`中重写抽象方法：

```java
@Autowired
AlbumMapper albumMapper;

@Override
public void addNew(AlbumAddNewDTO albumAddNewDTO) {
    // 从参数albumAddNewDTO中获取尝试添加的相册名称
    // 检查此相册名称是否已经存在：调用Mapper对象的countByName()方法，判断结果是否不为0
    // 是：名称已存在，不允许创建，抛出异常
    
    // 创建Album对象
    // 调用Album对象的setName()方法来封装数据：来自参数albumAddNewDTO
    // 调用Album对象的setDescription()方法来封装数据：来自参数albumAddNewDTO
    // 调用Album对象的setSort()方法来封装数据：来自参数albumAddNewDTO
	// 调用Mapper对象的insert()方法，插入相册数据
}
```

所以，在实际以上业务之前，需要先补充“检查相册名称是否已经存在”的查询功能，此查询功能需要执行的SQL语句大致是：

```mysql
SELECT count(*) FROM pms_album WHERE name=?
```

则在`AlbumMapper`接口中添加抽象方法：

```java
/**
 * 根据相册名称，统计数据的数量
 *
 * @param name 相册名称
 * @return 此名称的相册数据的数量
 */
int countByName(String name);
```

并在`AlbumMapper.xml`中配置以上抽象方法映射的SQL语句：

```xml
<!-- int countByName(String name); -->
<select id="countByName" resultType="int">
    SELECT count(*) FROM pms_album WHERE name=#{name}
</select>
```

完成后，还应该在`AlbumMapperTests`中测试：

```java
@Test
void testCountByName() {
    String name = "测试相册001";
    int count = mapper.countByName(name);
    System.out.println("根据名称【" + name + "】统计数据完成，数量=" + count);
}
```

当补全Mapper层的查询功能后，再实现Service的方法，即`AlbumServiceImpl`中的`addNew()`方法：

```java
package cn.tedu.csmall.product.service.impl;

import cn.tedu.csmall.product.mapper.AlbumMapper;
import cn.tedu.csmall.product.pojo.dto.AlbumAddNewDTO;
import cn.tedu.csmall.product.pojo.entity.Album;
import cn.tedu.csmall.product.service.IAlbumService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 处理相册数据的业务实现类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Service
public class AlbumServiceImpl implements IAlbumService {

    @Autowired
    AlbumMapper albumMapper;

    public AlbumServiceImpl() {
        System.out.println("创建业务实现类对象：AlbumServiceImpl");
    }

    @Override
    public void addNew(AlbumAddNewDTO albumAddNewDTO) {
        // 从参数albumAddNewDTO中获取尝试添加的相册名称
        String name = albumAddNewDTO.getName();
        // 检查此相册名称是否已经存在：调用Mapper对象的countByName()方法，判断结果是否不为0
        int count = albumMapper.countByName(name);
        if (count != 0) {
            // 是：名称已存在，不允许创建，抛出异常
            throw new RuntimeException();
        }

        // 创建Album对象
        Album album = new Album();
        // 将参数DTO中的数据复制到Album对象中
        BeanUtils.copyProperties(albumAddNewDTO, album);
        // 调用Mapper对象的insert()方法，插入相册数据
        albumMapper.insert(album);
    }

}
```

完成后，在`src/test/java`的根包下创建`service.AlbumServiceTests`测试类，编写并执行测试：

```java
package cn.tedu.csmall.product.service;

import cn.tedu.csmall.product.pojo.dto.AlbumAddNewDTO;
import cn.tedu.csmall.product.service.impl.AlbumServiceImpl;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class AlbumServiceTests {

    @Autowired
    IAlbumService service;

    @Test
    void contextLoads() {
        System.out.println(service);
    }

    @Test
    void testAddNew() {
        AlbumAddNewDTO albumAddNewDTO = new AlbumAddNewDTO();
        albumAddNewDTO.setName("测试相册名称001");
        albumAddNewDTO.setDescription("测试相册简介001");
        albumAddNewDTO.setSort(88);

        try {
            service.addNew(albumAddNewDTO);
            System.out.println("添加相册成功！");
        } catch (RuntimeException e) {
            System.out.println("添加相册失败，相册名称已经被占用！");
        }
    }

}
```

# 7. 关于控制器

在编写控制器相关代码之前，需要在项目中添加`spring-boot-starter-web`依赖项。

`spring-boot-starter-web`是基于（包含）`spring-boot-starter`的，所以，添加`spring-boot-starter-web`后，就不必再显式的添加`spring-boot-starter`了，则可以将原本的`spring-boot-starter`改成`spirng-boot-starter-web`即可，例如：

```xml
<!-- Spring Boot的Web依赖项，包含基础依赖项 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

`spring-boot-starter-web`包含了一个内置的Tomcat，当启动项目时，会自动将当前项目编译、打包并部署到此Tomcat上。

在项目中，控制器表现为各个`Controller`，是由Spring MVC框架实现的相关数据处理功能，以上添加的`spring-boot-starter-web`包含了Spring MVC框架的依赖项（`spring-webmvc`）。

控制器的主要作用是接收请求，并响应结果。

# 8. 控制器：添加相册

在项目的根包下创建`controller.AlbumController`类，并在类上添加`@RestController`注解：

```java
@RestController
public class AlbumController {
    
}
```

并在控制器类中定义处理请求的方法：

```java
@Autowired
IAlbumService albumService;

// http://localhost:8080/album/add-new?name=TestAlbumName001&description=TestAlbumDescription001&sort=77
@RequestMapping("/album/add-new")
public String addNew(AlbumAddNewDTO albumAddNewDTO) {
    try {
        albumService.addNew(albumAddNewDTO);
        return "添加相册成功！";
    } catch (RuntimeException e) {
        return "添加相册失败，尝试添加的相册名称已经被占用！";
    }
}
```

完成后，重启项目，在浏览器中可以通过 http://localhost:8080/album/add-new?name=TestAlbumName001&description=TestAlbumDescription001&sort=77 测试访问。

# 作业：

1. 实现：添加属性模板，业务规则为“属性模板的名称必须唯一”
2. 思考题：
   1. Java语言中异常的继承结构
   2. `RuntimeException`有什么特殊之处
   3. 什么是捕获，什么是抛出，怎么样操作才算是处理异常















