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
  Array(0, ...)  // 可以任意数量，但是以0开头
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

# Option类型

使用Option类型，可以用来有效避免空引用(null)异常。也就是说，将来返回某些数据时，可以返回一个Option类型来替代。

定义

scala中，Option类型来表示可选值。这种类型的数据有两种形式：

- Some(x)：表示实际的值
- None：表示没有值
- 使用getOrElse方法，当值为None是可以指定一个默认值

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
  val regex = """.+@(.+)\..+""".r

  val emlList =
    List("38123845@qq.com", "a1da88123f@gmail.com", "zhansan@163.com", "123afadff.com")

  val emlCmpList = emlList.map {
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

通过模式匹配，可以快速匹配样例类中的成员变量，但不是所有的类都可以进行这样的模式匹配，要支持模式匹配，必须要实现一个提取：样例类自动实现了apply、unapply方法。

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











































[1]:data:image/png;base64,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ALS8zqIHALA57d69O6VSKZOTk/nFL36RkRGPeQUAgOtxBQwAt6xUKuUb3/hG3n777YyPj2dtba3oSQAA0JIEGABuS1dXVw4dOpT7778/zz33XJaXl4ueBAAALUeAAWBdjIyM5MCBAxkfH8/8/HzRcwAAoKU4AwaAdTMwMJD77rsvExMTeeuttzI8PFz0JAAAaAmugAFgXfX29uapp57Kz372s0xNTaXRaBQ9CQAACifAALDuuru7MzY2lq6urhw+fDgrKytFTwIAgEIJMAA0RWdnZ0ZHR7Nnz548//zzWVhYKHoSAAAURoABoKkGBwdTrVZTr9dz4sSJoucAAEAhBBgAmq6/vz9PPfVUTp48mSNHjjgXBgCAtiPAALAhenp68uSTT2ZtbS3j4+NZXV0tehIAAGwYAQaADdPZ2ZmDBw/mC1/4Qp5//vksLS0VPQkAADaEAAPAhhseHs7o6GgmJiZy5syZoucAAEDTdRY9AID2tHv37pRKpUxMTOQXv/hFRkZGip4EAABN4woYAApTKpXy9NNP56233sr4+HjW1taKngQAAE0hwABQqK6uroyNjeX+++/P3//932d5ebnoSQAAsO4EGABawsjISIaGhjI+Pp75+fmi5wAAwLpyBgwALWNgYCA9PT2p1+t56623Mjw8XPQkAABYF66AAaCllMvlPPXUU/nZz36WqampNBqNoicBAMBt60hy8eLFi0XvAICPaDQaOXr0aM6dO5dqtZqenp6iJwEAwC3p6OgQYABobXNzczl27Fiq1Wr6+/uLngNtY3l5OaVSqegZALAlCDAAbAoLCwup1+s5cOBABgcHi54DW9bCwkL+/d//PfPz89m7d28effTRoicBwJYgwACwaaysrKRer6evry+PPfZYOjudIw/rYWlpKadPn85rr72WlZWVDz7+rW99yxUwALBOBBgANpVGo5Gpqamsrq6mWq2mu7u76Emwab3++us5evRolpeXr/pcqVTKt771rQJWAcDW1NHR4SlIAGwenZ2dqVar+cIXvpDDhw9naWmp6EmwafX391/3cOuHH354g9cAwNYnwACw6QwPD+exxx7LxMREzpw5U/Qc2JQ6OztTq9Wyc+fOqz730EMPFbAIALY2N9ADsCnt3r07PT09mZyczC9+8YuMjIwUPQk2nc7OznR3d2fHjh157733kiQ9PT3p7e0t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
[2]:data:image/png;base64,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
[3]:data:image/png;base64,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ZvXS1+17K9zmQtlRYftadKzLeU4fYK7S50WFi4/6pSfUOgdHFTz5qN8cSrZvkIC0q12GodkVC/GqVgfIe7/ZKedHtybDVO2o48PaqxMOSTBSvWaArMvr78TKhdnDD/cdkblUc2e4n9/t747uIatERESjwMeuXr161deStz8F/PrHQ7w5IywaDz42X0tLL86E8dIQU3NSbrsptyu0bld21T2PVtztPm/K+PpESobrBN2WWFJ6e9QVdzeRDrny72dY33DIdwjUcKlrliFPQYJhrt9noRmHd/qlytkHraR230Zg69PmEKNuKmz3/tG43Oz13jnOw+eczrPGSHY4ammz32ZVYdEp4La0AdM+4/1drXheQouqeElERERDxyPTMBwREREREdHY4JFpPj6Mm0JERERERDRqMRwRERERERGB4YiIiIiIiAgAwxEREREREREAhiMiIiIiIiIADEdEREREREQAGI6IiIiIiIgAMBwREREREREBYDgiIiIiIiICwHBEREREREQEgOGIiIiIiIgIAMMRERERERERAIYjIiIiIiIiAAxHREREREREABiOiIiIiIiIADAcERERERERAWA4IiIiIiIiAsBwREREREREBCBIOJpSDPT1D+GmEBERERERDZELF4FJE10X8R+O/urTwB//Pd9NIiIiIiIiGn7nLwKfLnJdJFjPUe/5fDeJiIiIiIho+P2xHwhNdl3EfzgKh4BuLd9NIiIiIiIiGn5n/w0oKVQ4WnIH8Mvf5rtJREREREREwy/2HlBV4bqI/3A0b5akLRZlICIiIiKia0nmioyC+5tbXRfzH44+8XHgc5XAweP5bhoREREREdHwOXhcgtG4/+a6WLD7HD1xP/D628B/ZPLZNCIiIiIiouFx5c/AngNAbbXnosHCUTgELL5dAhIREREREdFo13oIuO2vgbLpnosGC0cAsG4F8MYh4Oy5XDaNiIiIiIhoeHzYB+w+AKz7gq/Fg4ejKcVAfQ3wzVfkLrNERERERESjTeYK8MwPJBiFQ75eEjwcAcAX/0YmNG18FfjP/8ppFUREREREREPiP/8LeG4ncMuNwGP3+n5ZbuEIAOpXA5/8BPCNl1mggYiIiIiIRofMFeDZ7cB/XAb+11cCvTT3cPSJjwM/fBa4aRrwP77LOUhERERERDSyPuyTbDL5vwM7v+VZutvqY1evXr2a90b87F+BV94EHrkHWPtA4I0gIiIiIiLK2ZU/S1U6VXwhwFA6o8KEIwDoPQ+0tAFH/69s0LL5QNG4gqyaiIiIiIgoS+aK3OB1zwEp1x2g+IKdwoUj5f/9QRJb/JRMgFpyBzDjs8BfFQOTJwKTJhb07YiIiIiIaAy4cBE4fxH4Yz9w9t+A2HtAtyaF4mqrfd3HyEvhw5Fy5c/Av/4GiCWlVyl9Hvj3DMt/ExERERFRcJMmAp8uAkKTgZLJQFWFBKMCTukZunBERERERER0Dcm9Wh0REREREdF1hOGIiIiIiIgIDEdEREREREQAGI6IiIiIiIgAMBwREREREREBYDgiIiIiIiICAPx/sfG+wRWPCBUAAAAASUVORK5CYII=