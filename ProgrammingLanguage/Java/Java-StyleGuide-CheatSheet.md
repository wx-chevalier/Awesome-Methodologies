[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Java StyleGuide CheatSheet | Java 样式指南

当我们敲下每个点号时，都应该考虑：

* 是否会出现空指针？
* 是否会抛出异常？
* 是否在热点区域运行？
* 在哪个线程中执行？
* 是否存在并发锁间隙？
* 是否会并发修改不可见？

```java
// Bad, doesn't compile
switch (value) {
    case 1: int j = 1; break;
    case 2: int j = 2; break;
}

// Good
switch (value) {
    case 1: {
        final int j = 1;
        break;
    }
    case 2: {
        final int j = 2;
        break;
    }

    // Remember:
    default:
        throw new ThreadDeath("That'll teach them");
}
```

Within the switch statement, there is only one scope defined among all the case statements. In fact, these case statements aren’t even really statements, they’re like labels and the switch is a goto call. This means that the variable final int j is defined for all the different cases, regardless if we issue a break or not. Not very intuitive. Which is why it’s always a good idea to create a new, nested scope per case statement via a simple block.
