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



