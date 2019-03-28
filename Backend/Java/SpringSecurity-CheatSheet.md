[![返回目录](https://parg.co/UCb)](https://github.com/wx-chevalier/Awesome-CheatSheets)

# Spring Security CheatSheet

- 用户登陆，会被 AuthenticationProcessingFilter 拦截，调用 AuthenticationManager 的实现，而且 AuthenticationManager 会调用 ProviderManager 来获取用户验证信息（不同的 Provider 调用的服务不同，因为这些信息可以是在数据库上，可以是在 LDAP 服务器上，可以是 xml 配置文件上等），如果验证通过后会将用户的权限信息封装一个 User 放到 spring 的全局缓存 SecurityContextHolder 中，以备后面访问资源时使用。

- 访问资源（即授权管理），访问 url 时，会通过 AbstractSecurityInterceptor 拦截器拦截，其中会调用 FilterInvocationSecurityMetadataSource 的方法来获取被拦截 url 所需的全部权限，在调用授权管理器 AccessDecisionManager，这个授权管理器会通过 spring 的全局缓存 SecurityContextHolder 获取用户的权限信息，还会获取被拦截的 url 和被拦截 url 所需的全部权限，然后根据所配的策略（有：一票决定，一票否定，少数服从多数等），如果权限足够，则返回，权限不够则报错并调用权限不足页面。

![image](https://user-images.githubusercontent.com/5803001/47625333-65bc1080-db5f-11e8-8971-ec4925c9b801.png)

在 Spring Boot 项目中引入 Spring Security 同样是简单的引入 starter 封装：

```java
// gradle
compile("org.springframework.boot:spring-boot-starter-security")

// maven
<dependencies>
    ...
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
    ...
</dependencies>
```

然后声明 Web Security 相关的配置：

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        ...
    }

    ...
}
```

# Authentication | 权限认证

## HTTP Basic Auth

HTTP Basic Auth 是较为简单的静态用户名密码认证方式，分别需要声明路由规则与配置 AuthenticationManagerBuilder, 本部分完整代码参考 [spring-security-basic-auth](https://github.com/wx-chevalier/Backend-Boilerplate/tree/master/java/spring/spring-basic-auth)。

```java
// 声明路由规则
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        .anyRequest()
        .authenticated()
        .and()
        .httpBasic();
}

// 声明权限验证
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .inMemoryAuthentication()
        .withUser("user")
        .password("password")
        .roles("USER")
        .and()
        .withUser("admin")
        .password("admin")
        .roles("USER", "ADMIN");
}
```

## Form Login | 用户名密码表单登录

本部分完整代码参考 [spring-security-form-login](https://github.com/wx-chevalier/Backend-Boilerplate/tree/master/java/spring/spring-security-login), 首先在 WebSecurityConfig 的 configure 方法中，注册路由表：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
            .antMatchers("/", "/home").permitAll()
            .anyRequest().authenticated()
            .and()
        .formLogin()
            .loginPage("/login")
            .permitAll()
            .and()
        .logout()
            .permitAll();
}
```

然后需要声明 UserDetailsService，以供 Spring Context 来自动获取用户实例，该方法会在 `authenticationManager.authenticate()` 调用时被调用：

```java
@Bean
@Override
public UserDetailsService userDetailsService() {
    UserDetails user =
            User.withDefaultPasswordEncoder()
            .username("user")
            .password("password")
            .roles("USER")
            .build();

    return new InMemoryUserDetailsManager(user);
}
```

此时 Spring Security 为我们自动生成了 `/login` 与 `/logout` 两个 POST 接口，分别用来处理用户登录与登出，其对应的前台 Form 表单如下所示：

```html
<form th:action="@{/login}" method="post">
  <div>
    <label> User Name : <input type="text" name="username" /> </label>
  </div>
  <div>
    <label> Password: <input type="password" name="password" /> </label>
  </div>
  <div><input type="submit" value="Sign In" /></div>
</form>
```

在很多情况下，我们位于第三方存储中的密码是经过 Hash 混淆处理的，而不是直接读取的明文信息；此时我们可以为 Spring Security 提供自定义的密码编码器，来方便其执行比较操作：

```java
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder);
}
```

## Authentication Providers | 权限角色容器

# JWT

## Authorities | 角色权限

# CSRF

基于 Token 做 CSRF 防范的话，最基础的需要解决 Token 的生成、存储、查询等问题。针对这些问题，Spring Security 定义了 CSRF Token 仓库接口 org.springframework.security.web.csrf.CsrfTokenRepository.java。目前该接口有 3 个实现类：CookieCsrfTokenRepository、HttpSessionCsrfTokenRepository 以及 LazyCsrfTokenRepository。

CookieCsrfTokenRepository 将 Token 信息存放于客户端的 Cookie 中；每次生成 Token 后将其保存到 Cookie 中，该 Token 随 Cookie 一起返回给客户端。客户端请求被 CSRF 保护的 URL 时，需要携带 Cookie，便于服务端 loadToken 使用。使用该类型 Token 仓库时，通常需要允许 Javascript 脚本从 Cookied 中获取 Token，因此需要将 Cookie 的 httpOnly 属性值设置为 false。如果 Javascript 不需要从 Cookie 中获取 Token（例如将 Token 存放于 Session 中），建议将 httponly 设置为 true，以提高安全性。默认存放在 Cookie 中的 Token 名为 XSRF-TOKEN。

服务端在收到请求以后，会从请求中提取 Token，并与仓库中的 Token 校验，以判断该请求是否合法。具体看，Spring Security 通过 Filter 的方式提供了 CSRF 校验逻辑 org.springframework.security.web.csrf.CsrfFilter。

```java
@Override
protected void doFilterInternal(HttpServletRequest request,
        HttpServletResponse response, FilterChain filterChain)
                throws ServletException, IOException {
    request.setAttribute(HttpServletResponse.class.getName(), response);

    // 从Token仓库中加载token。如果不存在则生成并保存之
    CsrfToken csrfToken = this.tokenRepository.loadToken(request);
    final boolean missingToken = csrfToken == null;
    if (missingToken) {
        csrfToken = this.tokenRepository.generateToken(request);
        this.tokenRepository.saveToken(csrfToken, request, response);
    }
    request.setAttribute(CsrfToken.class.getName(), csrfToken);
    request.setAttribute(csrfToken.getParameterName(), csrfToken);

    // 判断是否为需要保护的路径
    if (!this.requireCsrfProtectionMatcher.matches(request)) {
        filterChain.doFilter(request, response);
        return;
    }

    // 从请求头或参数中获取携带的token
    String actualToken = request.getHeader(csrfToken.getHeaderName());
    if (actualToken == null) {
        actualToken = request.getParameter(csrfToken.getParameterName());
    }

    // 比较请求中携带的token和仓库中的token是否一致，若不一致则请求非法
    if (!csrfToken.getToken().equals(actualToken)) {
        ...
    }

    filterChain.doFilter(request, response);
}
```
