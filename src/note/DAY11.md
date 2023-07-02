# 32. 删除管理员--Mapper层

删除管理员需要执行的SQL大致是：

```mysql
delete from ams_admin where id=?
```

在删除之前，还应该检查数据是否存在，可以通过以下SQL查询来实现检查：

```mysql
select count(*) from ams_admin where id=?
```

```mysql
select * from ams_admin where id=?
```

首先，在`pojo.vo`包下创建`AdminStandardVO`类：

```java
package cn.tedu.csmall.passport.pojo.vo;

import lombok.Data;

import java.io.Serializable;
import java.time.LocalDateTime;

/**
 * 管理员的标准VO类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Data
public class AdminStandardVO implements Serializable {

    /**
     * 数据id
     */
    private Long id;

    /**
     * 用户名
     */
    private String username;

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

}
```

然后，在`AdminMapper.java`中添加：

```java
/**
 * 根据id删除管理员数据
 *
 * @param id 管理员id
 * @return 受影响的行数
 */
int deleteById(Long id);

/**
 * 查询管理员列表
 *
 * @return 管理员列表
 */
List<AdminListItemVO> list();
```

在`AdminMapper.xml`中配置：

```xml
<!-- int deleteById(Long id); -->
<delete id="deleteById">
    DELETE FROM ams_admin WHERE id=#{id}
</delete>

<!-- AdminStandardVO getStandardById(Long id); -->
<select id="getStandardById" resultMap="StandardResultMap">
    SELECT
        <include refid="StandardQueryFields" />
    FROM
        ams_admin
    WHERE
        id=#{id}
</select>

<!-- List<AdminListItemVO> list(); -->
<select id="list" resultMap="ListResultMap">
    SELECT
        <include refid="ListQueryFields" />
    FROM
        ams_admin
    ORDER BY
        enable DESC, id
</select>

<sql id="StandardQueryFields">
    <if test="true">
        id, username, nickname, avatar, phone,
        email, description, enable, last_login_ip, login_count,
        gmt_last_login
    </if>
</sql>

<sql id="ListQueryFields">
    <if test="true">
        id, username, nickname, avatar, phone,
        email, description, enable, last_login_ip, login_count,
        gmt_last_login
    </if>
</sql>

<resultMap id="StandardResultMap" type="cn.tedu.csmall.passport.pojo.vo.AdminStandardVO">
    <id column="id" property="id"/>
    <result column="username" property="username"/>
    <result column="nickname" property="nickname"/>
    <result column="avatar" property="avatar"/>
    <result column="phone" property="phone"/>
    <result column="email" property="email"/>
    <result column="description" property="description"/>
    <result column="enable" property="enable"/>
    <result column="last_login_ip" property="lastLoginIp"/>
    <result column="login_count" property="loginCount"/>
    <result column="gmt_last_login" property="gmtLastLogin"/>
</resultMap>
```

完成后，在`AdminMapperTests`中编写并执行测试：

```java
@Test
void testDeleteById() {
    Long id = 11L;
    int rows = mapper.deleteById(id);
    System.out.println("删除数据完成，受影响的行数=" + rows);
}

@Test
void testGetStandardById() {
    Long id = 1L;
    Object result = mapper.getStandardById(id);
    System.out.println("根据id=" + id + "查询标准信息完成，结果=" + result);
}
```

# 33. 删除管理员--Service层

在`IAdminService`接口中添加抽象方法：

```java
/**
 * 删除管理员
 *
 * @param id 尝试删除的管理员的id
 */
void delete(Long id);
```

在`AdminServiceImpl`中实现：

```java
@Override
public void delete(Long id) {
    log.debug("开始处理【删除管理员】的业务，参数：{}", id);
    // 根据参数id查询管理员数据
    AdminStandardVO queryResult = adminMapper.getStandardById(id);
    // 判断管理员数据是否不存在
    if (queryResult == null) {
        String message = "删除管理员失败，尝试访问的数据不存在！";
        log.warn(message);
        throw new ServiceException(ServiceCode.ERR_NOT_FOUND, message);
    }

    // 执行删除管理员
    adminMapper.deleteById(id);
}
```

在`AdminServiceTests`中编写并执行测试：

```java
@Test
void testDelete() {
    Long id = 9L;

    try {
        service.delete(id);
        System.out.println("删除成功！");
    } catch (ServiceException e) {
        System.out.println(e.getMessage());
    }
}
```

# 34. 删除管理员--Controller层

在`AdminController`中添加处理请求的方法：

```java
// http://localhost:8080/admins/9527/delete
@ApiOperation("删除管理员")
@ApiOperationSupport(order = 200)
@ApiImplicitParam(name = "id", value = "管理员id", required = true, dataType = "long")
@PostMapping("/{id:[0-9]+}/delete")
public JsonResult<Void> delete(@PathVariable Long id) {
    log.debug("开始处理【删除管理员】的请求，参数：{}", id);
    adminService.delete(id);
    return JsonResult.ok();
}
```

完成后，重启项目，通过Knife4j在线API文档进行测试访问。

# 35. 删除管理员--前端

# 36. 完善添加管理员

添加管理员时，还需要为管理员分配某些（某种）角色，进而使得该管理员具备某些操作权限！

为管理分配角色的本质是向`ams_admin_role`这张表中插入数据！同时，需要注意，每个管理员账号可能对应多种角色，所以，需要执行的SQL语句大致是：

```mysql
insert into ams_admin_role (admin_id, role_id) values (?, ?), (?, ?) ... (?, ?);
```

在`pojo.entity`包下创建`AdminRole`实体类：

```java
package cn.tedu.csmall.passport.pojo.entity;

import lombok.Data;

import java.io.Serializable;
import java.time.LocalDateTime;

/**
 * 管理员与角色的关联数据的实体类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Data
public class AdminRole implements Serializable {

    /**
     * 数据id
     */
    private Long id;

    /**
     * 管理员id
     */
    private Long adminId;
    
    /**
     * 角色id
     */
    private Long roleId;

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

在`mapper`包下创建`AdminRoleMapper`接口，并在接口中添加抽象方法：

```java
package cn.tedu.csmall.passport.mapper;

import cn.tedu.csmall.passport.pojo.entity.AdminRole;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * 处理管理员与角色的关联数据的Mapper接口
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Repository
public interface AdminRoleMapper {

    /**
     * 批量插入管理员与角色的关联数据
     *
     * @param adminRoleList 管理员与角色的关联数据的列表
     * @return 受影响的行数
     */
    int insertBatch(List<AdminRole> adminRoleList);
}
```

在`src/main/resources/mapper`文件夹下通过复制粘贴得到`AdminRoleMapper.xml`文件，在此文件中配置以上抽象方法映射的SQL语句：

```xml
<mapper namespace="xx.xx.xx.xx.AdminRoleMapper">

    <!-- int insertBatch(List<AdminRole> adminRoleList); -->
    <insert id="insertBatch" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO ams_admin_role (admin_id, role_id) VALUES 
        <foreach collection="list" item="adminRole" separator=",">
            (#{adminRole.adminId}, #{adminRole.roleId})
        </foreach>
    </insert>

</mapper>
```

完成后，在`src/test/java`下的根包下创建`mapper.AdminRoleMapperTests`测试类，编写并执行测试：

```java
package cn.tedu.csmall.passport.mapper;

import cn.tedu.csmall.passport.pojo.entity.AdminRole;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.ArrayList;
import java.util.List;

@Slf4j
@SpringBootTest
public class AdminRoleMapperTests {

    @Autowired
    AdminRoleMapper mapper;

    @Test
    void testInsertBatch() {
        List<AdminRole> adminRoleList = new ArrayList<>();
        for (int i = 1; i <= 3; i++) {
            AdminRole adminRole = new AdminRole();
            adminRole.setAdminId(100L);
            adminRole.setRoleId(i + 0L);
            adminRoleList.add(adminRole);
        }

        int rows = mapper.insertBatch(adminRoleList);
        log.debug("批量插入数据完成！受影响的行数={}", rows);
    }
    
}
```

完成Mapper层的补充后，先在`AdminAddNewDTO`中添加新的属性，表示“客户端将提交到服务器端的，添加管理员时选择的若干个角色id”：

```java
/**
 * 尝试添加的管理员的角色id列表
 */
private Long[] roleIds;
```

然后，在`AdminServiceImpl`类中，补充声明：

```java
@Autowired
AdminRoleMapper adminRoleMapper;
```

并在`addNew()`方法的最后补充：

```java
// 调用adminRoleMapper的insertBatch()方法插入关联数据
Long[] roleIds = adminAddNewDTO.getRoleIds();
List<AdminRole> adminRoleList = new ArrayList<>();
for (int i = 0; i < roleIds.length; i++) {
    AdminRole adminRole = new AdminRole();
    adminRole.setAdminId(admin.getId());
    adminRole.setRoleId(roleIds[i]);
    adminRoleList.add(adminRole);
}
adminRoleMapper.insertBatch(adminRoleList);
```

完成后，在`AdminServiceTests`中原有的测试中，在测试数据上添加封装`roleIds`属性的值，并进行测试：

```java
@Test
void testAddNew() {
    AdminAddNewDTO adminAddNewDTO = new AdminAddNewDTO();
    adminAddNewDTO.setUsername("wangkejing6");
    adminAddNewDTO.setPassword("123456");
    adminAddNewDTO.setPhone("13800138006");
    adminAddNewDTO.setEmail("wangkejing6@baidu.com");
    adminAddNewDTO.setRoleIds(new Long[]{3L, 4L, 5L}); // ===== 新增 =====

    try {
        service.addNew(adminAddNewDTO);
        log.debug("添加管理员成功！");
    } catch (ServiceException e) {
        log.debug("{}", e.getMessage());
    }
}
```

# 37. 基于Spring JDBC框架的事务管理

事务：Transaction，是数据库中的一种能够保证**多个写操作**要么全部成功，要么全部失败的机制！

在基于Spring JDBC的数据库编程中，在业务方法上添加`@Transactional`注解，即可使得这个业务方法是“事务性”的！

假设，存在某银行转账的操作，转账时需要执行的SQL语句大致是：

```mysql
UPDATE 存款表 SET 余额=余额-50000 WHERE 账号='国斌老师';
```

```mysql
UPDATE 存款表 SET 余额=余额+50000 WHERE 账号='苍松老师';
```

以上的转账操作就涉及多次数据库的写操作，如果由于某些意外原因（例如停电、服务器死机等），导致第1条SQL语句成功执行，但是第2条SQL语句未能成功执行，就会出现数据不完整的问题！使用事务就可以解决此问题！

关于`@Transationcal`注解，可以添加在：

- 业务实现类的方法上
  - 仅作用于当前方法
- 业务实现类上
  - 将作用于当前类中所有方法
- 业务接口的抽象方法上
  - 仅作用于当前方法
  - 无论是哪个类重写此方法，都将是事务性的
- 业务接口上
  - 将作用于当前接口中所有抽象方法
  - 无论是哪个类实现了此接口，重写的所有方法都是将是事务性的







