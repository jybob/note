# 40. 启动或禁用管理员（续）

**关于Controller层**

由于Service设计了2个业务方法，分别用于启用和禁用，在控制器层，应该也设计2个方法，分别用于处理启用管理员的请求和禁用管理员的请求，则客户端在提交请求时，不需要提交`enable`属性的值！

则在`AdminController`中添加处理请求的方法：

```java
// http://localhost:9081/admins/9527/enable
@ApiOperation("启用管理员")
@ApiOperationSupport(order = 310)
@ApiImplicitParam(name = "id", value = "管理员id", required = true, dataType = "long")
@PostMapping("/{id:[0-9]+}/enable")
public JsonResult<Void> setEnable(@PathVariable Long id) {
    log.debug("开始处理【启用管理员】的请求，参数：{}", id);
    adminService.setEnable(id);
    return JsonResult.ok();
}

// http://localhost:9081/admins/9527/disable
@ApiOperation("禁用管理员")
@ApiOperationSupport(order = 311)
@ApiImplicitParam(name = "id", value = "管理员id", required = true, dataType = "long")
@PostMapping("/{id:[0-9]+}/disable")
public JsonResult<Void> setDisable(@PathVariable Long id) {
    log.debug("开始处理【禁用管理员】的请求，参数：{}", id);
    adminService.setDisable(id);
    return JsonResult.ok();
}
```

# 41. 关于Spring Security框架

Spring Security主要解决了认证与授权的相关问题。

Spring Security的基础依赖项是`spring-security-core`，在Spring Boot项目中，通常添加`spring-boot-starter-security`这个依赖项，它包含了`spring-security-core`，并且，还自动执行了一系列配置！默认的配置效果有：

- 所有请求都是必须通过认证的
  - 如果未认证，同步请求将自动跳转到 `/login`，是框架自带的登录页，非跨域的异步请求将响应 `403` 错误
- 提供了默认的登录信息，用户名为 `user`，密码是启动项目是随机生成的，在启动日志中可以看到
  - 当登录成功后，会自动重定向到此前访问的URL
  - 当登录成功后，可以执行所有同步请求，所有异步的POST请求都暂时不可用
  - 可以通过 `/logout` 退出登录

# 42. 关于BCrypt算法

当添加了Spring Security相关的依赖项后，此依赖项中将包含`BCryptPasswordEncoder`工具类，是一个使用`BCrypt`算法的密码编码器，它实现了`PasswordEncoder`接口，并重写了接口中的`String encode(String rawPassword)`方法，用于对密码原文进行编码（加密），及重写了`boolean matches(String rawPassword, String encodedPassword)`方法，用于验证密码原文与密文是否对应。

BCrypt算法会自动使用随机的盐值进行加密处理，所以，当反复对同一个原文进行加密处理，每次得到的密文都是不同的，但这并不影响验证密码！

BCrypt算法被设计为是一种慢速运算的算法，可以一定程度上避免或缓解密码被暴力破解（使用循环进行穷举的破解）。

# 43. 关于Spring Security的基本配置

```java
package cn.tedu.csmall.passport.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Slf4j
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    // @Bean
    public PasswordEncoder passwordEncoder() {
        log.debug("创建@Bean方法定义的对象：PasswordEncoder");
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 【配置白名单】
        // 在配置路径时，星号是通配符
        // 1个星号只能匹配任何文件夹或文件的名称，但不能跨多个层级
        // 例如：/*/test.js，可以匹配到 /a/test.js 和 /b/test.js，但不可以匹配到 /a/b/test.js
        // 2个连续的星号可以匹配若干个文件夹的层级
        // 例如：/**/test.js，可以匹配 /a/test.js 和 /b/test.js 和 /a/b/test.js
        String[] urls = {
                "/doc.html",
                "/**/*.js",
                "/**/*.css",
                "/swagger-resources",
                "/v2/api-docs"
        };

        http.csrf().disable(); // 禁用CSRF（防止伪造的跨域攻击）

        http.authorizeRequests() // 对请求执行认证与授权
                .antMatchers(urls) // 匹配某些请求路径
                .permitAll() // （对此前匹配的请求路径）不需要通过认证即允许访问
                .anyRequest() // 除以上配置过的请求路径以外的所有请求路径
                .authenticated(); // 要求是已经通过认证的

        http.formLogin(); // 开启表单验证，即视为未通过认证时，将重定向到登录表单，如果无此配置，则直接响应403
    }

}
```

# 44. 关于登录的账号

默认情况下，Spring Security使用`user`作为用户名，使用随机的UUID作为密码来登录！如果需要自行指定登录账号，需要自定义一个组件类，实现`UserDetailsService`接口，此接口中定义了`UserDetails loadUserByUsername(String username)`，在处理认证时，当用户（使用者）输入了用户名、密码并提交，Spring Security就会自动使用用户在表单中输入的用户名来调用`loadUserByUsername()`方法，作为开发者，应该重写此方法，并根据用户名来返回匹配的`UserDetails`对象，此对象中应该包含用户的相关信息，例如密码等，当Spring Security得到调用`loadUserByUsername()`返回的`UserDetails`对象后，会自动处理后续的认证过程，例如验证密码是否匹配等。

例如，在根包下创建`security.UserDetailsServiceImpl`类：

```java
package cn.tedu.csmall.passport.security;

import lombok.extern.slf4j.Slf4j;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Slf4j
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        log.debug("Spring Security调用了loadUserByUsername()方法，参数：{}", s);
        // 暂时使用模拟数据来处理登录认证，假设正确的用户名和密码分别是root和123456
        if ("root".equals(s)) {
            UserDetails userDetails = User.builder()
                    .username("root")
                    .password("123456")
                    .accountExpired(false)
                    .accountLocked(false)
                    .disabled(false)
                    .authorities("这是一个山寨的权限标识") // 权限，注意，此方法的参数不可以为null，在不处理权限之前，可以写一个随意的字符串值
                    .build();
            log.debug("即将向Spring Security返回UserDetails对象：{}", userDetails);
            return userDetails;
        }
        log.debug("此用户名【{}】不存在，即将向Spring Security返回为null的UserDetails值", s);
        return null;
    }

}
```

另外，Spring Security在执行认证时，需要使用到密码编码器（`PasswordEncoder`），则在`SecurityConfiguration`配置类中添加：

```java
@Bean
public PasswordEncoder passwordEncoder() {
    log.debug("创建@Bean方法定义的对象：PasswordEncoder");
    return NoOpPasswordEncoder.getInstance(); // 无操作的密码编码器，即：不会执行加密处理
}
```

提示：一旦启动项目时，Spring Security从Spring容器中找到了`UserDetailsService`接口类型的对象，则默认的用户名和随机的密码都不会再使用（启动项目中也不会再看到随机的临时密码）。

# 作业

完成以下功能：

- 在前端项目中，显示品牌列表
  - 后端的开发应该在DAY09作业中已完成
- 在前端项目中，在品牌列表界面中实现删除品牌功能
  - 后端的开发应该在DAY08作业中已完成
- 在前端项目中，在品牌列表界面中实现启用、禁用品牌这2个功能
  - 除了Mapper层的开发，这2个功能未布置过作业
- 在前端项目中，显示属性模板列表
  - 后端的开发应该在DAY09作业中已完成
- 在前端项目中，在属性模板列表界面中实现删除属性模板功能
  - 后端的开发应该在DAY08作业中已完成
- 在前端项目中，新增“添加属性”功能
  - 除了Mapper层的开发，此功能未布置过作业
  - 在Service层，要求“归属同一个属性模板的情况下，属性名称必须唯一”
    - 此规则对应的查询检查需另行开发
  - 在前端界面，应该通过下拉菜单显示“此属性归属的属性模板”，可参考“添加管理员”界面中的“角色”的下拉菜单，只不过，此界面中的“属性模板”下拉菜单是单选的

此作业请于本周五（10月14日）23:00之前提交到作业系统。