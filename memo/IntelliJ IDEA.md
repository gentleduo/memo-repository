# 定义

集成开发环境（IDE，Integrated Development Environment）是用于提供程序开发环境的应用程序，一般包括代码编辑器，编译器，调试器和图形用户界面等工具，集成了代码编写功能，分析功能，编译功能，调试功能等一体化的开发软件服务。所有具备这一特性的软件或者软件套都可以叫集成开发环境。

# 页面展示

## 工具栏、菜单栏的设置

View==>Appearance

![image](assets\IntelliJ-IDEA-1.png)

## Main Menu

![image](assets\IntelliJ-IDEA-2.png)

Main Menu不小心被隐藏后，可通过搜索[main menu]，出现View|Appearrance MainMenu，右侧会有off/on状态选择，保持on高亮，就会恢复主菜单窗口。如下：

1.点击IDEA工具类中的搜索引擎（放大镜）或者双击shift键

![image](assets\IntelliJ-IDEA-3.png)

2.搜索main menu 出现View|Appearrance MainMenu,右侧会有off/on状态选择，保持on高亮，就会恢复主菜单窗口。

![image](assets\IntelliJ-IDEA-4.png)

## Toolbar

![image](assets\IntelliJ-IDEA-5.png)

## Navigation Bar

![image](assets\IntelliJ-IDEA-6.png)

## Tool Window Bars

![image](assets\IntelliJ-IDEA-7.png)

## Status Bar

![image](assets\IntelliJ-IDEA-8.png)

## Details in Tree Viewi

![image](assets\IntelliJ-IDEA-9.png)

## Members in Navigation Bar

![image](assets\IntelliJ-IDEA-10.png)

# Module

在Eclipse中我们有workspace和projec的概念，在IDEA中只有project和module的概念。

Eclipse中的workspace相当于IDEA中的Project

Eclipse中的Project相当于IDEA中的Module

在Intellij IDEA中project是最顶级的级别，次级别是Module，一个project下可以有多个module。

在Eclipse中可以在同一个窗口管理n个项目，这在Intellij IDEA中是无法做到的，Intellij IDEA提供的解决方案是打开多个项目示例，即：打开多个项目窗口，一个project打开一个window窗口。

IDEA这样设置的原因：

目前主流的大型项目都是分布式部署的，结构都是类似这种多Module的。这类项目一般是这样划分的，比如：积分模块、任务模块、活动模块等等，模块之间彼此可以相互依赖。这些Module之间都是处于同一个项目业务的模块，彼此之间是有不可分割的业务关系的。

# 常用设置

## 主题

File==>Settings==>Appearance & Behavior==>Appearance==>Theme

## 字体

File==>Settings==>Editor==>General==>Mouse Control==>Change font size with Ctrl + Mouse Wheel

## 鼠标悬浮在代码上有提示

File==>Settings==>Editor==>Code Editing==>Quick Documentation==>Show quick documentation on mouse move

## 自动导包和优化多余的包

File==>Settings==>Editor==>General==>Auto Import==>Java，勾选余下两项

Add unambiguous imports on the fly（自动导包）

Optimize imports on the fly（优化多余的包）

## 同一包下的类，超过指定个数的时候，导包合并为*

File==>Settings==>Code Style==>Java==>Imports

## 显示代码行数和代码之间的分隔符

File==>Settings==>Editor==>General==>Appearance

Show line numbers

Show method separators

## 忽略大小写进行提示

File==>Settings==>Editor==>General==>Code Completion==>Match case(取消勾选)

## 设置多个类不隐藏，多行显示

File==>Settings==>Editor==>General==>Editor Tabs

Appearance==>Show tabs in：Multiple rows

Closing Policy==>Tab limit：20

## 设置默认字体大小

File==>Settings==>Editor==>Font

Font(字体类型)、Size(大小)、Line height(行距)

## 修改代码注释的颜色

File==>Settings==>Editor==>Color Scheme==>Language Defaults==>Comments

## 修改类注释信息

File==>Settings==>Editor==>File and Code Templates==>Includes==>File Header

```java
/**
*  @Auther:duo
*  @Date: ${DATE} - ${MONTH} - ${DAY} - ${TIME}
*  @Description: ${PACKAGE_NAME}
*  @Version: 1.0
*/
```

## 设置文件的编码格式

File==>Settings==>Editor==>File Encodings

Global Encoding、Project Encoding、Default encoding for properties files

## 自动编译

File==>Settings==>Build, Execution, Deployment==>Compiler，勾选余下两项

Build project automatically

Compile independent modules in parallel

## 省电模式

在省电模式下代码检查和代码提升功能就没有了，所以如果突然代码提示功能没有去看看是不是勾选了  Power  Save  Mode

File==>Power Save Mode

## 代码显示结构

Split Right：水平展示

Split Down：垂直展示

## 导入jar包

File==>Project Structure==>Project Settings==>Libraries==>+==>Java

## 生成序列化版本号

File==>Settings==>Editor==>Inspections==>Serialization issues==>Serializable class without 'serialVersionUID'

设置完成后，在实现了Serializable接口的类中，按Alt+Enter就会提示是否添加serialVersionUID

# 常用快捷键

## 创建内容

Alt + Insert

## main方法

psvm

## 输出语句

sout

## 复制一行

Ctrl + d

## 删除一行

Ctrl + y

## 代码向上/向下移动

Ctrl + Shift + ↑ / ↓

## 搜索类

Ctrl + n

## 生成代码

Alt + Insert

## 导包、生成变量

Alt + Enter

## 单行注释或多行注释

Ctrl + / 或 Ctrl + Shift + /

## 重命名

Shift + F6（重命名类名）

## for循环

fori

## 代码块包围

Ctrl + Alt + t（选中想要包围的代码块，然后按Ctrl + Alt + t）

## 代码自动补全提示

一般自己设置：File==>Settings==>Keymap==>搜索Code Completion

Alt + /

## 代码字体放大和缩小的快捷键

一般自己设置：File==>Settings==>Keymap==>搜索font size

Decrease Font Size、Increase Font Size

## 代码一层一层调用的快捷键

一般自己设置：File==>Settings==>Keymap==>搜索navigate

Back、Forward

## 显示代码结构

Alt + 7

## 显示导航栏

Alt + 1

## 撤回

Ctrl + z

## REDO操作

Ctrl + Shift + z（如果跟搜狗输入法的快捷键冲突，可以选择将搜狗的快捷键取消）

# 代码模板

它的原理就是配置一些常用代码字母缩写，在输入简写时可以出现你预定义的固定模式的代码，使得开发效率大大提高，同时也可以增加个性化。最简单的例子就是在Java中输入sout会出现System.out.println

## Live Templates

可以做用户的个性化定制；用法：直接键入+回车

File==>Settings==>Editor==>Live Templates

```java
package org.gen;

import java.io.Serializable;

/**
 * @Auther:duo
 * @Date: 2022-12-24 - 12 - 24 - 19:45
 * @Description: org.gen
 * @Version: 1.0
 */

public class Test implements Serializable
{

    public static void main(String[] args) {

        // 直接输入【fori】就会生成一个for循环
        // fori
        for (int i = 0; i < ; i++) {
            
        }
    }
}

```

## Postfix Completion

只能用，不能修改；用法：通过【变量 + .后缀】的方式

File==>Settings==>Editor==>General==>Postfix Completion

```java
package org.gen;

import java.io.Serializable;

/**
 * @Auther:duo
 * @Date: 2022-12-24 - 12 - 24 - 19:45
 * @Description: org.gen
 * @Version: 1.0
 */

public class Test implements Serializable
{

    public static void main(String[] args) {
        
        int[] arr = new int[]{1,2,3,4};
        // 通过定义好的数组【.fori】就会生成针对于该数组的循环
        // arr.fori
        for (int i = 0; i < arr.length; i++) {
            
        }
    }
}
```

## 常用模板

### main

main或者psvm

### 输出语句

sout或者.sout

soutp：打印方法的形参

soutm：打印方法的名字

soutv：打印变量

### 循环

普通for循环：fori（正向）或者.fori（正向）.forr（逆向）

增强for循环：iter或者.for

### 条件判断

ifn或者.null：判断是否为null

inn或者.nn：判断不等于null

### 属性修饰符

prsf：private static final

psf：public static final

## 自定义代码模板

File==>Settings==>Editor==>Live Templates==>+==>Template Group

### 示例1：

Abbreviation:test

Description:创建一个测试方法

Template text:

```java
public void test$var1${
$END$
}
```

Define:java

在template中$$中的内容其实就是在定义光标的位置，定义多个$$相当于多个光标并且光标可以切换，用回车切换

### 示例2：

Abbreviation:pri

Description:生成一个私有的int类型的变量

Template text:

```java
private int $var1$ ; // $var2$
```

Define:java

### 示例3：

Abbreviation:/**

Description:方法前的多行注释

Template text:

```java
/**
 *功能描述：
 *@param:$param$
 *@return:$return$
 *@auther:$user$
 *@date:$date$ $time$
 */
```

Define:java

# 断点调试

File==>Settings==>Build, Execution, Deployment==>Debugger==>Java==>Transport

在windows环境下一般将次选项设置成：Shared memory

## 常用快捷键

F8：一步一步执行，不会进入任何方法

F7：一步一步执行，不会进入系统类库中的方法，但是会进入自定义的方法

Alt + Shift + F7：一步一步执行，会进入系统类库中的方法，也会进入自定义的方法

Shift + F8：跳出方法

Alt + F8：查看表达式的值

F9：进入到一个断点，如果没有下一个断点，就直接运行到程序结束

## 条件判断

![image](assets\IntelliJ-IDEA-11.png)
