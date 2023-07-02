# 37. 基于Spring JDBC框架的事务管理（续）

在执行数据访问操作时，数据库有一个“自动提交”的机制。

事务的本质是会先将“自动提交”关闭，当业务方法执行结束之后，再一次性“提交”。

在事务中，涉及几个概念：

- 开启事务：BEGIN
- 提交事务：COMMIT
- 回滚事务：ROLLBACK

在基于Spring JDBC的程序设计中，通过`@Transactional`注解即可使得业务方法是事务性的，其实现过程大致是：

```
开启事务
try {
    执行业务方法
    提交事务
} catch (RuntimeException e) {
	回滚事务
}
```

可以看到，Spring JDBC框架在处理事务时，**默认**将根据`RuntimeException`进行回滚！

> 提示：可以配置`@Transactional`注解的`rollbackFor`或`rollbackForClassName`属性来指定回滚的异常类型，即根据其它类型的异常来回滚，例如：
>
> ```java
> @Transactional(rollbackFor = {IOException.class})
> ```
>
> ```java
> @Transactional(rollbackForClassName = {}"java.io.IOException"})
> ```
>
> 另外，还可以通过`noRollbackFor`或`noRollbackForClassName`属性用于指定不回滚的异常！

建议在业务方法中执行了任何增、删、改操作后，都获取受影响的行数，并判断此值是否符合预期，如果不符合，应该及时抛出`RuntimeException`或其子孙类异常！

补充：Spring JDBC框架在实现事务管理时，使用到了Spring AOP技术及基于接口的代理模式，由于使用了基于接口的代理模式，所以，如果将`@Transactional`注解添加在**实现类中自定义的方法**（不是重写的接口中的抽象方法）上，是错误的做法！

最后，可自行补充相关知识点：事务的ACID特性，事务的传播，事务的隔离。

# 38. 完善删除管理员

由于添加管理员时，在`ams_admin_role`表中插入了数据，在删除管理员时，也应该将`ams_admin_role`表中对应的数据删除！

删除`ams_admin_role`表中某管理员的数据需要执行的SQL语句大致是：

```mysql
DELETE FROM ams_admin_role WHERE admin_id=?
```

则在`AdminRoleMapper`接口中添加抽象方法：

```java
/**
 * 根据管理员id删除管理员与角色的关联数据
 *
 * @param adminId 管理员id
 * @return 受影响的行数
 */
int deleteByAdminId(Long adminId);
```

并在`AdminRoleMapper.xml`中配置SQL语句：

```xml
<!-- int deleteByAdminId(Long adminId); -->
<delete id="deleteByAdminId">
    DELETE FROM ams_admin_role WHERE admin_id=#{adminId}
</delete>
```

完成后，在`AdminRoleMapperTests`中编写并执行测试：

```java
@Test
void testDeleteByAdminId() {
    Long adminId = 7L;
    int rows = mapper.deleteByAdminId(adminId);
    log.debug("删除数据完成！受影响的行数={}", rows);
}
```

当Mapper层的开发完成后，在`AdminServiceImpl`中的`delete()`方法最后补充调用以上方法即可：

```java
// 执行删除此管理员与角色的关联数据
rows = adminRoleMapper.deleteByAdminId(id);
if (rows < 1) {
    String message = "删除管理员失败，服务器忙，请稍后再次尝试！";
    log.warn(message);
    throw new ServiceException(ServiceCode.ERR_DELETE, message);
}
```

完成后，删除管理员的功能将暂时全部完成！

**但是，需要注意：不要使用错误的测试数据进行测试！**

# 39. 显示角色列表

在“添加管理员”的界面中，需要将角色列表显示出来，则用户（软件的使用者）可以在界面上选择“添加管理员”时此管理员的角色。

**关于“显示角色列表”的Mapper层**

需要执行的SQL语句大致是：

```mysql
SELECT * FROM ams_role ORDER BY sort DESC, id
```

则需要在`pojo.vo`包中创建`RoleListItemVO`类：

```java
package cn.tedu.csmall.passport.pojo.vo;

import lombok.Data;

import java.io.Serializable;

/**
 * 角色的列表项的VO类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Data
public class RoleListItemVO implements Serializable {

    private Long id;
    private String name;
    private String description;
    private Integer sort;

}
```

然后，在`mapper`包中创建`RoleMapper`接口，并添加抽象方法：

```java
package cn.tedu.csmall.passport.mapper;

import cn.tedu.csmall.passport.pojo.vo.RoleListItemVO;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * 处理角色数据的Mapper接口
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Repository
public interface RoleMapper {

    /**
     * 查询角色列表
     *
     * @return 角色列表
     */
    List<RoleListItemVO> list();
    
}
```

然后，在`src/main/resources/mapper`文件夹下，通过复制粘贴得到`RoleMapper.xml`文件，在此文件中配置以上抽象方法映射的SQL语句：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.tedu.csmall.passport.mapper.RoleMapper">

    <!-- List<RoleListItemVO> list(); -->
    <select id="list" resultMap="ListResultMap">
        SELECT
            <include refid="ListQueryFields" />
        FROM
            ams_role
        ORDER BY
            sort DESC, id
    </select>

    <sql id="ListQueryFields">
        <if test="true">
            id, name, description, sort
        </if>
    </sql>

    <resultMap id="ListResultMap" type="cn.tedu.csmall.passport.pojo.vo.RoleListItemVO">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="description" property="description"/>
        <result column="sort" property="sort"/>
    </resultMap>

</mapper>
```

完成后，在`src/test/java`下的根包下创建`mapper.RoleMapperTests`测试类，在此类中编写并执行测试：

```java
package cn.tedu.csmall.passport.mapper;

import cn.tedu.csmall.passport.pojo.entity.Admin;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@Slf4j
@SpringBootTest
public class RoleMapperTests {

    @Autowired
    RoleMapper mapper;

    @Test
    void testList() {
        List<?> list = mapper.list();
        System.out.println("查询列表完成，列表中的数据的数量=" + list.size());
        for (Object item : list) {
            System.out.println(item);
        }
    }

}
```

**关于“显示角色列表”的Service层**

在`service`包下创建`IRoleService`接口，并在接口中添加抽象方法：

```java
/**
 * 处理角色数据的业务接口
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Transactional
public interface IRoleService {

    /**
     * 查询角色列表
     *
     * @return 角色列表
     */
    List<RoleListItemVO> list();
    
}
```

然后，在`service.impl`包下创建`RoleServiceImpl`实现类，实现以上接口，并在类上添加`@Service`注解，在类中声明`RoleMapper`属性并自动装配：

```java
/**
 * 处理角色数据的业务实现类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Slf4j
@Service
public class RoleServiceImpl implements IRoleService {
    
    @Autowired
    RoleMapper roleMapper;

    @Override
    public List<RoleListItemVO> list() {
        log.debug("开始处理【查询角色列表】的业务");
        return roleMapper.list();
    }
    
}
```

完成后，在`src/test/java`下的根包下创建`service.RoleServiceTests`测试类，在此类中编写并执行测试：

```java
package cn.tedu.csmall.passport.service;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@Slf4j
@SpringBootTest
public class RoleServiceTests {

    @Autowired
    IRoleService service;

    @Test
    void testList() {
        List<?> list = service.list();
        System.out.println("查询列表完成，列表中的数据的数量=" + list.size());
        for (Object item : list) {
            System.out.println(item);
        }
    }

}
```

**关于“显示角色列表”的Controller层**

在`controller`包下创建`RoleController`类，在类上添加`@RestController`和`@RequestMapping("/roles")`注解，在类中声明并自动装配`IRoleService`属性，然后，在类中添加处理请求的方法：

```java
package cn.tedu.csmall.passport.controller;

import cn.tedu.csmall.passport.pojo.vo.RoleListItemVO;
import cn.tedu.csmall.passport.service.IRoleService;
import cn.tedu.csmall.passport.web.JsonResult;
import com.github.xiaoymin.knife4j.annotations.ApiOperationSupport;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@Slf4j
@Api(tags = "02. 角色管理模块")
@RestController
@RequestMapping("/roles")
public class RoleController {

    @Autowired
    IRoleService roleService;

    public RoleController() {
        log.info("创建控制器类：RoleController");
    }

    // http://localhost:9081/roles
    @ApiOperation("查询角色列表")
    @ApiOperationSupport(order = 420)
    @GetMapping("")
    public JsonResult<List<RoleListItemVO>> list() {
        log.debug("开始处理【查询角色列表】的请求");
        return JsonResult.ok(roleService.list());
    }

}
```

完成后，重启项目，可以通过在线API文档测试访问。

# 40. 启动或禁用管理员

管理的启用状态是通过数据表中的`enable`字段的值来控制的，所以，启用、禁用功能的本质是修改此字段的值！

在开发功能时，应该在Mapper层开发一个能够修改所有字段的值的功能，则此功能可以适用于当前表的任何修改功能！

**关于Mapper层**

先在`AdminMapper`接口中添加抽象方法：

```java
/**
 * 根据管理员id修改管理员的数据
 *
 * @param admin 封装了管理员id和新的数据的对象
 * @return 受影响的行数
 */
int updateById(Admin admin);
```

然后在`AdminMapper.xml`中配置SQL：

```xml
<!-- int updateById(Admin admin); -->
<update id="updateById">
    UPDATE ams_admin
    <set>
        <if test="username != null">
            username=#{username},
        </if>
        <if test="password != null">
            password=#{password},
        </if>
        <if test="nickname != null">
            nickname=#{nickname},
        </if>
        <if test="avatar != null">
            avatar=#{avatar},
        </if>
        <if test="phone != null">
            phone=#{phone},
        </if>
        <if test="email != null">
            email=#{email},
        </if>
        <if test="description != null">
            description=#{description},
        </if>
        <if test="enable != null">
            enable=#{enable},
        </if>
        <if test="lastLoginIp != null">
            last_login_ip=#{lastLoginIp},
        </if>
        <if test="loginCount != null">
            login_count=#{loginCount},
        </if>
        <if test="gmtLastLogin != null">
            gmt_last_login=#{gmtLastLogin},
        </if>
    </set>
    WHERE id=#{id}
</update>
```

完成后，在`AdminMapperTests`中编写并执行测试：

```java
@Test
void testUpdateById() {
    Admin data = new Admin();
    data.setId(1L);
    data.setUsername("新的测试数据名称");

    int rows = mapper.updateById(data);
    System.out.println("修改数据完成，受影响的行数=" + rows);
}
```

**关于Service层**

在`IAdminService`接口中添加抽象方法，抽象方法可以是：

```java
void updateEnableById(Long id, Integer enable);
```

提示：也可以将以上方法的2个参数进行封装。

但是，更建议声明为：

```java
/**
 * 启用管理员
 *
 * @param id 管理员的id
 */
void setEnable(Long id);

/**
 * 禁用管理员
 *
 * @param id 管理员的id
 */
void setDisable(Long id);
```

然后，在`AdminServiceImpl`中实现：

```java
@Override
public void setEnable(Long id) {
    updateEnableById(id, 1);
}

@Override
public void setDisable(Long id) {
    updateEnableById(id, 0);
}

private void updateEnableById(Long id, Integer enable) {
    String[] tips = {"禁用", "启用"};
    log.debug("开始处理【{}管理员】的业务，参数：{}", tips[enable], id);
    // 检查尝试访问的数据是否存在
    AdminStandardVO queryResult = adminMapper.getStandardById(id);
    if (queryResult == null) {
        String message = tips[enable] + "管理员失败，尝试访问的数据不存在！";
        log.debug(message);
        throw new ServiceException(ServiceCode.ERR_NOT_FOUND, message);
    }

    // 判断状态是否冲突（当前已经是目标状态）
    if (queryResult.getEnable() == enable) {
        String message = tips[enable] + "管理员失败，管理员账号已经处于" + tips[enable] + "状态！";
        log.debug(message);
        throw new ServiceException(ServiceCode.ERR_CONFLICT, message);
    }

    // 准备执行更新
    Admin admin = new Admin();
    admin.setId(id);
    admin.setEnable(enable);
    int rows = adminMapper.updateById(admin);
    if (rows != 1) {
        String message = tips[enable] + "管理员失败，服务器忙，请稍后再次尝试！";
        log.warn(message);
        throw new ServiceException(ServiceCode.ERR_UPDATE, message);
    }
}
```

完成后，在`AdminServiceTests`中编写并执行测试：

```java
@Test
void testSetEnable() {
    Long id = 500L;

    try {
        service.setEnable(id);
        System.out.println("启用管理员成功！");
    } catch (ServiceException e) {
        System.out.println(e.getMessage());
    }
}

@Test
void testSetDisable() {
    Long id = 5L;

    try {
        service.setDisable(id);
        System.out.println("禁用管理员成功！");
    } catch (ServiceException e) {
        System.out.println(e.getMessage());
    }
}
```

**关于Controller层**









