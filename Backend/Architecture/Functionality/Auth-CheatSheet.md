[![返回目录](https://parg.co/Udx)](https://parg.co/UdT)

# Auth CheatSheet | 权限认证机制概述

可以阅读 [Web 安全实践清单](https://parg.co/GWc) 了解权限校验、密码规范等安全相关知识。WebAPI 或者 RPC 接口，[HTTP CheatSheet]()，[DOM CheatSheet/数据存储]()

# HTTP Basic 认证

# 基于 Session 的认证

Cookie 与 Session 的存在主要是为了解决 HTTP 这一无状态协议下服务器如何识别用户的问题，其原理就是在用户登录通过验证后，服务端将数据加密后保存到客户端浏览器的 Cookie 中，同时服务器保留相对应的 Session（文件或 DB）。用户之后发起的请求都会携带 Cookie 信息，服务端需要根据 Cookie 寻回对应的 Session，从而完成验证，确认这是之前登陆过的用户。其工作原理如下图所示：

![image](https://user-images.githubusercontent.com/5803001/43043318-d9211e10-8dc3-11e8-806c-e3074eb4dd39.png)

# 基于 Token 的认证

# JSON Web Token

JWT 是 Auth0 提出的通过对 JSON 进行加密签名来实现授权验证的方案。可以参考 [Backend-Boilerplate](https://github.com/wxyyxc1992/Backend-Boilerplate) 中 JWT 在于 Node.js/Java/Go 等常见服务端应用开发中的应用。

![image](https://user-images.githubusercontent.com/5803001/48563391-33682c80-e92f-11e8-9cb1-a855546837c9.png)

与前文介绍的基于 Session 的机制相比，使用 JWT 可以省去服务端读取 Session 的步骤，因此 JWT 是用来取代服务端的 Session 而非客户端 Cookie 的方案，也更符合 RESTful 的规范。这样但是对于客户端（或 App 端）来说，为了保存用户授权信息，仍然需要通过 Cookie 或 localStorage/sessionStorage 等机制进行本地保存。JWT 也可以被广泛用于微服务场景下各个服务的认证，免去重复中心化认证的繁琐。

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

// 3. Signature, 根据 alg 算法与私有秘钥进行加密得到的签名字串；这一段是最重要的敏感信息，只能在服务端解密；
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

以 [node/jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) 为例，我们可以使用字符串密钥，或者加载 cert 文件:

```js
const jwt = require('jsonwebtoken');
const token = jwt.sign({ foo: 'bar' }, 'shhhhh');

// sign with RSA SHA256
const cert = fs.readFileSync('private.key');
const token = jwt.sign({ foo: 'bar' }, cert, { algorithm: 'RS256' });

// 设置过期时间
jwt.sign(
  {
    exp: Math.floor(Date.now() / 1000) + 60 * 60,
    data: 'foobar'
  },
  'secret'
);
```

在进行内容读取时，我们可以直接解码 Payload 部分获取用户信息，也可以对签名内容进行验证：

```js
// 直接获取到 Payload 而忽略签名
var decoded = jwt.decode(token);

// 获取完整的 Payload 与 Header
var decoded = jwt.decode(token, { complete: true });
console.log(decoded.header);
console.log(decoded.payload);

// 同步校验 Token 是否有效
var decoded = jwt.verify(token, 'shhhhh');
console.log(decoded.foo); // bar

// 异步校验是否有效
jwt.verify(token, 'shhhhh', function(err, decoded) {
  console.log(decoded.foo); // bar
});
```

# OAuth

# 认证策略

## 单点登录

## 2FA 双因子认证

# HTTP Based Authorization

## Basic Authorization

桌面应用程序也通过 HTTP 协议跟 Web 服务器交互， 桌面应用程序一般不会使用 cookie, 而是把 " 用户名 + 冒号 + 密码 " 用 BASE64 算法加密后的字符串放在 http request 中的 header Authorization 中发送给服务端， 这种方式叫 HTTP 基本认证 (Basic Authentication)

当浏览器访问使用基本认证的网站的时候， 浏览器会提示你输入用户名和密码，如下图

![](http://pic002.cnblogs.com/images/2012/263119/2012092510283354.png)

假如用户名密码错误的话， 服务器会返回 401 如下图

![](http://pic002.cnblogs.com/images/2012/263119/2012092510293780.png)

HTTP 基本认证的过程

第一步 : 客户端发送 http request 给服务器，

第二步 : 因为 request 中没有包含 Authorization header, 服务器会返回一个 401 Unauthozied 给客户端，并且在 Response 的 header "WWW-Authenticate" 中添加信息。

![](http://pic002.cnblogs.com/images/2012/263119/2012092121494456.png)

第三步：客户端把用户名和密码用 BASE64 加密后，放在 Authorization header 中发送给服务器， 认证成功。

第四步：服务器将 Authorization header 中的用户名密码取出，进行验证， 如果验证通过，将根据请求，发送资源给客户端

![](http://pic002.cnblogs.com/images/2012/263119/2012092121495881.png)

使用 Fiddler Inspectors 下的 Auth 选项卡，可以很方便的看到用户名和密码 :

![](http://pic002.cnblogs.com/images/2012/263119/2012092121505442.png)

HTTP 基本认证的优点

HTTP 基本认证，简单明了。Rest API 就是经常使用基本认证的。

每次都要进行认证

http 协议是无状态的， 同一个客户端对 服务器的每个请求都要求认证。

## Token Based Authorization-JWT

笔者在这里以 JWT 为例。
