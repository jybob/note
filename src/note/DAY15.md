# 50. 关于JWT（续）

JWT是不安全的，因为在不知道`secretKey`的情况下，任何JWT都是可以解析出Header、Payload部分的，这2部分的数据并没有做任何加密处理，所以，如果JWT数据被暴露，则任何人都可以从中解析出Header、Payload中的数据！

至于JWT中的`secretKey`，及生成JWT时使用的算法，是用于对Header、Payload执行签名算法的，JWT中的Signature是用于验证JWT真伪的。

当然，如果你认为有必要的话，可以自行另外使用加密算法，将Payload中应该封装的数据先加密，再用于生成JWT！

另外，如果JWT数据被泄露，他人使用有效的JWT是可以正常使用的！所以，通常，在相对比较封闭的操作系统（例如智能手机的操作系统）中，JWT的有效时间可以设置得很长，但是，不太封闭的操作系统（例如PC端的操作系统）中，JWT的有效时间应该相对较短。

所以，在JWT时，需要注意：

- 根据你所需的安全性，来设置JWT的有效时间
- 不要在JWT中存放敏感数据，例如：手机号码、身份证号码、明文密码
- 如果一定要在JWT中存放敏感数据，应该自行使用加密算法处理过后再用于生成JWT

# 51. 登录成功时生成并响应JWT

在使用JWT的项目，用户登录就相当于现实生活乘车之前**购买火车票**的过程，所以，当用户登录成功时，需要生成对应的JWT数据，并响应到客户端。

首先，需要修改`IAdminService`接口中处理登录的抽象方法的声明，将返回值类型改为`String`，表示将返回成功登录的JWT数据：

```java
/**
 * 管理员登录
 *
 * @param adminLoginDTO 封装了管理员的登录信息的对象
 * @return 成功登录的JWT数据
 */
String login(AdminLoginDTO adminLoginDTO);
```

然后，在`AdminServiceImpl`实现类中，也修改重写的方法的声明，并且，在登录成功后，生成、返回JWT数据：

```java
log.debug("准备生成JWT数据");
Map<String, Object> claims = new HashMap<>();
// claims.put("id", null); // 向JWT中封装id
claims.put("username", adminLoginDTO.getUsername()); // 向JWT中封装username

String secretKey = "kns439a}fdLK34jsmfd{MF5-8DJSsLKhJNFDSjn";
Date expirationDate = new Date(System.currentTimeMillis() + 10 * 24 * 60 * 60 * 1000);
String jwt = Jwts.builder()
        .setHeaderParam("alg", "HS256")
        .setHeaderParam("typ", "JWT")
        .setClaims(claims)
        .setExpiration(expirationDate)
        .signWith(SignatureAlgorithm.HS256, secretKey)
        .compact();
log.debug("返回JWT数据：{}", jwt);
return jwt;
```

提示：以上代码并不是最终版本。

最后，在`AdminController`中，还需要响应JWT数据：

```java
// http://localhost:9081/admins/login
@PostMapping("/login")
public JsonResult<String> login(AdminLoginDTO adminLoginDTO) {
    log.debug("开始处理【管理员登录】的请求，参数：{}", adminLoginDTO);
    String jwt = adminService.login(adminLoginDTO);
    return JsonResult.ok(jwt);
}
```

完成后，重启项目，可以在API文档中测试访问，当登录成功后，响应的结果大致是：

```json
{
  "state": 20000,
  "data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2NjY0ODk1NjMsInVzZXJuYW1lIjoic3VwZXJfYWRtaW4ifQ.T5wnIVFk-AhvxPETloDsSgx46vdV45Y3BRk1_0oc3CM"
}
```

**关于处理认证的细节**

当调用了`AuthenticationManager`对象的`authenticate()`方法，且通过认证后，此方法将返回`Authentication`接口类型的对象，此对象的具体类型是`UsernamePasswordAuthenticationToken`，此对象中包含名为`Principal`（当事人）的属性，值为`UserDetailsService`对象中`loadUserByUsername()`返回的对象！

另外，目前在`UserDetailsServiceImpl`中返回的`UserDetails`接口类型的对象是`User`类型的，此类型没有`id`属性，如果需要向JWT中封装`id`甚至其它属性，必须自定义类，继承自`User`或实现`UserDetails`接口，在自定义类中补充声明所需的属性，并在`UserDetailsServiceImpl`中返回自定义类的对象，则处理认证通过后，返回的`Authentication`中的`Principal`就是自定义类的对象！

在`security`包中创建`AdminDetails`类，继承自`User`对其进行扩展：

```java
@Setter
@Getter
@EqualsAndHashCode
@ToString(callSuper = true)
public class AdminDetails extends User {

    private Long id;

    public AdminDetails(String username, String password, boolean enabled,
                        Collection<? extends GrantedAuthority> authorities) {
        super(username, password, enabled,
                true, true, true,
                authorities);
    }

}
```

在`UserDetailsServiceImpl`中，调整为返回`AdminDetails`类型的对象：

```java
@Override
public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
    log.debug("Spring Security调用了loadUserByUsername()方法，参数：{}", s);
    AdminLoginInfoVO loginInfo = adminMapper.getLoginInfoByUsername(s);
    log.debug("从数据库查询与用户名【{}】匹配的管理员信息：{}", s, loginInfo);

    if (loginInfo == null) {
        log.debug("此用户名【{}】不存在，即将抛出异常");
        String message = "登录失败，用户名不存在！";
        throw new BadCredentialsException(message);
    }
    
    // ===== 以下是调整的内容 =====
    List<GrantedAuthority> authorities = new ArrayList<>();
    GrantedAuthority authority = new SimpleGrantedAuthority("这是一个山寨的权限标识");
    authorities.add(authority);

    AdminDetails adminDetails = new AdminDetails(
            loginInfo.getUsername(), loginInfo.getPassword(),
            loginInfo.getEnable() == 1, authorities);
    adminDetails.setId(loginInfo.getId());

    log.debug("即将向Spring Security返回UserDetails接口类型的对象：{}", adminDetails);
    return adminDetails;
}
```

经过以上调整，当`AuthenticationManager`执行`authenticate()`认证方法后，如果登录成功，返回的`Authentication`中的`Principal`就是以上返回的`AdminDetails`对象，则可以从中获取`id`、`username`等数据，用于生成JWT数据，则在`AdminServiceImpl`中的`login()`方法中：

```java
@Override
public String login(AdminLoginDTO adminLoginDTO) {
    log.debug("开始处理【管理员登录】的业务，参数：{}", adminLoginDTO);
    // 调用AuthenticationManager对象的authenticate()方法处理认证
    Authentication authentication
            = new UsernamePasswordAuthenticationToken(
                    adminLoginDTO.getUsername(), adminLoginDTO.getPassword());
    Authentication authenticateResult
            = authenticationManager.authenticate(authentication);
    log.debug("执行认证成功，AuthenticationManager返回：{}", authenticateResult);
    Object principal = authenticateResult.getPrincipal();
    log.debug("认证结果中的Principal数据类型：{}", principal.getClass().getName());
    log.debug("认证结果中的Principal数据：{}", principal);
    AdminDetails adminDetails = (AdminDetails) principal;

    log.debug("准备生成JWT数据");
    Map<String, Object> claims = new HashMap<>();
    claims.put("id", adminDetails.getId()); // 向JWT中封装id
    claims.put("username", adminDetails.getUsername()); // 向JWT中封装username

    String secretKey = "kns439a}fdLK34jsmfd{MF5-8DJSsLKhJNFDSjn";
    Date expirationDate = new Date(System.currentTimeMillis() + 10 * 24 * 60 * 60 * 1000);
    String jwt = Jwts.builder()
            .setHeaderParam("alg", "HS256")
            .setHeaderParam("typ", "JWT")
            .setClaims(claims)
            .setExpiration(expirationDate)
            .signWith(SignatureAlgorithm.HS256, secretKey)
            .compact();
    log.debug("返回JWT数据：{}", jwt);
    return jwt;
}
```

至此，当客户端向服务器端提交登录请求，且登录成功后，将得到服务器端响应的JWT数据，此JWT中包含了`id`和`username`。

# 解析JWT

当客户端已经登录成功并得到JWT，相当于现实生活中某人已经成功购买到了火车票，接下来，此人应该携带火车票去乘车，在程序中，就表现为：客户端应该携带JWT向服务器端提交请求。

关于客户端携带JWT数据，业内惯用的做法是客户端应该将JWT放在请求头（Request Headers）中名为`Authorization`的属性中。

在服务器端，通常使用**过滤器**组件来解析JWT数据。

在项目的根包下创建`JwtAuthorizationFilter`：

```java
package cn.tedu.csmall.passport.filter;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jws;
import io.jsonwebtoken.Jwts;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.context.SecurityContext;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * JWT认证过滤器
 *
 * <p>Spring Security框架会自动从SecurityContext读取认证信息，如果存在有效信息，则视为已登录，否则，视为未登录</p>
 * <p>当前过滤器应该尝试解析客户端可能携带的JWT，如果解析成功，则创建对应的认证信息，并存储到SecurityContext中</p>
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Slf4j
@Component
public class JwtAuthorizationFilter extends OncePerRequestFilter {

    public static final int JWT_MIN_LENGTH = 100;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        // 尝试获取客户端提交请求时可能携带的JWT
        String jwt = request.getHeader("Authorization");
        log.debug("接收到JWT数据：{}", jwt);

        // 判断是否获取到有效的JWT
        if (!StringUtils.hasText(jwt) || jwt.length() < JWT_MIN_LENGTH) {
            // 直接放行
            log.debug("未获取到有效的JWT数据，将直接放行");
            filterChain.doFilter(request, response);
            return;
        }

        // 尝试解析JWT，从中获取用户的相关数据，例如id、username等
        log.debug("将尝试解析JWT……");
        String secretKey = "kns439a}fdLK34jsmfd{MF5-8DJSsLKhJNFDSjn";
        Claims claims = Jwts.parser().setSigningKey(secretKey).parseClaimsJws(jwt).getBody();
        Long id = claims.get("id", Long.class);
        String username = claims.get("username", String.class);
        log.debug("从JWT中解析得到数据：id={}", id);
        log.debug("从JWT中解析得到数据：username={}", username);

        // 将根据从JWT中解析得到的数据来创建认证信息
        List<GrantedAuthority> authorities = new ArrayList<>();
        GrantedAuthority authority = new SimpleGrantedAuthority("这是一个山寨的权限标识");
        authorities.add(authority);
        Authentication authentication = new UsernamePasswordAuthenticationToken(
                username, null, authorities);

        // 将认证信息存储到SecurityContext中
        SecurityContext securityContext = SecurityContextHolder.getContext();
        securityContext.setAuthentication(authentication);

        // 放行
        filterChain.doFilter(request, response);
    }

}
```

完成后，还需要在`SecurityConfiguration`中自动装配自定义的JWT过滤器：

```java
@Autowired
JwtAuthorizationFilter jwtAuthorizationFilter;
```

并在`configurer()`方法中补充：

```java
// 将自定义的JWT过滤器添加在Spring Security框架内置的过滤器之前
http.addFilterBefore(jwtAuthorizationFilter, UsernamePasswordAuthenticationFilter.class);
```

# 52. 关于认证信息中的Principal

关于`SecurityContext`中的认证信息，应该包含当事人（Principal）和权限（Authorities），其中，当事人（Principal）被声明为`Object`类型的，则可以使用任意数据类型作为当事人！

在使用了Spring Security框架的项目中，当事人的数据是可以被注入到处理请求的方法中的！所以，使用哪种数据作为当事人，主要取决于“你在编写控制器中处理请求的方法时，需要通过哪些数据来区分当前登录的用户”。

通常，使用自定义的数据类型作为当事人，并在此类型中封装关键数据，例如`id`、`username`等。

则在`security`包下创建`LoginPrincipal`类：

```java
@Data
public class LoginPrincipal implements Serializable {
    private Long id;
    private String username;
}
```

在JWT过滤器创建认证信息时，使用以上类型的对象作为认证信息中的当事人：

```java
LoginPrincipal loginPrincipal = new LoginPrincipal(); // 新增
loginPrincipal.setId(id); // 新增
loginPrincipal.setUsername(username); // 新增

// 注意：以下调用构造方法时，第1个参数是以上创建的对象
Authentication authentication = new UsernamePasswordAuthenticationToken(
        loginPrincipal, null, authorities);
```

完成后，在当前项目任何控制器中任何处理请求的方法上，都可以添加`@AuthenticationPrincipal LoginPrincipal loginPrincipal`参数（与原有的其它参数不区分先后顺序），此参数的值就是以上过滤器中存入到认证信息中的当事人，所以，可以通过这种做法，在处理请求时识别当前登录的用户：

```java
@ApiOperation("删除管理员")
@ApiOperationSupport(order = 200)
@ApiImplicitParam(name = "id", value = "管理员id", required = true, dataType = "long")
@PostMapping("/{id:[0-9]+}/delete")
public JsonResult<Void> delete(@PathVariable Long id,
        // ===== 以下是新增的方法参数 =====
        @ApiIgnore @AuthenticationPrincipal LoginPrincipal loginPrincipal) {
    log.debug("开始处理【删除管理员】的请求，参数：{}", id);
    log.debug("当前登录的当事人：{}", loginPrincipal); // 新增，可以控制台观察数据
    adminService.delete(id);
    return JsonResult.ok();
}
```

# 53. 关于CORS与PreFlight

如果客户端向服务器端提交请求，在跨域的前提下，如果提交的请求配置了请求头中的非典型参数，例如配置了`Authorization`，此请求会被视为“复杂请求”，则会要求执行“预检”（PreFlight），如果预检不通过，则会导致跨域请求错误！

关于预检，浏览器会自动向服务器端提交`OPTIONS`类型的请求执行预检，为了确保预检通过，不影响处理正常的请求，需要在`SecurityConfiguration`的`configurer()`方法中对预检请求放行，可以采取的解决方案有：

```java
http.authorizeRequests()
                .antMatchers(urls)
                .permitAll()

    			// 以下2行代码是用于对预检的OPTIONS请求直接放行的
                .antMatchers(HttpMethod.OPTIONS, "/**")
                .permitAll()

                .anyRequest()
                .authenticated();
```

或者，也可以：

```java
http.cors(); // 启用Spring Security框架的处理跨域的过滤器，此过滤器将放行跨域请求，包括预检的OPTIONS请求
```

则客户端可以携带复杂请求头进行访问：

```java
loadAdminList() {
  console.log('loadAdminList ...');
  let url = 'http://localhost:9081/admins';
  console.log('url = ' + url);
  this.axios
      .create({
        'headers': {
          'Authorization': localStorage.getItem('jwt')
        }
      })
      .get(url).then((response) => {
    let responseBody = response.data;
    console.log(responseBody);
    this.tableData = responseBody.data;
  });
}
```

# 作业

- 在“类别”表（`pms_category`）中，存在名为`parent_id`的字段，表示“父级类别”，例如存在id=1的类别名为“家电”，则名为“冰箱”的类别的`parent_id`值就应该是`1`，所以，“家电”是一级类别，而“冰箱”是二级类别，另外，所有一级类别的`parent_id`值为`0`，现要求实现：
  - Mapper层：根据父级类别查询子级类别列表
  - Service层：同上，无特别的业务规则
  - Controller层：同上
  - 前端页面：显示所有一级类别的列表，暂不关心子级类别的显示
- 在前序作业中，已经完成“添加属性”的功能，且已知每个“属性”都归属于某个“属性模板”，现要求实现：
  - Mapper层：根据“属性模板”查询“属性”列表
  - Service层：同上，无特别的业务规则
  - Controller层：同上
  - 前端页面：显示某个“属性模板”中的属性列表，关于“属性模板”的id，可暂时写成一个固定值

此作业请于本周日（10月16日）23:00之前提交到作业系统。









