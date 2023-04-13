![](<assets/1681390414159.png>)

小甲鱼 据说，Python 的对象天生拥有一些神奇的方法，它们总被双下划线所包围，他们是面向对象的 Python 的一切。  
他们是可以给你的类增加魔力的特殊方法，如果你的对象实现（重载）了这些方法中的某一个，那么这个方法就会在特殊的情况下被 Python 所调用，你可以定义自己想要的行为，而这一切都是自动发生的。  
  
Python 的魔术方法非常强大，然而随之而来的则是责任。了解正确的方法去使用非常重要！  
  
  
<table cellspacing="0"><tbody><tr><td width="30%"><div align="center"><strong>魔法方法</strong></div></td><td><div align="center"><strong>含义</strong></div></td></tr><tr><td><br></td><td><div align="left"><div align="center"><font color="#ff0000"><strong>基本的魔法方法</strong></font></div></div></td></tr><tr><td>__new__(cls[, ...])</td><td>1. __new__ 是在一个对象实例化的时候所调用的第一个方法<br>2. 它的第一个参数是这个类，其他的参数是用来直接传递给 __init__ 方法<br>3. __new__ 决定是否要使用该 __init__ 方法，因为 __new__ 可以调用其他类的构造方法或者直接返回别的实例对象来作为本类的实例，如果 __new__ 没有返回实例对象，则 __init__ 不会被调用<br>4. __new__ 主要是用于继承一个不可变的类型比如一个 tuple 或者 string</td></tr><tr><td>__init__(self[, ...])</td><td>构造器，当一个实例被创建的时候调用的初始化方法</td></tr><tr><td>__del__(self)</td><td>析构器，当一个实例被销毁的时候调用的方法</td></tr><tr><td>__call__(self[, args...])</td><td>允许一个类的实例像函数一样被调用：x(a, b) 调用 x.__call__(a, b)</td></tr><tr><td>__len__(self)</td><td>定义当被 len() 调用时的行为</td></tr><tr><td>__repr__(self)</td><td>定义当被 repr() 调用时的行为</td></tr><tr><td>__str__(self)</td><td>定义当被 str() 调用时的行为</td></tr><tr><td>__bytes__(self)</td><td>定义当被 bytes() 调用时的行为</td></tr><tr><td>__hash__(self)</td><td>定义当被 hash() 调用时的行为</td></tr><tr><td>__bool__(self)</td><td>定义当被 bool() 调用时的行为，应该返回 True 或 False</td></tr><tr><td>__format__(self, format_spec)</td><td>定义当被 format() 调用时的行为</td></tr><tr><td></td><td><div align="center"><font color="#ff0000"><strong>有关属性</strong></font></div></td></tr><tr><td>__getattr__(self, name)</td><td>定义当用户试图获取一个不存在的属性时的行为</td></tr><tr><td>__getattribute__(self, name)</td><td>定义当该类的属性被访问时的行为</td></tr><tr><td>__setattr__(self, name, value)</td><td>定义当一个属性被设置时的行为</td></tr><tr><td>__delattr__(self, name)</td><td>定义当一个属性被删除时的行为</td></tr><tr><td>__dir__(self)</td><td>定义当 dir() 被调用时的行为</td></tr><tr><td>__get__(self, instance, owner)</td><td>定义当描述符的值被取得时的行为</td></tr><tr><td>__set__(self, instance, value)</td><td>定义当描述符的值被改变时的行为</td></tr><tr><td>__delete__(self, instance)</td><td>定义当描述符的值被删除时的行为</td></tr><tr><td></td><td><div align="center"><font color="#ff0000"><strong>比较操作符</strong></font></div></td></tr><tr><td>__lt__(self, other)</td><td>定义小于号的行为：x &lt;y 调用 x.__lt__(y)</td></tr><tr><td>__le__(self, other)</td><td>定义小于等于号的行为：x &lt;= y 调用 x.__le__(y)</td></tr><tr><td>__eq__(self, other)</td><td>定义等于号的行为：x == y 调用 x.__eq__(y)</td></tr><tr><td>__ne__(self, other)</td><td>定义不等号的行为：x != y 调用 x.__ne__(y)</td></tr><tr><td>__gt__(self, other)</td><td>定义大于号的行为：x &gt; y 调用 x.__gt__(y)</td></tr><tr><td>__ge__(self, other)</td><td>定义大于等于号的行为：x &gt;= y 调用 x.__ge__(y)</td></tr><tr><td></td><td><div align="center"><font color="#ff0000"><strong>算数运算符</strong></font></div></td></tr><tr><td>__add__(self, other)</td><td>定义加法的行为：+</td></tr><tr><td>__sub__(self, other)</td><td>定义减法的行为：-</td></tr><tr><td>__mul__(self, other)</td><td>定义乘法的行为：*</td></tr><tr><td>__truediv__(self, other)</td><td>定义真除法的行为：/</td></tr><tr><td>__floordiv__(self, other)</td><td>定义整数除法的行为：//</td></tr><tr><td>__mod__(self, other)</td><td>定义取模算法的行为：%</td></tr><tr><td>__divmod__(self, other)</td><td>定义当被 divmod() 调用时的行为</td></tr><tr><td>__pow__(self, other[, modulo])</td><td>定义当被 power() 调用或 ** 运算时的行为</td></tr><tr><td>__lshift__(self, other)</td><td>定义按位左移位的行为：&lt;&lt;</td></tr><tr><td>__rshift__(self, other)</td><td>定义按位右移位的行为：&gt;&gt;</td></tr><tr><td>__and__(self, other)</td><td>定义按位与操作的行为：&amp;</td></tr><tr><td>__xor__(self, other)</td><td>定义按位异或操作的行为：^</td></tr><tr><td>__or__(self, other)</td><td>定义按位或操作的行为：|</td></tr><tr><td></td><td><div align="center"><font color="#ff0000"><strong>反运算</strong></font></div></td></tr><tr><td>__radd__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__rsub__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__rmul__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__rtruediv__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__rfloordiv__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__rmod__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__rdivmod__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__rpow__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__rlshift__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__rrshift__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__rand__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__rxor__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td>__ror__(self, other)</td><td>（与上方相同，当左操作数不支持相应的操作时被调用）</td></tr><tr><td></td><td><div align="center"><font color="#ff0000"><strong>增量赋值运算</strong></font></div></td></tr><tr><td>__iadd__(self, other)</td><td>定义赋值加法的行为：+=</td></tr><tr><td>__isub__(self, other)</td><td>定义赋值减法的行为：-=</td></tr><tr><td>__imul__(self, other)</td><td>定义赋值乘法的行为：*=</td></tr><tr><td>__itruediv__(self, other)</td><td>定义赋值真除法的行为：/=</td></tr><tr><td>__ifloordiv__(self, other)</td><td>定义赋值整数除法的行为：//=</td></tr><tr><td>__imod__(self, other)</td><td>定义赋值取模算法的行为：%=</td></tr><tr><td>__ipow__(self, other[, modulo])</td><td>定义赋值幂运算的行为：**=</td></tr><tr><td>__ilshift__(self, other)</td><td>定义赋值按位左移位的行为：&lt;&lt;=</td></tr><tr><td>__irshift__(self, other)</td><td>定义赋值按位右移位的行为：&gt;&gt;=</td></tr><tr><td>__iand__(self, other)</td><td>定义赋值按位与操作的行为：&amp;=</td></tr><tr><td>__ixor__(self, other)</td><td>定义赋值按位异或操作的行为：^=</td></tr><tr><td>__ior__(self, other)</td><td>定义赋值按位或操作的行为：|=</td></tr><tr><td></td><td><div align="center"><font color="#ff0000"><strong>一元操作符</strong></font></div></td></tr><tr><td>__pos__(self)</td><td>定义正号的行为：+x</td></tr><tr><td>__neg__(self)</td><td>定义负号的行为：-x</td></tr><tr><td>__abs__(self)</td><td>定义当被 abs() 调用时的行为</td></tr><tr><td>__invert__(self)</td><td>定义按位求反的行为：~x</td></tr><tr><td></td><td><div align="center"><font color="#ff0000"><strong>类型转换</strong></font></div></td></tr><tr><td>__complex__(self)</td><td>定义当被 complex() 调用时的行为（需要返回恰当的值）</td></tr><tr><td>__int__(self)</td><td>定义当被 int() 调用时的行为（需要返回恰当的值）</td></tr><tr><td>__float__(self)</td><td>定义当被 float() 调用时的行为（需要返回恰当的值）</td></tr><tr><td>__round__(self[, n])</td><td>定义当被 round() 调用时的行为（需要返回恰当的值）</td></tr><tr><td>__index__(self)</td><td>1. 当对象是被应用在切片表达式中时，实现整形强制转换<br>2. 如果你定义了一个可能在切片时用到的定制的数值型, 你应该定义 __index__<br>3. 如果 __index__ 被定义，则 __int__ 也需要被定义，且返回相同的值</td></tr><tr><td></td><td><div align="center"><font color="#ff0000"><strong>上下文管理（with 语句）</strong></font></div></td></tr><tr><td>__enter__(self)</td><td>1. 定义当使用 with 语句时的初始化行为<br>2. __enter__ 的返回值被 with 语句的目标或者 as 后的名字绑定</td></tr><tr><td>__exit__(self, exc_type, exc_value, traceback)</td><td>1. 定义当一个代码块被执行或者终止后上下文管理器应该做什么<br>2. 一般被用来处理异常，清除工作或者做一些代码块执行完毕之后的日常工作</td></tr><tr><td></td><td><div align="center"><strong><font color="#ff0000">容器类型</font></strong></div></td></tr><tr><td>__len__(self)</td><td>定义当被 len() 调用时的行为（返回容器中元素的个数）</td></tr><tr><td>__getitem__(self, key)</td><td>定义获取容器中指定元素的行为，相当于 self[key]</td></tr><tr><td>__setitem__(self, key, value)</td><td>定义设置容器中指定元素的行为，相当于 self[key] = value</td></tr><tr><td>__delitem__(self, key)</td><td>定义删除容器中指定元素的行为，相当于 del self[key]</td></tr><tr><td>__iter__(self)</td><td>定义当迭代容器中的元素的行为</td></tr><tr><td>__reversed__(self)</td><td>定义当被 reversed() 调用时的行为</td></tr><tr><td>__contains__(self, item)</td><td>定义当使用成员测试运算符（in 或 not in）时的行为</td></tr></tbody></table>  
![](<assets/1681390414437.png>)

黄种人 看来面向对象的语言好好相像呢: big![](<assets/1681390414664.png>)

noroot 过来学习下![](<assets/1681390414921.png>)

wei_Y 这个好长啊。记在小本本上慢慢看。![](<assets/1681390415178.png>)

starhqy2 一下子记不住，要用到才知道，，，![](<assets/1681390415402.png>)

xiaom 虽然看了视频但完全不懂，怎么办？

![](<assets/1681390415616.png>)

![](<assets/1681390415727.png>)

小甲鱼

[xiaom 发表于 2014-9-26 16:42](https://fishc.com.cn/forum.php?mod=redirect&goto=findpost&pid=2091089&ptid=48793)  
虽然看了视频但完全不懂，怎么办？

木有独立完成作业~![](<assets/1681390416017.png>)

whl510 过来学习下![](<assets/1681390416271.png>)

xxywyc 感觉 py 太强大了![](<assets/1681390416478.png>)

zsdzlsz 好多。。![](<assets/1681390416716.png>)

langzijinge 面向对象对我来所还是太难了![](<assets/1681390417025.png>)

zhangyuge14 先收藏，在学习![](<assets/1681390417221.png>)

mengtao1010 小甲鱼老师好厉害！![](<assets/1681390417450.png>)

fanyangok 呵呵，好多呀！好强大！！！![](<assets/1681390417692.png>)

小强。。。 学习学习![](<assets/1681390417922.png>)

dothink 小甲鱼，这里的容器类型的魔法方法是不是写少了个 next 方法呢？![](<assets/1681390418137.png>)

富友郑鹏展 的风格化风格化臭豆腐更好发挥![](<assets/1681390418382.png>)

gongfucha888 感谢小甲鱼~![](<assets/1681390418621.png>)

zdhmeng _本帖最后由 MichaelYan 于 2015-8-6 18:09 编辑_  
楼主把'__neg__(self)'和‘__pos__(self)’的作用给写反了  
neg:negtive  
pos:positive