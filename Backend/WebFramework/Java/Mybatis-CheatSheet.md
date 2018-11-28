[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

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

在 [SPRING INITIALIZR](https://start.spring.io/) 可以直接创建包含 Mybatis 的项目模板，也可以前往 [Backend-Boilerplate/spring](https://github.com/wxyyxc1992/Backend-Boilerplate/tree/master/java/spring) 查看相关模板。

我们首先需要引入 `org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.2` 依赖，然后在 Application 中添加 Mapper 扫描路径，或者在相关的 Mapper 类中添加注解：

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
mybatis.mapper-locations=classpath:mapper/*.xml
```

或者指定专门的 mybatis 配置文件：

```yaml
mybatis.config-location=classpath:/mybatis/mybatis-config.xml
```

接下来我们如常定义实体类：

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

# XML 映射文件

```java
public int insertUser(UserBean user) throws Exception;
```

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
    <!-- 在各种标签中的id属性必须和接口中的方法名相同 ， id属性值必须是唯一的，不能够重复使用。parameterType属性指明查询时使用的参数类型，
    resultType属性指明查询返回的结果集类型-->
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

## 类型

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

generatorConfig.xml 的详细配置阐述如下：

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

    <context id="default" targetRuntime="MyBatis3">

        <!-- optional，旨在创建class时，对注释进行控制 -->
        <commentGenerator>
            <property name="suppressDate" value="true" />
        </commentGenerator>

        <!--jdbc的数据库连接 -->
        <jdbcConnection driverClass="${jdbc.driverClass}" connectionURL="${jdbc.connectionURL}" userId="${jdbc.userId}" password="${jdbc.password}">
        </jdbcConnection>

        <!-- 非必需，类型处理器，在数据库类型和java类型之间的转换控制-->
        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!-- Model模型生成器,用来生成含有主键key的类，记录类 以及查询Example类
            targetPackage     指定生成的model生成所在的包名
            targetProject     指定在该项目下所在的路径
        -->
        <javaModelGenerator targetPackage="wx.model.po" targetProject="src/main/java">
            <!-- 是否对model添加 构造函数 -->
            <property name="constructorBased" value="true"/>

            <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="false"/>

            <!-- 建立的Model对象是否 不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
            <property name="immutable" value="true"/>

            <!-- 给Model添加一个父类 -->
            <property name="rootClass" value="wx.model.Hello"/>

            <!-- 是否对类CHAR类型的列的数据进行trim操作 -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!--Mapper映射文件生成所在的目录 为每一个数据库的表生成对应的SqlMap文件 -->
        <sqlMapGenerator targetPackage="wx.map.domain" targetProject="src/main/java">
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>


        <!-- 客户端代码，生成易于使用的针对Model对象和XML配置文件 的代码
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

        <table tableName="lession" schema="mybatis">
            <!-- optional   , only for mybatis3 runtime
                 自动生成的键值(identity,或者序列值)
               如果指定此元素，MBG将会生成<selectKey>元素，然后将此元素插入到SQL Map的<insert> 元素之中
               sqlStatement 的语句将会返回新的值
               如果是一个自增主键的话，你可以使用预定义的语句,或者添加自定义的SQL语句. 预定义的值如下:
                  Cloudscape 	This will translate to: VALUES IDENTITY_VAL_LOCAL()
                  DB2: 		VALUES IDENTITY_VAL_LOCAL()
                  DB2_MF:		SELECT IDENTITY_VAL_LOCAL() FROM SYSIBM.SYSDUMMY1
                  Derby: 		VALUES IDENTITY_VAL_LOCAL()
                  HSQLDB: 	CALL IDENTITY()
                  Informix: 	select dbinfo('sqlca.sqlerrd1') from systables where tabid=1
                  MySql: 		SELECT LAST_INSERT_ID()
                  SqlServer: 	SELECT SCOPE_IDENTITY()
                  SYBASE: 	SELECT @@IDENTITY
                  JDBC:		This will configure MBG to generate code for MyBatis3 suport of JDBC standard generated keys. This is a database independent method of obtaining the value from identity columns.
                  identity: 自增主键  If true, then the column is flagged as an identity column and the generated <selectKey> element will be placed after the insert (for an identity column). If false, then the generated <selectKey> will be placed before the insert (typically for a sequence).

            -->
            <generatedKey column="" sqlStatement="" identity="" type=""/>

            <!-- optional.
                    列的命名规则：
                    MBG使用 <columnRenamingRule> 元素在计算列名的对应 名称之前，先对列名进行重命名，
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
            <!--optional.覆盖MBG对Model 的生成规则
                 column: 数据库的列名
                 javaType: 对应的Java数据类型的完全限定名
                 在必要的时候可以覆盖由JavaTypeResolver计算得到的java数据类型. For some databases, this is necessary to handle "odd" database types (e.g. MySql's unsigned bigint type should be mapped to java.lang.Object).
                 jdbcType:该列的JDBC数据类型(INTEGER, DECIMAL, NUMERIC, VARCHAR, etc.)，该列可以覆盖由JavaTypeResolver计算得到的Jdbc类型，对某些数据库而言，对于处理特定的JDBC 驱动癖好 很有必要(e.g. DB2's LONGVARCHAR type should be mapped to VARCHAR for iBATIS).
                 typeHandler:

            -->
            <columnOverride column="" javaType=""	jdbcType=""	typeHandler=""	delimitedColumnName="" />

        </table>
    </context>
</generatorConfiguration>
```
