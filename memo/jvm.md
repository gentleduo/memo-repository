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

### 工具

JOL = Java Object Layout

```
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.9</version>
</dependency>
```

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

JVM中的同步是基于进入与退出监视器对象(Monitor，也叫管程对象)来实现的，每个对象实例都会有一个Monitor对象，Monitor对象会和Java对象一同创建并销毁。Monitor对象是JVM底层通过C++实现的。当多个线程同时访问一段同步代码时，这些线程会被放到一个Monitor对象的EntrySet集合中，处于阻塞状态的线程都会被放到该列表中。接下来，当线程获取到对象的Monitor时，Monitor是依赖于底层操作系统的mutex lock来实现互斥的，线程获取mutex成功，则会持有该mutex，这时其他线程就无法再获取到该mutex。如果线程调用了wait()方法，那么该线程就会释放掉所有的mutex，并且该线程会进入到Monitor对象的WaitSet集合(等待集合)中，等待下一次被其他线程调用notify/notifyAll唤醒。如果当前线程顺利执行完方法，那么它也会释放掉所持有的mutex。
总结：同步锁在这种实现方式当中，因为Monitor是依赖底层的操作系统实现的，这样就存在用户态和内核态之间的切换，所以会增加性能开销。

synchronized关键字的底层C++实现，虽然Hospot代码没有开源，但是社区版本的JDK是开源了，在openjdk上可以阅读得到，具体路径如下：

http://openjdk.java.net/ -> Mercurial -> jdk8u -> hotspot -> browse -> /src/share/vm/runtime/objectMonitor.cpp

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
5. 如果这个更新操作失败了，虚拟机首先会检查对象的Mark  Word是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行。否则说明多个线程竞争锁，当竞争加剧(JDK1.6之前的版本：当有线程超过10次自旋，-XX:PreBlockSpin，或者自旋线程数超过CPU核数的一半，升级重量级锁；JDK1.6之后的版本加入自适应自旋 Adapative Self Spinning ， JVM自己控制)，轻量级锁就要膨胀为重量级锁，锁标志的状态值变为“10”，Mark Word中存储的就是指向重量级锁（互斥量）的指针，后面等待锁的线程也要进入阻塞状态。 

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

#### 启动偏向锁设置

JVM提供了参数- XX:+UseBiasedLocking以开启或者关闭偏向锁优化（默认开启），但jvm延迟了4s启动偏向锁，想要在程序一启动时，就启动偏向锁，有两种方式：

- 程序启动时，让主线程睡眠超过4s，这样就会在jvm延迟启动偏向锁后，才开始运行程序。
- 设置jvm启动参数，添加-XX:BiasedLockingStartupDelay=0，即设置延迟启动偏向锁的时间为0s，这样jvm启动时，偏向锁就会启动。

#### 批量重偏向与批量撤销

从偏向锁的加锁解锁过程中可看出，当只有一个线程反复进入同步块时，偏向锁带来的性能开销基本可以忽略，但是当有其他线程尝试获得锁时，就需要等到safe point时，再将偏向锁撤销为无锁状态或升级为轻量级，会消耗一定的性能，所以在多线程竞争频繁的情况下，偏向锁不仅不能提高性能，还会导致性能下降。于是，就有了批量重偏向与批量撤销的机制。

批量重偏向(bulk rebias)：如果一个类的大量对象被一个线程T1执行了同步操作，也就是大量对象先偏向了T1，T1同步结束后，另一个线程也将这些对象作为锁对象进行操作，会导偏向锁重偏向的操作。

批量撤销(bulk revoke)机制是为了解决：当一个偏向锁如果撤销次数到达40的时候就认为这个对象设计得有问题；那么JVM会把这个对象所对应的类所有的对象都撤销偏向锁；并且新实例化的对象也是不可偏向的。

批量重偏向与批量撤销相关的JVM参数：

1. BiasedLockingBulkRebiasThreshold：偏向锁批量重偏向的默认阀值为20次。
2. BiasedLockingBulkRevokeThreshold：偏向锁批量撤销的默认阀值为40次。
3. BiasedLockingDecayTime：距上次批量重偏向25秒内，撤销计数达到40，就会发生批量撤销。每隔(>=)25秒，会重置在[20, 40)内的计数，这意味着可以发生多次批量重偏向。

epoch：每个class对象会有一个对应的epoch字段，每个处于偏向锁状态对象的Mark Word中也有该字段，其初始值为创建该对象时class中的epoch的值。每次发生批量重偏向时，就将class对象中的该值+1，同时遍历JVM中所有线程的栈，找到该class所有正处于偏向锁状态的锁实例对象，将其epoch字段改为新值。当触发批量重偏向的线程再次给该类的其他对象加锁时发现该对象的epoch值和class的epoch不相等，就知道发生了重偏向，将不会执行撤销偏向锁操作，而是直接将其markword的线程Id改成当前线程Id。每次执行批量撤销时，将class的markword修改为不可偏向无锁状态，也就是偏向标记位为0，锁标记位为01。接着遍历所有当前存活的线程的栈，找到该class所有正处于偏向锁状态的锁实例对象，执行偏向锁的撤销操作。

示例代码

```java
package org.duo.markword;

import org.openjdk.jol.info.ClassLayout;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.LockSupport;

public class BulkBias {

    static Thread A;
    static Thread B;
    static Thread C;
    static int loopFlag = 40;

    public static void main(String[] args) {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        final List<Object> list = new ArrayList<>();
        // 线程A：1-40加锁都是101，即偏向锁状态；线程A首次加的锁，并且没有别的线程竞争，所以对象头是偏向锁状态，对应的Thread Id为线程A
        A = new Thread(() -> {
            for (int i = 0; i < loopFlag; i++) {
                BiasedLocking c = new BiasedLocking();
                list.add(c);
                System.out.println(String.format("加锁前第 %s 次 %s", String.valueOf(i + 1), ClassLayout.parseInstance(c).toPrintable()));
                synchronized (c) {
                    System.out.println(String.format("加锁中第 %s 次 %s", String.valueOf(i + 1), ClassLayout.parseInstance(c).toPrintable()));
                }
                System.out.println(String.format("加锁结束第 %s 次 %s", String.valueOf(i + 1), ClassLayout.parseInstance(c).toPrintable()));
            }
            System.out.println("============线程A=============");
            //防止竞争 执行完后叫醒 线程B
            LockSupport.unpark(B);
        });
        // 线程B：1-19都执行了锁撤销，第20再进行锁撤销的话就到达BiasedLockingBulkRebiasThreshold的次数，触发批量重偏向，线程id指向线程B；
        B = new Thread(() -> {
            //防止竞争 先睡眠线程B
            LockSupport.park();
            for (int i = 0; i < loopFlag; i++) {
                BiasedLocking c = (BiasedLocking) list.get(i);
                System.out.println(String.format("加锁前第 %s 次 %s", String.valueOf(i + 1), ClassLayout.parseInstance(c).toPrintable()));
                synchronized (c) {
                    System.out.println(String.format("加锁中第 %s 次 %s", String.valueOf(i + 1), ClassLayout.parseInstance(c).toPrintable()));
                }
                //前19次是轻量级锁，释放之后为无锁不可偏向
                //但是第20次是偏向锁 偏向线程B 释放之后依然是偏向线程B
                System.out.println(String.format("加锁结束第 %s 次 %s", String.valueOf(i + 1), ClassLayout.parseInstance(c).toPrintable()));
            }
            // 新创建的对象为无锁可偏向状态，并且epoch=01(10进制中的1)
            System.out.println("新产生的对象" + ClassLayout.parseInstance(new BiasedLocking()).toPrintable());
            System.out.println("============线程B=============");
            //防止竞争 执行完后叫醒 线程C
            LockSupport.unpark(C);
        });
        // 线程C：20-40又都执行了锁撤销，到达BiasedLockingBulkRevokeThreshold的次数，触发批量撤销
        C = new Thread(() -> {
            //防止竞争 先睡眠线程C
            LockSupport.park();
            // 如果在这里让线程休眠超过BiasedLockingDecayTime设定的时间(默认25s)，也就是说第一次触发批量重偏向后，25s内没有触发批量锁撤销，那么偏向锁撤销计数器会被重置
            // 那么接下来的20次锁撤销会触发第二次批量重偏向
            try {
                Thread.sleep(25000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            for (int i = 20; i < loopFlag; i++) {
                BiasedLocking c = (BiasedLocking) list.get(i);
                System.out.println(String.format("加锁前第 %s 次 %s", String.valueOf(i + 1), ClassLayout.parseInstance(c).toPrintable()));
                synchronized (c) {
                    System.out.println(String.format("加锁中第 %s 次 %s", String.valueOf(i + 1), ClassLayout.parseInstance(c).toPrintable()));
                }
                System.out.println(String.format("加锁结束第 %s 次 %s", String.valueOf(i + 1), ClassLayout.parseInstance(c).toPrintable()));
            }
            // 如果，触发批量撤销，那么新创建的锁对象仍然为无锁不可偏向状态
            // 如果，触发第二次批量重偏向(即线程C在运行前先睡眠25s)，那么新创建的对象为无锁可偏向状态，并且epoch=10(10进制中的2)
            System.out.println("新产生的对象" + ClassLayout.parseInstance(new BiasedLocking()).toPrintable());
        });
        A.start();
        B.start();
        C.start();
    }
}

class BiasedLocking {
}
```

### 锁重入

sychronized是可重入锁，可重入性是锁的一个基本要求，是为了解决自己锁死自己的情况。一个类中的同步方法调用另一个同步方法，假如Synchronized不支持重入，进入method2方法时当前线程获得锁，method2方法里面执行method1时当前线程又要去尝试获取锁，对Synchronized来说，可重入性是显而易见的，刚才提到，在执行monitorenter指令时，如果这个对象没有锁定，或者当前线程已经拥锁，这时如果不支持重入，它就要等释放，把自己阻塞，导致自己锁死自己。有了这个对象的锁（而不是已拥有了锁则不能继续获取），就把锁的计数器+1，其实本质上就通过这种方式实现了可重入性。

重入次数必须记录，因为要解锁几次必须得对应

偏向锁和自旋锁记录在线程栈中，每一次加锁会在线程栈中增加一条LockRecord，解锁则是将线程栈的LockRecord弹出栈，重量级锁记录在monitor对象上。

注意：如果计算过对象的hashCode，则对象无法进入偏向状态

# GC Tuning

## 垃圾发现

- 引用计数法(reference count)：
  - 引用计数器的实现很简单，对于一个对象A，只要有任何一个对象引用了A，则A的引用计数器就加1，当引用失效时，引用计数器就减1。只要对象A的引用计数器的值为0，则对象A就不可能再被使用。
  - 引用计数法的问题：
    1. 引用和去引用伴随加法和减法，影响性能
    2. 很难处理循环引用
- 根可达算法(Root Searching)
  - which instances are root？
    - JVM Stack
    - native method stack
    - run-time constant pool
    - static references in method area
    - clazz

## 垃圾回收算法

1. 标记清除(Mark-Sweep)：标记-清除算法是现代垃圾回收算法的思想基础。标记-清除算法将垃圾回收分为两个阶段：标记阶段和清除阶段。一种可行的实现是，在标记阶段，首先通过根节点，标记所有从根节点开始的可达对象。因此，未被标记的对象就是未被引用的垃圾对象。然后，在清除阶段，清除所有未被标记的对象。标记清除算法相对简单，在存活对象比较多的情况下效率较高，并且两遍扫描(一遍标记，一遍清除)，效率偏低，容易产生碎片。
2. 复制算法(copying)：与标记-清除算法相比，复制算法是一种相对高效的回收方法，它将原有的内存空间分为两块，每次只使用其中一块，在垃圾回收时，将正在使用的内存中的存活对象复制到未使用的内存块中，之后，清除正在使用的内存块中的所有对象，交换两个内存的角色，完成垃圾回收。复制算法适用于存活对象较少的情况，只扫描一次且没有碎片，但是它最大的问题是：空间浪费、移动复制对象，需要调整对象引用。
3. 标记压缩(Mark-Compact)：标记-压缩算法适合用于存活对象较多的场合，如老年代。它在标记-清除算法的基础上做了一些优化。和标记-清除算法一样，标记-压缩算法也首先需要从根节点开始，对所有可达对象做一次标记。但之后，它并不简单的清理未标记的对象，而是将所有的存活对象压缩到内存的一端。之后，清理边界外所有的空间。标记压缩算法不会产生碎片，方便对象分配，也不会造车空间浪费，但是需要扫描两次，并且移动对象，效率偏低。

## 堆内逻辑分区

新生代大量死去，少量存活，采用复制算法

老年代存活率高，回收较少，采用MC或者MS

```
------------|-----------|-------------|---------------
8           |  1        |  1          |     
eden(伊甸)   |  survivor |  survivor   |  tenured(终身)
--------------------------------------|---------------
   new/young(新生代:1) -Xmn            |  old(老年代:2)  
--------------------------------------|---------------
                  -Xms - Xmx
------------------------------------------------------	
```

MinorGC/YGC：年轻代空间耗尽时触发

MajorGC/FGC：在老年代无法继续分配空间时触发，新生代和老年代同时进行回收	

-XX:NewRatio：新生代(eden+2*s)和老年代的比值，例如：2表示 新生代:老年代=1:4，即年轻代占堆的1/5

-XX:SurvivorRatio：设置两个Survivor区和eden的比，例如：8表示 两个Survivor:eden=2:8，即一个Survivor占年轻代的1/10

-Xmn2G：设置年轻代大小为2G

-Xms3550M：设置JVM初始内存为3550M。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。

-Xmx3550M：设置JVM最大可用内存为3550M。

-Xss128K：设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。

## YGC

HotSpot JVM把年轻代分为了三部分：1个Eden区和2个Survivor区（分别叫from和to）默认比例为8：1；一般情况下，新创建的对象都会被分配到Eden区(一些大对象特殊处理),这些对象经过第一次Minor GC后，如果仍然存活，将会被移到Survivor区。对象在Survivor区中每熬过一次Minor GC，年龄就会增加1岁，当它的年龄增加到一定程度时，就会被移动到年老代中。因为年轻代中的对象基本都是朝生夕死的(80%以上)，所以在年轻代的垃圾回收算法使用的是复制算法，复制算法的基本思想就是将内存分为两块，每次只用其中一块，当这一块内存用完，就将还活着的对象复制到另外一块上面。复制算法不会产生内存碎片。在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor区“To”是空的。紧接着进行GC，Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”。不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中。

## 栈上分配

1. 栈上分配指的是什么
   - 标量替换：将线程中的私有对象打散，让它在栈上分配，而不是在堆上分配，也就是说，JVM不会创建对象，而会将该对象的成员变量以局部变量的形式分配在栈上。
   - 逃逸分析：判断对象是否会被多个线程共享
2. 栈上分配有什么好处
   - 不需要GC介入去回收这个对象，出栈即释放资源，可以提高性能，原理：由于GC每次回收对象的时候，都会触发Stop The World，这时候所有线程都停止了，然后再去进行垃圾回收，如果对象频繁创建在我们的堆中，也就意味着也要频繁的暂停所有线程，这对于用户无非是非常影响体验的，栈上分配就是为了减少垃圾回收的次数
3. 参数
   - -XX:+DoEscapeAnalysis 开启逃逸分析；关闭逃逸分析则是-DoEscapeAnalysis
   - -XX:+EliminateAllocations 开启标量替换
   - -XX:+UseTLAB 开启栈上分配
   - -XX:TLABWasteTargetPercent 占eden区的百分比，默认情况下仅占有整个Eden空间的1%

## 常见的垃圾回收器

### 年轻代常用的垃圾回收器

1. Serial 年轻代 串行回收

   a stop-the-world(STW), copying collector which uses a single GC thread(可以理解在进行回收的时候，其它的工作线程全部暂停)

2. ParallelScavenge(PS) 年轻代 并行回收

   a stop-the-world(STW), copying collector which uses multiple GC threads

3. ParNew 年轻代 配合CMS的并行回收

   a stop-the-world(STW), copying collector which uses multiple GC threads

   it differs from "Parallel Scavenge" in that it has enhancements that make it usable with CMS

   for example, "ParNew" does the synchronization needed so that it can run during the concurrent phases of CMS

### 老年代常用的垃圾回收器

1. SerialOld

   a stop-the-world(STW), mark-sweep-compact collector that uses a single GC thread

2. ParallelOld(PO)

   a stop-the-world(STW), a compacting collector which uses multiple GC threads

3. ConcurrentMarkSweep(CMS)

   1.4版本后期引入，是里程碑式的GC，它开启了并发回收的过程，但不成熟，因此目前没有任何一个JDK版本的默认垃圾回收器是CMS。CMS主要是缩短stop-the-world(STW)的时间，CMS回收器存在的最大的问题：因为CMS采用的是标记清除(Mark-Sweep)算法，所以会产生内存碎片，当从eden区进入老年区的对象由于碎片问题找不到空间的时候就会使用SerialOld进行标记压缩，但是由于SerialOld是单线程，所以在内存比较大的情况下停顿时间就会很长。这个方案的解决方案是:降低触发CMS的阀值，保持老年代有足够的空间：-XX:CMSInitiatingOccupancyFraction=70是指设定CMS在对内存占用率达到70%的时候开始GC(因为CMS会有浮动垃圾,所以一般都较早启动GC);

   CMS以下述四部实现a mostly concurrent, low-pause collector(并发的、低暂停的收集器)

   1. 初始标记(initial mark):标记根对象，这个时候是stop-the-world(STW)的，但是由于根对象比较少所以停顿的时间很短
   2. 并发标记(concurrent mark):和应用程序同时运行，即一边标记一边运行应用(这个时候由于应用在运行会不断的产生新的垃圾)，应用不会停只会慢一点
   3. 重新标记(remark):标记在并发标记过程中产生的新垃圾，这个时候是stop-the-world(STW)的，由于新产生的垃圾并不多所以这个停顿的时间也不会很长
   4. 并发清理(concurrent sweep):和应用程序同时运行，即一边清理一边运行应用(这个时候由于应用在运行会不断的产生新的垃圾：浮动垃圾)

### 不分代的垃圾回收器

1. G1(10ms)(逻辑上区分年轻代、老年代，物理上不区分)
2. ZGC(1ms)PK C++(不分代)
3. Shenandoah(不分代)
4. Eplison(debug用)

### 常见的垃圾回收器组合

1. Serial + CMS
2. Serial + SerialOld
3. ParNew + CMS
4. ParNew + SerialOld
5. PS + SerialOld
6. PS + ParallelOld

### 常见垃圾回收器组合参数设定

1. -XX:+UseSerialGC = Serial New(DefNew) + Serial Old

   小型程序。默认情况下不会是这种选项，HotSpot会根据计算及配置和JDK版本自动选择收集器

2. -XX:+UseParNewGC = ParNew + SerialOld

   这组合已经很少用(在某些版本中已经废弃)

3. -XX:+UseConc(urrent)MarkSweepGC = ParNew + CMS + Serial Old

4. -XX:+UseParallelGC = ParallelScavenge + Parallel Old

   1.8默认

5. -XX:+UseParallelOldGC = ParallelScavenge + SerialOld Old

6. -XX:+UseG1GC = G1

7. Linux服务器中使用java -XX:+PrintCommandLineFlags -version无法打印默认GC的类型，但是可以通过jmap查看jvm采用的垃圾收集器
   1.ps -ef|grep tomcat（获取tomcat的PID获得）
   2.查看java垃圾收集器 jmap -heap pid
   其中using thread-local object allocation下面就是采用的java垃圾收集器
   比如Mark Sweep Compact GC、Concurrent Mark-Sweep GC、Garbage-First (G1) GC等

## GC常用参数

### 通用参数

-Xmn -Xms -Xmx -Xss //年轻代 最小堆 最大堆 栈空间
-XX:+UseTLAB //使用TLAB，默认打开
-XX:+PrintTLAB //打印TLAB的使用情况
-XX:TLABSize //设置TLAB大小
-XX:+DisableExplictGC //System.gc()不管用 ，FGC
-XX:+PrintFlagsFinal //打印所有系统最终参数值
-XX:+PrintFlagsInitial //打印所有系统默认参数值
-XX:+PrintGC //输出GC日志
-XX:+PrintGCDetails //输出详细的GC日志信息
-XX:+PrintHeapAtGC //在GC的前后打印出堆的信息，
-XX:+PrintGCTimeStamps //输出GC的时间戳，以JVM启动时间为基准
-XX:+PrintGCDateStamps //输出GC的时间戳，以日期的形式。
-XX:+PrintGCApplicationConcurrentTime //打印应用程序时间
-XX:+PrintGCApplicationStoppedTime  //打印暂停时长
-XX:+PrintReferenceGC //记录回收了多少种不同引用类型的引用
-XX:+PrintVMOptions //打印显式参数
-XX:+PrintCommandLineFlags //打印显式隐式参数
-verbose:class //类加载详细过程
-Xloggc:/opt/log/gc.log //GC日志
-XX:MaxTenuringThreshold //升代年龄，最大值15
-XX:+UseSpining //开启自旋锁 
-XX:PreBlockSpin  //更改自旋锁的自旋次数，使用这个参数必须先开启自旋锁
-XX:CompileThreshold //通过JIT编译器，将方法编译成机器码的触发阀值，可以理解为调用方法的次数，例如调1000次，将方法编译为机器码
-XX:+DoEscapeAnalysis //逃逸分析
-XX:+EliminateAllocations //标量替换

### Parallel常用参数

-XX:SurvivorRatio  //新生代中Eden区域和Survivor区域(From区和To区)的比例，默认为8，就是Eden占新生代的8/10，From幸区和To幸存区各占新生代的1/10
-XX:PreTenureSizeThreshold //大对象到底多大
-XX:MaxTenuringThreshold   //该参数主要是控制新生代需要经历多少次GC晋升到老年代中的最大阈值
-XX:+ParallelGCThreads     //并行收集器的线程数，同样适用于CMS，一般设为和CPU核数相同
-XX:+UseAdaptiveSizePolicy //自动选择各区大小比例

### CMS常用参数

-XX:+UseConcMarkSweepGC                 // 使用CMS垃圾回收器
-XX:ParallelCMSThreads                  // CMS线程数量
-XX:CMSInitiatingOccupancyFraction      // 使用多少比例的老年代后开始CMS收集，默认是68%(近似值)，如果频繁发生SerialOld卡顿，应该调小。
-XX:UseCMSCompactAtFullCollection       // 在FGC时进行压缩
-XX:CMSFullGCsBeforeCompaction          // 多少次FGC之后进行压缩
-XX:CMSClassUnloadingEnabled            // 
-XX:CMSInitiatingPermOccupancyFraction  // 达到什么比例进行Perm回收
GCTimeRatio                             // 设置GC时间占用程序运行时间的百分比
-XX:MaxGCPauseMillis                    // 停顿时间，是一个建议时间，GC会尝试用各种手段达到这个时间，比如减少年轻代

### G1常用参数

-XX:UseG1GC                // 使用CMS垃圾回收器
-XX:MaxGCPauseMillis       // 建议值，G1会尝试调整Young区的块数来达到这个值
-XX:GCPauseIntervalMillis  // GC的间隔时间
-XX:+G1HeapRegionSize      // 分区大小，建议逐渐增大该值，1 2 4 8 16 32。随着size增加，垃圾的存活时间更长，GC间隔更长，但每次GC的时间也会更长
-XX:G1NewSizePercent       // 新生代最小比例，默认为5%
-XX:G1MaxNewSizePercent    // 新生代最大比例，默认为60%
-XX:GCTimeRatio            // GC时间建议比例，G1会根据这个值调整堆空间
-XX:ConcGCThreads          // 线程数量
-XX:InitiatingHeapOccupancyPercent  //启动G1的堆空间占用比例

# JVM&GC分析

## JVM参数

HotSpot参数分类：

1. 标准：-开头，所有的HotSpot都支持
2. 非标准:-X开头，特定版本HotSpot支持特定命令
3. 不稳定:-XX开头，下个版本可能取消
4. 默认参数值：java -XX:+PrintFlagsInitial
5. 最终参数值：java -XX:+PrintFlagsFinal -version
6. 查找对应的参数：java -XX:+PrintFlagsFinal -version | grep GC

## GC日志详解

样例代码：

```java
import java.util.LinkedList;
import java.util.List;

public class HelloGC {

	public static void main(String[] args) {
		System.out.println("HelloGC");
		List list = new LinkedList<>();
		for (;;) {
			byte[] bytes = new byte[1024 * 1024];
			list.add(bytes);
		}
	}
}
```

在linux环境javac编译后，执行以下命令：

java -Xmn10M -Xms40M -Xmx60M -XX:+PrintCommandLineFlags -XX:+PrintGC HelloGC

```
[GC (Allocation Failure) [DefNew: 7675K->263K(9216K), 0.0020013 secs] 7675K->7431K(39936K), 0.0020304 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
[Full GC (Allocation Failure) [Tenured: 50196K->50196K(51200K), 0.0013511 secs] 57608K->57596K(60416K), [Metaspace: 2510K->2510K(1056768K)], 0.0013692 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
GC类型：GC、FullGC
GC原因：Allocation Failure
产生的年代：年轻代(DefNew)
回收前后年轻代占用空间、年轻代总空间，以及回收时间：DefNew: 7675K->263K(9216K)
回收前后堆占用空间、以及总堆空间：7675K->7431K(39936K)，注意第一次回收的时候old区还没有对象所有堆占用的空间等于年轻代占用的空间
【Times: user=0.00 sys=0.00, real=0.00 secs】表示在linux中执行time ls命令分别在用户态、系统态以及总共花费的时间

heap dump部分：
Heap
 def new generation   total 9216K, used 7723K [0x00000000fc400000, 0x00000000fce00000, 0x00000000fce00000)
  eden space 8192K,  94% used [0x00000000fc400000, 0x00000000fcb8aec0, 0x00000000fcc00000)
  from space 1024K,   0% used [0x00000000fcd00000, 0x00000000fcd00000, 0x00000000fce00000)
  to   space 1024K,   0% used [0x00000000fcc00000, 0x00000000fcc00000, 0x00000000fcd00000)
 tenured generation   total 51200K, used 50196K [0x00000000fce00000, 0x0000000100000000, 0x0000000100000000)
   the space 51200K,  98% used [0x00000000fce00000, 0x00000000fff05098, 0x00000000fff05200, 0x0000000100000000)
 Metaspace       used 2541K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 275K, capacity 386K, committed 512K, reserved 1048576K
年轻代：def new generatio(total=eden space + 1个survivor)
老年代：tenured generation
元数据：Metaspace，class space指的是元数据区中专门存储类信息的空间
[0x00000000fc400000, 0x00000000fcb8aec0, 0x00000000fcc00000)
后面的内存地址指的是：起始地址，使用空间结束地址，整体空间结束地址
used 2541K, capacity 4486K, committed 4864K, reserved 1056768K
used 已经使用
capacity 总容量
committed 虚拟内存占用
reserved 虚拟内存保留
```

设定GC日志参数
-Xloggc:/opt/logs/gc-%t.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=1K -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause
例如：
java -Xmn10M -Xms40M -Xmx60M -XX:+PrintCommandLineFlags -Xloggc:/opt/logs/gc-%t.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=1K -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause HelloGC

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

## jconsole

1. 在linux服务器中使用如下参数启动应用

   java -Djava.rmi.server.hostname=192.168.56.110 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8999 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false HelloGC

2. 在windows环境中使用jdk自带的工具jconsole工具连接远程linux服务器(192.168.56.110:8999)

## jvisualvm

1. 远程分析

   1. 在linux服务器中使用如下参数启动应用

      java -Djava.rmi.server.hostname=192.168.56.110 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8999 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false HelloGC

   2. 在windows环境中使用jdk自带的工具jvisualvm工具连接远程linux服务器(192.168.56.110:8999 )

2. 本地分析

   1. 将linux服务器生成的dump文件下载的windows本地
   2. 打开C:\Program Files\Java\jdk1.8.0_144\bin目录下的jvisualvm.exe，点击文件==>装入

## jmap

1. 查看堆内存的配置和使用情况：jmap -heap 18230

   ```markdown
    Heap Configuration:
       MinHeapFreeRatio         = 0             //JVM堆缩减空间比率，低于则进行内存缩减
       MaxHeapFreeRatio         = 100           //JVM堆扩大内存空闲比例，高于则进行内存扩张 
       MaxHeapSize              = 994050048 (948.0MB)   //堆最大内
       NewSize                  = 20971520 (20.0MB)     //新生代初始化内存大小
       MaxNewSize               = 331350016 (316.0MB)   //新生代最大内存大小
       OldSize                  = 41943040 (40.0MB)     //老年代内存大小
       NewRatio                 = 2                     //新生代和老年代占堆内存比率
       SurvivorRatio            = 8                      //s区和Eden区占新生代内存比率
       MetaspaceSize            = 21807104 (20.796875MB)  //元数据初始化空间大小
       CompressedClassSpaceSize = 1073741824 (1024.0MB)     //类指针压缩空间大小
       MaxMetaspaceSize         = 17592186044415 MB       //元数据最大内存代销      
       G1HeapRegionSize         = 0 (0.0MB)             //G1收集器Region单元大小
    ​
    Heap Usage:
    PS Young Generation
    Eden Space: 
       capacity = 303038464 (289.0MB)             //Eden区总容量
       used     = 22801000 (21.744728088378906MB)  //Eden区已使用荣浪
       free     = 280237464 (267.2552719116211MB)   //Eden区剩余容量
       7.524127366221075% used                      //Eden区使用比例
    From Space:      //From区(也就是Survivor中的S1区)                             
       capacity = 13107200 (12.5MB)                    //S1区总容量大小
       used     = 5364536 (5.116020202636719MB)          //S1区已使用大小
       free     = 7742664 (7.383979797363281MB)           //S1区剩余大小
       40.92816162109375% used                       //S1使用比例
    To Space:      //To区 (也就是Survivor中的S2区)      
       capacity = 13631488 (13.0MB)              //S2区总容量大小
       used     = 0 (0.0MB)                     //S2区已使用大小
       free     = 13631488 (13.0MB)             //S2区剩余大小
       0.0% used                                //S2区使用比率
    PS Old Generation           
       capacity = 110624768 (105.5MB)           //老年代总容量大小
       used     = 49431224 (47.14128875732422MB) //老年代已使用大小
       free     = 61193544 (58.35871124267578MB) //老年代剩余大小
       44.68368602589973% used                   //老年代使用功能比例
   ```

2. jmap -histo pid | head -20 //使用jmap找到堆中占用内存量最大的20个类(查找有多少个对象产生)

3. jmap -dump:format=b,file=/opt/heapDump.hprof pid

   线上系统，内存特别大，堆转储文件命令：jmap -dump执行期间会对进程产生很大影响，甚至卡顿（所以线上系统慎用jmap），解决方案如下：

   1. 设定参数HeapDumpOnOutOfMemoryError，OOM的时候会自动产生堆转储文件（也会有影响，也不建议）

      比如：java -Xmn10M -Xms40M -Xmx60M -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/opt/java_heapdump.hprof HelloGC

      -XX:+HeapDumpOnOutOfMemoryError参数表示当JVM发生OOM时，自动生成DUMP文件。

      -XX:HeapDumpPath=${目录}参数表示生成DUMP文件的路径，也可以指定文件名称，例如：-XX:HeapDumpPath=${目录}/java_heapdump.hprof。如果不指定文件名，默认为：java_<pid>_<date>_<time>_heapDump.hprof。

   2. 很多服务器备份(高可用)，停掉这台服务器对其他服务器不影响（比较好的方式）

   3. 在线定位，比如阿里的arthas（在线定位很困难）

## jhat

1. jhat -J-mx512M xxx.hprof
2. jhat命令会在linux服务器上启动一个web服务，在windows通过链接：http://192.168.56.110:7000访问（拉倒最后：找到对应的链接）

## jstat

利用JVM内建的指令对Java应用程序的资源和性能进行实时的命令行的监控，包括了对Heap size和垃圾回收状况的监控。jstat -options 可以列出当前JVM版本支持的选项，常见的有

1. jstat -gc  pid   垃圾回收统计

   ```undefined
   - S0C：第一个幸存区的大小
   - S1C：第二个幸存区的大小
   - S0U：第一个幸存区的使用大小
   - S1U：第二个幸存区的使用大小
   - EC：伊甸园区的大小
   - EU：伊甸园区的使用大小
   - OC：老年代大小
   - OU：老年代使用大小
   - MC：方法区大小
   - MU：方法区使用大小
   - CCSC:压缩类空间大小
   - CCSU:压缩类空间使用大小
   - YGC：年轻代垃圾回收次数
   - YGCT：年轻代垃圾回收消耗时间
   - FGC：老年代垃圾回收次数
   - FGCT：老年代垃圾回收消耗时间
   - GCT：垃圾回收消耗总时间
   ```

2. jstat -gcutil pid  总结垃圾回收统计

   ```undefined
   S0：幸存1区当前使用比例
   S1：幸存2区当前使用比例
   E：伊甸园区使用比例
   O：老年代使用比例
   M：元数据区使用比例
   CCS：压缩使用比例
   YGC：年轻代垃圾回收次数
   FGC：老年代垃圾回收次数
   FGCT：老年代垃圾回收消耗时间
   GCT：垃圾回收消耗总时间
   ```

3. jstat -gcnew  pid  新生代垃圾回收统计

   ```undefined
   - S0C：第一个幸存区大小
   - S1C：第二个幸存区的大小
   - S0U：第一个幸存区的使用大小
   - S1U：第二个幸存区的使用大小
   - TT:对象在新生代存活的次数
   - MTT:对象在新生代存活的最大次数
   - DSS:期望的幸存区大小
   - EC：伊甸园区的大小
   - EU：伊甸园区的使用大小
   - YGC：年轻代垃圾回收次数
   - YGCT：年轻代垃圾回收消耗时间
   ```

4. jstat -gccapacity pid 堆内存统计

   ```undefined
   NGCMN：新生代最小容量
   NGCMX：新生代最大容量
   NGC：当前新生代容量
   S0C：第一个幸存区大小
   S1C：第二个幸存区的大小
   EC：伊甸园区的大小
   OGCMN：老年代最小容量
   OGCMX：老年代最大容量
   OGC：当前老年代大小
   OC:当前老年代大小
   MCMN:最小元数据容量
   MCMX：最大元数据容量
   MC：当前元数据空间大小
   CCSMN：最小压缩类空间大小
   CCSMX：最大压缩类空间大小
   CCSC：当前压缩类空间大小
   YGC：年轻代gc次数
   FGC：老年代GC次数
   ```

5. jstat -gcmetacapacity pid   元数据空间统计

   ```undefined
   MCMN:最小元数据容量
   MCMX：最大元数据容量
   MC：当前元数据空间大小
   CCSMN：最小压缩类空间大小
   CCSMX：最大压缩类空间大小
   CCSC：当前压缩类空间大小
   YGC：年轻代垃圾回收次数
   FGC：老年代垃圾回收次数
   FGCT：老年代垃圾回收消耗时间
   GCT：垃圾回收消耗总时间
   ```

6. jstat -gcnewcapacity pid 新生代内存空间统计

   ```undefined
   NGCMN：新生代最小容量
   NGCMX：新生代最大容量
   NGC：当前新生代容量
   S0CMX：最大幸存1区大小
   S0C：当前幸存1区大小
   S1CMX：最大幸存2区大小
   S1C：当前幸存2区大小
   ECMX：最大伊甸园区大小
   EC：当前伊甸园区大小
   YGC：年轻代垃圾回收次数
   FGC：老年代回收次数
   ```

7. jstat -gcoldcapacity pid 老年代内存空间统计

   ```undefined
   OGCMN：老年代最小容量
   OGCMX：老年代最大容量
   OGC：当前老年代大小
   OC：老年代大小
   YGC：年轻代垃圾回收次数
   FGC：老年代垃圾回收次数
   FGCT：老年代垃圾回收消耗时间
   GCT：垃圾回收消耗总时间
   ```

8. jstat -gccause pid  最近一次GC统计和原因

9. jstat -gcold pid  老年代统计

10. jstat -class pid  类加载器

11. jstat -compiler pid  JIT

12. jstat -printcompilation pid  HotSpot编译统计
