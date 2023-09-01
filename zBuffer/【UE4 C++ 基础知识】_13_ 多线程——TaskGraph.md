# 概述

*   TaskGraph 系统是 UE4 一套抽象的异步任务处理系统
*   TaskGraph 可以看作一种” 基于任务的并行编程 “设计思想下的实现
*   通过 TaskGraph ，可以创建任意多线程任务， 异步任务， 序列任务， 并行任务等，并可以指定任务顺序， 设置任务间的依赖， 最终形成一个任务图， 该系统按照设定好的依赖关系来分配任务图中的任务到各个线程中执行， 最终执行完整个任务图。
*   TaskGraph 适合简单的任务或者想实现有依赖关系的线程，复杂的任务推荐使用 Runnable 或者 AsynTask

# TaskGraph 类定义

## 模块构成

自定义的任务必须要满足 TGraphTask 中对 Task 的接口需求

*   构造函数可以传参，最好不要使用引用类型，会有” 悬空引用 “的风险，可以使用指针来代替引用
    
*   `GetStatId()` 固定写法，函数内传入自定义 TaskGraph 类型
    
*   `GetDesiredThread()` 指定在哪个线程运行
    
    *   `ENamedThreads::Type`
        *   `AnyThread`
            
        *   `GameThread` 适合访问 UObject，可能会阻塞主线程
            
        *   `RHIThread`
            
        *   `AudioThread`
            
    *   也可以通过 `FAutoConsoleTaskPriority` 对象获取合适的线程
*   `GetSubsequentsMode()` 后续执行模式，因为可以有子任务
    
    *   `ESubsequentsMode::TrackSubsequents` 存在后续任务，实际没有后续任务也不影响，常用该类型
        
    *   `ESubsequentsMode::FireAndForget` 没有后续任务
        
*   `DoTask()` 线程逻辑执行函数
    

## TaskGraph 简单实现

*   FTaskGraph_SimpleTask 任务类
    
    ```
    class FTaskGraph_SimpleTask
    {
    	FString m_ThreadName;
    
    public:
    	FTaskGraph_SimpleTask(const FString& ThreadName) : m_ThreadName(ThreadName) {}
    	~FTaskGraph_SimpleTask(){}
    
    	// 固定写法
    	FORCEINLINE TStatId GetStatId() const {
    		RETURN_QUICK_DECLARE_CYCLE_STAT(FTaskGraph_SimpleTask, STATGROUP_TaskGraphTasks);
    	}
    
    	// 指定在哪个线程运行
    	static ENamedThreads::Type GetDesiredThread() { return ENamedThreads::AnyThread; }
    
    	// 后续执行模式
    	static ESubsequentsMode::Type GetSubsequentsMode() { return ESubsequentsMode::TrackSubsequents; }
    
    	// 线程逻辑执行函数
    	void DoTask(ENamedThreads::TypeCurrentThread, const FGraphEventRef& MyCompletionGraphEvent) {
    		// 逻辑任务
    		// 可创建 Child Task 
    		UE_LOG(LogTemp, Warning, TEXT("Thread %s Begin!"), *m_ThreadName);
    		UE_LOG(LogTemp, Warning, TEXT("Thread %s End!"), *m_ThreadName);
    	}
    };
    ```
    
*   ATaskGraphActor 调用的 AActor
    
    ```
    UFUNCTION(BlueprintCallable)
    		void CreateTaskGraph_SimpleTask(const FString& ThreadName);
    ```
    
    ```
    void ATaskGraphActor::CreateTaskGraph_SimpleTask(const FString& ThreadName)
    {
    	TGraphTask<FTaskGraph_SimpleTask>::CreateTask().ConstructAndDispatchWhenReady(ThreadName); // ThreadName 为 FTaskGraph_SimpleTask 构造函数参数
    }
    ```
    
    ![](https://img2020.cnblogs.com/blog/2369154/202104/2369154-20210430231057033-1835107324.png)
    

## 了解任务的创建

在上一小节中，我们可以使用以下代码创建任务

```
FGraphEventRef GraphEventRef = TGraphTask<FTaskGraph_SimpleTask>::CreateTask().ConstructAndDispatchWhenReady(ThreadName); // 创建任务立即执行
TGraphTask<FTaskGraph_SimpleTask>*  GraphTask = TGraphTask<FTaskGraph_SimpleTask>::CreateTask().ConstructAndHold(ThreadName); // 创建任务挂起，等待 unlock() 触发任务执行
```

*   `TGraphTask<T>` 是模板类，可以指定自定义任务的类型
    
*   `CreateTask()`
    
    *   第一个参数 `Prerequisites`
        *   用来指定该任务依赖的事件数组，默认为 NULL.
        *   在所有依赖事件都触发后，该任务才会放到任务队列里面分配给线程执行。
    *   第二个参数 `ENamedThreads::Type`
        *   用来指定线程类型
    
    ```
    // 完整函数为
    static FConstructor CreateTask(const FGraphEventArray* Prerequisites = NULL, ENamedThreads::Type CurrentThreadIfKnown = ENamedThreads::AnyThread)
    ```
    
*   `ConstructAndDispatchWhenReady()`
    
    *   创建任务后立即执行
    *   调用创建的 TaskGraph 类型（本例中为 FTaskGraph_SimpleTask）的构造函数
    *   可以传递构造函数的参数，进行初始化
*   `ConstructAndHold()`
    
    *   创建任务后挂起，等待 unlock() 唤醒任务执行
    *   参数同上

# TaskGraph 并行任务

## 具有依赖关系的并行任务

*   如图所示，B 依赖于 A0 和 A1，即 A0、A1 执行完毕后 B 才开始执行
*   C0 和 C1 依赖于 B 的完成

![](https://img2020.cnblogs.com/blog/2369154/202104/2369154-20210430231545691-1793635807.png)

![](https://img2020.cnblogs.com/blog/2369154/202104/2369154-20210430231554631-1010905172.png)

## 派发任务

*   如图所示，虚线框为派发子任务
*   B 的开始依赖于 A0 和 A1 的完成
*   B 有两个子任务 B0 和 B1，B 的完成 需要满足 B0 和 B1 也完成
*   C0 和 C1 的开始依赖于 B 的完成，因而 C0 和 C1 的开始 需要满足 B、B0 和 B1 都完成

![](https://img2020.cnblogs.com/blog/2369154/202104/2369154-20210430231601490-175410015.png)

![](https://img2020.cnblogs.com/blog/2369154/202104/2369154-20210430231609953-1672164350.png)

## C++ 代码

*   TaskGraph_SimpleTask.h

```
#pragma once
#include "CoreMinimal.h"
#include "TaskGraph_SimpleTask.generated.h"

DECLARE_DELEGATE_OneParam(FGraphTaskDelegate,const FString&); // 单播委托

USTRUCT(BlueprintType)
struct FTaskGraphItem {													// 结构体用来传参
	GENERATED_USTRUCT_BODY()

public:
	UPROPERTY(BlueprintReadWrite)
		FString m_ThreadName;											// 线程名称

	FGraphEventRef m_GraphEventRef;										// 自动执行的任务
	TGraphTask<class FTaskGraphWithPrerequisitesAndChild>* m_GraphTask; // 需要触发执行的任务

	// 构造函数
	FTaskGraphItem(FString ThreadName = TEXT("None"), FGraphEventRef GraphEventRef = nullptr, TGraphTask<class FTaskGraphWithPrerequisitesAndChild>* GraphTask = nullptr)
		:m_ThreadName(ThreadName), m_GraphEventRef(GraphEventRef), m_GraphTask(GraphTask) {}

	~FTaskGraphItem()
	{
		m_GraphEventRef = nullptr;
		m_GraphTask = nullptr;
	}
};

class FTaskGraph_SimpleTask  // 作为具体执行的任务
{
	FString m_ThreadName;
	FGraphTaskDelegate m_GraphTaskDelegate; 

public:
	FTaskGraph_SimpleTask(const FString& ThreadName, FGraphTaskDelegate GraphTaskDelegate) 
		: m_ThreadName(ThreadName), m_GraphTaskDelegate(GraphTaskDelegate) {}
	~FTaskGraph_SimpleTask(){}

	// 固定写法
	FORCEINLINE TStatId GetStatId() const {
		RETURN_QUICK_DECLARE_CYCLE_STAT(FTaskGraph_SimpleTask, STATGROUP_TaskGraphTasks);
	}

	// 指定在主线程，因为用到 AActor 蓝图里的函数
	static ENamedThreads::Type GetDesiredThread() { return ENamedThreads::GameThread; }

	// 后续执行模式
	static ESubsequentsMode::Type GetSubsequentsMode() { return ESubsequentsMode::TrackSubsequents; }

	// 线程逻辑执行函数
	void DoTask(ENamedThreads::Type CurrentThread, const FGraphEventRef& MyCompletionGraphEvent) {
		check(IsInGameThread()); //确认是否在主线程
		FString message = FString::Printf(TEXT("SimpleTaskTask[%s] execute the GraphTaskDelegate!"), *m_ThreadName);
		m_GraphTaskDelegate.ExecuteIfBound(message);
		//UE_LOG(LogTemp, Warning, TEXT("SimpleTaskTask[%s] execute!"), *m_ThreadName);
	}
};

class FTaskGraphWithPrerequisitesAndChild  // 作为通用任务，可作为依赖事件的任务，也可作为子任务
{

	FString m_ThreadName;	
	TArray<TGraphTask<FTaskGraphWithPrerequisitesAndChild>*> m_ChildGraphTask;	// 子任务数组
	FGraphTaskDelegate m_GraphTaskDelegate; // 单播委托

public:
	// 构造函数
	FTaskGraphWithPrerequisitesAndChild(const FString& ThreadName, const TArray<TGraphTask<FTaskGraphWithPrerequisitesAndChild>*>& ChildTask, FGraphTaskDelegate GraphTaskDelegate)
		: m_ThreadName(ThreadName), m_ChildGraphTask(ChildTask), m_GraphTaskDelegate(GraphTaskDelegate) {}
	
	~FTaskGraphWithPrerequisitesAndChild() {}

	// 固定写法
	FORCEINLINE TStatId GetStatId() const {
		RETURN_QUICK_DECLARE_CYCLE_STAT(FTaskGraphWithPrerequisitesAndChild, STATGROUP_TaskGraphTasks);
	}

	// 指定在哪个线程运行
	static ENamedThreads::Type GetDesiredThread() { return ENamedThreads::AnyThread; }

	// 后续执行模式
	static ESubsequentsMode::Type GetSubsequentsMode() { return ESubsequentsMode::TrackSubsequents; }

	// 线程逻辑执行函数
	void DoTask(ENamedThreads::Type CurrentThread, const FGraphEventRef& MyCompletionGraphEvent) { 

		UE_LOG(LogTemp, Warning, TEXT("Task[%s] Begin!"), *m_ThreadName);
		// 执行子任务，此处通用任务作为子任务
		if (m_ChildGraphTask.Num()>0)
		{
			for (auto GraphTaskItem : m_ChildGraphTask)
			{
				GraphTaskItem->Unlock();  // 唤醒子任务
				MyCompletionGraphEvent->DontCompleteUntil(GraphTaskItem->GetCompletionEvent());				
			}
			// 如有需要，可设法检测所有子任务是否都完成
		}

		// 创建并执行子任务，本处作为具体执行的任务
		MyCompletionGraphEvent->DontCompleteUntil(TGraphTask<FTaskGraph_SimpleTask>::CreateTask().ConstructAndDispatchWhenReady(m_ThreadName, m_GraphTaskDelegate));
		UE_LOG(LogTemp, Warning, TEXT("Task[%s] End!"), *m_ThreadName);
	}
};
```

*   TaskGraphActor.h

```
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "TaskGraph_SimpleTask.h"
#include "TaskGraphActor.generated.h"

UCLASS()
class TIPS_API ATaskGraphActor : public AActor
{
	GENERATED_BODY()	
public:	
	ATaskGraphActor();
protected:
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// 创建任务
	UFUNCTION(BlueprintCallable)
		FTaskGraphItem CreateGraphTask(const FString& ThreadName, const TArray<FTaskGraphItem>& Prerequisites,const TArray<FTaskGraphItem>& ChildTasks,bool DispatchWhenReady );

	// 创建任务，CreateGraphTask 的简化
	UFUNCTION(BlueprintCallable)
		FTaskGraphItem CreateGraphTaskPure(const FString& ThreadName, bool DispatchWhenReady) {
		return CreateGraphTask(ThreadName, TArray<FTaskGraphItem>(), TArray<FTaskGraphItem>(), DispatchWhenReady);
	}

	// 唤醒挂起的任务
	UFUNCTION(BlueprintCallable)
		void TriggerGraphTask(FTaskGraphItem TaskGraphItem);

	// 用于任务中执行的回调函数
	UFUNCTION(BlueprintCallable, BlueprintImplementableEvent)
		void OnTaskFinished(const FString& message);
}
```

*   TaskGraphActor.cpp

```
FTaskGraphItem ATaskGraphActor::CreateGraphTask(const FString& ThreadName, const TArray<FTaskGraphItem>& Prerequisites, const TArray<FTaskGraphItem>& ChildTasks, bool DispatchWhenReady)
{
	FGraphEventArray PrerequisiteEvents;  // 依赖事件
	TArray<TGraphTask<FTaskGraphWithPrerequisitesAndChild>*> ChildGraphTask; // 子任务
	UE_LOG(LogTemp, Warning, TEXT("Task[%s] is Created!"), *ThreadName);
	if (Prerequisites.Num()>0)
	{
		for (FTaskGraphItem item : Prerequisites) // 结构体数组提取依赖事件
		{
			if (item.m_GraphEventRef)
			{
				PrerequisiteEvents.Add(item.m_GraphEventRef); 
				UE_LOG(LogTemp, Warning, TEXT("Task[%s] wait Task[%s]!"), *ThreadName,*item.m_ThreadName);
			}
			else if (item.m_GraphTask)
			{
				PrerequisiteEvents.Add(item.m_GraphTask->GetCompletionEvent());
				UE_LOG(LogTemp, Warning, TEXT("Task[%s] wait Task[%s]!"), *ThreadName,*item.m_ThreadName);
			}
		}
	}
	if (ChildTasks.Num()>0)
	{
		for (FTaskGraphItem item : ChildTasks) // 提取子任务
		{
			if (item.m_GraphTask)
			{
				ChildGraphTask.Add(item.m_GraphTask);
				UE_LOG(LogTemp, Warning, TEXT("Task[%s] is Task[%s] child task!"), *item.m_ThreadName, *ThreadName);
			}
		}
	}

	FGraphTaskDelegate GraphTaskDelegate = FGraphTaskDelegate::CreateUObject(this, &ATaskGraphActor::OnTaskFinished);
	if (DispatchWhenReady)
	{	// 创建立即执行的任务，返回结构体参数
		return FTaskGraphItem(ThreadName, TGraphTask<FTaskGraphWithPrerequisitesAndChild>::CreateTask(&PrerequisiteEvents).ConstructAndDispatchWhenReady(ThreadName, ChildGraphTask, GraphTaskDelegate));
	}
		// 创建任务后挂起，等待触发，返回结构体参数
	return  FTaskGraphItem(ThreadName, nullptr, TGraphTask<FTaskGraphWithPrerequisitesAndChild>::CreateTask(&PrerequisiteEvents).ConstructAndHold(ThreadName, ChildGraphTask, GraphTaskDelegate));
}

void ATaskGraphActor::TriggerGraphTask(FTaskGraphItem TaskGraphItem)
{
	if (TaskGraphItem.m_GraphTask)
	{
		TaskGraphItem.m_GraphTask->Unlock();
		UE_LOG(LogTemp, Warning, TEXT("Task %s Trigger!"), *TaskGraphItem.m_ThreadName);
	}
	
}
```

# 扩展

## ParallelFor

*   基于 TaskGraph
*   多次调用函数体，可以做简单的遍历处理
*   bForceSingleThread 设置单线程还是多线程

```
ParallelFor
(
    int32 Num,
    TFunctionRef < void )> Body,
    bool bForceSingleThread,
    bool bPumpRenderingThread
)

void ParallelForWithPreWork
(
    int32 Num,
    TFunctionRef < void )> Body,
    TFunctionRef < void ()> CurrentThreadWorkToDoBeforeHelping,
    bool bForceSingleThread,
    bool bPumpRenderingThread
)
```

```
ParallelFor(100, [](int32 CurrIdx) {
    int32 Sum = 0;
    for (int32 Idx = 0; Idx < CurrIdx * 100; ++Idx)
        Sum += FMath::Sqrt(1234.56f);
});
```

## AsyncTask

*   本质上使用 TaskGraph

```
//异步执行一个Function 函数指针
void AsyncTask(ENamedThreads::Type Thread, TUniqueFunction<void()> Function) {
	TGraphTask<FAsyncGraphTask>::CreateTask().ConstructAndDispatchWhenReady(Thread, MoveTemp(Function));
}
```

```
//异步执行一个 Lambda 表达式
void AsyncTask(ENamedThreads::Type Thread, [&](){} );
```

# 参考

*   [Legacy/Multi-Threading: Task Graph System](https://ue4community.wiki/legacy/multi-threading:-task-graph-system-pah8k101)
*   [虚幻 4 与现代 C++：基于任务的并行编程与 TaskGraph 入门](https://neil3d.github.io/unreal/mcpp-task-begining.html)
*   [虚幻 4 与现代 C++：使用 TaskGraph 实现 Fork-Join 模型](https://neil3d.github.io/unreal/mcpp-fork-join.html)
*   [UE4 C++ 进阶 07 异步操作 - 基于 TaskGraph 的多线程](https://www.bilibili.com/video/BV1fh411S7dL/)
*   [虚幻 4 Task Graph System 介绍](http://altdev.io/2016/11/24/intro-ue4-task-graph-system/)
*   [ParallelFor](https://docs.unrealengine.com/en-US/API/Runtime/Core/Async/ParallelFor/index.html)
*   [ParallelForWithPreWork](https://docs.unrealengine.com/en-US/API/Runtime/Core/Async/ParallelForWithPreWork/2/index.html)