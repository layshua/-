# 函数部分

## 一、函数

**（1）定义和调用**

**定义一个函数**：

```
def  函数名(参数):
	...
```

**调用**： 函数名 (参数)  
eg：

```
>>> def add(num1, num2):
		num = num1 + num2
		print(num)
>>> add(2, 3)
```

**（2）函数的返回值（return）**  
eg：

```
>>> def add(num1, num2):
		return (num1 + num2)
>>> print(add(2, 3))
```

**（3）形参和实参**

**形参**：形式参数（parameter）指的是函数创建和定义过程中小括号里的参数  
eg： def add(num1, num2): 中 num1, num2 是形参  
**实参**：实际参数（argument）指的是函数在调用过程中传递进去的参数  
eg：`add(2, 3)`中 2，3 是实参

**（4）关键字参数和默认参数**

**关键字参数**

```
add(num1 = 2, num2 = 3)
```

**默认参数**：在定义函数时就给到的参数  
eg:

```
>>> def SaySome(name = "小甲鱼", words = "让编程改变世界！"):
		print(name + '-->' + words)
>>> SaySome()
小甲鱼-->让编程改变世界！
```

**收集参数（可变参数）**：在参数前加 *  
eg：

```
>>> def test(*params):
		print('参数的长度是：', len(params))
		print('第二个参数是：', params[1])
>>> test(1, 'Percy', 9.99, 7, 5)
参数的长度是： 5
第二个参数是： Percy
```

**（5）函数和过程**

**函数（function）**：有返回值  
**过程（procedure）**：是简单、特殊并且没有返回值的

**python 只有函数，没有过程。**

**（6）变量的作用域（可见性）**

**局部变量**（local variable）：在函数里定义的参数以及变量  
**全局变量**（global variable）：函数外的参数以及变量。函数内可以去访问全局变量，但不要去修改全局变量。

**不到万不得已不要使用全局变量**！简洁的概括为：  
a）代码可读性变差  
b）代码安全性降低

**global 关键字**：在函数内使用，将函数内的局部变量变为全局变量

**（7）内嵌函数**

在函数中再定义一个函数

```
def function1():
	print('function1()被调用')
	def function2():
		print('function2()被调用')
	function2()
	return
```

注意：  
1. 内部函数`function2()`整个作用域都在外部函数`function1()`之内。  
2. 只能在`function1()`中调用`function2()`，在`function1()`之外，不能调用`function2()`。

**（8）闭包**

如果在一个内部函数里对外部作用域（但不是在全局作用域）的变量进行引用，那么内部函数就会被认为是闭包。

```
>>> def FunX(x):
		def FunY(y):
			return x * y
		return FunY

>>> i = FunX(8)
>>> i
<function FunX.<locals>.FunY at 0x03DCAF58>
>>> type(i)
<class 'function'>
>>> i(5)
>>> FunX(8)(5)
```

变量的存储是放在栈里边的，而列表是不存在栈中的可以被内置函数所引用

将 x 改成列表，即可运行

**`nonlocal`关键字**：修饰变量后，标识该变量是上一级函数中的局部变量

**（9）lambda 表达式**

运用`lambda`关键字，创建匿名函数

eg1:

```
>>> lambda x : 2 * x + 1
<function <lambda> at 0x031AAF58>
>>> g = lambda x : 2 * x + 1
>>> g(5)
```

eg2:

```
>>> lambda x, y : x + y
<function <lambda> at 0x031AAF58>
>>> h = lambda x, y : x + y
>>> h(3, 4)
```

**lambda 表达式的作用：**  
1.Python 写一些执行脚本时，使用`lambda`就可以省下定义函数过程，`lambda`使代码更加精简。  
2. 对于一些比较抽象并且整个程序执行下来只需要调用一两次的函数，使用`lambda`不需要考虑命名问题。  
3. 简化代码的可读性。

**（10）两个厉害的 BIF**

① **`filter()`**：过滤器

filter(function or None, iterable) --> filter object

eg1：

```
>>> filter(None, [1, 0, False, True])
<filter object at 0x034CE0A0>
>>> list(filter(None, [1, 0, False, True]))
[1, True]
```

eg2：

```
>>> def odd(x):
	return x % 2

>>> temp = range(10)
>>> show = filter(odd, temp)
>>> list(show)
[1, 3, 5, 7, 9]
```

结合`lambda`关键字，修改代码 eg2：

```
>>> list(filter(lambda x : x % 2, range(10)))
[1, 3, 5, 7, 9]
```

② **`map()`**：映射

map(func, *iterables) --> map object

将序列的每一个元素作为函数的参数进行运算加工，直到可迭代序列的每个元素都加工完毕，返回所有加工后的元素构成的新序列。

eg：

```
>>> list(map(lambda x : x * 2, range(10)))
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

## 二、递归

符合以下两个条件：  
1. 有调用函数自身的行为  
2. 有一个正确的返回条件（停止的条件）

eg：求 n 的阶乘

```
def factorial(n):
    if n == 1:
        return 1
    else:
        return n * factorial(n - 1)

num = int(input('请输入一个正整数：'))
print(num, '的阶乘是：', factorial(num))
```

运行：

```
请输入一个正整数：5
5 的阶乘是： 120
```

**eg：斐波那契数列**  
①运用迭代

```
>>> def fab1(n):
	a = [1, 1]
	for i in range(n - 2):
		result = a[i] + a[i + 1]
		a.append(result)
	print(a.pop())

>>> fab1(12)
```

②运用递归

```
>>> def fab2(n):
	if n < 1:
		return 0
	elif n == 1:
		return 1
	elif n == 2:
		return 1
	else:
		return fab2(n - 1) + fab2(n - 2)

>>> fab2(12)
```

递归所占内存空间大，运算慢

**eg：求解汉诺塔**

```
>>> def hanoi(n, x, y, z):
	if n == 1:
		print(x, '->', z)
	else:
		hanoi(n - 1, x, z, y)# 将前n-1个盘子从x移动到y上
		print(x, '->', z)# 将最底下的最后一个盘子从x移动到z上
		hanoi(n - 1, y, x, z)# 将y上的n-1个盘子从y移动到z上

>>> hanoi(3, 'x', 'y', 'z')
x -> z
x -> y
z -> y
x -> z
y -> x
y -> z
x -> z
```