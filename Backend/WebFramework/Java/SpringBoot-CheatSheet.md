# Spring Boot CheatSheet

# 依赖声明与注入

## 依赖声明

### 注解声明

```java
@Component
public class MyComponent{}
```

在Spring2.0之前的版本中，@Repository注解可以标记在任何的类上，用来表明该类是用来执行与数据库相关的操作（即dao对象），并支持自动处理数据库操作产生的异常

在Spring2.5版本中，引入了更多的Spring类注解：@Component,@Service,@Controller。@Component是一个通用的Spring容器管理的单例bean组件。而@Repository, @Service, @Controller就是针对不同的使用场景所采取的特定功能化的注解组件。

因此，当你的一个类被@Component所注解，那么就意味着同样可以用@Repository, @Service, @Controller来替代它，同时这些注解会具备有更多的功能，而且功能各异。

最后，如果你不知道要在项目的业务层采用@Service还是@Component注解。那么，@Service是一个更好的选择。


```java
@Configuration
public class TestConfig {
    @Bean(name="helloClient")
    public HessianProxyFactoryBean helloClient() {
        HessianProxyFactoryBean factory = new HessianProxyFactoryBean();
        ...
        return factory;
    }
}
```

### 作用域与生命周期

Spring 中为Bean定义了5中作用域，分别为singleton（单例）、prototype（原型）、request、session和global session，5种作用域说明如下：

- singleton：单例模式，Spring IoC容器中只会存在一个共享的Bean实例，无论有多少个Bean引用它，始终指向同一对象。Singleton作用域是Spring中的缺省作用域，也可以显示的将Bean定义为singleton模式

- prototype:原型模式，每次通过Spring容器获取prototype定义的bean时，容器都将创建一个新的Bean实例，每个Bean实例都有自己的属性和状态，而singleton全局只有一个对象。根据经验，对有状态的bean使用prototype作用域，而对无状态的bean使用singleton作用域。

- request：在一次Http请求中，容器会返回该Bean的同一实例。而对不同的Http请求则会产生新的Bean，而且该bean仅在当前Http Request内有效。

- session：在一次Http Session中，容器会返回该Bean的同一实例。而对不同的Session请求则会创建新的实例，该bean实例仅在当前Session内有效。

- global Session：在一个全局的Http Session中，容器会返回该Bean的同一个实例，仅在使用portlet context时有效。

可以通过 @Scope 注解来指定作用域。Spring容器可以管理singleton作用域下Bean的生命周期，在此作用域下，Spring能够精确地知道Bean何时被创建，何时初始化完成，以及何时被销毁。而对于prototype作用域的Bean，Spring只负责创建，当容器创建了Bean的实例后，Bean的实例就交给了客户端的代码管理，Spring容器将不再跟踪其生命周期，并且不会管理那些被配置成prototype作用域的Bean的生命周期。Spring中Bean的生命周期的执行是一个很复杂的过程，读者可以利用Spring提供的方法来定制Bean的创建过程。Spring容器在保证一个bean实例能够使用之前会做很多工作：

![image](https://user-images.githubusercontent.com/5803001/47697770-78aa1000-dc47-11e8-9c3c-88b036aa99da.png)

## 配置管理

Spring Boot 的一大特性即是外置所有的配置，并且提供了多种配置访问方式；首先我们可以通过动态地指定配置文件的方式来完成不同环境下的配置文件加载：

```sh
# 在本地跑时，默认是 local，在其它环境跑时，要通过 -Dspring.profiles.active= 来指定
spring.profiles.active=local
```

对于配置文件的读取，可以通过配置类映射的方式：

```java
@Configuration
@PropertySource("classpath:configprops.properties")
@ConfigurationProperties(prefix = "mail")
public class ConfigProperties {
 
    public static class Credentials {
        private String authMethod;
        private String username;
        private String password;
 
       // standard getters and setters
    }
    private String host;
    private int port;
    private String from;
    private Credentials credentials;
    private List<String> defaultRecipients;
    private Map<String, String> additionalHeaders;
  
    // standard getters and setters
}
```

```sh
#Simple properties
mail.host=mailer@mail.com
mail.port=9000
mail.from=mailer@mail.com
 
#List properties
mail.defaultRecipients[0]=admin@mail.com
mail.defaultRecipients[1]=owner@mail.com
 
#Map Properties
mail.additionalHeaders.redelivery=true
mail.additionalHeaders.secure=true
 
#Object properties
mail.credentials.username=john
mail.credentials.password=password
mail.credentials.authMethod=SHA1
```

也可以为配置类添加校验：

```java
@Length(max = 4, min = 1)
private String authMethod;
```
