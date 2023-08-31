
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
    
    ```c++
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
    *   `AsShared()` 将裸指针转换为共享引用，如果需要，我们可以再隐式转为共享指针
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
    //创建共享指针访问成员函数
    TSharedPtr<BaseClass> A = MakeShareable(new BaseClass());
    A->printf();

    //将A解引用，将共享指针转换为裸指针
    BaseClass* B = A.Get();

    //对于普通的类，我们如果想把B在转换为共享指针，需要再次调用MakeShareable创建新的共享指针
    //这里BaseClass继承了TSharedFromThis，因此我们可以直接将指向BaseClass的裸指针B转换为共享引用
    //通将共享引用转换为弱指针即可
    if(B)
    {
        B->AsShared();    
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
    