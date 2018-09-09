[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Maven CheatSheet

# QuickStart

## Introduction

Maven 是一个异常强大的构建工具，能够帮我们自动化构建过程，从清理、编译、测试到生成报告，再到打包和部署。通过 Maven，我们只需要输入简单的命令(如 mvn clean install)，就会帮我们处理繁琐的任务。Maven 最大化的消除了构建的重复，抽象了构建生命周期，并且为绝大部分的构建任务提供了已实现的插件。比如说测试，我们只需要遵循 Maven 的约定编写好测试用例，当我们运行构建的时候，这些测试便会自动运行。除此之外，Maven 能帮助我们标准化构建过程。在 Maven 之前，十个项目可能有十种构建方式，但通过 Maven，所有项目的构建命令都是简单一致的。有利于促进项目团队的标准化。

Maven 是笔者接触的第一个脱离于 IDE 的命令行构建工具，笔者之前一直是基于 Visual Studio 下进行 Windows 驱动开发，并不是很能明白 Builder 与 IDE 之间的区别。依赖大量的手工操作。编译、测试、代码生成等工作都是相互独立的，很难一键完成所有工作。手工劳动往往意味着低效，意味着容易出错。很难在项目中统一所有的 IDE 配置，每个人都有自己的喜好。也正是由于这个原因，一个在机器 A 上可以成功运行的任务，到了机器 B 的 IDE 中可能就会失败。

### Make

Make 将自己和操作系统绑定在一起了。也就是说，使用 Make，就不能实现(至少很难)跨平台的构建，这对于 Java 来说是非常不友好的。此外，Makefile 的语法也成问题，很多人抱怨 Make 构建失败的原因往往是一个难以发现的空格或 Tab 使用错误。

### Ant

- 和 Make 一样，Ant 也都是过程式的，开发者显式地指定每一个目标，以及完成该目标所需要执行的任务。针对每一个项目，开发者都需要重新编写这一过程，这里其实隐含着很大的重复。Maven 是声明式的，项目构建过程和过程各个阶段所需的工作都由插件实现，并且大部分插件都是现成的，开发者只需要声明项目的基本元素，Maven 就执行内置的、完整的构建过程。这在很大程度上消除了重复。
- Ant 是没有依赖管理的，所以很长一段时间 Ant 用户都不得不手工管理依赖，这是一个令人头疼的问题。幸运的是，Ant 用户现在可以借助 Ivy 管理依赖。而对于 Maven 用户来说，依赖管理是理所当然的，Maven 不仅内置了依赖管理，更有一个可能拥有全世界最多 Java 开源软件包的中央仓库，Maven 用户无须进行任何配置就可以直接享用。

## Usage

### Installation

可从 apache 官方下载最新的 Maven 压缩包，解压即可。然后设置下系统的环境变量。如下所示:

- M2HOME:maven 安装目录
- Path:追加 maven 安装目录下的 bin 目录

在用户目录下，我们可以发现.m2 文件夹。默认情况下，该文件夹下放置了 Maven 本地仓库.m2/repository。所有的 Maven 构件(artifact)都被存储到该仓库中，以方便重用。默认情况下，~/.m2 目录下除了 repository 仓库之外就没有其他目录和文件了，不过大多数 Maven 用户需要复制 M2HOME/conf/settings.xml 文件到~/.m2/settings.xml

### Commands List

本节列举出部分常用的 Maven 命令：

mvn -v 查看 maven 版本

mvn compile 编译

mvn test 测试

mvn package 打包

mvn clean 删除 target

mvn install 安装 jar 包到本地仓库中

- 创建一个新工程

mvn archetype:generate -DgroupId=co.hoteam -DartifactId=Zigbee -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

### Pom

就像 Make 的 Makefile，Ant 的 build.xml 一样，Maven 项目的核心是 pom.xml。

首先创建一个名为 hello-world 的文件夹，打开该文件夹，新建一个名为 pom.xml 的文件，输入其内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
  http://maven.apache.org/maven-v4_0_0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <groupId>com.juvenxu.mvnbook</groupId>
  <artifactId>hello-world</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>Maven Hello World Project</name>
</project>
```

- 代码的第一行是 XML 头，指定了该 xml 文档的版本和编码方式。紧接着是 project 元素，project 是所有 pom.xml 的根元素，它还声明了一些 POM 相关的命名空间及 xsd 元素，虽然这些属性不是必须的，但使用这些属性能够让第三方工具(如 IDE 中的 XML 编辑器)帮助我们快速编辑 POM。

* 根元素下的第一个子元素 modelVersion 指定了当前 POM 模型的版本，对于 Maven2 及 Maven 3 来说，它只能是 4.0.0。这段代码中最重要的是 groupId，artifactId 和 version 三行。这三个元素定义了一个项目基本的坐标，在 Maven 的世界，任何的 jar、pom 或者 war 都是以基于这些基本的坐标进行区分的。

- groupId 定义了项目属于哪个组，这个组往往和项目所在的组织或公司存在关联，譬如你在 googlecode 上建立了一个名为 myapp 的项目，那么 groupId 就应该是 com.googlecode.myapp，如果你的公司是 mycom，有一个项目为 myapp，那么 groupId 就应该是 com.mycom.myapp。本书中所有的代码都基于 groupId com.juvenxu.mvnbook。

* artifactId 定义了当前 Maven 项目在组中唯一的 ID，我们为这个 Hello World 项目定义 artifactId 为 hello-world，本书其他章节代码会被分配其他的 artifactId。而在前面的 groupId 为 com.googlecode.myapp 的例子中，你可能会为不同的子项目(模块)分配 artifactId，如：myapp-util、myapp-domain、myapp-web 等等。

- version 指定了 Hello World 项目当前的版本——1.0-SNAPSHOT。SNAPSHOT 意为快照，说明该项目还处于开发中，是不稳定的版本。随着项目的发展，version 会不断更新，如升级为 1.0、1.1-SNAPSHOT、1.1、2.0 等等。
- 最后一个 name 元素声明了一个对于用户更为友好的项目名称，虽然这不是必须的，但我还是推荐为每个 POM 声明 name，以方便信息交流。 没有任何实际的 Java 代码，我们就能够定义一个 Maven 项目的 POM，这体现了 Maven 的一大优点，它能让项目对象模型最大程度地与实际代码相独立，我们可以称之为解耦，或者正交性，这在很大程度上避免了 Java 代码和 POM 代码的相互影响。比如当项目需要升级版本时，只需要修改 POM，而不需要更改 Java 代码；而在 POM 稳定之后，日常的 Java 代码开发工作基本不涉及 POM 的修改。

### Main

项目主代码和测试代码不同，项目的主代码会被打包到最终的构件中(比如 jar)，而测试代码只在运行测试时用到，不会被打包。默认情况下，Maven 假设项目主代码位于 src/main/java 目录，我们遵循 Maven 的约定，创建该目录，然后在该目录下创建文件 com/juvenxu/mvnbook/helloworld/HelloWorld.java，其内容如下:

```java
package com.juvenxu.mvnbook.helloworld;

public class HelloWorld
{
    public String sayHello()
    {
        return "Hello Maven";
    }

    public static void main(String[] args)
    {
        System.out.print( new HelloWorld().sayHello() );
    }
}
```

关于该 Java 代码有两点需要注意。首先，在 95%以上的情况下，我们应该把项目主代码放到 src/main/java/目录下(遵循 Maven 的约定)，而无须额外的配置，Maven 会自动搜寻该目录找到项目主代码。其次，该 Java 类的包名是 com.juvenxu.mvnbook.helloworld，这与我们之前在 POM 中定义的 groupId 和 artifactId 相吻合。一般来说，项目中 Java 类的包都应该基于项目的 groupId 和 artifactId，这样更加清晰，更加符合逻辑，也方便搜索构件或者 Java 类。 代码编写完毕后，我们使用 Maven 进行编译，在项目根目录下运行命令 mvn clean compile 即可。

clean 告诉 Maven 清理输出目录 target/，compile 告诉 Maven 编译项目主代码，从输出中我们看到 Maven 首先执行了 clean:clean 任务，删除 target/目录，默认情况下 Maven 构建的所有输出都在 target/目录中；接着执行 resources:resources 任务(未定义项目资源，暂且略过)；最后执行 compiler:compile 任务，将项目主代码编译至 target/classes 目录(编译好的类为 com/juvenxu/mvnbook/helloworld/HelloWorld.Class)。

## Configuration

### Network

#### Proxy

编辑~/.m2/settings.xml 文件(如果没有该文件，则复制$M2HOME/conf/settings.xml)。添加代理配置如下：

```xml
<settings>
  ...
  <proxies>
    <proxy>
      <id>my-proxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>代理服务器主机名</host>
      <port>端口号</port>
      <!--
          <username>***</username>
          <password>***</password>
          <nonProxyHosts>repository.mycom.com|*.google.com</nonProxyHosts>
      -->
  </proxy>
  </proxies>
  ...
</settings>
```

如果不行试试重启机器或者 eclipse 等 ide 还不行试试下面这种方式：windows-->preferences-->maven-->installations add

![maven config](http://outofmemory.cn/ugc/upload/00/20/20130620/maven-config.png)

这样配置后将使用指定目录下的 maven，而非 eclipse 的 maven 内置插件。

#### Mirror

众所周知的原因，国内有时候并不能够很顺畅的访问 Maven 的中央仓库，往往我们需要访问国内的镜像地址：

> - [OSChina Maven 教程][2]

```xml
<mirror>
  <id>CN</id>
  <name>OSChina Central</name>
  <url>http://maven.oschina.net/content/groups/public/</url>
  <mirrorOf>central</mirrorOf>
</mirror>
```

## Error List

### 网络问题

(1)有时候因为众所周知的网络问题，导致 Maven 无法访问中央仓库然后扔出一大堆错误，这个时候可以尝试参考上文中的设置代理。但是也要注意，是不是有一些私库中的 Repository。

### 编译问题

(1)有时候执行`mvn compile`时候会爆出无法找到 junit 的错误，可能的解决方法有：

- 在 Eclipse 的 Projects 选项中使用 Projects Clean

- 在 pom.xml 中引入 junit 依赖项，并且保证其 scope 为 compile:

  ```
  <dependency>
  	<groupId>junit</groupId>
  	<artifactId>junit</artifactId>
  	<version>4.11</version>
  	<scope>test</scope>
  </dependency>
  ```

(2)有时候在 Eclipse 下执行`mvn compile`或者相关命令时，会报某某文件出现不识别字符或者非 UTF-8 编码，此时可以做几步检查：

- 检查对应的 Java 文件是否有 Bom 头
- 检查对应的 Java 文件的编码
- 如果都没有问题，在 Eclipse 中先将文件编码设置为 GBK，再改回 UTF-8 试试。

## Reference

- [Maven 学习](https://tracylihui.github.io/2015/07/09/Maven%E5%AD%A6%E4%B9%A0/)​
- [Maven 实战(许晓斌著)](http://www.linuxidc.com/Linux/2014-12/110503.htm)

### Tutorials&Docs

- [CSDN-Maven 学习每周总结](http://blog.csdn.net/lfsfxy9/article/category/1516519)

### Practice&Resource

- [maven-best-practices](http://www.kyleblaney.com/maven-best-practices/)

# Dependence(依赖管理)

> 《记一次依赖项冲突》

## dependencyManagement

Maven 使用 dependencyManagement 元素来提供了一种管理依赖版本号的方式。通常会在一个组织或者项目的最顶层的父 POM 中看到 dependencyManagement 元素。使用 pom.xml 中的 dependencyManagement 元素能让所有在子项目中引用一个依赖而不用显式的列出版本号。Maven 会沿着父子层次向上走，直到找到一个拥有 dependencyManagement 元素的项目，然后它就会使用在这个 dependencyManagement 元素中指定的版本号。

例如在父项目里：

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.2</version>
    </dependency>
    ...
  <dependencies>
</dependencyManagement>
```

然后在子项目里就可以添加 mysql-connector 时可以不指定版本号，例如：

```xml
<dependencies>
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
  </dependency>
</dependencies>
```

同时在 dependenceManagement 种，也可以从外部导入 POM 文件中的依赖项：

```xml
<dependencyManagement>
     <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.3.0.RC1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

# Resources(资源管理)

## Resource Directories：资源文件夹

它对应的配置方式为：

```xml
<project>
 ...
 <build>
   ...
   <resources>
     <resource>
       <directory>[your folder here]</directory>
     </resource>
   </resources>
   ...
 </build>
 ...
</project>
```

对于如下的结构：

```yaml
Project
|-- pom.xml
`-- src
`-- my-resources
```

我们需要在 pom 文件中进行如下配置：

```xml
...
   <resources>
     <resource>
       <directory>src/my-resources</directory>
     </resource>
   </resources>
...
```

譬如如果我们要用 Maven 构建一个 Web 项目，会在 src/main 目录下构建一个

## 选择包含或者忽视文件或者目录

```xml
<project>
  ...
  <name>My Resources Plugin Practice Project</name>
  ...
  <build>
    ...
    <resources>
      <resource>
        <directory>src/my-resources</directory>
        <excludes>
          <exclude>**/*.bmp</exclude>
          <exclude>**/*.jpg</exclude>
          <exclude>**/*.jpeg</exclude>
          <exclude>**/*.gif</exclude>
        </excludes>
      </resource>
      <resource>
        <directory>src/my-resources2</directory>
        <includes>
          <include>**/*.txt</include>
        </includes>
        <excludes>
          <exclude>**/*test*.*</exclude>
        </excludes>
      </resource>
      ...
    </resources>
    ...
  </build>
  ...
</project>
```

## Filter：过滤与内容替换

# Run & Package(运行与打包)

## Run

### Main

如果需要在 Maven 中直接运行某个类中的 Main 方法：

```
mvn exec:java -Dexec.mainClass="com.example.Main"
```

如果是经常使用的话，可以在 pom 文件中添加如下的配置：

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>1.2.1</version>
  <executions>
    <execution>
      <goals>
        <goal>java</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <mainClass>com.example.Main</mainClass>
    <arguments>
      <argument>foo</argument>
      <argument>bar</argument>
    </arguments>
  </configuration>
</plugin>
```

### Jetty-Plugin

#### Scan(文件扫描)

```xml
<project>
...
  <plugins>
...
    <plugin>
      <groupId>org.eclipse.jetty</groupId>
      <artifactId>jetty-maven-plugin</artifactId>
      <version>9.3.1-SNAPSHOT</version>
      <configuration>
        <webAppSourceDirectory>${project.basedir}/src/staticfiles</webAppSourceDirectory>
        <webApp>
          <contextPath>/</contextPath>
          <descriptor>${project.basedir}/src/over/here/web.xml</descriptor>
          <jettyEnvXml>${project.basedir}/src/over/here/jetty-env.xml</jettyEnvXml>
        </webApp>
        <classesDirectory>${project.basedir}/somewhere/else</classesDirectory>
        <scanClassesPattern>
          <excludes>
             <exclude>**/Foo.class</exclude>
          </excludes>
        </scanClassesPattern>
        <scanTargets>
          <scanTarget>src/mydir</scanTarget>
          <scanTarget>src/myfile.txt</scanTarget>
        </scanTargets>
        <scanTargetPatterns>
          <scanTargetPattern>
            <directory>src/other-resources</directory>
            <includes>
              <include>**/*.xml</include>
              <include>**/*.properties</include>
            </includes>
            <excludes>
              <exclude>**/myspecial.xml</exclude>
              <exclude>**/myspecial.properties</exclude>
            </excludes>
          </scanTargetPattern>
        </scanTargetPatterns>
      </configuration>
    </plugin>
  </plugins>
</project>
```

#### Port(监听端口)

在运行 Jetty 时往往需要改变其监听的端口，主要就是修正 HttpConnector 的参数来建立一些 ServerConnector 的配置，主要有如下的三种方式：

- Change the port when just at runtime:

  ```
  mvn jetty:run -Djetty.http.port=9999
  ```

- Set the property inside your *pom.xml* file:

  ```
  <properties>
    <jetty.http.port>9999</jetty.http.port>
  </properties>
  ```

  Then just run:

  ```
  mvn jetty:run
  ```

- Set the port in your plugin declaration inside the *pom.xml* file:

  ```
  <build>
    <plugins>
      <plugin>
        <groupId>org.eclipse.jetty</groupId>
        <artifactId>jetty-maven-plugin</artifactId>
        <version>9.2.1.v20140609</version>
        <configuration>
          <httpConnector>
            <!--host>localhost</host-->
            <port>9999</port>
          </httpConnector>
        </configuration>
      </plugin>
    </plugins>
  </build>
  ```

## Package(打包)

### Profile:构建不同环境的部署包

项目开发好以后，通常要在多个环境部署，象我们公司多达 5 种环境：本机环境(**local**)、(开发小组内自测的)开发环境(**dev**)、(提供给测试团队的)测试环境(**test**)、预发布环境(**pre**)、正式生产环境(**prod**)，每种环境都有各自的配置参数，比如：数据库连接、远程调用的 ws 地址等等。如果每个环境 build 前手动修改这些参数，显然会非常的麻烦。而 Maven 本身就可以允许我们通过定义 Profile 的方式来在编译是动态注入配置：

```xml
<profiles>
        <profile>
            <!-- 本地环境 -->
            <id>local</id>
            <properties>
                <db-url>jdbc:oracle:thin:@localhost:1521:XE</db-url>
                <db-username>***</db-username>
                <db-password>***</db-password>
            </properties>
        </profile>
        <profile>
            <!-- 开发环境 -->
            <id>dev</id>
            <properties>
                <db-url>jdbc:oracle:thin:@172.21.129.51:1521:orcl</db-url>
                <db-username>***</db-username>
                <db-password>***</db-password>
            </properties>
            <!-- 默认激活本环境 -->
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        ...
</profiles>
```

profiles 节点中，定义了二种环境：local、dev(默认激活 dev 环境)，可以在各自的环境中添加需要的 property 值，接下来修改 build 节点，参考下面的示例：

<build>

```xml
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.5.1</version>
            <configuration>
                <source>1.6</source>
                <target>1.6</target>
                <encoding>utf-8</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>
```

resource 节点是关键，它表明了哪个目录下的配置文件(不管是 xml 配置文件，还是 properties 属性文件)，需要根据 profile 环境来替换属性值。通常配置文件放在 resources 目录下，build 时该目录下的文件都自动会 copy 到 class 目录下:

![](http://images.cnitblog.com/blog/27612/201408/281044295329658.jpg)

以上图为例，其中 spring-database.xml 的内容为：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dataSource"
        class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver" />
        <property name="url" value="${db-url}" />
        <property name="username" value="${db-username}" />
        <property name="password" value="${db-password}" />
    </bean>
</beans>
```

各属性节点的值，用占位符"${属性名}"占位，maven 在 package 时，会根据 profile 的环境自动替换这些占位符为实际属性值。

默认情况下：

`maven package`

将采用默认激活的 profile 环境来打包，也可以手动指定环境，比如：

`maven package -P dev`

将自动打包成 dev 环境的部署包(注：参数 P 为大写)

**1、开发环境与生产环境数据源采用不同方式的问题**

本机开发时为了方便，很多开发人员喜欢直接用 JDBC 直接连接数据库，这样修改起来方便；

```
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver" />
        <property name="url" value="${db-url}" />
        <property name="username" value="${db-username}" />
        <property name="password" value="${db-password}" />
        <property name="defaultAutoCommit" value="false" />
        <property name="initialSize" value="2" />
        <property name="maxActive" value="10" />
        <property name="maxWait" value="60000" />
    </bean>
```

而生产环境，通常是在 webserver(比如 weblogic 上)配置一个 JNDI 数据源，

```xml
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
         <property name="jndiName" value="appDS" />
</bean>
```

如果每次发布生产前，都要手动修改，未免太原始，可以通过 maven 的 profile 来解决。先把配置文件改成  ：

```xml
<bean id="${db-source-jdbc}" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver" />
        <property name="url" value="${db-url}" />
        <property name="username" value="${db-username}" />
        <property name="password" value="${db-password}" />
        <property name="defaultAutoCommit" value="false" />
        <property name="initialSize" value="2" />
        <property name="maxActive" value="10" />
        <property name="maxWait" value="60000" />
    </bean>

    <bean id="${db-source-jndi}" class="org.springframework.jndi.JndiObjectFactoryBean">
        <property name="jndiName" value="appDS" />
</bean>
```

即用占位符来代替 bean 的 id，然后在 pom.xml 里类似下面设置

```xml
<profile>
            <!-- 本机环境 -->
            <id>local</id>
            <properties>
                ...
                <db-source-jdbc>dataSource</db-source-jdbc>
                <db-source-jndi>NONE</db-source-jndi>
                <db-url>jdbc:oracle:thin:@172.21.129.51:1521:orcl</db-url>
                <db-username>mu_fsu</db-username>
                <db-password>mu_fsu</db-password>
                ...
            </properties>
            <!-- 默认激活本环境 -->
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <profile>
            <!-- 生产环境 -->
            <id>pro</id>
            <properties>
                ...
                <db-source-jdbc>NONE</db-source-jdbc>
                <db-source-jndi>dataSource</db-source-jndi>
                ...
            </properties>
        </profile>
    </profiles>
```

这样，mvn clean package -P local 打包本地开发环境时，将生成

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver" />
        <property name="url" value="jdbc:oracle:thin:@172.21.129.***:1521:orcl" />
        <property name="username" value="***" />
        <property name="password" value="***" />
        <property name="defaultAutoCommit" value="false" />
        <property name="initialSize" value="2" />
        <property name="maxActive" value="10" />
        <property name="maxWait" value="60000" />
    </bean>

    <bean id="NONE" class="org.springframework.jndi.JndiObjectFactoryBean">
        <property name="jndiName" value="appDS" />
    </bean>
```

而打包生产环境 mvn clean package -P pro 时，生成

```xml
<bean id="NONE" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver" />
        <property name="url" value="${db-url}" />
        <property name="username" value="${db-username}" />
        <property name="password" value="${db-password}" />
        <property name="defaultAutoCommit" value="false" />
        <property name="initialSize" value="2" />
        <property name="maxActive" value="10" />
        <property name="maxWait" value="60000" />
    </bean>

    <bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
        <property name="jndiName" value="appDS" />
    </bean>
```

**2、不同 webserver 环境，依赖 jar 包，是否打包的问题**

weblogic 上，允许多个 app，把共用的 jar 包按约定打包成一个 war 文件，以 library 的方式部署，然后各应用在 WEB-INF/weblogic.xml 中，用类似下面的形式

```xml
<?xml version="1.0" encoding="utf-8"?>
<weblogic-web-app xmlns="http://www.bea.com/ns/weblogic/90">
    ...
    <library-ref>
        <library-name>my-share-lib</library-name>
    </library-ref>
</weblogic-web-app>
```

指定共享 library 的名称即可。这样的好处是，即节省了服务器开销，而且各 app 打包时，就不必再重复打包这些 jar 文件，打包后的体积大大减少，上传起来会快很多。

而其它 webserver 上却未必有这个机制，一般为了方便，我们开发时，往往采用一些轻量级的 webserver，比如:tomcat,jetty,jboss 之类，正式部署时才发布到 weblogic 下，这样带来的问题就是，本机打包时，要求这些依赖 jar 包，全打包到 app 的 WEB-INF/lib 下；而生产环境下，各应用的 WEB-INF/lib 下并不需要这些 jar 文件，同样还是用 profile 来搞定，先处理 pom.xml，把依赖项改成类似下面的形式：

```
<dependency>
            <groupId>dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>1.6.1</version>
            <scope>${jar.scope}</scope>
        </dependency>
```

即 scope 这里，用一个占位符来代替，然后 profile 这样配置

<profile>

```xml
        <!-- 本机环境 -->
        <id>local</id>
        <properties>
            <jar.scope>compile</jar.scope>
            ...
        </properties>
        <!-- 默认激活本环境 -->
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <!-- 生产环境 -->
        <id>pro</id>
        <properties>
            <jar.scope>provided</jar.scope>
            ...
        </properties>
    </profile>
```

在 maven 里，如果一个依赖项的 scope 是 provided，表示由容器提供，打包时将不会打包进最终的 package 里，所以这样配置后，生产环境打包时，依赖项的 scope 全变成了 provided，即不打包进 war 文件，而本机环境下，因为 scope 是 compile，所以会打包到 war 里。

# Test(测试)

> 参考资料
>
> - [Maven 单元测试][1]

Maven 本身并不是一个单元测试框架，它只是在构建执行到特定生命周期阶段的时候，通过插件来执行 JUnit 或者 TestNG 的测试用例。这个插件就是 maven-surefire-plugin，也可以称为测试运行器(Test Runner)，它能兼容 JUnit 3、JUnit 4 以及 TestNG。在默认情况下，maven-surefire-plugin 的 test 目标会自动执行测试源码路径(默认为 src/test/java/)下所有符合一组命名模式的测试类。这组模式为：

- \*_/Test_.java：任何子目录下所有命名以 Test 开关的 Java 类。
- \**/*Test.java：任何子目录下所有命名以 Test 结尾的 Java 类。
- \**/*TestCase.java：任何子目录下所有命名以 TestCase 结尾的 Java 类。

## JUnit

在 Java 世界中，由 Kent Beck 和 Erich Gamma 建立的 JUnit 是事实上的单元测试标准。要使用 JUnit，我们首先需要为 Hello World 项目添加一个 JUnit 依赖，修改项目的 POM 如代码清单。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.juvenxu.mvnbook</groupId>
  <artifactId>hello-world</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>Maven Hello World Project</name>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.7</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

代码中添加了 dependencies 元素，该元素下可以包含多个 dependency 元素以声明项目的依赖，这里我们添加了一个依赖——groupId 是 junit，artifactId 是 junit，version 是 4.7。前面我们提到 groupId、artifactId 和 version 是任何一个 Maven 项目最基本的坐标，JUnit 也不例外，有了这段声明，Maven 就能够自动下载 junit-4.7.jar。也许你会问，Maven 从哪里下载这个 jar 呢？在 Maven 之前，我们可以去 JUnit 的官网下载分发包。而现在有了 Maven，它会自动访问中央仓库([http://repo1.maven.org/maven2/](http://repo1.maven.org/maven2/)) ,下载需要的文件。读者也可以自己访问该仓库，打开路径 junit/junit/4.7/，就能看到 junit-4.7.pom 和 junit-4.7.jar。

上述 POM 代码中还有一个值为 test 的元素 scope，scope 为依赖范围，若依赖范围为 test 则表示该依赖只对测试有效，换句话说，测试代码中的 import JUnit 代码是没有问题的，但是如果我们在主代码中用 import JUnit 代码，就会造成编译错误。如果不声明依赖范围，那么默认值就是 compile，表示该依赖对主代码和测试代码都有效。

配置了测试依赖，接着就可以编写测试类，回顾一下前面的 HelloWorld 类，现在我们要测试该类的 sayHello()方法，检查其返回值是否为“Hello Maven”。在 src/test/java 目录下创建文件，其内容如代码清单如下：

```
package com.juvenxu.mvnbook.helloworld;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class HelloWorldTest
{
    @Test
    public void testSayHello()
    {
        HelloWorld helloWorld = new HelloWorld();

        String result = helloWorld.sayHello();

        assertEquals( "Hello Maven", result );
    }
}
```

测试用例编写完毕之后就可以调用 Maven 执行测试，运行 mvn clean test。

构建在执行 compiler:testCompile 任务的时候失败了，Maven 输出提示我们需要使用-source 5 或更高版本以启动注释，也就是代码中 JUnit 4 的@Test 注解。这是 Maven 初学者常常会遇到的一个问题。由于历史原因，Maven 的核心插件之一 compiler 插件默认只支持编译 Java 1.3，因此我们需要配置该插件使其支持 Java 5，见代码清单：

```xml
<project>
…
<build>
<plugins>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
       <source>1.5</source>
       <target>1.5</target>
     </configuration>
   </plugin>
</plugins>
</build>
…
</project>
```

该 POM 省略了除插件配置以外的其他部分。现在再执行 mvn clean test,结果正常。

## 测试命令

Maven 中使用 package、install 等命令时会自动调用 Test 组件，`mvn package -DskipTests`命令可以跳过测试。也可以在插件配置的时候设置跳过：

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.5</version>
    <configuration>
        <skipTests>true</skipTests>
    </configuration>
</plugin>
```

### 指定测试用例

maven-surefire-plugin 提供了一个 test 参数让 Maven 用户能够在命令行指定要运行的测试用例。如：

```
mvn test -Dtest=RandomGeneratorTest
```

也可以使用通配符：

```
mvn test -Dtest=Random*Test
```

或者也可以使用“，”号指定多个测试类：

```
mvn test -Dtest=Random*Test,AccountCaptchaServiceTest
```

如果由于历史原因，测试类不符合默认的三种命名模式，可以通过 pom.xml 设置 maven-surefire-plugin 插件添加命名模式或排除一些命名模式。

```
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.5</version>
        <configuration>
            <includes>
                <include>**/*Tests.java</include>
            </includes>
            <excludes>
                <exclude>**/*ServiceTest.java</exclude>
                <exclude>**/TempDaoTest.java</exclude>
            </excludes>
        </configuration>
    </plugin>
```

## Coverage(测试覆盖率)

### Cobertura

[1]: http://blog.csdn.net/sin90lzc/article/details/7543262
[2]: http://maven.oschina.net/help.html
