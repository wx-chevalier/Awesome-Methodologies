# 主流加固技术

第一代：落地加载
其原理是基于 Java 虚拟机提供的动态加载技术。缺点是 Dex 落地加载，破解成本非常低。

第二代：内存加载
其原理是基于 inline hook 技术对系统 read 和 write 函数添加额外的加解密处理，或直接调用 Dalvik 虚拟机提供的函数，以字节流方式加载。该加固方案虽然避免文件落地的缺陷，但缺点是解密后的 Dex 连续完整的在内存中存在。

第三代：指令抽离
其原理是在类被执行时才进行函数修复，解决了 Dex 内存连续的问题。但由于指令抽离技术方案使用了大量虚拟机内部结构的特性，再加上 Android ROM 厂商复杂的定制，带来了大量的兼容性问题。

第四代：Java2C
Java2C 是目前 Dex 加固保护中安全性最高的方案，与前三代 Dex 加固不同的是它脱离了虚拟提供的动态加载技术。其原理是将 Java 代码翻译成 C 并生对应的二进制文件。

# APK 加载流程

系统为每个 APK 创建进程时，都会通过 PathClassLoader 类进行加载，同时开发者也可以通过 DexClassLoader 动态加载额外的 Dex 文件，有点类似于 dlopen 和 dlsym 函数的作用。PathClassLoader 和 DexClassLoader 两者都承继自 BaseDexClassLoader，最终都是通过 DexFile 完成对 dex 的加载。一般情况下每个 ClassLoader 对应一个 DexFile，但其本身是可以包含多个 DexFile 的，当要加载一个 Class 时，会遍历各个 DexFile。

类 DexFile 底层是通过 JNI 方式实现的，针对 APK 文件(包括 jar 和 zip)和二进制字节流，系统分别提供了 dvmDexFileOpenFromFd 和 dvmDexFileOpenPartial 两个函数进行处理。这两个流程最终目的都是构造出 DexOrJar 结构体，并通过 JNI 接口把该结构体的地址保存到 DexFile 的私有成员变量 mCookie。DexOrJar 结构主要由 RawDexFile 和 JarFile 组成。
