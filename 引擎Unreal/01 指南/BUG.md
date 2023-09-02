
# Rider DirectX 报错
## 问题:

UE5.2 版本下 Rider 启动项目会提示 `在9个文件中有20个错误`

大家第一反应可能觉得 Rider 出问题了，尝试清除缓存和重新构建索引甚至是更新和重装都不能解决问题

实际上这是因为 UE5.2 的官方构建发行版本缺少了 `DirectX` 库的 `DirectX.Build.cs`，因此这个模块也没有被 Rider 扫描进引擎项目，自然提示找不到符号

虽然不影响我们正常编译，但是报错提示让人非常不舒服

![[8bbfedb9f0b391fb132413e220894232_MD5.jpg]]

## 如何修复:

首先在 UE5.2 的源码 Github 中下载 `DirectX.Build.cs`

(如果之前没有绑定过 Epic Games 与 Github 的关联，打开源码 Github 地址可能会提示 404，可以复制后文内容)

[Build software better, together](https://github.com/EpicGames/UnrealEngine/blob/5.2/Engine/Source/ThirdParty/Windows/DirectX/DirectX.Build.cs)

将其放入路径形如 `E:\UE_5.2(UE根目录)\Engine\Source\ThirdParty\Windows\DirectX` 中

你也可以选择自己 `新建一个DirectX.Build.cs` 粘贴入以下内容，建议将文件保存为只读，保持一致性

```
// Copyright Epic Games, Inc. All Rights Reserved.

using System.IO;
using UnrealBuildTool;

public class DirectX : ModuleRules
{
	public static string GetDir(ReadOnlyTargetRules Target)
	{
		return Target.UEThirdPartySourceDirectory + "Windows/DirectX";
	}

	public static string GetIncludeDir(ReadOnlyTargetRules Target)
	{
		return GetDir(Target) + "/include";
	}

	public static string GetLibDir(ReadOnlyTargetRules Target)
	{
		return Path.Combine(GetDir(Target), "Lib", Target.Architecture.WindowsName) + "/";
	}

	public static string GetDllDir(ReadOnlyTargetRules Target)
	{
		return Path.Combine(Target.RelativeEnginePath, "Binaries/ThirdParty/Windows/DirectX", Target.Architecture.WindowsName) + "/";
	}

	public DirectX(ReadOnlyTargetRules Target) : base(Target)
	{
		Type = ModuleType.External;
	}
}
```

![[132c4a2d6201f94df556856c9160bbf9_MD5.png]]

然后在 Rider 定位到 `E:\UE_5.2\Engine\Source\ThirdParty\Windows` 对着 `Windows` 文件夹右键 `添加现有条目` 且选择 `DirectX` 文件夹即可

![[281502d8d9febadfefd4ec6edb9c953a_MD5.png]]

![[7d8d9e6190abfe77c3e3bef52ce8f40c_MD5.png]]

![[332d01d63b3b220c3c23976d25ce71a7_MD5.png]]
