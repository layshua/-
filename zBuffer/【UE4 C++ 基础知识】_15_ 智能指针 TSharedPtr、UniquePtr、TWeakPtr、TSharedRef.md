# 基本概念

*   UE4 对 UObject 对象提供垃圾回收
*   UE4 对原生对象不提供垃圾回收，需要手动进行清理
    *   方式
        *   **malloc / free**
        *   **new / delete**  
            new 与 malloc 的区别在于，new 在分配内存完成之后会调用构造函数。
    *   缺点
        *   如果不及时清理，则会占用内存，或者导致内存泄漏
        *   如果不小心提前清理，则会导致野指针
*   UE4 提供共享指针库来管理内存，它是 C++11 智能指针的自定义实现
    *   分类
        *   **TSharedPtr**
        *   **UniquePtr**
        *   **TWeakPtr**
        *   **TSharedRef**
    *   优点
        *   **防止内存泄漏** 共享引用不存在时，智能指针（弱指针除外）会自动删除对象。
        *   **弱引用** 弱指针会中断引用循环并阻止悬挂指针。
        *   **可选择的线程安全** 虚幻智能指针库包括线程安全代码，可跨线程管理引用计数。如无需线程安全，可用其换取更好性能。
        *   **运行时安全** 共享引用从不为空，可固定随时取消引用。
        *   **授予意图** 可轻松区分对象所有者和观察者。
        *   **内存** 智能指针在 64 位下仅为 C++ 指针大小的两倍（加上共享的 16 字节引用控制器）。唯一指针除外，其与 C++ 指针大小相同。

# 共享指针 TSharedPtr

*   TSharedPtr 不能指向 UObject。如果想要指向 UObject，可以使用 TWeakObjectPtr
*   TSharedPtr 可以对 FStructures 使用

## 创建 / 初始化 / 重置

*   `MakeShareable()/MakeShared<T>()` 函数
    
*   `Reset()` 函数
    
    ```
    class SimpleObject {
    public:
    	SimpleObject() { UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"SimpleObject Construct")); }
    	~SimpleObject() { UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"SimpleObject Destruct")); }
    	void ExeFun() { UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"Execute")); }
    };
    ```
    
    ```
    // 快速创建共享指针
    TSharedPtr<SimpleObject> simObjectPtr(new SimpleObject());
    // MakeShareable 创建共享指针
    TSharedPtr<SimpleObject> simObjectPtr2 = MakeShareable(new SimpleObject());
    // 创建线程安全
    TSharedPtr<SimpleObject, ESPMode::ThreadSafe> simObjectPtr3 = MakeShareable(new SimpleObject());
    // 查看引用计数
    
    UE_LOG(LogTemp, Warning,
    	TEXT(__FUNCTION__"引用计数: simObjectPtr[%d], simObjectPtr2[%d], simObjectPtr3[%d] "),
    simObjectPtr.GetSharedReferenceCount(), simObjectPtr2.GetSharedReferenceCount(), simObjectPtr3.GetSharedReferenceCount());
    
    // 重置共享指针
    simObjectPtr.Reset();
    simObjectPtr2 = nullptr;
    ```
    
    ![[b1adbee81e2ad0d03af6b3e44dff4e41_MD5.png]]
    

## 复制 / 转移

*   赋值
    
*   MoveTemp / MoveTempIfPossible
    
    ```
    // 复制共享指针
    TSharedPtr<SimpleObject> simObjectPtr_copy = simObjectPtr;
    UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"引用计数: simObjectPtr[%d], simObjectPtr_copy[%d],"),
    	simObjectPtr.GetSharedReferenceCount(), simObjectPtr_copy.GetSharedReferenceCount());
    
    // 转移共享指针
    TSharedPtr<SimpleObject> simObjectPtr_MoveTemp = MoveTemp(simObjectPtr_copy);  // 另 MoveTempIfPossible()
    UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"引用计数: simObjectPtr[%d], simObjectPtr_copy[%d], simObjectPtr_MoveTemp[%d]"),
    	simObjectPtr.GetSharedReferenceCount(), simObjectPtr_copy.GetSharedReferenceCount(), simObjectPtr_MoveTemp.GetSharedReferenceCount()
    ```
    
    ![[5ad0851ece9ef9722328639bfb3da2e0_MD5.png]]
    

## 条件判断 / 对比 / 解引用与访问

*   `->` 运算符
    
*   `Get()` 函数
    
*   `IsValid()` 函数
    
*   `==` `!=` 运算符
    
    ```
    if (simObjectPtr)					// 条件判断
    {
    	simObjectPtr->ExeFun();			// 解引用
    }
    if (simObjectPtr.Get() != nullptr)	// 条件判断
    {
    	simObjectPtr.Get()->ExeFun();	//解引用
    }
    if (simObjectPtr.IsValid())			// 条件判断
    {
    	(*simObjectPtr).ExeFun();		// 解引用
    }
    if (simObjectPtr == simObjectPtr_copy)	// 对比
    {
    	UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"simObjectPtr_copy == simObjectPtr"));
    }
    ```
    
    ![[c17a68d25f12801ac30b4b26206640eb_MD5.png]]
    

# 共享引用 TSharedRef

*   共享引用不可为空
*   不可用于 UObject 对象
*   没有 IsValid() 函数

## 创建 / 初始化

*   `MakeShareable()/MakeShared<T>()` 函数
    
    ```
    // 创建共享引用
    TSharedRef<SimpleObject> objRef(new SimpleObject());
    TSharedRef<SimpleObject> objRef2 = MakeShareable(new SimpleObject());
    TSharedRef<SimpleObject> objRef3 = MakeShared<SimpleObject>();
    ```
    

## TSharedRef 与 TSharedPtr 转换

*   隐式转化
    
*   `ToSharedRef()`
    
    ```
    // TSharedRef -> TSharedPtr
    TSharedPtr<SimpleObject> objPtr = objRef;
    
    // TSharedPtr -> TSharedRef
     objRef3= objPtr.ToSharedRef();
    ```
    

## 比较

*   没有 IsValid() 函数
    
    ```
    // 共享指针比较
    if (objRef == objRef3)
    {
    	UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"objRef == objRef3 , 引用计数：%d"), objPtr.GetSharedReferenceCount());
    }
    ```
    
    ![[c476999fd034bdb72ef8e069c9baa2b6_MD5.png]]
    

# 弱指针 TWeakPtr

*   与 TSharedPtr 相比，不参与引用计数
*   对象不存在共享指针时，TWeakPtr 将自动失效
*   使用时需要判断有效性

## 创建 / 初始化 / 转换 / 重置

*   通过 TSharedPtr 创建
    
*   通过 TSharedRef 创建
    
*   运算符 `=` 赋值
    
*   `IsValid()` 函数判断有效性
    
*   `Pin()` 函数转成 TSharedPtr , 再解引用访问对象
    
*   `Reset()` 或 `nullptr` 重置
    
    ```
    // 强指针创建弱指针
    TSharedPtr<SimpleObject> ObjPtr=MakeShared<SimpleObject>();
    TWeakPtr<SimpleObject> ObjWeakPtr(ObjPtr);
    UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"step1 引用计数：ObjPtr[%d]"), ObjPtr.GetSharedReferenceCount());
    
    //强引用创建弱指针
    TSharedRef<SimpleObject> objRef = MakeShareable(new SimpleObject());
    TWeakPtr<SimpleObject> ObjWeakPtr2(objRef);
    
    TWeakPtr<SimpleObject> ObjWeakPtr_Copy = ObjWeakPtr;
    UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"step2 引用计数：ObjPtr[%d]"), ObjPtr.GetSharedReferenceCount());
    
    // 判断有效性
    if (ObjWeakPtr.IsValid())
    {
    	TSharedPtr<SimpleObject> ObjPtr2 = ObjWeakPtr.Pin();
    	ObjPtr2->ExeFun();
    }
    
    // 清空强指针
    ObjPtr.Reset();
    TSharedPtr<SimpleObject> ObjPtr2 = ObjWeakPtr.Pin();
    UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"step3 引用计数：ObjPtr[%d]"), ObjPtr.GetSharedReferenceCount());
    
    // 判断有效性
    if (!ObjPtr2)
    {
    	UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"弱指针已空 "));
    }
    
    // 重置
    ObjWeakPtr.Reset();
    ObjWeakPtr_Copy = nullptr;
    ```
    
    ![[968a88be7a341ca5c7a4a16c6a275be0_MD5.png]]
    

# 唯一指针 TUniquePtr

*   TUniquePtr 指向的对象只能被唯一指向，因而 Unique 指针不能赋值给其它指针
*   不要为共享指针或共享引用引用的对象创建唯一指针

## 创建 / 初始化 / 判断 / 解引用 / 重置

*   MakeUnique()
    
*   IsValid()
    
*   `->` 运算符
    
*   `Get()` 函数
    
*   `Release()` 释放并返回指针
    
*   `Reset()` 或 `nullptr` 重置
    
    ```
    // 创建唯一指针
    	TUniquePtr<SimpleObject> ObjUniquePtr = MakeUnique<SimpleObject>();
    	UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__" Validity: ObjUniquePtr[%d]"), ObjUniquePtr.IsValid());
    
    	// 判断有效性
    	if (ObjUniquePtr.IsValid()) 
    	{
    		ObjUniquePtr->ExeFun(); // 解引用
    	}
    
    	// 释放指针，移交
    	TUniquePtr<SimpleObject> ObjUniquePtr2(ObjUniquePtr.Release());
    	UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__" Validity: ObjUniquePtr[%d], ObjUniquePtr2[%d]"), ObjUniquePtr.IsValid(), ObjUniquePtr2.IsValid());
    
    	// 重置
    	ObjUniquePtr.Reset();		
    	ObjUniquePtr2 = nullptr;
    ```
    
    ![[9c7c64764b95fb87ac99e1e551146652_MD5.png]]
    

# 基类与派生类的智能转换

## 共享指针转换

*   派生类转基类 隐式转换
    
*   基类转派生类 StaticCastSharedPtr
    
*   非常量转常量 ConstCastSharedPtr
    
```c++
TSharedPtr<SimpleObject> simpleObj;
TSharedPtr<ComplexObject> complexObj = MakeShared<ComplexObject>();

// 派生类转基类
simpleObj = complexObj;
UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"simpleObj is %s"), simpleObj.IsValid() ? TEXT("Valid") : TEXT("Not Valid"));

// 基类转派生类
TSharedPtr<ComplexObject> complexObj2 = StaticCastSharedPtr<ComplexObject>(simpleObj);
UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"complexObj2 is %s"), complexObj2.IsValid() ? TEXT("Valid") : TEXT("Not Valid"));

// 常量指针转非常量指针

const TSharedPtr<SimpleObject> simpleObj_const(new SimpleObject());
TSharedPtr<SimpleObject> simpleObj_mutable = ConstCastSharedPtr<SimpleObject>(simpleObj_const);
UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__"simpleObj_mutable is %s"), simpleObj_mutable.IsValid() ? TEXT("Valid") : TEXT("Not Valid"));
```
    

![[ea4025f32fd90d7225cbb290d392b7a0_MD5.png]]

## 共享引用转换

*   隐式转换
*   StaticCastSharedRef

// 创建唯一指针  
TUniquePtr ObjUniquePtr = MakeUnique();  
UE_LOG(LogTemp, Warning, TEXT(**FUNCTION**"Validity: ObjUniquePtr[%d]"), ObjUniquePtr.IsValid());

```
// 判断有效性
if (ObjUniquePtr.IsValid())
{
	ObjUniquePtr->ExeFun(); // 解引用
}

// 释放指针，移交
TUniquePtr<SimpleObject> ObjUniquePtr2(ObjUniquePtr.Release());
UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__" Validity: ObjUniquePtr[%d], ObjUniquePtr2[%d]"), ObjUniquePtr.IsValid(), ObjUniquePtr2.IsValid());

// 重置
ObjUniquePtr.Reset();
ObjUniquePtr2 = nullptr;
```

*   ConstStaticCastSharedRef

代码省略

# 助手类 TSharedFromThis

*   自定义类继承 `TSharedFromThis` 模板类
*   `TSharedFromThis` 会保存一个**弱引用**，可以通过弱指针转换成共享指针。
    *   `AsShared()` 将裸指针转换为共享引用，可再隐式转为共享指针
    *   `SharedThis(this)` 会返回具备 "this" 类型的 TSharedRef

*   不要在构造函数中调用 `AsShared` 或 `Shared`，共享引用此时并未初始化，将导致崩溃或断言
    
```c++
class BaseClass : public TSharedFromThis<BaseClass>
{
    public:
        void printf(){};
}

void NewMain()
{
    //
    TSharedFromThis<BaseClass> A = MakeShareable(new BaseClass());
    A->printf();

    BaseClass* B = A.Get();
    if(B)
    {
        A->AsShared();    
    }
}
```


```c++
// 基类
class BaseClass :public TSharedFromThis<BaseClass>
{
public:
    BaseClass() { UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__)); }
    virtual ~BaseClass() { UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__)); }
    virtual void ExeFun() { 
        TSharedRef<BaseClass> ThisAsSharedRef = AsShared();
    }
};

// 派生类
class ChildClass :public BaseClass 
{
public:
    ChildClass() { UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__)); }
    virtual ~ChildClass() { UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__)); }
    virtual void ExeFun() override{
         //AsShared()返回 TSharedRef<BaseClass>, 因而编译不通过
        //TSharedRef<ChildClass> AsSharedRef = AsShared(); 

        TSharedRef<ChildClass> AsSharedRef = SharedThis(this);
    }
};
```
    
```c++
TSharedPtr<BaseClass> BaseClassPtr = MakeShared<BaseClass>();
UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__" 引用计数：BaseClassPtr[%d]"), BaseClassPtr.GetSharedReferenceCount());

BaseClass* tempPtr = BaseClassPtr.Get();
TSharedPtr<BaseClass> BaseClassPtr_Shared =tempPtr->AsShared();
UE_LOG(LogTemp, Warning, TEXT(__FUNCTION__" 引用计数：BaseClassPtr[%d], BaseClassPtr_Shared[%d]"), 
    BaseClassPtr.GetSharedReferenceCount(), BaseClassPtr_Shared.GetSharedReferenceCount());

// 使用下面语句运行，程序死机
// TSharedPtr<BaseClass> BaseClassPtr_New = MakeShareable(tempPtr);
```
    ![[6a59e5818b262084b4dca42dadd8be69_MD5.png]]

# 注意

*   避免将数据作为 TSharedRef 或 TSharedPtr 参数传到函数，此操作将因取消引用和引用计数而产生开销。相反，建议将引用对象作为 const & 进行传递。
*   共享指针与虚幻对象 (UObject 及其衍生类) 不兼容。引擎具有 UObject 管理的单独内存管理系统（对象处理文档），两个系统未互相重叠。

## 实践遇到的问题

*   按照附录源码头文件 Tip，智能指针其实可以作为函数参数的
    
*   智能指针在 TArray 里排序或者堆操作时，符号重载 operator 要在全局来写，原类里的貌似不识别
    
    ```
    //用于排序比较
     bool operator <(const TSharedPtr<FPathPoint>& a,const TSharedPtr<FPathPoint>& b) 
     { 
     	return a->total_cost < b->total_cost; 
     }
    
     //用于数组里的查找
     bool operator ==(const TSharedPtr<FPathPoint>& a,const TSharedPtr<FPathPoint>& b) 
     { 
     	return a->pos.Equals(b->pos, 1.0f); 
     }
    ```
    
*   智能指针和原生指针混用，容易扑街，可能会遇到内存释放等问题
    