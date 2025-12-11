# 虚拟环境

1. venv
   1. 使用 venv 模块（推荐，Python 3.3+ 内置）
   2. 进入你的项目目录：/path/to/your/project
   3. 创建虚拟环境：python -m venv myenv（myenv 是你为虚拟环境起的名字，可以自定义（如 venv, env, myproject_env 等）。
   4. 激活虚拟环境
      1. Windows (命令提示符或 PowerShell)：myenv\Scripts\activate
      2. macOS / Linux：source ./myenv/bin/activate
   5. 退出虚拟环境：deactivate
   6. 删除虚拟环境：直接删除整个虚拟环境文件夹即可（例如 myenv 文件夹）。
2. virtualenv
   1. virtualenv 是一个更早、功能更丰富的第三方工具，但现在 venv 通常已足够。
   2. 安装 virtualenv：pip install virtualenv
   3. 创建虚拟环境：virtualenv milvus_env
   4. 也可以指定Python版本：virtualenv -p python3.9 myenv
   5. 激活、使用、退出虚拟环境的步骤与venv完全相同。

# 脚本执行

在Python中，使用python命令执行脚本有两种主要方式：直接使用python后接脚本文件，以及使用python -m后接模块名。

1. 本质区别
   1. python script.py，script.py 被解释为一个文件系统路径
   2. python -m my_module，my_module 被解释为一个Python模块名。如果my_module是包，则执行其包下的`__main__.py`子模块

2. sys.path的区别

   1. sys.path是Python查找模块的路径列表。

   2. ```markdown
      project/
      ├── package_a/
      │   ├── __init__.py
      │   └── module_a.py
      └── package_b/
          ├── __init__.py
          └── module_b.py
      ```

      `__init__.py`

      ```python
      print("package_a's init")
      ```

      module_a.py

      ```python
      import sys
      
      print(sys.path);
      print("="*80)
      print(__name__)
      print(__package__)
      print("this is package_a's module_a")
      ```

      `__init__.py`

      ```python
      print("package_b's init")
      ```

      module_b.py

      ```python
      import sys
      
      print(sys.path);
      print("="*80)
      print(__name__)
      print(__package__)
      print("this is package_b's module_b")
      ```

   3. 在project目录下，执行python package_a/module_a.py时，Python解释器将脚本所在目录package_a加入sys.path的第0个位置

   4. python -m package_a.module_a时，Python解释器将执行命令的当前目录project加入sys.path的第0个位置

   5. | 特性          | python                  | python -m                      |
      | ------------- | ----------------------- | ------------------------------ |
      | sys.path[0]   | 脚本所在目录            | 执行命令的当前目录             |
      | 相对导入      | 不支持                  | 支持                           |
      | `__name__`    | `__main__`              | `__main__`                     |
      | `__package__` | None                    | 模块的包名（例如 "package_a"） |
      | 包初始化      | 不执行包的`__init__.py` | 会执行包的`__init__.py`        |

# 内置变量

1. `__name__`

   1. 表示当前模块的名称
   2. 当模块被直接执行时：`__name__` 的值为 `"__main__"`
   3. 当模块被导入时：`__name__` 的值为包名+模块的文件名（不包括 .py 扩展名）
   4. `if __name__ == "__main__"`确保了只有在模块作为主程序运行时才会执行特定代码

2. `__file__`

   1. 包含当前模块的文件路径

   2. ```python
      print(f"当前文件路径: {__file__}")
      print(f"当前文件所在目录: {os.path.dirname(__file__)}")
      print(f"文件的绝对路径: {os.path.abspath(__file__)}")
      ```

   3. | 操作       | `os.path` 传统写法                  | 等价 Pathlib 写法                                   |
      | ---------- | ----------------------------------- | --------------------------------------------------- |
      | 拼路径     | `os.path.join(base, f"{name}.txt")` | `Path(base) / f"{name}.txt"`                        |
      | 取绝对路径 | `os.path.abspath(__file__)`         | `Path(__file__).resolve()`                          |
      | 创建文件夹 | `os.makedirs(p, exist_ok=True)`     | `Path('folder').mkdir(parents=True, exist_ok=True)` |

      >Pathlib 更短、可链式、跨平台自动适配。
      >
      >Pathlib 用 Path("foo") / "bar.txt" 自动帮你选对分隔符。

3. `__doc__`

   1. 包含模块、类、函数的文档字符串（docstring）

   2. ```python
      def calculate_area(radius):
          """计算圆的面积
          参数:
              radius (float): 圆的半径
          返回:
              float: 圆的面积
          """
          return 3.14 * radius ** 2
      
      print(calculate_area.__doc__)  # 输出文档字符串
      ```

4. `__all__`

   1. `__all__` 是一个特殊的模块级变量，用于定义模块的公共接口。它的主要作用是控制在使用 `from module import *` 时导入哪些名称

   2. ```python
      # mymodule.py
      __all__ = ['public_func', 'PublicClass']
      
      def public_func():
          return "This is public"
      
      def _private_func():
          return "This is private"
      
      class PublicClass:
          pass
      
      class _PrivateClass:
          pass
      ```

   3. ```python
      # 使用 from module import *
      from mymodule import *
      
      print(public_func())  # ✅ 可以访问
      print(PublicClass())  # ✅ 可以访问
      
      # print(_private_func())   # ❌ NameError: name '_private_func' is not defined
      # print(_PrivateClass())   # ❌ NameError: name '_PrivateClass' is not defined
      ```

   4. 通常用于包级别的`__init__.py`，而不是单个模块，比如：

   5. ```markdown
      calculator/
      ├── __init__.py
      ├── add.py
      ├── subtract.py
      └── internal.py
      ```

   6. ```python
      def add(a, b):
          return a + b
      ```

   7. ```python
      def subtract(a, b):
          return a - b
      ```

   8. ```python
      def _multiply(a, b):
          return a * b
      ```

   9. ```python
      # 这里定义哪些模块可以被"公开导入"
      __all__ = ['add', 'subtract']
      
      # 也可以在这里导入模块（不过通常不推荐，因为__all__已经指定了）
      # from . import add, subtract
      ```

   10. ```python
       from calculator import *
       
       # 这些可以正常用
       print(add.add(2, 3))  # 输出: 5
       print(subtract.subtract(5, 3))  # 输出: 2
       
       # 这个会报错！因为internal.py不在__all__里
       try:
           print(internal._multiply(2, 3))
       except ImportError as e:
           print("错误：internal模块没有被公开导入，所以不能使用")
       ```

5. sys.path

   1. 是一个列表，包含Python解释器在导入模块时搜索的路径

   2. ```python
      import sys
      print("当前模块搜索路径:", sys.path)
      sys.path.append('/my/custom/path')  # 添加自定义路径
      ```

6. `__builtins__`

   1. 包含所有内置函数和变量的命名空间，如 len、range、str 等。

   2. ```python
      print("内置函数列表:", dir(__builtins__))
      print("len函数:", __builtins__.len)
      print("使用__builtins__调用len:", __builtins__.len([1, 2, 3]))
      ```

7. sys.exc_info()

   1. 获取当前异常信息，用于错误处理

   2. ```python
      import sys
      
      try:
          1 / 0
      except:
          exc_type, exc_value, exc_traceback = sys.exc_info()
          print("异常类型:", exc_type)
          print("异常值:", exc_value)
      ```

# 生成器

## Python的栈帧对象模型

Python中"一切皆对象"，特别是栈帧对象的堆分配特性，是掌握Python高级特性（如协程、元编程、调试技巧）的关键基础。这种设计让Python极其灵活，但也需要注意内存管理。

栈帧对象（Frame Objects）

```python
import inspect

def bar():
    # 获取当前栈帧
    frame = inspect.currentframe()
    print(f"Bar帧ID: {id(frame)}")
    print(f"Bar帧对象: {frame}")
    return frame  # 可以返回栈帧对象！

def foo():
    frame = bar()
    # 栈帧对象可以被传递和存储
    frames.append(frame)
    print(f"Foo结束，但bar的栈帧还被引用着")

# 存储栈帧对象的列表
frames = []
foo()

# 即使函数调用结束，栈帧对象还存在
print(f"存储的帧数: {len(frames)}")
print(frames[0].f_code.co_name)  # 输出: bar
```

字节码对象（Code Objects）

```python
def example():
    x = 10
    y = 20
    return x + y

# 获取函数的字节码对象
code_obj = example.__code__

print(f"字节码对象: {code_obj}")
print(f"常量: {code_obj.co_consts}")    # (None, 10, 20)
print(f"变量名: {code_obj.co_varnames}")  # ('x', 'y')
print(f"字节码: {code_obj.co_code}")     # 原始字节码
```

示例

```python
#1.python中函数的工作原理
import inspect
import dis

#python.exe会用一个叫做PyEval_EvalFramEx(c函数)去执行函数， 首先会创建一个栈帧(stack frame)
"""
python一切皆对象，栈帧对象， 字节码对象
当foo调用子函数bar，又会创建一个栈帧对象，
因为所有的对象都是分配在堆内存上，这就决定了栈帧可以独立于调用者存在
这跟java等静态语言有较大的区别，java中函数调用完成后整个栈会全部被销毁
"""

frame = None
def foo():
    bar()
def bar():
    global frame
    # 通过inspect获取方法的栈帧
    frame = inspect.currentframe()

# 可以通过dis.dis获取字节码
print(dis.dis(foo))

foo()
# 这里虽然foo()函数运行结束了，但是由于栈帧分配在堆内存中，并且将栈帧赋值给了全局变量，所有仍然可以获取函数的名字和调用者的名字
print(frame.f_code.co_name)
caller_frame = frame.f_back
print(caller_frame.f_code.co_name)
```

Python vs Java 函数调用对比

| 特性         | Python                  | Java           |
| :----------- | :---------------------- | :------------- |
| **栈帧存储** | 堆内存                  | 栈内存         |
| **生命周期** | 由GC管理                | 函数结束即销毁 |
| **可访问性** | 可被引用、传递          | 不可访问       |
| **协程支持** | 天然支持（可暂停/恢复） | 需要额外机制   |

## send

`send()` 方法是生成器对象的一个核心方法，它允许**双向通信**：在恢复生成器执行的同时，向生成器发送一个值。这个值会成为当前暂停的 `yield` 表达式的返回值。

```python
def simple_generator():
    print("生成器开始执行")
    x = yield "第一次暂停"  # yield 作为表达式，接收 send 的值
    print(f"收到 send 的值: {x}")
    y = yield "第二次暂停"
    print(f"收到 send 的值: {y}")
    return "生成器结束"

gen = simple_generator()

# 启动生成器
initial_value = next(gen)  # 或 gen.send(None)
print(f"初始值: {initial_value}")

# 发送第一个值
result1 = gen.send(100)
print(f"第二次返回: {result1}")

# 发送第二个值
try:
    result2 = gen.send(200)
except StopIteration as e:
    # 最终的返回值通过StopIteration的value获取
    print(f"生成器结束，返回值: {e.value}")
```

首次必须使用 next() 或 send(None)

```python
def must_start_with_next():
    print("开始")
    value = yield "准备接收"
    print(f"收到: {value}")

gen = must_start_with_next()

# # 错误的方式
# try:
#     gen.send(10)  # TypeError: can't send non-None value to a just-started generator
# except TypeError as e:
#     print(f"错误: {e}")

# 正确的方式
print(gen.send(None))  # 输出: "开始" 和 "准备接收"
try:
    gen.send(10)      # 输出: "收到: 10"
except StopIteration as e:
    print("生成器结束")
```

## throw

```python
def gen_func():
    try:
        yield "http://projectsedu.com"
    except Exception as e:
        print(e)
    yield 2
    yield 3
    return "bobby"

if __name__ == "__main__":
    gen = gen_func()
    print(next(gen))  # 输出: http://projectsedu.com
    print(gen.throw(Exception, "download error"))  # 引发异常，被 except 捕获，并输出2
    print(next(gen))  # 际输出 3
```

- **`next(gen)` 第一次调用**：
  - 生成器执行到第一个 `yield`，产出 `"http://projectsedu.com"`，并暂停。
  - `print(next(gen))` 输出：`http://projectsedu.com`。
- **`gen.throw(Exception, "download error")` 调用**：
  - 在生成器暂停点（即第一个 `yield` 处）引发 `Exception("download error")`。
  - 异常被 `try` 块捕获，执行 `except` 块：`print(e)` 输出 `download error`。
  - **关键点**：由于异常被处理（`except` 块执行完毕），生成器**继续执行**，并执行下一条语句 `yield 2`。
  - **重要行为**：`gen.throw()` 方法在异常被处理后，**会返回生成器的下一个值**（即 `yield 2` 的值 `2`）。但您的代码中没有打印这个返回值（`gen.throw(...)` 的返回值被忽略）。
- **`print(next(gen))` 调用**：
  - 生成器已经执行到 `yield 2` 位置（因为 `throw` 使生成器继续执行并到达了 `yield 2`）。
  - 因此，`next(gen)` 会跳过 `yield 2`，直接返回**下一个 `yield` 的值**，即 `3`。
  - `print(next(gen))` 输出：`3`。

## close

1. **主要目的**：`close()` 方法用于提前终止生成器，释放资源
2. **工作原理**：向生成器内部抛出 `GeneratorExit` 异常
3. **最佳实践**：
   - 在生成器中正确处理 `GeneratorExit` 异常
   - 使用 `try-finally` 确保资源被释放
   - 不要在 `GeneratorExit` 处理块中再 `yield` 值
   - 一旦生成器被关闭，不要再尝试迭代它
4. **适用场景**：
   - 处理大文件或网络流时提前中断
   - 数据库查询需要提前终止
   - 任何需要显式资源管理的生成器场景
5. `GeneratorExit` 处理块中 `yield` 值或 `return` 值外面都收不到。`GeneratorExit` 是一个特殊的异常，它**不是用来捕获和处理的错误**，而是一个**信号**，告诉生成器："请立即清理并退出，不要再尝试生成任何值。"

```python
def generator_example():
    try:
        yield 1
        yield 2
        yield 3
    except GeneratorExit as e:
        print(f"GeneratorExit 被捕获: {type(e).__name__}")
        # 这是一个信号，不是错误
        # 我们应该清理资源，然后结束
        
def test_yield_in_generatorexit():
    try:
        yield "正常值"
    except GeneratorExit:
        print("进入 GeneratorExit 处理")
        yield "尝试在 GeneratorExit 中 yield"  # 这里会引发 RuntimeError
        print("这行不会执行")

gen = test_yield_in_generatorexit()
print(next(gen))  # 输出: "正常值"
gen.close()
# 输出: "进入 GeneratorExit 处理"
# RuntimeError: generator ignored GeneratorExit

def generator_with_return():
    try:
        yield "第一个值"
        return "返回值"  # 这会变成 StopIteration 的 value
    except GeneratorExit:
        return "在 GeneratorExit 中的返回值"  # 这个值被丢弃

gen = generator_with_return()
print(next(gen))  # 输出: "第一个值"

# 当调用 close() 时
gen.close()
# 返回的值 "在 GeneratorExit 中的返回值" 被完全忽略
```

**原因分析**：

1. 当 `close()` 被调用时，Python 向生成器抛出 `GeneratorExit`
2. 如果生成器捕获了这个异常并尝试 `yield`，Python 会认为生成器"忽略"了关闭信号
3. 这会引发 `RuntimeError: generator ignored GeneratorExit`

## yield from

```python
final_result = {}

def sales_sum(pro_name):
    total = 0
    nums = []
    while True:
        x = yield
        print(pro_name+"销量: ", x)
        if not x:
            break
        total += x
        nums.append(x)
    return total, nums

"""

"""
def middle(key):
    """
    中间协程函数，用于处理销售数据统计
    while True循环作用是让middle协程能够持续处理多个数据集合，但是在main函数中，为每个key都创建了一个新的middle协程（m = middle(key)），然后预激它，接着发送数据，最后发送一个None结束sales_sum协程；并且在main函数已经不会再给这个协程发送数据了，所以这个协程就变成了一个悬挂的协程，无法结束。严格来说，这会造成资源泄露（协程未关闭），直到程序结束时，所有资源才会被回收。
    这里增加while True表示协程永远不会结束，所以不会抛出StopIteration，所以在main里面第二次m.send(None)的时候不需要进行异常处理
    
    参数:
        key (str): 销售数据的键值，用于标识不同的销售项目
        
    返回值:
        None: 该函数作为协程使用，通过yield from将控制权交给sales_sum函数，
              并将统计结果存储到final_result全局字典中
    """
    # 将传入的键值key对应的销售数据统计结果存储到全局字典final_result中
    # 使用yield from语法委托sales_sum生成器处理具体的销售数据收集和计算
    # yield from会自动处理sales_sum生成器与调用者之间的数据传递
    # 去掉while True
    final_result[key] = yield from sales_sum(key)
    # 当sales_sum生成器完成数据统计并返回(total, nums)元组后
    # 将结果存储在final_result字典中，并打印统计完成提示信息
    print(key+"销量统计完成！！.")

def main():
    data_sets = {
        "bobby牌面膜": [1200, 1500, 3000],
        "bobby牌手机": [28,55,98,108 ],
        "bobby牌大衣": [280,560,778,70],
    }
    for key, data_set in data_sets.items():
        print("start key:", key)
        m = middle(key)
        m.send(None) # 预激middle协程
        for value in data_set:
            m.send(value)   # 给协程传递每一组的值
        # m.send(None)
        # 去掉while True后，每个商品的统计完成后，协程结束，会抛出StopIteration，因此需要在main函数中捕获
        try:
            m.send(None)  # 发送结束信号
        except StopIteration:
            # 协程正常结束，这是预期的
            pass
    print("final_result:", final_result)

if __name__ == '__main__':
    main()
```

# 协程

## 协程的基本概念与核心原理

协程（Coroutine）本质上是一种用户态的轻量级线程，它不依赖操作系统调度，而是由程序自身控制执行切换。协程的核心特征包括状态保存、非抢占式调度和单线程并发执行。在Python中，协程通过async/await语法实现，允许函数在执行过程中暂停并恢复执行，从而实现非阻塞的并发效果。

状态保存机制是协程实现的关键。当协程遇到await关键字时，会保存当前的执行上下文（包括寄存器、栈帧和局部变量），然后让出控制权给事件循环。这种保存机制使得协程能够记住自己执行到哪个位置，当事件循环重新调度该协程时，可以从保存的位置继续执行  。与线程相比，协程的上下文保存在用户空间，无需切换到内核空间，因此上下文切换的代价极小。Python线程的切换需要保存和恢复整个线程的上下文（包括寄存器、栈指针等），每次切换需要消耗约2微秒的时间；而协程的切换仅需保存协程自身的状态，切换代价仅为纳秒级别，几乎可以忽略不计  。

非抢占式调度是协程的另一重要特征。协程必须通过await关键字主动让出控制权，而不是像线程那样被操作系统抢占。这意味着协程的调度更加确定性，避免了线程调度的不确定性带来的问题。但也意味着如果协程中没有使用await，一旦开始运行，它将一直占用CPU直到完成，这可能导致其他协程无法执行。因此，在编写协程时，应当在可能阻塞的I/O操作前使用await，以便让出CPU资源给其他协程。

协程的生命周期包括四个状态：Pending（未启动）、Running（正在执行）、Suspended（因await暂停）和Completed（执行完毕） 。每个协程被编译为生成器对象，通过这些状态的转换实现并发执行。在Python中，协程通过async def定义，返回一个协程对象，需要通过事件循环来执行。调用协程函数不会立即执行函数体，而是返回一个协程对象，必须显式启动才能执行。

## Python协程的演进历程

Python协程的实现经历了从生成器到async/await语法的演变，这一过程反映了Python社区对并发编程需求的不断深入理解。

**生成器时代（Python 2.5~3.4）**是协程在Python中的最早实现方式。通过yield关键字，函数可以暂停执行并返回值给调用方，然后在下次调用send()方法时从暂停的位置继续执行。这种方式虽然可以实现协程的基本功能，但需要手动管理协程的切换，使用起来较为复杂。

**async/await语法引入（Python 3.5）**是协程在Python中的重大进步。PEP 492提案引入了async和await关键字，使协程成为Python语言的原生特性  。这种语法使得协程的编写更加直观和简洁，开发者可以像编写同步代码一样编写异步代码，但通过await关键字在需要时让出控制权。

**asyncio生态完善（Python 3.7+）**进一步提升了协程的易用性和性能。Python 3.7版本引入了asyncio.run()作为协程的主入口点，简化了协程程序的启动过程  ；同时，async和await成为保留关键字，避免了与普通变量名的冲突。Python 3.11版本增加了任务组（TaskGroup）等新特性，使得协程的管理更加方便。

协程的演进历程体现了Python社区对并发编程的不断探索和优化。从早期的手动管理生成器协程，到引入async/await语法简化协程的编写，再到asyncio生态的完善，协程在Python中的实现越来越高效和易用。这一演进也反映了协程作为一种轻量级并发机制的优势，特别是在处理I/O密集型任务时的高效性。

## 事件循环与协程状态转换的工作流程

事件循环（Event Loop）是协程调度的核心，它负责管理协程的执行和切换。在Python中，asyncio库提供了事件循环的实现，它通过维护多个队列来管理协程的执行状态。

事件循环的核心组件包括任务队列（管理就绪协程）、回调队列、I/O观察器（如epoll）和定时器 32 。事件循环的工作流程可以分为以下几个步骤：

- 创建事件循环：通过asyncio.get_event_loop()获取当前事件循环，或通过asyncio.new_event_loop()创建新的事件循环。
- 注册协程任务：将协程对象包装为Task对象并加入事件循环的任务队列。
- 执行协程：从任务队列中取出协程执行，直到遇到await关键字或完成。
- 挂起协程：当协程遇到await时，保存当前执行上下文，将协程标记为挂起状态，并将控制权返回给事件循环。
- 监听I/O事件：事件循环通过I/O多路复用（如epoll、kqueue）监听文件描述符，等待I/O操作完成。
- 恢复协程：当被等待的I/O操作完成时，事件循环将对应的协程重新加入任务队列，等待下一次调度执行。

在这一过程中，协程的状态会从Running（执行中）变为Suspended（挂起），然后在I/O操作完成后又变为Running。事件循环不断循环执行这些步骤，实现了协程的并发执行。

协程与任务的关系是理解事件循环的关键。协程（Coroutine）是使用async def定义的函数，它返回一个协程对象；而任务（Task）是由事件循环管理的协程封装体，它表示协程的执行过程。当协程被事件循环执行时，它会被包装为Task对象，并加入任务队列。Task对象提供了更多功能，如异常处理、结果获取等，使得协程的管理更加方便。

协程切换的底层机制涉及Python解释器的字节码处理。当协程执行到await关键字时，CPython解释器会执行对应的字节码（如AWAIT），保存当前帧的上下文，并切换到事件循环 35 。事件循环通过维护任务队列和等待队列，实现了协程的并发执行。这种切换完全在用户态完成，无需内核参与，因此切换代价极小。

协程的唤醒机制基于Future对象。当协程执行到await future时，它会挂起并将自身与Future对象关联。当Future对象完成（如I/O操作完成）时，事件循环会触发相应的回调，将协程重新加入就绪队列，等待下一次调度执行。这种机制使得协程能够在等待I/O操作时主动让出CPU资源，待I/O操作完成后继续执行，实现了高效的资源利用。

## 协程与线程/进程的对比

| 特性       | 线程                            | 协程                     | 进程                        |
| ---------- | ------------------------------- | ------------------------ | --------------------------- |
| 调度方式   | 操作系统内核调度                | 程序自身调度（事件循环） | 操作系统内核调度            |
| 内存开销   | 较大（MB级）                    | 极小（KB级）             | 较大（MB级）                |
| 上下文切换 | 依赖内核，开销大                | 用户态切换，开销小       | 依赖内核，开销大            |
| 并发能力   | 受GIL限制，单线程内无法真正并行 | 单线程内可高效并发       | 可利用多核CPU并行           |
| 适用场景   | CPU密集型任务                   | I/O密集型任务            | 需要真正并行的CPU密集型任务 |

协程与线程的主要区别在于调度方式和资源消耗。线程的调度由操作系统负责，切换需要内核参与，因此切换代价较大；而协程的调度由程序自身负责，切换在用户态完成，代价极小。线程的上下文保存在任务控制块（TCB）中，每个线程占用的内存较大（通常为MB级）；而协程的上下文保存在协程对象中，占用的内存较小（通常为KB级）。这使得协程在资源利用上更加高效，特别是在处理大量并发连接时。

协程与进程的主要区别在于进程之间是隔离的，而协程共享同一个进程的内存空间。进程之间需要通过IPC（进程间通信）来共享数据，而协程可以直接共享数据，无需额外的通信机制。这使得协程在处理I/O密集型任务时更加高效，但也意味着协程不能像进程那样利用多核CPU并行执行  。

协程的局限性在于它无法利用多核CPU资源，因为协程本质是单线程的。对于CPU密集型任务，协程的优势不明显，甚至可能因为事件循环的调度开销而性能下降。解决这一问题的方法是结合多进程和协程，即在多个进程中运行协程事件循环，这样既能够利用多核CPU资源，又能够在每个进程中高效处理大量I/O操作 2 。

协程的优势在于处理I/O密集型任务时的高效性。例如，网络爬虫需要发送大量HTTP请求并等待响应，使用协程可以显著提高效率  。数据库操作通常涉及网络请求或磁盘I/O，使用协程可以避免线程阻塞，提高并发能力  。文件读写也是协程的典型应用场景，通过异步文件操作可以提高处理效率。

## 应用

协程在Python中的实际应用主要集中在I/O密集型任务处理上，如网络请求、文件读写和数据库操作等。下面的例子通过原生的yield实现协程、原生的epoll实现事件驱动，爬取获取百度，163，新浪网，搜狐等几个主流门户网站首页的数据，

linux版本：python3

```python
import socket
import select
import urllib.parse
import time
import ssl
import gzip
import io

class EventLoop:
    """事件循环类：极简稳定版，减少不必要的日志和操作"""
    def __init__(self, timeout=15):
        self.epoll = select.epoll()
        self.fd_map = {}  # fd -> (coro, events, create_time)
        self.results = []
        self.timeout = timeout  # 更长的超时，适配HTTPS

    def register(self, fd, coro, events):
        """安全注册：忽略重复注册"""
        if fd in self.fd_map:
            try:
                self.epoll.modify(fd, events | select.EPOLLERR | select.EPOLLHUP)
            except Exception:
                pass
        else:
            try:
                self.epoll.register(fd, events | select.EPOLLERR | select.EPOLLHUP)
                self.fd_map[fd] = (coro, events, time.time())
            except Exception:
                pass

    def unregister(self, fd):
        """安全注销：全异常捕获"""
        if fd in self.fd_map:
            try:
                self.epoll.unregister(fd)
            except Exception:
                pass
            del self.fd_map[fd]

    def _check_timeout(self):
        """超时清理：仅清理超时且无响应的fd"""
        now = time.time()
        timeout_fds = [fd for fd, (_, _, ct) in self.fd_map.items() if now - ct > self.timeout]
        for fd in timeout_fds:
            self.unregister(fd)
            try:
                sock = socket.fromfd(fd, socket.AF_INET, socket.SOCK_STREAM)
                sock.close()
            except Exception:
                pass

    def add_coroutine(self, coro):
        """添加协程：容错处理"""
        try:
            fd, events = next(coro)
            self.register(fd, coro, events)
        except StopIteration:
            pass
        except Exception:
            pass

    def run(self):
        """极简事件循环：减少不必要的操作"""
        try:
            while self.fd_map:
                self._check_timeout()
                if not self.fd_map:
                    break
                
                # 短超时，高频检查
                events = self.epoll.poll(0.5)
                for fd, event in events:
                    if fd not in self.fd_map:
                        continue
                    
                    coro, _, _ = self.fd_map[fd]

                    # 处理错误事件
                    if event & (select.EPOLLERR | select.EPOLLHUP):
                        self.unregister(fd)
                        try:
                            sock = socket.fromfd(fd, socket.AF_INET, socket.SOCK_STREAM)
                            sock.close()
                        except Exception:
                            pass
                        continue

                    # 执行协程
                    try:
                        next_fd, next_events = coro.send(event)
                        self.unregister(fd)
                        self.register(next_fd, coro, next_events)
                    except StopIteration:
                        self.unregister(fd)
                    except Exception:
                        self.unregister(fd)
                        try:
                            sock = socket.fromfd(fd, socket.AF_INET, socket.SOCK_STREAM)
                            sock.close()
                        except Exception:
                            pass
        finally:
            self.epoll.close()
            self.fd_map.clear()


def decompress_data(data, encoding):
    """解压数据：处理gzip/deflate压缩"""
    try:
        if encoding == 'gzip':
            with gzip.GzipFile(fileobj=io.BytesIO(data)) as f:
                return f.read()
        elif encoding == 'deflate':
            return zlib.decompress(data)
        else:
            return data
    except Exception:
        return data


def parse_http_response(response_data):
    """解析HTTP响应：分离头、体、编码"""
    header_end = response_data.find(b"\r\n\r\n")
    if header_end == -1:
        return 200, {}, response_data
    
    header_part = response_data[:header_end].decode('utf-8', errors='ignore')
    body_part = response_data[header_end+4:]
    
    # 解析状态码
    status_code = 200
    encoding = 'identity'
    headers = {}
    lines = header_part.split("\r\n")
    if lines:
        status_line = lines[0]
        if status_line.startswith("HTTP/"):
            try:
                status_code = int(status_line.split()[1])
            except Exception:
                pass
        
        # 解析响应头
        for line in lines[1:]:
            if ":" in line:
                key, value = line.split(":", 1)
                key = key.strip().lower()
                value = value.strip()
                headers[key] = value
                if key == 'content-encoding':
                    encoding = value.lower()
    
    # 解压数据
    body_part = decompress_data(body_part, encoding)
    return status_code, headers, body_part


def fetch_url(url, loop, max_redirects=2):
    """
    稳定版协程：兼容SSL/压缩/乱码，全异常捕获
    """
    if max_redirects <= 0:
        return
    
    sock = None
    ssl_sock = None
    fd = -1
    is_https = False
    try:
        # 1. 解析URL
        parsed_url = urllib.parse.urlparse(url)
        host = parsed_url.netloc or parsed_url.path
        path = parsed_url.path or '/'
        if parsed_url.query:
            path += f"?{parsed_url.query}"
        
        # 2. 识别HTTPS和端口
        if parsed_url.scheme == 'https':
            is_https = True
            port = parsed_url.port or 443
        else:
            port = parsed_url.port or 80
       
        print(f'host:{host},path:{path},port:{port},is_https:{is_https}')
        # 3. DNS解析（容错）
        try:
            addr_info = socket.getaddrinfo(host, port, socket.AF_INET, socket.SOCK_STREAM)[0]
            ip = addr_info[4][0]
        except Exception:
            raise Exception("DNS解析失败")
        
        # 4. 创建基础Socket（纯非阻塞）
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.setblocking(False)
        fd = sock.fileno()
        
        # 5. 非阻塞连接
        try:
            sock.connect((ip, port))
        except BlockingIOError:
            pass
        
        # 等待连接建立
        yield (fd, select.EPOLLOUT)
        
        # 6. HTTPS处理（兼容SSL异常）
        if is_https:
            # 兼容SSL版本和加密套件
            context = ssl.create_default_context()
            context.options |= ssl.OP_NO_TLSv1 | ssl.OP_NO_TLSv1_1  # 禁用旧版本
            context.set_ciphers('DEFAULT:@SECLEVEL=1')  # 降低安全级别，兼容更多网站
            context.check_hostname = False
            context.verify_mode = ssl.CERT_NONE
            
            # 非阻塞SSL握手
            ssl_sock = context.wrap_socket(sock, server_hostname=host, do_handshake_on_connect=False)
            fd = ssl_sock.fileno()
            while True:
                try:
                    ssl_sock.do_handshake()
                    break
                except ssl.SSLWantReadError:
                    yield (fd, select.EPOLLIN)
                except ssl.SSLWantWriteError:
                    yield (fd, select.EPOLLOUT)
                except Exception as e:
                    raise Exception(f"SSL握手失败: {e}")
            sock = ssl_sock  # 替换为SSL socket
        
        # 7. 发送请求（强制不压缩）
        request = (
            f"GET {path} HTTP/1.1\r\n"
            f"Host: {host}\r\n"
            f"Connection: close\r\n"
            f"User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0.0.0 Safari/537.36\r\n"
            f"Accept: text/html,*/*;q=0.8\r\n"
            f"Accept-Language: zh-CN,zh;q=0.9\r\n"
            f"Accept-Encoding: gzip, deflate\r\n"  # 允许压缩，后续解压
            f"\r\n"
        )
        sock.send(request.encode('utf-8'))
        
        # 等待数据可读
        yield (fd, select.EPOLLIN)
        
        # 8. 稳定读取数据（兼容SSL读取）
        response_data = b""
        read_start = time.time()
        while time.time() - read_start < 10:  # 10秒读取超时
            try:
                # SSL读取需要特殊处理
                if is_https:
                    chunk = ssl_sock.recv(4096)
                else:
                    chunk = sock.recv(4096)
                
                if not chunk:
                    break
                response_data += chunk
                
                # 提前终止：已获取完整页面
                if b"</html>" in response_data and len(response_data) > 1024:
                    break
            except ssl.SSLWantReadError:
                yield (fd, select.EPOLLIN)
            except ssl.SSLWantWriteError:
                yield (fd, select.EPOLLOUT)
            except BlockingIOError:
                time.sleep(0.001)
                continue
            except Exception:
                break
        
        if not response_data:
            raise Exception("未读取到任何数据")
        
        # 9. 解析响应（处理压缩）
        status_code, headers, body = parse_http_response(response_data)
        
        # 10. 处理重定向
        if status_code in [301, 302] and 'location' in headers:
            redirect_url = urllib.parse.urljoin(url, headers['location'])
            sock.close()
            loop.unregister(fd)
            coro = fetch_url(redirect_url, loop, max_redirects-1)
            try:
                next_fd, next_evt = next(coro)
                yield (next_fd, next_evt)
            except Exception:
                pass
            return
        
        # 11. 存储结果（解决乱码）
        content = body.decode('utf-8', errors='replace')  # 替换乱码字符
        loop.results.append({
            "url": url,
            "status_code": status_code,
            "length": len(content),
            "content": content,
            "is_https": is_https
        })
        print(f"✅ 爬取成功 {url} | 状态码: {status_code} | 长度: {len(content)} 字节")
    
    except Exception as e:
        print(f"❌ 爬取失败 {url}: {str(e)[:60]}")
    finally:
        # 安全清理资源
        if fd != -1:
            loop.unregister(fd)
        if ssl_sock:
            try:
                ssl_sock.close()
            except Exception:
                pass
        if sock and not ssl_sock:
            try:
                sock.close()
            except Exception:
                pass


if __name__ == "__main__":
    # 目标网站
    target_urls = [
        #"http://www.baidu.com",
        "https://www.jd.com",
        "https://www.163.com",
        "https://www.sina.com.cn",
        #"https://www.sohu.com"
        "https://www.vip.com"
    ]

    # 初始化事件循环
    event_loop = EventLoop(timeout=15)

    # 添加协程
    for url in target_urls:
        event_loop.add_coroutine(fetch_url(url, event_loop))

    # 启动爬取
    print("🚀 开始并发爬取...")
    event_loop.run()

    # 输出结果（清晰展示）
    print("\n===== 最终爬取结果 ======")
    if event_loop.results:
        newline = '\n'
        for i, res in enumerate(event_loop.results, 1):
            print(f"\n{i}. 原始URL: {res['url']}")
            print(f"   HTTPS: {res['is_https']} | 状态码: {res['status_code']}")
            print(f"   内容长度: {res['length']} 字符")
            # 预览前200个字符，替换换行
            preview = res['content'][:200].replace(newline, ' ')
            print(f"   内容预览: {preview}...")
    else:
        print("⚠️  未获取到任何有效数据")


```

windows：

```python
import socket
import select
import urllib.parse
import time
import ssl
import gzip
import io
import zlib  # 补充缺失的导入


# 自定义事件常量，替代epoll的EPOLLIN/EPOLLOUT
READ_EVENT = 1
WRITE_EVENT = 2


class EventLoop:
    """事件循环类：适配Windows，使用select替代epoll，修复超时清理逻辑"""
    def __init__(self, timeout=15):
        # Windows无epoll，改用select，保存socket而非fd
        # 数据结构调整：sock -> (coro, events, create_time, last_active_time)
        self.sockets = {}  
        self.results = []
        self.timeout = timeout  # 更长的超时，适配HTTPS

    def register(self, sock, coro, events):
        """安全注册：忽略重复注册，初始化最后活动时间"""
        current_time = time.time()
        if sock in self.sockets:
            # 重复注册时，更新事件和最后活动时间，保留创建时间
            self.sockets[sock] = (coro, events, self.sockets[sock][2], current_time)
        else:
            try:
                # 首次注册：创建时间=最后活动时间=当前时间
                self.sockets[sock] = (coro, events, current_time, current_time)
            except Exception:
                pass

    def unregister(self, sock):
        """安全注销：全异常捕获"""
        if sock in self.sockets:
            del self.sockets[sock]

    def _update_last_active(self, sock):
        """更新socket的最后活动时间（事件触发时调用）"""
        if sock in self.sockets:
            coro, events, create_time, _ = self.sockets[sock]
            self.sockets[sock] = (coro, events, create_time, time.time())

    def _check_timeout(self):
        """优化的超时清理：仅清理长时间无活动的socket"""
        now = time.time()
        timeout_socks = []
        
        for sock, (_, _, create_time, last_active_time) in self.sockets.items():
            # 两种情况判定为超时：
            # 1. 最后活动时间超过超时阈值（核心判断）
            # 2. 从未活动过，但创建时间超过2倍超时阈值（兜底）
            if (now - last_active_time > self.timeout) or (last_active_time == create_time and now - create_time > self.timeout * 2):
                timeout_socks.append(sock)
        
        for sock in timeout_socks:
            self.unregister(sock)
            try:
                sock.close()
                print(f"⏰ 清理超时socket：{sock.getpeername() if hasattr(sock, 'getpeername') else '未知地址'}")
            except Exception as e:
                print(f"⚠️  清理超时socket失败：{str(e)[:50]}")

    def add_coroutine(self, coro):
        """添加协程：容错处理"""
        try:
            sock, events = next(coro)
            self.register(sock, coro, events)
        except StopIteration:
            pass
        except Exception:
            pass

    def run(self):
        """适配Windows的事件循环：使用select替代epoll"""
        try:
            while self.sockets:
                self._check_timeout()
                if not self.sockets:
                    break
                
                # 拆分可读/可写socket列表（select需要明确区分）
                read_socks = []
                write_socks = []
                for sock, (_, events, _, _) in self.sockets.items():
                    if events & READ_EVENT:  # 使用自定义读事件
                        read_socks.append(sock)
                    if events & WRITE_EVENT:  # 使用自定义写事件
                        write_socks.append(sock)
                
                # Windows的select：参数为(可读列表, 可写列表, 异常列表, 超时时间)
                readable, writable, exceptional = select.select(read_socks, write_socks, self.sockets.keys(), 0.5)
                
                # 处理异常socket
                for sock in exceptional:
                    self.unregister(sock)
                    try:
                        sock.close()
                    except Exception:
                        pass
                    continue
                
                # 处理可读事件（更新最后活动时间）
                for sock in readable:
                    if sock not in self.sockets:
                        continue
                    self._update_last_active(sock)  # 关键：更新活动时间
                    coro, _, _, _ = self.sockets[sock]
                    self._handle_event(sock, coro, READ_EVENT)
                
                # 处理可写事件（更新最后活动时间）
                for sock in writable:
                    if sock not in self.sockets:
                        continue
                    self._update_last_active(sock)  # 关键：更新活动时间
                    coro, _, _, _ = self.sockets[sock]
                    self._handle_event(sock, coro, WRITE_EVENT)
        finally:
            # 清理所有socket
            for sock in self.sockets.keys():
                try:
                    sock.close()
                except Exception:
                    pass
            self.sockets.clear()

    def _handle_event(self, sock, coro, event):
        """统一处理事件的辅助函数"""
        try:
            # 执行协程，获取下一个要监听的socket和事件
            next_sock, next_events = coro.send(event)
            self.unregister(sock)
            self.register(next_sock, coro, next_events)
        except StopIteration:
            self.unregister(sock)
            try:
                sock.close()
            except Exception:
                pass
        except Exception:
            self.unregister(sock)
            try:
                sock.close()
            except Exception:
                pass


def decompress_data(data, encoding):
    """解压数据：处理gzip/deflate压缩"""
    try:
        if encoding == 'gzip':
            with gzip.GzipFile(fileobj=io.BytesIO(data)) as f:
                return f.read()
        elif encoding == 'deflate':
            return zlib.decompress(data)
        else:
            return data
    except Exception:
        return data


def parse_http_response(response_data):
    """解析HTTP响应：分离头、体、编码"""
    header_end = response_data.find(b"\r\n\r\n")
    if header_end == -1:
        return 200, {}, response_data
    
    header_part = response_data[:header_end].decode('utf-8', errors='ignore')
    body_part = response_data[header_end+4:]
    
    # 解析状态码
    status_code = 200
    encoding = 'identity'
    headers = {}
    lines = header_part.split("\r\n")
    if lines:
        status_line = lines[0]
        if status_line.startswith("HTTP/"):
            try:
                status_code = int(status_line.split()[1])
            except Exception:
                pass
        
        # 解析响应头
        for line in lines[1:]:
            if ":" in line:
                key, value = line.split(":", 1)
                key = key.strip().lower()
                value = value.strip()
                headers[key] = value
                if key == 'content-encoding':
                    encoding = value.lower()
    
    # 解压数据
    body_part = decompress_data(body_part, encoding)
    return status_code, headers, body_part


def fetch_url(url, loop, max_redirects=2):
    """
    稳定版协程：兼容Windows/SSL/压缩/乱码，全异常捕获
    """
    if max_redirects <= 0:
        return
    
    sock = None
    ssl_sock = None
    is_https = False
    try:
        # 1. 解析URL
        parsed_url = urllib.parse.urlparse(url)
        host = parsed_url.netloc or parsed_url.path
        path = parsed_url.path or '/'
        if parsed_url.query:
            path += f"?{parsed_url.query}"
        
        # 2. 识别HTTPS和端口
        if parsed_url.scheme == 'https':
            is_https = True
            port = parsed_url.port or 443
        else:
            port = parsed_url.port or 80
       
        print(f'host:{host},path:{path},port:{port},is_https:{is_https}')
        # 3. DNS解析（容错）
        try:
            addr_info = socket.getaddrinfo(host, port, socket.AF_INET, socket.SOCK_STREAM)[0]
            ip = addr_info[4][0]
        except Exception:
            raise Exception("DNS解析失败")
        
        # 4. 创建基础Socket（纯非阻塞）
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.setblocking(False)
        
        # 5. 非阻塞连接（Windows兼容）
        try:
            sock.connect((ip, port))
        except BlockingIOError:
            pass
        except Exception as e:
            # Windows下非阻塞连接可能抛出其他异常，兼容处理
            if "WSAEWOULDBLOCK" not in str(e):
                raise
        
        # 等待连接建立：返回socket和自定义写事件
        yield (sock, WRITE_EVENT)
        
        # 6. HTTPS处理（兼容SSL异常，消除废弃警告）
        if is_https:
            # 兼容SSL版本和加密套件（新方式，消除DeprecationWarning）
            context = ssl.create_default_context()
            # 替代废弃的 ssl.OP_NO_TLSv1 | ssl.OP_NO_TLSv1_1
            context.minimum_version = ssl.TLSVersion.TLSv1_2
            context.set_ciphers('DEFAULT:@SECLEVEL=1')  # 降低安全级别，兼容更多网站
            context.check_hostname = False
            context.verify_mode = ssl.CERT_NONE
            
            # 非阻塞SSL握手
            ssl_sock = context.wrap_socket(sock, server_hostname=host, do_handshake_on_connect=False)
            while True:
                try:
                    ssl_sock.do_handshake()
                    break
                except ssl.SSLWantReadError:
                    yield (ssl_sock, READ_EVENT)
                except ssl.SSLWantWriteError:
                    yield (ssl_sock, WRITE_EVENT)
                except Exception as e:
                    raise Exception(f"SSL握手失败: {e}")
            sock = ssl_sock  # 替换为SSL socket
        
        # 7. 发送请求（强制不压缩）
        request = (
            f"GET {path} HTTP/1.1\r\n"
            f"Host: {host}\r\n"
            f"Connection: close\r\n"
            f"User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0.0.0 Safari/537.36\r\n"
            f"Accept: text/html,*/*;q=0.8\r\n"
            f"Accept-Language: zh-CN,zh;q=0.9\r\n"
            f"Accept-Encoding: gzip, deflate\r\n"  # 允许压缩，后续解压
            f"\r\n"
        )
        sock.send(request.encode('utf-8'))
        
        # 等待数据可读：返回自定义读事件
        yield (sock, READ_EVENT)
        
        # 8. 稳定读取数据（兼容SSL读取）
        response_data = b""
        read_start = time.time()
        while time.time() - read_start < 10:  # 10秒读取超时（单批次读取超时，和全局超时区分）
            try:
                # SSL读取需要特殊处理
                chunk = sock.recv(4096)
                
                if not chunk:
                    break
                response_data += chunk
                
                # 提前终止：已获取完整页面
                if b"</html>" in response_data and len(response_data) > 1024:
                    break
            except ssl.SSLWantReadError:
                yield (sock, READ_EVENT)
            except ssl.SSLWantWriteError:
                yield (sock, WRITE_EVENT)
            except BlockingIOError:
                time.sleep(0.001)
                continue
            except Exception:
                break
        
        if not response_data:
            raise Exception("未读取到任何数据")
        
        # 9. 解析响应（处理压缩）
        status_code, headers, body = parse_http_response(response_data)
        
        # 10. 处理重定向
        if status_code in [301, 302] and 'location' in headers:
            redirect_url = urllib.parse.urljoin(url, headers['location'])
            sock.close()
            loop.unregister(sock)
            coro = fetch_url(redirect_url, loop, max_redirects-1)
            try:
                next_sock, next_evt = next(coro)
                yield (next_sock, next_evt)
            except Exception:
                pass
            return
        
        # 11. 存储结果（解决乱码）
        content = body.decode('utf-8', errors='replace')  # 替换乱码字符
        loop.results.append({
            "url": url,
            "status_code": status_code,
            "length": len(content),
            "content": content,
            "is_https": is_https
        })
        print(f"✅ 爬取成功 {url} | 状态码: {status_code} | 长度: {len(content)} 字节")
    
    except Exception as e:
        print(f"❌ 爬取失败 {url}: {str(e)[:60]}")
    finally:
        # 安全清理资源
        if ssl_sock:
            try:
                ssl_sock.close()
            except Exception:
                pass
        elif sock:
            try:
                sock.close()
            except Exception:
                pass
        # 注销socket
        if sock in loop.sockets:
            loop.unregister(sock)


if __name__ == "__main__":
    # 目标网站
    target_urls = [
        #"http://www.baidu.com",
        "https://www.jd.com",
        "https://www.163.com",
        "https://www.sina.com.cn",
        #"https://www.sohu.com"
        "https://www.vip.com"
    ]

    # 初始化事件循环
    event_loop = EventLoop(timeout=15)

    # 添加协程
    for url in target_urls:
        event_loop.add_coroutine(fetch_url(url, event_loop))

    # 启动爬取
    print("🚀 开始并发爬取...")
    event_loop.run()

    # 输出结果（清晰展示）
    print("\n===== 最终爬取结果 ======")
    if event_loop.results:
        newline = '\n'
        for i, res in enumerate(event_loop.results, 1):
            print(f"\n{i}. 原始URL: {res['url']}")
            print(f"   HTTPS: {res['is_https']} | 状态码: {res['status_code']}")
            print(f"   内容长度: {res['length']} 字符")
            # 预览前200个字符，替换换行
            preview = res['content'][:200].replace(newline, ' ')
            print(f"   内容预览: {preview}...")
    else:
        print("⚠️  未获取到任何有效数据")
```

