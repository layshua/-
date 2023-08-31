
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
    