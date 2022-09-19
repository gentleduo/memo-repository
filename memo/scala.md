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



















[1]:data:image/png;base64,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ALS8zqIHALA57d69O6VSKZOTk/nFL36RkRGPeQUAgOtxBQwAt6xUKuUb3/hG3n777YyPj2dtba3oSQAA0JIEGABuS1dXVw4dOpT7778/zz33XJaXl4ueBAAALUeAAWBdjIyM5MCBAxkfH8/8/HzRcwAAoKU4AwaAdTMwMJD77rsvExMTeeuttzI8PFz0JAAAaAmugAFgXfX29uapp57Kz372s0xNTaXRaBQ9CQAACifAALDuuru7MzY2lq6urhw+fDgrKytFTwIAgEIJMAA0RWdnZ0ZHR7Nnz548//zzWVhYKHoSAAAURoABoKkGBwdTrVZTr9dz4sSJoucAAEAhBBgAmq6/vz9PPfVUTp48mSNHjjgXBgCAtiPAALAhenp68uSTT2ZtbS3j4+NZXV0tehIAAGwYAQaADdPZ2ZmDBw/mC1/4Qp5//vksLS0VPQkAADaEAAPAhhseHs7o6GgmJiZy5syZoucAAEDTdRY9AID2tHv37pRKpUxMTOQXv/hFRkZGip4EAABN4woYAApTKpXy9NNP56233sr4+HjW1taKngQAAE0hwABQqK6uroyNjeX+++/P3//932d5ebnoSQAAsO4EGABawsjISIaGhjI+Pp75+fmi5wAAwLpyBgwALWNgYCA9PT2p1+t56623Mjw8XPQkAABYF66AAaCllMvlPPXUU/nZz36WqampNBqNoicBAMBt60hy8eLFi0XvAICPaDQaOXr0aM6dO5dqtZqenp6iJwEAwC3p6OgQYABobXNzczl27Fiq1Wr6+/uLngNtY3l5OaVSqegZALAlCDAAbAoLCwup1+s5cOBABgcHi54DW9bCwkL+/d//PfPz89m7d28effTRoicBwJYgwACwaaysrKRer6evry+PPfZYOjudIw/rYWlpKadPn85rr72WlZWVDz7+rW99yxUwALBOBBgANpVGo5Gpqamsrq6mWq2mu7u76Emwab3++us5evRolpeXr/pcqVTKt771rQJWAcDW1NHR4SlIAGwenZ2dqVar+cIXvpDDhw9naWmp6EmwafX391/3cOuHH354g9cAwNYnwACw6QwPD+exxx7LxMREzpw5U/Qc2JQ6OztTq9Wyc+fOqz730EMPFbAIALY2N9ADsCnt3r07PT09mZyczC9+8YuMjIwUPQk2nc7OznR3d2fHjh157733kiQ9PT3p7e0t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