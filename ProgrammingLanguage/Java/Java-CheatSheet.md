[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Java CheatSheet | Java 语法速览与实践清单

当我们谈起 Java 的时候，往往是将其作为一门编程语言来讨论；然而编程语言的特性只是 Java 架构的某部分，保障其平台独立性的一系列底层架构也是 Java 不可分割的组成。宏观来看，我们认为 Java 主要包含以下四个部分：Java 编程语言、Java 类文件格式、Java API 以及 JVM。当我们在进行 Java 开发时，我们使用 Java 编程语言来编写代码，然后将其编译为 Java 类文件，最终在 JVM 中执行这些类文件；目前我们也可以使用 Gradle、Kotlin 等其他优秀的语言来编写 Java 应用程序。而 JVM 与 Java 平台的核心库就构成了我们所熟知的 Java Runtime Environment(JRE)：

![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2017/8/1/java.png)

## 版本

2018 年 3 月 21 日，Oracle 官方宣布 Java 10 正式发布。这是 Java 大版本周期变化后的第一个正式发布版本。需要注意的是 Java 9 和 Java 10 都不是 LTS 版本。和过去的 Java 大版本升级不同，这两个只有半年左右的开发和维护期。而未来的 Java 11，也就是 18.9 LTS，才是 Java 8 之后第一个 LTS 版本。

```sh
/jdk-10/bin$ ./java -version

openjdk version "10" 2018-03-20

OpenJDK Runtime Environment 18.3 (build 10+46)

OpenJDK 64-Bit Server VM 18.3 (build 10+46, mixed mode)
```

这种发布模式已经得到了广泛应用，一个成功的例子就是 Ubuntu Linux 操作系统，在偶数年 4 月的发行版本为 LTS，会有很长时间的支持。如 2014 年 4 月份发布的 14.04 LTS，Canonical 公司和社区支持到 2019 年。类似的，Node.js，Linux kernel，Firefox 也采用类似的发布方式。

Java 未来的发布周期，将每半年发布一个大版本，每个季度发布一个中间特性版本。这样可以把一些关键特性尽早合并入 JDK 之中，快速得到开发者反馈，可以在一定程度上避免 Java 9 两次被迫推迟发布日期的尴尬。

# Syntax | 语法基础

![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2017/8/3/68747470733a2f2f73342e706f7374696d672e6f72672f6a7a7668306b6977642f68656c6c6f2e706e67.png)

## 条件选择

## 循环

```java
do {
        System.out.println("Count is: " + count);
        count++;
    } while (count < 11);
```

# 数据结构

## String | 字符串

```java
// 字符串比较
boolean result = str1.equals(str2);
boolean result = str1.equalsIgnoreCase(str2);
```

### 模板字符串

```jav
for (int i=0;i<str1.length();i++){
    char aChar = str1.charAt(i);
}
```

### 字符串操作

```java
// 搜索与检索
int result = str1.indexOf(str2);
int result = str1.indexOf(str2,5);
String index = str1.substring(14);

// 大小写转化
String strUpper = str1.toUpperCase();
String strLower = str1.toLowerCase();

// 首尾空格移除
String str1 = "     asdfsdf   ";
str1.trim(); //asdfsdf
// 移除全部空格
str1.replace(" ","");

// 字符串反转
String str1 = "whatever string something";
StringBuffer str1buff = new StringBuffer(str1);
String str1rev = str1buff.reverse().toString();

// 字符串转化为数组
String str = "tim,kerry,timmy,camden";
String[] results = str.split(",");
```

#  数据结构

## 数组

### 构建与检索

```java
// 数组复制
int[] myArray = new int[10];

int[] tmp = new int[myArray.length + 10];
System.arraycopy(myArray, 0, tmp, 0, myArray.length);
myArray = tmp;

// 将 List 转化为数组
String[] stockArr = new String[stockList.size()];
stockArr = stockList.toArray(stockArr);

// 将 Stream 转化为数组
Stream<String> stringStream = Stream.of("a", "b", "c");

String[] stringArray = stringStream.toArray(size -> new String[size]);
String[] stringArray = stringStream.toArray(String[]::new);

Arrays.stream(stringArray).forEach(System.out::println);
```

####  创建映射集合:

```
        HashMap map = new HashMap();
        map.put(key1,obj1);
        map.put(key2,obj2);
        map.put(key2,obj2);
```

####  数组排序:

```
       int[] nums = {1,4,7,324,0,-4};
       Arrays.sort(nums);
       System.out.println(Arrays.toString(nums));
```

####  列表排序:

```
        List<String> unsortList = new ArrayList<String>();

        unsortList.add("CCC");
        unsortList.add("111");
        unsortList.add("AAA");
        Collections.sort(unsortList);
```

####  列表搜索:

```
int index = arrayList.indexOf(obj);
```

#### finding an object by value in a hashmap:

```
hashmap.containsValue(obj);
```

#### finding an object by key in a hashmap:

```
hashmap.containsKey(obj);
```

####  二分搜索:

```
int[] nums = new int[]{7,5,1,3,6,8,9,2};
Arrays.sort(nums);
int index = Arrays.binarySearch(nums,6);
System.out.println("6 is at index: "+ index);
```

#### arrayList  转化为  array:

```
Object[] objects = arrayList.toArray();
```

####  将  hashmap  转化为  array:

```
Object[] objects = hashmap.entrySet().toArray();
```

##  时间与日期类型

####  打印时间与日期:

```
Date todaysDate = new Date(); //todays date
SimpleDateFormat formatter = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss"); //date format
String formattedDate = formatter.format(todaysDate);
System.out.println(formattedDate);
```

####  将日期转化为日历:

```
Date mDate = new Date();
Calendar mCal = Calendar.getInstance();
mCal.setTime(mDate);
```

####  将  calendar  转化为  date:

```
Calendar mCal = Calendar.getInstance();
Date mDate = mDate.getTime();
```

####  字符串解析为日期格式:

```
public void StringtoDate(String x) throws ParseException{
String date = "March 20, 1992 or 3:30:32pm";
DateFormat df = DateFormat.getDateInstance();
Date newDate = df.parse(date);

    }
```

#### date arithmetic using date objects:

```
Date date = new Date();
long time = date.getTime();
time += 5*24*60*60*1000; //may give a numeric overflow error on IntelliJ IDEA
Date futureDate = new Date(time);

System.out.println(futureDate);
```

#### date arithmetic using calendar objects:

```
Calendar today = Calendar.getInstance();
today.add(Calendar.DATE,5);
```

#### difference between two dates:

```
 long diff = time1 - time2;
 diff = diff/(1000*60*60*24);
```

#### comparing dates:

```
 boolean result = date1.equals(date2);
```

#### getting details from calendar:

```
Calendar cal = Calendar.getInstance();
cal.get(Calendar.MONTH);
cal.get(Calendar.YEAR);
cal.get(Calendar.DAY_OF_YEAR);
cal.get(Calendar.WEEK_OF_YEAR);
cal.get(Calendar.DAY_OF_MONTH);
cal.get(Calendar.DAY_OF_WEEK_IN_MONTH);
cal.get(Calendar.DAY_OF_MONTH);
cal.get(Calendar.HOUR_OF_DAY);
```

#### calculating the elapsed time:

```
long startTime = System.currentTimeMillis();
//times flies by..
long finishTime =  System.currentTimeMillis();
long timeElapsed = startTime-finishTime;
System.out.println(timeElapsed);
```

##  正则表达式

####  使用  REGEX  寻找匹配字符串:

```
String pattern = "[TJ]im";
       Pattern regPat = Pattern.compile(pattern,Pattern.CASE_INSENSITIVE);
       String text = "This is Jim and that's Tim";
       Matcher matcher = regPat.matcher(text);

       if (matcher.find()){

           String matchedText = matcher.group();
           System.out.println(matchedText);
       }
```

####  替换匹配字符串:

```
    String pattern = "[TJ]im";
       Pattern regPat = Pattern.compile(pattern,Pattern.CASE_INSENSITIVE);
       String text = "This is jim and that's Tim";
       Matcher matcher = regPat.matcher(text);
       String text2 = matcher.replaceAll("Tom");
       System.out.println(text2);
```

####  使用  StringBuffer  替换匹配字符串:

```
 Pattern p = Pattern.compile("My");
       Matcher m = p.matcher("My dad and My mom");
       StringBuffer sb = new StringBuffer();
       boolean found = m.find();

       while(found){
           m.appendReplacement(sb,"Our");
           found = m.find();

       }
        m.appendTail(sb);
        System.out.println(sb);
```

####  打印所有匹配次数:

```
String pattern = "\\sa(\\w)*t(\\w)*"; //contains "at"
      Pattern regPat = Pattern.compile(pattern);
      String text = "words something at atte afdgdatdsf hey";
      Matcher matcher = regPat.matcher(text);
      while(matcher.find()){


          String matched = matcher.group();
          System.out.println(matched);
      }
```

####  打印包含固定模式的行:

```
 String pattern = "^a";
      Pattern regPat = Pattern.compile(pattern);
      Matcher matcher = regPat.matcher("");
        BufferedReader reader = new BufferedReader(new FileReader("file.txt"));
        String line;
        while ((line = reader.readLine())!= null){
            matcher.reset(line);
            if (matcher.find()){
                System.out.println(line);
            }
        }
```

####  匹配新行:

```
String pattern = "\\d$"; //any single digit
     String text = "line one\n line two\n line three\n";
     Pattern regPat = Pattern.compile(pattern, Pattern.MULTILINE);
     Matcher matcher = regPat.matcher(text);
     while (matcher.find()){

         System.out.println(matcher.group());


     }
```

#### regex:

- beginning of a string: ^
- end of a string: \$
- 0 or 1 times: ?
- 0 or more times:  (\*) //without brackets
- 1 or more times: +
- alternative characters: [...]
- alternative patterns: |
- any character: .
- a digit: \d
- a non-digit: \D
- whitespace: \s
- non-whitespace: \S
- word character: \w
- non word character: \W

##  数字与数学操作处理

#### built-in types:

![alt tag](https://s3.postimg.org/7ihyxdvar/Ekran_Resmi_2017_03_04_10_03_48.png)

- byte: 8bits, Byte
- short: 16bits, Short
- long: 64bits, Long
- float: 32bits, Float

####  判断字符串是否为有效数字:

```
  String str = "dsfdfsd54353%%%";

     try{

         int result = Integer.parseInt(str);

     }

     catch (NumberFormatException e){
         System.out.println("not valid");
     }
```

####  比较  Double:

```
Double a = 4.5;
      Double b= 4.5;

      boolean result = a.equals(b);

      if (result) System.out.println("equal");
```

#### rounding:

```
double doubleVal = 43.234234200000000234040324;
       float floatVal = 2.98f;

      long longResult = Math.round(doubleVal);
      int intResult = Math.round(floatVal);

        System.out.println(longResult + " and " + intResult); // 43 and 3
```

####  格式化数字:

```
double value = 2343.8798;
        NumberFormat numberFormatter;
        String formattedValue;
        numberFormatter = NumberFormat.getNumberInstance();
        formattedValue = numberFormatter.format(value);
        System.out.format("%s%n",formattedValue); //2.343,88
```

####  格式化货币:

```
double currency = 234546457.99;
       NumberFormat currencyFormatter;
       String formattedCurrency;

       currencyFormatter = NumberFormat.getCurrencyInstance();

       formattedCurrency = currencyFormatter.format(currency);

        System.out.format("%s%n",formattedCurrency); // $ 234.546.457,99
```

####  二进制、八进制、十六进制转换:

```
int val = 25;
String binaryStr = Integer.toBinaryString(val);
String octalStr = Integer.toOctalString(val);
String hexStr = Integer.toHexString(val);
```

####  随机数生成:

```
double rn = Math.random();
        int rint = (int) (Math.random()*10); // random int between 0-10

        System.out.println(rn);
        System.out.println(rint);
```

####  计算三角函数:

```
double cos = Math.cos(45);
        double sin = Math.sin(45);
        double tan = Math.tan(45);
```

####  计算对数

```
double logVal = Math.log(125.5);
```

#### Math library:

[![Ekran Resmi 2017-03-04 10.42.52.png](https://s27.postimg.org/fuya7a83n/Ekran_Resmi_2017_03_04_10_42_52.png)](https://postimg.org/image/f5fhux7jz/)

[![library-calls.png](https://s29.postimg.org/ux3o2zijb/library_calls.png)](https://postimg.org/image/ow5z5wvwz/)

##  输入输出操作:

####  从输入流读取:

```
//throw IOexception first

BufferedReader inStream = new BufferedReader(new InputStreamReader(System.in));
      String inline ="";
      while (!(inline.equalsIgnoreCase("quit"))){
          System.out.println("prompt> ");
          inline=inStream.readLine();
      }
```

####  格式化输出:

```
StringBuffer buffer = new StringBuffer();
      Formatter formatter = new Formatter(buffer, Locale.US);
      formatter.format("PI: "+Math.PI);
        System.out.println(buffer.toString());
```

#### formatter format calls:

[![Ekran Resmi 2017-03-04 11.21.45.png](https://s24.postimg.org/6st8e3epx/Ekran_Resmi_2017_03_04_11_21_45.png)](https://postimg.org/image/qanvu1bnl/)

####  打开文件:

```
BufferedReader br = new BufferedReader(new FileReader(textFile.txt)); //for reading
    BufferedWriter bw = new BufferedWriter(new FileWriter(textFile.txt)); //for writing
```

####  读取二进制数据:

InputStream is = new FileInputStream(fileName);

```
    int offset = 0;
    int bytesRead = is.read(bytes, ofset, bytes.length-offset);
```

####  文件随机访问:

```
 File file = new File(something.bin);
    RandomAccessFile raf = new RandomAccessFile(file,"rw");
    raf.seek(file.length());
```

####  读取  Jar/zip/rar  文件:

```
ZipFile file =new ZipFile(filename);
    Enumeration entries = file.entries();
    while(entries.hasMoreElements()){

        ZipEntry entry = (ZipEntry) entries.nextElement();
        if (entry.isDirectory()){
            //do something
        }
        else{
            //do something
        }
    }
    file.close();
```

##  文件与目录

####  创建文件:

```
File f = new File("textFile.txt");
boolean result = f.createNewFile();
```

####  文件重命名:

File f = new File("textFile.txt");

```
File newf = new File("newTextFile.txt");
boolean result = f.renameto(newf);
```

####  删除文件:

```
File f = new File("somefile.txt");
f.delete();
```

####  改变文件属性:

```
File f = new File("somefile.txt");
f.setReadOnly(); // making the file read only
f.setLastModified(desired time);
```

####  获取文件大小:

File f = new File("somefile.txt");

```
long length = file.length();
```

####  判断文件是否存在:

```
File f = new File("somefile.txt");
boolean status = f.exists();
```

####  移动文件:

```
File f = new File("somefile.txt");
File dir = new File("directoryName");
boolean success = f.renameTo(new File(dir, file.getName()));
```

####  获取绝对路径:

```
File f = new File("somefile.txt");
File absPath = f.getAbsoluteFile();
```

####  判断是文件还是目录:

```
File f = new File("somefile.txt");
    boolean isDirectory = f.isDirectory();
    System.out.println(isDirectory); //false
```

####  列举目录下文件:

```
File directory = new File("users/ege");
    String[] result = directory.list();
```

####  创建目录:

```
boolean result = new File("users/ege").mkdir();
```

##  网络客户端

####  服务器连接:

```
String serverName = "www.egek.us";
    Socket socket = new Socket(serverName, 80);
    System.out.println(socket);
```

####  网络异常处理:

```
try {
            Socket sock = new Socket(server_name, tcp_port);
            System.out.println("Connected to " + server_name);
        sock.close(  );

    } catch (UnknownHostException e) {
        System.err.println(server_name + " Unknown host");
        return;
    } catch (NoRouteToHostException e) {
        System.err.println(server_name + " Unreachable" );
        return;
    } catch (ConnectException e) {
        System.err.println(server_name + " connect refused");
        return;
    } catch (java.io.IOException e) {
        System.err.println(server_name + ' ' + e.getMessage(  ));
        return;
    }
```

##  包与文档

####  创建包:

```
package com.ege.example;
```

####  使用  JavaDoc  注释某个类:

```
javadoc -d \home\html
    -sourcepath \home\src
    -subpackages java.net
```

#### Jar  打包:

```
jar cf project.jar *.class
```

####  运行  Jar:

```
java -jar something.jar
```

##  排序算法

- Bubble Sort
- Linear Search
- Binary Search
- Selection Sort
- Insertion Sort

[Over here](https://github.com/egek92/SortAlgorithms)

# 文件路径

```java
{Thread.currentThread().getContextClassLoader() / SomeClass.class}.
{getResource("").getFile() / getResourceAsStream()}
```

# 类与对象

## 实例化

### 单例模式

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

## Interface

### Functional Interface & Lambda

Java 原本作为纯粹的面向对象的语言，需要对 Lambda 表达式特性进行支持，其实是基于了一种特殊的函数式接口。换言之，`()->{}` 这样的语法本质上还是继承并且实现了一个接口。FI 的定义其实很简单：任何接口，如果只包含 唯一 一个抽象方法，那么它就是一个 FI。为了让编译器帮助我们确保一个接口满足 FI 的要求(也就是说有且仅有一个抽象方法)，Java8 提供了@FunctionalInterface 注解。以 Runnble 为例：

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

常见的内置函数式接口还有如下：

```java
Predicate<String> predicate = (s) -> s.length() > 0;

Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);
```

闭包一般指存在自由变量的代码块，它与对象类似，都是用来描述一段代码与其环境的关系。在 Java 中，Lambda 表达式就是闭包。Lambda 表达式本身是构造了一个继承自某个函数式接口的子类，所以可以用父类指针指向它。Java 中本质上闭包中是采用的值捕获，即不可以在闭包中使用可变对象。但是它实际上是允许捕获事实上不变量，譬如不可变的 ArrayList，只是指针指向不可变罢了。

Lambda 表达式还可以进一步简化为方法引用(Method References)，一共有四种形式的方法引用：

```java
// 静态方法引用
List<Integer> ints = Arrays.asList(1, 2, 3);
ints.sort(Integer::compare);

// 某个特定对象的实例方法
words.forEach(System.out::println);

// 某个类的实例方法
words.stream().map(word -> word.length()); // lambda
words.stream().map(String::length); // method reference

// 构造函数引用
words.stream().map(word -> {
return new StringBuilder(word);
});
// constructor reference
words.stream().map(StringBuilder::new);
```

## Stream

Java 8 API 添加了一个新的抽象称为流 Stream，可以让你以一种声明的方式处理数据；Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选，排序，聚合等。

Stream（流）是一个来自数据源的元素队列并支持聚合操作，元素是特定类型的对象，形成一个队列；Java 中的 Stream 并不会存储元素，而是按需计算。元素流在管道中经过中间操作(intermediate operation)的处理，最后由最终操作(terminal operation)得到前面处理的结果。

```sh
+--------------------+       +------+   +------+   +---+   +-------+
| stream of elements +-----> |filter+-> |sorted+-> |map+-> |collect|
+--------------------+       +------+   +------+   +---+   +-------+
```

最简流程的描述如下：

```java
List<Integer> transactionsIds =
widgets.stream()
             .filter(b -> b.getColor() == RED)
             .sorted((x,y) -> x.getWeight() - y.getWeight())
             .mapToInt(Widget::getWeight)
             .sum();
```

和以前的 Collection 操作不同，Stream 操作还有两个基础的特征：

- Pipelining: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化，比如延迟执行(laziness)和短路( short-circuiting)。
- 内部迭代： 以前对集合遍历都是通过 Iterator 或者 For-Each 的方式, 显式的在集合外部进行迭代，这叫做外部迭代。 Stream 提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

## 流构建

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");

// 创建串行流
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());

// 创建并行流，获取空字符串的数量
int count = strings.parallelStream().filter(string -> string.isEmpty()).count();

// 构建空流
Stream<String> streamEmpty = Stream.empty();

// 构建数值流
IntStream intStream = IntStream.range(1, 3); // 1, 2
LongStream longStream = LongStream.rangeClosed(1, 3);
Random random = new Random();
DoubleStream doubleStream = random.doubles(3);

// 构建数组流
Stream<String> streamOfArray = Stream.of("a", "b", "c");
String[] arr = new String[]{"a", "b", "c"};
Stream<String> streamOfArrayFull = Arrays.stream(arr);
Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3);

// 构建文件流
Path path = Paths.get("C:\\file.txt");
Stream<String> streamOfStrings = Files.lines(path);
Stream<String> streamWithCharset =
  Files.lines(path, Charset.forName("UTF-8"));
```

## Transform

```java
// map 方法用于映射每个元素到对应的结果
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);

// 获取对应的平方数
List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());

// filter 方法用于通过设置的条件过滤出元素。以下代码片段使用 filter 方法过滤出空字符串
int count = strings.stream().filter(string -> string.isEmpty()).count();

// limit 方法用于获取指定数量的流。 以下代码片段使用 limit 方法打印出 10 条数据
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);

// sorted 方法用于对流进行排序
Random random = new Random();
random.ints().limit(10).sorted().forEach(System.out::println);
```

## Collectors | 归约操作

```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");

List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
System.out.println("筛选列表: " + filtered);

String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
System.out.println("合并字符串: " + mergedString);
```

```java
// 提取为 Map
IntStream.range(0, alphabet.size())
         .boxed()
         .collect(toMap(alphabet::get, i -> i));
```

# 数据结构

## 数组

```java
// Array 转化为 List
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);

// List 转化为 Array
List<String> stockList = new ArrayList<String>();
...
String[] stockArr = new String[stockList.size()];
stockArr = stockList.toArray(stockArr);
```

## 时间与日期

# 集合类型

## Map

### 遍历与检索

```java
// 遍历集合
for (Iterator it = map.entrySet().iterator();it.hasNext();){
    Map.Entry entry = (Map.Entry)it.next();
    Object key = entry.getKey();
    Object value = entry.getValue();
}
```

# Network | 网络

# Storage | 存储

## 流

# Todos

- https://github.com/in28minutes/java-cheat-sheet
