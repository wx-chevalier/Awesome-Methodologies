[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Maven CheatSheet

Maven 是一个异常强大的构建工具，能够帮我们自动化构建过程，从清理、编译、测试到生成报告，再到打包和部署。通过 Maven，我们只需要输入简单的命令(如 mvn clean install)，就会帮我们处理繁琐的任务。Maven 最大化的消除了构建的重复，抽象了构建生命周期，并且为绝大部分的构建任务提供了已实现的插件。比如说测试，我们只需要遵循 Maven 的约定编写好测试用例，当我们运行构建的时候，这些测试便会自动运行。除此之外，Maven 能帮助我们标准化构建过程。在 Maven 之前，十个项目可能有十种构建方式，但通过 Maven，所有项目的构建命令都是简单一致的。有利于促进项目团队的标准化。

![](https://ww1.sinaimg.cn/large/007rAy9hgy1fzqpenbb80j30eo0hegm2.jpg)

## 背景与对比

Make 将自己和操作系统绑定在一起了。也就是说，使用 Make，就不能实现(至少很难)跨平台的构建，这对于 Java 来说是非常不友好的。此外，Makefile 的语法也成问题，很多人抱怨 Make 构建失败的原因往往是一个难以发现的空格或 Tab 使用错误。

和 Make 一样，Ant 也都是过程式的，开发者显式地指定每一个目标，以及完成该目标所需要执行的任务。针对每一个项目，开发者都需要重新编写这一过程，这里其实隐含着很大的重复。Maven 是声明式的，项目构建过程和过程各个阶段所需的工作都由插件实现，并且大部分插件都是现成的，开发者只需要声明项目的基本元素，Maven 就执行内置的、完整的构建过程。这在很大程度上消除了重复。

Ant 是没有依赖管理的，所以很长一段时间 Ant 用户都不得不手工管理依赖，这是一个令人头疼的问题。幸运的是，Ant 用户现在可以借助 Ivy 管理依赖。而对于 Maven 用户来说，依赖管理是理所当然的，Maven 不仅内置了依赖管理，更有一个可能拥有全世界最多 Java 开源软件包的中央仓库，Maven 用户无须进行任何配置就可以直接享用。

## 安装

可从 Apache 官方下载最新的 Maven 压缩包，解压即可。然后设置下系统的环境变量。如下所示:

- M2HOME: Maven 安装目录
- Path: 追加 Maven 安装目录下的 bin 目录

在用户目录下，我们可以发现.m2 文件夹。默认情况下，该文件夹下放置了 Maven 本地仓库.m2/repository。所有的 Maven 构件(artifact)都被存储到该仓库中，以方便重用。默认情况下，~/.m2 目录下除了 repository 仓库之外就没有其他目录和文件了，不过大多数 Maven 用户需要复制 M2HOME/conf/settings.xml 文件到~/.m2/settings.xml

```sh
# 查看 maven 版本
mvn -v

mvn compile 编译

mvn test 测试

mvn package 打包

mvn clean 删除 target

mvn install 安装 jar 包到本地仓库中

# 跳过测试
-DskipTests -Dmaven.test.skip

# 创建新工程
mvn archetype:generate -DgroupId=co.hoteam -DartifactId=Zigbee -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

# POM 配置

就像 Make 的 Makefile，Ant 的 build.xml 一样，Maven 项目的核心是 pom.xml。首先创建一个名为 hello-world 的文件夹，打开该文件夹，新建一个名为 pom.xml 的文件，输入其内容如下：

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

## 仓库

在 Maven 的术语中，仓库是一个位置(place)，例如目录，可以存储所有的工程 jar 文件、library jar 文 件、插件或任何其他的工程指定的文件。严格意义上说，Maven 只有两种类型的仓库:

- 本地(local)
- 远程(remote)

Maven 的本地仓库保存你的工程的所有依赖(library jar、plugin jar 等)。当你运行一次 Maven 构建时，Maven 会自动下载所有依赖的 jar 文件到本地仓库中。它避免了每次构建时都引用存放在远程仓库上的依赖文件。

Maven 的本地仓库默认被创建在 \${user.home}/.m2/repository 目录下。要修改默认位置，只要在 settings.xml 文件中定义另一个路径即可，例如：

```xml
<localRepository>
/anotherDirectory/.m2/respository</localRepository>
```

Maven 的远程仓库可以是任何其他类型的存储库，可通过各种协议，例如 file：//和 http：// 来访问。

这些存储库可以是由第三方提供的可供下载的远程仓库，例如 Maven 的中央仓库(central repository)：

- repo.maven.apache.org/maven2

- uk.maven.org/maven2

也可以是在公司内的 FTP 服务器或 HTTP 服务器上设置的内部存储库，用于在开发团队和发布之间共享私有的 artifacts。

首先 Maven 会到本地仓库中去寻找所需要的 jar 吧，如果找不到就会到配置的私有仓库中去找，如果私有仓库中也找不到的话，就会到配置的中央仓库中去找，如果还是找不到就会报错。但是这中间只要在某一个仓库中找到了就会返回了，除非仓库中有更新的版本，或者是 snapshot 版本。

## 镜像

Mirror 则相当于一个代理，它会拦截去指定的远程 Repository 下载构件的请求，然后从自己这里找出构件回送给客户端。配置 Mirror 的目的一般是出于网速考虑。Repository 和 Mirror 是两个不同的概念：前者本身是一个仓库，可以堆外提供服务，而后者本身并不是一个仓库，它只是远程仓库的网络加速器。需要注意的是很多本地仓库搭建工具往往也提供 Mirror 服务，比如 Nexus 就可以让同一个 URL,既用作 internalrepository，又使它成为所有 repository 的 Mirror。

如果 仓库 X 可以提供 仓库 Y 存储的所有内容，那么就可以认为 X 是 Y 的一个镜像。这也意味着，任何一个可以从某个仓库中获得的构件，都可以从它的镜像中获取。举个例子：http://maven.net.cn/content/groups/public/ 是中央仓库 http://repo1.maven.org/maven2/ 在中国的镜像，由于地理位置的因素，该镜像往往能够提供比中央仓库更快的服务。

```xml
<mirror>
  <id>CN</id>
  <name>OSChina Central</name>
  <url>http://maven.oschina.net/content/groups/public/</url>
  <mirrorOf>central</mirrorOf>
</mirror>
```

编辑~/.m2/settings.xml 文件(如果没有该文件，则复制\$M2HOME/conf/settings.xml)。添加代理配置如下：

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

## 坐标

说到 Maven 的坐标，我们首先就需要想到 GAV ，即 groupId artifactId version。由这三个属性就可以唯一确定一个 jar 包了。其中每个属性的意义如下：

- groupId：表示一个团体，可以是公司、组织等

- artifactId：表示团体下的某个项目

- version：表示某个项目的版本号

他们之间的关系是一对多的，即每个团体下可以有多个项目，每个项目可以有多个版本号，可以用下面这张图来表示：

![](https://mmbiz.qpic.cn/mmbiz_png/GtXvavW2UlwyGfDVvuLSpndp2xBreDuF94QUz56jsKxmzTrWX194dTLDLWg9SQ8PDibZ1nP8OHwm9guVicC1aPjQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 运行与构建

## 构建基础

生命周期是由一组有顺序的阶段构成的一个整体，这么说可能有点绕，那让我们来关注他里面的几个重要的点：

- 一组：指的是可能有多个
- 顺序：指的是按照顺序执行，执行某一个阶段的指令时会依次先执行该阶段之前的指令
- 阶段：指的是具体要执行的内容

例如 Maven 有三个内置的构建生命周期： default， clean 和 site。每个生命周期都由一系列的阶段所构成，比如 default 生命周期的一个简易阶段如下，完整的生命周期请参考官方文档：

![](https://mmbiz.qpic.cn/mmbiz_png/GtXvavW2UlwyGfDVvuLSpndp2xBreDuFibAX42NbRaUTIEicW7DtSPiblI0NpXpoa5mHn31ZkiaQGDHMKx7zrIMSmg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上图中的每一个节点都是一个 阶段 ，阶段的执行是按顺序的，一个阶段执行完成之后才会执行下一个阶段。比如我们执行了一个如下的指令：

```sh
mvn install
```

他实际会执行 install 阶段之前的所有阶段，然后才会执行 install 阶段本身。

## Run & Package | 运行与打包

如果需要在 Maven 中直接运行某个类中的 Main 方法：

```sh
$ mvn exec:java -Dexec.mainClass="com.example.Main"
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

## Resource | 资源处理

### Resource Directories | 资源文件夹

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

譬如如果我们要用 Maven 构建一个 Web 项目，会在 src/main 目录下构建一个选择包含或者忽视文件或者目录：

```xml
<project>
  ...
  <build>
    ...
    <resources>
      <resource>
        <directory>src/my-resources</directory>
        <excludes>
          <exclude>**/*.bmp</exclude>
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

### Filter | 过滤与内容替换

## Test| 测试

Maven 本身并不是一个单元测试框架，它只是在构建执行到特定生命周期阶段的时候，通过插件来执行 JUnit 或者 TestNG 的测试用例。这个插件就是 maven-surefire-plugin，也可以称为测试运行器(Test Runner)，它能兼容 JUnit 3、JUnit 4 以及 TestNG。在默认情况下，maven-surefire-plugin 的 test 目标会自动执行测试源码路径(默认为 src/test/java/)下所有符合一组命名模式的测试类。这组模式为：

- \*_/Test_.java：任何子目录下所有命名以 Test 开关的 Java 类。
- \**/*Test.java：任何子目录下所有命名以 Test 结尾的 Java 类。
- \**/*TestCase.java：任何子目录下所有命名以 TestCase 结尾的 Java 类。

Maven 中使用 package、install 等命令时会自动调用 Test 组件，`mvn package -DskipTests`命令可以跳过测试。也可以在插件配置的时候设置跳过：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.5</version>
    <configuration>
        <skipTests>true</skipTests>
    </configuration>
</plugin>
```

maven-surefire-plugin 提供了一个 test 参数让 Maven 用户能够在命令行指定要运行的测试用例。如：

```sh
$ mvn test -Dtest=RandomGeneratorTest
```

也可以使用通配符：

```sh
$ mvn test -Dtest=Random*Test
```

或者也可以使用“，”号指定多个测试类：

```sh
$ mvn test -Dtest=Random*Test,AccountCaptchaServiceTest
```

如果由于历史原因，测试类不符合默认的三种命名模式，可以通过 pom.xml 设置 maven-surefire-plugin 插件添加命名模式或排除一些命名模式。

```xml
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

## 插件

插件是 Maven 的核心，所有执行的操作都是基于插件来完成的。为了让一个插件中可以实现众多的相类似的功能，Maven 为插件设定了目标，一个插件中有可能有多个目标。其实生命周期中的每个阶段都是由插件的一个具体目标来执行的。例如可以用下面的方式配置一个插件：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.2.1</version>
            <!-- 配置执行 -->
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>jar-no-fork</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

配置目标 goal 的目的是：这样在执行 mvnpackage 的时候，就会自动执行 mvn source:jar-no-fork 了，jar-no-fork 这个目标是用来进行源码打包的。除了可以在 build 元素中配置插件，当然也可以在 parent 项目中，用 pluginManagement 来配置，然后在子项目继承即可使用。

# Dependence Management | 依赖管理

Maven 核心特点之一是依赖管理。一旦我们开始处理多模块工程(包含数百个子模块或者子工程)的时候，模块间的依赖关系就变得非常复杂，管理也变得很困难。

## 依赖声明与冲突管理

依赖传递很好理解，假设 B 依赖于 C，当 A 需要依赖 B 时，则 A 自动获得了对 C 的依赖。依赖传递有时非常好，当我们需要依赖很多 jar 包时，我们可以声明一个包来依赖所有的 jar，然后只要依赖这个包就可以了。但是有时又很麻烦，因为很可能会造成依赖的冲突。当同一个项目中由于不同的 jar 包依赖了相同的 jar 包，此时就会发生依赖冲突的情况，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GtXvavW2UlwyGfDVvuLSpndp2xBreDuFVjNAjnWhVR26OumcRMdiaQld7hyneYmP2OpZ1h41BSUTQiaiayZjiciaMOQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当项目中依赖了 a 和 c，而 a 和 c 都依赖了 b，这时就造成了冲突。为了避免冲突的产生，Maven 使用了两种策略来解决冲突，分别是短路优先和声明优先。短路优先是从项目一直到最终依赖的 jar 的距离，哪个距离短就依赖哪个，距离长的将被忽略掉。例如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GtXvavW2UlwyGfDVvuLSpndp2xBreDuFaZSKSoXYPWzMiap1VVTHF0g2LlHk29cWnllqXkpej2WQlImM0qsaPicA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

声明优先的意思是，通过 jar 包声明的顺序来决定使用哪个，最先声明的 jar 包总是被选中，后声明的 jar 包则会被忽略，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GtXvavW2UlwyGfDVvuLSpndp2xBreDuFcbfic0Hdw1CsICryqWtKPNKUicgs2uVBNonvcibOgC2He1YiaFEEr2FajQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 依赖排除

如果我们只想引用我们直接依赖的 jar 包，而不想把间接依赖的 jar 包也引入的话，那可以使用依赖排除的方式，将间接引用的 jar 包排除掉，如下面的配置所示：

```xml
<exclusions>
    <exclusion>
        <groupId>
            excluded.groupId
        </groupId>
        <artifactId>
            excluded-artifactId
        </artifactId>
    </exclusion>
</exclusions>
```

如果我们启动项目时报错：NoSuchMethodError，那么极有可能是依赖冲突，可以通过 dependency 命令来查看依赖列表：

```sh
$ mvn dependency:tree -Dverbose
```

## 项目聚合与继承

在 Maven 中，parent 模块组织好 childA 和 childB，叫做"聚合"，多个模块联合编译。实现起来很简单，只需要在 parent 的 pom 文件里加入以下内容：

```xml
<packaging>pom</packaging>
<modules>
    <module>module-1</module>
    <module>module-2</module>
    <module>module-3</module>
</modules>
```

这样只是告诉 Maven 编译器，在读取 parent 的 POM 文件时去找到 childA 和 childB，但还是会分别去编译他们引入的依赖。这样就会导致 POM 文件引入的包重复；于是我们引入了"继承"的概念，也就是形成"父子"关系，子 POM 可以引用到父 POM 中引入的依赖。Maven 的继承特性也会继承父 pom 中的依赖，假设我们定义了一个父 pom：

```xml
<groupId>wx</groupId>
<artifactId>maven-parent</artifactId>
<version>0.0.1-SNAPSHOT</version>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.30</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

然后在子 pom 中引入这个父 pom：

```xml
<!-- 指定parent，说明是从哪个pom继承 -->
<parent>
    <groupId>wx</groupId>
    <artifactId>maven-parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <!-- 指定相对路径 -->
    <relativePath>../maven-parent</relativePath>
</parent>

<!-- 只需要指明groupId + artifactId，就可以到父pom找到了，无需指明版本 -->
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
</dependencies>
```

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
