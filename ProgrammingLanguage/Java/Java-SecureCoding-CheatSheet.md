[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Java 安全编码索引

信息保护

# 序列化

对于序列化对象而言，应该使用 transient 来定义敏感数据，即避免其被序列化；而使用 serialPersistentFields 定义非敏感数据。

而我们常见的 FileNotFoundException、SQLException、BindException、OutOfMemoryError 等都有可能会引起信息泄露从而给攻击者攻击系统提供帮助。

程序中我们可以使用 SecurityManager、AccessController 来检查代码对于敏感资源或操作的访问权限。

任何敏感情况下都禁用 java.util.Random() 类生成业务的随机数，譬如 Web 应用会话标识、验证码随机数、安全敏感文件的随机文件名、生成密钥相关的随机数。

内存安全

校验过滤

线程安全
