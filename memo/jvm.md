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

