# Java Design Pattern CheatSheet

## 单例模式

- 非延迟加载单例类

```java
public class Singleton {
　　private Singleton(){}
　　private static final Singleton instance = new Singleton();
　　public static Singleton getInstance() {
　　　　return instance;
　　}
}
```

- 简单的同步延迟加载

```java
public class Singleton {

　　private static Singleton instance = null;

　　public static synchronized Singleton getInstance() {
　　　　if (instance == null)
　　　　　　instance ＝ new Singleton();
　　　　return instance;
　　}
}
```

- 双重检查成例延迟加载

```java
public class Singleton {

　　private static volatile Singleton instance = null;

　　public static Singleton getInstance() {
　　　　if (instance == null) {
　　　　　　　　synchronized (Singleton.class) {
　　　　　　　　　　　　if (instance == null) {
　　　　　　　　　　　　　　　　instance ＝ new Singleton();
　　　　　　　　　　　　}
　　　　　　　　}
　　　　}
　　　　return instance;
　　}

}
```

- 类加载器延迟加载

```java
public class Singleton {
　　private static class Holder {
　　  static final Singleton instance = new Singleton();
　　}
　　public static Singleton getInstance() {
　　　　return Holder.instance;
　　}
}
```
