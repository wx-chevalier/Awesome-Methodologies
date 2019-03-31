[![返回目录](https://i.postimg.cc/JzFTMvjF/image.png)](https://github.com/wx-chevalier/Awesome-CheatSheets)

> 本文整理自 [Awesome CheatSheet/Java & Spring 系列]()，[Mybatis Links]() 或者 [微服务架构与实践](https://github.com/wx-chevalier/Backend-Series)。

# Mybatis CheatSheet

MyBatis 是支持普通 SQL 查询，存储过程和高级映射的优秀持久层框架。 MyBatis 消除了几乎所有的 JDBC 代码和参数的手工设置以及对结果集的检索。 MyBatis 可以使用简单的 XML 或注解用于配置和原始映射，将接口和 Java 的 POJO（ Plain Old Java Objects，普通的 Java 对象）映射成数据库中的记录。

mybatis 提供一种“半自动化”的 ORM 实现。
这里的“半自动化”，是相对 Hibernate 等提供了全面的数据库封装机制的“全自动化”ORM 实现而言，“全自动”ORM 实现了 POJO 和数据库表之间的映射，以及 SQL 的自动生成和执行。

# 基础使用

## 手工创建 SqlSessionFactory

每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为中心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

XML 配置文件（configuration XML）中包含了对 MyBatis 系统的核心设置，包含获取数据库连接实例的数据源（DataSource）和决定事务作用域和控制方式的事务管理器（TransactionManager）。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        ...
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

我们也可以直接从 Java 程序而不是 XML 文件中创建 configuration，或者创建你自己的 configuration 构建器，MyBatis 也提供了完整的配置类，提供所有和 XML 文件相同功能的配置项。

```java
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```

有了 SqlSessionFactory ，顾名思义，我们就可以从中获得 SqlSession 的实例了。SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句。例如：

```java
SqlSession session = sqlSessionFactory.openSession();

// 直接使用 SQL 语句
try {
  Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
} finally {
  session.close();
}

// 或者使用 Mapper
BlogMapper mapper = session.getMapper(BlogMapper.class);
Blog blog = mapper.selectBlog(101);
```

Mapper 即是 Mybatis 的核心魅力所在，其支持基于 XML 与基于注解的 Mapper 定义方式：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

或者直接在接口上添加注解并使用：

```java
public interface BlogMapper {
  @Select("SELECT * FROM blog WHERE id = #{id}")
  Blog selectBlog(int id);
}
```

## Spring Boot 中集成使用

在 [SPRING INITIALIZR](https://start.spring.io/) 可以直接创建包含 Mybatis 的项目模板，也可以前往 [Backend-Boilerplate/spring](https://github.com/wx-chevalier/Backend-Boilerplate/tree/master/java/spring) 查看相关模板。我们首先需要引入 `org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.2` 依赖，然后在 Application 中添加 Mapper 扫描路径，或者在相关的 Mapper 类中添加注解：

```java
// 自定义 Mapper
@SpringBootApplication(scanBasePackages = "wx")
@MapperScan(basePackages = "wx")
@Slf4j
public class Application{}

@Mapper
public interface UserRepository {}
```

然后我们可以在 Spring Boot 的 application.properties 文件中添加 Mybatis 配置参数：

```yaml
# application.properties
mybatis.config-location=classpath:/mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mapper/*.xml
mybatis.type-aliases-package=com.example.domain.model
mybatis.type-handlers-package=com.example.typehandler
mybatis.configuration.map-underscore-to-camel-case=true
mybatis.configuration.default-fetch-size=100
mybatis.configuration.default-statement-timeout=30
```

然后在 Mybatis 配置文件中，接下来我们如常定义实体类：

```java
public class City implements Serializable {
  ...
}
```

与基于注解的 Mapper 类：

```java
@Mapper
public interface CityMapper {
    @Select("SELECT id, name, state, country FROM city WHERE state = #{state}")
    City findByState(String state);
}
```

在使用的地方直接注入该 Mapper 类实例即可：

```java
@Autowired
CityMapper cityMapper;

this.cityMapper.findByState("CA");
```

## XML 配置

将下划线表示映射到 camelCase:

```xml
<setting name="mapUnderscoreToCamelCase" value="true"/>
```

# XML 映射文件

```java
public int insertUser(UserBean user) throws Exception;
```

在各种标签中的 id 属性必须和接口中的方法名相同 ， id 属性值必须是唯一的，不能够重复使用。parameterType 属性指明查询时使用的参数类型，resultType 属性指明查询返回的结果集类型。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 不写会报错 -->
<!DOCTYPE mapper PUBLIC "-//mybatis.org/DTD Mapper 3.0" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.wenyin.mybatis.mapper.UserMapper">
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

## 数据类型

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 都会用类型处理器将获取的值以合适的方式转换成 Java 类型。从 3.4.5 开始，MyBatis 默认支持 JSR-310(日期和时间 API) 。

```
 JDBCType            JavaType
  CHAR                String
  VARCHAR             String
  LONGVARCHAR         String
  NUMERIC             java.math.BigDecimal
  DECIMAL             java.math.BigDecimal
  BIT                 boolean
  BOOLEAN             boolean
  TINYINT             byte
  SMALLINT            short
  INTEGER             int
  BIGINT              long
  REAL                float
  FLOAT               double
  DOUBLE              double
  BINARY              byte[]
  VARBINARY           byte[]
  LONGVARBINARY               byte[]
  DATE                java.sql.Date
  TIME                java.sql.Time
  TIMESTAMP           java.sql.Timestamp
  CLOB                Clob
  BLOB                Blob
  ARRAY               Array
  DISTINCT            mapping of underlying type
  STRUCT              Struct
  REF                 Ref
  DATALINK            java.net.URL
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

## Mybatis PageHelper

## Mybatis Generator

Mybatis Generator 可以直接通过命令行调用：

```sh
java -jar mybatis-generator-core-x.x.x.jar -configfile generatorConfig.xml
```

或者添加 Maven 插件：

```xml
<project ...>
    ...
    <build>
      ...
      <plugins>
      ...
      <plugin>
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-maven-plugin</artifactId>
        <version>1.3.7</version>
        <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
      </plugin>
      ...
    </plugins>
    ...
  </build>
  ...
</project>
```

然后在项目目录下执行 mvn 命令：

```sh
$ mvn mybatis-generator:generate
```

### 配置详解

generatorConfig.xml 的基础结构如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--导入属性配置 -->
    <properties resource="generator.properties"></properties>

    <!--指定特定数据库的jdbc驱动jar包的位置 -->
    <classPathEntry location="${jdbc.driverLocation}"/>

    <!--一或多个-->
    <context id="default" targetRuntime="MyBatis3">
     ...
    </context>
</generatorConfiguration>
```

`<context>` 元素用于指定生成一组对象的环境。例如指定要连接的数据库，要生成对象的类型和要处理的数据库中的表。运行 MBG 的时候还可以指定要运行的 `<context>`。该元素只有一个必选属性 id，用来唯一确定一个 `<context>` 元素，该 id 属性可以在运行 MBG 的使用。`<context>` 还包含了 defaultModelType 属性，用来指定 MBG 生成实体类的规则：

- conditional: 默认设置，这个模型和下面的 hierarchical 类似，除了如果那个单独的类将只包含一个字段，将不会生成一个单独的类。 因此,如果一个表的主键只有一个字段,那么不会为该字段生成单独的实体类,会将该字段合并到基本实体类中。
- flat: 该模型为每一张表只生成一个实体类，这个实体类包含表中的所有字段。
- hierarchical: 如果表有主键,那么该模型会产生一个单独的主键实体类,如果表还有 BLOB 字段， 则会为表生成一个包含所有 BLOB 字段的单独的实体类,然后为所有其他的字段生成一个单独的实体类。 MBG 会在所有生成的实体类之间维护一个继承关系。

MBG 配置中的其他几个元素，基本上都是<context>的子元素，这些子元素（有严格的配置顺序）包括：

- `<property>` (0 个或多个)

property 属性能够用于处理数据库表中的特殊字符，譬如 SQL 关键字的处理等。

- `<plugin>` (0 个或多个)

plugin 元素用来定义一个插件。插件用于扩展或修改通过 MyBatis Generator (MBG)代码生成器生成的代码。

- `<commentGenerator>` (0 个或 1 个)

commentGenerator 旨在创建 class 时，对注释进行控制。一般情况下由于 MBG 生成的注释信息没有任何价值，而且有时间戳的情况下每次生成的注释都不一样，使用版本控制的时候每次都会提交，因而一般情况下我们都会屏蔽注释信息，可以如下配置：

```xml
<commentGenerator>
    <property name="suppressAllComments" value="true"/>
    <property name="suppressDate" value="true"/>
</commentGenerator>
```

- `<jdbcConnection>` (1 个)

jdbcConnection 用于指定数据库连接信息，该元素必选，并且只能有一个。配置该元素只需要注意如果 JDBC 驱动不在 classpath 下，就需要通过 `<classPathEntry>` 元素引入 jar 包，这里推荐将 jar 包放到 classpath 下。

```xml
<jdbcConnection driverClass="com.mysql.jdbc.Driver"
                connectionURL="jdbc:mysql://localhost:3306/test"
                userId="root"
                password="">
</jdbcConnection>

<!--或者使用外部依赖-->
<jdbcConnection driverClass="${jdbc.driverClass}" connectionURL="${jdbc.connectionURL}" userId="${jdbc.userId}" password="${jdbc.password}">
</jdbcConnection>
```

- `<javaTypeResolver>` (0 个或 1 个)

这个元素的配置用来指定 JDBC 类型和 Java 类型如何转换。该元素提供了一个可选的属性 type，和`<commentGenerator>`比较类型，提供了默认的实现 DEFAULT，一般情况下使用默认即可，需要特殊处理的情况可以通过其他元素配置来解决，不建议修改该属性。可以配置的属性为 forceBigDecimals，该属性可以控制是否强制 DECIMAL 和 NUMERIC 类型的字段转换为 Java 类型的 java.math.BigDecimal,默认值为 false，一般不需要配置。

```xml
<javaTypeResolver >
    <!-- 如果精度>0或者长度>18，就会使用java.math.BigDecimal -->
    <!-- 如果精度=0并且10<=长度<=18，就会使用java.lang.Long -->
    <!-- 如果精度=0并且5<=长度<=9，就会使用java.lang.Integer -->
    <!-- 如果精度=0并且长度<5，就会使用java.lang.Short -->
    <property name="forceBigDecimals" value="false" />
</javaTypeResolver>
```

- `<javaModelGenerator>` (1 个)

Model 模型生成器,用来生成含有主键 key 的类，记录类 以及查询 Example 类；targetPackage 指定生成的 model 生成所在的包名，targetProject 指定在该项目下所在的路径。

```xml
<javaModelGenerator targetPackage="wx.model.po" targetProject="src/main/java">
    <!-- 是否对model添加 构造函数 -->
    <property name="constructorBased" value="true"/>

    <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
    <property name="enableSubPackages" value="false"/>

    <!-- 建立的 Model 对象是否不可改变，即生成的Model对象不会有 setter方法，只有构造方法 -->
    <property name="immutable" value="true"/>

    <!-- 给 Model 添加一个父类 -->
    <property name="rootClass" value="wx.model.Hello"/>

    <!-- 是否对类 CHAR 类型的列的数据进行 trim 操作 -->
    <property name="trimStrings" value="true"/>
</javaModelGenerator>
```

- `<sqlMapGenerator>` (0 个或 1 个)

Mapper 映射文件生成所在的目录 为每一个数据库的表生成对应的 SqlMap 文件：

```xml
<sqlMapGenerator targetPackage="wx.map.domain" targetProject="src/main/java">
    <property name="enableSubPackages" value="false"/>
</sqlMapGenerator>
```

- `<javaClientGenerator>` (0 个或 1 个)

客户端代码，生成易于使用的针对 Model 对象和 XML 配置文件的代码：

```xml
<!--
  type="ANNOTATEDMAPPER",生成Java Model 和基于注解的Mapper对象
  type="MIXEDMAPPER",生成基于注解的Java Model 和相应的Mapper对象
  type="XMLMAPPER",生成SQLMap XML文件和独立的Mapper接口
-->
<javaClientGenerator targetPackage="wx.dao" targetProject="src/main/java" type="MIXEDMAPPER">
    <property name="enableSubPackages" value=""/>
    <!--
            定义Maper.java 源代码中的ByExample() 方法的可视性，可选的值有：
            public;
            private;
            protected;
            default
            注意：如果 targetRuntime="MyBatis3",此参数被忽略
      -->
    <property name="exampleMethodVisibility" value=""/>
    <!--
      方法名计数器
      Important note: this property is ignored if the target runtime is MyBatis3.
      -->
    <property name="methodNameCalculator" value=""/>

    <!--
        为生成的接口添加父接口
    -->
    <property name="rootInterface" value=""/>
</javaClientGenerator>
```

- `<table>` (1 个或多个)

```xml
<table tableName="lession" schema="mybatis" domainObjectName="ObjName">
    <!-- optional
         自动生成的键值(identity,或者序列值)，如果指定此元素，MBG将会生成<selectKey>元素，然后将此元素插入到SQL Map的<insert> 元素之中，sqlStatement 的语句将会返回新的值；如果是一个自增主键的话，你可以使用预定义的语句, 或者添加自定义的SQL语句. 预定义的值如下:
          MySql: SELECT LAST_INSERT_ID()

    -->
    <generatedKey column="id" sqlStatement="Mysql" identity="" type=""/>

    <!-- optional.
            列的命名规则：
            MBG使用 <columnRenamingRule> 元素在计算列名的对应名称之前，先对列名进行重命名，
            作用：一般需要对BUSI_CLIENT_NO 前的BUSI_进行过滤
            支持正在表达式
              searchString 表示要被换掉的字符串
              replaceString 则是要换成的字符串，默认情况下为空字符串，可选
    -->
    <columnRenamingRule searchString="" replaceString=""/>

    <!-- optional.告诉 MBG 忽略某一列
            column，需要忽略的列
            delimitedColumnName:true ,匹配column的值和数据库列的名称 大小写完全匹配，false 忽略大小写匹配
            是否限定表的列名，即固定表列在Model中的名称
    -->
    <ignoreColumn column="PLAN_ID"  delimitedColumnName="true" />

    <!--optional.覆盖MBG对 Model 的生成规则
          column: 数据库的列名
          javaType: 对应的Java数据类型的完全限定名
          在必要的时候可以覆盖由JavaTypeResolver计算得到的java数据类型. For some databases, this is necessary to handle "odd" database types (e.g. MySql's unsigned bigint type should be mapped to java.lang.Object).
          jdbcType:该列的JDBC数据类型(INTEGER, DECIMAL, NUMERIC, VARCHAR, etc.)，该列可以覆盖由JavaTypeResolver计算得到的Jdbc类型，对某些数据库而言，对于处理特定的JDBC 驱动癖好 很有必要(e.g. DB2's LONGVARCHAR type should be mapped to VARCHAR for iBATIS).
          typeHandler:

    -->
    <columnOverride column="" javaType=""	jdbcType=""	typeHandler=""	delimitedColumnName="" />

</table>
```

如果我们希望在生成的实体类中支持 Lombok，那可以参考 [mybatis-generator-lombok-plugin](https://github.com/softwareloop/mybatis-generator-lombok-plugin) 等项目。

### 模板使用

对于 Mybatis Generator 生成的代码模板，不建议修改生成后的模板，这样在数据库发生变化时候可以直接重新生成，根据错误提示修改对应代码。Mybatis Generator 为我们自动生成了 Model, Mapper 与 Example 文件，其中 Example 能够被用于构建搜索条件，譬如：

```java
// 模糊搜索用户名
String name = "明";
UserExample ex = new UserExample();
ex.createCriteria().andNameLike('%'+name+'%');
List<User> userList = userDao.selectByExample(ex);

// 通过某个字段排序
String orderByClause = "id DESC";
UserExample ex = new UserExample();
ex.setOrderByClause(orderByClause);
List<User> userList = userDao.selectByExample(ex);

// 条件搜索，不确定条件的个数
UserExample ex = new UserExample();
Criteria criteria = ex.createCriteria();
if(StringUtils.isNotBlank(user.getAddress())){
	criteria.andAddressEqualTo(user.getAddress());
}
if(StringUtils.isNotBlank(user.getName())){
	criteria.andNameEqualTo(user.getName());
}
List<User> userList = userDao.selectByExample(ex);

// 分页搜索列表
pager.setPageNum(1);
pager.setPageSize(5);
UserExample ex = new UserExample();
ex.setPage(pager);
List<User> userList = userDao.selectByExample(ex);
```

# Todos

- https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md
