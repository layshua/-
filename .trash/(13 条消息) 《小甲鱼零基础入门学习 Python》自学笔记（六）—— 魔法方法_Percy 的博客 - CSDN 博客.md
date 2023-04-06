# 魔法方法

## 一、Python 魔法方法详解

[https://fishc.com.cn/forum.php?mod=viewthread&tid=48793&extra=page%3D1%26filter%3Dtypeid%26typeid%3D403](https://fishc.com.cn/forum.php?mod=viewthread&tid=48793&extra=page=1&filter=typeid&typeid=403)

## 二、构造和析构

*   魔法方法总是被双下划线包围，例如：`__init__`方法
*   魔法方法是面向对象的 Python 的一切，如果你不知道魔法方法，说明你还没能意识到面向对象的 Python 的强大
*   魔法方法的 "魔力" 体现在它们总能够在适当的时候被自动调用

**（1）`__init__(self[, ...])`**  
在需要对对象进行初始化操作时，才会用到。  
必须 return None  
不能另加 return，会报错！

**（2）`__new__(cls[, ...])`**  
实例化对象时，第一个被调用的方法。  
默认执行，一般不需要重写。  
需要重写的情况：当继承一个不可变类型，又需要修改的时候，那么它的特性就显得尤为重要。

```
eg：
>>> class CapStr(str):
	def __new__(cls, string):
		string = string.upper()
		return str.__new__(cls, string)

	
>>> a = CapStr('Love')
>>> a
'LOVE'
```

这里的`CapStr`是要继承`str`，但是`CapStr`又要把输入的字符串变成大写，`str`没有这个性质，它是不能改变的，类`CapStr`就成了继承`str`且能把输入变大写的类

**（3）`__del__(self)`**  
析构器。  
当对象将要被销毁的时候，这个方法就会被调用。  
并非`del x`就相当于自动调用`x.__del__()`，`__del__`方法是当垃圾回收机制回收这个对象的时候调用的。  
eg：

```
>>> class C:
	def __init__(self):
		print('我是__init__方法，我被调用了！')
	def __del__(self):
		print('我是__del__方法，我被调用了！')

		
>>> c1 = C()
我是__init__方法，我被调用了！
>>> c2 = c1
>>> c3 = c2
>>> del c3
>>> del c2
>>> del c1
我是__del__方法，我被调用了！
```

要删除所有的实例化对象时，才会被调用`__del__`方法。

## 三、算术运算

算数运算符  
[https://fishc.com.cn/forum.php?mod=viewthread&tid=48793&extra=page%3D1%26filter%3Dtypeid%26typeid%3D403](https://fishc.com.cn/forum.php?mod=viewthread&tid=48793&extra=page=1&filter=typeid&typeid=403)

反运算  
[https://fishc.com.cn/forum.php?mod=viewthread&tid=48793&extra=page%3D1%26filter%3Dtypeid%26typeid%3D403](https://fishc.com.cn/forum.php?mod=viewthread&tid=48793&extra=page=1&filter=typeid&typeid=403)

eg：

```
>>> class Nint(int):
	def __rsub__(self, other):
		return int.__sub__(self, other)

	
>>> a = Nint(5)
>>> 2 - a
```

当执行反运算时，`a`是`self`，`2`是`other`。所以`2 - a`其实是 5-2，结果为 3。

## 四、属性访问

*   **`__getattr__(self, name)`**

定义当用户试图获取一个不存在的属性时的行为

*   **`__getattribute__(self, name)`**

定义当该类的属性被访问时的行为

*   **`__setattr__(self, name, value)`**

定义当一个属性被设置时的行为

*   **`__delattr__(self, name)`**

定义当一个属性被删除时的行为

eg：  
写一个矩形类，默认有宽和高两个属性；如果为一个叫 square 的属性赋值，那么说明这是一个正方形，值就是正方形的边长，此时宽和高都应该等于边长。

```
class Rectangle:
    def __init__(self, width=0, height=0):
        self.width = width
        self.height = height

    def __setattr__(self, name, value):
        if name == 'square':
            self.width = value
            self.height = value
        else:
            super().__setattr__(name, value)

    def getArea(self):
        return self.width * self.height
```

运行：

```
>>> r = Rectangle(4, 5)
>>> r.getArea()
>>> r.square = 7
>>> r.getArea()
```

## 五、描述符

描述符就是将某种特殊类型的类的实例指派给另一个类的属性。

**特殊类型**：至少要实现以下三个方法的其中一个。

*   **`__get__(self, instance, owner)`**

用于访问属性，返回属性的值

*   **`__set__(self, instance, value)`**

将在属性分配操作中调用，不返回任何内容

*   **`__delete__(self, instance)`**

控制删除操作，不返回任何内容

eg：

```
>>> class MyDecriptor:

	def __get__(self, instance, owner):
		print('getting...', self, instance, owner)

	def __set__(self, instance, value):
		print('setting...', self, instance, value)

	def __delete__(self, instance):
		print('deleting...', self, instance)

		
>>> class Test:
	x = MyDecriptor()

	
>>> t = Test()
>>> t.x
getting... <__main__.MyDecriptor object at 0x03CE2760> <__main__.Test object at 0x03FF6040> <class '__main__.Test'>
>>> t.x = '美少女'
setting... <__main__.MyDecriptor object at 0x03CE2760> <__main__.Test object at 0x03FF6040> 美少女
>>> del t.x
deleting... <__main__.MyDecriptor object at 0x03CE2760> <__main__.Test object at 0x03FF6040>
```

**Property 的原理：**

重写 MyProperty 类（作用与 Property 相同）：

```
>>> class MyProperty:
	def __init__(self, fget=None, fset=None, fdel=None):
		self.fget = fget
		self.fset = fset
		self.fdel = fdel
		
	def __get__(self, instance, owner):
		return self.fget(instance)
	
	def __set__(self, instance, value):
		self.fset(instance, value)
		
	def __delete__(self, instance):
		self.fdel(instance)

		
>>> class C:
	def __init__(self):
		self.s = None
		
	def getS(self):
		return self.s
	
	def setS(self, value):
		self.s = value
		
	def delS(self):
		del self.S
		
	x = MyProperty(getS, setS, delS)
```

运行：

```
>>> c = C()
>>> c.s = '美少女'
>>> c.s
'美少女'
>>> del c.s
>>> c.s
Traceback (most recent call last):
  File "<pyshell#52>", line 1, in <module>
    c.s
AttributeError: 'C' object has no attribute 's'
```

结果与 Property 相同

## 六、定制容器

**协议（Protocols）**

与其他编程语言中的接口很相似，它规定你哪些方法必须要定义。然而，在 Python 中的协议就显得不那么正式。事实上，在 Python 中，协议更像是一种指南。

**容器类型的协议**

*   如果说你希望定制的容器是不可变的话，你只需要定义`__len__()`和`__getitem__()`方法。
    
*   如果你希望定制的容器是可变的话，除了`__len__()`和`__getitem__()`方法，你还需要定义`__setitem__()`和`__delitem__()`两个方法。
    
    **容器类型**
    

![](<assets/1680774447290.png>)

eg：  

![](<assets/1680774447393.png>)

  
代码：

```
class CountList:
    def __init__(self, *args):
        self.values = [x for x in args]
        self.count = {}.fromkeys(range(len(self.values)), 0)
    
    def __len__(self):
        return len(self.values)

    def __getitem__(self, key):
        self.count[key] += 1
        return self.values[key]
```

运行：

```
>>> c1 = CountList(1, 3, 5, 7, 9)
>>> c2 = CountList(2, 4, 6, 8, 10)
>>> c1[4]
>>> c2[0]
>>> c1[4] + c2[3]
>>> c1.count
{0: 0, 1: 0, 2: 0, 3: 0, 4: 2}
>>> c2.count
{0: 1, 1: 0, 2: 0, 3: 1, 4: 0}
```

## 七、[迭代器](https://so.csdn.net/so/search?q=%E8%BF%AD%E4%BB%A3%E5%99%A8&spm=1001.2101.3001.7020)

**`iter()`**  
对于一个容器对象调用，得到迭代器  
**`next()`**  
迭代器返回下一个值（如果迭代器没有值可以返回了，那么就会抛出一个`StopIteration`的异常）

eg：

```
>>> string = 'FishC'
>>> it = iter(string)
>>> next(it)
'F'
>>> next(it)
'i'
>>> next(it)
's'
>>> next(it)
'h'
>>> next(it)
'C'
>>> next(it)
Traceback (most recent call last):
  File "<pyshell#7>", line 1, in <module>
    next(it)
StopIteration
```

**for 循环的原理：**

```
>>> string = 'FishC'
>>> it = iter(string)
>>> while True:
	try:
		each = next(it)
	except StopIteration:
		break
	print(each)

	
F
i
s
h
C
```

**迭代器的魔法方法：**

**（1）`__iter__()`  
（2）`__next__()`**

**eg：斐波那契数列**

```
>>> class Fibs:
	def __init__(self, n=10):
		self.a = 0
		self.b = 1
		self.n = n
	def __iter__(self):
		return self
	def __next__(self):
		self.a, self.b = self.b, self.a + self.b
		if self.a > self.n:
			raise StopIteration			# 用raise语句来引发一个异常
		return self.a

	
>>> fibs = Fibs()
>>> for each in fibs:
	print(each)

	

>>> fibs = Fibs(50)
>>> for each in fibs:
	print(each)
```

## 八、生成器

迭代器的一种实现

**生成器**：在普通的函数中加入`yield`语句。

**协同程序**：  
可以运行的独立函数调用，函数可以暂停或者挂起，并在需要的时候从程序离开的地方继续或者重新开始。

eg：

```
>>> def myGen():
	print("生成器被执行！")
	yield 1
	yield 2

	
>>> g = myGen()
>>> next(g)
生成器被执行！
>>> next(g)
>>> next(g)
Traceback (most recent call last):
  File "<pyshell#60>", line 1, in <module>
    next(g)
StopIteration
>>> for i in myGen():
	print(i)

	
生成器被执行！
```

eg：斐波那契数列

```
>>> def fibs():
	a = 0
	b = 1
	while True:
		a, b = b, a + b
		yield a

		
>>> for each in fibs():
	if each > 20:
		break
	print(each, end = ' ')

	
1 1 2 3 5 8 13
```

*   **列表推导式**：

```
>>> a = [i for i in range(20) if not(i % 2) and i % 3]
>>> a
[2, 4, 8, 10, 14, 16]
```

*   **字典推导式**：

```
>>> b = {i : i % 2 == 0 for i in range(10)}
>>> b
{0: True, 1: False, 2: True, 3: False, 4: True, 5: False, 6: True, 7: False, 8: True, 9: False}
```

*   **集合推导式**：

```
>>> c = {i for i in [1, 2, 2, 3, 4, 5, 5, 6, 7, 8, 4, 3, 1]}
>>> c
{1, 2, 3, 4, 5, 6, 7, 8}
```

*   **生成器推导式**：

```
>>> d = (i for i in range(10) if i % 2 == 0)
>>> d
<generator object <genexpr> at 0x03C06290>
>>> next(d)
>>> next(d)
>>> for each in d:
	print(each)
```

eg：

```
>>> sum((i for i in range(10) if i % 2 == 0))
>>> sum(i for i in range(10) if i % 2 == 0)
```

一对括号、两对括号，都可以！一对括号更加简洁！