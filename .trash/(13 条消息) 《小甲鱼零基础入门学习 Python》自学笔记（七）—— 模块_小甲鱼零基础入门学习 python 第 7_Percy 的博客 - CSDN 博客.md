# 模块

## 一、模块就是程序

*   容器 ---- > 数据的封装
*   函数 ---- > 语句的封装
*   类 ---- > 方法和属性的封装
*   模块 ---- > 模块就是程序

**导入模块：**  
（1）`import 模块名`  
（2）`from 模块名 import 函数名`  
（3）`import 模块名 as 新名字`

**模块的作用：**  
（1）封装组织 Python 的代码  
（2）实现代码的重用

## 二、`__name__ == '__main__'`

eg：  
在主程序中使用`__name__`变量：

```
>>> __name__
'__main__'
```

在模块中使用`__name__`变量：得到模块名

```
>>> import random as r
>>> r.__name__
'random'
```

## 三、搜索路径

搜索路径其实是一个列表，系统有默认的搜索路径列表。  
可以选择创建一个文件夹来保存你写的模块，但需要在搜索路径列表中添加这个文件夹的所在位置。  
添加可以采用`append()`方法。

## 四、包（package）

创建一个包：  
（1）创建一个文件夹，用于存放相关的模块，文件夹的名字即包的名字；  
（2）在文件夹中创建一个`__init__.py`的模块文件，内容可以为空；  
（3）将相关的模块放入文件夹中。

导入时：`import 包名.模块名`

## 五、Python 标准库

Python 标准库中包含一般任务所需要的模块

快速了解一个模块：

（1）`print(模块名.__doc__)`

```
>>> import timeit
>>> print(timeit.__doc__)
			# 得到balabala一大堆...（这里省略）
>>> dir(timeit)
['Timer', '__all__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '_globals', 'default_number', 'default_repeat', 'default_timer', 'dummy_src_name', 'gc', 'itertools', 'main', 'reindent', 'repeat', 'sys', 'template', 'time', 'timeit']
>>> timeit.__all__
['Timer', 'timeit', 'repeat', 'default_timer']			# 可以被外部调用的
```

*   当使用`from timeit import *`这种方法导入：导入的参数只有`timeit.__all__`中的参数，其他参数不导入。

```
>>> from timeit import *
>>> Timer
<class 'timeit.Timer'>
>>> gc
Traceback (most recent call last):
  File "<pyshell#8>", line 1, in <module>
    gc
NameError: name 'gc' is not defined
```

*   `__file__`：指明该模块源代码所在位置

```
>>> import timeit
>>> timeit.__file__
'C:\\Users\\zhuya\\AppData\\Local\\Programs\\Python\\Python38-32\\lib\\timeit.py'
```

（2）`help(模块名)`

```
>>> help(timeit)
			# 得到balabala一大堆...（这里省略）
```