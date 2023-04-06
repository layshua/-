# 字典、[集合](https://so.csdn.net/so/search?q=%E9%9B%86%E5%90%88&spm=1001.2101.3001.7020)、文件部分

## 一、字典（dict）

**字典是映射类型**

**（1）创造和访问字典**：

**关键符号：**{}  
“`:`” 前：键（key）  
“`:`” 后：值（value）  
eg：

```
>>> dict1 = {'李宁':'一切皆有可能', '耐克':'Just do it', '阿迪达斯':'Impossible is noting', '鱼C工作室':'让编程改变世界'}
>>> print('鱼C工作室的口号是：', dict1['鱼C工作室'])
鱼C工作室的口号是： 让编程改变世界
```

eg：

```
>>> dict2 = {1:'one', 2:'two', 3:'three'}
>>> dict2[2]
'two'
```

**（2）字典的方法：**

**`fromkeys(s[,v])`**：创建并返回一个新的字典  
eg：

```
>>> dict1 = {}
>>> dict1.fromkeys((1, 2, 3))
{1: None, 2: None, 3: None}
>>> dict1.fromkeys((1, 2, 3), 'Number')
{1: 'Number', 2: 'Number', 3: 'Number'}
```

**`keys()`**：访问字典的键  
eg：

```
>>> dict1 = {1: 'Number', 2: 'Number', 3: 'Number'}
>>> for eachKey in dict1.keys():
	print(eachKey)
```

**`values()`**：访问字典的值  
**`items()`**：访问字典的项（键和值）  
**`get()`**：访问字典的值（更灵活），键可以不在字典中  
eg：

```
>>> dict1 = {1: 'Number', 2: 'Number', 3: 'Number'}
>>> dict1[4]				# 会报错
Traceback (most recent call last):
  File "<pyshell#35>", line 1, in <module>
    dict1[4]
KeyError: 4
>>> dict1.get(4)			# 不会报错
>>> print(dict1.get(4))
None
>>> dict1.get(4,'无')		# 可以设置键不在字典中时的输出
'无'
>>> dict1.get(3,'无')		# 当键在字典中，正常输出该键对应的值
'Number'
```

**如果不知道一个键是否在字典中，可以用成员资格操作符来判断。**（`in` 和 `not in`）

```
>>> 3 in dict1
True
>>> 4 in dict2
False
```

注意：成员资格操作符在字典中查找的是键，不是值；而在序列中查找的是元素的值，不是索引号。

**`clear()`**：清空字典  
**`copy()`**：浅拷贝  
**`pop()`**：弹出值，参数：键  
**`popitem()`**：弹出一项（键和值） 字典没有一定的顺序，可以理解为随机弹出  
**`setdefault()`**：与 get() 类似，但当键不在字典中时，会自动添加进去

```
>>> dict1 = {1: 'Number', 2: 'Number', 3: 'Number'}
>>> dict1.setdefault(4)
>>> dict1
{1: 'Number', 2: 'Number', 3: 'Number', 4: None}
>>> dict1.setdefault(5, 'Five')
'Five'
>>> dict1
{1: 'Number', 2: 'Number', 3: 'Number', 4: None, 5: 'Five'}
```

**`update()`**：更新

```
>>> dict1
{1: 'Number', 2: 'Number', 3: 'Number', 4: None, 5: 'Five'}
>>> dict2 = {2:'Two'}
>>> dict1.update(dict2)
>>> dict1
{1: 'Number', 2: 'Two', 3: 'Number', 4: None, 5: 'Five'}
```

## 二、集合（set）

**集合里的元素具有唯一性，自动清除重复的数据  
集合是无序的**

**（1）创建：**  
直接把一堆元素用 “`{}`” 括起来

```
>>> num = {1, 2, 3, 4, 4, 5, 5}
>>> num
{1, 2, 3, 4, 5}
```

使用`set()`工厂函数

```
>>> set1 = set([1, 3, 5, 7, 9])
>>> set1
{1, 3, 5, 7, 9}
```

**（2）访问集合中的值：**  
可以使用`for`把集合中的数据一个个读取出来  
可以通过`in`和`not in`判断一个元素是否在集合中已经存在

**（3）方法：**

```
>>> num
{1, 2, 3, 4, 5}
>>> num.add(6)
>>> num
{1, 2, 3, 4, 5, 6}
>>> num.remove(5)
>>> num
{1, 2, 3, 4, 6}
```

**集合类型内建方法总结：**  
[https://fishc.com.cn/forum.php?mod=viewthread&tid=45276&extra=page%3D1%26filter%3Dtypeid%26typeid%3D403](https://fishc.com.cn/forum.php?mod=viewthread&tid=45276&extra=page=1&filter=typeid&typeid=403)

**（4）不可变集合：**  
**`frozenset()`**：创建一个不可变集合

```
>>> num2 = frozenset([1, 2, 3, 4, 5])
>>> num2.add(0)
Traceback (most recent call last):
  File "<pyshell#12>", line 1, in <module>
    num2.add(0)
AttributeError: 'frozenset' object has no attribute 'add'
```

## 三、文件（file）

扩展名：.exe .txt .ppt .jpg .mp4 .avi 等

**（1）文件打开模式：**

![](1680774411405.png)

‘x’ 和 ‘w’ 均是以 “可写入” 的模式打开文件

**（2）文件对象方法：**

![](1680774411489.png)

**注意：字节和字符是不同的！**

打印文件的每一行：

```
for each_line in f:
	print(each_line)
```

文件的写入：

```
>>> f = open('D:\\test.txt', 'w')
>>> f.write('Hello,World!')
>>> f.close()
```

## 四、文件系统

模块是一个包含所有你定义的函数和变量的文件，其后缀名是. py。  
模块可以被别的程序引入，以使用该模块中的函数等功能。

**OS：Operating System 操作系统**  
有了 OS 模块，我们不需要关心什么操作系统下使用什么模块，OS 模块会帮你选择正确的模块并调用。

**`os`、`os.path` 模块中关于文件、目录常用的函数使用方法**  
[https://fishc.com.cn/forum.php?mod=viewthread&tid=45512&extra=page%3D1%26filter%3Dtypeid%26typeid%3D403](https://fishc.com.cn/forum.php?mod=viewthread&tid=45512&extra=page=1&filter=typeid&typeid=403)

## 五、永久存储

**`pickle`模块**

**存放**：pickling  
**读取**：unpickling

eg：  
存放：

```
>>> import pickle								# 导入
>>> my_list = [123, 3.14, 'Percy', ['another list']]
>>> pickle_file = open('my_list.pkl', 'wb')			# 创建并打开文件
>>> pickle.dump(my_list, pickle_file)				# 把数据倒入文件
>>> pickle_file.close()							# 关闭文件
```

读取：

```
>>> pickle_file = open('my_list.pkl', 'rb')			# 打开文件
>>> my_list2 = pickle.load(pickle_file)				# 加载文件中的数据
>>> my_list2
[123, 3.14, 'Percy', ['another list']]
```