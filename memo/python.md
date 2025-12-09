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



