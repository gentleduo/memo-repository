# 简介

scala是运行在JVM上的多范式编程语言，同时支持面向对象和面向函数编程

## 优势

* 开发大数据应用程序（Spark程序、Flink程序）

- 表达能力强，一行代码抵得上Java多行，开发速度快
- 兼容Java，可以访问庞大的Java类库，例如：操作mysql、redis、freemarker、activemq等等

## scala对比Java

下面通过两个案例，分别使用java和scala实现的代码数量

### 案例一

定义三个实体类（用户、订单、商品）

Java代码

```java
/**
 * 用户实体类
 */
public class User {
    private String name;
    private List<Order> orders;

    public String getName() {
    	return name;
    }

    public void setName(String name) {
    	this.name = name;
    }

    public List<Order> getOrders() {
    	return orders;
    }

    public void setOrders(List<Order> orders) {
    	this.orders = orders;
    }
}
```

```java
/**
 * 订单实体类
 */
public class Order {
    private int id;
    private List<Product> products;

    public int getId() {
    	return id;
    }

    public void setId(int id) {
    	this.id = id;
    }

    public List<Product> getProducts() {
    	return products;
    }

    public void setProducts(List<Product> products) {
    	this.products = products;
    }
}
```

```java
/**
 * 商品实体类
 */
public class Product {
    private int id;
    private String category;

    public int getId() {
    	return id;
    }

    public void setId(int id) {
    	this.id = id;
    }

    public String getCategory() {
    	return category;
    }

    public void setCategory(String category) {
    	this.category = category;
    }
}
```

scala

```scala
case class User(var name:String, var orders:List[Order])	// 用户实体类
case class Order(var id:Int, var products:List[Product])	// 订单实体类
case class Product(var id:Int, var category:String)  		// 商品实体类
```

### 案例二

有一个字符串（数字）列表，需要将列表中所有的字符创转换为整数：

Java

```java
List<Integer> ints = new ArrayList<Integer>();
for (String s : list) {
    ints.add(Integer.parseInt(s));
}
```

scala

```scala
val ints = list.map(s => s.toInt)
```

# 开发环境安装

要编译运行scala程序，需要

* jdk（jvm）
* scala编译器（scala SDK）

接下来，需要依次安装以下内容：

- 安装JDK
- 安装scala SDK
- 安装IDEA插件

## 安装JDK

安装JDK 1.8 64位版本，并配置好环境变量

## 安装scala SDK

scala SDK是scala语言的编译器，要开发scala程序，必须要先安装SDK

https://www.scala-lang.org/download/all.html

## 安装IDEA scala插件

IDEA默认是不支持scala程序开发，所以需要来安装scala插件来支持scala语言。

1. 下载指定版本：https://plugins.jetbrains.com/plugin/1347-scala
2. IDEA配置scala插件
3. 重新启动IDEA

### 具体操作

1. 查看IDEA的版本号：Help ==> About
2. 到IDEA官网下载对应版本的，务必下载IDEA版本一致的scala插件
3. 安装插件：File ==> Settings ==> Plugins ==> 点击小齿轮 ==> Install Plugin from Disk

# 变量

## val和var

在scala中，可以使用val或者var来定义变量，语法格式如下:

```scala
val/var 变量标识:变量类型 = 初始值
```

其中

- val定义的是不可重新赋值的变量
- var定义的是可重新赋值的变量

## 类型推断

scala可以自动根据变量的值来自动推断变量的类型，这样编写代码更加简洁。

```scala
scala> val name = "tom"
name: String = tom
```

## 惰性赋值

当有一些变量保存的数据较大时，但是不需要马上加载到JVM内存。可以使用惰性赋值来提高效率

语法格式：

```scala
lazy val/var 变量名 = 表达式
```

# 字符串

scala提供多种定义字符串的方式，将来我们可以根据需要来选择最方便的定义方式。

## 使用双引号

```scala
val/var 变量名 = “字符串”
```

使用插值表达式

```scala
val/var 变量名 = “字符串”
```

参考代码：

```scala
scala> val name = "zhangsan"
name: String = zhangsan

scala> val age = 30
age: Int = 30

scala> val sex = "male"
sex: String = male

scala> val info = s"name=${name}, age=${age}, sex=${sex}"
info: String = name=zhangsan, age=30, sex=male

scala> println(info)
name=zhangsan, age=30, sex=male
```

使用三引号

如果有大段的文本需要保存，就可以使用三引号来定义字符串。例如：保存一大段的SQL语句。三个引号中间的所有字符串都将作为字符串的值。

```scala
scala> val sql = """select
     | *
     | from
     |     t_user
     | where
     |     name = "zhangsan""""
sql: String =
select
*
from
    t_user
where
    name = "zhangsan"
    
scala> println(sql)
select
*
from
    t_user
where
    name = "zhangsan"
```

# 数据类型与操作符

scala中的类型以及操作符绝大多数和Java一样，我们主要来学习

* 与Java不一样的一些用法
* scala类型的继承体系

## 数据类型

| 基础类型 | 类型说明                 |
| -------- | ------------------------ |
| Byte     | 8位带符号整数            |
| Short    | 16位带符号整数           |
| Int      | 32位带符号整数           |
| Long     | 64位带符号整数           |
| Char     | 16位无符号Unicode字符    |
| String   | Char类型的序列（字符串） |
| Float    | 32位单精度浮点数         |
| Double   | 64位双精度浮点数         |
| Boolean  | true或false              |

注意下 scala类型与Java的区别

1. scala中所有的类型都使用大写字母开头
2. 整形使用Int而不是Integer
3. scala中定义变量可以不写类型，让scala编译器自动推断

## 运算符

| 类别       | 操作符                     |
| ---------- | -------------------------- |
| 算术运算符 | +、-、*、/                 |
| 关系运算符 | >、<、==、!=、>=、<=       |
| 逻辑运算符 | &&、&#124;&#124;、!        |
| 位运算符   | &、&#124;&#124;、^、<<、>> |

注意下 scala类型与Java的区别

1. scala中没有，++、--运算符

2. 与Java不一样，在scala中，可以直接使用==、!=进行比较，它们与equals方法表示一致。而比较两个对象的引用值，使用eq

## 类型层次结构

![image][1]

| 类型    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| Any     | 所有类型的父类，,它有两个子类AnyRef与AnyVal                  |
| AnyVal  | 所有数值类型的父类                                           |
| AnyRef  | 所有对象类型（引用类型）的父类                               |
| Unit    | 表示空，Unit是AnyVal的子类，它的实例是()<br>它类似于Java中的void，但scala要比Java更加面向对象 |
| Null    | Null是AnyRef的子类，也就是说它是所有引用类型的子类。它的实例是null<br>可以将null赋值给任何对象类型 |
| Nothing | 所有类型的子类<br>不能直接创建该类型实例，某个方法抛出异常时，返回的就是Nothing类型，因为Nothing是所有类的子类，那么它可以赋值为任何类型 |

# 条件表达式

与Java不一样的是：

* 在scala中，条件表达式也是有返回值的
* 在scala中，没有三元表达式，可以使用if表达式替代三元表达式

定义一个变量sex，再定义一个result变量，如果sex等于"male"，result等于1，否则result等于0

```scala
scala> val sex = "male"
sex: String = male

scala> val result = if(sex == "male") 1 else 0
result: Int = 1
```

# 块表达式

* scala中，使用{}表示一个块表达式
* 和if表达式一样，块表达式也是有值的
* 值就是最后一个表达式的值

```scala
// 下面这个| 其实是回车
scala> val a = {
     | println("1 + 1")
     | 1 + 1
     | }
1 + 1
a: Int = 2

scala>
//在scala中，可以用回车和;来表示一句代码的结束
scala> val a = {println("1+1");1+1}
1+1
a: Int = 2

scala>
```

# 循环

## for

```scala
for(i <- 表达式/数组/集合) {
    // 表达式
}
```

使用for表达式打印1-10的数字

```scala
// 生成1-10的数字（使用to方法）；scala是完全面向对象的，1也是一个对象，所以可以调用to方法
scala> val nums = 1.to(10)                                                              
nums: scala.collection.immutable.Range.Inclusive = Range(1, 2, 3, 4, 5, 6, 7, 8, 9, 10) 
// 使用for表达式遍历，打印每个数字                 
scala> for(i <- nums) println(i) 
```

```scala
// 中缀调用法
scala> for(i <- 1 to 10) println(i)
```

嵌套循环

```scala
// 嵌套循环
for(i <- 1 to 3; j <- 1 to 5) {print("*");if(j == 5) println("")}
```

守卫：for表达式中，可以添加if判断语句，这个if判断就称之为守卫。我们可以使用守卫让for表达式更简洁。

```scala
for(i <- 表达式/数组/集合 if 表达式) {
    // 表达式
}
```

```scala
// 添加守卫，打印能够整除3的数字
for(i <- 1 to 10 if i % 3 == 0) println(i)
```

for推导式

* 将来可以使用for推导式生成一个新的集合（一组数据）

* 在for循环体中，可以使用yield表达式构建出一个集合，我们把使用yield的for表达式称之为推导式

* for循环中的yield会把当前的元素记下来，保存在集合中，循环结束后将返回该集合。Scala中for循环是有返回值的。如果被循环的是Map，返回的就是Map，被循环的是List，返回的就是List，以此类推。

生成一个10、20、30...100的集合

```scala
// for推导式：for表达式中以yield开始，该for表达式会构建出一个集合
val v = for(i <- 1 to 10) yield i * 10
```

## while循环

scala中while循环和Java中是一致的

打印1-10的数字

```scala
scala> var i = 1
i: Int = 1

scala> while(i <= 10) {
     | println(i)
     | i = i+1
     | }
```

## break和continue

* 在scala中，类似Java和C++的break/continue关键字被移除了
* 如果一定要使用break/continue，就需要使用scala.util.control包的Break类的breable和break方法。

### 实现break

* 导入Breaks包import scala.util.control.Breaks._
* 使用breakable将for表达式包起来
* for表达式中需要退出循环的地方，添加break()方法调用

使用for表达式打印1-100的数字，如果数字到达50，退出for表达式

```scala
// 导入scala.util.control包下的Break
import scala.util.control.Breaks._

breakable{
    for(i <- 1 to 100) {
        if(i >= 50) break()
        else println(i)
    }
}
```

### 实现continue

continue的实现与break类似，但有一点不同：实现break是用breakable{}将整个for表达式包起来，而实现continue是用breakable{}将for表达式的循环体包含起来就可以了

打印1-100的数字，使用for表达式来遍历，如果数字能整除10，不打印

```scala
// 导入scala.util.control包下的Break    
import scala.util.control.Breaks._

for(i <- 1 to 100 ) {
    breakable{
        if(i % 10 == 0) break()
        else println(i)
    }
}
```

# 方法

## 语法

```scala
def methodName (参数名:参数类型, 参数名:参数类型) : [return type] = {
    // 方法体：一系列的代码
}
```

* 参数列表的参数类型不能省略
* 返回值类型可以省略，由scala编译器自动推断
* 返回值可以不写return，默认就是{}块表达式的值

定义一个方法，实现两个整形数值相加，返回相加后的结果

```scala
scala> def add(a:Int, b:Int):Int = a + b
add: (a: Int, b: Int)Int
// scala定义方法可以省略返回值，由scala自动推断返回值类型。这样方法定义后更加简洁。
scala> def add(a:Int, b:Int) = a + b
m1: (x: Int, y: Int)Int
```

## 参数

### 默认参数

```scala
// x，y带有默认值为1
scala> def add(a:Int=1,b:Int=1) = a + b
add: (a: Int, b: Int)Int

scala> add()
res5: Int = 2
```

### 带名参数

```scala
scala> def add(x:Int = 0, y:Int = 0) = x + y
add: (x: Int, y: Int)Int

scala> add(x=1)
res6: Int = 1
```

### 变长参数

在参数类型后面加一个*号，表示参数可以是0个或者多个

```scala
scala> def add(num:Int*) = num.sum
add: (num: Int*)Int

scala> add(1,2,3,4,5)
res1: Int = 15
```

## 调用方式

### 后缀调用法

```scala
scala> Math.abs(-1)
res3: Int = 1
```

### 中缀调用法

```scala
scala> Math abs -1
res4: Int = 1
```

### 操作符即方法

在scala中，+ - * / %等这些操作符和Java一样，但在scala中：

- 所有的操作符都是方法
- 操作符是一个方法名字是符号的方法

```scala
1 + 1
```

### 花括号调用法

方法只有一个参数，才能使用花括号调用法

```scala
scala> Math.abs{println("求绝对值：");-10}
求绝对值：
res8: Int = 10
```

### 无括号调用法

如果方法没有参数，可以省略方法名后面的括号

```scala
scala> def m3=println("hello")
m3: Unit

scala> m3
hello
```

# 函数

## 定义

```scala
val 函数变量名 = (参数名:参数类型, 参数名:参数类型....) => 函数体
```

* 函数是一个对象（变量）
* 类似于方法，函数也有输入参数和返回值
* 函数定义不需要使用`def`定义
* 无需指定返回值类型

```scala
scala> val add = (x:Int, y:Int) => x + y
add: (Int, Int) => Int = <function2>

scala> add(1,2)
res9: Int = 3
```

## 方法和函数的区别

- 方法是隶属于类或者对象的，在运行时，它是加载到JVM的方法区中
- 可以将函数对象赋值给一个变量，方法无法赋值给变量，在运行时，它是加载到JVM的堆内存中
- 函数是一个对象，继承自FunctionN，函数对象有apply，curried，toString，tupled这些方法。方法则没有

## 方法转换为函数

* 有时候需要将方法转换为函数，作为变量传递，就需要将方法转换为函数

* 使用_即可将方法转换为函数

```scala
//定义一个方法用来进行两个数相加
scala> def add(x:Int,y:Int)=x+y
add: (x: Int, y: Int)Int

// 将该方法转换为一个函数，赋值给变量
scala> val a = add _
a: (Int, Int) => Int = <function2>
```

# 数组

scala中数组的概念是和Java类似，可以用数组来存放一组数据。scala中，有两种数组，一种是定长数组，另一种是变长数组

## 定长数组

* 定长数组指的是数组的长度是不允许改变的
* 数组的元素是可以改变的

```scala
// 通过指定长度定义数组
val/var 变量名 = new Array[元素类型](数组长度)

// 用元素直接初始化数组
val/var 变量名 = Array(元素1, 元素2, 元素3...)
```

* 在scala中，数组的泛型使用[]来指定

* 使用()来获取元素

```scala
// 通过指定长度定义数组
scala> val a = new Array[Int](100)
a: Array[Int] = Array(0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0)

scala> a(0) = 110

scala> println(a(0))
110
```

```scala
// 用元素直接初始化数组
// 定义包含jave、scala、python三个元素的数组
scala> val a = Array("java", "scala", "python")
a: Array[String] = Array(java, scala, python)

scala> a.length
res17: Int = 3
```

## 变长数组

变长数组指的是数组的长度是可变的，可以往数组中添加、删除元素；创建变长数组，需要提前导入ArrayBuffer类import scala.collection.mutable.ArrayBuffer

创建空的ArrayBuffer变长数组，语法结构：

```scala
val/var a = ArrayBuffer[元素类型]()
```

创建带有初始元素的ArrayBuffer

```scala
val/var a = ArrayBuffer(元素1，元素2，元素3....)
```

示例

```scala
// 定义一个长度为0的整型变长数组
val a = ArrayBuffer[Int]()
// 定义一个包含以下元素的变长数组
// "hadoop", "storm", "spark"
scala> val a = ArrayBuffer("hadoop", "storm", "spark")
a: scala.collection.mutable.ArrayBuffer[String] = ArrayBuffer(hadoop, storm, spark)
```

### 添加/修改/删除元素

- 使用+=添加元素
- 使用-=删除元素
- 使用++=追加一个数组到变长数组

```scala
// 定义变长数组
scala> val a = ArrayBuffer("hadoop", "spark", "flink")
a: scala.collection.mutable.ArrayBuffer[String] = ArrayBuffer(hadoop, spark, flink)

// 追加一个元素
scala> a += "flume"
res10: a.type = ArrayBuffer(hadoop, spark, flink, flume)

// 删除一个元素
scala> a -= "hadoop"
res11: a.type = ArrayBuffer(spark, flink, flume)

// 追加一个数组
scala> a ++= Array("hive", "sqoop")
res12: a.type = ArrayBuffer(spark, flink, flume, hive, sqoop)
```

## 遍历数组

可以使用以下两种方式来遍历数组：

- 使用for表达式直接遍历数组中的元素

  ```scala
  scala> val a = Array(1,2,3,4,5)
  a: Array[Int] = Array(1, 2, 3, 4, 5)
  
  scala> for(i<-a) println(i)
  1
  2
  3
  4
  5
  ```

- 使用索引遍历数组中的元素

  ```scala
  scala> val a = Array(1,2,3,4,5)
  a: Array[Int] = Array(1, 2, 3, 4, 5)
  
  scala> for(i <- 0 to a.length - 1) println(a(i))
  1
  2
  3
  4
  5
  
  scala> for(i <- 0 until a.length) println(a(i))
  1
  2
  3
  4
  5
  ```

  0 until n——生成一系列的数字，包含0，不包含n

  0 to n ——包含0，也包含n

## 数组常用算法

scala中的数组封装了一些常用的计算操作，将来在对数据处理的时候，不需要自己再重新实现。以下为常用的几个算法：

* 求和——sum方法

  ```scala
  scala> val a = Array(1,2,3,4)
  a: Array[Int] = Array(1, 2, 3, 4)
  
  scala> a.sum
  res49: Int = 10
  ```

- 求最大值——max方法

  ```scala
  scala> val a = Array(4,1,2,4,10)
  a: Array[Int] = Array(4, 1, 2, 4, 10)
  
  scala> a.max
  res50: Int = 10
  ```

- 求最小值——min方法

  ```scala
  scala> val a = Array(4,1,2,4,10)
  a: Array[Int] = Array(4, 1, 2, 4, 10)
  
  scala> a.min
  res51: Int = 1
  ```

- 排序——sorted方法

  ```scala
  // 升序排序
  scala> a.sorted
  res53: Array[Int] = Array(1, 2, 4, 4, 10)
  
  // 降序
  scala> a.sorted.reverse
  res56: Array[Int] = Array(10, 4, 4, 2, 1)
  ```


# 元组

元组可以用来包含一组不同类型的值。例如：姓名，年龄，性别，出生年月。元组的元素是不可变的。

使用括号来定义元组

```scala
val/var 元组 = (元素1, 元素2, 元素3....)
```

使用箭头来定义元组（元组只有两个元素）

```scala
val/var 元组 = 元素1->元素2
```

示例：

* 定义一个元组，包含学生的姓名和年龄（zhangsan、20）
* 分别使用括号、和箭头来定义元组

```scala
scala> val a = ("zhangsan", 20)
a: (String, Int) = (zhangsan,20)

scala> val a = "zhangsan" -> 20
a: (String, Int) = (zhangsan,20)
```

访问元组：

使用\_1、\_2、\_3....来访问元组中的元素，_1表示访问第一个元素，依次类推

```scala
scala> val a = "zhangsan" -> "male"
a: (String, String) = (zhangsan,male)

// 获取第一个元素
scala> a._1
res41: String = zhangsan

// 获取第二个元素
scala> a._2
res42: String = male
```

# 列表

列表是scala中最重要的、也是最常用的数据结构。List具备以下性质：

- 可以保存重复的值
- 有先后顺序

在scala中，也有两种列表，一种是不可变列表、另一种是可变列表

## 不可变列表

不可变列表就是列表的元素、长度都是不可变的。

使用List(元素1, 元素2, 元素3, ...)来创建一个不可变列表，语法格式：

```scala
val/var 变量名 = List(元素1, 元素2, 元素3...)
```

```scala
scala> val a = List(1,2,3,4)
a: List[Int] = List(1, 2, 3, 4)
```

使用Nil创建一个不可变的空列表

```scala
val/var 变量名 = Nil
```

```scala
scala> val a = Nil
a: scala.collection.immutable.Nil.type = List()
```

使用::方法创建一个不可变列表

```scala
val/var 变量名 = 元素1 :: 元素2 :: Nil
```

```scala
scala> val a = -2 :: -1 :: Nil
a: List[Int] = List(-2, -1)
```

使用**::**拼接方式来创建列表，必须在最后添加一个Nil

## 可变列表

可变列表就是列表的元素、长度都是可变的。

要使用可变列表，先要导入import scala.collection.mutable.ListBuffer

- 可变集合都在mutable包中
- 不可变集合都在immutable包中（默认导入）

使用ListBuffer\[元素类型\]()创建空的可变列表，语法结构：

```scala
val/var 变量名 = ListBuffer[Int]()
```

```scala
  scala> val a = ListBuffer[Int]()
  a: scala.collection.mutable.ListBuffer[Int] = ListBuffer()
```

使用ListBuffer(元素1, 元素2, 元素3...)创建可变列表，语法结构：

```scala
val/var 变量名 = ListBuffer(元素1，元素2，元素3...)
```

```scala
scala> val a = ListBuffer(1,2,3,4)
a: scala.collection.mutable.ListBuffer[Int] = ListBuffer(1, 2, 3, 4)
```

### 可变列表操作

#### 基本操作

```scala
// 导入不可变列表
scala> import scala.collection.mutable.ListBuffer
import scala.collection.mutable.ListBuffer

// 创建不可变列表
scala> val a = ListBuffer(1,2,3)
a: scala.collection.mutable.ListBuffer[Int] = ListBuffer(1, 2, 3)

// 获取第一个元素
scala> a(0)
res19: Int = 1

// 追加一个元素
scala> a += 4
res20: a.type = ListBuffer(1, 2, 3, 4)

// 追加一个列表
scala> a ++= List(5,6,7)
res21: a.type = ListBuffer(1, 2, 3, 4, 5, 6, 7)

// 删除元素
scala> a -= 7
res22: a.type = ListBuffer(1, 2, 3, 4, 5, 6)

// 转换为不可变列表
scala> a.toList
res23: List[Int] = List(1, 2, 3, 4, 5, 6)

// 转换为数组
scala> a.toArray
res24: Array[Int] = Array(1, 2, 3, 4, 5, 6)
```

#### 判断列表是否为空

```scala
scala> val a = List(1,2,3,4)
a: List[Int] = List(1, 2, 3, 4)

scala> a.isEmpty
res51: Boolean = false
```

#### 拼接两个列表

```scala
scala> val a = List(1,2,3)
a: List[Int] = List(1, 2, 3)

scala> val b = List(4,5,6)
b: List[Int] = List(4, 5, 6)

scala> a ++ b
res52: List[Int] = List(1, 2, 3, 4, 5, 6)
```

#### 获取列表的首个元素和剩余部分

```scala
scala> val a = List(1,2,3)
a: List[Int] = List(1, 2, 3)

// head方法，获取列表的首个元素
scala> a.head
res4: Int = 1

// tail方法，获取除第一个元素以外的元素，它也是一个列表
scala> a.tail
res5: List[Int] = List(2, 3)
```

#### 反转列表

```scala
scala> val a = List(1,2,3)
a: List[Int] = List(1, 2, 3)

scala> a.reverse
res6: List[Int] = List(3, 2, 1)
```

#### 获取列表前缀和后缀

```scala
scala> val a = List(1,2,3,4,5)
a: List[Int] = List(1, 2, 3, 4, 5)

// 使用take方法获取前缀（前三个元素）
scala> a.take(3)
res56: List[Int] = List(1, 2, 3)

// 使用drop方法获取后缀（除前三个以外的元素）
scala> a.drop(3)
res60: List[Int] = List(4, 5)
```

#### 扁平化

扁平化(压平)表示将列表中的列表中的所有元素放到一个列表中。

* 有一个列表，列表中又包含三个列表，分别为：List(1,2)、List(3)、List(4,5)
* 使用flatten将这个列表转换为List(1,2,3,4,5)

```scala
scala> val a = List(List(1,2), List(3), List(4,5))
a: List[List[Int]] = List(List(1, 2), List(3), List(4, 5))

scala> a.flatten
res0: List[Int] = List(1, 2, 3, 4, 5)
```

#### 拉链与拉开

* 拉链：使用zip将两个列表，组合成一个元素为元组的列表
* 拉开：将一个包含元组的列表，解开成包含两个列表的元组

示例：

* 有两个列表
  * 第一个列表保存三个学生的姓名，分别为：zhangsan、lisi、wangwu
  * 第二个列表保存三个学生的年龄，分别为：19, 20, 21
* 使用zip操作将两个列表的数据"拉"在一起，形成 zhangsan->19, lisi ->20, wangwu->21

```scala
scala> val a = List("zhangsan", "lisi", "wangwu")
a: List[String] = List(zhangsan, lisi, wangwu)

scala> val b = List(19, 20, 21)
b: List[Int] = List(19, 20, 21)

scala> a.zip(b)
res1: List[(String, Int)] = List((zhangsan,19), (lisi,20), (wangwu,21))

scala> res1.unzip
res2: (List[String], List[Int]) = (List(zhangsan, lisi, wangwu),List(19, 20, 21))
```

#### 转换字符串

toString方法可以返回List中的所有元素

```scala
scala> val a = List(1,2,3,4)
a: List[Int] = List(1, 2, 3, 4)

scala> println(a.toString)
List(1, 2, 3, 4)
```

#### 生成字符串

mkString方法，可以将元素以分隔符拼接起来。默认没有分隔符

```scala
scala> val a = List(1,2,3,4)
a: List[Int] = List(1, 2, 3, 4)

scala> a.mkString
res7: String = 1234

scala> a.mkString(":")
res8: String = 1:2:3:4
```

#### 并集

union表示对两个列表取并集，不去重

示例：

* 定义第一个列表，包含以下元素：1,2,3,4
* 定义第二个列表，包含以下元素：3,4,5,6
* 使用union操作，获取这两个列表的并集
* 使用distinct操作，去除重复的元素

```scala
scala> val a1 = List(1,2,3,4)
a1: List[Int] = List(1, 2, 3, 4)

scala> val a2 = List(3,4,5,6)
a2: List[Int] = List(3, 4, 5, 6)

// 并集操作
scala> a1.union(a2)
res17: List[Int] = List(1, 2, 3, 4, 3, 4, 5, 6)

// 可以调用distinct去重
scala> a1.union(a2).distinct
res18: List[Int] = List(1, 2, 3, 4, 5, 6)
```

#### 交集

intersect表示对两个列表取交集

示例：

- 定义第一个列表，包含以下元素：1,2,3,4
- 定义第二个列表，包含以下元素：3,4,5,6
- 使用intersect操作，获取这两个列表的交集

#### 差集

diff表示对两个列表取差集，例如： a1.diff(a2)，表示获取a1在a2中不存在的元素

- 定义第一个列表，包含以下元素：1,2,3,4
- 定义第二个列表，包含以下元素：3,4,5,6
- 使用diff获取这两个列表的差集

```scala
scala> val a1 = List(1,2,3,4)
a1: List[Int] = List(1, 2, 3, 4)

scala> val a2 = List(3,4,5,6)
a2: List[Int] = List(3, 4, 5, 6)

scala> a1.diff(a2)
res24: List[Int] = List(1, 2)
```

# Set

Set(集)是代表没有重复元素的集合。Set具备以下性质：

1. 元素不重复
2. 不保证插入顺序

scala中的集也分为两种，一种是不可变集，另一种是可变集。

## 不可变集

创建一个空的不可变集，语法格式：

```scala
val/var 变量名 = Set[类型]()
```

```scala
scala> val a = Set[Int]()
a: scala.collection.immutable.Set[Int] = Set()
```

给定元素来创建一个不可变集，语法格式：

```scala
val/var 变量名 = Set(元素1, 元素2, 元素3...)
```

```scala
scala> val a = Set(1,1,3,2,4,8)
a: scala.collection.immutable.Set[Int] = Set(1, 2, 3, 8, 4)
```

### 基本操作

```scala
// 创建集
scala> val a = Set(1,1,2,3,4,5)
a: scala.collection.immutable.Set[Int] = Set(5, 1, 2, 3, 4)

// 获取集的大小
scala> a.size
res0: Int = 5

// 遍历集
scala> for(i <- a) println(i)

// 删除一个元素
scala> a - 1
res5: scala.collection.immutable.Set[Int] = Set(5, 2, 3, 4)

// 拼接两个集
scala> a ++ Set(6,7,8)
res2: scala.collection.immutable.Set[Int] = Set(5, 1, 6, 2, 7, 3, 8, 4)

// 拼接集和列表
scala> a ++ List(6,7,8,9)
res6: scala.collection.immutable.Set[Int] = Set(5, 1, 6, 9, 2, 7, 3, 8, 4)
```

## 可变集

可变集合不可变集的创建方式一致，只不过需要提前导入一个可变集类。

手动导入：import scala.collection.mutable.Set

```scala
scala> val a = Set(1,2,3,4)
a: scala.collection.mutable.Set[Int] = Set(1, 2, 3, 4)                          

// 添加元素
scala> a += 5
res25: a.type = Set(1, 5, 2, 3, 4)

// 删除元素
scala> a -= 1
res26: a.type = Set(5, 2, 3, 4)
```

# Map

Map可以称之为映射。它是由键值对组成的集合。在scala中，Map也分为不可变Map和可变Map。

## 不可变Map

```scala
val/var map = Map(键->值, 键->值, 键->值...)	// 推荐，可读性更好
val/var map = Map((键, 值), (键, 值), (键, 值), (键, 值)...)
```

```scala
scala> val map = Map("zhangsan"->30, "lisi"->40)
map: scala.collection.immutable.Map[String,Int] = Map(zhangsan -> 30, lisi -> 40)

scala> val map = Map(("zhangsan", 30), ("lisi", 30))
map: scala.collection.immutable.Map[String,Int] = Map(zhangsan -> 30, lisi -> 30)

// 根据key获取value
scala> map("zhangsan")
res10: Int = 30
```

## 可变Map

定义语法与不可变Map一致。但定义可变Map需要手动导入import scala.collection.mutable.Map

```scala
scala> val map = Map("zhangsan"->30, "lisi"->40)
map: scala.collection.mutable.Map[String,Int] = Map(lisi -> 40, zhangsan -> 30)

// 修改value
scala> map("zhangsan") = 20
```

## 基本操作

```scala
scala> val map = Map("zhangsan"->30, "lisi"->40)
map: scala.collection.mutable.Map[String,Int] = Map(lisi -> 40, zhangsan -> 30)

// 获取zhagnsan的年龄
scala> map("zhangsan")
res10: Int = 30

// 获取所有的学生姓名
scala> map.keys
res13: Iterable[String] = Set(lisi, zhangsan)

// 获取所有的学生年龄
scala> map.values
res14: Iterable[Int] = HashMap(40, 30)

// 打印所有的学生姓名和年龄
scala> for((x,y) <- map) println(s"$x $y")
lisi 40
zhangsan 30

// 获取wangwu的年龄，如果wangwu不存在，则返回-1
scala> map.getOrElse("wangwu", -1)
res17: Int = -1

// 新增一个学生：wangwu, 35
scala> map + "wangwu"->35
res22: scala.collection.mutable.Map[String,Int] = Map(lisi -> 40, zhangsan -> 30, wangwu -> 35)

// 将lisi从可变映射中移除
scala> map - "lisi"
res23: scala.collection.mutable.Map[String,Int] = Map(zhangsan -> 30)
```

# iterator

scala针对每一类集合都提供了一个迭代器（iterator）用来迭代访问集合：

- 使用iterator方法可以从集合获取一个迭代器
- 迭代器的两个基本操作
  - hasNext——查询容器中是否有下一个元素
  - next——返回迭代器的下一个元素，如果没有，抛出NoSuchElementException
- 每一个迭代器都是有状态的
  - 迭代完后保留在最后一个元素的位置
  - 再次使用则抛出NoSuchElementException
- 可以使用while或者for来逐个返回元素

```scala
scala> val ite = a.iterator
ite: Iterator[Int] = non-empty iterator

scala> while(ite.hasNext) {
     | println(ite.next)
     | }
```

```scala
scala> val a = List(1,2,3,4,5)
a: List[Int] = List(1, 2, 3, 4, 5)

scala> val ite = a.iterator
ite: Iterator[Int] = non-empty iterator

scala> for (i <- ite) println(i)
1
2
3
4
5
```

# 函数式编程

## foreach

之前，学习过了使用for表达式来遍历集合。接下来将学习scala的函数式编程，使用foreach方法来进行遍历、迭代。它可以让代码更加简洁。

方法签名

```scala
foreach(f: (A) ⇒ Unit): Unit
```

说明

| foreach | API           | 说明                                                         |
| ------- | ------------- | ------------------------------------------------------------ |
| 参数    | f: (A) ⇒ Unit | 接收一个函数对象<br />函数的输入参数为集合的元素，返回值为空 |
| 返回值  | Unit          | 空                                                           |

```scala
// 定义一个列表
scala> val a = List(1,2,3,4)
a: List[Int] = List(1, 2, 3, 4)

// 迭代打印
scala> a.foreach((x:Int)=>println(x))
```

### 使用类型推断简化函数定义

上述案例有更简洁的写法。因为使用foreach去迭代列表，而列表中的每个元素类型是确定的

* scala可以自动来推断出来集合中每个元素参数的类型
* 创建函数时，可以省略其参数列表的类型

```scala
scala> val a = List(1,2,3,4)
a: List[Int] = List(1, 2, 3, 4)

// 省略参数类型
scala> a.foreach(x=>println(x))
```

### 使用下划线来简化函数定义

当函数参数，只在函数体中出现一次，而且函数体没有嵌套调用时，可以使用下划线来简化函数定义

```scala
scala> val a = List(1,2,3,4)
a: List[Int] = List(1, 2, 3, 4)

a.foreach(println(_))
```

* 如果方法参数是函数，如果出现了下划线，scala编译器会自动将代码封装到一个函数中
* 参数列表也是由scala编译器自动处理

## map

集合的映射操作是将来在编写Spark/Flink用得最多的操作，是我们必须要掌握的。因为进行数据计算的时候，就是一个将一种数据类型转换为另外一种数据类型的过程。

方法签名

```scala
def map[B](f: (A) ⇒ B): TraversableOnce[B]
```

说明

| map方法 | API                | 说明                                                         |
| ------- | ------------------ | ------------------------------------------------------------ |
| 泛型    | [B]                | 指定map方法最终返回的集合泛型                                |
| 参数    | f: (A) ⇒ B         | 传入一个函数对象<br />该函数接收一个类型A（要转换的列表元素），返回值为类型B |
| 返回值  | TraversableOnce[B] | B类型的集合                                                  |

### 案例一

1. 创建一个列表，包含元素1,2,3,4

2. 对List中的每一个元素加1

```scala
scala> a.map(x=>x+1)
res4: List[Int] = List(2, 3, 4, 5)
```

### 案例二

1. 创建一个列表，包含元素1,2,3,4

2. 使用下划线来定义函数，对List中的每一个元素加1

```scala
scala> val a = List(1,2,3,4)
a: List[Int] = List(1, 2, 3, 4)

scala> a.map(_ + 1)
```

## flatMap

扁平化映射也是将来用得非常多的操作，也是必须要掌握的。可以把flatMap，理解为先map，然后再flatten。

- map是将列表中的元素转换为一个List
- flatten再将整个列表进行扁平化

方法签名

```scala
def flatMap[B](f: (A) ⇒ GenTraversableOnce[B]): TraversableOnce[B]
```

方法解析

| flatmap方法 | API                            | 说明                                                         |
| ----------- | ------------------------------ | ------------------------------------------------------------ |
| 泛型        | [B]                            | 最终要转换的集合元素类型                                     |
| 参数        | f: (A) ⇒ GenTraversableOnce[B] | 传入一个函数对象<br />函数的参数是集合的元素<br />函数的返回值是一个集合 |
| 返回值      | TraversableOnce[B]             | B类型的集合                                                  |

### 案例

1. 有一个包含了若干个文本行的列表："hadoop hive spark flink flume", "kudu hbase sqoop storm"
2. 获取到文本行中的每一个单词，并将每一个单词都放到列表中

```scala
// 定义文本行列表
scala> val a = List("hadoop hive spark flink flume", "kudu hbase sqoop storm")
a: List[String] = List(hadoop hive spark flink flume, kudu hbase sqoop storm)

// 使用map将文本行转换为单词数组
scala> a.map(x=>x.split(" "))
res5: List[Array[String]] = List(Array(hadoop, hive, spark, flink, flume), Array(kudu, hbase, sqoop, storm))

// 扁平化，将数组中的
scala> a.map(x=>x.split(" ")).flatten
res6: List[String] = List(hadoop, hive, spark, flink, flume, kudu, hbase, sqoop, storm)
```

使用flatMap简化操作

```scala
scala>  val a = List("hadoop hive spark flink flume", "kudu hbase sqoop storm")
a: List[String] = List(hadoop hive spark flink flume, kudu hbase sqoop storm)

scala> a.flatMap(_.split(" "))
res7: List[String] = List(hadoop, hive, spark, flink, flume, kudu, hbase, sqoop, storm)
```

## filter

过滤符合一定条件的元素

方法签名

```scala
def filter(p: (A) ⇒ Boolean): TraversableOnce[A]
```

方法解析

| filter方法 | API                | 说明                                                         |
| ---------- | ------------------ | ------------------------------------------------------------ |
| 参数       | p: (A) ⇒ Boolean   | 传入一个函数对象<br />接收一个集合类型的参数<br />返回布尔类型，满足条件返回true, 不满足返回false |
| 返回值     | TraversableOnce[A] | 列表                                                         |

案例

1. 有一个数字列表，元素为：1,2,3,4,5,6,7,8,9

2. 请过滤出所有的偶数

```scala
scala> List(1,2,3,4,5,6,7,8,9).filter(_ % 2 == 0)
res8: List[Int] = List(2, 4, 6, 8)
```

## sort

### sorted

默认排序

```scala
scala> List(3,1,2,9,7).sorted
res16: List[Int] = List(1, 2, 3, 7, 9)
```

### sortBy

指定字段排序

方法签名

```scala
def sortBy[B](f: (A) ⇒ B): List[A]
```

方法解析

| sortBy方法 | API        | 说明                                                         |
| ---------- | ---------- | ------------------------------------------------------------ |
| 泛型       | [B]        | 按照什么类型来进行排序                                       |
| 参数       | f: (A) ⇒ B | 传入函数对象<br />接收一个集合类型的元素参数<br />返回B类型的元素进行排序 |
| 返回值     | List[A]    | 返回排序后的列表                                             |

示例

1. 有一个列表，分别包含几下文本行："01 hadoop", "02 flume", "03 hive", "04 spark"
2. 请按照单词字母进行排序

```scala
scala> val a = List("01 hadoop", "02 flume", "03 hive", "04 spark")
a: List[String] = List(01 hadoop, 02 flume, 03 hive, 04 spark)

// 获取单词字段
scala> a.sortBy(_.split(" ")(1))
res8: List[String] = List(02 flume, 01 hadoop, 03 hive, 04 spark)
```

### sortWith

自定义排序，根据一个函数来进行自定义排序

方法签名

```scala
def sortWith(lt: (A, A) ⇒ Boolean): List[A]
```

方法解析

| sortWith方法 | API                  | 说明                                                         |
| ------------ | -------------------- | ------------------------------------------------------------ |
| 参数         | lt: (A, A) ⇒ Boolean | 传入一个比较大小的函数对象<br />接收两个集合类型的元素参数<br />返回两个元素大小，小于返回true，大于返回false，升序<br />返回两个元素大小，小于返回false，大于返回true，降序 |
| 返回值       | List[A]              | 返回排序后的列表                                             |

示例

1. 有一个列表，包含以下元素：2,3,1,6,4,5
2. 使用sortWith对列表进行降序排序

```scala
scala> val a = List(2,3,1,6,4,5)
a: List[Int] = List(2, 3, 1, 6, 4, 5)

scala> a.sortWith((x,y) => if(x<y)false else true)
res20: List[Int] = List(6, 5, 4, 3, 2, 1)
```

使用下划线简写上述案例

```scala
scala> val a = List(2,3,1,6,4,5)
a: List[Int] = List(2, 3, 1, 6, 4, 5)

// 函数参数只在函数体中出现一次，可以使用下划线代替
scala> a.sortWith(_ > _)
res21: List[Int] = List(6, 5, 4, 3, 2, 1)
```

## groupBy

将数据按照分组来进行统计分析

方法签名

```scala
def groupBy[K](f: (A) ⇒ K): Map[K, List[A]]
```

方法解析

| groupBy方法 | API             | 说明                                                         |
| ----------- | --------------- | ------------------------------------------------------------ |
| 泛型        | [K]             | 分组字段的类型                                               |
| 参数        | f: (A) ⇒ K      | 传入一个函数对象<br />接收集合元素类型的参数<br />返回一个K类型的key，这个key会用来进行分组，相同的key放在一组中 |
| 返回值      | Map[K, List[A]] | 返回一个映射，K为分组字段，List为这个分组字段对应的一组数据  |

示例

1. 有一个列表，包含了学生的姓名和性别: 

   ```scala
   "张三", "男"
   "李四", "女"
   "王五", "男"
   ```

2. 请按照性别进行分组，统计不同性别的学生人数

步骤

1. 定义一个元组列表来保存学生姓名和性别
2. 按照性别进行分组
3. 将分组后的Map转换为列表：List(("男" -> 2), ("女" -> 1))

```scala
scala> val a = List("张三"->"男", "李四"->"女", "王五"->"男")
a: List[(String, String)] = List((张三,男), (李四,女), (王五,男))

// 按照性别分组
scala> a.groupBy(_._2)
res0: scala.collection.immutable.Map[String,List[(String, String)]] = Map(男 -> List((张三,男), (王五,男)),
女 -> List((李四,女)))

// 将分组后的映射转换为性别/人数元组列表
scala> res0.map(x => x._1 -> x._2.size)
res3: scala.collection.immutable.Map[String,Int] = Map(男 -> 2, 女 -> 1)
```

## reduce

聚合操作，可以将一个列表中的数据合并为一个。reduce表示将列表，传入一个函数进行聚合计算

方法签名

```scala
def reduce[A1 >: A](op: (A1, A1) ⇒ A1): A1
```

方法解析

| reduce方法 | API               | 说明                                                         |
| ---------- | ----------------- | ------------------------------------------------------------ |
| 泛型       | [A1 >: A]         | （下界）A1必须是集合元素类型的子类                           |
| 参数       | op: (A1, A1) ⇒ A1 | 传入函数对象，用来不断进行聚合操作<br />第一个A1类型参数为：当前聚合后的变量<br />第二个A1类型参数为：当前要进行聚合的元素 |
| 返回值     | A1                | 列表最终聚合为一个元素                                       |

* reduce和reduceLeft效果一致，表示从左到右计算

* reduceRight表示从右到左计算

案例

1. 定义一个列表，包含以下元素：1,2,3,4,5,6,7,8,9,10
2. 使用reduce计算所有元素的和

```scala
scala> val a = List(1,2,3,4,5,6,7,8,9,10)
a: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

scala> a.reduce((x,y) => x + y)
res5: Int = 55

// 第一个下划线表示第一个参数，就是历史的聚合数据结果
// 第二个下划线表示第二个参数，就是当前要聚合的数据元素
scala> a.reduce(_ + _)
res53: Int = 55

// 与reduce一样，从左往右计算
scala> a.reduceLeft(_ + _)
res0: Int = 55

// 从右往左聚合计算
scala> a.reduceRight(_ + _)
res1: Int = 55
```

## fold

fold与reduce很像，但是多了一个指定初始值参数

方法签名

```scala
def fold[A1 >: A](z: A1)(op: (A1, A1) ⇒ A1): A1
```

方法解析

| reduce方法 | API               | 说明                                                         |
| ---------- | ----------------- | ------------------------------------------------------------ |
| 泛型       | [A1 >: A]         | （下界）A1必须是集合元素类型的子类                           |
| 参数1      | z: A1             | 初始值                                                       |
| 参数2      | op: (A1, A1) ⇒ A1 | 传入函数对象，用来不断进行折叠操作<br />第一个A1类型参数为：当前折叠后的变量<br />第二个A1类型参数为：当前要进行折叠的元素 |
| 返回值     | A1                | 列表最终折叠为一个元素                                       |

* fold和foldLet效果一致，表示从左往右计算

* foldRight表示从右往左计算

案例

1. 定义一个列表，包含以下元素：1,2,3,4,5,6,7,8,9,10
2. 使用fold方法计算所有元素的和

```scala
scala> val a = List(1,2,3,4,5,6,7,8,9,10)
a: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

scala> a.fold(0)(_ + _)
res4: Int = 155
```

# 类和对象

scala是支持面向对象的，也有类和对象的概念。依然可以基于scala语言来开发面向对象的应用程序。

## 创建类和对象

* 使用class来定义一个类
* 使用new来创建对象

示例

创建一个Person类，并创建它的对象

步骤

1. 创建一个scala项目，并创建一个Object
2. 添加main方法
3. 创建类和对象

实现

1. 在IDEA中创建项目，并创建一个Object（main方法必须放在Object中）
2. 添加main方法
3. 创建一个Person类
4. 在main方法中创建Person类对象

```scala
package org.duo.oop

object _01ClassDemo {

  // 创建类
  class Person {}

  def main(args: Array[String]): Unit = {
    // 创建对象
    val p = new Person()
    println(p)
  }
}

```

简写方法

用法

* 如果类是空的，没有任何成员，可以省略{}

* 如果构造器的参数为空，可以省略()

示例

使用简写方法重新创建Person类和对象

```scala
package org.duo.oop

object _02ClassDemo {

  // 创建类，省略花括号
  class Person

  def main(args: Array[String]): Unit = {
    // 创建对象，省略括号
    val person = new Person
  }
}
```

## 定义和访问成员变量

一个类会有自己的属性，例如：人这样一个类，有自己的姓名和年龄。

- 在类中使用var/val定义成员变量
- 对象直接使用成员变量名称来访问成员变量

示例

1. 定义一个Person类，包含一个姓名和年龄字段
2. 创建一个名为"张三"、年龄为20岁的对象
3. 打印对象的名字和年龄

步骤

1. 创建一个Object，添加main方法
2. 创建Person类，添加姓名字段和年龄字段，并对字段进行初始化，让scala自动进行类型推断
3. 在main方法中创建Person类对象，设置成员变量为"张三"、20
4. 打印对象的名字和年龄

```scala
package org.duo.oop

object _03ClassDemo {

  class Person {
    // 定义成员变量
    var name = ""
    var age = 0
  }

  def main(args: Array[String]): Unit = {
    // 创建Person对象
    val person = new Person
    person.name = "zhangsan"
    person.age = 20

    // 获取变量值
    println(person.name)
    println(person.age)
  }
}

```

## 使用下划线初始化成员变量

scala中有一个更简洁的初始化成员变量的方式，可以让代码看起来更加简洁

用法

- 在定义var类型的成员变量时，可以使用_来初始化成员变量
  - String => null
  - Int => 0
  - Boolean => false
  - Double => 0.0
  - ...
- val类型的成员变量，必须要自己手动初始化

示例

1. 定义一个Person类，包含一个姓名和年龄字段
2. 创建一个名为"张三"、年龄为20岁的对象
3. 打印对象的名字和年龄

步骤

1. 创建一个Object，添加main方法
2. 创建Person类，添加姓名字段和年龄字段，指定数据类型，使用下划线初始化
3. 在main方法中创建Person类对象，设置成员变量为"张三"、20
4. 打印对象的名字和年龄

```scala
package org.duo.oop

object _04ClassDemo {

  class Person{
    // 使用下划线进行初始化
    var name:String = _
    var age:Int = _
  }

  def main(args: Array[String]): Unit = {
    val person = new Person

    println(person.name)
    println(person.age)
  }
}
```

## 定义成员方法

类可以有自己的行为，scala中也可以通过定义成员方法来定义类的行为。在scala的类中，也是使用def来定义成员方法。

```scala
package org.duo.oop

object _05ClassDemo {

  class Customer {
    var name: String = _
    var sex: String = _

    // 定义成员方法
    def sayHi(msg: String) = {
      println(msg)
    }
  }

  def main(args: Array[String]): Unit = {
    val customer = new Customer
    customer.name = "张三"
    customer.sex = "男"
    customer.sayHi("你好")
  }
}
```

## 访问修饰符

和Java一样，scala也可以通过访问修饰符，来控制成员变量和成员方法是否可以被访问。Java中的访问控制，同样适用于scala，可以在成员前面添加private/protected关键字来控制成员的可见性。但在scala中，没有public关键字，任何没有被标为private或protected的成员都是公共的

## 类的构造器

当创建类对象的时候，会自动调用类的构造器。之前使用的都是默认构造器，接下来要学习如何自定义构造器。

### 主构造器

Java的构造器，有构造列表和构造代码块

```java
class Person {
    // 成员变量
    private String name;
    private Integer age;

    // Java构造器
    public Person(String name, Integer age) {
        // 初始化成员变量
        this.name = name;
        this.age = age;
    }
}
```

在scala中，可以使用更简洁的语法来实现。

语法

```scala
class 类名(var/val 参数名:类型 = 默认值, var/val 参数名:类型 = 默认值){
    // 构造代码块
}
```

* 主构造器的参数列表是直接定义在类名后面，添加了val/var表示直接通过主构造器定义成员变量
* 构造器参数列表可以指定默认值
* 创建实例，调用构造器可以指定字段进行初始化
* 整个class中除了字段定义和方法定义的代码都是构造代码

示例

1. 定义一个Person类，通过主构造器参数列表定义姓名和年龄字段，并且设置它们的默认值
2. 在主构造器中输出"调用主构造器"
3. 创建"张三"对象（姓名为张三，年龄为20），打印对象的姓名和年龄
4. 创建"空"对象，不给构造器传入任何的参数，打印对象的姓名和年龄
5. 创建"man40"对象，不传入姓名参数，指定年龄为40，打印对象的姓名和年龄

```scala
package org.duo.oop

object _06ConstructorDemo {

  // 定义类的主构造器
  // 指定默认值
  class Person(var name: String = "", var age: Int = 0) {
    println("调用主构造器")
  }

  def main(args: Array[String]): Unit = {

    // 给构造器传入参数
    val zhangsan = new Person("张三", 20)
    println(zhangsan.name)
    println(zhangsan.age)
    println("---")

    // 不传入任何参数
    val empty = new Person
    println(empty.name)
    println(empty.age)
    println("---")

    // 指定字段进行初始化
    val man40 = new Person(age = 40)
    println(man40.name)
    println(man40.age)
  }
}
```

### 辅助构造器

在scala中，除了定义主构造器外，还可以根据需要来定义辅助构造器。例如：允许通过多种方式，来创建对象，这时候就可以定义其他更多的构造器。把除了主构造器之外的构造器称为辅助构造器。

语法

- 定义辅助构造器与定义方法一样，也使用def关键字来定义
- 这个方法的名字为this
- 辅助构造器的第一行代码，必须要调用主构造器或者其他辅助构造器

```scala
def this(参数名:类型, 参数名:类型) {
    // 第一行需要调用主构造器或者其他构造器
    // 构造器代码
}
```

示例

* 定义一个Customer类，包含一个姓名和地址字段
* 定义Customer类的主构造器（初始化姓名和地址）
* 定义Customer类的辅助构造器，该辅助构造器接收一个数组参数，使用数组参数来初始化成员变量
* 使用Person类的辅助构造器来创建一个"zhangsan"对象
  - 姓名为张三
  - 地址为北京
* 打印对象的姓名、地址

```scala
package org.duo.oop

object _07ConstructorDemo {

  class Customer(var name: String = "", var address: String = "") {
    // 定义辅助构造器
    def this(arr: Array[String]) = {
      // 辅助构造器必须要调用主构造器或者其他辅助构造器
      this(arr(0), arr(1))
    }
  }

  def main(args: Array[String]): Unit = {
    val zhangsan = new Customer(Array("张三", "北京"))

    println(zhangsan.name)
    println(zhangsan.address)
  }
}
```

## 单例对象

scala中没有Java中的静态成员，如果想要定义类似于Java的static变量、static方法，就要使用到scala中的单例对象——object.

单例对象表示全局仅有一个对象（类似于Java static概念）

* 定义单例对象和定义类很像，就是把class换成object
* 在object中定义的成员变量类似于Java的静态变量
* 可以使用object直接引用成员变量

示例

* 定义一个Dog单例对象，保存狗有几条腿
* 在main方法中打印狗腿的数量

```scala
package org.duo.oop

object _08ObjectDemo {

  // 定义一个单例对象
  object Dog {
    // 定义腿的数量
    val LEG_NUM = 4
  }

  def main(args: Array[String]): Unit = {
    println(Dog.LEG_NUM)
  }
}
```

### 工具类案例

- 编写一个DateUtil工具类专门用来格式化日期时间
- 定义一个方法，用于将日期（Date）转换为年月日字符串，例如：2030-10-05

```scala
package org.duo.oop

import java.text.SimpleDateFormat
import java.util.Date

object _10ObjectDemo {

  object DateUtils {
    // 在object中定义的成员变量，相当于Java中定义一个静态变量
    // 定义一个SimpleDateFormat日期时间格式化对象
    val simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm")

    // 相当于Java中定义一个静态方法
    def format(date: Date) = simpleDateFormat.format(date)
  }

  // main是一个静态方法，所以必须要写在object中
  def main(args: Array[String]): Unit = {
    println(DateUtils.format(new Date()))
  }
}
```

### main方法

scala和Java一样，如果要运行一个程序，必须有一个main方法。而在Java中main方法是静态的，而在scala中没有静态方法。在scala中，这个main方法必须放在一个单例对象中。

示例

```scala
package org.duo.oop

object Main5 {
  def main(args:Array[String]) = {
    println("hello, scala")
  }
}
```

实现App Trait来定义入口

示例

```scala
package org.duo.oop

object Main5 extends App {
  println("hello, scala")
}
```

## 伴生对象

一个class和object具有同样的名字。这个object称为伴生对象，这个class称为伴生类

* 伴生对象必须要和伴生类一样的名字
* 伴生对象和伴生类在同一个scala源文件中
* 伴生对象和伴生类可以互相访问private属性

```scala
package org.duo.oop

object _11ObjectDemo {

  class CustomerService {
    def save() = {
      println(s"${CustomerService.SERVICE_NAME}:保存客户")
    }
  }

  // CustomerService的伴生对象
  object CustomerService {
    private val SERVICE_NAME = "CustomerService"
  }

  def main(args: Array[String]): Unit = {
    val customerService = new CustomerService()
    customerService.save()
  }
}
```

### private[this]访问权限

如果某个成员的权限设置为private[this]，表示只能在当前类中访问。伴生对象也不可以访问

示例

* 定义一个Person类，包含一个name字段
* 定义Person类的伴生对象，定义printPerson方法
* 测试伴生对象是否能访问private[this]权限的成员

```scala
package org.duo.oop

object _12ObjectDemo {

  class Person(private[this] var name: String)

  object Person {
    def printPerson(person: Person): Unit = {
      // 这里编译报错。但移除掉[this]就可以访问了
      println(person.name)
    }
  }

  def main(args: Array[String]): Unit = {
    val person = new Person("张三")
    Person.printPerson(person)
  }
}
```

### apply方法

之前使用过这种方式来创建一个Array对象。

```scala
// 创建一个Array对象
val a = Array(1,2,3,4)
```

这种写法非常简便，不需要再写一个new，然后敲一个空格，再写类名。可以通过伴生对象的apply方法来实现。

定义apply方法

```scala
object 伴生对象名 {
	def apply(参数名:参数类型, 参数名:参数类型...) = new 类(...)
}
```

创建对象

伴生对象名(参数1, 参数2...)

示例

- 定义一个Person类，它包含两个字段：姓名和年龄
- 重写apply方法，使用Person类名就可以创建对象
- 在main方法中创建该类的对象，并打印姓名和年龄

```scala
package org.duo.oop

object _13ApplyDemo {

  class Person(var name: String = "", var age: Int = 0)

  object Person {
    // 定义apply方法，接收两个参数
    def apply(name: String, age: Int) = new Person(name, age)
  }

  def main(args: Array[String]): Unit = {
    // 使用伴生对象名称来创建对象
    val zhangsan = Person("张三", 20)
    println(zhangsan.name)
    println(zhangsan.age)
  }

```

# 继承

scala语言是支持面向对象编程的，可以使用scala来实现继承，通过继承来减少重复代码。

* scala和Java一样，使用extends关键字来实现继承
* 可以在子类中定义父类中没有的字段和方法，或者重写父类的方法
* 类和单例对象都可以从某个父类继承

语法

```scala
class/object 子类 extends 父类 {
    ..
}
```

## 类继承

```scala
package org.duo.oop

object _14ExtendsDemo {

  class Person {

    var name = "super"
    def getName = this.name
  }

  class Student extends Person

  def main(args: Array[String]): Unit = {

    val p2 = new Student()
    p2.name = "张三"
    println(p2.getName)
  }
}
```

## 单例对象继承

```scala
package org.duo.oop

object _14ExtendsDemo {

  class Person {

    var name = "super"
    def getName = this.name
  }

  object Student extends Person

  def main(args: Array[String]): Unit = {

    println(Student.getName)
  }
}
```

## override和super

类似于Java语言，在子类中使用override需要来重写父类的成员，可以使用super来引用父类

- 子类要覆盖父类中的一个方法，必须要使用override关键字
- 使用override来重写一个val字段
- 使用super关键字来访问父类的成员方法

示例

- 定义一个Person类，包含
  - 姓名字段（不可重新赋值）
  - 获取姓名方法
- 定义一个Student类
  - 重写姓名字段
  - 重写获取姓名方法，返回"hello, "  + 姓名
- 创建Student对象示例，调用它的getName方法

```scala
package org.duo.oop

object _15OverrideDemo {

  class Person {

    val name = "super"

    def getName = name
  }

  class Student extends Person {

    // 重写val字段
    override val name: String = "child"

    // 重写getName方法，调用父类的getName方法，注意这里虽然用的是父类的方法，但是由于name变量被子类重写了，所以答应的仍然是被子类重写过的值
    override def getName: String = "hello, " + super.getName
  }

  def main(args: Array[String]): Unit = {
    println(new Student().getName)
  }
}
```

# 类型判断

在scala中，有两种方式来进行类型判断

* isInstanceOf

  判断对象是否为指定类的对象

* getClass/classOf

  asInstanceOf将对象转换为指定类型

用法

```scala
// 判断对象是否为指定类型
val trueOrFalse:Boolean = 对象.isInstanceOf[类型]
// 将对象转换为指定类型
val 变量 = 对象.asInstanceOf[类型]
```

```scala
package org.duo.oop

object _16InstanceDemo {

  class Person3

  class Student3 extends Person3

  def main(args: Array[String]): Unit = {
    val s1: Person3 = new Student3

    // 判断s1是否为Student3类型
    if (s1.isInstanceOf[Student3]) {
      // 将s1转换为Student3类型
      val s2 = s1.asInstanceOf[Student3]
      println(s2)
    }

  }
}
```

isInstanceOf 只能判断对象是否为指定类以及其子类的对象，而不能精确的判断出，对象就是指定类的对象。如果要求精确地判断出对象就是指定类的对象，那么就只能使用 getClass 和 classOf 。

用法

- p.getClass可以精确获取对象的类型
- classOf[x]可以精确获取类型
- 使用==操作符可以直接比较类型

```scala
package org.duo.oop

object _17InstanceDemo {

  class Person4

  class Student4 extends Person4

  def main(args: Array[String]) {

    val p: Person4 = new Student4
    //判断p是否为Person4类的实例
    println(p.isInstanceOf[Person4]) //true
    //判断p的类型是否为Person4类
    println(p.getClass == classOf[Person4]) //false
    //判断p的类型是否为Student4类
    println(p.getClass == classOf[Student4]) //true
  }
}
```

# 抽象类&抽象字段

## 抽象类

如果类的某个成员在当前类中的定义是不包含完整的，它就是一个抽象类

不完整定义有两种情况：

1. 方法没有方法体（抽象方法）
2. 变量没有初始化（抽象字段）

定义抽象类和Java一样，在类前面加上abstract关键字

```scala
// 定义抽象类
abstract class 抽象类名 {
  // 定义抽象字段
  val 抽象字段名:类型
  // 定义抽象方法
  def 方法名(参数:参数类型,参数:参数类型...):返回类型
}
```

```scala
package org.duo.oop

object _18AbstractDemo {

  // 创建形状抽象类
  abstract class Shape {
    def area: Double
  }

  // 创建正方形类
  class Square(var edge: Double /*边长*/) extends Shape {
    // 实现父类计算面积的方法
    override def area: Double = edge * edge
  }

  // 创建长方形类
  class Rectangle(var length: Double /*长*/ , var width: Double /*宽*/) extends Shape {
    override def area: Double = length * width
  }

  // 创建圆形类
  class Cirle(var radius: Double /*半径*/) extends Shape {
    override def area: Double = Math.PI * radius * radius
  }

  def main(args: Array[String]): Unit = {

    val s1: Shape = new Square(2)
    val s2: Shape = new Rectangle(2, 3)
    val s3: Shape = new Cirle(2)

    println(s1.area)
    println(s2.area)
    println(s3.area)
  }
}
```

## 抽象字段

语法

```scala
abstract class 抽象类 {
    val/var 抽象字段:类型
}
```

```scala
package org.duo.oop

object _19AbstractDemo {

  // 定义一个人的抽象类
  abstract class Person6 {
    // 没有初始化的val字段就是抽象字段
    val WHO_AM_I: String
  }

  class Student6 extends Person6 {
    override val WHO_AM_I: String = "学生"
  }

  class Policeman6 extends Person6 {
    override val WHO_AM_I: String = "警察"
  }

  def main(args: Array[String]): Unit = {

    val p1 = new Student6
    val p2 = new Policeman6

    println(p1.WHO_AM_I)
    println(p2.WHO_AM_I)
  }
}
```

# 匿名内部类

匿名内部类是没有名称的子类，直接用来创建实例对象。Spark的源代码中有大量使用到匿名内部类。scala中的匿名内部类使用与Java一致。

语法

```scala
val/var 变量名 = new 类/抽象类 {
    // 重写方法
}
```

示例说明

1. 创建一个Person抽象类，并添加一个sayHello抽象方法
2. 添加main方法，通过创建匿名内部类的方式来实现Person
3. 调用匿名内部类对象的sayHello方法

```scala
package org.duo.oop

object _20AnonymousDemo {

  abstract class Person7 {
    def sayHello:Unit
  }

  def main(args: Array[String]): Unit = {
    // 直接用new来创建一个匿名内部类对象
    val p1 = new Person7 {
      override def sayHello: Unit = println("我是一个匿名内部类")
    }
    p1.sayHello
  }
}
```

# 特质

scala中没有Java中的接口（interface），替代的概念是——特质

## 定义

- 特质是scala中代码复用的基础单元
- 它可以将方法和字段定义封装起来，然后添加到类中
- 与类继承不一样的是，类继承要求每个类都只能继承一个超类，而一个类可以添加任意数量的特质。
- 特质的定义和抽象类的定义很像，但它是使用trait关键字

语法

```scala
trait 名称 {
    // 抽象字段
    // 抽象方法
}
```

继承特质

```scala
class 类 extends 特质1 with 特质2 {
    // 字段实现
    // 方法实现
}
```

- 使用extends来继承trait（scala不论是类还是特质，都是使用extends关键字）

- 如果要继承多个trait，则使用with关键字

## trait作为接口使用

trait作为接口使用，与java的接口使用方法一样。

### 继承单个trait

1. 创建一个Logger特质，添加一个接受一个String类型参数的log抽象方法
2. 创建一个ConsoleLogger类，继承Logger特质，实现log方法，打印消息
3. 添加main方法，创建ConsoleLogger对象，调用log方法

```scala
package org.duo.oop

object _21TraitDemo {

  trait Logger {
    // 抽象方法
    def log(message: String)
  }

  class ConsoleLogger extends Logger {
    override def log(message: String): Unit = println("控制台日志:" + message)
  }

  def main(args: Array[String]): Unit = {
    val logger = new ConsoleLogger
    logger.log("这是一条日志")
  }
}
```

### 继承多个trait

```scala
package org.duo.oop

object _22TraitDemo {

  trait MessageSender {
    def send(msg: String)
  }

  trait MessageReceive {
    def receive(): String
  }

  class MessageWorker extends MessageSender with MessageReceive {
    override def send(msg: String): Unit = println(s"发送消息:${msg}")

    override def receive(): String = "你好！我叫一个好人！"
  }

  def main(args: Array[String]): Unit = {
    val worker = new MessageWorker
    worker.send("hello")
    println(worker.receive())
  }
}
```

### object继承trait

```scala
package org.duo.oop

object _23TraitDemo {

  trait Logger {
    def log(message: String)
  }

  object ConsoleLogger extends Logger {
    override def log(message: String): Unit = println("控制台消息:" + message)
  }

  def main(args: Array[String]): Unit = {
    ConsoleLogger.log("程序退出!")
  }
}
```

## 定义具体的方法

和类一样，trait中还可以定义具体的方法

```scala
package org.duo.oop

object _24TraitDemo {

  trait LoggerDetail {
    // 在trait中定义具体方法
    def log(msg: String) = println(msg)
  }

  class UserService extends LoggerDetail {
    def add() = log("添加用户")
  }

  def main(args: Array[String]): Unit = {
    val userService = new UserService
    userService.add()
  }
}
```

## 定义抽象字段

* 在trait中可以定义具体字段和抽象字段

* 继承trait的子类自动拥有trait中定义的字段
* 字段直接被添加到子类中

```scala
package org.duo.oop

import java.text.SimpleDateFormat
import java.util.Date

object _25TraitDemo {

  trait Logger {
    val sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm")

    def log(msg: String)
  }

  class ConsoleLogger extends Logger {
    override def log(msg: String): Unit = {
      val info = s"${sdf.format(new Date())}:控制台消息:${msg}"
      println(info)
    }
  }

  def main(args: Array[String]): Unit = {
    val logger = new ConsoleLogger()
    logger.log("NullPointerException")
  }
}
```

## 实现模板模式

```scala
package org.duo.oop

object _26TraitDemo {

  trait Logger {

    def log(msg: String)

    def info(msg: String) = log("INFO:" + msg)

    def warn(msg: String) = log("WARN:" + msg)

    def error(msg: String) = log("ERROR:" + msg)
  }

  class ConsoleLogger extends Logger {

    override def log(msg: String): Unit = {
      println(msg)
    }
  }

  def main(args: Array[String]): Unit = {

    val logger = new ConsoleLogger
    logger.info("信息日志")
    logger.warn("警告日志")
    logger.error("错误日志")
  }
}
```

## 对象混入trait

scala中可以将trait混入到对象中，就是将trait中定义的方法、字段添加到一个对象中

语法

```scala
val/var 对象名 = new 类 with 特质
```

示例

```scala
package org.duo.oop

object _27TraitDemo {

  trait Logger {
    def log(msg: String) = println(msg)
  }

  class UserService

  def main(args: Array[String]): Unit = {
    val service = new UserService with Logger
    service.log("混入的方法")
  }
}
```

## 实现调用链模式

类继承了多个trait后，可以依次调用多个trait中的同一个方法，只要让多个trait中的同一个方法在最后都依次执行super关键字即可。类中调用多个tait中都有这个方法时，首先会从最右边的trait方法开始执行，然后依次往左执行，形成一个调用链条。

```scala
package org.duo.oop

object _28TraitDemo {

  trait HandlerTrait {
    def handle(data: String) = println("处理数据...")
  }

  trait DataValidHanlderTrait extends HandlerTrait {
    override def handle(data: String): Unit = {
      println("验证数据...")
      super.handle(data)
    }
  }

  trait SignatureValidHandlerTrait extends HandlerTrait {
    override def handle(data: String): Unit = {
      println("校验签名...")
      super.handle(data)
    }
  }

  class PayService extends DataValidHanlderTrait with SignatureValidHandlerTrait {
    override def handle(data: String): Unit = {
      println("准备支付...")
      super.handle(data)
    }
  }

  def main(args: Array[String]): Unit = {
    val service = new PayService
    service.handle("支付参数")
  }
}
```

# 样例类&样例对象

## 样例类

样例类是一种特殊类，它可以用来快速定义一个用于保存数据的类（类似于Java POJO类），在后续要学习并发编程和spark、flink这些框架也都会经常使用它。

语法格式

```scala
case class 样例类名([var/val] 成员变量名1:类型1, 成员变量名2:类型2, 成员变量名3:类型3)
```

* 如果要实现某个成员变量可以被修改，可以添加var
* 默认为val，可以省略

示例一

* 定义一个Person样例类，包含姓名和年龄成员变量
* 创建样例类的对象实例（"张三"、20），并打印它

```scala
package org.duo.oop

object _01CaseClassDemo {

  case class Person(name: String, age: Int)

  def main(args: Array[String]): Unit = {
    val zhangsan = Person("张三", 20)

    println(zhangsan)
  }
}
```

示例二

* 定义一个Person样例类，包含姓名和年龄成员变量
* 创建样例类的对象实例（"张三"、20）
* 修改张三的年龄为23岁，并打印

```scala
package org.duo.oop

object _02CaseClassDemo {

  case class Person(var name: String, var age: Int)

  def main(args: Array[String]): Unit = {

    val zhangsan = Person("张三", 20)
    zhangsan.age = 23
    println(zhangsan)
  }
}
```

当定义一个样例类，编译器自动帮助实现了以下几个有用的方法：

* apply方法

  apply方法可以让我们快速地使用类名来创建对象。参考以下代码：

  ```scala
  package org.duo.oop
  
  object _03CaseClassDemo {
  
    case class CasePerson(name: String, age: Int)
  
    def main(args: Array[String]): Unit = {
      val lisi = CasePerson("李四", 21)
      println(lisi.toString)
    }
  }
  ```

* toString方法

  toString返回样例类名称(成员变量1, 成员变量2, 成员变量3....)，我们可以更方面查看样例类的成员

  ```scala
  package org.duo.oop
  
  object _03CaseClassDemo {
  
    case class CasePerson(name: String, age: Int)
  
    def main(args: Array[String]): Unit = {
      val lisi = CasePerson("李四", 21)
      println(lisi.toString)
      // 输出：CasePerson(李四,21)
    }
  }
  ```

* equals方法

  样例类自动实现了equals方法，可以直接使用==比较两个样例类是否相等，即所有的成员变量是否相等

  ```scala
  package org.duo.oop
  
  object _04CaseClassDemo {
  
    case class CasePerson(name: String, age: Int)
  
    def main(args: Array[String]): Unit = {
  
      val lisi1 = CasePerson("李四", 21)
      val lisi2 = CasePerson("李四", 21)
      println(lisi1 == lisi2)
      // 输出：true
    }
  }
  ```

* hashCode方法

  样例类自动实现了hashCode方法，如果所有成员变量的值相同，则hash值相同，只要有一个不一样，则hash值不一样。

  ```scala
  package org.duo.oop
  
  object _04CaseClassDemo {
  
    case class CasePerson(name: String, age: Int)
  
    def main(args: Array[String]): Unit = {
  
      val lisi1 = CasePerson("李四", 21)
      val lisi2 = CasePerson("李四", 21)
      println(lisi1.hashCode())
      println(lisi2.hashCode())
    }
  }
  ```

* copy方法

  样例类实现了copy方法，可以快速创建一个相同的实例对象，可以使用带名参数指定给成员进行重新赋值

  ```scala
  package org.duo.oop
  
  object _04CaseClassDemo {
  
    case class CasePerson(name: String, age: Int)
  
    def main(args: Array[String]): Unit = {
  
      val lisi1 = CasePerson("李四", 21)
      val wangwu = lisi1.copy(name="王五")
      println(wangwu)
      // 输出：CasePerson(王五,21)
    }
  }
  ```

## 样例对象

它主要用在两个地方：

1. 定义枚举
2. 作为没有任何参数的消息传递（Akka编程）

使用case object可以创建样例对象。样例对象是单例的，而且它没有主构造器

定义

```scala
case object 样例对象名
```

定义枚举

* 定义一个性别Sex枚举，它只有两个实例（男性——Male、女性——Female）
* 创建一个Person类，它有两个成员（姓名、性别）
* 创建两个Person对象（"张三"、男性）、（"李四"、"女"）

```scala
package org.duo.oop

object _01CaseObjectDemo {

  /*定义一个性别特质*/
  trait Sex
  case object Male extends Sex // 定义一个样例对象并实现了Sex特质
  case object Female extends Sex

  case class Person(name: String, sex: Sex)

  def main(args: Array[String]): Unit = {
    val zhangsan = Person("张三", Male)
    val lisi = Person("李四", Female)
    println(zhangsan)
    println(lisi)
  }
}
```

# 模式匹配

scala中有一个非常强大的模式匹配机制，可以应用在很多场景：

- switch语句
- 类型查询
- 使用模式匹配快速获取数据

## 简单模式匹配

在Java中，有switch关键字，可以简化if条件判断语句。在scala中，可以使用match表达式替代。

语法格式

```scala
变量 match {
    case "常量1" => 表达式1
    case "常量2" => 表达式2
    case "常量3" => 表达式3
    case _ => 表达式4		// 默认匹配
}
```

示例

```scala
package org.duo.oop

import scala.io.StdIn

object _01PatternMatchingDemo {

  def main(args: Array[String]): Unit = {

    println("请输出一个词：")
    // StdIn.readLine表示从控制台读取一行文本
    val name = StdIn.readLine()

    val result = name match {
      case "hadoop" => "大数据分布式存储和计算框架"
      case "zookeeper" => "大数据分布式协调服务框架"
      case "spark" => "大数据分布式内存计算框架"
      case _ => "未匹配"
    }

    println(result)
  }
}
```

## 匹配类型

除了像Java中的switch匹配数据之外，match表达式还可以进行类型匹配。如果要根据不同的数据类型，来执行不同的逻辑，也可以使用match表达式来实现。

语法格式

```scala
变量 match {
    case 类型1变量名: 类型1 => 表达式1
    case 类型2变量名: 类型2 => 表达式2
    case 类型3变量名: 类型3 => 表达式3
    ...
    case _ => 表达式4
}
```

示例

```scala
package org.duo.oop

object _02PatternMatchingDemo {

  def main(args: Array[String]): Unit = {

    val a: Any = "hadoop"

    val result = a match {
      case _: String => "String"
      case _: Int => "Int"
      case _: Double => "Double"
    }

    println(result)
  }
}
```

## 守卫

在Java中，只能简单地添加多个case标签，例如：要匹配0-7，就需要写出来8个case语句。例如：

```java
int a = 0;
switch(a) {
    case 0: a += 1;
    case 1: a += 1;
    case 2: a += 1;
    case 3: a += 1;
    case 4: a += 2;
    case 5: a += 2;
    case 6: a += 2;
    case 7: a += 2;
    default: a = 0;
}
```

在scala中，可以使用守卫来简化上述代码——也就是在case语句中添加if条件判断。

示例

需求说明

* 从控制台读入一个数字a（使用StdIn.readInt）
* 如果 a >= 0 而且 a <= 3，打印[0-3]
* 如果 a >= 4 而且 a <= 8，打印[3,8]
* 否则，打印未匹配

```scala
package org.duo.oop

import scala.io.StdIn

object _03PatternMatchingDemo {

  def main(args: Array[String]): Unit = {

    val a = StdIn.readInt()

    a match {
      case _ if a >= 0 && a <= 3 => println("[0-3]")
      case _ if a >= 4 && a <= 8 => println("[3-8]")
      case _ => println("未匹配")
    }
  }
}
```

## 匹配样例类

scala可以使用模式匹配来匹配样例类，从而可以快速获取样例类中的成员数据。

示例

需求说明

* 创建两个样例类Customer、Order
  * Customer包含姓名、年龄字段
  * Order包含id字段
* 分别定义两个案例类的对象，并指定为Any类型
* 使用模式匹配这两个对象，并分别打印它们的成员变量值

```scala
package org.duo.oop

object _04PatternMatchingDemo {

  // 1. 创建两个样例类
  case class Person(name: String, age: Int)

  case class Order(id: String)

  def main(args: Array[String]): Unit = {

    // 2. 创建样例类对象，并赋值为Any类型
    val zhangsan: Any = Person("张三", 20)
    val order1: Any = Order("001")

    // 3. 使用match...case表达式来进行模式匹配
    // 获取样例类中成员变量
    order1 match {
      case Person(name, age) => println(s"姓名：${name} 年龄：${age}")
      case Order(id1) => println(s"ID为：${id1}")
      case _ => println("未匹配")
    }
  }
}
```

## 匹配集合

scala中的模式匹配，还能用来匹配集合。

### 匹配数组

示例说明

* 依次修改代码定义以下三个数组

  ```scala
  Array(1,x,y)   // 以1开头，后续的两个元素不固定
  Array(0)	   // 只匹配一个0元素的元素
  Array(0, _*)  // 可以任意数量，但是以0开头
  ```

* 使用模式匹配上述数组

  ```scala
  package org.duo.oop
  
  object _05PatternMatchingDemo {
  
    def main(args: Array[String]): Unit = {
  
      val arr = Array(1, 3, 5)
      arr match {
        case Array(1, x, y) => println(x + " " + y)
        case Array(0) => println("only 0")
        case Array(0, _*) => println("0 ...")
        case _ => println("something else")
      }
    }
  }
  ```

### 匹配列表

示例说明

* 依次修改代码定义以下三个列表

  ```scala
  List(0)				// 只保存0一个元素的列表
  List(0,...)   		// 以0开头的列表，数量不固定
  List(x,y)	   		// 只包含两个元素的列表  
  ```

* 使用模式匹配上述列表

  ```scala
  package org.duo.oop
  
  object _06PatternMatchingDemo {
  
    def main(args: Array[String]): Unit = {
  
      val list = List(0, 1, 2)
  
      list match {
        case 0 :: Nil => println("只有0的列表")
        case 0 :: tail => println("0开头的列表")
        case x :: y :: Nil => println(s"只有另两个元素${x}, ${y}的列表")
        case _ => println("未匹配")
      }
    }
  }
  ```

### 匹配元组

示例说明

* 依次修改代码定义以下两个元组

  ```scala
  (1, x, y)		// 以1开头的、一共三个元素的元组
  (x, y, 5)   // 一共有三个元素，最后一个元素为5的元组
  ```

* 使用模式匹配上述元素

  ```scala
  package org.duo.oop
  
  object _07PatternMatchingDemo {
  
    def main(args: Array[String]): Unit = {
  
      val tuple = (2, 2, 5)
  
      tuple match {
        case (1, x, y) => println(s"三个元素，1开头的元组：1, ${x}, ${y}")
        case (x, y, 5) => println(s"三个元素，5结尾的元组：${x}, ${y}, 5")
        case _ => println("未匹配")
      }
    }
  }
  ```

### 变量声明中的模式匹配

在定义变量的时候，可以使用模式匹配快速获取数据

### 获取数组中的元素

需求说明

* 生成包含0-10数字的数组，使用模式匹配分别获取第二个、第三个、第四个元素

参考代码

```scala
val array = (1 to 10).toArray
val Array(_, x, y, z, _*) = array

println(x, y, z)
```

### 获取List中的数据

需求说明

- 生成包含0-10数字的列表，使用模式匹配分别获取第一个、第二个元素

参考代码

```scala
val list = (1 to 10).toList
val x :: y :: tail = list

println(x, y)
```

## 模式匹配中的@操作符

当需要处理匹配的对象本身时就可以用@将匹配对象绑定到一个变量上

```scala
package org.duo.syntax

object _28PatternMatchingDemo {

  // 1. 创建两个样例类
  case class Person(name: String, age: Int)

  def main(args: Array[String]): Unit = {

    val zhangsan: Any = Person("张三", 20)

    // p表示匹配的对象本身，当需要使用匹配中的对象时，就需要用@了
    zhangsan match {
      case p@Person(_, age) => println(s"name:${p.name}, age: $age")
      case _ => println("Not a person")
    }

    // 还有一种用法，和上面的@用法很相似，不过是常见的类型匹配，这里的p也是一个匹配上的对象，和上面的区别是，这里p需要定义类型，而上面@的用法是个对象，对象中的age变量，是可以直接用的。
    zhangsan match {
      case p: Person => println(s"name:${p.name}, age: ${p.age}")
      case _ => println("Not a person")
    }

    // 在看下这种用法，这种是不带@或:的，匹配成功后，可以使用name、age变量，但是无法引用匹配的对象本身。
    zhangsan match {
      case Person(name, age) => println(s"name:$name, age: $age")
      case _ => println("Not a person")
    }
  }
}
```

# Option类型

使用Option类型，可以用来有效避免空引用(null)异常。也就是说，将来返回某些数据时，可以返回一个Option类型来替代。

定义

scala中，Option类型来表示可选值。这种类型的数据有两种形式：

- Some(x)：表示实际的值
- None：表示没有值；使用getOrElse方法，当值为None是可以指定一个默认值

示例一

示例说明

* 定义一个两个数相除的方法，使用Option类型来封装结果
* 然后使用模式匹配来打印结果
  * 不是除零，打印结果
  * 除零打印异常错误

```scala
package org.duo.oop

object _01OptionDemo {

  /**
   * 定义除法操作
   *
   * @param a 参数1
   * @param b 参数2
   * @return Option包装Double类型
   */
  def dvi(a: Double, b: Double): Option[Double] = {
    if (b != 0) {
      Some(a / b)
    }
    else {
      None
    }
  }

  def main(args: Array[String]): Unit = {
    val result1 = dvi(1.0, 5)

    result1 match {
      case Some(x) => println(x)
      case None => println("除零异常")
    }
  }
}
```

示例二

示例说明

* 重写上述案例，使用getOrElse方法，当除零时，或者默认值为0

```scala
package org.duo.oop

object _02OptionDemo {

  def dvi(a: Double, b: Double) = {
    if (b != 0) {
      Some(a / b)
    }
    else {
      None
    }
  }

  def main(args: Array[String]): Unit = {
    val result = dvi(1, 0).getOrElse(0)

    println(result)
  }
}
```

# 偏函数

偏函数可以提供了简洁的语法，可以简化函数的定义。配合集合的函数式编程，可以让代码更加优雅。

定义

* 偏函数被包在花括号内没有match的一组case语句是一个偏函数

* 偏函数是PartialFunction[A, B]的一个实例
  * A代表输入参数类型
  * B代表返回结果类型

示例一

示例说明

定义一个偏函数，根据以下方式返回

| 输入 | 返回值 |
| ---- | ------ |
| 1    | 一     |
| 2    | 二     |
| 3    | 三     |
| 其他 | 其他   |

```scala
package org.duo.oop

object _01PartialFunctionDemo {

  // func1是一个输入参数为Int类型，返回值为String类型的偏函数
  val func1: PartialFunction[Int, String] = {
    case 1 => "一"
    case 2 => "二"
    case 3 => "三"
    case _ => "其他"
  }

  def main(args: Array[String]): Unit = {
    func1(2)
  }
}
```

示例二

示例说明

* 定义一个列表，包含1-10的数字

* 请将1-3的数字都转换为[1-3]
* 请将4-8的数字都转换为[4-8]
* 将其他的数字转换为(8-*]

```scala
package org.duo.oop

object _02PartialFunctionDemo {

  val list = (1 to 10).toList

  val list2 = list.map {
    case x if x >= 1 && x <= 3 => "[1-3]"
    case x if x >= 4 && x <= 8 => "[4-8]"
    case x if x > 8 => "(8-*]"
  }

  def main(args: Array[String]): Unit = {

    println(list2)
  }
}
```

# 正则表达式

在scala中，可以很方便地使用正则表达式来匹配数据。

Regex类

* scala中提供了Regex类来定义正则表达式

* 要构造一个RegEx对象，直接使用String类的r方法即可

* 建议使用三个双引号来表示正则表达式，不然就得对正则中的反斜杠来进行转义

```scala
val regEx = """正则表达式""".r
```

findAllMatchIn方法

* 使用findAllMatchIn方法可以获取到所有正则匹配到的字符串

示例一

示例说明

* 定义一个正则表达式，来匹配邮箱是否合法
* 合法邮箱测试：qq12344@163.com
* 不合法邮箱测试：qq12344@.com

```scala
package org.duo.oop

object _01RegExDemo {

  val r = """.+@.+\..+""".r
  val eml1 = "qq12344@163.com"
  val eml2 = "qq12344@.com"

  def main(args: Array[String]): Unit = {

    if (r.findAllMatchIn(eml1).size > 0) {
      println(eml1 + "邮箱合法")
    }
    else {
      println(eml1 + "邮箱不合法")
    }

    if (r.findAllMatchIn(eml2).size > 0) {
      println(eml2 + "邮箱合法")
    }
    else {
      println(eml2 + "邮箱不合法")
    }
  }
}
```

示例二

示例说明

找出以下列表中的所有不合法的邮箱

```markdown
"38123845@qq.com", "a1da88123f@gmail.com", "zhansan@163.com", "123afadff.com"
```

```scala
package org.duo.oop

object _02RegExDemo {

  val emlList =
    List("38123845@qq.com", "a1da88123f@gmail.com", "zhansan@163.com", "123afadff.com")

  val regex = """.+@.+\..+""".r

  val invalidEmlList = emlList.filter {
    x =>
      if (regex.findAllMatchIn(x).size < 1) true else false
  }

  def main(args: Array[String]): Unit = {

    println(invalidEmlList)
  }
}
```

示例三

示例说明

* 有以下邮箱列表

  ```scala
  "38123845@qq.com", "a1da88123f@gmail.com", "zhansan@163.com", "123afadff.com"
  ```

* 使用正则表达式进行模式匹配，匹配出来邮箱运营商的名字。例如：邮箱zhansan@163.com，需要将163匹配出来

  * 使用括号来匹配分组

* 打印匹配到的邮箱以及运营商

```scala
package org.duo.oop

object _03RegExDemo {

  // 使用括号表示一个分组
  val regex = """.+@(.+)\..+""".r // 使用".r"方法可使任意字符串变成一个Regex实例

  val emlList =
    List("38123845@qq.com", "a1da88123f@gmail.com", "zhansan@163.com", "123afadff.com")

  val emlCmpList = emlList.map {
    // company为第一个分组结果，可以匹配多个分组，
    // 如果要实现匹配多个分组的话：
    // 1.首先在定义正则表示式的地方，增加一个分组
    // val regex = """.+@(.+)\.(.+)""".r
    // 2.在模式匹配的地方定义两个变量
    // case x@regex(company, net) => s"${x} => ${company} => ${net}"
    case x @ regex(company) => s"${x} => ${company}"
    case x => x + "=>未知"
  }

  def main(args: Array[String]): Unit = {
    println(emlCmpList)
  }
}
```

# 异常处理

语法格式

```scala
try {
    // 代码
}
catch {
    case ex:异常类型1 => // 代码
    case ex:异常类型2 => // 代码
}
finally {
    // 代码
}
```

- try中的代码是我们编写的业务处理代码
- 在catch中表示当出现某个异常时，需要执行的代码
- 在finally中，是不管是否出现异常都会执行的代码

抛出异常

```scala
def main(args: Array[String]): Unit = {
  throw new Exception("这是一个异常")
}
```

- scala不需要在方法上声明要抛出的异常，它已经解决了在Java中被认为是设计失败的检查型异常。

# 提取器

通过模式匹配，可以快速匹配样例类中的成员变量，但不是所有的类都可以进行这样的模式匹配，普通类要支持模式匹配，必须要实现一个提取（样例类自动实现了apply、unapply方法）：

实现一个类的伴生对象中的apply方法，可以用类名来快速构建一个对象。伴生对象中，还有一个unapply方法。与apply相反，unapply是将该类的对象，拆解为一个个的元素。

要实现一个类的提取器，只需要在该类的伴生对象中实现一个unapply方法即可。

语法格式

```scala
def unapply(stu:Student):Option[(类型1, 类型2, 类型3...)] = {
    if(stu != null) {
        Some((变量1, 变量2, 变量3...))
    }
    else {
        None
    }
}
```

示例

示例说明

* 创建一个Student类，包含姓名年龄两个字段
* 实现一个类的解构器，并使用match表达式进行模式匹配，提取类中的字段。

```scala
package org.duo.oop

object _01UnapplyDemo {

  class Student(var name: String, var age: Int)

  object Student {

    def apply(name: String, age: Int) = {

      new Student(name, age)
    }

    def unapply(student: Student) = {

      val tuple = (student.name, student.age)
      Some(tuple)
    }
  }

  def main(args: Array[String]): Unit = {

    val zhangsan = Student("张三", 20)
    zhangsan match {
      case Student(name, age) => println(s"${name} => ${age}")
    }
  }
}
```

# 泛型

## 泛型方法

scala和Java一样，类和特质、方法都可以支持泛型。

```scala
// 定义泛型变量
val list1:List[String] = List("1", "2", "3")
```

定义一个泛型方法

语法格式

```scala
def 方法名[泛型名称](..) = {
    //...
}
```

示例

示例说明

* 用一个方法来获取任意类型数组的中间的元素
  * 不考虑泛型直接实现（基于Array[Int]实现）
  * 加入泛型支持

不考虑泛型的实现

```scala
package org.duo.oop

object _01GenericDemo {

  def getMiddle(arr: Array[Int]) = arr(arr.length / 2)

  def main(args: Array[String]): Unit = {
    val arr1 = Array(1, 2, 3, 4, 5)
    println(getMiddle(arr1))
  }
}
```

加入泛型支持

```scala
package org.duo.oop

object _02GenericDemo {

  def getMiddleElement[T](array: Array[T]) =
    array(array.length / 2)

  def main(args: Array[String]): Unit = {
    println(getMiddleElement(Array(1, 2, 3, 4, 5)))
    println(getMiddleElement(Array("a", "b", "c", "d", "e")))
  }
}
```

## 泛型类

scala的类也可以定义泛型。

语法格式

```scala
class 类[T](val 变量名: T)
```

* 定义一个泛型类，直接在类名后面加上方括号，指定要使用的泛型参数
* 指定类对应的泛型参数后，就使用这些类型参数来定义变量了

示例

* 实现一个Pair泛型类
* Pair类包含两个字段，而且两个字段的类型不固定
* 创建不同类型泛型类对象，并打印

```scala
package org.duo.oop

object _03GenericDemo {

  case class Pair[T, W](var a: T, var b: W)

  def main(args: Array[String]): Unit = {
    val pairList = List(
      Pair("Hadoop", "Storm"),
      Pair("Hadoop", 2008),
      Pair(1.0, 2.0),
      Pair("Hadoop", Some(1.9))
    )
    println(pairList)
  }
}
```

## 上下界

在定义方法/类的泛型时，限定必须从哪个类继承、或者必须是哪个类的父类。此时，就需要使用到上下界。

### 上界定义

使用<: 类型名表示给类型添加一个上界，表示泛型参数必须要从该类（或本身）继承

语法格式

```scala
[T <: 类型]
```

示例

* 定义一个Person类
* 定义一个Student类，继承Person类
* 定义一个demo泛型方法，该方法接收一个Array参数，
* 限定demo方法的Array元素类型只能是Person或者Person的子类
* 测试调用demo，传入不同元素类型的Array

```scala
package org.duo.oop

object _04GenericDemo {

  class Person

  class Student extends Person

  def demo[T <: Person](a: Array[T]) = println(a)

  def main(args: Array[String]): Unit = {
    demo(Array(new Person))
    demo(Array(new Student))
    // 编译出错，必须是Person的子类
    // demo(Array("hadoop"))
  }
}
```

### 下界定义

上界是要求必须是某个类的子类，或者必须从某个类继承，而下界是必须是某个类的父类（或本身）

语法格式

```scala
[T >: 类型]
```

如果类既有上界、又有下界。下界写在前面，上界写在后面

示例

* 定义一个Person类
* 定义一个Policeman类，继承Person类
* 定义一个Superman类，继承Policeman类
* 定义一个demo泛型方法，该方法接收一个Array参数，
* 限定demo方法的Array元素类型只能是Person、Policeman
* 测试调用demo，传入不同元素类型的Array

```scala
package org.duo.oop

object _05GenericDemo {

  class Person

  class Policeman extends Person

  class Superman extends Policeman

  def demo[T >: Policeman](array: Array[T]) = println(array)

  def main(args: Array[String]): Unit = {
    demo(Array(new Person))
    demo(Array(new Policeman))
    // 编译出错：Superman是Policeman的子类
    // demo(Array(new Superman))
  }
}
```

## 协变、逆变、非变

### 非变

默认泛型类是非变的，即：类型B是A的子类型，Pair[A]和Pair[B]没有任何从属关系，这跟Java是一样的。

```scala
package org.duo.oop

object _06GenericDemo {

  class Pair[T]

  def main(args: Array[String]): Unit = {
    val p1 = Pair("hello")
    // 编译报错，无法将p1转换为p2
    val p2: Pair[AnyRef] = p1

    println(p2)
  }
}
```

### 协变

语法格式

```scala
class Pair[+T]
```

* 类型B是A的子类型，Pair[B]可以认为是Pair[A]的子类型
* 参数化类型的方向和类型的方向是一致的。

### 逆变

语法格式

```scala
class Pair[-T]
```

* 类型B是A的子类型，Pair[A]反过来可以认为是Pair[B]的子类型
* 参数化类型的方向和类型的方向是相反的

示例

* 定义一个Super类、以及一个Sub类继承自Super类
* 使用协变、逆变、非变分别定义三个泛型类
* 分别创建泛型类来演示协变、逆变、非变

```scala
package org.duo.oop

object _07GenericDemo {
  
  class Super

  class Sub extends Super

  class Temp1[T]

  class Temp2[+T]

  class Temp3[-T]

  def main(args: Array[String]): Unit = {
      
    val a: Temp1[Sub] = new Temp1[Sub]
    // 编译报错
    // 非变
    //val b:Temp1[Super] = a

    // 协变
    val c: Temp2[Sub] = new Temp2[Sub]
    val d: Temp2[Super] = c

    // 逆变
    val e: Temp3[Super] = new Temp3[Super]
    val f: Temp3[Sub] = e
  }
}
```

# Actor

scala的Actor并发编程模型可以用来开发比Java线程效率更高的并发程序。scala在2.11.x版本中加入了Akka并发编程框架，老版本已经废弃。Actor的编程模型和Akka很像，这里学习Actor的目的是为学习Akka做准备。

## Java并发编程的问题

在Java并发编程中，每个对象都有一个逻辑监视器（monitor），可以用来控制对象的多线程访问。我们添加sychronized关键字来标记，需要进行同步加锁访问。这样，通过加锁的机制来确保同一时间只有一个线程访问共享数据。但这种方式存在资源争夺、以及死锁问题，程序越大问题越麻烦。

## Actor并发编程模型

Actor并发编程模型，是scala提供给程序员的一种与Java并发编程完全不一样的并发编程模型，是一种基于事件模型的并发机制。Actor并发编程模型是一种不共享数据，依赖消息传递的一种并发编程模式，有效避免资源争夺、死锁等情况。

## 创建Actor

创建Actor的方式和Java中创建线程很类似，也是通过继承来创建。

使用方式

1. 定义class或object继承Actor特质
2. 重写act方法
3. 调用Actor的start方法执行Actor

类似于Java线程，这里的每个Actor是并行执行的

示例

创建两个Actor，一个Actor打印1-10，另一个Actor打印11-20

* 使用class继承Actor创建（如果需要在程序中创建多个相同的Actor）
* 使用object继承Actor创建（如果在程序中只创建一个Actor）

使用class继承Actor创建

```scala
package org.duo.oop

import scala.actors.Actor

object _01ActorDemo {

  class Actor1 extends Actor {
    override def act(): Unit = (1 to 10).foreach(println(_))
  }

  class Actor2 extends Actor {
    override def act(): Unit = (11 to 20).foreach(println(_))
  }

  def main(args: Array[String]): Unit = {
    new Actor1().start()
    new Actor2().start()
  }
}
```

使用object继承Actor创建

```scala
package org.duo.oop

import scala.actors.Actor

object _02ActorDemo {

  object Actor1 extends Actor {
    override def act(): Unit =
      for (i <- 1 to 10) {
        println(i)
      }
  }

  object Actor2 extends Actor {
    override def act(): Unit =
      for (i <- 11 to 20) {
        println(i)
      }
  }

  def main(args: Array[String]): Unit = {
    Actor1.start()
    Actor2.start()
  }
}
```

## 发送消息/接收消息

Actor是基于事件（消息）的并发编程模型，那么Actor是如何发送消息和接收消息的呢？

### 发送消息

可以使用三种方式来发送消息：

| 标识 | 含义                              |
| ---- | --------------------------------- |
| ！   | 发送异步消息，没有返回值          |
| !?   | 发送同步消息，等待返回值          |
| !!   | 发送异步消息，返回值是Future[Any] |

### 接收消息

Actor中使用receive方法来接收消息，需要给receive方法传入一个偏函数

```scala
{
    case 变量名1:消息类型1 => 业务处理1,
    case 变量名2:消息类型2 => 业务处理2,
    ...
}
```

receive方法只接收一次消息，接收完后继续执行act方法

示例

* 创建两个Actor（ActorSender、ActorReceiver）
* ActorSender发送一个异步字符串消息给ActorReceiver
* ActorReceive接收到该消息后，打印出来

```scala
package org.duo.oop

import scala.actors.Actor

//创建两个Actor（ActorSender、ActorReceiver）
//ActorSender发送一个异步字符串消息给ActorReceiver
//ActorReceive接收到该消息后，打印出来
object _03ActorDemo {
  // 1. 创建两个Actor（ActorSender、ActorReceiver）
  object ActorSender extends Actor {
    override def act(): Unit = {
      // 发送一个字符串消息给ActorReceiver
      // 使用!以异步的方式发送字符串消息
      ActorReceiver ! "你好!"
    }
  }

  object ActorReceiver extends Actor {
    override def act(): Unit = {
      // 3. 接收消息
      receive {
        case msg:String => println(msg)
      }
    }
  }

  // 2. 启动Actor,发送异步消息
  def main(args: Array[String]): Unit = {
    ActorSender.start()
    ActorReceiver.start()
  }
}
```

## 持续接收消息

```scala
package org.duo.oop

import java.util.concurrent.TimeUnit

import scala.actors.Actor

object _04ActorDemo {
  // 1. 创建两个Actor
  // ActorSender--每隔一秒发送一个消息
  // ActorReceiver--不停地接收消息
  object ActorSender extends Actor {
    override def act(): Unit = {
      while(true) {
        // 发送异步消息
        ActorReceiver ! "你好"
        TimeUnit.SECONDS.sleep(1)
      }
    }
  }

  object ActorReceiver extends Actor {
    override def act(): Unit = {
      while(true) {
        receive{
          case msg:String => println(msg)
        }
      }
    }
  }

  // 2. 启动Actor测试
  def main(args: Array[String]): Unit = {
    ActorSender.start()
    ActorReceiver.start()
  }
}
```

上述代码，使用while循环来不断接收消息。

* 如果当前Actor没有接收到消息，线程就会处于阻塞状态
* 如果有很多的Actor，就有可能会导致很多线程都是处于阻塞状态
* 每次有新的消息来时，重新创建线程来处理
* 频繁的线程创建、销毁和切换，会影响运行效率

在scala中，可以使用loop + react来复用线程。比while + receive更高效

```scala
package org.duo.oop

import java.util.concurrent.TimeUnit

import scala.actors.Actor

object _05ActorDemo {
  // 1. 创建两个Actor
  // ActorSender--每隔一秒发送一个消息
  // ActorReceiver--不停地接收消息
  object ActorSender extends Actor {
    override def act(): Unit = {
      while(true) {
        // 发送异步消息
        ActorReceiver ! "你好"
        TimeUnit.SECONDS.sleep(1)
      }
    }
  }

  object ActorReceiver extends Actor {
    override def act(): Unit = {
      // 使用loop+react来复用线程，提高执行效率
      loop{
        react {
          case msg:String => println(msg)
        }
      }
    }
  }

  // 2. 启动Actor测试
  def main(args: Array[String]): Unit = {
    ActorSender.start()
    ActorReceiver.start()
  }
}
```

## 自定义消息

示例一

示例说明

* 创建一个MsgActor，并向它发送一个同步消息，该消息包含两个字段（id、message）
* MsgActor回复一个消息，该消息包含两个字段（message、name）
* 打印回复消息

* 使用!?来发送同步消息
* 在Actor的act方法中，可以使用sender获取发送者的Actor引用

```scala
package org.duo.oop

import scala.actors.Actor

object _06ActorDemo {

  // 定义样例类封装消息
  case class ReplyMessage(message: String, name: String)

  // 1. 创建Actor，接收消息，回复消息
  object MsgActor extends Actor {
    override def act(): Unit = {
      loop {
        react {
          case Message(id, message) =>
            println(s"MsgActor接收到消息：${id}, ${message}")
            // 回复消息
            sender ! ReplyMessage("我不好", "张三")
        }
      }
    }
  }

  // 定义一个自定义消息
  case class Message(id: Int, message: String)

  // 2. 启动Actor，发送消息
  def main(args: Array[String]): Unit = {

    MsgActor.start()

    // 发送同步自定义消息
    val reply: Any = MsgActor !? Message(1, "你好")

    // 3. 打印回复消息
    if (reply.isInstanceOf[ReplyMessage]) {
      println(reply.asInstanceOf[ReplyMessage])
    }
  }
}
```

示例二

示例说明

* 创建一个MsgActor，并向它发送一个异步无返回消息，该消息包含两个字段（message, company）
* 使用!发送异步无返回消息

```scala
package org.duo.oop

import scala.actors.Actor

object _07ActorDemo {

  // 1. 创建一个Actor，打印接收消息
  object MsgActor extends Actor {

    override def act(): Unit = {
      loop {
        react {
          case Message(message, company) => println(s"MsgActor接收到消息：${message}, ${company}")
        }
      }
    }
  }

  // 创建样例类封装消息
  case class Message(message: String, company: String)

  // 2. 启动Actor，给Actor发送异步无返回消息
  def main(args: Array[String]): Unit = {

    MsgActor.start()
    // 发送异步无返回消息
    MsgActor ! Message("您好，大爷，快交话费！", "中国联通")
  }
}
```

示例三

示例说明

* 创建一个MsgActor，并向它发送一个异步有返回消息，该消息包含两个字段（id、message）
* MsgActor回复一个消息，该消息包含两个字段（message、name）
* 打印回复消息
* 使用!!发送异步有返回消息
* 发送后，返回类型为Future[Any]的对象
* Future表示异步返回数据的封装，虽获取到Future的返回值，但不一定有值，可能在将来某一时刻才会返回消息
* Future的isSet()可检查是否已经收到返回消息，apply()方法可获取返回数据

```scala
package org.duo.oop

import scala.actors.{Actor, Future}

object _08ActorDemo {

  // 定义样例类封装数据
  case class Message(id: Int, message: String)

  case class ReplyMessage(message: String, name: String)

  // 1. 创建Actor，接收消息，回复消息
  object MsgActor extends Actor {

    override def act(): Unit = {
      loop {
        react {
          case Message(id, message) =>
            println(s"MsgActor接收到消息：${id}, ${message}")
            // 回复一个Reply消息
            sender ! ReplyMessage("我不好", "韩梅梅")
        }
      }
    }
  }

  // 2. 启动Actor，发送异步有返回消息
  def main(args: Array[String]): Unit = {

    MsgActor.start()

    // future表示将来会返回一个数据
    val future: Future[Any] = MsgActor !! Message(1, "你好")

    // 3. 获取打印返回消息
    // 3.1 提前通过一个循环等到future中有数据，再执行
    // 3.2 调用future.isSet方法就可以判断，数据是否已经被接收到
    while (!future.isSet) {}

    // 3.3 使用future的apply方法获取数据
    val replyMessagse = future.apply().asInstanceOf[ReplyMessage]
    println(s"接收到回复消息：${replyMessagse.message}, ${replyMessagse.name}")
  }
}
```

# 高阶函数

## 作为值的函数

在scala中，函数就像和数字、字符串一样，可以将函数传递给一个方法。可以对算法进行封装，然后将具体的动作传递给方法，这种特性很有用。

```scala
package org.duo.oop

object _01FuncDemo {

  def main(args: Array[String]): Unit = {

    // 1. 创建函数，将数字转换为小星星
    // func: Int => String表示这个函数的入参和返回结果类型，也就是这个函数的类型，所以是可以省略的：
    // val func = (num:Int) => "*" * num；这种省略的写法实现的功能是完全一致的
    val func: Int => String = (num: Int) => {
      "*" * num
    }

    // 2. 创建列表，执行转换
    val starList = (1 to 10).map(func)

    // 3. 打印测试
    println(starList)

  }
}

```

## 匿名函数

在scala中，可以不需要给函数赋值给变量，没有赋值给变量的函数就是匿名函数

```scala
package org.duo.oop

object _02FuncDemo {

  def main(args: Array[String]): Unit = {

    // 使用匿名函数简化代码编写
    val startList = (1 to 10).map(x => "*" * x)
    println(startList)

    // 因为此处num变量只使用了一次，而且只是进行简单的计算，所以可以省略参数列表，使用_替代参数
    val starList2 = (1 to 10).map("*" * _)
    println(starList2)
  }
}
```

## 柯里化

柯里化（Currying）是指将原先接受多个参数的方法转换为多个只有一个参数的参数列表的过程。

![image][2]

柯里化过程解析

![image][3]

```scala
package org.duo.oop

object _03FuncDemo {


  def func(x: Int)(y: Int) = x + y

  // 1. 定义一个方法（柯里化），计算两个Int类型的值
  def calculate(a: Int, b: Int)(calc: (Int, Int) => Int) = {
    calc(a, b)
  }

  def main(args: Array[String]): Unit = {

    // 调用func(1)其实是返回一个函数:func1=(y:Int)=>1+y
    // func(1)(2)是调用func(1)返回的函数，再传入第二个参数，得到最终的值
    println(func(1)(9));
    // 调用calculate(1, 2)后返回的是函数:func1=(calc: (Int, Int) => Int) => calc(a,b)，而函数:func1的入参又是一个函数(入参为2个整型、返回值为整型)
    // calculate(1, 2)(_ + _)最终返回的是：将第一个参数的值作为第二个参数的入参，计算后得到的结果
    // 因此要得到最终结果传入calculate的第二个参数必须是一个函数
    println(calculate(1, 2)(_ + _))
    println(calculate(1, 2)(_ * _))
    println(calculate(1, 2)(_ - _))
  }
}
```

## 闭包

闭包其实就是一个函数，只不过这个函数的返回值依赖于声明在函数外部的变量。

```scala
package org.duo.oop

object _04FuncDemo {

  def main(args: Array[String]): Unit = {

    val y = 10
    // 定义一个函数，访问函数作用域外部的变量
    val add: Int => Int = (x: Int) => x + y
    println(add(1))
  }
}
```

例如：

```scala
scala> var more = 1
more: Int = 1

scala> val addMore = (x: Int) => x + more
addMore: Int => Int = <function1>

scala> addMore(10)
res0: Int = 11
```

运行时从这个函数字面量创建出来的函数值（对象）被称为闭包。该名称源于“捕获”其自由变量从而“闭合”该函数字面量的动作。没有自由变量的函数字面量，比如`(x: Int) => x + 1`，称为闭合语（这里的语指的是一段源代码）。因此，运行时从这个函数字面量创建出来的函数值严格来说并不是一个闭包，因为`(x: Int) => x + 1`按照目前这个写法已经是闭合的了。而运行时从任何带有自由变量的函数字面量，比如`(x: Int) => x + more`创建的函数，按照定义，要求捕获到它的自由变量more的绑定。相应的函数值结果（包含指向被捕获的more变量的引用）就被称为闭包，因为函数值是通过闭合这个开放语的动作产生的。

如果more在闭包创建以后被改变，闭包能够看到这个改变，例如：

```scala
scala> var more = 1
more: Int = 1

scala> val addMore = (x: Int) => x + more
addMore: Int => Int = <function1>

scala> addMore(10)
res0: Int = 11

scala> more = 9999
more: Int = 9999

scala> addMore(10)
res1: Int = 10009
```

Scala的闭包捕获的是变量本身，而不是变量引用的值。创建的闭包能够看到闭包外对more的修改。反过来也是成立的：闭包对捕获到的变量的修改也能在闭包外被看到

```scala
scala> val someNumbers = List(-11, -10, -5, 0, 5, 10)
someNumbers: List[Int] = List(-11, -10, -5, 0, 5, 10)

scala> var sum = 0
sum: Int = 0

scala> someNumbers.foreach(sum += _)

scala> sum
res3: Int = -11
```

sum这个变量位于函数字面量`sum += _`的外围作用域，这个函数将数字加给sum。虽然运行时是这个闭包对sum进行的修改，最终的结果-11仍然能被闭包外部看到。

如果一个闭包访问了某个随着程序运行会产生多个副本的变量，那么闭包引用的实例是在闭包被创建时活跃的那一个，例如：

```scala
scala> def makeIncreaser(more: Int) = (x: Int) => x + more
makeIncreaser: (more: Int)Int => Int

// 该函数每调用一次，就会创建一个新的闭包。每个闭包都会访问那个在它创建时活跃的变量more
// 当调用makeIncreaser(1)时，一个捕获了more的绑定值为1的闭包就被创建并返回。
scala> val inc1 = makeIncreaser(1)
inc1: Int => Int = <function1>

// 同理，当调用makeIncreaser(9999)时，返回的是一个捕获了more的绑定值9999的闭包。
scala> val inc9999 = makeIncreaser(9999)
inc9999: Int => Int = <function1>

// 当你将这些闭包应用到入参时，其返回结果取决于闭包创建时more的定义
scala> inc1(10)
res4: Int = 11

scala> inc9999(10)
res5: Int = 10009
```

## 隐式转换和隐式参数

隐式转换和隐式参数是scala非常有特色的功能，也是Java等其他编程语言没有的功能。可以很方便地利用隐式转换来丰富现有类的功能。

### 隐式转换

定义

所谓隐式转换，是指以implicit关键字声明的带有单个参数的方法。它是自动被调用的，自动将某种类型转换为另外一种类型。

示例

使用隐式转换，让File具备有read功能——实现将文本中的内容以字符串形式读取出来

1. 创建RichFile类，提供一个read方法，用于将文件内容读取为字符串
2. 定义一个隐式转换方法，将File隐式转换为RichFile对象
3. 创建一个File，导入隐式转换，调用File的read方法

```scala
package org.duo.oop

import java.io.File
import scala.io.Source

object _05ImplicitDemo {

  // 1. 创建扩展类，实现对File的扩展，读取文件内容
  class RichFile(val file: File) {
    def read() = {
      // 将文件内容读取为字符串
      Source.fromFile(file).mkString
    }
  }

  // 2. 创建隐式转换
  object ImplicitDemo {
    // 将File对象转换为RichFile对象
    implicit def fileToRichFile(file: File) = new RichFile(file)
  }

  // 3. 导入隐式转换，测试读取文件内容
  def main(args: Array[String]): Unit = {

    val file = new File("./data/1.txt")
    import ImplicitDemo.fileToRichFile
    // 调用隐式转换的方法
    println(file.read())
  }
}
```

隐式转换的时机

* 当对象调用类中不存在的方法或者成员时，编译器会自动将对象进行隐式转换
* 当方法中的参数的类型与目标类型不一致时

### 自动导入隐式转换方法

在scala中，如果在当前作用域中有隐式转换方法，会自动导入隐式转换。

示例：将隐式转换方法定义在main所在的object中

```scala
package org.duo.oop

import java.io.File
import scala.io.Source

object _06ImplicitDemo {

  // 1. 创建扩展类，实现对File的扩展，读取文件内容
  class RichFile(val file: File) {

    def read() = {
      // 将文件内容读取为字符串
      Source.fromFile(file).mkString
    }
  }

  // 3. 导入隐式转换，测试读取文件内容
  def main(args: Array[String]): Unit = {

    // 2. 创建隐式转换
    // 将File对象转换为RichFile对象
    implicit def fileToRichFile(file: File) = new RichFile(file)

    val file = new File("./data/1.txt")
    // 调用隐式转换的方法
    println(file.read())
  }
}
```

### 隐式参数

方法可以带有一个标记为implicit的参数列表。这种情况，编译器会查找缺省值，提供给该方法。

定义

1. 在方法后面添加一个参数列表，参数使用implicit修饰
2. 在object中定义implicit修饰的隐式值
3. 调用方法，可以不传入implicit修饰的参数列表，编译器会自动查找缺省值
4. 和隐式转换一样，可以使用import手动导入隐式参数
5. 如果在当前作用域定义了隐式值，会自动进行导入

示例

* 定义一个方法，可将传入的值，使用一个分隔符前缀、后缀包括起来
* 使用隐式参数定义分隔符
* 调用该方法，并打印测试

```scala
package org.duo.oop

object _07ImplicitDemo {

  // 1. 定义一个方法，这个方法有一个隐式参数
  def quote(what: String)(implicit delimeters: (String, String)) = {
    delimeters._1 + what + delimeters._2
  }

  // 2. 定义一个隐式参数
  object ImplicitDemo {
    implicit val delimeterParam = ("<<", ">>")
  }

  // 3. 调用方法执行测试
  def main(args: Array[String]): Unit = {

    import ImplicitDemo._
    println(quote("你好"))
  }
}
```

# Akka

## 介绍

Akka是一个用于构建高并发、分布式和可扩展的基于事件驱动的应用的工具包。Akka是使用scala开发的库，同时可以使用scala和Java语言来开发基于Akka的应用程序。

## 特性

- 提供基于异步非阻塞、高性能的事件驱动编程模型
- 内置容错机制，允许Actor在出错时进行恢复或者重置操作
- 超级轻量级的事件处理（每GB堆内存几百万Actor）
- 使用Akka可以在单机上构建高并发程序，也可以在网络中构建分布式程序。

## 通信过程

以下图片说明了Akka Actor的并发编程模型的基本流程：

1. 学生创建一个ActorSystem
2. 通过ActorSystem来创建一个ActorRef（老师的引用），并将消息发送给ActorRef
3. ActorRef将消息发送给Message Dispatcher（消息分发器）
4. Message Dispatcher将消息按照顺序保存到目标Actor的MailBox中
5. Message Dispatcher将MailBox放到一个线程中
6. MailBox按照顺序取出消息，最终将它递给TeacherActor接受的方法中

![image][4]

## 创建Actor

使用Akka需要导入Akka库，我们这里使用Maven来管理项目

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.duo</groupId>
    <artifactId>akka-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <encoding>UTF-8</encoding>
        <scala.version>2.11.8</scala.version>
        <scala.compat.version>2.11</scala.compat.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
        </dependency>

        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-actor_2.11</artifactId>
            <version>2.3.14</version>
        </dependency>

        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-remote_2.11</artifactId>
            <version>2.3.14</version>
        </dependency>

    </dependencies>

    <build>
        <sourceDirectory>src/main/scala</sourceDirectory>
        <testSourceDirectory>src/test/scala</testSourceDirectory>
        <plugins>
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>3.2.2</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                        <configuration>
                            <args>
                                <arg>-dependencyfile</arg>
                                <arg>${project.build.directory}/.scala_dependencies</arg>
                            </args>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>reference.conf</resource>
                                </transformer>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass></mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

```scala
package org.duo.akka

// 提交任务消息
case class SubmitTaskMessage(message:String)

// 提交任务成功消息
case class SuccessSubmitTaskMessage(message:String)

```

实现Actor类

- 继承Actor（注意：要导入akka.actor包下的Actor）
- 实现receive方法，receive方法中直接处理消息即可，不需要添加loop和react方法调用。Akka会自动调用receive来接收消息

- 【可选】还可以实现preStart()方法，该方法在Actor对象构建后执行，在Actor声明周期中仅执行一次

```scala
package org.duo.akka

import akka.actor.Actor

object ReceiverActor extends Actor {

  override def receive: Receive = {

    case SubmitTaskMessage(message) => {
      println(s"ReceiverActor:接收到任务提交消息 ${message}")
      // 回复一个任务提交成功消息给SenderActor
      sender ! SuccessSubmitTaskMessage("成功提交任务")
    }
  }
}
```

```scala
package org.duo.akka

import akka.actor.Actor

// 实现AkkaActor
object SenderActor extends Actor {

  // 在Actor并发编程模型，需要实现act，想要持续接收消息
  // loop + react
  // 但在akka编程模型中，直接在receive方法中编写偏函数直接处理消息就可以了
  override def receive: Receive = {

    case "start" => {

      println("SenderActor：接收到start消息")

      // 发送一个SubmitTaskMessage消息给ReceiverActor
      // “akka://” + actorSystem的名字 + “/user/” + actor的名字
      // actorSystem和actor的名字是在Entrance中定义的
      val receiverActor = context.actorSelection("akka://actorSystem/user/receiverActor")
      // 发送消息
      receiverActor ! SubmitTaskMessage("提交任务")
    }
    case SuccessSubmitTaskMessage(message) =>
      println(s"SenderActor：接收到任务提交成功消息 ${message}")
  }
}
```

ActorSystem

在Akka中，ActorSystem是一个重量级的结构，它需要分配多个线程，所以在实际应用中，ActorSystem通常是一个单例对象，可以使用这个ActorSystem创建很多Actor。它负责创建和监督actor

Actor中获取ActorSystem

直接使用context.system就可以获取到管理该Actor的ActorSystem的引用

加载Akka Actor

1. 要创建Akka的Actor，必须要先获取创建一个ActorSystem。需要给ActorSystem指定一个名称，并可以去加载一些配置项（后面会使用到）
2. 调用ActorSystem.actorOf(Props(Actor对象), "Actor名字")来加载Actor

```scala
package org.duo.akka

import akka.actor.{ActorSystem, Props}
import com.typesafe.config.ConfigFactory

object Entrance {

  def main(args: Array[String]): Unit = {

    // 1. 实现一个Actor trait
    // 2. 创建ActorSystem
    val actorSystem = ActorSystem("actorSystem", ConfigFactory.load())

    // 3. 加载Actor
    val senderActor = actorSystem.actorOf(Props(SenderActor), "senderActor")
    val receiverActor = actorSystem.actorOf(Props(ReceiverActor), "receiverActor")

    // 在main方法中，发送一个start字符串消息给SenderActor
    senderActor ! "start"
  }
}
```

Actor Path

每一个Actor都有一个Path，就像使用Spring MVC编写一个Controller/Handler一样，这个路径可以被外部引用。路径的格式如下：

| Actor类型 | 路径                                         | 示例                                         |
| --------- | -------------------------------------------- | -------------------------------------------- |
| 本地Actor | akka://actorSystem名称/user/Actor名称        | akka://SimpleAkkaDemo/user/senderActor       |
| 远程Actor | akka.tcp://my-sys@ip地址:port/user/Actor名称 | akka.tcp://192.168.10.17:5678/user/service-b |

## 定时任务

Akka中，提供一个scheduler对象来实现定时调度功能。使用ActorSystem.scheduler.schedule方法，可以启动一个定时任务。

schedule方法针对scala提供两种使用形式：

第一种：发送消息

```scala
def schedule(
    initialDelay: FiniteDuration,		// 延迟多久后启动定时任务
    interval: FiniteDuration,			// 每隔多久执行一次
    receiver: ActorRef,					// 给哪个Actor发送消息
    message: Any)						// 要发送的消息
(implicit executor: ExecutionContext)	// 隐式参数：需要手动导入
```

```scala
package org.duo.akka.scheduler

import akka.actor.{Actor, ActorSystem, Props}
import com.typesafe.config.ConfigFactory

object _01SchedulerDemo {

  // 1. 创建一个Actor，接收打印消息
  object ReceiveActor extends Actor {
    override def receive: Receive = {
      case x => println(x)
    }
  }

  // 2. 构建ActorSystem，加载Actor
  def main(args: Array[String]): Unit = {

    val actorSystem = ActorSystem("actorSystem", ConfigFactory.load())
    val receiveActor = actorSystem.actorOf(Props(ReceiveActor), "receiveActor")

    // 导入隐式转换
    import scala.concurrent.duration._
    // 导入隐式参数
    import actorSystem.dispatcher

    // 3. 定时发送消息给Actor
    // 3.1 延迟多久启动定时任务
    // 3.2 定时任务的周期
    // 3.3 指定发送消息给哪个Actor
    // 3.4 发送的消息是什么
    actorSystem.scheduler.schedule(0 seconds,
      1 seconds,
      receiveActor,
      "hello"
    )
  }
}
```

第二种：自定义实现

```scala
def schedule(
    initialDelay: FiniteDuration,			// 延迟多久后启动定时任务
    interval: FiniteDuration				// 每隔多久执行一次
)(f: ⇒ Unit)								// 定期要执行的函数，可以将逻辑写在这里
(implicit executor: ExecutionContext)		// 隐式参数：需要手动导入
```

```scala
package org.duo.akka.scheduler

import akka.actor.{Actor, ActorSystem, Props}
import com.typesafe.config.ConfigFactory

object _02SchedulerDemo {

  // 1. 创建Actor，接收打印消息
  object ReceiveActor extends Actor {
    override def receive: Receive = {
      case x => println(x)
    }
  }

  // 2. 构建ActorSystem，加载Actor
  def main(args: Array[String]): Unit = {

    val actorSystem = ActorSystem("actorSystem", ConfigFactory.load())
    val receiveActor = actorSystem.actorOf(Props(ReceiveActor), "receiveActor")

    // 导入隐式转换
    import scala.concurrent.duration._
    // 导入隐式参数
    import actorSystem.dispatcher

    // 3. 定时发送消息（自定义方式）
    actorSystem.scheduler.schedule(0 seconds, 1 seconds) {
      // 业务逻辑
      receiveActor ! "hello"
    }
  }
}
```

注意：

1. 需要导入隐式转换import scala.concurrent.duration._才能调用0 seconds方法
2. 需要导入隐式参数import actorSystem.dispatcher才能启动定时任务

## 进程间通信

基于Akka实现在两个进程间发送、接收消息。Worker启动后去连接Master，并发送消息，Master接收到消息后，再回复Worker消息。

### akka-worker

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.duo</groupId>
    <artifactId>akka-worker</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <encoding>UTF-8</encoding>
        <scala.version>2.11.8</scala.version>
        <scala.compat.version>2.11</scala.compat.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
        </dependency>

        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-actor_2.11</artifactId>
            <version>2.3.14</version>
        </dependency>

        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-remote_2.11</artifactId>
            <version>2.3.14</version>
        </dependency>

    </dependencies>

    <build>
        <sourceDirectory>src/main/scala</sourceDirectory>
        <testSourceDirectory>src/test/scala</testSourceDirectory>
        <plugins>
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>3.2.2</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                        <configuration>
                            <args>
                                <arg>-dependencyfile</arg>
                                <arg>${project.build.directory}/.scala_dependencies</arg>
                            </args>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>reference.conf</resource>
                                </transformer>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass></mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

application.conf

```properties
akka.actor.provider = "akka.remote.RemoteActorRefProvider"
akka.remote.netty.tcp.hostname = "127.0.0.1"
akka.remote.netty.tcp.port = "9999"
```

WorkerActor.scala

```scala
package org.duo.akka

import akka.actor.Actor

object WorkerActor extends Actor {

  override def receive: Receive = {

    case "setup" => {

      println("WorkerActor：接收到消息setup")
      // 发送消息给Master
      // 1. 获取到MasterActor的引用
      // Master的引用路径：akka.tcp://actorSystem@127.0.0.1:8888/user/masterActor
      val masterActor = context.actorSelection("akka.tcp://actorSystem@127.0.0.1:8888/user/masterActor")

      // 2. 再发送消息给MasterActor
      masterActor ! "connect"
    }
    case "success" => {

      println("WorkerActor：收到success消息")
    }
  }
}
```

Entrance.java

```java
package org.duo.akka

import akka.actor.{ActorSystem, Props}
import com.typesafe.config.ConfigFactory

object Entrance {

  def main(args: Array[String]): Unit = {

    // 1. 创建一个ActorSystem
    val actorSystem = ActorSystem("actorSystem", ConfigFactory.load())

    // 2. 加载Actor
    val workerActor = actorSystem.actorOf(Props(WorkerActor), "workerActor")

    // 3. 发送消息给Actor
    workerActor ! "setup"
  }
}
```

### akka-master

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.duo</groupId>
    <artifactId>akka-master</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <encoding>UTF-8</encoding>
        <scala.version>2.11.8</scala.version>
        <scala.compat.version>2.11</scala.compat.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
        </dependency>

        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-actor_2.11</artifactId>
            <version>2.3.14</version>
        </dependency>

        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-remote_2.11</artifactId>
            <version>2.3.14</version>
        </dependency>

    </dependencies>

    <build>
        <sourceDirectory>src/main/scala</sourceDirectory>
        <testSourceDirectory>src/test/scala</testSourceDirectory>
        <plugins>
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>3.2.2</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                        <configuration>
                            <args>
                                <arg>-dependencyfile</arg>
                                <arg>${project.build.directory}/.scala_dependencies</arg>
                            </args>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>reference.conf</resource>
                                </transformer>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass></mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

application.conf

```properties
akka.actor.provider = "akka.remote.RemoteActorRefProvider"
akka.remote.netty.tcp.hostname = "127.0.0.1"
akka.remote.netty.tcp.port = "8888"
```

Entrance.java

```java
package org.duo.akka

import akka.actor.{ActorSystem, Props}
import com.typesafe.config.ConfigFactory

object Entrance {

  def main(args: Array[String]): Unit = {

    // 1. 构建ActorSystem
    val actorSystem = ActorSystem("actorSystem", ConfigFactory.load())

    // 2. 加载Actor
    val masterActor = actorSystem.actorOf(Props(MasterActor), "masterActor")
  }
}
```

MasterActor.java

```java
package org.duo.akka

import akka.actor.Actor

object MasterActor extends Actor {

  override def receive: Receive = {

    case "connect" => {
      println("MasterActor：接收到connect消息")
      // 获取发送者Actor的引用
      sender ! "success"
    }
  }
}
```





















[1]:data:image/png;base64,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ALS8zqIHALA57d69O6VSKZOTk/nFL36RkRGPeQUAgOtxBQwAt6xUKuUb3/hG3n777YyPj2dtba3oSQAA0JIEGABuS1dXVw4dOpT7778/zz33XJaXl4ueBAAALUeAAWBdjIyM5MCBAxkfH8/8/HzRcwAAoKU4AwaAdTMwMJD77rsvExMTeeuttzI8PFz0JAAAaAmugAFgXfX29uapp57Kz372s0xNTaXRaBQ9CQAACifAALDuuru7MzY2lq6urhw+fDgrKytFTwIAgEIJMAA0RWdnZ0ZHR7Nnz548//zzWVhYKHoSAAAURoABoKkGBwdTrVZTr9dz4sSJoucAAEAhBBgAmq6/vz9PPfVUTp48mSNHjjgXBgCAtiPAALAhenp68uSTT2ZtbS3j4+NZXV0tehIAAGwYAQaADdPZ2ZmDBw/mC1/4Qp5//vksLS0VPQkAADaEAAPAhhseHs7o6GgmJiZy5syZoucAAEDTdRY9AID2tHv37pRKpUxMTOQXv/hFRkZGip4EAABN4woYAApTKpXy9NNP56233sr4+HjW1taKngQAAE0hwABQqK6uroyNjeX+++/P3//932d5ebnoSQAAsO4EGABawsjISIaGhjI+Pp75+fmi5wAAwLpyBgwALWNgYCA9PT2p1+t56623Mjw8XPQkAABYF66AAaCllMvlPPXUU/nZz36WqampNBqNoicBAMBt60hy8eLFi0XvAICPaDQaOXr0aM6dO5dqtZqenp6iJwEAwC3p6OgQYABobXNzczl27Fiq1Wr6+/uLngNtY3l5OaVSqegZALAlCDAAbAoLCwup1+s5cOBABgcHi54DW9bCwkL+/d//PfPz89m7d28effTRoicBwJYgwACwaaysrKRer6evry+PPfZYOjudIw/rYWlpKadPn85rr72WlZWVDz7+rW99yxUwALBOBBgANpVGo5Gpqamsrq6mWq2mu7u76Emwab3++us5evRolpeXr/pcqVTKt771rQJWAcDW1NHR4SlIAGwenZ2dqVar+cIXvpDDhw9naWmp6EmwafX391/3cOuHH354g9cAwNYnwACw6QwPD+exxx7LxMREzpw5U/Qc2JQ6OztTq9Wyc+fOqz730EMPFbAIALY2N9ADsCnt3r07PT09mZyczC9+8YuMjIwUPQk2nc7OznR3d2fHjh157733kiQ9PT3p7e0t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
[2]:data:image/png;base64,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
[3]:data:image/png;base64,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ZvXS1+17K9zmQtlRYftadKzLeU4fYK7S50WFi4/6pSfUOgdHFTz5qN8cSrZvkIC0q12GodkVC/GqVgfIe7/ZKedHtybDVO2o48PaqxMOSTBSvWaArMvr78TKhdnDD/cdkblUc2e4n9/t747uIatERESjwMeuXr161deStz8F/PrHQ7w5IywaDz42X0tLL86E8dIQU3NSbrsptyu0bld21T2PVtztPm/K+PpESobrBN2WWFJ6e9QVdzeRDrny72dY33DIdwjUcKlrliFPQYJhrt9noRmHd/qlytkHraR230Zg69PmEKNuKmz3/tG43Oz13jnOw+eczrPGSHY4ammz32ZVYdEp4La0AdM+4/1drXheQouqeElERERDxyPTMBwREREREdHY4JFpPj6Mm0JERERERDRqMRwRERERERGB4YiIiIiIiAgAwxEREREREREAhiMiIiIiIiIADEdEREREREQAGI6IiIiIiIgAMBwREREREREBYDgiIiIiIiICwHBEREREREQEgOGIiIiIiIgIAMMRERERERERAIYjIiIiIiIiAAxHREREREREABiOiIiIiIiIADAcERERERERAWA4IiIiIiIiAsBwREREREREBCBIOJpSDPT1D+GmEBERERERDZELF4FJE10X8R+O/urTwB//Pd9NIiIiIiIiGn7nLwKfLnJdJFjPUe/5fDeJiIiIiIho+P2xHwhNdl3EfzgKh4BuLd9NIiIiIiIiGn5n/w0oKVQ4WnIH8Mvf5rtJREREREREwy/2HlBV4bqI/3A0b5akLRZlICIiIiKia0nmioyC+5tbXRfzH44+8XHgc5XAweP5bhoREREREdHwOXhcgtG4/+a6WLD7HD1xP/D628B/ZPLZNCIiIiIiouFx5c/AngNAbbXnosHCUTgELL5dAhIREREREdFo13oIuO2vgbLpnosGC0cAsG4F8MYh4Oy5XDaNiIiIiIhoeHzYB+w+AKz7gq/Fg4ejKcVAfQ3wzVfkLrNERERERESjTeYK8MwPJBiFQ75eEjwcAcAX/0YmNG18FfjP/8ppFUREREREREPiP/8LeG4ncMuNwGP3+n5ZbuEIAOpXA5/8BPCNl1mggYiIiIiIRofMFeDZ7cB/XAb+11cCvTT3cPSJjwM/fBa4aRrwP77LOUhERERERDSyPuyTbDL5vwM7v+VZutvqY1evXr2a90b87F+BV94EHrkHWPtA4I0gIiIiIiLK2ZU/S1U6VXwhwFA6o8KEIwDoPQ+0tAFH/69s0LL5QNG4gqyaiIiIiIgoS+aK3OB1zwEp1x2g+IKdwoUj5f/9QRJb/JRMgFpyBzDjs8BfFQOTJwKTJhb07YiIiIiIaAy4cBE4fxH4Yz9w9t+A2HtAtyaF4mqrfd3HyEvhw5Fy5c/Av/4GiCWlVyl9Hvj3DMt/ExERERFRcJMmAp8uAkKTgZLJQFWFBKMCTukZunBERERERER0Dcm9Wh0REREREdF1hOGIiIiIiIgIDEdEREREREQAGI6IiIiIiIgAMBwREREREREBYDgiIiIiIiICAPx/sfG+wRWPCBUAAAAASUVORK5CYII=
[4]:data:image/png;base64,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