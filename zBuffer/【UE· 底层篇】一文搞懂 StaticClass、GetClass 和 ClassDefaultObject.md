学 UE 也有一段时间了，关于 **StaticClass** 和 **GetClass** 一直搞得不太懂。看别人代码时也莫名其妙，为什么这里用 StaticClass？为什么那里用 GetClass？ 这都是啥玩意？

![[ab38b10d996664857570dddcf251df94_MD5.jpg]]

正好最近看 **《大象无形 虚幻引擎程序设计浅析》** 还看到了 **ClassDefaultObject**。正好把这三个东西彻底研究了一番。这篇文章我会尽力讲的连新手也能看懂，中间会少量掺杂 ue4 底层的一些源码。（顺带写了一些测试代码，[github 工程。](https://github.com/756915370/UE4_LearnUClass)）

### UClass 和反射

什么是反射？不是光学那个反射，是编程语言的那种反射。有 c# 或者 java 使用经验的肯定知道**反射**这个东西。简单来说，**反射的作用就是在不知道这个类是什么类的情况下获取到它的一些信息。** C++ 是没有反射机制的所以 UE 底层实现了一套反射机制。C# 怎么用反射：

```
string s = "鸡桑大帅逼”；
Type t = s.GetType()；
```

Type 里面就包含了 string 这个类的各种信息。接下来请我和大声念三遍：

*   **UClass 就是 C# 里的 Type!**
*   **UClass 就是 C# 里的 Type!**
*   **UClass 就是 C# 里的 Type!**

虽然 UClass 和 C# 里的 Type 的作用并不完全相同，但是这么理解会帮我们弄明白 UE 底层的很多事情。

### GetClass()

现在我们对 UClass 是干嘛用的已经有了一个初步的了解，来看 GetClass。**GetClass()** 的作用就是我们生成一个 UObject 实例后，去拿这个实例的 UClass，类似 C# 的 **GetType**。接下来我们做个小实验。新建一个自己的 Actor 就叫 MyActor 好了，写一个方法并且让它蓝图可调用：

![[d5d6733a8dda58a5199ef0087152203c_MD5.jpg]]

建两个蓝图，并拖到场景中，在关卡蓝图里调用 **IsSameUClass**：

![[8843ff51e75527c39334e8d163224d9e_MD5.jpg]]

![[3aec8a80e5a071ec461cf21d1cff9c47_MD5.jpg]]

显示的结果是 0，也就是两个 **MyActor 的 GetClass() 结果不一样**。但是如果场景上的两个 Actor **都是 BP_MyActor 同一个蓝图，它们的 GetClass() 结果是一样的。**

先解释第一个结果：**为什么同样 C++ 类型不同蓝图返回的 GetClass() 结果不一样？** 那是因为 UClass 不仅需要记录类型信息，还需要承担**序列化**的工作。同样 C++ 类型，但是蓝图不一样，需要序列化数据是不一样的，所以 UClass 不一样。同样类型，同样蓝图，但是场景中实例不一样，但是这时需要序列化的蓝图数据是一样的（像场景中的位置信息不是记录在蓝图里的），所以它们的 UClass 是一样的，这是第二个结果的原因。

### GetStaticClass()

GetClass 很好很强大。但是有个问题，**它是 UObject 的成员函数**。我现在没有 UObject 实例，但是我想拿到某个类的 UClass 怎么办？锵锵，**StaticClass** 来了。我可以这么调用：

```
UClass* myUClass = AMyActor::StaticClass();
```

这样子我就**不需要有 UObject 实例也能拿到某个类的 UClass 了**。同时因为它是 Static 的所以每次调用 T::StaticClass 返回的都是同一个结果。

```
auto static1Class = AMyActor::StaticClass();
auto static2Class = AMyActor::StaticClass();
auto result = static1Class == static2Class;
//结果是1
UE_LOG(LogTemp, Warning, TEXT("Is StaticClass Same: %d"), result);
```

再讲 StaticClass 的另一个功能，帮助大家更加理解 StaticClass。先来看 UObject、UClass、Actor 类图（虽然中间有一些类名大家可能没看过但是没有关西）：

![[6f4a4c16ca3964e8d81a7f7984fba90b_MD5.jpg]]

现在又有个问题来了，我现在有一个 **UClass，如何知道这个 UClass 和另一个 UClass 是不是继承关系？** 直接转类型判断吗？不对，看上面那张图，UClass 是没有子类的。那怎么办？答案是 **UClass 存储了它要描述的类的父类的 StaticClass。** 有点绕，我举例说明一下。我现在有个 MyActor 类的 UClass，想知道 MyActor 是不是 UObject 的子类，我就这么做：

*   调用 UObject::StaticClass()
*   拿到 MyActor 的 UClass 存储的 MyActor 的父类的 StaticClass 也就是 Actor 的 StaticClass
*   比较 Actor 的 StaticClass 和 UObject 的 StaticClass 是不是相等
*   发现不相等，这次我拿 Actor 的 StaticClass（因为它也是一个 UClass）里存的 Actor 父类的 StaticClass 也就是 UObject 的 StaticClass，然后比较，发现相等，返回 true
*   如果我发现不相等，我就一直比，比到最后没有父类，返回 false

上面这段逻辑就是 **UClass 的 IsChildOf 的实现原理**。调用 IsChildOf 的时候传入泛型：

![[5cde1608b92f443c7ba603fef61f58c6_MD5.png]]

这是具体实现。**UStruct** 是 UClass 父类。

![[dfbfbc18d7faa479a8bfe0218ef448cf_MD5.jpg]]

GetSuperStruct() 就是返回它存储的父类的 StaticClass。

![[b22a0653c9c4c750ec4aa20b0aeb8e09_MD5.png]]

顺带再讲一下 **IsA** 这个函数，功能和 IsChildOf 一样，不同的是 IsChildOf 给 UClass 用的，IsA 是给 UObject 对象实例用的，比如想知道一个 MyActor 实例是不是 UObject 的子类：

```
//myActor是MyAcor*类型
bool result = myActor->IsA<UObject>();
```

### ClassDefaultObject

**GetClass** 和 **GetStaticClass** 都明白了之后，理解 **ClassDefaultObject** 就简单多了。ClassDefaultObject（简称 CDO），类默认对象。通过 CDO 我可以拿到**一个 UObject 初始化时的值**。虽然 CDO 有个 Default 默认，但是用默认去描述它的功能不是很准确。回到我们创建的类 MyActor，给它添加一个外部可以修改变量 **testIntValue**，默认值 1:

![[a7bc95bbdf6ebca930c9a2914cd1cd7a_MD5.jpg]]

对于两个蓝图分别修改，一个 20，另一个 10：

![[57874a26c52ee95fecabf36963ab8645_MD5.jpg]]

![[d18d08201c2014c335b39b11499ef9f0_MD5.jpg]]

然后在 BeginPlay 写下测试代码：

```
//先修改值再获取ClassDefaultObject
testIntValue = -1;
auto defaultInt = GetClass()->GetDefaultObject<AMyActor>()->testIntValue;
UE_LOG(LogTemp, Warning, TEXT("DefaultObject int: %d"), defaultInt);
```

最后打印的结果是一个 20，一个 10。  
**如果想拿到代码里给它设的默认值 1 要用 StaticClass**：

```
auto staticDefaultInt = AMyActor::StaticClass()->GetDefaultObject<AMyActor>()->testIntValue;
//打印1
UE_LOG(LogTemp, Warning, TEXT("StaticDefaultObject int: %d"), staticDefaultInt);
```

### GetClass()->GetClass()->GetClass()

最后来点丧心病狂的东西。上面我画了一张类图，UClass 也是 UObject 的子类。

![[6f4a4c16ca3964e8d81a7f7984fba90b_MD5.jpg]]

一个 UObject 实例可以调用 GetClass() 来获得它的 UClass，那么对 UClass 调用 GetClass() 会出现什么结果呢？在 MyActor 里我这么写：

![[48e4b8c5b43fb016de8c944ed1728c29_MD5.jpg]]

结果如下：

![[561e435fb6760478155967e25821838b_MD5.jpg]]

**调用第一次 GetClass 和第二次产生的结果不一样，第三次以后结果是一样的了**。至于为什么会这样我不知道，网上也没查出个所以然来，如果有谁弄懂请告诉我，我这里就先记录下来。

### 总结

*   **UClass。存储类信息，用于反射。把它当成 C# 的 Type 来理解。**
*   **GetClass()。获得一个 UObject 实例的 UClass，是 UObject 成员函数。**
*   **GetStaticClass()。不需要有实例就能获得 UClass。是静态的，每次调用返回相同结果。**
*   **ClassDefaultObject。类默认对象，可以获得 UObject 初始化时的值。注意 GetClass()->GetDefaultObject() 和 T::StaticClass()->GetDefaultObject() 不一样。**

关于作者

*   **水曜日鸡**，喜欢 ACG 的游戏程序员。曾参与索尼中国之星项目《硬核机甲》的开发。 目前在某大厂做 UE4 项目。

CSDN 博客：[https://blog.csdn.net/j756915370](https://blog.csdn.net/j756915370)  
知乎专栏：[https://zhuanlan.zhihu.com/c_1241442143220363264](https://zhuanlan.zhihu.com/c_1241442143220363264)  
游戏同行聊天群：891809847