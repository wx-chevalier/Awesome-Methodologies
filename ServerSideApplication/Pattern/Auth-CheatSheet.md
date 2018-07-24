[![返回目录](https://parg.co/Udx)](https://parg.co/UdT)

# Auth CheatSheet | 权限认证机制概述

可以阅读 [Web 安全实践清单](https://parg.co/GWc) 了解权限校验、密码规范等安全相关知识。WebAPI 或者 RPC 接口，[HTTP CheatSheet]()，[DOM CheatSheet/数据存储]()

# HTTP Basic 认证

# 基于 Session 的认证

Cookie+Session 的存在主要是为了解决 HTTP 这一无状态协议下服务器如何识别用户的问题，其原理就是在用户登录通过验证后，服务端将数据加密后保存到客户端浏览器的 Cookie 中，同时服务器保留相对应的 Session（文件或 DB）。用户之后发起的请求都会携带 Cookie 信息，服务端需要根据 Cookie 寻回对应的 Session，从而完成验证，确认这是之前登陆过的用户。其工作原理如下图所示：

![image](https://user-images.githubusercontent.com/5803001/43043318-d9211e10-8dc3-11e8-806c-e3074eb4dd39.png)

# 基于 Token 的认证

# JSON Web Token

JWT 是 Auth0 提出的通过对 JSON 进行加密签名来实现授权验证的方案。可以参考 [Backend-Boilerplate](https://github.com/wxyyxc1992/Backend-Boilerplate) 中 JWT 在于 Node.js/Java/Go 等常见服务端应用开发中的应用。

![image](https://user-images.githubusercontent.com/5803001/43043338-5fe74ffa-8dc4-11e8-9c44-04bfc4da5a01.png)

与前文介绍的基于 Session 的机制相比，使用 JWT 可以省去服务端读取 Session 的步骤，因此 JWT 是用来取代服务端的 Session 而非客户端 Cookie 的方案，也更符合 RESTful 的规范。这样但是对于客户端（或 App 端）来说，为了保存用户授权信息，仍然需要通过 Cookie 或 localStorage/sessionStorage 等机制进行本地保存。

## 组成结构

编码之后的 JWT 就是简单的字符串，包含又 `.` 分割的三部分：

![](https://cdn-images-1.medium.com/max/1600/1*0SEbHdFcVpaejejGA-1DDw.png)

```js
// 1. Header, 包括类别（typ）、加密算法（alg）；
{
  "alg": "HS256",
  "typ": "JWT"
}

// 2. Payload, 包括需要传递的用户信息；
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}

// 3. Signature, 根据alg算法与私有秘钥进行加密得到的签名字串；这一段是最重要的敏感信息，只能在服务端解密；
HMACSHA256(  
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    SECREATE_KEY
)
```

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`

## 处理流程

在使用过程中，服务端通过用户登录验证之后，将 Header+Claim 信息加密后得到第三段签名，然后将签名返回给客户端，在后续请求中，服务端只需要对用户请求中包含的 JWT 进行解码，即可验证是否可以授权用户获取相应信息，其原理如下图所示：

![](https://cdn-images-1.medium.com/max/1600/1*44waelPu4JvYALzkvoh8zw.png)

# OAuth

# 认证策略

## 单点登录

## 2FA 双因子认证
