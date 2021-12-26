# Core Java Volume I

## Chapter 2 Java环境编程环境

1. 下载对应的jdk

2. 设置PATH

   ```shell
   vi ~/.bash_profile
   #添加JAVA命令路径
   export PATH=jdk/bin:$PATH
   ```

3. 测试环境配置

   ```shell
   javac -version
   ```

4. 安装源文件与文档

   JDK lib中包含一个压缩文件**src.zip**,包含了java公有库的源代码

   ```shell
   mkdir javasrc
   cd javasrc
   jar xvf jdk/src.zip
   cd ..
   ```

   JDK文档与JDK是分开的，需要单独下载

   ```shell
   jar xvf jdk-version-docs-all.zip
   mv doc javadoc
   ```

   Ps: javac 是Java编译器，负责编译.java为.class文件，java命令启动java虚拟机，执行编译后的字节码(bytecode)

## Chapter 3 Java 程序结构基础

1. Java中所有的函数都是某个类(class)的方法(method)，因此main()函数必须要放于一个包装类中
2. Java中main()函数不返回值，默认退出码为0；可通过System.exit()设置不同的退出码以终止程序

### 注释

Java中有3类注释方法:

- // 单行注释
- /*  */ 块注释
- /** */ 文档注释(用于自动生成文档)

###数据类型

Java中有8种原始类型

**整型**

- byte  1byte
- short  2byte
- int  4byte
- long 8byte

**浮点型**

- float 4byte
- double 8byte

**字符类型**

- char 

**布尔类型**

- boolean

Note: 

- 在Java中，整型与boolean类型之间不能转换



### 变量

Tips: 

- java中应(每一行)分别声明变量，便于可读
- java中使用**未初始化的变量**将导致编译错误



**常量** 

- 使用**final**来声明常量(const)
- 使用 static final声明类常量(class const)

Tips: 常量一般用全大写字母表示



### 字符串

Java中没有内建的string类型，只有一个标准库预先定义的类**String**

**拼接**

可以使用+拼接String，当后者一个不是String时，将会被转换为String

Note: 

- 每一个java对象都可以被转换为String
- String对象是不可更改的(与Python中类似)

- Java中的String不能使用==进行相等比较；使用==比较只能判断两个String是否存储在相同的地址

**空字符串与NULL**

- 空字符串是一个java对象，其字符串长度为0，内容为空
- null表示没有一个对象与该变量关联

Tips: 判断一个String既不是null也不是空，使用如下方式

```java
// 必须先判断为null, 因在null上调用方法是错误的
if (str != null && str.length() != 0)
```

Note: 

​	java中code point(码点)与code uint(码元)的区别？？？

​	java默认使用utf-16编码？

**StringBuilder**

使用StringBuilder构造字符串，避免拼接造成String的不停构造

**StringBuffer**

用于多线程中添加或删除字符，效率较低



### input/output

1. 为从终端获取输入，需要构造一个Scanner，同时与System.in关联

Note:

- 当使用一个未在**java.lang**中定义的类时，需要使用**import**指令进行导入

2. File Input/Output

   - 读取文件时，构造一个Scanner对象
   - 写入文件时，构造一个PrintWriter对象

   Note: 默认读取或写入文件时，使用平台默认的编码方式

### Control Flow

1. java中不能在嵌套块(block)中声明同名变量(这与C/C++不同)

2. java中没有goto，但有带标签的break语句

3. java中的switch语句中，case标签可以为如下值:

   - char, byte,short, int的常量表达式
   - 枚举常量
   - 字符串字面量(string literal)

4. labeled break(类似于goto的功能)

5. java中声明数组的形式:

   ```java
   type[] arrayName;
   type arrayName[];
   
   // 第一种声明方式更为通用
   // 
   ```

   Note: java中只有一维数组，多维数组是数组的数组模拟而成

6. java中加强版for循环

   ```java
   // 类似C++中的range for
   for (variable : collection) statement
   ```

### Class and Object

1. Java中，所有的对象变量都引用(refer)一个存储在某处的对象，其并不实际包含对象

   Note: java中必须使用**clone()**方法来完全拷贝一个对象

2. 所有Java对象存储在**堆(heap)**上，并且一个构造函数必须结合new一起调用

**Final**

final用于一个原始类型或一个不可更改(immutable class)的类时，表示常量对象；final修饰一个可更改的(mutable)类时，该变量仅能引用该对象，不能再引用其他对象，但对象数据可被修改



**Static Fields and Methods**

- java中通过this(...)可以调用另一个构造函数

Java中初始化类**普通数据字段**有3种方式

```java
public class Sample
{
    // 1. 声明处初始化
    private String a = "xxx";
    private int b = 1;
    
    // 2. initialization block 初始化块
    {
        a = "yyy";
        b = 2;
    }
    
    // 3. 构造函数初始化
    public Sample()
    {
        a = "zzz";
        b = 3;
    }
}
```

Java中初始化类**静态变量**有2种方式

```java
public class Sample
{
    // 1. 声明处初始化
    private static int a = 1;
    
    // 2. static initialization block 静态初始化块
    static
    {
        a = 2;
    }
}
```

Note:

- 所有静态字段初始化器或静态初始化块以其出现在类声明中的顺序依次执行



**Package**

Java中以嵌套子目录的形式组织包

Note:

- 只能使用带*标记的import语句在单个包上

  ```java
  import java.util.*; // ok
  import java.*;  // error, java拥有多个package
  ```

- 静态导入

  如果只想导入静态方法和字段而不是类，可以使用静态导入

  ```java
  import static java.lang.System.*; // 导入所有static
  import static java.lang.System.out;	// 导入指定的static
  ```

- 包作用域

  对于类(class), public 表示可被任意类使用, private用于类声明

  如果没有指定public或private，类、方法、变量可以被相同包(package)中的所有方法(method)访问

Note: 可java中的package与import类似于C++中的namespace和using指令

**Class Path**

Java中有两种方式指定classpath

- 通过java命令的**-classpath**或**-cp**选项指定
- 通过设定环境变量**CLASSPATH**

Note: 当整个命令比较长时，一般将其写入shell脚本中

```shell
#!/usr/bin/evn bash
java -classpath PATH1:PATH2:.
export CLASSPATH=PATH1:PATH2:.
```

Note:

​	javac 编译参数说明:

​		-classpath 指定编译所需的.class文件的目录

​		-sourcepath 指定编译所需的.java文件所在目录，该目录会被当做package导入的根目录，将从该目录下按照package路径寻找到java文件

```shell
###### javac编译示例 #####
# 不指定-classpath或-sourcepath时，默认为当前目录
javac Test.java

# 通过-sourcepath指定.java文件查找根目录
javac -sourcepath . Test.java

# 通过-classpath指定.class文件查找根目录
javac -classpath . Test.java

# 同时指定-classpath, -sourcepath
javac -classpath ./build -sourcepath ./src Test.java

# 当通过-classpath -sourcepath分别都找到同一个类的.class与.java文件时
# 1. 如果.class与.java文件内容一致, 则遵循.class
# 2. 如果.class与.java文件内容不一致, 则遵循.java文件, 并编译.java源文件

##### JAVA运行示例 #####
# -classpath指定.class文件查找目录(package的根目录)
java -classpath . Test

# 当运行的main class属于某个package时
java -classpath ./build packageName.Test
```

NOTE:

**javac <options> <source files>**

- Javac 常用参数
  - -g 生成调试信息
  - -verbose 输出编译器执行操作的相关信息
  - -d <directory> 指定放置生成的.class文件的目录
  - -s <directory> 指定放置生成的源文件的目录
  - -nowarn 不产生警告信息
  - -Werror 产生警告时终止编译
  - --class-path <path>, -classpath <path>, -cp <path> 指定何处查找用户类文件以及注释处理程序
  - --source-path <path>, -sourcepath <path> 指定何处查找源文件
  - -encoding <encoding> 指定源文件的编码
  - -J<flag> 传递参数flag给java启动器
  - @<filename> 从文件读取选项和文件名

其他命令参考:

​	**java javap**



### Documentation Comments

java文档注释一般形式

```java
/**
 free-form text
 free-form text
 @xxxx
 @yyyy
 */
```

javadoc工具提取以下4类注释信息

1. Packages
2. Public  classes and interfaces
3. Public  and protected fields
4. Public  and protected constructors and methods

Note: private的私有信息不会在doc中展示



## 5 Inheritance(继承)

### Class, SuperClass, and SubClass

Java使用**extends**关键字来表示继承关系

```java
public class SubClass extends SuperClass
{
	// add methods and fields
}
```

Note:

- java中所有继承都是公有(public)继承
- java中使用super关键字调用父类的方法

Note: java不支持多个继承

**Polymorphism(多态)**

> 一个对象变量可以指示多种实际类型的现象被称为**多态(polymorphism)**
>
> 在运行时能够自动地选择调用哪个方法的现象被称为**动态绑定(dynamic binding)**

Note: 在Java中，不需要将方法声明为虚拟方法。动态绑定是默认的处理方式。如果不希望让一个方法具有虚拟特征，可以将它标记为final

**Abstract Class(抽象类)**

java使用关键字**abstract**声明一个抽象类；一个包含一个或多个抽象函数的类属于抽象类

Ps: 抽象类除了拥有抽象函数外，还可以拥有实例字段与具体函数，这一点与C++相同

Note: Java中访问标识符可见性总结

1. private: 仅仅对类(class)可见
2. public: 对所有可见
3. protected: 对包(package)以及所有子类可见(这点与C++有些微小差别)
4. 默认情况下(没有指定标识符时)，对包(package)可见

**5.2 Object 类**

Java中的每个类都继承至Object类

在Java中，只有原始类型(numbers, characters, boolean)的值不是对象；array数组也是继承至Object的类类型



**5.6 Enumeration class(枚举类)**

JAVA中的枚举类可以拥有构造函数、方法以及实例字段；其构造函数仅在枚举常量构建时被调用

Note: 所有的枚举类型均是Enum的子类



**6.3 lambda表达式**

java中的闭包(lambda expression)由3部分组成:

- 代码块
- 参数
- 自由变量

Note:

1. Java中的lambda表达式可以捕获外围作用域中的变量(与C++ lambda相同)，不同在于java的lambda表达式中, 只能引用值不会改变的变量
2. lambda表达式捕获的变量必须是最终变量(变量一旦被初始化，就不能再赋值)







