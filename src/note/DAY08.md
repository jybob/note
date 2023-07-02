# 19. 创建passport项目

创建新的项目，相关参数：

项目仓库：`https://gitee.com/chengheng2022/jsd2206-csmall-passport-teacher.git`

项目名称：`jsd2206-csmall-passport-teacher`

Group：`cn.tedu`

Artifact：`csmall-passport`

Package：`cn.tedu.csmall.passport`

当项目创建成功后，需要先创建本项目所需的数据库`mall_ams`，然后导入SQL脚本，以创建数据表，及导入测试使用的数据，并在IntelliJ IDEA中配置Database面板。

接下来，调整当前项目的`pom.xml`文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!-- 模块版本 -->
    <modelVersion>4.0.0</modelVersion>

    <!-- 父级项目 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.9</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <!-- 当前项目的参数 -->
    <groupId>cn.tedu</groupId>
    <artifactId>csmall-passport</artifactId>
    <version>0.0.1</version>

    <!-- 属性配置 -->
    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!-- 当前项目使用的依赖项 -->
    <!-- scope > test：此依赖项仅用于测试，则此依赖项不会被打包，并且这些依赖项在src/main中不可用 -->
    <!-- scope > runtime：此依赖项仅在运行时需要，即编写代码时并不需要此依赖项 -->
    <!-- scope > provided：在执行（启动项目，或编译源代码）时，需要运行环境保证此依赖项一定是存在的 -->
    <dependencies>
        <!-- Spring Boot的Web依赖项，包含基础依赖项 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- Mybatis整合Spring Boot的依赖项 -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.2</version>
        </dependency>
        <!-- MySQL的依赖项 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!-- Lombok的依赖项，主要用于简化POJO类的编写 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.20</version>
            <scope>provided</scope>
        </dependency>
        <!-- Spring Boot测试的依赖项 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

当添加以上依赖项后，项目暂时无法正常启动，也无法通过任何测试，必须配置连接数据库的相关参数！

先将`application.properties`重命名为`application.yml`，并创建出`application-dev.yml`，在`application-dev.yml`中配置：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mall_ams?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Chongqing
    username: root
    password: root
```

并且，在`application.yml`激活`application-dev.yml`中的配置：

```yaml
spring:
  profiles:
    active: dev
```

完成后，可以执行默认的测试类下的`contextLoads()`测试方法，应该可以通过测试。然后，在测试类中添加：

```java
@Autowired
DataSource dataSource;

@Test
void testGetConnection() throws Exception {
    dataSource.getConnection();
    System.out.println("当前配置可以成功的连接到数据库！");
}
```

# 20. 添加管理员--Mapper层

使用Mybatis实现数据库编程时，必须要编写的配置：

- 在配置类上使用`@MapperScan`配置接口文件的根包
- 在配置文件中使用`mybatis.mapper-locations`属性配置XML文件的位置

则先在根包下创建`config.MybatisConfiguration`类，在类上添加`@Configuration`注解，并在类上添加`@MapperScan("cn.tedu.csmall.passport.mapper")`，例如：

```java
package cn.tedu.csmall.passport.config;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan("cn.tedu.csmall.passport.mapper")
public class MybatisConfiguration {
}
```

在`application.yml`中添加配置：

```yaml
mybatis:
  mapper-locations: classpath:mapper/*.xml
```

接下来，在根包下创建`pojo.entity.Admin`实体类：

```java
package cn.tedu.csmall.passport.pojo.entity;

import lombok.Data;

import java.io.Serializable;
import java.time.LocalDateTime;

/**
 * 管理员的实体类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Data
public class Admin implements Serializable {

    /**
     * 数据id
     */
    private Long id;

    /**
     * 用户名
     */
    private String username;

    /**
     * 密码（密文）
     */
    private String password;

    /**
     * 昵称
     */
    private String nickname;

    /**
     * 头像URL
     */
    private String avatar;

    /**
     * 手机号码
     */
    private String phone;

    /**
     * 电子邮箱
     */
    private String email;

    /**
     * 描述
     */
    private String description;

    /**
     * 是否启用，1=启用，0=未启用
     */
    private Integer enable;

    /**
     * 最后登录IP地址（冗余）
     */
    private String lastLoginIp;

    /**
     * 累计登录次数（冗余）
     */
    private Integer loginCount;

    /**
     * 最后登录时间（冗余）
     */
    private LocalDateTime gmtLastLogin;

    /**
     * 数据创建时间
     */
    private LocalDateTime gmtCreate;

    /**
     * 数据最后修改时间
     */
    private LocalDateTime gmtModified;

}
```

在根包下创建`mapper.AdminMapper`接口，并在接口中添加抽象方法：

```java
/**
 * 处理管理员数据的Mapper接口
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Repository
public interface AdminMapper {

    /**
     * 插入管理员数据
     *
     * @param admin 管理员数据
     * @return 受影响的行数
     */
    int insert(Admin admin);
    
}
```

在`src/main/resources`下创建`mapper`文件夹，并在`mapper`文件夹下粘贴得到`AdminMapper.xml`，配置以上抽象方法对应的SQL语句：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.tedu.csmall.passport.mapper.AdminMapper">

    <!-- int insert(Admin admin); -->
    <insert id="insert" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO ams_admin (
            username, password, nickname, avatar, phone,
            email, description, enable, last_login_ip, login_count,
            gmt_last_login
        ) VALUES (
            #{username}, #{password}, #{nickname}, #{avatar}, #{phone},
            #{email}, #{description}, #{enable}, #{lastLoginIp}, #{loginCount},
            #{gmtLastLogin}
        )
    </insert>
    
</mapper>
```

完成后，在`src/test/java`的根包下创建`mapper.AdminMapperTests`测试类，编写并执行测试：

```java
@Slf4j
@SpringBootTest
public class AdminMapperTests {

    @Autowired
    AdminMapper mapper;

    @Test
    void testInsert() {
        Admin admin = new Admin();
        admin.setUsername("wangkejing");
        admin.setPassword("123456");
        admin.setPhone("13800138001");
        admin.setEmail("wangkejing@baidu.com");

        log.debug("插入数据之前，参数：{}", admin);
        int rows = mapper.insert(admin);
        log.debug("插入数据完成，受影响的行数：{}", rows);
        log.debug("插入数据之后，参数：{}", admin);
    }
    
}
```

在编写Service层之前，可以先分析一下“添加管理员”的业务规则，目前，应该设置的规则有：

- 用户名不允许重复
- 手机号码不允许重复
- 电子邮箱不允许重复
- 其它规则，暂不考虑

以上“不允许重复”的规则，都可以通过统计查询来实现，例如：

```mysql
select count(*) from ams_admin where username=?;
select count(*) from ams_admin where phone=?;
select count(*) from ams_admin where email=?;
```

则在`AdminMapper`接口中添加：

```java
/**
 * 根据用户名统计管理员的数量
 *
 * @param username 用户名
 * @return 匹配用户名的管理员的数据
 */
int countByUsername(String username);

/**
 * 根据手机号码统计管理员的数量
 *
 * @param phone 手机号码
 * @return 匹配手机号码的管理员的数据
 */
int countByPhone(String phone);

/**
 * 根据电子邮箱统计管理员的数量
 *
 * @param email 电子邮箱
 * @return 匹配电子邮箱的管理员的数据
 */
int countByEmail(String email);
```

并在`AdminMapper.xml`中配置以上3个抽象方法对应的3个`<select>`标签：

```xml
<!-- int countByUsername(String username); -->
<select id="countByUsername" resultType="int">
    SELECT count(*) FROM ams_admin WHERE username=#{username}
</select>

<!-- int countByPhone(String phone); -->
<select id="countByPhone" resultType="int">
    SELECT count(*) FROM ams_admin WHERE phone=#{phone}
</select>

<!-- int countByEmail(String email); -->
<select id="countByEmail" resultType="int">
    SELECT count(*) FROM ams_admin WHERE email=#{email}
</select>
```

最后，在`AdminMapperTests`中编写并执行测试：

```java
@Test
void testCountByUsername() {
    String username = "wangkejing";
    int count = mapper.countByUsername(username);
    log.debug("根据用户名【{}】统计管理员账号的数量：{}", username, count);
}

@Test
void testCountByPhone() {
    String phone = "13800138001";
    int count = mapper.countByPhone(phone);
    log.debug("根据手机号码【{}】统计管理员账号的数量：{}", phone, count);
}

@Test
void testCountByEmail() {
    String email = "wangkejing@baidu.com";
    int count = mapper.countByEmail(email);
    log.debug("根据电子邮箱【{}】统计管理员账号的数量：{}", email, count);
}
```

# 21. 添加管理员--Service层

在根包下创建`pojo.dto.AdminAddNewDTO`类，封装需要由客户端提交的参数：

```java
package cn.tedu.csmall.passport.pojo.dto;

import lombok.Data;

import java.io.Serializable;

/**
 * 添加管理员的DTO类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Data
public class AdminAddNewDTO implements Serializable {

    /**
     * 用户名
     */
    private String username;

    /**
     * 密码（原文）
     */
    private String password;

    /**
     * 昵称
     */
    private String nickname;

    /**
     * 头像URL
     */
    private String avatar;

    /**
     * 手机号码
     */
    private String phone;

    /**
     * 电子邮箱
     */
    private String email;

    /**
     * 描述
     */
    private String description;

    /**
     * 是否启用，1=启用，0=未启用
     */
    private Integer enable;

}
```

在根包下创建`service.IAdminService`接口，并在接口中添加抽象方法：

```java
public interface IAdminService {
    void addNew(AdminAddNewDTO adminAddNewDTO);
}
```

在根包下创建`service.impl.AdminServiceImpl`类，实现以上接口，并在类上添加`@Service`注解，在类中自动装配`AdminMapper`对象：

```java
@Service
public class AdminServiceImpl implements IAdminService {
    @Autowired
    AdminMapper adminMapper;
}
```

在实现接口中的方法之前，需要将前序项目（`csmall-product`）的`ServiceCode`和`ServiceException`复制到当前项目相同的位置。

并在以上实现类中实现接口中的方法：

```java
public void addNew(AdminAddNewDTO adminAddNewDTO) {
    // 从参数对象中获取username
    // 调用adminMapper的countByUsername()方法执行统计查询
    // 判断统计结果是否不等于0
    // 是：抛出异常
    
    // 从参数对象中获取手机号码
    // 调用adminMapper的countByPhone()方法执行统计查询
    // 判断统计结果是否不等于0
    // 是：抛出异常
    
    // 从参数对象中获取电子邮箱
    // 调用adminMapper的countByEmail()方法执行统计查询
    // 判断统计结果是否不等于0
    // 是：抛出异常
    
    // 创建Admin对象
    // 通过BeanUtils.copyProperties()方法将参数对象的各属性值复制到Admin对象中
    // 调用adminMapper的insert()方法插入数据
}
```

实际代码为：

```java
package cn.tedu.csmall.passport.service.impl;

import cn.tedu.csmall.passport.ex.ServiceException;
import cn.tedu.csmall.passport.mapper.AdminMapper;
import cn.tedu.csmall.passport.pojo.dto.AdminAddNewDTO;
import cn.tedu.csmall.passport.pojo.entity.Admin;
import cn.tedu.csmall.passport.service.IAdminService;
import cn.tedu.csmall.passport.web.ServiceCode;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 处理管理员数据的业务实现类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Slf4j
@Service
public class AdminServiceImpl implements IAdminService {

    @Autowired
    AdminMapper adminMapper;

    public AdminServiceImpl() {
        System.out.println("创建业务实现类：AdminServiceImpl");
    }

    @Override
    public void addNew(AdminAddNewDTO adminAddNewDTO) {
        log.debug("开始处理【添加管理员】的业务，参数：{}", adminAddNewDTO);
        log.debug("即将检查用户名是否被占用……");
        {
            // 从参数对象中获取username
            String username = adminAddNewDTO.getUsername();
            // 调用adminMapper的countByUsername()方法执行统计查询
            int count = adminMapper.countByUsername(username);
            // 判断统计结果是否不等于0
            if (count != 0) {
                // 是：抛出异常
                String message = "添加管理员失败，用户名【" + username + "】已经被占用！";
                log.debug(message);
                throw new ServiceException(ServiceCode.ERR_CONFLICT, message);
            }
        }

        log.debug("即将检查手机号码是否被占用……");
        {
            // 从参数对象中获取手机号码
            String phone = adminAddNewDTO.getPhone();
            // 调用adminMapper的countByPhone()方法执行统计查询
            int count = adminMapper.countByPhone(phone);
            // 判断统计结果是否不等于0
            if (count != 0) {
                // 是：抛出异常
                String message = "添加管理员失败，手机号码【" + phone + "】已经被占用！";
                log.debug(message);
                throw new ServiceException(ServiceCode.ERR_CONFLICT, message);
            }
        }

        log.debug("即将检查电子邮箱是否被占用……");
        {
            // 从参数对象中获取电子邮箱
            String email = adminAddNewDTO.getEmail();
            // 调用adminMapper的countByEmail()方法执行统计查询
            int count = adminMapper.countByEmail(email);
            // 判断统计结果是否不等于0
            if (count != 0) {
                // 是：抛出异常
                String message = "添加管理员失败，电子邮箱【" + email + "】已经被占用！";
                log.debug(message);
                throw new ServiceException(ServiceCode.ERR_CONFLICT, message);
            }
        }

        // 创建Admin对象
        Admin admin = new Admin();
        // 通过BeanUtils.copyProperties()方法将参数对象的各属性值复制到Admin对象中
        BeanUtils.copyProperties(adminAddNewDTO, admin);
        // TODO 从Admin对象中取出密码，进行加密处理，并将密文封装回Admin对象中
        // 补全Admin对象中的属性值：loginCount >>> 0
        admin.setLoginCount(0);
        // 调用adminMapper的insert()方法插入数据
        log.debug("即将插入管理员数据，参数：{}", admin);
        adminMapper.insert(admin);
    }

}
```

完成后，在`src/test/java`下的根包下创建`service.AdminServiceTests`测试类，编写并执行测试（在测试方法中记得使用`try...catch`）：

```java
package cn.tedu.csmall.passport.service;

import cn.tedu.csmall.passport.ex.ServiceException;
import cn.tedu.csmall.passport.pojo.dto.AdminAddNewDTO;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@Slf4j
@SpringBootTest
public class AdminServiceTests {

    @Autowired
    IAdminService service;

    @Test
    void testAddNew() {
        AdminAddNewDTO adminAddNewDTO = new AdminAddNewDTO();
        adminAddNewDTO.setUsername("wangkejing3");
        adminAddNewDTO.setPassword("123456");
        adminAddNewDTO.setPhone("13800138003");
        adminAddNewDTO.setEmail("wangkejing3@baidu.com");

        try {
            service.addNew(adminAddNewDTO);
            log.debug("添加管理员成功！");
        } catch (ServiceException e) {
            log.debug("{}", e.getMessage());
        }
    }
}
```

# 22. 添加管理员--Controller层

由于非常推荐自行指定服务端口，则在`application-dev.yml`中添加配置：

```yaml
# 服务端口
server:
  port: 9081
```

将前序项目中的`JsonResult`类复制到当前项目同样位置。

在根包下创建`controller.AdminController`类，添加`@RestController`注解和`@RequestMapping("/admins")`注解，并在类中添加处理请求的方法：

```java
package cn.tedu.csmall.passport.controller;

import cn.tedu.csmall.passport.pojo.dto.AdminAddNewDTO;
import cn.tedu.csmall.passport.service.IAdminService;
import cn.tedu.csmall.passport.web.JsonResult;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
@RequestMapping("/admins")
public class AdminController {

    @Autowired
    IAdminService adminService;

    public AdminController() {
        log.info("创建控制器类：AdminController");
    }

    // http://localhost:9081/admins/add-new?username=aa&phone=bb&email=cc
    @RequestMapping("/add-new")
    public JsonResult addNew(AdminAddNewDTO adminAddNewDTO) {
        log.debug("开始处理【添加管理员】的请求，参数：{}", adminAddNewDTO);
        adminService.addNew(adminAddNewDTO);
        return JsonResult.ok();
    }

}
```

在`application.yml`中添加配置，使得为`null`的属性不会显示在响应的JSON数据中：

```yaml
spring:
  # Jackson框架相关配置
  jackson:
    # JSON结果中是否包含为null的属性的默认配置
    default-property-inclusion: non_null
```

【处理异常】

完成后，重启项目，可以通过 http://localhost:9081/admins/add-new?username=aa&phone=bb&email=cc 测试访问。

# 23. 关于Knife4j框架

Knife4j是一款基于Swagger 2的在线API文档框架。

使用Knife4j，需要：

- 添加Knife4j的依赖
  - 当前建议使用的Knife4j版本，只适用于Spring Boot 2.6以下版本，不含Spring Boot 2.6
- 在主配置文件（`application.yml`）中开启Knife4j的增强模式
  - 必须在主配置文件中进行配置，不要配置在个性化配置文件中
- 添加Knife4j的配置类，进行必要的配置
  - 必须指定控制器的包

关于依赖项的代码：

```xml
<!-- Knife4j Spring Boot：在线API -->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>2.0.9</version>
</dependency>
```

在`application.yml`中添加配置：

```yaml
knife4j:
  enable: true
```

在根包下创建`config.Knife4jConfiguration`配置类：

```java
package cn.tedu.csmall.passport.config;

import com.github.xiaoymin.knife4j.spring.extension.OpenApiExtensionResolver;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2WebMvc;

/**
 * Knife4j配置类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Slf4j
@Configuration
@EnableSwagger2WebMvc
public class Knife4jConfiguration {

    /**
     * 【重要】指定Controller包路径
     */
    private String basePackage = "cn.tedu.csmall.passport.controller";
    /**
     * 分组名称
     */
    private String groupName = "passport";
    /**
     * 主机名
     */
    private String host = "http://java.tedu.cn";
    /**
     * 标题
     */
    private String title = "酷鲨商城在线API文档--管理员管理";
    /**
     * 简介
     */
    private String description = "酷鲨商城在线API文档--管理员管理";
    /**
     * 服务条款URL
     */
    private String termsOfServiceUrl = "http://www.apache.org/licenses/LICENSE-2.0";
    /**
     * 联系人
     */
    private String contactName = "Java教学研发部";
    /**
     * 联系网址
     */
    private String contactUrl = "http://java.tedu.cn";
    /**
     * 联系邮箱
     */
    private String contactEmail = "java@tedu.cn";
    /**
     * 版本号
     */
    private String version = "1.0.0";

    @Autowired
    private OpenApiExtensionResolver openApiExtensionResolver;

    public Knife4jConfiguration() {
        log.debug("加载配置类：Knife4jConfiguration");
    }

    @Bean
    public Docket docket() {
        String groupName = "1.0.0";
        Docket docket = new Docket(DocumentationType.SWAGGER_2)
                .host(host)
                .apiInfo(apiInfo())
                .groupName(groupName)
                .select()
                .apis(RequestHandlerSelectors.basePackage(basePackage))
                .paths(PathSelectors.any())
                .build()
                .extensions(openApiExtensionResolver.buildExtensions(groupName));
        return docket;
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title(title)
                .description(description)
                .termsOfServiceUrl(termsOfServiceUrl)
                .contact(new Contact(contactName, contactUrl, contactEmail))
                .version(version)
                .build();
    }

}
```

完成后，启动项目，通过 http://localhost:9081/doc.html 即可访问在线API文档！

关于Knife4j的在线API文档，可以通过一系列注解来配置此文件的显示：

- `@Api`：添加在控制器类上，通过此注解的`tags`属性，可以指定模块名称，并且，在指定名称时，建议在名称前添加数字作为序号，Knife4j会根据这些数字将各模块升序排列，例如：

  ```java
  @Api(tags = "01. 管理员管理模块")
  ```

- `@ApiOpearation`：添加在控制器类中处理请求的方法上，通过此注解的`value`属性，可以指定业务/请求资源的名称，例如：

  ```java
  @ApiOperation("添加管理员")
  ```

- `@ApiOperationSupport`：添加在控制器类中处理请求的方法上，通过此注解的`order`属性（`int`），可以指定排序序号，Knife4j会根据这些数字将各业务/请求资源升序排列，例如：

  ```java
  @ApiOperationSupport(order = 100)
  ```

# 24. 关于`@RequestBody`

在Spring MVC框架中，在处理请求的方法的参数前：

- 当添加了`@RequestBody`注解，则客户端提交的请求参数**必须**是对象格式的，例如：

  ```json
  {
      "name": "小米11的相册",
      "description": "小米11的相册的简介",
      "sort": 88
  }
  ```

  如果客户端提交的数据不是对象，而是FormData格式的，在接收到请求时将报错：

  ```
  org.springframework.web.HttpMediaTypeNotSupportedException: Content type 'application/x-www-form-urlencoded;charset=UTF-8' not supported
  ```

- 当没有添加`@RequestBody`注解，则客户端提交的请求参数**必须**是FormData格式的，例如：

  ```
  name=小米11的相册&description=小米11的相册的简介&sort=88
  ```

  如果客户端提交的数据不是FormData格式的，而是对象，则无法接收到参数（不会报错，控制器中各参数值为`null`）

另外，Knife4j框架的调试界面中，如果是对象格式的参数（使用了`@RequestBody`），将不会显示各请求参数的输入框，而是提供一个JSON字符串供编辑，如果是FormData格式的参数（没有使用`@RequestBody`），则会显示各请求参数对应的输入框。

通常，更建议使用FormData格式的请求参数！则在控制器处理请求的方法的参数上不需要添加`@RequestBody`注解！

在Vue脚手架项目中，为了更便捷的使用FormData格式的请求参数，可以在项目中使用`qs`框架，此框架的工具可以轻松的将JavaScript对象转换成FormData格式！

则在前端的Vue脚手架项目中，先安装`qs`：

```
npm i qs -S
```

然后，在`main.js`中添加配置：

```javascript
import qs from 'qs';

Vue.prototype.qs = qs;
```

最后，在提交请求之前，可以将对象转换成FormData格式，例如：

```javascript
let formData = this.qs.stringify(this.ruleForm);
console.log('formData：' + formData);
```













