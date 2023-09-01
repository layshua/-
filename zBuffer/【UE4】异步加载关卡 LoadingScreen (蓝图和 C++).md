*   一般先跳转到一个临时的关卡，然后异步加载目标关卡，同时展示 Loading 界面
*   对于含有流关卡的目标关卡，可以先载入子关卡

# 蓝图异步加载

*   无进度条

![[bff764b85e76ac94b6a8905a01071652_MD5.png]]

# C++ 异步加载关卡

## LoadPackageAsync

```
bool bIsLoaded;
	 
UPROPERTY()
	FString LoadPackagePath;

// 用于加载完成的回调
DECLARE_DYNAMIC_DELEGATE(FOnAsyncLoadFinished);

UFUNCTION(BlueprintCallable)
	void MyAsyncLoad(const FOnAsyncLoadFinished& OnAsyncLoadFinished);

UFUNCTION(BlueprintCallable)
	float GetLoadProgress();
```

```
//开始异步加载
void ALoadActor::MyAsyncLoad(const FOnAsyncLoadFinished& OnAsyncLoadFinished)
{
//不使用GetBaseFilename，编译 error C4458: LoadPackageAsync failed to begin to load a package···
	LoadPackagePath = FPaths::GetBaseFilename(LoadPackageSoftObjectPath.ToString(), false);

	bIsLoaded = false;

	UE_LOG(LogTemp, Warning, TEXT("String: %s"), *LoadPackagePath);
	LoadPackageAsync(
		LoadPackagePath,
		FLoadPackageAsyncDelegate::CreateLambda([=](const FName& PackageName, UPackage* LoadedPackage, EAsyncLoadingResult::Type Result) 
			{
				if (Result == EAsyncLoadingResult::Failed)
				{
					UE_LOG(LogTemp, Warning, TEXT("Load Failed"));
				}
				else if (Result == EAsyncLoadingResult::Succeeded)
				{
					bIsLoaded = true;
					UE_LOG(LogTemp, Warning, TEXT("Load Succeeded"));
					OnAsyncLoadFinished.ExecuteIfBound();
				}
			}), 0, PKG_ContainsMap);
}

//获取加载的进度 -1 —— 100
float ALoadActor::GetLoadProgress()
{
	float FloatPercentage = GetAsyncLoadPercentage(*LoadPackagePath);
	if (!bIsLoaded)
	{
		FString ResultStr = FString::Printf(TEXT("Percentage: %f"), FloatPercentage);
		GEngine->AddOnScreenDebugMessage(-1, 2.0f, FColor::Green, ResultStr);
		UE_LOG(LogTemp, Warning, TEXT("Percentage: %f"), FloatPercentage);
	}
	else {
		FloatPercentage = 100;
	}
	return FloatPercentage;	
}
```

![[3028266419c39433b9cdb36b3da7c023_MD5.png]]

### 效果

![[add4d9cd75cb48cfb78a99647ac0b7cf_MD5.gif]]

## FStreamableManager

*   获取依赖项
*   将依赖项转为 `FSoftObjectPath` 存储到容器中
*   使用 `RequestAsyncLoad` 加载
*   使用 `GetLoadCount` 获取加载进度

参考部分代码

![[76903491057907c5cb1e889683cfb4cd_MD5.png]]

# 参考

*   [Unreal Engine 4 —— 异步加载关卡的实现方法及思考](https://blog.csdn.net/noahzuo/article/details/59104339)
    
*   [[中文直播] 第 33 期 | UE4 资产管理基础 1 | Epic 大钊](https://www.bilibili.com/video/BV1Mr4y1A7nZ/?t=1h03m33s)
    
*   引擎版本 4.26