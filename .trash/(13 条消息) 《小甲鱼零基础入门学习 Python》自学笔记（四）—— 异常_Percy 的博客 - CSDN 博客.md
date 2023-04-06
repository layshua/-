# 异常

## 一、Python 标准异常总结

[https://fishc.com.cn/forum.php?mod=viewthread&tid=45814&extra=page%3D1%26filter%3Dtypeid%26typeid%3D403](https://fishc.com.cn/forum.php?mod=viewthread&tid=45814&extra=page=1&filter=typeid&typeid=403)

## 二、[异常处理](https://so.csdn.net/so/search?q=%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86&spm=1001.2101.3001.7020)

**（1）`try-except`语句**

```
try:
	检测范围
except Exception[as reason]:
	出现异常（Exception）后处理代码
```

eg：

```
try:
    sum = 1 + '1'
    f = open('文件1.txt')
    print(f.read())
    f.close()
except OSError as reason:
    print('文件出错啦！！\n错误的原因是：', reason)
except TypeError as reason:
	print('类型出错啦！！\n错误的原因是：', reason)
```

注意：`try`语句检测范围一旦出现异常，剩下的语句将不会被执行！

同时对多个异常进行统一的处理：  
eg：

```
try:
    sum = 1 + '1'
    f = open('文件1.txt')
    print(f.read())
    f.close()
except (OSError, TypeError):
	print('出错啦！！')
```

**（2）`try-finally`语句**

```
try:
	检测范围
except Exception[as reason]:
	出现异常（Exception）后处理代码
finally:
	无论如何都会被执行的代码
```

eg:

```
try:
    f = open('文件1.txt', 'w')
    print(f.write('我存在了！'))
    sum = 1 + '1'
except (OSError, TypeError):
    print('出错啦！！')
finally:
	f.close()
```

## 三、引发异常

**`raise`语句**

eg：

```
>>> raise ZeroDivisionError('除数为零的异常')
Traceback (most recent call last):
  File "<pyshell#2>", line 1, in <module>
    raise ZeroDivisionError('除数为零的异常')
ZeroDivisionError: 除数为零的异常
```

## 四、丰富的 else 语句及简洁的 with 语句

**（1）丰富的`else`语句:**

当没有异常时，执行`else`语句

eg:

```
try:
    print(int('123'))
except ValueError as reason:
    print('出错啦！')
else:
    print('没有任何异常')
```

**（2）简洁的`with`语句：**

**考虑关闭文件的问题**：当文件不再需要用到的时候，with 语句会自动关闭文件。

eg：

```
try:
    with open('文件1.txt', 'w') as f:
        for each_line in f:
            print(each_line)
except OSError as reason:
    print('出错啦！！', reason)
```