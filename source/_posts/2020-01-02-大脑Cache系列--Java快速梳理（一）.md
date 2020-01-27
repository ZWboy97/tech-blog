---
title: 大脑Cache系列--Java快速梳理，方便随时load到大脑，减少低级bug (一)
categories:
  - [技术]
  - [大脑Cache]
tags:
  - Java
date: 2020-01-02 17:39:52
---

### 背景

> - Java程序常常会遇到一些蛋疼的bug，最后发现，都是在一些很基础的方面造成的。大量的时间花在调试代码找低级bug上是十分没有性价比的。所以，再系统梳理下Java，是十分必要的。
> - 已经反复学习和使用Java多次了，但只要有段时间没用Java之后，每次使用前都想要重头再梳理一遍。本文章将更注重Java只是的系统性，而不是细节性。
> - 本文是本人大脑的专属Cache，所以逻辑上可能只有我自己能够看懂，见谅。

### 一、目录
- 数据类型，字符串，数组
- 基础语句语法
- 类与对象

<!--more-->

### 二、语法
> - 虽然没那么重要，但既然系统学Java技术了，就得了解下吧。
> - 其实还是有很多基础知识，是自己平时写代码的时候没有注意到的，而这很容易导致一些低级bug，耗费大量的调试时间。

##### 基础
- 编译命令: javac Main.java
- 运行:java Main 
- 版本：java -version

##### 8种基本类型
- 整数：byte，short，int，long
- 浮点：float，double
- Unicode编码的char字符类型
- boolean

##### 整型
- java的整形均是有符号的，需要会计算整型的数据范围
- **注意：在java中，为了保障移植性，整型的范围与实际的机器无关（C和C++中整型范围和目标平台是相关的），JVM解决了不同机器整型之间的差别。**
- byte：1字节，一般很少用，用在底层文件处理，或者占用存储空间的大数组
- short：2字节，和byte用处类似
- int：4字节，最常用
- long：8字节，比如需要表示星球上的居住人数，可以使用long，后缀可以加一个L或者l标识
- 更易读的整数方式：1_000_000, java编译器会自动去除这些下划线。

##### 浮点类型
- float 4字节，有效位6-7位 
- double 8字节，有效位15位
- **默认为double，一般也很少用float（float的精度一般很难满足需求），除非在需要处理或者存储大量数据**
- double后缀D或d，float后缀F或者f
- 特殊浮点数值（一般用不到）
  - 正无穷大：Double.POSITIVE_INFINITY
  - 负无穷大: Double.NEGATIVE_INFINITY
  - 非数字: Double.NaN, 判断用Double.isNaN(n),不能用 == 号（所有非数值都是不相等的）

##### 字符类型char
- Java的char采用Unicode编码
- 'a',"A"的区别
- 码点与代码单元

##### boolean
- **和C++不一样，整型和boolean之间不能相互转换**

##### 大数值 BigInteger BigDecimal
- 不是一个java类型，而是一个java对象，可以表示任意精度的整型和浮点数
- 无法通过运算符计算，需要通过方法进行运算

##### 变量
- 声明->初始化->使用
- Java变量在使用之前必须要进行初始化，否则会编译报错

##### 常量与类常量
- final表示常量，只能被赋值一次，且声明的时候必须初始化
- 常量名推荐使用全大写,`final int NUMBER = 10;`
- static final，类常量，可以在类中额多个方法中使用
- 注意：与C++不同，常量不是通过const声明的，Java中const是保留字，但没有实际作用。

##### 常用的Math静态方法
- 平方根：`double result = Math.sqrt(double x);`
- 幂运算：`double result = Math.pow(x,a);`
- 四舍五入：`long n = Math.round(double x);`
- 随机数: `Math.random()`,返回0-1之间的随机浮点数，比如要取0 - n-1之间的随机数，`int result = (int)(Math.random() * n)`.

##### 类型转换
- 强制类型转换会导致结果被截断成一个完全不同的值，比如(byte)300 的值为 44
- 不要将boolean与任何类型之间做类型转换。

##### 枚举类型
- 自定义枚举类型

```
enum Size {
  SMALL, LARGE, EXTRA_LARGE
};
```
- 使用

```
Size size = Size.SMALL;
```

##### 字符串
- 不是一个基础类型，而是一个预定义类
- **Java 的字符串是不可变的，对字符串的修改，实际上是创建了一个新的String对象，目的是为了复用共享字符串**
- 对字符串修改的需求没有对字符串进行比较的需求大
- **和C++不同，Java中的String不是字符数组，而是一个对象，可以理解为char*指针**
- 字符串比较：一定要使用equal(str)方法，而不能使用 == 来判断
  - 因为Java没有像C++那样，重载了==运算符，==判断的是两个字符串变量是否引用的是同一个位置的。
  - 所以 == 判断的结果是未知的，常常会导致隐藏bug
- 字符串判空,**区分空串和Null串**
  - if(str != null && str.length()!=0)
- 码点与代码单元
  - str.length()返回的是代码单元个数
  - str.codePointCount(0,str.length()),统计的是码点数量
  - str.charAt(n),获取n位置的代码单元
  - **最好避免直接操作char，这太底层了**
  - 关于代码单元与码点：

##### 常用的String API
```java
char charAt(int index)
boolean equals(Object obj)
boolean equalsIgnoreCase(String str)
boolean startsWith(String prefix)，判断是否是以prefix为前缀
boolean endsWith(String suffix)，判断是否以suffix为后缀
boolean indexof(String str), 找到字串str第一次出现的位置，没有则返回-1
int length(),字符串长度
int codePointCount(int startIndex, int endIndex),统计代码点数目
String substring(int begin , int end);
String toLowerCase(), toUpperCase()
Stirng trim(), 删除字符串开始和结尾的空格
String join('divider',CharSequence...emements)
```

##### StringBuilder字符串构造
- 在构造字符串的时候，每次连接字符串都需要构建一个新的String对象，效率比较低，推荐使用StringBuilder
- 使用方式

```
String builder = new StringBuilder(); // 创建一个构造器
builder.append("hello");
builder.append("world"); // 追加字符或者字符串
String str = builder.toString(); // 构造String对象
// 其他api
builder.setCharAt(index,char)
builder.delete(start,end)
builder.insert(offset,string)
```
- StringBuilder的前身是StringBuffer，但是StringBuffer的效率要低一些，因为它是线程安全的，允许多线程操作
- 而StringBuilder是非线程安全的，一般在单线程的应用中使用StringBuilder
- 他们的API是一样的

##### 读取输入
- 从控制台读取基础数据类型

```
Scanner scan = new Scanner(System.in); //创建scanner，并与标准输入关联,System.in的类型为InputStream
scan.nextLine();
scan.next();
scan.nextInt();
scan.nextDouble();
scan.hasNext();
scan.hasNextInt();
```
- 从控制台读取密码

```
// java se 6 提供了Console类实现读取密码
Console console = System.console();
Stirng userName = console.readLine("UserName:");
char [] psd = console.readPassword("Password:");
```

##### 字符文件输入输出
- 读取字符文件

```
Scanner scan = new Scanner(Path.get("pathstring"),"UTF-8"); // 路径是相对于Java虚拟机启动路径的相对位置
scan.readLine()....等一系列方法
```
- 写字符文件

```
PrintWriter out = new PrintWriter("filename.txt","UTF-8");
out.println("hhh");
```
- 对于处理文件的Scanner以及PrintWriter，需要在方法中处理异常

##### Java的块作用域
- **和C++不一样，Java不允许在嵌套块中重复定义一个变量，会编译错误**

```
{
  int n;
  {
    int n; // 这种写法在java中是编译不过的
  }
}
```

##### switch case break
- 虽然很简单，但周围很多人都会用错
- break的意思是，本case如果命中了，在执行结束之后，中断之后的case检查，直接返回。
- 多个case可以公用同一个处理函数
- case标签的类型可以是char，4种整型，枚举常量，以及字符串常量（Java7开始支持）

##### 数组

- 数组声明：int [] array;

- 数组初始化： array = new int [100];
  - 需要指定数组初始大小，可以是整型n变量，数组创建之后大小无法更改。（如果需要，使用数组列表）
  - 整型数组所有元素初始化为0，boolean初始化为false，对象类型初始化为null
- 数组元素个数：array.length
- 数组for循环：for(int item : array)
- 数组拷贝
  - 浅拷贝：直接将一个数据变量的值赋值给另一个数组变量，两个数组变量引用的是堆中的同一个数组，对一个的修改会影响到另一个
  - 深拷贝：在原来的基础上，新创建一个一模一样的数组，或者更长的数组。Array提供了copyOf方法
  ```java
  int [] newArray = Array.copyOf(array,array.length+10); // 拷贝array数组，长度加10
  ```
- Java中的数据是在堆上创建的，相当于C++中的`int * a = new int[30];`,而不是`int a[10];`,这是在栈中的数组。
- 多维数组：数组的元素还是数组而已，且Java还支持不规则的数组（行列不一定要求是整齐的），C++是不支持的

### 类与对象
- 类与对象的关系：模板，实例
- 面向对象编程的含义：封装
- 类之间的三种关系
- 对象与对象变量之间的关系
- 预定义类（String，Date，LocalDate）与自定义类
- Java多源文件的使用：Java编译器内置了类似UNIX的make功能，在编译的时候，能够自动查找需要依赖的其他类，有class文件就直接使用，没有则查找java文件并编译。
- Java中，所有的方法都必须包含在类中
- private私有域，只有属于同一个类的对象（本身，其他同类对象）才可以访问。
- final常量、final方法：
- 静态变量、静态常量、静态方法
- 静态方法中不能访问非静态实例域，但可以访问静态域。
- main()方法是一个静态方法，它不对任何对象进行操作，负责在程序启动的时候创建对象。
- 每个类都可以实现一个main方法，用来进行单元测试，在执行完整Application时候，每个单元中的main并不会执行。

##### 对象构造
- 构造函数：Java的对象都是在堆上构造的，通过new操作符在堆上创建新对象。构造函数在new对象的时候被调用。
- 与C++中对象的构造做区分，C++中支持在栈和堆上创建对象，Java对象与后者类似。

##### 对象域的初始化
- 三种域初始化方式：在构造器中初始化，声明中初始化，在初始化块
- 在用户没给构造函数的时候，系统会自动提供一个无参构造函数，一旦用户提供了，系统就不再提供无参构造函数。
- 对象中的域，默认被初始化为0，false，以及null
- 域的初始化也可以直接在声明时进行，好习惯是在声明变量域的时候，就给个安全的初始值。（C++中是不允许的，只能通过构造函数初始化域）
- 初始化块：一个或多个代码块，只要构造类对象，这些初始化块就一定会执行，一般用于初始化比较复杂的情况。

```java
class Employee{
  private int id;

  // 初始化块
  static
  {
    Random generator = new Random(); 
    id = generator.nextInt(1000); // 生成0-999的随机数
    System.out.println("hello"); //也可以执行非初始化语句的
  }
}
```
- 具体的域初始化步骤
 - 1.所有数据域被初始化为0，false、null
 - 2.按照类中各个数据域声明出现的前后顺序，执行各自的初始化语句以及初始化块。
 - 3.执行构造器内容


##### 对象析构 finalize方法
- Java有自动的垃圾回收器，所以不像C++，没有显式的析构器。
- 可以为类添加一个finalize方法，该方法在对象被清楚之前调用。
- **实际应用中，避免在finalize中去释放资源，因为finalize什么时候被执行是无法确定的**
- 对于一些资源的使用，要提供一个close方法，在使用完毕后调用。
- 可以通过Runtime.addShutdownHook 方法来添加关闭钩子的方式更好的实现。

##### 方法参数传递
- Java的参数传递是**传值调用**的，方法得到的是参数的拷贝，且**方法内无法修改传递进来的参数**。
- 对于基本数据类型，直接传入的是数据拷贝，而对于对象，传入的是对象引用的拷贝（**容易理解为传引用调用，但实际是通过传值调用实现的，只是这里的值是对象引用**）。
 - 对于基本数据类型参数，参数传递不会改变参数变量的值。
 - 对于对象引用参数，方法内可以通过对象应用去修改对象内容。
 - 对于对象引用参数，无法将对象引用参数指向另一个新对象。

##### 重载
- 重载：overloading，多个方法，有相同的名字，但有不同的参数列表。
- 重载解析：编译器负责根据参数列表，匹配正确的函数。
- 函数签名：函数名+参数列表，**注意，返回类型不属于函数签名，不能通过不同的返回类型来重载**

##### 包 package
- 通过包来组织类，文件目录方式，避免重名
- import导入类、静态导入
- 将类放入包中：
  - 1.类源代码开头添加 package path; （没有该行，则该类属于默认包）
  - 2.将类源文件放在package对应的问价夹中
- **编译器在编译文件的时候是不关注目录结构的，但定位一个类的时候，通过包名和类名来定位。所以，必须将类源代码放在package对应的文件夹下，否则虚拟机找不到类**
- 包作用域：不添加public和private时，默认的作用域。同一个包中都可以访问。

##### 类路径
- 从三个地方加载类
  - 1.JRE中的JAR文件
  - 2.第三方JAR文件
  - 3.用户源程序目录
- 通过`java -classpath `或者设置`CLASSPATH`环境变量来设置类路径。

##### 类注释
- javadoc，由源文件生成html
- 类注释

```java
/**
  类注释
*/
class hhh{}
```
- 方法注释

```java
/**
* desc hh
* @param id
* @return void
* @throws
*/
private void hh(int id){}
```
- 域注释

```java
/**
* desc
*/
int id;
```

### 总结
欲知后事如何，且听下回，太长了，逃。