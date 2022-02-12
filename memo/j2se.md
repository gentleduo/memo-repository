# 枚举

## JDK1.5之前定义枚举类型：

```java
package org.duo.enum01;

/**
 * 枚举类型：季节
 */
public class Season {

    // 属性：
    private final String seasonName;//季节名字
    private final String seasonDesc;//季节描述

    // 利用构造器对属性进行赋值操作：
    // 构造器私有化，外界不能调用这个构造器，只能Season内部自己调用
    public Season(String seasonName, String seasonDesc) {
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    // 提供枚举类的有限的、确定的对象：
    public static final Season SPRING = new Season("春天", "春暖花开");
    public static final Season SUMMER = new Season("夏天", "夏日炎炎");
    public static final Season AUTUMN = new Season("秋天", "硕果累累");
    public static final Season WINTER = new Season("冬天", "冰天雪地");

    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

    @Override
    public String toString() {
        final StringBuilder sb = new StringBuilder("Season{");
        sb.append("seasonName='").append(seasonName).append('\'');
        sb.append(", seasonDesc='").append(seasonDesc).append('\'');
        sb.append('}');
        return sb.toString();
    }

}
```

## JDK1.5之后使用enum关键字创建枚举类：

```java 
package org.duo.enum02;

public enum Season {

    // 提供枚举类的有限的、确定的对象： 
    // ==> enum枚举类要求对象(常量)必须放在最开始的位置：相当于一组Season类型的对象
    // ==> 并且可以省略类型修饰符和new
    // ==> 多个对象之间用","进行连接，最后一个对象后面用";"结束
    // ==> 枚举类型的父类是：java.lang.Enum
    SPRING("春天", "春暖花开"),
    SUMMER("夏天", "夏日炎炎"),
    AUTUMN("秋天", "硕果累累"),
    WINTER("冬天", "冰天雪地");

    // 属性：
    private final String seasonName;//季节名字
    private final String seasonDesc;//季节描述

    // 利用构造器对属性进行赋值操作：
    // 构造器私有化，外界不能调用这个构造器，只能Season内部自己调用
    private Season(String seasonName, String seasonDesc) {
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

    @Override
    public String toString() {
        final StringBuilder sb = new StringBuilder("Season{");
        sb.append("seasonName='").append(seasonName).append('\'');
        sb.append(", seasonDesc='").append(seasonDesc).append('\'');
        sb.append('}');
        return sb.toString();
    }
}
```

在源码中经常可以看到别人定义的枚举类形态：

```java
package org.duo.enum03;

public enum Season {

    SPRING,
    SUMMER,
    AUTUMN,
    WINTER;
}
```

这是因为这个枚举底层没有属性，所以属性、构造器、get、toString方法都删掉不写了，然后按理来说应该写为MAN()，现在连()也省略了就变成了MAN

## Enum类的常用方法

```java
package org.duo.enum03;

public enum Season {

    SPRING,
    SUMMER,
    AUTUMN,
    WINTER;
}
```

```java
package org.duo.enum03;

public class TestSeason {

    public static void main(String[] args) {
        // 用enum关键字创建的Season枚举类型上面的父类是:java.lang.Enum，常用方法子类Season可以直接
        // toString；========> 获取对象的名字
        Season autumn = Season.AUTUMN;
        System.out.println(autumn);
        System.out.println("---------------------------------");
        // values；========> 返回枚举类型对象的数组
        Season[] values = Season.values();
        for (Season s : values) {
            System.out.println(s);
        }
        System.out.println("---------------------------------");
        // valueof；========> 通过对象名字获取枚举对象
        // 注意：对象的名字必须传正确
        Season autumn1 = Season.valueOf("AUTUMN");
        System.out.println(autumn1);
    }
}
```

## 枚举类实现接口

```java
package org.duo.enum04;

public interface TestInterface {
    void show();
}
```

```java
package org.duo.enum04;

public enum Season implements TestInterface{

    SPRING,
    SUMMER,
    AUTUMN,
    WINTER;

    @Override
    public void show() {
        System.out.println("这是Season类。");
    }
}
```

```java
package org.duo.enum04;

public class TestMain {

    public static void main(String[] args) {
        Season spring = Season.valueOf("SPRING");
        spring.show();
        Season winter = Season.valueOf("WINTER");
        winter.show();
    }
}
```

上述的写法所有的枚举对象调用show方法打印的结果都一样，如果想要不同的枚举对象实现各自的方法逻辑则可以使用以下的写法：

```java
package org.duo.enum04;

public enum Season implements TestInterface {

    SPRING {
        @Override
        public void show() {
            System.out.println("这是春天。");
        }
    },
    SUMMER {
        @Override
        public void show() {
            System.out.println("这是夏天。");
        }
    },
    AUTUMN {
        @Override
        public void show() {
            System.out.println("这是秋天。");
        }
    },
    WINTER {
        @Override
        public void show() {
            System.out.println("这是冬天。");
        }
    };

//    @Override
//    public void show() {
//        System.out.println("这是Season类。");
//    }
}
```

## 枚举的应用

```java 
package org.duo.enum05;

public enum Gender {

    男,
    女;
}
```

```java 
package org.duo.enum05;

public class Person {

    private int age;
    private String name;
    private Gender sex;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Gender getSex() {
        return sex;
    }

    public void setSex(Gender sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        final StringBuilder sb = new StringBuilder("Person{");
        sb.append("age=").append(age);
        sb.append(", name='").append(name).append('\'');
        sb.append(", sex=").append(sex);
        sb.append('}');
        return sb.toString();
    }
}
```

```java
package org.duo.enum05;

public class TestMain {

    public static void main(String[] args) {

        Person person = new Person();
        person.setAge(18);
        person.setName("lily");
        person.setSex(Gender.男); // 传入枚举类Gender的对象：===>在入口处对参数进行了限制
        System.out.println(person);
    }
}
```



# 注解

## JDK内置的3个注解

1. @Override:限定重写父类方法，该注解只能用于方法

   ```java
   package org.duo.anno;
   
   public class Person {
   
       public void eat() {
   
       }
   }
   ```

   ```java
   package org.duo.anno;
   
   public class Student extends Person{
   
       /**
        * @Override的作用，限定重写的方法，只要重写方法有问题，就有错误提示
        */
       @Override
       public void eat() {
           System.out.println("子类eat");
       }
   }
   ```

2. @Deprecated:用于表示所修饰的元素（类、方法、构造器、属性等）已过时，通常是因为所修饰的结构危险或存在更好的选择

   ```java
   package org.duo.anno;
   
   public class Student extends Person{
   
       /**
        * @Override的作用，限定重写的方法，只要重写方法有问题，就有错误提示
        */
       @Override
       public void eat() {
           System.out.println("子类eat");
       }
   
       /**
        * 在方法前加入@Deprecated，这个方法就会变成一个废弃方法/过期方法/过时方法
        */
       @Deprecated
       public void study() {
           System.out.println("学习。。。。");
       }
   }
   ```

   ```java
   package org.duo.anno;
   
   public class Test {
   
       public static void main(String[] args) {
   
           Student student = new Student();
           student.study();
       }
   }
   ```

3. @SuppressWarnings:抑制编译器警告

   ```java
   package org.duo.anno;
   
   public class Test {
   
       public static void main(String[] args) {
   
           Student student = new Student();
           student.study();
   
           @SuppressWarnings("unused")
           int age = 10;
       }
   }
   ```


## 自定义注解

```java
package org.duo.anno;

public @interface CustomAnnotation {

    /**
     *
     * 看上去是无参数方法，实际上理解为一个成员变量，一个属性
     * String[]：无参数方法的返回值 ==> 成员变量的名字
     * value()：无参数方法的名字 ==> 成员变量的类型
     * 这个参数叫配置参数
     *
     * 无参数方法的类型：基本数据类型、String、枚举、注解类型、以及以上类型对应的数组
     * PS：如果只有一个成员变量的话，名字尽量叫value
     *
     * 1.使用注解的话，如果你定义了配置参数，就必须给赋值操作
     * 2.如果只有一个参数，并且这个参数的名字为value的话，那么value=可以省略不写。
     * 3.如果配置参数设置了默认值，那么使用的时候可以无需传值
     * 4.一个注解的内部可以是不可以配置参数的，内部没有配置参数的注解叫标记，内部定义配置参数的注解叫元数据
     */
    String[] value();
}
```

```java
package org.duo.anno;

public @interface CustomAnnotation2 {

    String[] value() default "abc";
}
```

```java
package org.duo.anno;

public @interface CustomAnnotation3 {
}
```

```java
package org.duo.anno;

@CustomAnnotation2
//@CustomAnnotation(value = {"a", "b"})
@CustomAnnotation({"a", "b"})
public class Hero {
}
```

## 元注解

元注解是修饰注解的注解，JDK提供了四种元注解：Retention、Target、Documented、Inherited

### Retention

@Retention用于修饰注解，用于指定修饰的那个注解的生命周期，@Retention包含一个RetentionPolicy的枚举类型的成员变量，使用Retention时必须为该value成员变量指定值：

1. RetentionPolicy.SOURCE：在源文件中有效(即源文件保留)，编译器直接丢弃这种策略的注解，在.class文件中不会保留注解信息
2. RetentionPolicy.CLASS：在class文件中有效(即class保留)，保留在.class文件中，但是运行java程序时，就不会继续加载了，不会保留在内存中，JVM不会保留注解。如果注解没有加Retention元注解，那么相当于默认的注解就是这种状态。
3. RetentionPolicy.RUNTIME：在运行时有效(即运行时保留)，当运行java程序时，JVM会保留注解，加载在内存中了，那么程序可以通过反射获得该注解。

### Target

用于指定被修饰的注解能用于修饰哪些程序元素（即：被描述的注解可以用在什么地方），Target通过 java.lang.annotation.ElementType 来指定注解可使用范围的枚举集合

1. TYPE：类, 接口 (包括注释类型), 或枚举声明
2. FIELD：字段声明（包括枚举常量）
3. METHOD：方法声明(Method declaration)
4. PARAMETER：正式的参数声明
5. CONSTRUCTOR：构造函数声明
6. LOCAL_VARIABLE：局部变量声明
7. ANNOTATION_TYPE：注释类型声明
8. PACKAGE：包声明
9. TYPE_PARAMETER：类型参数声明
10. TYPE_USE：使用的类型

### Documented

用于指定被该元注解修饰的注解类将被javadoc工具提取成文档。默认情况下，javadoc是不包括注解的，但是加上了这个注解生成的文档中就会带着注解了。（使用得很少）

### Inherited

被它修饰的Annotation将具有继承性。如果某个类使用了被@Inherited修饰的Annotation，则子类将自动具有该注解。（使用得极少）

# 发射

## 发射的引入

考虑一个美团外卖在线支付的场景：可能美团会跟很多的支付机构合作，所以美团会规定一个支付的接口类：

```java 
package org.duo.reflect;

/**
 * 接口的指定方：美团外卖
 */
public interface Mtwm {

    // 在线支付功能：
    void payOnline();
}
```

各个支付机构分别实现：

```java 
package org.duo.reflect;

public class AliPay implements Mtwm{
    @Override
    public void payOnline() {
        System.out.println("我已经点了外卖，正在使用支付宝支付");
    }
}
```

```java 
package org.duo.reflect;

public class WeChat implements Mtwm {

    @Override
    public void payOnline() {
        System.out.println("我已经点了外卖，正在使用微信支付");
    }
}
```

```java 
package org.duo.reflect;

public class BankCard implements Mtwm {

    @Override
    public void payOnline() {
        System.out.println("我已经点了外卖，正在使用招商银行信用卡支付");
    }
}
```

最后由美团来调用：

```java 
package org.duo.reflect;

public class TestMain {

    public static void main(String[] args) {

        // 定义一个字符串，用来模拟前台的支付方式：
        String str = "支付宝";

        if ("微信".equals(str)) {
            // 微信支付
            pay(new WeChat());
        }

        if ("支付宝".equals(str)) {
            // 支付宝支付
            pay(new AliPay());
        }

        if ("招商银行".equals(str)) {
            // 招商银行支付
            pay(new BankCard());
        }
    }

    // 微信支付
    public static void pay(WeChat weChat) {
        weChat.payOnline();
    }

    // 支付宝支付
    public static void pay(AliPay aliPay) {
        aliPay.payOnline();
    }

    // 招商银行支付
    public static void pay(BankCard bankCard) {
        bankCard.payOnline();
    }
}
```

上面的写法代码扩展性太差，增加一种合作方式就要改代码，所以可以通过多态来修改：

```java 
package org.duo.reflect;

public class TestMain {

    public static void main(String[] args) {

        // 定义一个字符串，用来模拟前台的支付方式：
        String str = "支付宝";

        if ("微信".equals(str)) {
            // 微信支付
            pay(new WeChat());
        }

        if ("支付宝".equals(str)) {
            // 支付宝支付
            pay(new AliPay());
        }

        if ("招商银行".equals(str)) {
            // 招商银行支付
            pay(new BankCard());
        }
    }

    public static void pay(Mtwm mtwm) {
        mtwm.payOnline();
    }
}
```

多态确实可以提高代码的扩展性，但是扩展性没有达到最好：比如增加了合作机构还是要改代码；可以利用发射机制进行改进

```java 
package org.duo.reflect;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class ReflectDemo {

    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InstantiationException, IllegalAccessException, InvocationTargetException {
        // 定义一个字符串，用来模拟前台的支付方式：
        String str = "org.duo.reflect.AliPay"; // 字符串：实际上就是支付宝的全限定路径
        //下面的代码就是利用发射：
        Class clazz = Class.forName(str);
        Object o = clazz.newInstance();
        Method method = clazz.getMethod("payOnline");
        method.invoke(o);
    }
}
```

## 获取字节码信息的四种方式

```java 
package org.duo.reflect02;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.ElementType.LOCAL_VARIABLE;

@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String value();//属性
}
```

```java
package org.duo.reflect02;

public interface MyInterface {

    void myMethod();
}

```

```java 
package org.duo.reflect02;

import java.io.Serializable;

/**
 * 作为父类
 */
public class Person implements Serializable {

    //属性
    private int age;
    public String name;

    private void eat() {
        System.out.println("Person---eat");
    }

    public void sleep() {
        System.out.println("Person---sleep");
    }
}

```

```java 
package org.duo.reflect02;

/**
 * Student作为子类
 */
@MyAnnotation("hello")
public class Student extends Person implements MyInterface {

    //属性：
    private int sno;//学号
    double height;//身高
    protected double weight;//体重
    public double score;//成绩

    //方法；
    @MyAnnotation("himethod")
    public String showInfo() {
        return "我是一名三好学生";
    }

    public String showInfo(int a, int b) {
        return "重载方法======我是一名三好学生";
    }

    private void work() {
        System.out.println("我以后会找工作-->成为码农");
    }

    void happy() {
        System.out.println("做人最重要的就是开心每一天");
    }

    protected int getSno() {
        return this.sno;
    }

    //构造器：
    public Student() {
        System.out.println("空参构造器");
    }

    public Student(double weight, double height) {
        this.weight = weight;
        this.height = height;
    }

    private Student(int sno) {
        this.sno = sno;
    }

    Student(int sno, double weight) {
        this.sno = sno;
        this.weight = weight;
    }

    protected Student(int sno, double height, double weight) {
        this.sno = sno;
    }

    @Override
    @MyAnnotation("hellomyMethod")
    public void myMethod() {
        System.out.println("我重写了myMethod方法。。");
    }

    @Override
    public String toString() {
        final StringBuilder sb = new StringBuilder("Student{");
        sb.append("sno=").append(sno);
        sb.append(", height=").append(height);
        sb.append(", weight=").append(weight);
        sb.append(", score=").append(score);
        sb.append('}');
        return sb.toString();
    }
}
```

```java
package org.duo.reflect02;

public class TestMain {

    public static void main(String[] args) throws ClassNotFoundException {

        // 获取Person的字节码信息
        // 方式1：通过getClass()方法获取
        Person p = new Person();
        Class c1 = p.getClass();
        System.out.println(c1);

        // 方式2：通过内置Class属性：
        Class c2 = Person.class;
        System.out.println(c2);

        System.out.println(c1 == c2);
        // 方式3：调用Class类提供的静态方法forName
        Class c3 = Class.forName("org.duo.reflect02.Person");

        // 方式4：利用类的加载器
        ClassLoader classLoader = TestMain.class.getClassLoader();
        Class c4 = classLoader.loadClass("org.duo.reflect02.Person");

        // 方式1、2不常用，方式3用得最多，方式4了解即可
    }
}
```

## 可以作为Class类的实例的种类：

1. 类：外部类、内部类
2. 接口
3. 注解
4. 数组
5. 基本数据类型
6. void

```java
package org.duo.reflect02;

public class Demo {

    public static void main(String[] args) {

        // 类
        Class c1 = Person.class;
        // 接口
        Class c2 = Comparable.class;
        // 注解
        Class c3 = Override.class;
        // 数组
        int[] arr1 = {1, 2, 3};
        Class c4 = arr1.getClass();
        int[] arr2 = {5, 6, 7};
        Class c5 = arr2.getClass();
        System.out.println(c4 == c5); // 同一维度，同一元素类型，得到的字节码就是一个
        // 基本数据类型
        Class c6 = int.class;
        // void
        Class c7 = void.class;
    }
}
```

## 获取运行时类的完整结构

### 获取构造器和创建对象

```java
package org.duo.reflect02;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

public class Test01 {

    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {

        // 获取字节码信息：
        Class clazz = Student.class;
        // 通过字节码信息可以获取构造器：
        // getConstructors只能获取当前运行时类的被public修饰的构造器
        Constructor[] c1 = clazz.getConstructors();
        for (Constructor c : c1) {
            System.out.println(c);
        }
        System.out.println("--------------------------");
        // getDeclaredConstructors可以获取运行时类的全部修饰符的构造器
        Constructor[] c2 = clazz.getDeclaredConstructors();
        for (Constructor c : c2) {
            System.out.println(c);
        }
        System.out.println("--------------------------");
        // 获取指定的构造器：
        // 不传入参数：得到空构造器
        Constructor con1 = clazz.getConstructor();
        System.out.println(con1);
        // 得到两个参数的有参构造器
        Constructor con2 = clazz.getConstructor(double.class, double.class);
        System.out.println(con2);
        // 得到一个参数的有参构造器，并且是private修饰的
        Constructor con3 = clazz.getDeclaredConstructor(int.class);
        System.out.println(con3);
        System.out.println("--------------------------");
        // 拿到构造器后就可以创建对象了
        // 使用空构造器创建对象
        Object o1 = con1.newInstance();
        System.out.println(o1);
        System.out.println("--------------------------");
        Object o2 = con2.newInstance(1.3, 3.123);
        System.out.println(o2);
    }
}
```

### 获取属性和对属性进行赋值

### 获取方法和调用方法

### 获取类的接口，所在包，注解



