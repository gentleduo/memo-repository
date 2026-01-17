# 字符集和编码格式

**字符集（Charset）**：本质是「字符→唯一数字编号（码点 / Code Point）」的**映射表 / 字典**，只负责 “给每个字符分配一个独一无二的身份证号”，不关心这个 “身份证号” 如何转换成计算机能存储的二进制字节。

**编码格式（Encoding Format）**：本质是「字符集的数字码点→二进制字节流」的**转换规则 / 算法**，只负责 “把字符的身份证号转换成计算机可存储、可传输的二进制数据”，自身不负责给字符分配编号。

简单比喻：

- 字符集 = 全球公民身份证系统（给每个人分配唯一身份证号）；
- 编码格式 = 身份证号的 “快递打包规则”（把身份证号转换成可邮寄的包裹格式）；
- Unicode = 覆盖全球所有民族的 “统一身份证系统”；
- UTF-8 = 基于这套统一身份证系统的 “高效快递打包规则”。

## Unicode

全球统一字符集」（不是编码格式！）

Unicode 是一种**跨语言、跨平台的统一字符集（Charset）**，它的核心目标是「给全球所有语言的所有字符（中文、英文、日文、符号、表情等）分配一个唯一的数字码点（Code Point）」，解决传统字符集（如 ASCII、GB2312）覆盖范围有限、字符编号冲突的问题。

关键特点

- 「无编码能力」：Unicode 只做 “字符→码点” 的映射，不规定 “码点→二进制字节” 的转换规则。例如，它只规定 “中” 对应的码点是 `U+4E2D`（十六进制，对应十进制 20013），但不规定如何把 `4E2D` 转换成二进制字节存储到硬盘或传输到网络。
- 「覆盖范围极广」：目前已收录超过 14 万个字符，涵盖几乎所有已知语言和符号（包括 Emoji 表情）。
- 「码点表示格式」：通常用 `U+XXXX` 表示（XXXX 为十六进制数字，根据字符复杂度，码点长度可扩展，如`U+4E2D`、`U+1F600`（笑脸表情））。
- 「存储 / 传输的局限性」：由于没有统一的二进制转换规则，Unicode 本身无法直接用于计算机存储和传输，必须依赖后续的 “Unicode 编码格式”（如 UTF-8、UTF-16、UTF-32）才能落地。

## ASCII

ASCII（American Standard Code for Information Interchange，美国信息交换标准代码）是**最早的字符集之一，同时也是一种简单的编码格式（二合一）**，仅覆盖英文环境的基础字符。

关键特点

- 「二合一属性」：ASCII 既定义了 “字符→码点” 的映射（字符集），又直接规定了 “码点→二进制字节” 的转换规则（编码格式），且两者是直接对应的（无需复杂转换）。
  - 字符集范围：仅包含 128 个字符，包括英文字母（大小写）、数字（0-9）、基本标点符号（!、@、# 等）、控制字符（换行、回车、退格等）。
  - 编码规则：每个字符的码点直接用 **1 个字节（8 位二进制）** 存储，但只使用低 7 位（最高位固定为 0），因此取值范围是 `0x00`（0）~ `0x7F`（127）。例如：
    - 'A' 的码点是 65（十进制），对应二进制 `01000001`，编码后就是字节 `0x41`；
    - '0' 的码点是 48，对应二进制 `00110000`，编码后就是字节 `0x30`。
- 「局限性」：完全不支持中文、日文、韩文等非英文字符，仅适用于纯英文环境，这也是后续各种字符集 / 编码格式出现的原因。
- 「兼容性」：后续的几乎所有编码格式（如 UTF-8）都兼容 ASCII，即 ASCII 字符的编码结果和 ASCII 本身完全一致。

## GBK

GBK（Guo Biao Kuozhan，国标扩展）是**专门针对中文场景设计的 “字符集 + 编码格式” 二合一规范**，是 GB2312（仅支持简体中文）的扩展版本，广泛应用于中文 Windows 系统等中文环境。

关键特点

- 「二合一属性」：和 ASCII 一样，GBK 既定义了中文及相关字符的映射表（字符集），又规定了对应的二进制转换规则（编码格式），无需依赖其他字符集。
  - 字符集范围：覆盖简体中文、繁体中文、英文字母、数字、符号等，共收录超过 2 万个中文字符，满足日常中文使用需求。
  - 编码规则：采用**可变长度字节编码**，英文 / 数字等 ASCII 字符用「1 个字节」编码（兼容 ASCII），中文字符用「2 个字节」编码。例如：
    - 'A' 的编码结果是 `0x41`（和 ASCII、UTF-8 一致）；
    - “中” 的 GBK 编码结果是 `0xCAXC0`（对应字节流 `b'\xca\xc0'`）。
- 「局限性」：
  - 非跨平台、非国际化：仅在中文环境普及，在英文 / 其他语言系统中可能无法识别，不支持全球其他语言（如日文、阿拉伯文）；
  - 独立于 Unicode：GBK 的字符集和 Unicode 是两套独立的映射体系（但可以互相转换），它不是基于 Unicode 的编码格式。
- 「应用场景」：早期中文软件、中文 Windows 系统、部分中文文档（.txt）等。

## UTF-8

UTF-8（8-bit Unicode Transformation Format）是**目前全球最通用的编码格式**，它本身不是字符集，而是「基于 Unicode 字符集」的一种高效二进制转换规则，负责将 Unicode 的任意码点转换成可存储 / 传输的二进制字节流。

关键特点

- 「仅编码格式」：UTF-8 不负责给字符分配码点，它的唯一作用是 “将 Unicode 已经分配好的码点，转换成二进制字节”，依赖 Unicode 字符集存在。

- 「可变长度字节编码」：根据字符的 Unicode 码点大小，采用「1~4 个字节」进行编码，兼顾了存储空间和兼容性，这是它的核心优势：

  - 码点 `U+0000` ~ `U+007F`（ASCII 字符）：用 1 个字节编码，和 ASCII 编码完全一致（兼容 ASCII）；
  - 码点 `U+0080` ~ `U+07FF`（欧洲、中东等字符）：用 2 个字节编码；
  - 码点 `U+0800` ~ `U+FFFF`（中文、日文等常用多字节字符）：用 3 个字节编码；
  - 码点 `U+10000` ~ `U+10FFFF`（罕见字符、Emoji 表情）：用 4 个字节编码。

  示例：

  - 'A'（Unicode 码点 `U+0041`）：UTF-8 编码为 `0x41`（1 个字节，和 ASCII 一致）；
  - “中”（Unicode 码点 `U+4E2D`）：UTF-8 编码为 `0xE4B8AD`（3 个字节，对应字节流 `b'\xe4\xb8\xad'`）；
  - 笑脸表情 😀（Unicode 码点 `U+1F600`）：UTF-8 编码为 `0xF09F9880`（4 个字节）。

- 「核心优势」：

  - 跨平台、国际化：支持所有 Unicode 字符，可用于全球任何语言环境；
  - 节省存储空间：常用字符（英文、中文）采用较少字节编码，比固定长度的 UTF-32 更高效；
  - 兼容 ASCII：向下兼容传统 ASCII 格式，保障了旧系统的平滑过渡；
  - 无字节序问题：UTF-8 不需要考虑大端 / 小端存储（UTF-16 需要），使用更简单。

- 「应用场景」：网页（HTML）、数据库、大多数编程语言（Python/Java 默认）、网络传输（HTTP）等，是目前的绝对主流编码格式。

## 对比

| 规范名称 | 核心属性                    | 核心作用                                    | 覆盖范围                                     | 关键特点 / 局限                              |
| -------- | --------------------------- | ------------------------------------------- | -------------------------------------------- | -------------------------------------------- |
| Unicode  | 字符集（无编码能力）        | 给全球所有字符分配唯一码点，解决编号冲突    | 几乎所有语言、符号、Emoji（14 万 + 字符）    | 无法直接存储 / 传输，需依赖 UTF-8 等编码格式 |
| ASCII    | 字符集 + 编码格式（二合一） | 解决英文环境的字符存储 / 传输问题           | 仅英文、数字、基本符号（128 个字符）         | 不支持非英文字符，是后续规范的兼容基础       |
| GBK      | 字符集 + 编码格式（二合一） | 解决中文环境的字符存储 / 传输问题           | 简 / 繁体中文、英文、数字（2 万 + 字符）     | 非国际化、跨平台差，独立于 Unicode           |
| UTF-8    | 编码格式（基于 Unicode）    | 将 Unicode 码点转换成二进制字节流，落地存储 | 所有 Unicode 字符（继承 Unicode 的覆盖范围） | 可变长度、兼容 ASCII、跨平台，目前全球主流   |

## 总结

**最核心区别**：Unicode 是「字符集（给字符分配编号）」，UTF-8 是「编码格式（转换编号为字节）」，ASCII 和 GBK 是「字符集 + 编码格式二合一」；

**依赖关系**：UTF-8 依赖 Unicode 存在，ASCII 和 GBK 独立于 Unicode（但可与 Unicode 互相映射）；

**兼容性**：UTF-8 兼容 ASCII，GBK 兼容 ASCII，但 UTF-8 和 GBK 互不兼容（编码 / 解码格式不匹配会乱码）；

**选型原则**：优先使用「Unicode 字符集 + UTF-8 编码格式」，满足跨平台、国际化需求，避免乱码问题；

# 编码和解码

## 编码（Encode）

编码的核心是 **“从人类可识别的字符（→ 先映射为 Unicode 码点）→ 转换为计算机可识别的二进制字节流”**，

编码的目的：将易读的字符转换为计算机可处理的字节数据，解决 “字符无法直接存储 / 传输” 的问题。

## 解码（Decode）

解码是编码的反向操作，核心是 **“从计算机存储 / 传输的二进制字节流→（通过指定编码格式）→ 还原为人类可识别的字符”**

解码的目的：将计算机处理的字节数据还原为易读的字符，供人类查看和使用

## python

Python 3 对字符和字节做了严格区分，这是编码解码的基础：

- `str`类型：存储**Unicode 码点对应的字符**（人类可识别，对应上述 “字符”），不直接对应二进制字节；
- `bytes`类型：存储**二进制字节流**（计算机可处理，对应上述 “二进制字节流”）；
- 编码和解码的核心，就是`str`和`bytes`之间的相互转换。

Python 中的编码（str → bytes）

```python
# 1. 定义一个包含多语言的字符串（str类型，存储Unicode字符）
my_str = "Hello 世界！"

# 2. 以UTF-8编码（推荐）：str → bytes
bytes_utf8 = my_str.encode("utf-8")
print("UTF-8编码结果：", bytes_utf8)  # 输出：b'Hello \xe4\xb8\x96\xe7\x95\x8c\xef\xbc\x81'
print("UTF-8编码类型：", type(bytes_utf8))  # 输出：<class 'bytes'>

# 3. 以GBK编码：str → bytes（仅支持中文/英文等，不支持部分特殊符号）
bytes_gbk = my_str.encode("gbk")
print("GBK编码结果：", bytes_gbk)  # 输出：b'Hello \xca\xc0\xbd\xe7\xa3\xa1'
print("GBK编码类型：", type(bytes_gbk))  # 输出：<class 'bytes'>

# 4. 错误示例：用ASCII编码中文（ASCII不支持中文，会抛出异常）
try:
    bytes_ascii = my_str.encode("ascii")
except UnicodeEncodeError as e:
    print("ASCII编码失败：", e)  # 输出：ASCII编码失败： 'ascii' codec can't encode characters in position 6-8: ordinal not in range(128)
```

Python 中的解码（bytes → str）

```python
# 沿用上面编码得到的字节流
my_str = "Hello 世界！"
bytes_utf8 = my_str.encode("utf-8")
bytes_gbk = my_str.encode("gbk")

# 1. 以UTF-8解码（对应UTF-8编码，正常还原）
str_utf8 = bytes_utf8.decode("utf-8")
print("UTF-8解码结果：", str_utf8)  # 输出：Hello 世界！

# 2. 以GBK解码（对应GBK编码，正常还原）
str_gbk = bytes_gbk.decode("gbk")
print("GBK解码结果：", str_gbk)  # 输出：Hello 世界！

# 3. 错误示例：编码和解码格式不一致（出现乱码）
# 用GBK解码UTF-8编码的字节流
str_error = bytes_utf8.decode("gbk")
print("格式不匹配解码结果（乱码）：", str_error)  # 输出：格式不匹配解码结果（乱码）： Hello 涓栫晫锛�
```

## java

Java 中编码解码的核心对象是：

- `String`类型：内部存储的是**Unicode 码点（具体为 UTF-16 编码的字符数组）**（人类可识别，对应上述 “字符”）；
- `byte[]`数组：存储**二进制字节流**（计算机可处理，对应上述 “二进制字节流”）；
- 编码和解码的核心，是`String`和`byte[]`之间的相互转换。

Java 中的编码（String → byte []）

```java
import java.io.UnsupportedEncodingException;

public class EncodeDemo {
    public static void main(String[] args) {
        // 1. 定义一个包含多语言的字符串（String类型，存储Unicode字符）
        String myStr = "Hello 世界！";

        try {
            // 2. 以UTF-8编码（推荐）：String → byte[]
            byte[] bytesUtf8 = myStr.getBytes("UTF-8");
            System.out.print("UTF-8编码结果：");
            for (byte b : bytesUtf8) {
                System.out.print(b + " "); // 输出：72 101 108 108 111 32 -28 -72 -83 -27 -101 -117 -17 -68 -65 
            }
            System.out.println();

            // 3. 以GBK编码：String → byte[]
            byte[] bytesGbk = myStr.getBytes("GBK");
            System.out.print("GBK编码结果：");
            for (byte b : bytesGbk) {
                System.out.print(b + " "); // 输出：72 101 108 108 111 32 -70 -61 -54 -64 -93 -95 
            }
            System.out.println();

        } catch (UnsupportedEncodingException e) {
            // 捕获不支持的编码格式异常
            e.printStackTrace();
        }
    }
}
```

Java 中的解码（byte [] → String）

```java
import java.io.UnsupportedEncodingException;

public class DecodeDemo {
    public static void main(String[] args) {
        String myStr = "Hello 世界！";

        try {
            // 先编码得到字节流
            byte[] bytesUtf8 = myStr.getBytes("UTF-8");
            byte[] bytesGbk = myStr.getBytes("GBK");

            // 1. 以UTF-8解码（对应UTF-8编码，正常还原）
            String strUtf8 = new String(bytesUtf8, "UTF-8");
            System.out.println("UTF-8解码结果：" + strUtf8); // 输出：Hello 世界！

            // 2. 以GBK解码（对应GBK编码，正常还原）
            String strGbk = new String(bytesGbk, "GBK");
            System.out.println("GBK解码结果：" + strGbk); // 输出：Hello 世界！

            // 3. 错误示例：编码和解码格式不一致（出现乱码）
            String strError = new String(bytesUtf8, "GBK");
            System.out.println("格式不匹配解码结果（乱码）：" + strError); // 输出：格式不匹配解码结果（乱码）： Hello 涓栫晫锛�

        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }
}
```

总结：

**编码核心**：`字符（Unicode码点）→ 二进制字节流`，解决 “字符存储 / 传输” 问题，Python 用`str.encode()`，Java 用`String.getBytes()`；

**解码核心**：`二进制字节流→ 字符（Unicode码点）`，解决 “字节还原为易读字符” 问题，Python 用`bytes.decode()`，Java 用`new String(byte[], charset)`；

**关键原则**：编码和解码格式必须一致，否则必出乱码，优先使用`UTF-8`编码（跨平台、支持所有字符）；

**语言差异**：Python 3 严格区分`str`和`bytes`，Java`String`内部是 UTF-16 编码的 Unicode，两者均以 “Unicode 码点” 作为字符存储的中间载体。

# python解释器执行流程

Python 是**解释型语言（更准确地说，是 “编译 + 解释” 混合型）**，执行.py 文件并非直接 “边读边执行”，而是经过「字节加载→源码解码→语法编译→字节码解释→垃圾回收」的完整流程，其中你关心的 “读取文件” 和 “解码” 是前两个紧密关联但又不同的阶段。

阶段 1：磁盘读取 → 内存原始字节流（无解码操作）

这是 Python 解释器执行文件的第一步，对应你说的 “将文件读到内存”，**此阶段完全没有解码过程**。

1. **执行逻辑**：Python 解释器通过操作系统的文件 I/O 接口，发起对.py 文件的读取请求，操作系统将磁盘上存储的.py 文件以 ** 原始二进制字节流（bytes）** 的形式加载到内存中。
2. **核心特点**：
   - 此时内存中存储的是和磁盘上完全一致的字节序列，不区分 “字符” 和 “数据”，和读取.jpg、.zip 等二进制文件的过程完全相同；
   - 没有任何编码 / 解码操作，Python 解释器此时只负责 “搬运” 字节，不理解字节的含义（不知道是 UTF-8 编码还是 GBK 编码）；
   - 举例：一个保存为 UTF-8 编码的`test.py`文件，内容是`print("Hello 世界")`，磁盘上存储的是`b'print("Hello \xe4\xb8\x96\xe7\x95\x8c")'`，加载到内存后，依然是这串原始字节，没有任何变化。

阶段 2：内存原始字节流 → Unicode 字符（核心解码阶段，必有！）

这是你关心的 “解码” 关键环节，**此阶段存在强制解码过程，且是 Python 解释器执行后续步骤的前提**。

1. **为什么必须解码**：Python 解释器无法直接处理原始字节流，它只能理解**Unicode 字符**（对应 Python 3 的`str`类型）—— 因为 Python 的语法规则（如关键字`print`、`def`、引号`""`、中文变量名等）都是基于字符定义的，而非基于字节。

2. **解码的执行逻辑**：Python 解释器会将阶段 1 加载到内存的原始字节流，按照**指定的编码格式**解码成 Unicode 字符，得到可识别的源码文本。

3. **解码格式的优先级规则（关键，决定解码是否成功）**：Python 解释器会按以下顺序查找解码格式，找到即停止，全程无默认编码（Python 3 有默认兜底，下文说明）：

   - 第一步：查找.py 文件内部的**编码声明注释**（文件开头的特殊注释），这是最高优先级。

     支持两种格式：

     1. 标准格式：`# -*- coding: 编码格式 -*-`（如`# -*- coding: utf-8 -*-`、`# -*- coding: gbk -*-`）
     2. 简化格式：`# coding=编码格式`（如`# coding=gbk`）

   - 第二步：如果文件无编码声明，Python 3 会按照**PEP 3120 规范**，默认使用`UTF-8`编码进行解码（Python 2 默认使用 ASCII，这是早期乱码的核心原因）。

4. **解码失败的后果**：如果指定的解码格式（声明或默认 UTF-8）与.py 文件的实际编码格式不一致，会直接抛出`SyntaxError: Non-UTF-8 code starting with...`异常，后续流程无法继续执行。

5. **举例**：

   - 若`test.py`实际是 UTF-8 编码，且无编码声明，解释器用默认 UTF-8 解码，将`b'print("Hello \xe4\xb8\x96\xe7\x95\x8c")'`还原为 Unicode 字符`print("Hello 世界")`；
   - 若`test.py`实际是 GBK 编码，无编码声明，解释器用 UTF-8 解码会失败，抛出异常；此时需在文件开头添加`# coding=gbk`，才能正常解码。

阶段 3：Unicode 源码 → 字节码（编译阶段，无编码相关操作）

Python 解释器对解码后的 Unicode 源码进行后续处理，不涉及编码 / 解码：

1. 先进行**语法分析**：检查源码的语法是否符合 Python 规范（如括号是否匹配、关键字是否正确使用等）；
2. 语法无误后，进行**编译**：将 Unicode 源码编译成 Python 特有的**字节码（Byte Code）**，字节码是一种介于源码和机器码之间的中间代码（不是二进制机器码，无法直接被 CPU 执行）；
3. 字节码会被存储在内存中，若开启了缓存（默认开启），还会生成`.pyc`文件（在`__pycache__`目录下），供下次执行时直接复用，跳过 “字节加载→解码→编译” 三个阶段，提升执行效率。

阶段 4：字节码 → 机器指令（解释执行阶段，无编码相关操作）

由 Python 虚拟机（PVM，Python Virtual Machine）负责解释执行字节码：

1. PVM 逐行读取内存中的字节码，将其翻译成 CPU 可识别的机器指令；
2. 操作系统执行机器指令，完成具体的功能（如打印、变量赋值、函数调用等）；
3. 此阶段只处理字节码和机器指令，与字符、编码、解码均无关联。

阶段 5：执行收尾 → 垃圾回收与资源释放

执行完所有字节码后，Python 解释器会启动垃圾回收机制（GC），回收执行过程中不再使用的内存资源（如未被引用的变量、对象），并关闭相关文件句柄，完成整个执行流程。

补充：一个直观的错误示例（解码失败）

1. 用记事本创建`test.py`，输入`print("Hello 世界")`，保存时选择「编码格式：GBK」；
2. 不添加任何编码声明，直接用 Python 3 执行该文件；
3. 结果：抛出`SyntaxError: Non-UTF-8 code starting with '\xca' in file test.py on line 1, but no encoding declared; see https://peps.python.org/pep-0263/ for details`；
4. 解决：在文件开头添加`# coding=gbk`，重新执行，正常输出结果，此时解释器按 GBK 格式解码，成功得到 Unicode 源码。