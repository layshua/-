# 基础部分

## 一、数字类型

（1）整数

无限大

（2）浮点数

**精确计算浮点数**：`decimal`

```
import decimal
a = decimal.Decimal('0.1')
```

**E 记法**（科学计数法）  
0.00005

```
5e-05（也就是5*10^-5）
```

（3）复数

x = 1 + 2j 用浮点数存储  
**获取实部**：`real`

```
>>>x.real
1.0
```

**获取虚部**：`imag`

```
>>>x.imag
2.0
```

**运算**： + - * / %  
**`x // y`**： 地板除 确保得到一个整数结果（向下取整）  
x == (x // y) * y + (x % y)  
**`divmod(x, y)`**： 返回 (x // y, x % y)  
**`abs(x)`** ： 绝对值（如果是复数，返回模）  
**`int(x)`**： 取整数部分

```
>>>int('520')
>>>int(3.14)
>>>int(9.99)
```

**`float(x)`**：转成浮点数  
**`complex(re, im)`**： 转成复数

```
complex("1+2j")
```

**`pow(x, y)`**：计算 x^y  
**`x ** y`**：计算 x^y  
pow(x, y, z) == x ** y % z

```
>>>pow(2, 3, 5) == 2 ** 3 % 5
True
```

## 二、[布尔类型](https://so.csdn.net/so/search?q=%E5%B8%83%E5%B0%94%E7%B1%BB%E5%9E%8B&spm=1001.2101.3001.7020)

**`True`**：1 == True  
**`False`**：0 == False

定义为 False 的对象：None 和 False  
值为 0 的数字类型：0， 0.0， 0j， Decimal(0), Fraction(0, 1)  
空的序列和集合：’’, (), [], {}, set(), range(0)

**逻辑运算符**

and 与  
or 或  
not 非

**Python 中任何对象都能直接进行真值测试**  
（测试该对象的布尔类型值为 True 或者 False）  
用于 if 或者 while 语句的条件判断，也可以作为布尔逻辑运算符的操作数

**短路逻辑** or and  
从左往右，只有当第一个操作数的值无法确定逻辑运算的结果时，才对第二个操作数进行求值。

**运算符优先级**  

![](<assets/1680774386163.png>)

+, -, *, /, //, % > 比较运算符 > 逻辑运算符  
not > and > or

## 三、分支结构

**if 语句**  
（1）

```
if condition:
	statement(s)
```

（2）

```
if condition:
	statement(s)
else:
	statement(s)
```

（3）

```
if condition1:
	statement(s)
elif condition2:
	statement(s)
elif condition3:
	statement(s)
...
```

（4）

```
if condition1:
	statement(s)
elif condition2:
	statement(s)
elif condition3:
	statement(s)
...
else:
	statement(s)
```

（5）

```
条件成立时执行的语句 if condition else 条件不成立时执行的语句
```

## 四、循环结构

**（1）while 循环**

```
while condition:
	statement(s)
```

**`break`** 跳出循环（只能作用于一层循环体）  
**`continue`** 跳出本轮循环，回到循环开始的位置，继续判断条件（只能作用于一层循环体）  
**`else`** 当循环条件不再为真时，执行

**（2）for 循环**

```
for 变量 in 可迭代对象:
	statement(s)
```

eg:

```
>>>for each in 'FishC':
		print(each)
F
i
s
h
C
```

eg:

```
for i in range(10):		#range(10)是左闭右开区间[0, 10)
	print(i)
```

**`.len()`** ：获取一个对象的长度  
**`range()`** ：生成一个数字序列（参数只能是整数）  
range(stop) ———— 从 0 开始  
range(start, stop)  
range(start, step, stop)

## 五、列表

**1、创建**

```
r = [1, 2, 3, 4, 5, '上山打老虎']
```

元素：1，2，3，4，5，‘上山打老虎’  
下标：0，1，2，3，4 ，5  
下标：-6，-5，-4，-3，-2，-1

**2、列表切片**

```
>>> r[0:3]
[1, 2, 3]
>>> r[3:6]
[4, 5, '上山打老虎']
>>> r[:3]
[1, 2, 3]
>>> r[3:]
[4, 5, '上山打老虎']
>>> r[0:6:2]
[1, 3, 5]
>>> r[::-1]					#倒序输出
['上山打老虎', 5, 4, 3, 2, 1]
>>> r[:]					#整个列表
[1, 2, 3, 4, 5, '上山打老虎']
```

**3、方法**

**（1）增**

**`append()`**：添加一个元素，在列表末尾  
**`extend()`**：添加一个可迭代对象（加在列表最后一个元素后）

```
>>> s = [1, 2, 3, 4, 5]
>>> s[len(s):] = [6]				#和s.append(6)相同
>>> s
[1, 2, 3, 4, 5, 6]
>>> s[len(s):] = [7, 8, 9]			#和s.extend([7,8,9])相同
>>> s
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**`insert(a, b)`**：插入一个元素（第一个参数：位置下标，第二个参数：元素）

```
>>> s = [1, 3, 4, 5]
>>> s.insert(1, 2)
>>> s
[1, 2, 3, 4, 5]
```

**（2）删**

**`remove()`**：删除一个元素  
注意：

1.  如果列表中存在多个匹配的元素，只会删除第一个；
2.  如果指定的元素不存在，那么程序就会报错。

**`pop()`**：删除并返回最后一个元素  
**`clear()`**：清空列表

**（3）改**

**列表是可变的，字符串是不可变的。**

eg：

```
>>> r = [1, 2, 3, 4]
>>> r[0] = 5
>>> r
[5, 2, 3, 4]
>>> r[2:] = [6, 7]
>>> r
[5, 2, 6, 7]
```

**`sort()`**：从小到大排序 (sort() 中 reverse 默认为 False)  
**`reverse()`**：翻转列表中的元素  
①`r.sort()`  
②`r.reverse()`  
①+②：等同于`r.sort(reverse = True)`

**（4）查**

**`count()`** ： 查找某个元素在列表中有几个（参数：元素，返回值：数量）  
**`index()`** ： 查找某个元素的下标索引值（参数：元素，返回值：下标）  
当元素存在多个，返回第一个的下标索引值  
**`index(x, start, end)`** ：指定查找的开始和结束位置  
**`copy()`** ：拷贝一个列表 浅拷贝（shallow copy）

**4、加法和乘法**

**加法**：拼接  
**乘法**：重复列表内所有元素若干次

**5、嵌套列表**（创建二维列表，也就是矩阵）

**（1）访问**

```
>>> matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
>>> matrix[0]				#第一行
[1, 2, 3]
>>> matrix[1][1]			#第二行第二列
```

**（2）is 运算符（同一性运算符）**：  
检验两个变量是否指向同一个对象

①

```
>>> A = [0] * 3
>>> for i in range(3):
				A[i] = [0] *3				# √
>>> A
[[0, 0, 0], [0, 0, 0], [0, 0, 0]]
```

![](<assets/1680774386314.png>)

②

```
>>> B = [[0] * 3] *3						#× 有bug
>>> B
[[0, 0, 0], [0, 0, 0], [0, 0, 0]]
```

![](<assets/1680774386396.png>)

**检验**：

```
>>> A[0] is A[1]
False
>>> B[0] is B[1]
True
```

**6、浅拷贝和深拷贝**

**`y = x`** ： （引用）当 x 改变，y 也改变  
**`y = x.copy()`** ： 拷贝整个列表（浅拷贝）当 x 改变，y 不改变  
**`y = x[:]`**

**浅拷贝**：只拷贝了外层的对象，当包含嵌套列表时，只是引用

**copy 模块**:

**浅拷贝**：copy()

```
>>> import copy
>>> x = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
>>> y = copy.copy(x)
```

**深拷贝**：deepcopy()

```
>>> import copy
>>> x = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
>>> y = copy.deepcopy(x)
```

**7、列表推导式**

eg：

```
>>> oho = [1, 2, 3, 4, 5]
>>> oho = [i * 2 for i in oho]
>>> oho
[2, 4, 6, 8, 10]
```

**（1）**

```
[expression for target in iterable]
```

**`ord()`** : 将字符转换为对应的 Unicode 编码  
eg:

```
>>> code = [ord(c) for c in 'FishC']
>>> code
[70, 105, 115, 104, 67]
```

eg:

```
>>> matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
>>> col2 = [row[1] for row in matrix]
>>> col2
[2, 5, 8]
>>> diag = [matrix[i][i] for i in range(len(matrix))]
>>> diag
[1, 5, 9]
```

**改变列表内容时，用 for 循环和列表推导式是不完全一样的。**  
**for 循环**：通过迭代，逐个修改原列表中的元素。  
**列表推导式**：直接创建一个新的列表，再赋值给原来的变量。

**创建嵌套列表**：

```
s = [[0] * 3 for i in range(3)]
```

**（2）**

```
[expression for target in iterable if condition]
```

eg：

```
>>> even = [i for i in range(10) if i % 2 == 0]
>>> even
[0, 2, 4, 6, 8]
```

**（3）**

```
[expression for target in iterable1
for target in iterable2
...
for target in iterableN]
```

**（4）**

```
[expression for target in iterable1 if condition1
for target in iterable2 if condition2
...
for target in iterableN if conditionN]
```

## 六、元组

**不可以修改**

**1、创建**

```
tuple = (1, 2, 3, 4, 5, 6, 7, 8)
```

**2、访问**

```
>>> tuple[1]
>>> tuple[5:]
(6, 7, 8)
```

**元组的关键**：逗号

**创建空列表**：`temp = []`  
**创建空元组**：`temp = ()`  
**创建只有一个元素的元组**：eg：`tuple = (1,)` 或者`tuple = 1,`

**3、更新**

```
>>> temp = (3, 4, 6, 7)
>>> temp = temp[:2] + (5,) + temp[2:]		# (5,) 括号，逗号，缺一不可
>>> temp
(3, 4, 5, 6, 7)
```

**4、删除**  
删除整个元组：`del`

```
del temp
```

## 七、字符串

**1、字符串**

```
'abc' "abc" '''abc'''
```

**2、转义字符 (\)**

```
\'		\\	   \n（换行）等
```

`\`放在一行的末尾，说明还没有结束

**3、原始字符串**  
\ 没有转义的意义，在引号前加`r`

```
r"D:\three\two\one\now"
```

**4、长字符串**

```
'''abc''' """abc"""
```

换行不需要`\n`  
也可以用（）括起来 不需要`\n`

**5、字符串的加法和乘法**

**字符串的加法**：拼接  
**字符串的乘法**：复制

**6、方法：**  
[https://fishc.com.cn/forum.php?mod=viewthread&tid=38992&extra=page%3D1%26filter%3Dtypeid%26typeid%3D403](https://fishc.com.cn/forum.php?mod=viewthread&tid=38992&extra=page=1&filter=typeid&typeid=403)

**7、格式化**

**（1）format 方法**

**①位置参数**  
eg：

```
>>> "{0} love {1}.{2}".format("I","FishC","com")
'I love FishC.com'
```

**②关键字参数**  
eg：

```
>>> "{a} love {b}.{c}".format(a="I",b="FishC",c="com")
'I love FishC.com'
```

**（2）% 方法**

eg:

```
print('十进制 -> 十六进制 : %d -> 0x%x' % (num, num))
print('十进制 -> 八进制 : %d -> 0o%o' % (num, num))
print('十进制 -> 二进制 : %d ->' % (num), bin(num))
```

**8、字符串格式化符号含义及转义字符含义：**  
[https://fishc.com.cn/forum.php?mod=viewthread&tid=92997&extra=page%3D1%26filter%3Dtypeid%26typeid%3D403](https://fishc.com.cn/forum.php?mod=viewthread&tid=92997&extra=page=1&filter=typeid&typeid=403)

**9、random**

```
import random					#导入
random.randint(a, b)			#随机a~b之间的整数
```

## 八、序列

**1、列表、元组和字符串的共同点：**

(1) 都可以通过索引得到每一个元素  
(2) 默认索引值总是从 0 开始  
(3) 可以通过切片的方式得到一个范围内的元素的集合  
(4) 有很多共同的操作符（重复操作符、拼接操作符、成员关系操作符）

**2、方法**

**`list(iterable, /)`** ：把一个可迭代对象转换为列表  
**`tuple(iterable, /)`** ：把一个可迭代对象转换为元组  
**`str(obj)`** ：把 obj 对象转换为字符串  
**`len(sub)`** ：返回 sub 对象的长度  
**`max()`** ：返回序列或者参数集合中的最大值  
**`min()`** ：返回序列或者参数集合中的最小值  
**`sum(iterable[, start=0])`** ： 返回序列 iterable 和可选参数 start 的总和  
**`sorted()`** ： 从小到大排序  
**`reversed()`** ： 翻转（返回迭代器对象）, 如果想要显示，可以用`list(reversed())`

eg：

```
>>> numbers = [32, 35, 76, 29, 4, 0, -56, 17]
>>> reversed(numbers)
<list_reverseiterator object at 0x026DAFD0>
>>> list(reversed(numbers))
[17, -56, 0, 4, 29, 76, 35, 32]
```

**`enumerate()`** ： 枚举（返回迭代器对象），生成由每个元素的 index 值和 item 值组成的一个元组，如果想要显示，可以用`list(enumerate())`

eg：

```
>>> enumerate(numbers)
<enumerate object at 0x0271BC28>
>>> list(enumerate(numbers))
[(0, 32), (1, 35), (2, 76), (3, 29), (4, 4), (5, 0), (6, -56), (7, 17)]
```

**`zip()`** ：返回由各个参数组成的元组（返回迭代器对象），如果想要显示，可以用`list(zip())`  
eg：

```
>>> a = [1, 2, 3, 4, 5, 6]
>>> b = [4, 5, 6, 7]
>>> zip(a, b)
<zip object at 0x0271BC28>
>>> list(zip(a, b))
[(1, 4), (2, 5), (3, 6), (4, 7)]
```