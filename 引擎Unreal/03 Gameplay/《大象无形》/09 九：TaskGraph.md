本节主要探讨虚幻引擎的 TaskGraph 系统，在原书并未提及，是我根据 如下几篇文章学到的知识做的笔记：

[UE 并发 - TaskGraph 的实现和用法](https://zhuanlan.zhihu.com/p/398843895) by： [davidpp](https://www.zhihu.com/people/davidpp)

[UE4/UE5 的 TaskGraph](https://zhuanlan.zhihu.com/p/408012121) by： [quabqi](https://www.zhihu.com/people/quabqi)

感谢这些文章的优秀作者，那么，让我们开始吧！

## 概述

首先 TaskGraph 是 UE 中基于任务的并发机制。可以创建任务在指定类型的线程中执行，同时提供了等待机制，其强大之处在于可以调度一系列有依赖关系的任务，这些任务组成了一个有向无环的任务网络（DAG），并且任务的执行可以分布在不同的线程中。

![[08e7d8c800977b63212b192aa7123dc6_MD5.jpg]]

## DAG 任务测试

让我们从测试代码开始吧，我们要实现一个如上图所示的任务 DAG 有向图，要怎么做呢，首先我们利用上一节介绍的虚幻引擎的自动化测试系统：Automation System 来做代码例子。（如果不熟悉可以参照上一节[大象无形 UE 笔记八：并行与并发](https://zhuanlan.zhihu.com/p/574216210)）。

### 输入代码

首先我们在项目目录的 Source/MyGame/Private/Tests/ 文件夹创建了一个空的 TaskDagTest.cpp 文件，并右击 MyGame.uproject，重新生成了项目文件 MyGame.sln，然后用 Visual Studio 打开项目，如下图：

![[418fcbc9a242bdc39b0f89b6002271b2_MD5.jpg]]

然后，我们在这个文件中输入如下测试代码：

```
#include "CoreMinimal.h"
#include "HAL/ThreadManager.h"

DEFINE_LOG_CATEGORY_STATIC(TestLog, Log, All);

IMPLEMENT_SIMPLE_AUTOMATION_TEST(FTaskDagTest, "MyTest.PublicTest.TaskDagTest", EAutomationTestFlags::EditorContext | EAutomationTestFlags::EngineFilter)

// 一次性任务的类
class FGraphTaskSimple
{
public:
    FGraphTaskSimple(const TCHAR* TheName, int InSomeArgument, float InWorkingTime = 1.0f)
        : TaskName(TheName), SomeArgument(InSomeArgument), WorkingTime(InWorkingTime)
    {
        Log(__FUNCTION__);
    }

    ~FGraphTaskSimple()
    {
        Log(__FUNCTION__);
    }

    FORCEINLINE TStatId GetStatId() const {
        RETURN_QUICK_DECLARE_CYCLE_STAT(FGraphTaskSimple, STATGROUP_TaskGraphTasks);
    }

    // AnyThread中运行
    static ENamedThreads::Type GetDesiredThread() {
        return ENamedThreads::AnyThread;
    }

    // FireAndForget：一次性任务，没有依赖关系
    static ESubsequentsMode::Type GetSubsequentsMode() {
        return ESubsequentsMode::FireAndForget;
    }

    // 执行任务
    void DoTask(ENamedThreads::Type CurrentThread, const FGraphEventRef& MyCompletionGraphEvent) {
        // The arguments are useful for setting up other tasks. 
        // Do work here, probably using SomeArgument.
        FPlatformProcess::Sleep(WorkingTime);
        Log(__FUNCTION__);
    }

public:
    // 自定义参数
    FString TaskName;   // 任务名，比如TaskA, TaskB等
    int SomeArgument;   // 整形参数，暂时只用来打印log
    float WorkingTime;  // 工作时长，在DoTask时Sleep用，模拟执行任务的时间

    // 日志接口
    void Log(const char* Action) {
        uint32 CurrentThreadId = FPlatformTLS::GetCurrentThreadId();
        FString CurrentThreadName = FThreadManager::Get().GetThreadName(CurrentThreadId);
        UE_LOG(TestLog, Display, TEXT("%s@%s[%d] - %s, SomeArgument=%d"), *TaskName, *CurrentThreadName,
            CurrentThreadId,
            ANSI_TO_TCHAR(Action), SomeArgument);
    }
};

// 支持依赖关系的复合任务类
class FCombTask : public FGraphTaskSimple
{
public:
    using FGraphTaskSimple::FGraphTaskSimple;

    FORCEINLINE TStatId GetStatId() const {
        RETURN_QUICK_DECLARE_CYCLE_STAT(FGraphTask, STATGROUP_TaskGraphTasks);
    }

    static ENamedThreads::Type GetDesiredThread() {
        return ENamedThreads::AnyThread;
    }

    // TrackSubsequents - 支持依赖检查
    static ESubsequentsMode::Type GetSubsequentsMode() {
        return ESubsequentsMode::TrackSubsequents;
    }
};

// 测试主函数
bool FTaskDagTest::RunTest(const FString& Param)
{
	FGraphEventRef TaskEventA, TaskEventB, TaskEventC, TaskEventD, TaskEventE;

	// TaskA
        TaskEventA = TGraphTask<FCombTask>::CreateTask().ConstructAndDispatchWhenReady(TEXT("TaksA"), 1, 1);
        TaskEventA->SetDebugName(TEXT("TaksA"));

	// TaskB 依赖 TaskA
	{
		FGraphEventArray Prerequisites;
		Prerequisites.Add(TaskEventA);
                TaskEventB = TGraphTask<FCombTask>::CreateTask(&Prerequisites).ConstructAndDispatchWhenReady(TEXT("TaksB"), 1, 1);
                TaskEventB->SetDebugName(TEXT("TaksB"));
	}

	// TaskC 依赖 TaskB
	{
		FGraphEventArray Prerequisites;
		Prerequisites.Add(TaskEventB);
                TaskEventC = TGraphTask<FCombTask>::CreateTask(&Prerequisites).ConstructAndDispatchWhenReady(TEXT("TaksC"), 1, 1);
                TaskEventC->SetDebugName(TEXT("TaksB"));
	}

	// TaskD 依赖 TaskA
	{
		FGraphEventArray Prerequisites;
		Prerequisites.Add(TaskEventA);
                TaskEventD = TGraphTask<FCombTask>::CreateTask(&Prerequisites).ConstructAndDispatchWhenReady(TEXT("TaksD"), 1, 3);
                TaskEventD->SetDebugName(TEXT("TaksD"));
	}

	// TaskE 依赖 TaskC、TaskD
	{
		FGraphEventArray Prerequisites{ TaskEventC, TaskEventD };
                TaskEventE = TGraphTask<FCombTask>::CreateTask(&Prerequisites).ConstructAndDispatchWhenReady(TEXT("TaksE"), 1, 1);
                TaskEventE->SetDebugName(TEXT("TaksE"));
	}


	UE_LOG(TestLog, Display, TEXT("Construct is Done ......"));

	// 在当前线程等待，直到TaskE完成
        TaskEventE->Wait();
	UE_LOG(TestLog, Display, TEXT("Done ......"));
	return true;
}
```

### 运行测试

然后我们编译运行，并在 UE5 编辑器打开 “工具”--> “会话前端”：

![[c8adaba38696cbd368189747ff93abf4_MD5.jpg]]

然后在打开的 “会话前端” 窗口中，选择“自动化”Tab 按钮，然后选择 MyTest 里面的 TaskDagTest，并点击“开始测试”：

![[781eb99eef5e3a7f14bc7fc091a2b7a5_MD5.jpg]]

我们可以得到如下结果：

![[b8e3cccf86f6525b1ac2b8ecdfaa2b4c_MD5.jpg]]

从程序运行结果，再对比一下我的任务 DAG 图：

![[08e7d8c800977b63212b192aa7123dc6_MD5.jpg]]

可以得到如下结论：

（1），TaskEventA, TaskEventB, TaskEventC, TaskEventD, TaskEventE 的构造函数，都在 CreateTask().ConstructAndDispatchWhenReady(..) 中直接调用。

（2），TaskA，TaskB，TaskC 分在同一个线程：Foreground Worker #0 ，并且顺序执行。

（3），TaskD, TaskE 分在同一个线程： Foreground Workder #1，并且顺序执行。TaskD 是跟 TaskB, TaskC 并行执行的，TaskD 打印 DoTask 的 log 较晚，是因为它的执行时长为 3 秒，比如 TaskB(1 秒) + TaskC（1 秒）要长一些。

好的，我们通过实际的测试代码，对有依赖的关系的多个任务的执行有了一定的了解了。让我们开始深入理解 TaskGraph 的原理吧！

## TaskGraph 系统

### 线程类型

TaskGraph 支持两种类型线程：

1，是 TaskGraph 系统后台创建的线程，称为 AnyThread。

2，是外部线程，包括 主线程等系统线程，初始化的时候需要 Attach 到 TaskGraph 系统，此类线程成为 NamedThread。

![[659004c1faebb8b4a683bce592464246_MD5.jpg]]

TaskGraph 系统管理的 AnyThread 线程，在 UE4 中大致是由 NP(普通优先级) HP(高优先级) BP(低优先级) 三种类型的线程组成。在 UE5 由于 TaskGraph 的实现类换成了 FTaskGraphCompatibilityImplementation 类，所以 AnyThread 由 Forground Worker 和 Background Worker 两种类型的线程组成，其中，在我的 8 核 CPU 的电脑上，Forground 共有 2 个线程，用来处理高优先级的任务；而 Background 有 5 个线程，用来处理低优先级任务。

### 任务类

在 TaskGraph 中，每个任务节点，是一个 TGraphTask<> 模板类的对象。比如我们例子的图示的每个节点，都是一个 TGraphTask<FCombTask>，其中 FCombTask 是我们自己定义的任务结构体，如下图：

![[937865c64536344b05eeef0c60de6f3f_MD5.jpg]]

其中，TGraphTask<> 有如下几个重要的成员变量：

**TaskStorage**： 保存我们自定义的任务数据，比如 TaskA 保存的是我们 FCombTask 类的对象。

**NumberOfPrerequistitesOutstanding**： 前置任务个数，这个为 0 的时候，执行当前任务。

**Subsequents**：任务事件，即 FGraphEvent 对象，也是我们通过调用 ConstructAndDispatchWhenReady 函数得到的 TaskEventA 等对象，注意这个对象里面有一个 **SubsequentList** 成员变量**，**即它的后置任务队列。

也就是说 TaskA 节点的完整结构是：

![[2bbdc577f0f288670e54239defde315d_MD5.jpg]]

如上图，由于 TaskA 并没有前置任务，所以当调用 ConstructAndDispatchWhenReady 函数后，NumberOfPrerequistitesOutstanding 成员变量为 0，它有两个后置任务，分别是 TaskB, TaskD，所以它的 SubsuentList 的队列有两个元素，分别是 TGraphTaskB 和 TGraphTaskD，为了方便，我们可以简化为右边的图。

于是我们可以画出这个完整的 DAG：

![[e147eadfb9d9ddb44a17c1159aaefc57_MD5.jpg]]

我们看看这个任务 DAG 的执行过程，首先我们发现 TaskA 的 “前置任务个数” 为 0，则执行 TaskA，当 TaskA 完成后，我们会遍历它的 “后置任务列表” 的所有任务，在这里是 TaskB 和 TaskD 两个，并递减这两个后置任务的“前置任务个数”，递减为 0 时，会执行这个后置任务：

![[ee9bdf008f0c51ad048ed3e329766a59_MD5.jpg]]

然后我们分别执行 TaskB 和 TaskD，当 TaskB 执行结束后，会遍历它的 “后置任务列表” 的所有任务，这里是 TaskC 一个，递减 TaskC 的“前置任务个数”，在这里递减为 0，会开始执行 TaskC：

![[9e4e20b69fddb06c4e8e82f521a66dfc_MD5.jpg]]

然后我们完成 TaskD 后，把它的 “后置任务列表” 的 TaskE 的 “前置任务个数” 递减 1，这里 2-1 = 1，所以 TaskE 还不能马上执行。需要等到 TaskC 执行完毕后，再把 TaskC 的后置任务 TaskE 的 “前置任务个数” 递减 1，这时 TaskE 的 “前置任务个数” 为 0，则开始执行 TaskE。

这一块的 C++ 代码主要在这里：

![[7af9a511d004dcc106f3ce3e5a8468a5_MD5.jpg]]

如上代码，在调用一个任务的 Subsequncts->DispatchSubsequents 函数时，会把它的 “后置任务列表” 的所有任务的 “前置任务个数” 递减，并在递减到 0 时执行这个任务。

## 总结

好的，我们的 TaskGraph 暂时讲到这里了，首先我们做了一个有任务依赖关系的 DAG 图的完整代码例子，并了解了 TaskGraph 管理的内部与外部线程，也探讨了任务图节点的具体实现类 TGraphTask<>，以及派发有依赖关系的任务的原理。

下一节我们会开始讲解 UObject 系统，敬请期待！