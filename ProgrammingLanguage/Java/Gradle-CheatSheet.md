[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Introduction

## Terminology(术语与关系理解)

在 Grade 中，我们常见的几个关键术语有 Project、Plugin 以及 Task。和 Maven 一样，Gradle 只是提供了构建项目的一个框架，真正起作用的是 Plugin。Gradle 在默认情况下为我们提供了许多常用的 Plugin，其中包括有构建 Java 项目的 Plugin，还有 War，Ear 等。与 Maven 不同的是，Gradle 不提供内建的项目生命周期管理，只是 Java Plugin 向 Project 中添加了许多 Task，这些 Task 依次执行，为我们营造了一种如同 Maven 般项目构建周期。换言之，Project 为 Task 提供了执行上下文，所有的 Plugin 要么向 Project 中添加用于配置的 Property，要么向 Project 中添加不同的 Task。一个 Task 表示一个逻辑上较为独立的执行过程，比如编译 Java 源代码，拷贝文件，打包 Jar 文件，甚至可以是执行一个系统命令或者调用 Ant。另外，一个 Task 可以读取和设置 Project 的 Property 以完成特定的操作。

## Advantage(优势)

### [Ant vs Maven vs Gradle](http://blog.csdn.net/napolunyishi/article/details/39345995)

#### Ant with Ivy

Ant 是第一个“现代”构建工具，在很多方面它有些像 Make。2000 年发布，在很短时间内成为 Java 项目上最流行的构建工具。它的学习曲线很缓，因此不需要什么特殊的准备就能上手。它基于过程式编程的 idea。在最初的版本之后，逐渐具备了支持插件的功能。主要的不足是用 XML 作为脚本编写格式。 XML，本质上是层次化的，并不能很好地贴合 Ant 过程化编程的初衷。Ant 的另外一个问题是，除非是很小的项目，否则它的 XML 文件很快就大得无法管理。后来，随着通过网络进行依赖管理成为必备功能，Ant 采用了 Apache Ivy。

Ant 的主要优点在于对构建过程的控制上。Ivy 的依赖需要在 ivy.xml 中指定。我们的例子很简单，只需要依赖 JUnit 和 Hamcrest。

```xml
<ivy-module version="2.0">
    <info organisation="org.apache" module="java-build-tools"/>
    <dependencies>
        <dependency org="junit" name="junit" rev="4.11"/>
        <dependency org="org.hamcrest" name="hamcrest-all" rev="1.3"/>
    </dependencies>
</ivy-module>
```

现在我们来创建 Ant 脚本，任务只是编译一个 Jar 文件。最终文件是下面的 build.xml。

```xml
<project xmlns:ivy="antlib:org.apache.ivy.ant" name="java-build-tools" default="jar">

    <property name="src.dir" value="src"/>
    <property name="build.dir" value="build"/>
    <property name="classes.dir" value="${build.dir}/classes"/>
    <property name="jar.dir" value="${build.dir}/jar"/>
    <property name="lib.dir" value="lib" />
    <path id="lib.path.id">
        <fileset dir="${lib.dir}" />
    </path>

    <target name="resolve">
        <ivy:retrieve />
    </target>

    <target name="clean">
        <delete dir="${build.dir}"/>
    </target>

    <target name="compile" depends="resolve">
        <mkdir dir="${classes.dir}"/>
        <javac srcdir="${src.dir}" destdir="${classes.dir}" classpathref="lib.path.id"/>
    </target>

    <target name="jar" depends="compile">
        <mkdir dir="${jar.dir}"/>
        <jar destfile="${jar.dir}/${ant.project.name}.jar" basedir="${classes.dir}"/>
    </target>

</project>
```

首先，我们设置了几个属性，然后是一个接一个的 task。我们用 Ivy 来处理依赖，清理，编译和打包，这是几乎所有的 Java 项目都会进行的 task，配置有很多。

运行 Ant task 来生成 Jar 文件，执行下面的命令。

```
ant jar  
```

#### Maven

Maven 发布于 2004 年。目的是解决码农使用 Ant 所带来的一些问题。Maven 仍旧使用 XML 作为编写构建配置的文件格式，但是，文件结构却有巨大的变化。Ant 需要码农将执行 task 所需的全部命令都一一列出，然而 Maven 依靠约定（convention）并提供现成的可调用的目标（goal）。不仅如此，有可能最重要的一个补充是，Maven 具备从网络上自动下载依赖的能力（Ant 后来通过 Ivy 也具备了这个功能），这一点革命性地改变了我们开发软件的方式。

但是，Maven 也有它的问题。依赖管理不能很好地处理相同库文件不同版本之间的冲突（Ivy 在这方面更好一些）。XML 作为配置文件的格式有严格的结构层次和标准，定制化目标（goal）很困难。因为 Maven 主要聚焦于依赖管理，实际上用 Maven 很难写出复杂、定制化的构建脚本，甚至不如 Ant。用 XML 写的配置文件会变得越来越大，越来越笨重。在大型项目中，它经常什么“特别的”事还没干就有几百行代码。

Maven 的主要优点是生命周期。只要项目基于一定之规，它的整个生命周期都能够轻松搞定，代价是牺牲了灵活性。在对 DSL（Domain Specific Languages)的热情持续高涨之时，通常的想法是设计一套能够解决特定领域问题的语言。在构建这方面，DSL 的一个成功案例就是 Gradle。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0

http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.technologyconversations</groupId>
    <artifactId>java-build-tools</artifactId>
    <packaging>jar</packaging>
    <version>1.0</version>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>
        <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-all</artifactId>
            <version>1.3</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
            </plugin>
        </plugins>
    </build>

</project>
```

通过执行下面的命令来运行 Maven goal 生成 Jar 文件。

```xml
mvn package  
```

主要的区别在于 Maven 不需要指定执行的操作。我们没有创建 task，而是设置了一些参数（有哪些依赖，用哪些插件...). Maven 推行使用约定并提供了开箱即用的 goals。Ant 和 Maven 的 XML 文件都会随时间而变大，为了说明这一点，我们加入 CheckStyle，FindBugs 和 PMD 插件来进行静态检查，三者是 Java 项目中使用很普遍的的工具。我们希望将所有静态检查的执行以及单元测试一起作为一个单独的 targetVerify。当然我们还应该指定自定义的 checkstyle 配置文件的路径并且确保错误时能够提示。更新后的 Maven 代码如下：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>2.12.1</version>
    <executions>
        <execution>
            <configuration>
                <configLocation>config/checkstyle/checkstyle.xml</configLocation>
                <consoleOutput>true</consoleOutput>
                <failsOnError>true</failsOnError>
            </configuration>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>findbugs-maven-plugin</artifactId>
    <version>2.5.4</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-pmd-plugin</artifactId>
    <version>3.1</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

通过执行下面的命令来运行 Maven goal，包括单元测试，静态检查，如 CheckStyle，FindBugs 和 PMD。

```xml
mvn verify  
```

#### Gradle

Gradle 结合了前两者的优点，在此基础之上做了很多改进。它具有 Ant 的强大和灵活，又有 Maven 的生命周期管理且易于使用。最终结果就是一个工具在 2012 年华丽诞生并且很快地获得了广泛关注。例如，Google 采用 Gradle 作为 Android OS 的默认构建工具。Gradle 不用 XML，它使用基于 Groovy 的专门的 DSL，从而使 Gradle 构建脚本变得比用 Ant 和 Maven 写的要简洁清晰。Gradle 样板文件的代码很少，这是因为它的 DSL 被设计用于解决特定的问题：贯穿软件的生命周期，从编译，到静态检查，到测试，直到打包和部署。

它使用 Apache Ivy 来处理 Jar 包的依赖。Gradle 的成就可以概括为：约定好，灵活性也高。

```groovy
apply plugin: 'java'
apply plugin: 'checkstyle'
apply plugin: 'findbugs'
apply plugin: 'pmd'

version = '1.0'

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.11'
    testCompile group: 'org.hamcrest', name: 'hamcrest-all', version: '1.3'
}
```

## Reference

### Tutorials&Docs

* [Gradle 入门系列](http://blog.jobbole.com/71999/)
* [Gradle 学习系列](http://www.cnblogs.com/CloudTeng/p/3417762.html)

### Books

* [Gradle 实战](https://lippiouyang.gitbooks.io/gradle-in-action-cn/content/index.html)

### Practice

* [Gradle 奇技淫巧](http://blog.chengyunfeng.com/?p=833&utm_source=tuicool&utm_medium=referral)
* [受用不尽的 Gradle 使用方法与技巧](http://www.tuicool.com/articles/i2Ijiin)

# Quick Start

## Installation

### Windows 系统安装

### 类 Unix 系统安装

首先，先 download 最新版本的 gradle，网址如下：

[http://www.gradle.org/get-started](http://www.gradle.org/get-started)

然后将下载下来的 zip 包放在你要安装的路径上，我安装在

/usr/local/bin；

然后打开电脑上的.bash_profile 文件，输入以下命令：

GRADLE_HOME=/usr/local/bin/gradle-1.8;

export GRADLE_HOME

export PATH=$PATH:$GRADLE_HOME/bin

然后再在 console 上输入以下命令：

source ~/.bash_profile

这样就安装成功啦，可以通过以下命令来查看是否安装成功。

gradle -version

如果提示没有 gradle 命令，则有可能是：

1.GRADLE_HOME 路径可能不对；

2.没有执行 source ~/.bash_profile

## Setup

### Convert Maven Projects(Maven 项目的转化)

The first step is to run `gradle init` in the directory containing the (master) POM. This will convert the Maven build to a Gradle build, generating a `settings.gradle` file and one or more `build.gradle` files. For simpler Maven builds, this is all you need to do. For more complex Maven builds, it may be necessary to manually add functionality on the Gradle side that couldn't be converted automatically.

### Java Project

### Web Application

> * [gradle-spring-mvc-web-project-example](http://www.mkyong.com/spring-mvc/gradle-spring-mvc-web-project-example/)

## Build

### tasks

Gradle 在默认情况下为我们提供了几个常用的 Task，比如查看 Project 的 Properties、显示当前 Project 中定义的所有 Task 等。可以通过一下命令查看 Project 中所有的 Task：

```
gradle tasks
```

输出如下：

```groovy
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build Setup tasks
-----------------
setupBuild - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Help tasks
----------
dependencies - Displays all dependencies declared in root project 'gradle-blog'.
dependencyInsight - Displays the insight into a specific dependency in root project 'gradle-blog'.
help - Displays a help message
projects - Displays the sub-projects of root project 'gradle-blog'.
properties - Displays the properties of root project 'gradle-blog'.
tasks - Displays the tasks runnable from root project 'gradle-blog'.

Other tasks
-----------
copyFile
helloWorld

To see all tasks and more detail, run with --all.

BUILD SUCCESSFUL

Total time: 2.845 secs
```

上面的 other tasks 中列举出来的 task 即是我们自定义的 tasks。

#### Default tasks(默认任务)

Gradle 允许在脚本中配置默认的一到多个任务来响应没有带参数的`gradle`命令：

```groovy
defaultTasks 'clean', 'run'

task clean << {
    println 'Default Cleaning!'
}

task run << {
    println 'Default Running!'
}

task other << {
    println "I'm not a default task!"
}
```

**gradle -q**命令的输出：

```groovy
> gradle -q
Default Cleaning!
Default Running!
```

### properties

在默认情况下，Gradle 已经为 Project 添加了很多 Property，我们可以调用以下命令进行查看：

```
gradle properties
```

输出如下：

```groovy
:properties

------------------------------------------------------------
Root project
------------------------------------------------------------

allprojects: [root project 'gradle-blog']
ant: org.gradle.api.internal.project.DefaultAntBuilder@1342097

buildDir: /home/davenkin/Desktop/gradle-blog/build
buildFile: /home/davenkin/Desktop/gradle-blog/build.gradle
...
configurations: []
convention: org.gradle.api.internal.plugins.DefaultConvention@11492ed
copyFile: task ':copyFile'
...
ext: org.gradle.api.internal.plugins.DefaultExtraPropertiesExtension@1b5d53a
extensions: org.gradle.api.internal.plugins.DefaultConvention@11492ed
...
helloWorld: task ':helloWorld'
...
plugins: [org.gradle.api.plugins.HelpTasksPlugin@7359f7]
project: root project 'gradle-blog'
...
properties: {...}
repositories: []

tasks: [task ':copyFile', task ':helloWorld']
version: unspecified

BUILD SUCCESSFUL

Total time: 2.667 secs
```

# Dependence(依赖管理)

## Repository

```groovy
repositories {
    mavenCentral()          // 定义仓库为maven中心仓库
}
repositories {
    jcenter()               // 定义仓库为jcenter仓库
}
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"      // 定义依赖包协议是maven，地址是公司的仓库地址
    }
}
repositories {                              // 定义本地仓库目录
    flatDir {
        dirs 'lib'
    }
}
repositories {                              // 定义ivy协议类型的仓库
    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```

```groovy
repositories {
 mavenCentral artifactUrls:["file://C:/maven/.m2/repository/"]
 }
```

如果是系统的默认配置的：

```groovy
repositories {
  mavenLocal()
}
```

# Configuration(构建配置)

## Build Script Basics

### artifacts

和 Maven 一样，Gradle 也是通过 artifact 来打包构建的。得益于上述的 Gradle 本身的特性，artifact 在 Gradle 里实现得更灵活一些。看一个例子：

```groovy
bob [13:00]  ᐅ cat userguide/artifacts/uploading/build.gradle

## jar类型的artifact
task myJar(type: Jar)
artifacts {
    archives myJar
}
## file类型的artifact
def someFile = file('build/somefile.txt')
artifacts {
    archives someFile
}

## 根据自定义task来完成artifact
task myTask(type:  MyTaskType) {
    destFile = file('build/somefile.txt')
}
artifacts {
    archives(myTask.destFile) {
        name 'my-artifact'
        type 'text'
        builtBy myTask
    }
}

## 根据自定义task来完成artifact
task generate(type:  MyTaskType) {
    destFile = file('build/somefile.txt')
}
artifacts {
    archives file: generate.destFile, name: 'my-artifact', type: 'text', builtBy: generate
}
```

### publish(发布)

Gradle 构建的项目，发布到仓库中，也非常容易：

```
apply plugin: 'maven'

uploadArchives {
    repositories {
        ivy {
            credentials {
                username "username"
                password "pw"
            }
            url "http://repo.mycompany.com"
        }
    }
}
```

## Properties(属性)

| Name          | Type                                                                                   | Default Value                              |
| ------------- | -------------------------------------------------------------------------------------- | ------------------------------------------ |
| `project`     | [`Project`](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html)           | The `Project` instance                     |
| `name`        | `String`                                                                               | The name of the project directory.         |
| `path`        | `String`                                                                               | The absolute path of the project.          |
| `description` | `String`                                                                               | A description for the project.             |
| `projectDir`  | `File`                                                                                 | The directory containing the build script. |
| `buildDir`    | `File`                                                                                 | `*projectDir*/build`                       |
| `group`       | `Object`                                                                               | `unspecified`                              |
| `version`     | `Object`                                                                               | `unspecified`                              |
| `ant`         | [`AntBuilder`](https://docs.gradle.org/current/javadoc/org/gradle/api/AntBuilder.html) | An `AntBuilder` instance                   |

### Variables

## Task(任务)

> * [more_about_tasks](https://docs.gradle.org/current/userguide/more_about_tasks.html)

Gradle 的 Project 从本质上说只是含有多个 Task 的容器，一个 Task 与 Ant 的 Target 相似，表示一个逻辑上的执行单元。我们可以通过很多种方式定义 Task，所有的 Task 都存放在 Project 的 TaskContainer 中。让我们来看一个最简单的 Task，创建一个 build.gradle 文件，内容如下：

```groovy
task helloWorld << {
   println "Hello World!"
}
```

这里的“<<”表示向 helloWorld 中加入执行代码——其实就是 groovy 代码。Gradle 向我们提供了一整套 DSL，所以在很多时候我们写的代码似乎已经脱离了 groovy，但是在底层依然是执行的 groovy。比如上面的 task 关键字，其实就是一个 groovy 中的方法，而大括号之间的内容则表示传递给 task()方法的一个闭包。除了“<<”之外，我们还很多种方式可以定义一个 Task，我们将在本系列后续的文章中讲到。

在与 build.gradle 相同的目录下执行：

```
gradle helloWorld
```

命令行输出如下：

```
:helloWorld
Hello World!

BUILD SUCCESSFUL

Total time: 2.544 secs
```

在默认情况下，Gradle 将当前目录下的 build.gradle 文件作为项目的构建文件。在上面的例子中，我们创建了一个名为 helloWorld 的 Task，在执行 gradle 命令时，我们指定执行这个 helloWorld Task。这里的 helloWorld 是一个 DefaultTask 类型的对象，这也是定义一个 Task 时的默认类型，当然我们也可以显式地声明 Task 的类型，甚至可以自定义一个 Task 类型。

### Task Creator(任务创建)

Grade 提供了非常灵活的 Task 定义方式，可以适用于不同的应用场景或者编程风格。

```groovy
task(hello) << {
    println "hello"
}

task(copy, type: Copy) {
    from(file('srcDir'))
    into(buildDir)
}

//也可以用字符串作为任务名

task('hello') <<
{
    println "hello"
}

task('copy', type: Copy) {
    from(file('srcDir'))
    into(buildDir)
}

//Defining tasks with alternative syntax

tasks.create(name: 'hello') << {
    println "hello"
}

tasks.create(name: 'copy', type: Copy) {
    from(file('srcDir'))
    into(buildDir)
}
```

上述代码中使用了`<<`符号用来表征`{}`声明的动作是追加在某个任务的末尾，如果使用全声明即是：

```groovy
task printVersion {
//任务的初始声明可以添加first和last动作
    doFirst {
    println "Before reading the project version"
    }

    doLast {
    println "Version: $version"
    }
}
```

#### Locating tasks(任务定位)

### Task Dependence(任务依赖)

# Plugins

## Java

1，使用 Java plugin，只需要在 build.gradle 中加入这句话：

```
apply plugin: 'java'
```

2，了解或设置 Java project 布局。Gradle 和 Maven 一样，采用了“约定优于配置”的方式对 Java project 布局，并且布局方式是和 Maven 一样的，此外，Gradle 还可以方便的自定义布局。在 Gradle 中，一般把这些目录叫做 source set。看下官方的答案：

![gradle source set](http://tech.meituan.com/img/gradle/source_set.png)

这里要注意，每个 plugin 的 source set 可能都不一样。

同样的，Java plugin 还定义好了一堆 task，让我们可以直接使用，比如：clean、test、build 等等。这些 task 都是围绕着 Java plugin 的构建生命周期的：

![](http://tech.meituan.com/img/gradle/javaPluginTasks.png)

图中每一块都是一个 task，箭头表示 task 执行顺序/依赖，比如执行 task jar，那么必须先执行 task compileJava 和 task processResources。另外可以看到，Gradle 的 Java plugin 构建生命周期比较复杂，但是也表明了更加灵活，而且，在项目中，一般只使用其中常用的几个：clean test check build 等等。

gradle 构建过程中，所有的依赖都表现为配置，比如说系统运行时的依赖是 runtime，gradle 里有一个依赖配置叫 runtime，那么系统运行时会加载这个依赖配置以及它的相关依赖。这里说的有点绕，可以简单理解依赖和 maven 类似，只不过 gradle 用 configuration 实现，所以更灵活，有更多选择。下图是依赖配置关系图以及和 task 调用的关系图：

![javaPluginConfigurations](http://tech.meituan.com/img/gradle/javaPluginConfigurations.png)

可以看到，基本和 Maven 是一样的。其实 Gradle 里面这些依赖(scope)都是通过 configuration 来实现的，这里就不细说，有兴趣的可以研究一下官方资料。

## Web Application

# Test(测试)
