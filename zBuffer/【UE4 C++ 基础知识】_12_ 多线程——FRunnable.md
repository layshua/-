# 概述

*   UE4 里，提供的多线程的方法：
    *   继承 `FRunnable` 接口创建单个线程
    *   创建 `AsyncTask` 调用线程池里面空闲的线程
    *   通过 `TaskGraph` 系统来异步完成一些自定义任务
    *   支持原生的多线程 `std::thread`
*   在 GameThread 线程之外的其他线程中
    *   不要 spawning / modifying / deleting UObjects / AActors
    *   不要使用定时器 _TimerManager_
    *   不要使用任何绘制接口，例如 _DrawDebugLine，然有可能崩溃_
    *   如果想做的话，可以在主线程中异步处理
    *   其他线程中一般做数据收发和解析，数学运算等

本文主要介绍 FRunnable 类

# FRunnable

*   FRunnable 是 UE4 中多线程的实现方式之一，适用于复杂运算
    
*   FRunnable 是线程的执行体，提供相应的接口。FRunnable 需要依附与一个 FRunnableThread 对象，才能被执行
    
    ```
    class CORE_API FRunnable
    {
    public:
        // ....
        virtual bool Init(); // 初始化 runnable 对象，在FRunnableThread创建线程对象后调用
    
        virtual uint32 Run() = 0; // Runnable 对象逻辑处理主体，在Init成功后调用
    
        virtual void Stop() {} // 停止 runnable 对象, 线程提前终止时被用户调用
    
        virtual void Exit() {} // 退出 runnable 对象，由FRunnableThread调用
    };
    ```
    
*   FRunnableThread 表示一个可执行的线程，该类会派生出平台相关的子类。通过调用 `FRunnableThread::Create` 完成线程的创建
    

![](https://img2020.cnblogs.com/blog/2369154/202104/2369154-20210430012219695-768566798.png)

# 快速创建一个线程

## 创建 FRunnable 派生类

```
// .h 
class TIPS_API FSimpleRunnable: public FRunnable
{
public:
	FSimpleRunnable(const FString& ThreadName);
	~FSimpleRunnable();
	void PauseThread();				// 线程挂起 方法一 
	void WakeUpThread();			// 线程唤醒 方法一
	void Suspend(bool bSuspend);	// 线程挂起/唤醒 方法二
	void StopThread();				// 停止线程，一般用该方法
	void ShutDown(bool bShouldWait);// 停止线程，bShouldWait true的时候可强制 kill 线程

private:
	FString m_ThreadName;
	int32 m_ThreadID;
	bool bRun = true;				// 线程循环标志
	bool bPause = false;			//线程挂起标志
	FRunnableThread* ThreadIns;		// 线程实例
	FEvent* ThreadEvent;			//FEvent指针,挂起/激活线程, 在各自的线程内使用

	virtual bool Init() override;
	virtual uint32 Run() override;
	virtual void Stop() override;
	virtual void Exit() override;
};
```

```
// .cpp
FSimpleRunnable::FSimpleRunnable(const FString& ThreadName)
{
	// 获取 FEvent 指针
	ThreadEvent = FPlatformProcess::GetSynchEventFromPool();

	// 创建线程实例
	m_ThreadName = ThreadName;
	ThreadIns = FRunnableThread::Create(this, *m_ThreadName, 0, TPri_Normal);
	m_ThreadID = ThreadIns->GetThreadID();
	UE_LOG(LogTemp, Warning, TEXT("Thread Start! ThreadID = %d"), m_ThreadID);
}

FSimpleRunnable::~FSimpleRunnable()
{
	if (ThreadEvent)	// 清空 FEvent*
	{
		FPlatformProcess::ReturnSynchEventToPool(ThreadEvent); // delete ThreadEvent;
		ThreadEvent = nullptr;
	}
	if (ThreadIns)		// 清空 FRunnableThread*
	{
		delete ThreadIns; 
		ThreadIns = nullptr;
	}
}

bool FSimpleRunnable::Init()
{
	return true; //若返回 false ,线程创建失败，不会执行后续函数
}

uint32 FSimpleRunnable::Run()
{
	int32 count = 0;
	FPlatformProcess::Sleep(0.03f); //延时，等待初始化完成
	while (bRun) 
	{
		if (bPause)
		{
			ThreadEvent->Wait(); // 线程挂起
			if (!bRun)			 // 线程挂起时执行线程结束
			{
				return 0;
			}
		}	

		UE_LOG(LogTemp, Warning, TEXT("ThreadID: %d, Count: %d"),m_ThreadID, count);
		count++;
		FPlatformProcess::Sleep(0.1f); // 执行间隔，防止堵塞
	}
	return 0;
}

void FSimpleRunnable::Stop()
{
	bRun = false;
	bPause = false;
	if (ThreadEvent)
	{
		ThreadEvent->Trigger(); // 保证线程不挂起
	}	
	Suspend(false); // 保证线程不挂起，本例只是为了暂时不同的挂起方法，如果不使用Suspend(),无需使用
}

void FSimpleRunnable::Exit()
{
	UE_LOG(LogTemp, Warning, TEXT("Thread Exit!"));
}

void FSimpleRunnable::PauseThread()
{
	bPause = true;
	UE_LOG(LogTemp, Warning, TEXT("Thread Pause!"));
}

void FSimpleRunnable::WakeUpThread()
{
	bPause = false;
	if (ThreadEvent)
	{
		ThreadEvent->Trigger(); // 唤醒线程
	}	
	UE_LOG(LogTemp, Warning, TEXT("Thread Wakeup!"));
}

void FSimpleRunnable::Suspend(bool bSuspend)
{
	if (ThreadIns)
	{
		ThreadIns->Suspend(bSuspend); //挂起/唤醒
	}
}

void FSimpleRunnable::StopThread()
{
	Stop();
	ThreadIns->WaitForCompletion(); // 等待线程执行完毕
}

void FSimpleRunnable::ShutDown(bool bShouldWait)
{
	if (ThreadIns)
	{
		ThreadIns->Kill(bShouldWait); // bShouldWait 为false，Suspend(true)时，会崩
	}
}
```

## 创建调用多线程的 Actor

```
// .h
protected:
	virtual void EndPlay(const EEndPlayReason::Type EndPlayReason) override;

private:
	FSimpleRunnable* SimpleRunnable;

public:
	UFUNCTION(BlueprintCallable)
		void CreateNewThread(const FString& ThreadName);

	UFUNCTION(BlueprintCallable)
		void PauseThread();

	UFUNCTION(BlueprintCallable)
		void SuspendThread(bool bSuspend);

	UFUNCTION(BlueprintCallable)
		void WakeUpThread();

	UFUNCTION(BlueprintCallable)
		void StopThread();

UFUNCTION(BlueprintCallable)
		void ForceKillThread(bool bShouldWait);
};
```

```
// .cpp
void ARunnableActor::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
	if (SimpleRunnable) // 防止线程挂起，退出无响应
	{
		SimpleRunnable->StopThread();
		delete SimpleRunnable;
		SimpleRunnable = nullptr;
	}
}

void ARunnableActor::CreateNewThread(const FString& ThreadName)
{
	SimpleRunnable = new FSimpleRunnable(ThreadName);
}

void ARunnableActor::PauseThread()
{
	if (SimpleRunnable)
	{
		SimpleRunnable->PauseThread();
	}
}

void ARunnableActor::SuspendThread(bool bSuspend)
{
	if (SimpleRunnable)
	{
		SimpleRunnable->Suspend(bSuspend);
	}
}

void ARunnableActor::WakeUpThread()
{
	if (SimpleRunnable)
	{
		SimpleRunnable->WakeUpThread();
	}
}

void ARunnableActor::StopThread()
{
	if (SimpleRunnable)
	{
		SimpleRunnable->StopThread();
	}
}

void ARunnableActor::ForceKillThread(bool bShouldWait)
{
	if (SimpleRunnable)
	{
		SimpleRunnable->ShutDown(bShouldWait);
		delete SimpleRunnable;
		SimpleRunnable = nullptr;
	}
}
```

![](https://img2020.cnblogs.com/blog/2369154/202104/2369154-20210430012248857-99562402.png)

## 单例线程

*   当希望线程只能创建一次时，可以通过声明静态单例 FRunnable （本例为 FSimpleRunnable）

```
// .h
static FSimpleRunnable* MySimpleRunnable; // 声明静态单例
static FSimpleRunnable* JoyInit();  // 声明静态方法

// cpp 
// 初始化静态单例
FSimpleRunnable* FSimpleRunnable::MySimpleRunnable = nullptr;
//创建 SimpleRunnable 实例
FSimpleRunnable* FSimpleRunnable::JoyInit()
{
	if (!MySimpleRunnable && FPlatformProcess::SupportsMultithreading())
	{
		MySimpleRunnable = new FSimpleRunnable();
	}
	return MySimpleRunnable;
}
```

## 多个线程

当希望执行多个线程时

*   可用 TMap<Name, FRunnable> 存储，移除
*   也可设定线程结束条件，让其自行结束线程

# 线程锁

UE4 线程锁包括：

*   **FSpinLock 自旋锁**
*   **FScopeLock 区域锁**
*   **FCriticalSection 临界区**
*   **FRWLock 读写锁**

本文使用 **FScopeLock 、FCriticalSection** 作为测试

## 不使用线程锁

本例使用两个线程 为同一个整数做加法，知道该整数到达目标值

*   修改 SimpleRunnable 代码
    
    ```
    FSimpleRunnable(const FString& ThreadName, int32* CurrentNumber, int32 MaxNumber);
    
    int32* m_CurrentNumber;
    int32 m_MaxNumber;
    int32 m_CalcCount = 0;
    ```
    
    ```
    FSimpleRunnable::FSimpleRunnable(const FString& ThreadName, int32* CurrentNumber, int32 MaxNumber)
    {
    	/* 省略部分代码 */
    	m_CurrentNumber = CurrentNumber;
    	m_MaxNumber = MaxNumber;
    	/* 省略部分代码 */
    }
    
    uint32 FSimpleRunnable::Run()
    {
    
    	FPlatformProcess::Sleep(0.03f); //延时，等待初始化完成
    	while (bRun && *m_CurrentNumber<m_MaxNumber) 
    	{
    		/* 省略部分代码 */
    		(*m_CurrentNumber)++;		
    		m_CalcCount++;
    		if (m_CalcCount % 100 == 0)
    		{
    			UE_LOG(LogTemp, Warning, TEXT("ThreadID: %d, CurrentNumber: %d"),m_ThreadID, *m_CurrentNumber);
    		}
    		FPlatformProcess::Sleep(0.0001f); // 执行间隔，防止堵塞
    	}
    	return 0;
    }
    
    void FSimpleRunnable::Exit()
    {
    	UE_LOG(LogTemp, Warning, TEXT("Thread Exit! ThreadID: %d, CurrentNumber: %d, CalcCount: %d"),m_ThreadID, *m_CurrentNumber, m_CalcCount);
    }
    ```
    
*   修改 RunnableActor 代码
    
    ```
    UPROPERTY(EditAnywhere)
    		int32 m_MaxNumber = 1000;
    ```
    
    ```
    void ARunnableActor::CreateNewThread(const FString& ThreadName)
    {
    	SimpleRunnable = new FSimpleRunnable(TEXT("Thread1"), &m_CurrentNumber, m_MaxNumber);
    	SimpleRunnable = new FSimpleRunnable(TEXT("Thread2"), &m_CurrentNumber, m_MaxNumber);
    }
    ```
    
    ![](https://img2020.cnblogs.com/blog/2369154/202104/2369154-20210430012305479-1145267983.png)
    

## 使用线程锁

*   注意 FCriticalSection 是否使用 static 声明

### **FScopeLock**

*   方法一
    
    修改 SimpleRunnable 代码
    
    ```
    { // 注意这个作用域用于 **FScopeLock** 
    	static FCriticalSection m_mutex; //声明 staic 可以让线程之间互锁
    	FScopeLock ScopeLock(&m_mutex); // 该作用域内上锁
    
    	(*m_CurrentNumber)++;
    }	
    m_CalcCount++;
    ```
    
*   方法二
    
    修改 SimpleRunnable 代码
    
    ```
    static FCriticalSection m_mutex; //声明 staic 可以让线程之间互锁
    FScopeLock* ScopeLock = new FScopeLock(&m_mutex); // 上锁
    
    (*m_CurrentNumber)++;
    
    delete ScopeLock; // 解锁
    ```
    
    ![](https://img2020.cnblogs.com/blog/2369154/202104/2369154-20210430012342823-1360551510.png)
    

### FCriticalSection Lock()/UnLock()

修改 SimpleRunnable 代码

```
// 放在类声明static ,使用 Lock() 编译不通过
// static 可以让线程之间互锁，不使用 static 锁不生效
// 不使用 static，线程内可以上锁。可以在类中声明
static FCriticalSection m_mutex;

m_mutex.Lock();  // 上锁

(*m_CurrentNumber)++;	

m_mutex.Unlock(); // 解锁
```

![](https://img2020.cnblogs.com/blog/2369154/202104/2369154-20210430012353136-626059641.png)

# 参考

*   [FRunnable](https://docs.unrealengine.com/en-US/API/Runtime/Core/HAL/FRunnable/index.html)
*   [FScopeLock](https://docs.unrealengine.com/en-US/API/Runtime/Core/Misc/FScopeLock/index.html)
*   [UE4 异步编程专题 - 多线程](https://zhuanlan.zhihu.com/p/62310930)
*   [Legacy/MultiThreading and synchronization Guide](https://www.ue4community.wiki/legacy/multithreading-and-synchronization-guide-9l0xyz17)
*   [UE4 C++ 基础教程 - 多线程](https://zhuanlan.zhihu.com/p/133921916)
*   [Exploring in UE4》多线程机制详解 [原理分析]](https://zhuanlan.zhihu.com/p/38881269)