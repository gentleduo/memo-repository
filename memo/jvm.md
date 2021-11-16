# ClassFileFormat

## class文件查看plugins

1. jclasslib Bytecode Viewer

   使用时直接选择 View --> Show Bytecode With jclasslib

2. BinEd-Binary/Hexadecimal Editor

   选中.class文件，右键找到”open in Binary“即可显示16进制编码

## Magic Number

CAFEBABE，占两个字节

## Minor Version

class文件的版本号-小版本(根据JDK的版本)，占两个字节

## Major Version

class文件的版本号-大版本(根据JDK的版本)，占两个字节

## 常量池

### class文件常量池（class constant pool）

class文件中除了包含类的版本、字段、方法、接口等描述信息外，还有一项信息就是常量池(constant pool table)，用于存放编译器生成的各种字面量(Literal)和符号引用(Symbolic References)。 字面量就是我们所说的常量概念，如文本字符串、被声明为final的常量值等。 符号引用是一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可（它与直接引用区分一下，直接引用一般是指向方法区的本地指针，相对偏移量或是一个能间接定位到目标的句柄）。一般包括下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

### 运行时常量池（runtime constant pool）

jvm在执行某个类的时候，必须经过加载、连接、初始化，而连接又包括验证、准备、解析三个阶段。而当类加载到内存中后，jvm就会将class常量池中的内容存放到运行时常量池中，由此可知，运行时常量池也是每个类都有一个。在上面我也说了，class常量池中存的是字面量和符号引用，也就是说他们存的并不是对象的实例，而是对象的符号引用值。而经过解析（resolve）之后，也就是把符号引用替换为直接引用

### Constant_pool_count

常量池计数器，占两个字节，实际常量的个数为Constant_pool_count-1，因为常量池计数器是从1开始计数的，而不是从0开始，例如如果常量池计数器值Constant_pool_count=22，则后面的常量池项(cp_info)的个数就为21。（原因：在指定class文件规范的时候，将第0项常量空出来是特殊考虑的，这样做就是为了满足某些值向常量池的索引值的数据在特定的情况下表达“不引用任何一个常量池项”的意思，这种情况下可以将索引值设置为0来表示）；

### CONSTANT_Utf8_info

1. tag:1 占用一个字节（不同的tag值对应不同的类型）
2. length:UTF-8字符串占用的字节数
3. bytes:长度为length的字符串

### CONSTANT_Integer_info

1. tag:3 
2. bytes:4个字节，Big-Endian（高位在前）存储的int值

### CONSTANT_Float_info

1. tag:4
2. bytes:4个字节Big-Endian的float值

### CONSTANT_Long_info

1. tag:5
2. bytes:8个字节Big-Endian的long值

### CONSTANT_Double_info

1. tag:6
2. bytes:8个字节Big-Endian的double值

### CONSTANT_Class_info

1. tag:7
2. index:2字节指向类的全限定名项的索引

### CONSTANT_String_info

1. tag:8
2. index:2字节指向字符串字面量的索引

### CONSTANT_Fieldref_info

1. tag:9（符号引用：调用其他类；在类加载后会替换成直接引用：即运行时常量池的某个类的引用）
2. index:2字节指向声明字段的类或者接口描述符CONSTANT_Class_info的索引项
3. index:2字节指向字段描述符CONSTANT_NameAndType_info的索引项

### CONSTANT_Methodref_info

1. tag:10（符号引用：调用其他类的方法，在类加载后会替换成直接引用：即运行时常量池的某个类的方法的引用）
2. index:2字节指向声明方法的类或者接口描述符CONSTANT_Class_info的索引项
3. index:2字节指向字段描述符CONSTANT_NameAndType_info的索引项

### CONSTANT_InterfaceMethodref_info

1. tag:11（符号引用：调用其他类的方法，在类加载后会替换成直接引用：即运行时常量池的某个接口的方法的引用）
2. index:2字节指向声明方法的类或者接口描述符CONSTANT_Class_info的索引项
3. index:2字节指向字段描述符CONSTANT_NameAndType_info的索引项

### CONSTANT_NameAndType_info

1. tag:12（符号引用：调用其他类的方法，在类加载后会替换成直接引用：即运行时常量池的某个类的字段或者方法的引用）
2. index:2字节指向该字段或方法名称常量项的索引
3. index:2字节指向该字段或方法描述符常量项的索引

### CONSTANT_MethodHandle_info

1. tag:15
2. reference_kind:1字节1-9之间的一个值，决定了方法句柄的类型。方法句柄类型的值表示方法句柄的字节码行为。
3. reference_index:2字节对常量池的有效索引

### CONSTANT_MethodType_info

1. tag:16
2. descriptor_index:2字节指向Utf8_info结构表示的方法描述符

### CONSTANT_InvokeDynamic_info

1. tag:18
2. bootstrap_method_attr_index:2字节当前Class文件中引导方法表的bootstrap_methods[]数组的有效索引
3. name_and_type_index:2字节指向NameAndType_info表示的方法名和方法描述符

## access_flags

access_flags描述的是当前类（或者接口）的访问修饰符，如public，private等，此外，这里面还存在一个标志位，标志当前的额这个class描述的是类，还是接口。

| 标志名         | 标志值 | 标志含义                  | 针对的对像 |
| -------------- | ------ | ------------------------- | ---------- |
| ACC_PUBLIC     | 0x0001 | public类型                | 所有类型   |
| ACC_FINAL      | 0x0010 | final类型                 | 类         |
| ACC_SUPER      | 0x0020 | 使用新的invokespecial语义 | 类和接口   |
| ACC_INTERFACE  | 0x0200 | 接口类型                  | 接口       |
| ACC_ABSTRACT   | 0x0400 | 抽象类型                  | 类和接口   |
| ACC_SYNTHETIC  | 0x1000 | 该类不由用户代码生成      | 所有类型   |
| ACC_ANNOTATION | 0x2000 | 注解类型                  | 注解       |
| ACC_ENUM       | 0x4000 | 枚举类型                  | 枚举       |

## this_class

类索引，用于确定这个类的全限定名，占2字节

## super_class

父类索引，用于确定这个类父类的全限定名（Java语言不允许多重继承，故父类索引只有一个。除了java.lang.Object类之外所有类都有父类，故除了java.lang.Object类之外，所有类该字段值都不为0），占2字节

## interfaces_count

接口索引计数器，占2字节。如果该类没有实现任何接口，则该计数器值为0，并且后面的接口的索引集合将不占用任何字节，

## interfaces

接口索引集合，一组u2类型数据的集合。用来描述这个类实现了哪些接口，这些被实现的接口将按implements语句（如果该类本身为接口，则为extends语句）后的接口顺序从左至右排列在接口的索引集合中

## fields_count

字段表计数器，即字段表集合中的字段表数据个数。占2字节，其值为0x0001，即只有一个字段表数据，也就是测试类中只包含一个变量（不算方法内部变量）

## fields

字段表集合，一组字段表类型数据的集合。字段表用于描述接口或类中声明的变量，包括类级别（static）和实例级别变量，不包括在方法内部声明的变量。在Java中一般通过如下几项描述一个字段：字段作用域（public、protected、private修饰符）、是类级别变量还是实例级别变量（static修饰符）、可变性（final修饰符）、并发可见性（volatile修饰符）、可序列化与否（transient修饰符）、字段数据类型（基本类型、对象、数组）以及字段名称。在字段表中，变量修饰符使用标志位表示，字段数据类型和字段名称则引用常量池中常量表示。字段表格式如下表所示：

| 类型           | 名称             | 数量             |
| -------------- | ---------------- | ---------------- |
| u2             | access_flags     | 1                |
| u2             | name_index       | 1                |
| u2             | descriptor_index | 1                |
| u2             | attributes_count | 1                |
| attribute_info | attributes       | attributes_count |

字段修饰符放在access_flags中，占2字节

| 标志名称      | 标志值 | 含义                     |
| ------------- | ------ | ------------------------ |
| ACC_PUBLIC    | 0x0001 | 字段是否为public         |
| ACC_PRIVATE   | 0x0002 | 字段是否为private        |
| ACC_PROTECTED | 0x0004 | 字段是否为protected      |
| ACC_STATIC    | 0x0008 | 字段是否为static         |
| ACC_FINAL     | 0x0010 | 字段是否为final          |
| ACC_VOLATILE  | 0x0040 | 字段是否为volatile       |
| ACC_TRANSIENT | 0x0080 | 字段是否为transient      |
| ACC_SYNTHETIC | 0x1000 | 字段是否为编译器自动产生 |
| ACC_ENUM      | 0x4000 | 字段是否为enum           |

descriptor_index

| 标志名称 | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| B        | byte                                                         |
| C        | char                                                         |
| D        | double                                                       |
| F        | float                                                        |
| I        | int                                                          |
| J        | long                                                         |
| S        | short                                                        |
| Z        | boolean                                                      |
| V        | void                                                         |
| L        | L - Object例如Lcom/duo/jvm/Test                              |
| [        | 数组[ 一维数组[B[Ljava/lang/String 多维数组[[C [[[Ljava/lang/String |

## methods_count

方法表计数器，即方法表集合中的方法表数据个数。占2字节。

## methods

方法表集合，一组方法表类型数据的集合。方法表结构和字段表结构一样：

| 类型           | 名称             | 数量             |
| -------------- | ---------------- | ---------------- |
| u2             | access_flags     | 1                |
| u2             | name_index       | 1                |
| u2             | descriptor_index | 1                |
| u2             | attributes_count | 1                |
| attribute_info | attributes       | attributes_count |

数据项的含义非常相似，仅在访问标志位和属性表集合中的可选项上有略微不同。由于ACC_VOLATILE标志和ACC_TRANSIENT标志不能修饰方法，所以access_flags中不包含这两项，同时增加ACC_SYNCHRONIZED标志、ACC_NATIVE标志、ACC_STRICTFP标志和ACC_ABSTRACT标志

| 标志名称         | 标志值 | 含义                             |
| ---------------- | ------ | -------------------------------- |
| ACC_PUBLIC       | 0x0001 | 方法是否为public                 |
| ACC_PRIVATE      | 0x0002 | 方法是否为private                |
| ACC_PROTECTED    | 0x0004 | 方法是否为protected              |
| ACC_STATIC       | 0x0008 | 方法是否为static                 |
| ACC_FINAL        | 0x0010 | 方法是否为final                  |
| ACC_SYNCHRONIZED | 0x0020 | 方法是否为synchronized           |
| ACC_BRIDGE       | 0x0040 | 方法是否是由编译器产生的桥接方法 |
| ACC_VARARGS      | 0x0080 | 方法是否接受不定参数             |
| ACC_NATIVE       | 0x0100 | 方法是否为native                 |
| ACC_ABSTRACT     | 0x0400 | 方法是否为abstract               |
| ACC_STRICTFP     | 0x0800 | 方法是否为strictfp               |
| ACC_SYNTHETIC    | 0x1000 | 方法是否为编译器自动产生         |

descriptor_index

1. 先参数列表(放在小括号内部)，后返回值

   - void m() -> ()V

   - String toString() -> ()Ljava/lang/string

   - long pos(int[] arr1,int arr2,long length) -> ([llJ)J

Code

| 类型 | 名称                   | 数量                                                         |
| ---- | ---------------------- | ------------------------------------------------------------ |
| u2   | attribute_name_index   | attribute_name_index指向常量池中的一个CONSTANT_Utf8_info，这个CONSTANT_Utf8_info中存放的是当前属性的名字"Code" |
| u4   | attribute_length       | attribute_length给出了当前Code属性的长度（不包括前六字节）   |
| u2   | max_stack              | 指定当前方法被执行引擎执行的时候，在栈帧中需要分配的操作数栈的大小 |
| u2   | max_locals             | 指定当前方法被执行引擎执行的时候，在栈帧中需要分配的局部表量表的大小 |
| u4   | code_length            | 指定该方法的字节码的长度，class文件中每条字节码占一个字节    |
|      | code                   | 存放字节码指令本身，它的长度是code_length个字节              |
| u2   | exception_table_length | 指定异常表的大小                                             |
|      | exception_table        | exception_table就是所谓的异常表，它是对方法体中try-catch_finally的描述 |
| u2   | attributes_count       | 属性个数                                                     |
|      | attributes             | attributes_count个属性                                       |

# Java运行时数据区和常用指令

## 指令集分类

1. 基于寄存器的指令集
2. 基于栈的指令集

## Runtime Data Area

java中每个线程都有自己的PC程序计数器、JVM栈和本地方法栈；所以线程共享Heap和Method Area

### PC 程序计数器

存放指令位置

### JVM Stack

#### Frame

栈由一个一个栈帧组成(每个方法对应一个栈帧)，每个栈帧由以下四个部分组成：

1. 局部变量表(Local Variable Table)

   存放方法运行所需的所有局部变量，当方法为非静态方法时，局部变量表中下标为0的位置默认放的是this

2. 操作数栈(Operand Stack)

   所有运算所需的变量都要先压入栈中，而运算时则弹出栈进行计算。操作数栈相当于cpu的寄存器，cpu在计算时也是先将变量读取到寄存器中，然后再进行计算。所以说java的jvm虚拟机就相当于一个虚拟操作系统

3. 动态链接(Dynamic Linking)

   - 动态链接主要就是指向当前class的运行时常量池的引用
   - 每一个栈帧内存都包含一个指向该方法所属class的运行时常量池的引用，包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接(Dynamic Linking)。
   - 在Java源文件被编译到字节码文件中时，所有的变量和方法引用都作为符号引用(Symbolic Reference )保存在class文件的常量池里。但是jvm在执行某个类的时候，必须经过加载、连接、初始化，而连接又包括验证、准备、解析三个阶段，而当类加载到内存中后jvm就会将class常量池中的内容存放到运行时常量池中，并且经过解析之后，class常量池中原来存的符号引用会被替换为直接引用存放到运行时常量池中。
   - 当一个方法调用其他方法时，首先通过动态链接找到该class的运行时常量池的引用，然后再在运行时常量池中找到被调用方法的直接引用。

4. 动态返回地址(return address)

   a() -> b()，方法a调用了方法b, b方法的返回值放在什么地方

#### 字节码实例分析

案例1：

```java
package org.duo;

public class TestPlusPlus {
    public static void main(String[] args) {
        int i = 8;
        i = i++;
        System.out.println(i);
    }
}  
```

```
 0 bipush 8    #将8压入栈
 2 istore_1    #将8弹出栈，赋值给局部变量表下标为1的变量(此时局部变量表中下标为1的变量值为8)
 3 iload_1     #将局部变量表下标为1的数压入栈中(此时局部变量表中下标为1的变量值为8)
 4 iinc 1 by 1 #将局部变量表下标为1的数加1(完成自增后，局部变量表中下标为1的变量值为9)
 7 istore_1    #将操作数栈中的8弹出栈赋值给局部变量表下标为1的变量(局部变量表中下标为1的变量值又变成8了)
 8 getstatic #2 <java/lang/System.out : Ljava/io/PrintStream;>
11 iload_1
12 invokevirtual #3 <java/io/PrintStream.println : (I)V>
15 return
```

案例2：

```java
package org.duo;

public class TestPlusPlus {
    public static void main(String[] args) {
        int i = 8;
        i = ++i;
        System.out.println(i);
    }
}
```

```
 0 bipush 8     #将8压入栈
 2 istore_1     #将8弹出栈，赋值给局部变量表下标为1的变量(此时局部变量表中下标为1的变量值为8)
 3 iinc 1 by 1  #将局部变量表下标为1的数加1(完成自增后，局部变量表中下标为1的变量值为9)
 6 iload_1      #将局部变量表下标为1的数压入栈中(此时局部变量表中下标为1的变量值为9)
 7 istore_1     #将操作数栈中的9弹出栈赋值给局部变量表下标为1的变量(局部变量表中下标为1的变量值变成了9)
 8 getstatic #2 <java/lang/System.out : Ljava/io/PrintStream;>
11 iload_1
12 invokevirtual #3 <java/io/PrintStream.println : (I)V>
15 return
```

案例3：

```java
package org.duo;

public class Plus {
    public void add(int a, int b) {
        int c = a + b;
    }
}
```

```
0 iload_1  #将局部变量表下标为1的数压入栈中
1 iload_2  #将局部变量表下标为2的数压入栈中
2 iadd     #将栈中的数弹出后相加，然后将计算后的数压入栈中(iadd指令相当于出栈相加再入栈)
3 istore_3 #将操作数栈中栈顶的数弹出赋值给局部变量表下标为3的变量
4 return
```

案例4：

```java
package org.duo;

public class TestCreateObject {
    public static void main(String[] args) {
        TestCreateObject h = new TestCreateObject();
        int i = h.m1();
    }

    public int m1() {
        return 100;
    }
}

```

```
 0 new #2 <org/duo/TestCreateObject> #new对象，然后将对象地址压栈(此时对象中为默认值)
 3 dup        #dup指令的意思是，将栈中的引用复制一份再压入栈中(此时栈中的两个引用都指向new出来的对象)
 4 invokespecial #3 <org/duo/TestCreateObject.<init> : ()V> #执行对象的构造方法，这个时候会把栈顶的那个引用弹出去执行构造方法，因为执行构造方法的时候需要告诉它需要对哪个对象执行，所以会把栈顶的那个弹出来进行运算。在对象构造完成后栈中剩下的引用指向的是完成初始化的对象。
 7 astore_1   #将操作数栈中的引用弹出赋值给局部变量表下标为1的变量
 8 aload_1
 9 invokevirtual #4 <org/duo/TestCreateObject.m1 : ()I>
12 istore_2
13 return
```

案例5：

```java
package org.duo.classformat;

public class SingletonInstance {

    /**
     * instance要加volatile修饰的原因；
     * 其实new在转换成字节码后会变成下面的0,3,4,7四条指令,而invokespecial和astore_1有可能发生指令重排，如果invokespecial和astore_1发生指令重排会出现下面的问题：
     * 现在有2个线程A,B
     * A在执行new SingletonInstance的时候，B线程进来，此时A执行了new和astore_1没有执行invokespecial，
     * 此时B线程判断instance不为null 直接返回一个未初始化的对象，就会出现问题
     */
    private volatile static SingletonInstance instance;

    private SingletonInstance() {
    }

    public static SingletonInstance getInstance() {
        // 不把锁加在方法上的原因：
        // 缩小锁的粒度，如果将锁加在方法上那么所有调用getInstance的线程都要去竞争锁，但是其实除了第一次初始化instance对象，
        // 其他的时候都可以通过空判断直接获得instance，所以同步块放在if判断之后
        if (instance == null) {
            synchronized (SingletonInstance.class) {
                // 这里还需要对instance做一次空判断的原因
                // 在上一个对instance做空判断之后，到给SingletonInstance.class加锁之间这段时间内，有可能有其他的线程调用了getInstance方法对instance进行了初始化操作
                if (instance == null) {
                    instance = new SingletonInstance();
                }
            }
        }
        return instance;
    }

    public static void main(String[] args) {
        SingletonInstance instance = new SingletonInstance();
    }
}
```

```
0 new #3 <org/duo/classformat/SingletonInstance>
3 dup
4 invokespecial #4 <org/duo/classformat/SingletonInstance.<init> : ()V>
7 astore_1
8 return
```

### Heap

### Method Area

1. Perm Space (<1.8)
   字符串常量位于PermSpace
   FGC不会清理
   大小启动的时候指定，不能变
2. Meta Space (>=1.8)
   字符串常量位于堆
   会触发FGC清理
   不设定的话，最大就是物理内存

### Runtime Constant Pool

### Native Method Stack

### Direct Memory

## JVM常用指令

store

load

pop

mul

sub

invoke

1. InvokeStatic
2. InvokeVirtual
3. InvokeInterface
4. InovkeSpecial
   可以直接定位，不需要多态的方法
   private 方法 ， 构造方法
5. InvokeDynamic
   JVM最难的指令
   lambda表达式或者反射或者其他动态语言scala kotlin，或者CGLib ASM，动态产生的class，会用到的指令

# 类加载-初始化

## class cycle

class的加载主要分为下面三步：

1. 装载(loading)

   将class文件从硬盘load到内存

2. 链接(linking)

   1. 校验(verification)

      校验load进内存的文件符不符合class文件的格式标准(CAFEBABE)

   2. 准备(preparation)

      class中静态变量赋默认值(注意是默认值，比如整型的默认值是0)

   3. 解析(resolution)

      将类、方法、属性等符号引用解析为指针、偏移量等内存地址的直接引用

3. 初始化(initializing)

   class中静态变量赋初始值

## 类加载器

JVM是按需动态加载：

自底向上检查该类是否已经加载(child==>parent)

自顶向下进行实际加载(parent==>child)

注意父加载器不是父类，双亲委派是一个孩子向父亲方向查找，然后父亲向孩子方向加载的过程。

双亲委派机制是为了保证每一个类在各个类加载器中都是同一个类。一个非常明显的目的就是保证java官方的类库<JAVA_HOME>\lib和扩展类库<JAVA_HOME>\lib\ext的加载安全性，不会被开发者覆盖。例如类java.lang.Object，它存放在rt.jar之中，无论哪个类加载器要加载这个类，最终都是委派给启动类加载器加载，因此Object类在程序的各种类加载器环境中都是同一个类。

| 名称               | 范围                                                    |      |
| ------------------ | ------------------------------------------------------- | ---- |
| Bootstrap          | 加载lib/rt.jar等核心类，C++实现                         |      |
| Extension          | 加载扩展jar包jre/lib/ext/*.jar，或由-Djava.ext.dirs指定 |      |
| App                | 加载classpath指定内容                                   |      |
| Custom ClassLoader | 自定义ClassLoader                                       |      |

## 自定义类加载器

```java
package org.duo.classloader;

import org.duo.Hello;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.lang.reflect.Method;

/**
 * 通过继承ClassLoader的自定义classloder可以指定父classloder也可以使用默认的父classloder
 * 如果需要指定父classloder则初始化自定义的classloder时使用带参数的构造方法：ClassLoader(ClassLoader parent)
 * 如果使用默认的父classloder则使用无参的构造方法
 * 在ClassLoader中无参的构造方法，实际会调用this(checkCreateClassLoader(), getSystemClassLoader());
 * 而ClassLoader.getSystemClassLoader()返回的是AppClassLoader
 * 所以自定义的classloder一般默认的父classloder是AppClassLoader
 */
public class CustomClassLoader extends ClassLoader {

    @Override
    /**
     * 重写loadClass会打破双亲委派机制
     * 所以一般的自定义ClassLoader只要继承ClassLoader并重写findClass方法
     * 先将class文件以二进制的形式读入内存，然后再调用defineClass将二进制转化为类对象(byte[] -> Class clazz)
     */
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        File f = new File("D:\\intellij-workspace\\juc\\out\\production\\juc", name.replace(".", "\\").concat(".class"));
        try {
            FileInputStream fis = new FileInputStream(f);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int b = 0;
            while ((b = fis.read()) != -1) {
                baos.write(b);
            }
            byte[] bytes = baos.toByteArray();
            baos.close();
            fis.close();//可以写的更加严谨
            return defineClass(name, bytes, 0, bytes.length);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return super.findClass(name); //throws ClassNotFoundException
    }

    public static void main(String[] args) throws Exception {

        ClassLoader l = new CustomClassLoader();

        //如果这个准备被加载的类：org.duo.Hello在当前classpath下，那么上面的findClass是执行不到的
        //因为org.duo.Hello会被CustomClassLoader的父加载器：sun.misc.Launcher$AppClassLoader加载进来
        Class clazz = l.loadClass("org.duo.Hello");
        Hello h = (Hello) clazz.newInstance();
        h.m();

//        //如果这个准备被加载的类：org.duo.bytecode.ClassFormat不在当前classpath，那么上面的findClass将被执行
//        //由于加载的类不在当前classpath中，所以需要反射才能执行被加载的类中的方法
//        Class clazz = l.loadClass("org.duo.bytecode.ClassFormat");
//        Method method = clazz.getMethod("print");
//        method.invoke(clazz.newInstance());

        System.out.println("ClassLoader.getSystemClassLoader ==>" + ClassLoader.getSystemClassLoader());
        System.out.println(clazz.getName() + "的类加载器为：" + clazz.getClassLoader());
    }
}
```

class文件加密

```java
package org.duo.classloader;

import org.duo.Hello;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;

/**
 * 通过继承ClassLoader的自定义classloder可以指定父classloder也可以使用默认的父classloder
 * 如果需要指定父classloder则初始化自定义的classloder时使用带参数的构造方法：ClassLoader(ClassLoader parent)
 * 如果使用默认的父classloder则使用无参的构造方法
 * 在ClassLoader中无参的构造方法，实际会调用this(checkCreateClassLoader(), getSystemClassLoader());
 * 而ClassLoader.getSystemClassLoader()返回的是AppClassLoader
 * 所以自定义的classloder一般默认的父classloder是AppClassLoader
 */
public class CustomClassLoaderWithEncription extends ClassLoader {

    public static int seed = 0B10110110;

    @Override
    /**
     * 重写loadClass会打破双亲委派机制
     * 所以一般的自定义ClassLoader只要继承ClassLoader并重写findClass方法
     * 先将class文件以二进制的形式读入内存，然后再调用defineClass将二进制转化为类对象(byte[] -> Class clazz)
     */
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        File f = new File("D:\\intellij-workspace\\jvm\\out\\production\\jvm", name.replace("\\.", "\\").concat(".msbclass"));
        try {
            FileInputStream fis = new FileInputStream(f);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int b = 0;
            while ((b = fis.read()) != -1) {
                //对读入的加密过的二进制再进行异或
                baos.write(b ^ seed);
            }
            byte[] bytes = baos.toByteArray();
            baos.close();
            fis.close();//可以写的更加严谨
            return defineClass(name, bytes, 0, bytes.length);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return super.findClass(name); //throws ClassNotFoundException
    }

    public static void main(String[] args) throws Exception {

        encFile("org.duo.Hello");
        ClassLoader l = new CustomClassLoaderWithEncription();

        //如果这个准备被加载的类：org.duo.Hello在当前classpath下，那么上面的findClass是执行不到的
        //因为org.duo.Hello会被CustomClassLoaderWithEncription的父加载器：sun.misc.Launcher$AppClassLoader加载进来
        Class clazz = l.loadClass("org.duo.Hello");
        Hello h = (Hello) clazz.newInstance();
        h.m();

        System.out.println("ClassLoader.getSystemClassLoader ==>" + ClassLoader.getSystemClassLoader());
        System.out.println(clazz.getName() + "的类加载器为：" + clazz.getClassLoader());
    }

    /**
     * 对生成的class文件进行加密
     * 采用最简单的方式：对二进制的每一位进行异或(对异或完的值再进行异或就相当于解密)
     *
     * @param name 类名
     * @throws Exception
     */
    private static void encFile(String name) throws Exception {
        File f = new File("D:\\intellij-workspace\\jvm\\out\\production\\jvm", name.replace(".", "\\").concat(".class"));
        FileInputStream fis = new FileInputStream(f);
        FileOutputStream fos = new FileOutputStream(new File("D:\\intellij-workspace\\jvm\\out\\production\\jvm", name.replace(".", "\\").concat(".msbclass")));
        int b = 0;
        while ((b = fis.read()) != -1) {
            fos.write(b ^ seed);
        }
        fis.close();
        fos.close();
    }
}
```

双亲委派的打破

1. 重写loadClass()可以打破双亲委派
2. JDK1.2之前，自定义ClassLoader都必须重写loadClass()
3. ThreadContextClassLoader可以实现基础类调用实现类代码，通过thread.setContextClassLoader指定
4. Tomcat服务器在启动的时候可以加载多个WebApp，不同的WebApp可能会用到同一类库的不同版本，因此Tomcat自定义的classloader必须打破双亲委派(因为ClassLoader的委派机制，在第一次加载完某个类后是不会重复再加载的，所以如果不打破，无法实现加载同一类库的不同版本的要求)

```java
package org.duo.classloader;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

/**
 * 重写ClassLoader的loadclass方法，模拟tomcat的热部署
 */
public class ClassReloading extends ClassLoader {

    /**
     * 重写loadClass打破双亲委派
     */
    public Class<?> loadClass(String name) throws ClassNotFoundException {

        File f = new File("D:\\intellij-workspace\\juc\\out\\production\\juc", name.replace(".", "\\").concat(".class"));
        //当目录中存在指定的class文件则加载，否则交给父ClassLoader加载
        if (!f.exists()) return super.loadClass(name);
        try {
            InputStream is = new FileInputStream(f);
            byte[] b = new byte[is.available()];
            is.read(b);
            return defineClass(name, b, 0, b.length);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return super.loadClass(name);
    }

    public static void main(String[] args) throws Exception {

        //相当于tomcat启动后生成classloder
        ClassReloading m = new ClassReloading();
        Class clazz = m.loadClass("org.duo.bytecode.ClassFormat");

        //当代码更新后会生成新的class文件，如果想实现热部署就必须重新加载更新后的class文件
        //所有这里先new一个新的classloder赋值给该WebApp的类加载器(相当于将原来的类加载器对象废除)，然后再重新loading指定路径下的class
        m = new ClassReloading();
        Class clazzNew = m.loadClass("org.duo.bytecode.ClassFormat");

        System.out.println(clazz == clazzNew);
    }
}
```

# JVM的执行模式

## 解释器

bytecode intepreter

## JIT

Just In-Time compiler

## 混合模式

1. 混合使用解释器+热点代码编译
2. 源代码经javac编译成字节码，class文件
3. 程序字节码经过JIT环境变量进行判断，是否属于热点代码
   - 多次被调用的方法(方法计数器：监测方法执行频率)
   - 多次被调用的循环(循环计数器：检测循环执行频率)
4. 如果经判断属于热点代码，则直接由JIT编译为具体硬件处理器机器码
5. 如果经判断不是热点代码，则直接由解释器解释执行

## 参数

- -Xmixed 默认为混合模式，开始解释执行，启动速度较快 对热点代码实行检测和编译
- -Xint 使用编译模式，启动很快执行稍慢
- -Xcomp 使用纯编译模式，执行很快，启动很慢
- -XX:CompileThreshold = 10000

# 对象的内存布局

## 对象的创建过程

1. class loading
2. class linking(verification,preparation,resolution)
3. class initializing
4. 申请对象内存
5. 成员变量赋默认值
6. 调用构造方法<init>
   1. 成员变量顺序赋初始值
   2. 执行构造方法语句

## 对象在内存中的存储布局

### 观察虚拟机配置

java -XX:+PrintCommandLineFlags --version

### 普通对象

1. 对象头：markword  8字节

2. ClassPointer指针：-XX:+UseCompressedClassPointers 为4字节 不开启为8字节

3. 实例数据

   1. 引用类型：-XX:+UseCompressedOops 为4字节 不开启为8字节

      Oops Ordinary Object Pointers

4. Padding对齐，8的倍数

### 数组对象

1. 对象头：markword 8字节
2. ClassPointer指针同上
3. 数组长度：4字节
4. 数组数据
5. 对齐 8的倍数

## 对象在内存中占用多少字节

1. 新建项目ObjectSize （1.8）

2. 创建文件ObjectSizeAgent

   ```java
   package com.mashibing.jvm.agent;
   
   import java.lang.instrument.Instrumentation;
   
   public class ObjectSizeAgent {
       private static Instrumentation inst;
   
       public static void premain(String agentArgs, Instrumentation _inst) {
           inst = _inst;
       }
   
       public static long sizeOf(Object o) {
           return inst.getObjectSize(o);
       }
   }
   ```

3. src目录下创建META-INF/MANIFEST.MF

   ```
   Manifest-Version: 1.0
   Created-By: org.duo
   Premain-Class: org.duo.jvm.agent.ObjectSizeAgent
   
   ```

   注意Premain-Class这行必须是新的一行（回车 + 换行），确认idea不能有任何错误提示

4. project structure - project settings - Artifacts设置完成后，继续点击Build–>Build Artifacts生成jar

5. 在需要使用该Agent Jar的项目中引入该Jar包

   project structure - project settings - library 添加该jar包

6. 运行时需要该Agent Jar的类，加入VM参数：

   ```
   -javaagent:D:\intellij-workspace\agent\out\artifacts\agent_jar\agent.jar
   ```

7. 使用该类

   ```java
   package org.duo.jmm;
   
   import org.duo.jvm.agent.ObjectSizeAgent;
   
   public class SizeOfAnObject {
   
       public static void main(String[] args) {
           System.out.println(ObjectSizeAgent.sizeOf(new Object()));
           System.out.println(ObjectSizeAgent.sizeOf(new int[] {}));
           System.out.println(ObjectSizeAgent.sizeOf(new P()));
       }
   
       //一个Object占多少个字节
       // -XX:+UseCompressedClassPointers -XX:+UseCompressedOops
       // Oops = ordinary object pointers
       private static class P {
           //8 _markword
           //4 _class pointer
           int id;         //4
           String name;    //4
           int age;        //4
           byte b1;        //1
           byte b2;        //1
           Object o;       //4
           byte b3;        //1
       }
   }
   ```

## 对象头具体包括什么

以32位的JDK为例

| 锁状态   | 25bit                        | 4bit         | 1bit(是否偏向锁) | 2bit(锁标志位) |
| -------- | ---------------------------- | ------------ | ---------------- | -------------- |
| 轻量级锁 | 指向栈中锁记录的指针         | -            | -                | 00             |
| 重量级锁 | 指向互斥量（重量级锁）的指针 | -            | -                | 10             |
| GC标记   | 空                           | -            | -                | 11             |
| 偏向锁   | 线程ID(23bit)-Epoch(2bit)    | 对象分代年龄 | 1                | 01             |
| 无锁     | 对象的hashCode               | 对象分代年龄 | 0                | 01             |

32位JDK的Mark Word

|--------------------------------------------------------------------------------------------------------------------|----------------------------------|

|                                             Mark Word (32 bits)                                                               |                 State                |

| -------------------------------------------------------------------------------------------------------------------|----------------------------------|

|                              hashcode:25                           |    age:4   |   biased_lock:0   |   01    |                Normal            |

| -------------------------------------------------------------------------------------------------------------------|----------------------------------|

|               thread:23          |        epoch:2             |    age:4   |   biased_lock:1   |   01    |                Biased              |

| -------------------------------------------------------------------------------------------------------------------|----------------------------------|

|                                                  ptr_to_lock_record:30                                          |   00   |    Lightweight Locked    |

| -------------------------------------------------------------------------------------------------------------------|----------------------------------|

|                                                  ptr_to_heavyweight_monitor:30                        |   10    |  Heavyweight Locked   |

| -------------------------------------------------------------------------------------------------------------------|----------------------------------|

|                                                                                                                                   |   11    |        Marked for GC       |

| --------------------------------------------------------------------------------------------------------------------|---------------------------------|

64位JDK的Mark Word

|-------------------------------------------------------------------------------------------------------------------|-----------------------------------|

|                                             Mark Word (64 bits)                                                               |                  State               |

| -------------------------------------------------------------------------------------------------------------------|----------------------------------|

|    unsed:25    |    hashcode:31    |    unsed:1  |  age:4   |   biased_lock:0     | 01     |                  Normal           |

| -------------------------------------------------------------------------------------------------------------------|----------------------------------|

|    unsed:54    |    epoch:2             |    unsed:1  |  age:4   |   biased_lock:1     | 01     |                  Biased            |

| -------------------------------------------------------------------------------------------------------------------|----------------------------------|

|                                                  ptr_to_lock_record:62                                          | 00     |    Lightweight Locked    |

| -------------------------------------------------------------------------------------------------------------------|----------------------------------|

|                                                  ptr_to_heavyweight_monitor:62                        | 10      |  Heavyweight Locked   |

| -------------------------------------------------------------------------------------------------------------------|----------------------------------|

|                                                                                                                                   | 11     |         Marked for GC       |

| -------------------------------------------------------------------------------------------------------------------|----------------------------------|

Monitor（管程）的基本概念：

1. 管程：指的是管理共享变量以及对共享变量的操作过程，让他们支持并发。
2. Java中的monitor:每个Java对象都可以关联一个monitor对象，如果对一个对象使用synchronized关键字，那个这个对象的对象头的markword就被设置指向monitor对象的指针。

### synchronized的加锁过程分析：

1. 刚开始Monitor中Owner为null
2. 当Thread-2执行synchronized(obj)就会将Monitor的所有者Owner设置为Thread-2，Monitor中只能有一个Owner
3. 在Thread-2上锁的过程中，如果Thread-3、Thread-4、Thread-5也来执行synchronized(obj)就会进入EntryList BLOCKED
4. Thread-2执行完同步代码块的内容，然后唤醒EntryList中等待的线程来竞争锁
5. WaitSet中的线程Thread-0、Thread-1是之前获得过锁，但条件不满足所以进入WAITING状态的线程
6. 注意：
   - synchronized必须是进入同一对象的monitor才有上述的效果
   - 不加synchronized的对象不会关联监视器，不遵从以上规则

### 轻量级锁：

#### 加锁过程分析

1. 在代码进入同步块的时候，如果同步对象锁状态为无锁状态（锁标志位为“01”状态，是否为偏向锁为“0”），虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的Mark Word的拷贝，官方称之为 Displaced Mark Word。
2. 拷贝对象头中的Mark Word复制到锁记录中。
3. 拷贝成功后，虚拟机将使用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，并将Lock record里的owner指针指向object mark word。如果更新成功，则执行步骤（4），否则执行步骤（5）。
4. 如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象Mark Word的锁标志位设置为“00”，即表示此对象处于轻量级锁定状态，
5. 如果这个更新操作失败了，虚拟机首先会检查对象的Mark  Word是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行。否则说明多个线程竞争锁，轻量级锁就要膨胀为重量级锁，锁标志的状态值变为“10”，Mark Word中存储的就是指向重量级锁（互斥量）的指针，后面等待锁的线程也要进入阻塞状态。  而当前线程便尝试使用自旋来获取锁，自旋就是为了不让线程阻塞，而采用循环去获取锁的过程。

#### 解锁过程分析

1. 通过CAS操作尝试把线程中复制的Displaced Mark Word对象替换当前的Mark Word。
2. 如果替换成功，整个同步过程就完成了。
3. 如果替换失败，说明有其他线程尝试过获取该锁（此时锁已膨胀），那就要在释放锁的同时，唤醒被挂起的线程。

### 锁膨胀

1. 当 Thread-1 进行轻量级加锁时，Thread-0 已经对该对象加了轻量级锁
2. Thread1 为 Object 对象申请 Monitor 锁，让 Object 指向重量级锁地址,自己进入 Monitor 的 EntryList 阻塞等待
3. 当 Thread-0 退出同步块解锁时，使用 CAS 将 Mark Word 的值恢复给对象头，必定失败。这时会进入重量级解锁流程，即按照  Monitor 地址找到 Monitor 对象，设置 Owner 为 null，唤醒 EntryList 中 BLOCKED 线程

### 偏向锁

引入偏向锁是为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径，因为轻量级锁的获取及释放依赖多次CAS原子指令，而偏向锁只需要在置换ThreadID的时候依赖一次CAS原子指令。轻量级锁是为了在线程交替执行同步块时提高性能，而偏向锁则是在只有一个线程执行同步块时进一步提高性能。

- 轻量级锁
  - 优点：轻量级别锁通过线程栈帧中的锁记录结构替代重量级锁，不需要关联monitor对象。
  - 缺点：单个线程（没有其他线程与其竞争）使用轻量级锁，在**锁重入**的时候仍然需要执行CAS操作（栈帧中添加一个新的lock record）。
  - 锁重入：同一线程多次对同一对象加锁。

偏向锁为了克服轻量级锁的缺点而提出的。

#### 加锁过程分析

1. 访问Mark Word中偏向锁的标识是否设置成1，锁标志位是否为01——确认为可偏向状态。
2. 如果为可偏向状态，则测试线程ID是否指向当前线程，如果是，进入步骤（5），否则进入步骤（3）。
3. 如果线程ID并未指向当前线程，则通过CAS操作竞争锁。如果竞争成功，则将Mark Word中线程ID设置为当前线程ID，然后执行（5）；如果竞争失败，执行（4）。
4. 如果CAS获取偏向锁失败，则表示有竞争。当到达全局安全点（safepoint）时获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。
5. 执行同步代码。

#### 解锁过程分析

偏向锁的撤销在上述第四步骤中有提到**。**偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动去释放偏向锁。偏向锁的撤销，需要等待全局安全点（在这个时间点上没有字节码正在执行），它会首先暂停拥有偏向锁的线程然后判断该线程是否仍然需要持有偏向锁，如果仍然需要持有偏向锁，则偏向锁升级为轻量级锁（标志位为“00”，偏向锁就是这个时候升级为轻量级锁的），如果不需要使用了，则可以将对象回复成无锁状态（标志位为“01”），然后重新偏向。

# JVM分析

## jstack

可以使用jstack -l pid > jstack.tdump线程信息存入文件中，然后再从服务器下载到本地然后使用jvisualvm进行dump文件分析

jstack入参说明

- -F 当jstack [-l] pid没有相应的时候强制打印栈信息,如果直接jstack无响应时，用于强制jstack，一般情况不需要使用
- -l 长列表. 打印关于锁的附加信息,例如属于java.util.concurrent的ownable synchronizers列表，会使得JVM停顿得长久得多（可能会差很多倍，比如普通的jstack可能几毫秒和一次GC没区别，加了-l 就是近一秒的时间），-l 建议不要用。一般情况不需要使用
- -m 打印java和native c/c++ 框架的所有栈信息.可以打印JVM的堆栈,显示上Native的栈帧，一般应用排查不需要使用

jstack出参说明

- prio ： 表示线程优先级，就是Thread中定义的这个。
- os_prio ： 表示操作系统级别的优先级
- tid : 表示Java内的线程ID,同样在Thread类中
- nid：表示操作系统级别的线程ID的16进制形式

Top命令找出CPU占用较高的Java线程信息

1. 第一步：首先使用top找出占用CPU较高的进程ID 
2. 第二步：使用top -H -p pid查看该进程里占用CPU较高的线程ID 
3. 第三步：把得到的线程ID转成16进制
4. 第四步：在线程堆栈里找出线程ID对应的代码块，jstack -l 1541 | grep 0x610 -A 20

