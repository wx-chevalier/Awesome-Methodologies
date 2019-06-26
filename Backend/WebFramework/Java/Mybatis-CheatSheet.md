[![返回目录](https://i.postimg.cc/JzFTMvjF/image.png)](https://github.com/wx-chevalier/Awesome-CheatSheets)

> 本文整理自 [Awesome CheatSheet/Java & Spring 系列]()，[MyBatis Links]() 或者 [微服务架构与实践](https://github.com/wx-chevalier/Backend-Series)。

# MyBatis CheatSheet

MyBatis 是支持普通 SQL 查询，存储过程和高级映射的优秀持久层框架。 MyBatis 消除了几乎所有的 JDBC 代码和参数的手工设置以及对结果集的检索。 MyBatis 可以使用简单的 XML 或注解用于配置和原始映射，将接口和 Java 的 POJO（ Plain Old Java Objects，普通的 Java 对象）映射成数据库中的记录。

MyBatis 提供一种半自动化的 ORM 实现。这里的“半自动化”，是相对 Hibernate 等提供了全面的数据库封装机制的“全自动化”ORM 实现而言，全自动 ORM 实现了 POJO 和数据库表之间的映射，以及 SQL 的自动生成和执行。

# 基础使用

# XML 映射文件

```java
public int insertUser(UserBean user) throws Exception;
```

在各种标签中的 id 属性必须和接口中的方法名相同 ， id 属性值必须是唯一的，不能够重复使用。parameterType 属性指明查询时使用的参数类型，resultType 属性指明查询返回的结果集类型。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 不写会报错 -->
<!DOCTYPE mapper PUBLIC "-//mybatis.org/DTD Mapper 3.0" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="wx.mybatis.mapper.UserMapper">
  <!-- 自定义返回结果集 -->
   <resultMap id="userMap" type="UserBean">
        <id property="id" column="id" javaType="java.lang.Integer"></id>
        <result property="username" column="username" javaType="java.lang.String"></result>
        <result property="password" column="password" javaType="java.lang.String"></result>
        <result property="account" column="account" javaType="java.lang.Double"></result>
    </resultMap>
    <!-- useGeneratedKeys：（ 仅 对 insert 有 用 ） 这 会 告 诉 MyBatis 使 用 JDBC 的getGeneratedKeys
        方法来取出由数据（比如：像 MySQL 和 SQLServer 这样的数据库管理系统的自动递增字段）内部生成的主键。默认值： false。
        oracle 不支持应该设置成 useGeneratedKeys="false" 不然会报错
    -->
    <!--keyProperty：（仅对 insert有用）标记一个属性， MyBatis 会通过 getGeneratedKeys或者通过 insert 语句的 selectKey 子元素设置它的值。默认：不设置。 -->
    <!--#{}中的内容，为占位符，当参数为某个JavaBean时，表示放置该Bean对象的属性值  -->
    <insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
        insert into t_user (username,password,account) values (#{username},#{password},#{account})
    </insert>

    <update id="updateUser" >
      update t_user set username=#{username},password=#{password},account=#{account} where id=#{id}
    </update>

    <delete id="deleteUser" parameterType="int">
     delete from t_user where id=#{id}
    </delete>

    <select id="selectUserById" parameterType="int" resultMap="userMap">
     select * from t_user where id=#{id}
    </select>

    <select id="selectAllUser" resultMap="userMap">
     select * from t_user
    </select>
</mapper>
```

## 数据查询

因为 XML 本身语法的限制，如果我们需要在 SQL 语句中表述小于或者大于，需要使用 CDATA 宏：

```xml
<![CDATA[
AND STUDENT_ID <= #{joiningDate}
]]>
```

## 数据操作

## 动态 SQL

## SQL 构建器

# 关联关系处理

# 插件

## MyBatis PageHelper

## MyBatis Generator

# Todos

- https://github.com/pagehelper/MyBatis-PageHelper/blob/master/wikis/zh/HowToUse.md
