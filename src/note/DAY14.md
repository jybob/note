# 44. 关于登录的账号（续）

当Spring容器中存在密码编码器时，在Spring Security处理认证时会自动调用！

本质上，是调用了密码编码器的以下方法：

```java
boolean matches(CharSequence rawPassword, String encodedPassword);
```

也就是说，Spring Security会使用用户提交的密码作为以上方法的第1个参数，使用`UserDetails`对象中的密码作为以上方法的第2个参数，然后根据调用以上方法返回的`boolean`结果来判断此用户是否能通过密码验证！

所以，如果配置了`BCryptPasswordEncoder`，则返回的`UserDetails`对象中的密码必须是BCrypt密文，例如：

```java
@Override
public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
    log.debug("Spring Security调用了loadUserByUsername()方法，参数：{}", s);
    if ("root".equals(s)) {
        UserDetails userDetails = User.builder()
                .username("root")
            	// ====== 重点是以下这一行代码中的密文 ======
                .password("$2a$10$nO7GEum8P27F8S0EGEHryel7m89opm/AMdaqMBk.qdsdIpE/SWFwe")
                .accountExpired(false)
                .accountLocked(false)
                .disabled(false)
                .authorities("这是一个山寨的权限标识")
                .build();
        log.debug("即将向Spring Security返回UserDetails对象：{}", userDetails);
        return userDetails;
    }
    log.debug("此用户名【{}】不存在，即将向Spring Security返回为null的UserDetails值", s);
    return null;
}
```

# 45. 使用数据库中的管理员账号信息来登录

Spring Security在处理认证时，会自动调用用`UserDetailsService`接口中的`UserDetails loadUserByUsername(String username)`方法，此方法返回的结果将决定此用户是否能够成功登录，此用户的信息应该来自数据库的管理员表中的数据！

则需要通过数据库查询来实现“根据用户名查询用户登录时所需要的相关信息”！需要执行的SQL语句大致是：

```mysql
select id, username, password, enable from ams_admin where username=?
```

要实现以上查询，应该先在`pojo.vo`包下创建`AdminLoginInfoVO`类：

```java
package cn.tedu.csmall.passport.pojo.vo;

import lombok.Data;

import java.io.Serializable;

/**
 * 管理员的登录VO类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Data
public class AdminLoginInfoVO implements Serializable {

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
     * 是否启用，1=启用，0=未启用
     */
    private Integer enable;

}
```

然后，在`AdminMapper.java`中添加抽象方法：

```java
/**
 * 根据用户名查询管理员的登录信息
 *
 * @param username 用户名
 * @return 匹配的管理员详情，如果没有匹配的数据，则返回null
 */
AdminLoginInfoVO getLoginInfoByUsername(String username);
```

并在`AdminMapper.xml`中配置以上抽象方法映射的SQL语句：

```xml
<!-- AdminLoginInfoVO getLoginInfoByUsername(String username); -->
<select id="getLoginInfoByUsername" resultMap="LoginResultMap">
    SELECT
        <include refid="LoginQueryFields" />
    FROM
        ams_admin
    WHERE
        username=#{username}
</select>

<sql id="LoginQueryFields">
    <if test="true">
        id, username, password, enable
    </if>
</sql>

<resultMap id="LoginResultMap" type="cn.tedu.csmall.passport.pojo.vo.AdminLoginInfoVO">
    <id column="id" property="id"/>
    <result column="username" property="username"/>
    <result column="password" property="password"/>
    <result column="enable" property="enable"/>
</resultMap>
```

最后，在`AdminMapperTests`中编写并执行测试：

```java
@Test
void testGetLoginInfoByUsername() {
    String username = "root";
    Object result = mapper.getLoginInfoByUsername(username);
    System.out.println("根据username=" + username + "查询登录信息完成，结果=" + result);
}
```

完成后，调整`UserDetailsServiceImpl`中的方法实现，根据是否查询到管理员信息来决定是否返回有效的`UesrDetails`对象，并且，当查询到管理员信息时，将查询到的信息封装到`UserDetails`对象中并返回：

```java
@Override
public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
    log.debug("Spring Security调用了loadUserByUsername()方法，参数：{}", s);
    AdminLoginInfoVO loginInfo = adminMapper.getLoginInfoByUsername(s);
    log.debug("从数据库查询与用户名【{}】匹配的管理员信息：{}", s, loginInfo);

    if (loginInfo != null) {
        UserDetails userDetails = User.builder()
                .username(loginInfo.getUsername())
                .password(loginInfo.getPassword())
                .accountExpired(false)
                .accountLocked(false)
                .disabled(loginInfo.getEnable() == 0)
                .authorities("这是一个山寨的权限标识") // 权限，注意，此方法的参数不可以为null，在不处理权限之前，可以写一个随意的字符串值
                .build();
        log.debug("即将向Spring Security返回UserDetails对象：{}", userDetails);
        return userDetails;
    }

    log.debug("此用户名【{}】不存在，即将向Spring Security返回为null的UserDetails值", s);
    return null;
}
```

至此，重启项目，通过Spring Security的登录表单，可以使用数据库中正确的管理员信息实现登录。

注意：如果数据库中的某些管理员数据是错误的（例如密码不是BCrypt密文、`enable`字段为`null`），则不能使用这些错误的数据尝试登录，应该修复这些错误的数据，或删除这些错误的数据！

# 46. 前后端分离的登录认证

目前，可以通过Spring Security的登录表单来实现认证，但是，这并不是前后端分离的做法，如果需要实现前后端分离的登录认证，需要：

- 禁用Spring Security的登录表单
- 使用控制器接收客户端提交的登录请求
  - 需要将此请求的URL添加到“白名单”
- 在控制器处理登录请求时，调用Service对象处理登录认证
- 在Service实现类中处理登录认证
  - 调用`AuthenticationManager`对象的`authenticate()`方法，将由Spring完成认证
    - 调用`authentication()`方法时，需要传入用户名、密码，则Spring Security框架会自动调用`UserDetailsService`对象的`loadUserByUsername()`方法，并自动处理后续的认证（判断密码、`enable`等）

**禁用Spring Security的登录表单**

在`SecurityConfiguration`的`void configurer(HttpSecurity http)`方法中，不再调用`http.formLogin()`即可。

**使用控制器接收客户端提交的登录请求**

先在`pojo.dto`包中创建`AdminLoginDTO`类：

```java
package cn.tedu.csmall.passport.pojo.dto;

import lombok.Data;

import java.io.Serializable;

/**
 * 管理员登录的DTO类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Data
public class AdminLoginDTO implements Serializable {

    /**
     * 用户名
     */
    private String username;

    /**
     * 密码（原文）
     */
    private String password;

}
```

在`AdminController`中添加处理请求的方法：

```java
// http://localhost:9081/admins/login
@PostMapping("/login")
public JsonResult login(AdminLoginDTO adminLoginDTO) {
    log.debug("开始处理【管理员登录】的请求，参数：{}", adminLoginDTO);
    // TODO 调用Service处理登录
    return null;
}
```

并且，在`SecurityConfiguration`中，将 `/admins/login` 添加到“白名单”中。

**在控制器处理登录请求时，调用Service对象处理登录认证**

在`IAdminService`中添加处理登录认证的抽象方法：

```java
/**
 * 管理员登录
 *
 * @param adminLoginDTO 封装了管理员的登录信息的对象
 */
void login(AdminLoginDTO adminLoginDTO);
```

在`AdminServiceImpl`中实现以上抽象方法：

```java
public void login(AdminLoginDTO adminLoginDTO) {
    log.debug("开始处理【管理员登录】的业务，参数：{}", adminLoginDTO);
    // TODO 调用AuthenticationManager对象的authenticate()方法处理认证
}
```

回到`AdminController`中，可以补充调用Service的代码：

```java
// http://localhost:9081/admins/login
@PostMapping("/login")
public JsonResult login(AdminLoginDTO adminLoginDTO) {
    log.debug("开始处理【管理员登录】的请求，参数：{}", adminLoginDTO);
    adminService.login(adminLoginDTO);
    return JsonResult.ok();
}
```

**在Service实现类中处理登录认证**

首先，需要在`SecurityConfiguration`配置类（继承自`WebSecurityConfigurerAdapter`）中重写`authenticationManager()`或`authenticationManagerBean()`方法，例如：

```java
@Bean
@Override
public AuthenticationManager authenticationManagerBean() throws Exception {
    log.debug("创建@Bean方法定义的对象：AuthenticationManager");
    return super.authenticationManagerBean();
}
```

然后，在`AdminServiceImpl`类中就可以自动装配`AuthenticationManager`对象了：

```java
@Autowired
AuthenticationManager authenticationManager;
```

并且，调用此对象的方法执行认证：

```java
@Override
public void login(AdminLoginDTO adminLoginDTO) {
    log.debug("开始处理【管理员登录】的业务，参数：{}", adminLoginDTO);
    // 调用AuthenticationManager对象的authenticate()方法处理认证
    Authentication authentication
            = new UsernamePasswordAuthenticationToken(
                    adminLoginDTO.getUsername(), adminLoginDTO.getPassword());
    authenticationManager.authenticate(authentication);
    log.debug("执行认证成功");
}
```

Spring Security在执行认证时，如果不通过（可能是用户名不存在、密码错误、账号已经被禁用等），都会抛出更种异常，如果通过认证，则程序会继续向后执行。

**测试访问**

以上全部完成后，重启项目，通过在线API文档可以测试访问。

**处理异常**

在业务状态码的枚举类型中，添加新的枚举值：

```java
public enum ServiceCode {

    OK(20000),
    ERR_BAD_REQUEST(40000),
    ERR_UNAUTHORIZED(40100), // ===== 新增 =====
    ERR_UNAUTHORIZED_DISABLED(40101), // ===== 新增 =====
    ERR_NOT_FOUND(40400),
    ERR_CONFLICT(40900),
 	// 省略后续代码   
}
```

在全局异常处理器中，补充对相关异常的处理：

```java
@ExceptionHandler({
        InternalAuthenticationServiceException.class,
        BadCredentialsException.class
})
public JsonResult handleAuthenticationException(AuthenticationException e) {
    log.debug("捕获到AuthenticationException");
    log.debug("异常类型：{}", e.getClass().getName());
    log.debug("异常信息：{}", e.getMessage());
    String message = "登录失败，用户名或密码错！";
    return JsonResult.fail(ServiceCode.ERR_UNAUTHORIZED, message);
}

@ExceptionHandler
public JsonResult handleDisabledException(DisabledException e) {
    log.debug("捕获到DisabledException：{}", e.getMessage());
    String message = "登录失败，此管理员账号已经被禁用！";
    return JsonResult.fail(ServiceCode.ERR_UNAUTHORIZED_DISABLED, message);
}
```

**注意：尽管此时已经可以通过在线API文档尝试登录，并且可以得到预期的反馈，但是，这并不是真正意义的“登录成功”，因为还没有处理通过认证后保存用户信息（例如将用户信息存储到Session中）！**

# 48. 关于Session

HTTP协议是无状态的协议，即：从协议本身并没有约定需要保存用户状态！表现为：某个客户端访问了服务器之后，后续的每一次访问，服务器都无法识别出这是前序访问的客户端！

在传统的解决方案中，可以从技术层面来解决服务器端识别客户端并保存相关数据的问题，例如使用Session机制。

Session的本质是保存在服务器端的内存中的类似`Map`结构的数据，每个客户端都有一个属于自己的`Key`，在服务器端有对应的`Value`，就是Session。

关于客户端提交请求时的`Key`：当某客户端第1次向服务器端提交请求时，并没有可用的`Key`，所以并不携带`Key`来提交请求，当服务器端发现客户端没有携带`Key`时，就会响应一个`Key`到客户端，客户端会将这个`Key`保存下来，并在后续的每一次请求中自动携带这个`Key`。并且，服务器端为了保证各个`Key`不冲突，会使用UUID算法来生成各个`Key`。由于这些`Key`是用于访问Session数据的，所以，一般称之为Session ID。

基于Session的特点，在使用时，可能存在一些问题：

- 不能**直接**用于集群甚至分布式系统
  - 可以通过共享Session技术来解决
- 将占用服务器端的内存，则不宜长时间保存

# 49. 关于Token

Token：票据，令牌

Token机制是目前主流的取代Session用于服务器端识别客户端身份的机制。

Token就类似于现实生活中的“火车票”，当客户端向服务器端提交登录请求时，就类似于“买票”的过程，当登录成功后，服务器端会生成对应的Token并响应到客户端，则客户端就拿到了所需的“火车票”，在后续的访问中，客户端携带“火车票”即可，并且，服务器端有“验票”机制，能够根据客户端携带的“火车票”识别出客户端的身份。

# 50. 关于JWT

JWT：JSON Web Token，是使用JSON格式来组织多个属性于值，主要用于Web访问的Token。

JWT的本质就是只一个字符串，是通过算法进行编码后得到的结果。

在项目中，如果需要生成、解析JWT，需要添加相关依赖项，能够实现生成、解析JWT的工具包较多，可以自由选择，可参考：https://jwt.io/libraries?language=Java

例如，在`pom.xml`中添加：

```xml
<!-- JJWT（Java JWT） -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

关于生成JWT和解析JWT的代码大致是：

```java
package cn.tedu.csmall.passport;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.junit.jupiter.api.Test;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class JwtTests {

    String secretKey = "kns439a}fdLK34jsmfd{MF5-8DJSsLKhJNFDSjn";

    @Test
    public void testGenerate() {
        Map<String, Object> claims = new HashMap<>();
        claims.put("id", 9527);
        claims.put("username", "liucangsong");
        claims.put("email", "liucangsong@163.com");

        Date expirationDate = new Date(System.currentTimeMillis() + 10 * 60 * 1000);
        System.out.println("过期时间：" + expirationDate);

        String jwt = Jwts.builder()
                // Header
                .setHeaderParam("alg", "HS256")
                .setHeaderParam("typ", "JWT")
                // Payload
                .setClaims(claims)
                .setExpiration(expirationDate)
                // Signature
                .signWith(SignatureAlgorithm.HS256, secretKey)
                // 整合
                .compact();
        System.out.println(jwt);

        // 过期时间：Wed Oct 12 17:14:41 CST 2022
        // eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6OTUyNywiZXhwIjoxNjY1NTY2MDgxLCJlbWFpbCI6ImxpdWNhbmdzb25nQDE2My5jb20iLCJ1c2VybmFtZSI6ImxpdWNhbmdzb25nIn0.zimcqHJ9w9i1ut-uQtJASIfwz5_LzrUvU_l_SHq_crA
    }

    @Test
    public void testParse() {
        String jwt = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6OTUyNywiZXhwIjoxNjY1NTY4Mjc1LCJlbWFpbCI6ImxpdWNhbmdzb25nQDE2My5jb20iLCJ1c2VybmFtZSI6ImxpdWNhbmdzb25nIn0.ESPNerLR2uBt1UtUhPwEU_71fcX_Ve-Td6X4Pjvegak";
        Claims claims = Jwts.parser().setSigningKey(secretKey).parseClaimsJws(jwt).getBody();
        Integer id = claims.get("id", Integer.class);
        System.out.println("id=" + id);
        String username = claims.get("username", String.class);
        System.out.println("username=" + username);
        String email = claims.get("email", String.class);
        System.out.println("email=" + email);
        String phone = claims.get("phone", String.class);
        System.out.println("phone=" + phone);
    }

}
```

如果尝试解析的JWT已经过期，则会出现`ExpiredJwtException`：

```
io.jsonwebtoken.ExpiredJwtException: JWT expired at 2022-10-12T17:14:41Z. Current time: 2022-10-12T17:39:57Z, a difference of 1516448 milliseconds.  Allowed clock skew: 0 milliseconds.
```

如果尝试解析的JWT数据的签名有问题（也可能由于恶意修改正确JWT的任何部分导致），则会出现：`SignatureException`：

```
io.jsonwebtoken.SignatureException: JWT signature does not match locally computed signature. JWT validity cannot be asserted and should not be trusted.
```

如果尝试解析的JWT数据的格式错误，则会出现：`MalformedJwtException`：

```
io.jsonwebtoken.MalformedJwtException: Unable to read JSON value: {"alg":"HS25�uB'
```



