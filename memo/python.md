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