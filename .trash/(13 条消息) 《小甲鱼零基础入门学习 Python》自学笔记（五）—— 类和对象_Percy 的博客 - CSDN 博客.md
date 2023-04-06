# 类和对象

## 一、类和对象的简单介绍

**对象 = 属性 + 方法**

**定义一个类**：`class`关键字  
[Python](https://so.csdn.net/so/search?q=Python&spm=1001.2101.3001.7020) 中的类名约定以大写字母开头。

**面向对象的特征（Object Oriented）：**  
a. 封装：信息隐蔽技术  
b. 继承：子类自动共享父类之间数据和方法的机制  
c. 多态：不同对象对同一方法响应不同的行动

## 二、面向对象编程

**（1）`self`**：Python 的 self 相当于 C++ 的`this`指针

由同一个类可以生成无数个对象，这些对象长得都很相似，因为他们都是来源于同一个类的属性和方法，当一个对象的方法被调用的时候，对象会将自身作为第一个参数传给`self`参数，接收到`self`的时候，Python 就知道是哪一个对象在调用方法了。

eg：

```
>>> class Ball:
	def setName(self, name):
		self.name = name
	def talk(self):
		print('我是%s' % self.name)

>>> a = Ball()
>>> a.setName('足球')
>>> b = Ball()
>>> b.setName('篮球')
>>> a.talk()
我是足球
>>> b.talk()
我是篮球
```

**（2）Python 的魔法方法之一**

```
__init__(self)
__init__(self, param1, param2...)
```

在类实例化后，会自动调用`__init__(self)`方法。

eg：

```
>>> class Ball:
	def __init__(self, name):
		self.name = name
	def talk(self):
		print('我是%s' % self.name)

		
>>> a = Ball('羽毛球')
>>> a.talk()
我是羽毛球
```

**（3）公有和私有**

默认来说，对象的属性和方法都是公有的，可以通过 “.” 操作符进行访问。  
在 Python 中定义私有变量只需要在变量名或函数名前加上 “__” 两个下划线，那么这个函数或变量就会为私有的了。

## 三、继承

子类可以继承父类的属性和方法

**1、定义一个子类：**

```
class DerivedClassName(BaseClassName):
	…
```

**BaseClassName**：父类、基类或超类  
**DerivedClassName**：子类

eg：

```
import random as r

class Fish:
    def __int__(self):
        self.x = r.randint(0, 10)
        self.y = r.randint(0, 10)

    def move(self):
        self.x -= 1
        print('我的位置是：', self.x, self.y)

class Goldfish(Fish):
    pass

class Carp(Fish):
    pass

class Salmon(Fish):
pass
```

运行：

```
>>> fish = Goldfish()
>>> fish.__int__()
>>> fish.move()
我的位置是： 7 9
```

注意：  
如果子类中定义与父类同名的方法或者属性，则会自动覆盖父类对应的方法或属性。

**2、当子类中与父类同名的方法或属性覆盖掉父类中的方法或属性，但又想引用父类中的方法或属性时，我们可以采用以下两种方法：**

**a. 调用未绑定的父类方法  
b. 使用 super 函数**

eg：调用未绑定的父类方法

```
import random as r

class Fish:
    def __init__(self):
        self.x = r.randint(0, 10)
        self.y = r.randint(0, 10)

    def move(self):
        self.x -= 1
        print('我的位置是：', self.x, self.y)

class Goldfish(Fish):
    pass

class Carp(Fish):
    pass

class Salmon(Fish):
    pass

class Shark(Fish):
    def __init__(self):
        Fish.__init__(self)
        self.hungry = True

    def eat(self):
        if self.hungry == True:
            print('我饿了！我要吃饭！')
            self.hungry = False
        else:
            print('我好撑！吃不下了！')
```

eg：使用 super 函数

```
import random as r

class Fish:
    def __init__(self):
        self.x = r.randint(0, 10)
        self.y = r.randint(0, 10)

    def move(self):
        self.x -= 1
        print('我的位置是：', self.x, self.y)

class Goldfish(Fish):
    pass

class Carp(Fish):
    pass

class Salmon(Fish):
    pass

class Shark(Fish):
    def __init__(self):
        super().__init__()
        self.hungry = True

    def eat(self):
        if self.hungry == True:
            print('我饿了！我要吃饭！')
            self.hungry = False
        else:
            print('我好撑！吃不下了！')
```

## 四、多重继承

定义一个子类：

```
class DerivedClassName(Base1, Base2, Base3):
	…
```

注意：尽量避免使用多重继承！

## 五、组合

把类的实例化放在新的类里面，就把旧的类组合进去了。

eg：

```
class Turtle:
    def __init__(self, x):
        self.num = x

class Fish:
    def __init__(self, y):
        self.num = y

class Pool:
    def __init__(self, x, y):
        self.turtle = Turtle(x)
        self.fish = Fish(y)

    def print_num(self):
        print('水池中有%d只乌龟，%d条鱼。' % (self.turtle.num, self.fish.num))
```

运行：

```
>>> p = Pool(1, 5)
>>> p.print_num()
水池中有1只乌龟，5条鱼。
```

## 六、类、类对象和实例对象

```
>>> class C:
	count = 0

	
>>> a = C()
>>> b = C()
>>> c = C()
>>> a.count
>>> b.count
>>> c.count
>>> c.count += 10
>>> c.count
>>> a.count
>>> b.count
>>> C.count += 100
>>> a.count
>>> b.count
>>> c.count
```

![](<assets/1680774432595.png>)

如果实例化对象的属性和类的方法名相同，属性会覆盖掉方法。

注意：  
1. 不要试图在一个类里边定义出所有能想到的特性和方法，应该利用继承和组合机制来进行扩展。  
2. 用不同的词性命名，如属性名用名词，方法名用动词。

## 七、绑定

Python 严格要求方法需要有实例才能被调用，这种限制就是 Python 所谓的绑定概念。

```
>>> class CC:
	def setXY(self, x, y):
		self.x = x
		self.y = y
	def printXY(self):
		print(self.x, self.y)

		
>>> dd = CC()
>>> dd.__dict__
{}
>>> CC.__dict__
mappingproxy({'__module__': '__main__', 'setXY': <function CC.setXY at 0x041B0FA0>, 'printXY': <function CC.printXY at 0x041B8028>, '__dict__': <attribute '__dict__' of 'CC' objects>, '__weakref__': <attribute '__weakref__' of 'CC' objects>, '__doc__': None})
```

未给实例对象`dd`赋值时，`dd`的属性是一个空字典。类对象`CC`的属性中包含实例化对象的属性，不显示类属性和特殊属性。

```
>>> dd.setXY(4, 5)
>>> dd.__dict__
{'x': 4, 'y': 5}
>>> CC.__dict__
mappingproxy({'__module__': '__main__', 'setXY': <function CC.setXY at 0x041B0FA0>, 'printXY': <function CC.printXY at 0x041B8028>, '__dict__': <attribute '__dict__' of 'CC' objects>, '__weakref__': <attribute '__weakref__' of 'CC' objects>, '__doc__': None})
```

给实例对象`dd`赋值后，dd 的属性仅属于`dd`对象，类对象`CC`中不存在`x`和`y`。

```
>>> del CC
>>> ee = CC()
Traceback (most recent call last):
  File "<pyshell#15>", line 1, in <module>
    ee = CC()
NameError: name 'CC' is not defined
>>> dd.printXY()
4 5
```

删掉类实例`CC`，不能再实例化类`CC`，但实例对象`dd`依旧能调用类`CC`中的方法。因为类中定义的属性和方法都是静态变量，就算类对象被删除了，它们依然是存放在内存中的。只有当程序被退出的时候，这个变量才会被释放。

## 八、一些相关的 BIF

**1、`issubclass(class, classinfo)`**

检查`class`是否是`classinfo`的一个子类，是，返回`True`，否，返回`False`。  
注意：  
1. 这种检查是非严格性的检查，一个类被认为是其自身的子类。  
2.`classinfo`可以是类对象组成的元组，只要`class`是其中任何一个候选类的子类，则返回`True`。如果第二个参数不是类或者由类对象组成的元组，会抛出一个`TypeError`异常。

**2、`isinstance(object, classinfo)`**

检查实例对象`object`是否属于类`classinfo`，是，返回`True`，否，返回`False`。  
注意：  
1.`classinfo`可以是类对象组成的元组，只要`object`是其中任何一个候选类的实例对象，则返回`True`。  
2. 如果第一个参数不是对象，则永远返回`False`。  
3. 如果第二个参数不是类或者由类对象组成的元组，会抛出一个`TypeError`异常。

**3、`hasattr(object, name)`**

测试对象`objec`t 是否有指定的属性`name`，`name`为字符串格式。

**4、`getattr(object, name[, default])`**

返回对象`object`中指定的属性`name`的值，`default`是当属性`name`不存在时返回的参数。

**5、`setattr(object, name, value)`**

设置对象`object`中指定的属性`name`的值为`value`，如果属性`name`不存在，会新建一个属性并给它赋值。

**6、`delattr(object, name)`**

删除对象`object`中指定的属性`name`，如果属性`name`不存在，则抛出`AttributeError`异常。

**7、`property(fget=None, fset=None, fdel=None, doc=None)`**

通过属性设置属性。

eg：

```
>>> class C:
	def __init__(self, size=10):
		self.size = size
	def getSize(self):
		return self.size
	def setSize(self, value):
		self.size = value
	def delSize(self):
		del self.size
	x = property(getSize, setSize, delSize)

	
>>> c = C()
>>> c.x				# 调用 property第一个参数getSize方法
>>> c.x = 20			# 调用 property第二个参数setSize方法
>>> c.x
>>> del c.x			# 调用 property第三个参数delSize方法
>>> c.x
Traceback (most recent call last):
  File "<pyshell#18>", line 1, in <module>
    c.x
  File "<pyshell#11>", line 5, in getSize
    return self.size
AttributeError: 'C' object has no attribute 'size'
```