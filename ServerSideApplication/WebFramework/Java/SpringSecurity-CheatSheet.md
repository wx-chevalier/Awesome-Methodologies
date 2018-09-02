## CSRF

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
