![[Pasted image 20231003213722.png]]

# 连接Steam
打开 Steam 联机子系统插件
![[Pasted image 20231003215202.png]]

build. cs 添加模块：
```c++
"OnlineSubsystemSteam"
"OnlineSubSystem"
```

修改 DefaultEngine.ini：将以下代码复制粘贴进去即可
[虚幻引擎Online Subsystem Steam接口 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/online-subsystem-steam-interface-in-unreal-engine/)
![[Pasted image 20231003223720.png]]

代码：
```c++
//.h
TSharedPtr<class IOnlineSession, ESPMode::ThreadSafe> OnlineSessionInterface; 

//.cpp
IOnlineSubsystem* OnlineSubsystem = IOnlineSubsystem::Get(); //获取在线子系统  

if(OnlineSubsystem)  
{  
    OnlineSessionInterface = OnlineSubsystem->GetSessionInterface(); //获取用于访问会话管理服务的接口  
    
    GEngine->AddOnScreenDebugMessage(-1, 15.f, FColor::Blue, FString::Printf(TEXT("获取子系统 %s"), *OnlineSubsystem->GetSubsystemName().ToString())); //默认为SubsystemNULL  
}
```

登录 Steam 平台
![[Pasted image 20231003231740.png]]

此时 PIE 打印的名字是 NULL，**打包后运行打印的则是 Steam**。说明打包后才能获取成功

# 委托
![[Pasted image 20231003234343.png]]