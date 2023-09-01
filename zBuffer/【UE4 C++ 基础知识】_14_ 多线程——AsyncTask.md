# 概念

## AsyncTask

*   AsyncTask 系统是一套基于线程池的异步任务处理系统。每创建一个 AsyncTask，都会被加入到线程池中进行执行
*   AsyncTask 泛指 `FAsyncTask` 和 `FAutoDeleteAsyncTask`，一般声明为友元函数，FNonAbandonableTask 派生类作为模板
*   `FNonAbandonableTask` 是要继承的基类，不可被终止的任务，可以看作是任务执行体的抽象

### FAsyncTask

*   模板类
*   `DoWork()` 实现具体任务，自定义类作为模板参数
*   FAsyncTask 初始化后，默认添加到线程池 FQueuedThreadPool。可以指定其他线程池
    *   `StartBackgroundTask()` 将任务扔到线程池中去执行
    *   `StartSynchronousTask()` 直接在当前线程执行
*   `EnsureCompletion()` 可以等待任务完成
*   执行结束需要手动停止或删除任务

### FAutoDeleteAsyncTask

*   与 FAsyncTask 类似
*   执行结束自动删除伍。执行完成后，会通过线程池的 Destroy 函数删除，或者 DoWork 完成后删除

## FQueuedThreadPool 线程池

*   FQueuedThreadPool: 虚基类，定义线程池常用的接口。FQueueThreadPoolBase 继承 FQueuedThreadPool，实现具体的方法。
*   FQueueThreadPoolBase 维护了多个线程 FQueuedThread 与多个任务队列 IQueuedWork
    *   TArray<IQueuedWork*> QueuedWork(要被执行的任务)
    *   TArray<FQueuedThread*> QueuedThreads（空闲的线程）
    *   TArray<FQueuedThread*> AllThreads（所有的线程）。
*   线程池里的线程都是 FQueuedThread 类型，FQueuedThread 是继承自 FRunnable 的线程执行体
*   FRunnableThread 表示线程本身，该类会派生出平台相关的子类

## FEvent

*   FEvent: 虚基类，提供了事件操作的接口，用于线程的挂起 / 唤醒（Wait()/Trigger()）

![](https://img2020.cnblogs.com/blog/2369154/202105/2369154-20210501173612974-1718593868.png)

# 使用方式

## 简单使用方式 Lambda

*   可指定线程
*   可根据指定线程用于多线程调用主线程里的东西
    
    ```
    AsyncTask(ENamedThreads::GameThread, [=]()
    {
     // Do Something
    });
    ```
    

## 手动销毁方式：FNonAbandonableTask 与 FAsyncTask

*   SimpleAsyncTask 类
    
    ```
    class  SimpleAsyncTask :FNonAbandonableTask
    {
    	FString m_TaskName;
    	int32 m_MaxNumber;
    	friend class FAsyncTask<SimpleAsyncTask>;
    public:
    	// 构造函数
    	SimpleAsyncTask(const FString& TaskName, int32 MaxNumber)
    		: m_TaskName(TaskName), m_MaxNumber(MaxNumber) {}
    	// 析构函数
    	~SimpleAsyncTask() 
    	{
    		UE_LOG(LogTemp, Warning, TEXT("Task[%s] End"), *m_TaskName);
    	}
    
    	// 具体任务逻辑执行
    	void DoWork() {
    		UE_LOG(LogTemp, Warning, TEXT("Task[%s] Start"), *m_TaskName);
    		int CurrentNum = 0;
    		while (CurrentNum < m_MaxNumber)
    		{
    			CurrentNum++;
    			UE_LOG(LogTemp, Warning, TEXT("Task[%s] CurrentNum:%d"), *m_TaskName, CurrentNum);
    		}
    	}
    
    	// 固定写法，本类将作为函数参数
    	FORCEINLINE TStatId GetStatId() const {
    		RETURN_QUICK_DECLARE_CYCLE_STAT(SimpleAsyncTask, STATGROUP_ThreadPoolAsyncTasks);
    	}
    };
    ```
    
*   调用任务的 Actor——AAsyncTaskActor
    
    ```
    UFUNCTION(BlueprintCallable)
    	void CreateAsyncTask(const FString& TaskName, int32 MaxNumber);
    ```
    
    ```
    void AAsyncTaskActor::CreateAsyncTask(const FString& TaskName, int32 MaxNumber)
    {
    	UE_LOG(LogTemp, Warning, TEXT("Task[%s] Created"), *TaskName);
    	FAsyncTask<SimpleAsyncTask>* MyTask = new FAsyncTask<SimpleAsyncTask>(TaskName, MaxNumber);
    	MyTask->StartBackgroundTask();
    	
    	// MyTask->StartSynchronousTask; 在当前线程执行，可能会导致主线程阻塞
    	// MyTask->IsDone()  可以配合定时器检测是否完成任务
    
    	//等待任务完成后，进行手动删除
    	MyTask->EnsureCompletion();
    	delete MyTask;
    	MyTask = nullptr;	
    }
    ```
    

## 自动销毁方式：FNonAbandonableTask 与 FAutoDeleteAsyncTask

*   AutoDeleteSimpleAsyncTask 类
    
    ```
    class  AutoDeleteSimpleAsyncTask :FNonAbandonableTask
    {
    	FString m_TaskName;
    	int32 m_MaxNumber;
    	friend class FAutoDeleteAsyncTask<AutoDeleteSimpleAsyncTask>;
    public:
    	// 构造函数
    	AutoDeleteSimpleAsyncTask(const FString& TaskName, int32 MaxNumber)
    		: m_TaskName(TaskName), m_MaxNumber(MaxNumber) {}
    	// 析构函数
    	~AutoDeleteSimpleAsyncTask()
    	{
    		UE_LOG(LogTemp, Warning, TEXT("Task[%s] End"), *m_TaskName);
    	}
    
    	// 具体任务逻辑执行
    	void DoWork() {
    		UE_LOG(LogTemp, Warning, TEXT("Task[%s] Start"), *m_TaskName);
    		int CurrentNum = 0;
    		while (CurrentNum < m_MaxNumber)
    		{
    			CurrentNum++;
    			UE_LOG(LogTemp, Warning, TEXT("AutoDeleteTask[%s] CurrentNum:%d"), *m_TaskName, CurrentNum);
    		}
    	}
    
    	// 固定写法，本类将作为函数参数
    	FORCEINLINE TStatId GetStatId() const {
    		RETURN_QUICK_DECLARE_CYCLE_STAT(SimpleAsyncTask, STATGROUP_ThreadPoolAsyncTasks);
    	}
    };
    ```
    
*   调用任务的 Actor——AAsyncTaskActor
    
    ```
    UFUNCTION(BlueprintCallable)
    	void CreateAutoDeleteAsyncTask(const FString& TaskName, int32 MaxNumber);
    ```
    
    ```
    void AAsyncTaskActor::CreateAutoDeleteAsyncTask(const FString& TaskName, int32 MaxNumber)
    {
    	UE_LOG(LogTemp, Warning, TEXT("AutoDeleteTask[%s] Created"), *TaskName);
    	// 任务完成后，自动删除
    	(new FAutoDeleteAsyncTask<AutoDeleteSimpleAsyncTask>(TaskName, MaxNumber))->StartSynchronousTask();
    }
    ```
    

![](https://img2020.cnblogs.com/blog/2369154/202105/2369154-20210501174457351-1532704507.png)

# 参考

*   [FAsyncTask](https://docs.unrealengine.com/en-US/API/Runtime/Core/Async/FAsyncTask/index.html)
*   [FAutoDeleteAsyncTask](https://docs.unrealengine.com/en-US/API/Runtime/Core/Async/FAutoDeleteAsyncTask/index.html)
*   [FNonAbandonableTask](https://docs.unrealengine.com/en-US/API/Runtime/Core/Async/FNonAbandonableTask/index.html)
*   [Legacy/Using AsyncTasks](https://ue4community.wiki/legacy/using-asynctasks-1jpclff4)
*   [《Exploring in UE4》多线程机制详解 [原理分析]](https://zhuanlan.zhihu.com/p/38881269)
*   \EpicGames\UE_4.26\Engine\Source\Runtime\Core\Public\Async\AsyncWork.h