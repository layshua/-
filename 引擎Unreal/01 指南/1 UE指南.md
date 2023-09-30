---
title: 1 UE指南
create_time: 2023-09-13 20:35
uid: "202309132035"
reference: []
---
# 操作
## 常用快捷键
![[Pasted image 20221213161143.png]]
![[Pasted image 20221213161538.png]]

## 1 中英文快速切换
~控制台

culture=zh-Hans    =简体中文

culture=en             =英文作者

先手动设置一次，然后用鼠标宏即可
鼠标宏：左侧上键

插件：Switch Language
![[Pasted image 20230219154339.png]]
## 2 缩放
字体缩放：ctrl+shift+W
蓝图编辑器已超过 1：1 的比例缩放：ctrl+鼠标滚轮

## 5 修改锚点
[UE4中修改模型mesh资源的枢轴坐标原点位置](https://blog.csdn.net/Time_Waxk/article/details/108592797)
鼠标中间选中枢轴白点，可以整体移动
鼠标停在 xyz 轴上，按住 alt+鼠标中键可以按轴移动
最后右键保存锚点
![[Pasted image 20221222145956.png]]
## 6 对齐
**表面对齐**：按 End
使 Actor 对齐到地面或其他表面
![[Pasted image 20230112184158.png]]
**网格对齐**：![[Pasted image 20230112184344.png]]
**点对齐**：移动时按住 V
## 7 测量
视口按住鼠标中键盘
## 迁移资产
**资源类型**
![[Pasted image 20221212103413.png]]


**手动迁移：**
迁移文件时项目文件中的缓存文件要删掉：
![[Pasted image 20221212103817.png]]

**使用内置迁移功能（优先选择）：**
（1）单个文件夹
![[Pasted image 20221212103920.png|300]]
右键选择迁移→选择文件夹中所需迁移对象→迁移到该项目的 content 文件夹（一般打开就是）

（2）非文件夹素材 (可同时多选)
![[Pasted image 20221212104055.png|300]]
右键选资产操作→选迁移→The same as ↑

（3）复制粘贴
用于网络下载的素材
不推荐使用，若实在不能迁移，则使用。

需要复制原素材 content 文件里面的内容，到目标项目 content 文件内。
![[Pasted image 20221212104204.png]]
缺点：文件容易重名，可打开 UE4，将原 content 里面的内容全部放在一个文件夹里，这样只用复制一个文件夹，不会导致最后的重名问题。

## 导入模型

（1）建模软件导出的格式选择
![[Pasted image 20221212104556.png]]

（2）静态网格体和骨骼网格体

静态网格体: 静态的，不需要动画的，例如桌子房子摆件等

骨骼网格体：有动画有骨骼，例如人物、机械臂

（3）UE4 导入静态网格体（可导入可拖拽）

网格体：默认静态网格体。如果静态网格体勾选了骨骼，UE4 会自动给模型生成骨骼。

LOD：可自主选择

生成光照贴图：ue 自动帮我们生成

合并网格体：如果在建模软件中没有合并，选择这个将所有模型变成一个整体。

法线导入方向：如果导入的模型出现黑边等问题，则可以更改此处选择。

法线生成方法：如果导入的模型出现黄色警告，可以更改此处为内置，就会忽略黄色警告。黄色警告问题不大。

杂项: 对模型的向上轴、单位做调整（导出时就应调整）。UE4 默认 Z 轴向上，单位默认厘米。maya 默认 y 轴朝上，可以修改一下以匹配 ue4

Material： 材质贴图

FBX 文件信息: 可以查看你导入的模型基本信息（模型单元，轴方向）。

（4）UE4 导入骨骼网格体

①网格体：

骨架选项：可选择现有骨架

使用 TO 作为参考姿势：用动画的第 0 帧作为参考姿势，如果不选择，可能身体会变形，骨骼可能会丢失。

导入变形目标：如果有顶点动画的模型导入，需要勾选此选项。例如：有面目表情的。

法线等选项同静态网格。

②创建物理资源：就是给骨架网格体添加碰撞。

③Animation 选项:

动画：如果导入的模型有动画，则勾选。

FBX 文件信息: 虚幻默认动画帧率为 30 帧。

一般选择默认，后面根据需求。

## 安装插件
**安装到虚幻引擎（推荐）**
转到虚幻引擎安装位置的插件文件夹 Engine/Plugins
将解压得到的插件文件夹放入 Marketplace 文件夹下（如没有新建一个）。
启动虚幻引擎打开项目，菜单->编辑->插件里可查看到插件。

**安装到项目**
转到项目的根目录（`[ProjectName]. uproject` 文件所在的文件夹）
创建一个名为 Plugins 的文件夹（如果没有），将解压得到的插件文件夹放入 Plugins 文件夹。
启动虚幻引擎打开项目，菜单->编辑->插件里可查看到插件。

**将插件修改为默认启动：**
1. 打开插件的 uplugin 文件
![[Pasted image 20230913203502.png]]
2. 添加
```c++
"EnabledByDefault" : true,
```
## 编译版本独立程序
复制一份 BlankProgram，命名为 TestCpp
![[Pasted image 20230902135816.png]]

复制的文件中的文件名全部改为 TestCpp
![[Pasted image 20230902140147.png]]
文件中的 BlankProgram 全部替换为 TestCpp
![[Pasted image 20230902140006.png]]

然后生成项目
![[Pasted image 20230902140302.png]]

## 删除缓存
当我们向 build. cs 中添加内容，比如增删模块，可以删除一下文件刷新项目
Binaries 二进制文件
Intermediate 中间文件
Saved 已保存文件

然后右键
![[Pasted image 20230904172126.png]]
![[Pasted image 20230904172112.png]]

然后启动项目，确认重建模块
![[Pasted image 20230904172226.png]]



## 编辑器配置文件修改
### 默认配置
首先因为 UE4 的中文翻译不全面，以及学习资料很多是英文，为了方便我还是喜欢用英文界面。

有几个编辑器的偏好设置，每次新建项目都要修改，不如改引擎的默认配置文件，或者启动器的默认配置文件。

路径：
![[Pasted image 20230906195131.png]]


2. 鼠标控制标签位置：Editor Preferences->LevelEditor->Play，Play in Editor->Mouse Control Label Position，鼠标控制提醒标签，默认在左上角，经常挡住屏幕输出信息（PrintString 的输出），可改为右下角
```
[/Script/UnrealEd. LevelEditorPlaySettings]
MouseControlLabelPosition=LabelAnchorMode_BottomRight
```

3. 在编译时保存：Editor Preferences->Content Editors->Blueprint Editor，Compiler->Save on Compile，**默认是编译成功不保存，可以改为 On Success Only**。配置文件中“[/Script/BlueprintGraph. BlueprintEditorSettings]”节不存在，要新增
```
[/Script/BlueprintGraph. BlueprintEditorSettings]
SaveOnCompile=SoC_SuccessOnly
```

4. 资产编辑器打开路径：**资产编辑器打开位置设置为主窗口：**
```
[/Script/UnrealEd. EditorStyleSettings]
AssetEditorOpenLocation=MainWindow
```

# 功能

## 3 新建关卡为黑色的解决方法
[【UE5】（初学）定制彩色星空+解决新建关卡是黑的的问题_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Df4y1o7fP/?from=seopage&vd_source=9d1c0e05a6ea12167d6e82752c7bc22a)
内容浏览器中-》视图选项-》勾选显示引擎内容，在引擎内容中找到 EngineSky 文件夹，之后就找到 sky_Sphere 了
## 4 修改场景默认 pawn 的方法

playstart 方法

1. 放置 playstart 到场景中

2. 修改 world settings 的 GameMode


直接放置 pawn 方法

1. 放置 pawn 到场景中

2. 修改 world settings 的 GameMode

3. 修改 pawn 的 Auto Possess Player 为 Player 0


## 8 缩略图 Thumbnail
缩略图即在内容浏览器中的图标
![[Pasted image 20230116221435.png]]
生成 Level 的缩略图：
![[Pasted image 20230116221617.png]]

蓝图和材质的缩略图：
内容浏览器右上角 Setting
![[Pasted image 20230116221716.png]]
左键单击，进行旋转。右键单击拖动，进行缩放。
完成后，单击 Done Editing 完成编辑
![[Pasted image 20230116221853.png]]

Niagara 的缩略图：
点击后会补货 Preview 窗口中的图像
![[Pasted image 20230116222000.png]]
![[Pasted image 20230116222033.png]]

项目的缩略图：
项目设置->Project->Description
![[Pasted image 20230116222207.png]]
## 9 单位和坐标系
在 Unity 中，主要的测量单位是一米。
在虚幻引擎中，主要测量单位是**一厘米**。

Unity 和虚幻引擎都使用左手坐标系，但坐标轴的命名方式不同。**虚幻引擎 Z 的正方向是"上"。**

## 10 Live Coding 乱码
命令行输出的信息是 UTF8 编码， LiveCoding 用的是系统编码，所以最终导致了 LiveCoding 是乱码。

方案一：直接修改系统编码。会导致部分软件乱码

![[Pasted image 20230829165213.png]]


方案二：LiveCoding 的输出结果会储存到一个文件里，路径是
```
C:\Users\LiuKe\AppData\Local\UnrealBuildTool\Log.txt
```
用 IDE 打开这文件就能看到非乱码的输出了, 来替代livecoding



# 编辑器偏好设置
## 新选项卡位置
资产编辑器打开路径
General->Appearance
![[Pasted image 20230112184037.png]]
## 编译时保存
General->Blueprint Editor Settings
![[Pasted image 20230112174340.png]]
或者
![[Pasted image 20230112183915.png]]
## 显示帧率和内存
General->Performance
![[Pasted image 20230112174431.png]]
## 切换 IDE
General->SourceCode
![[Pasted image 20230112174507.png]]
# 项目设置
## 缓存位置
### 项目缓存
把`D:\Program Files\Epic Games\UE_5.0\Engine\Config\BaseEngine. ini`文件

里面的：Path="%ENGINEVERSIONAGNOSTICUSERDIR%DerivedDataCache"

替换为：Path="%GAMEDIR%DerivedDataCache"

以后每次创建项目都会在“项目名称”的目录下自动生成一个“DerivedDataCache”文件夹
### 库缓存目录
打开 epic 客户端，左下角设置
![[Pasted image 20230825232148.png]]

## 默认 map/gamemode
Project->Maps&Modes
![[Pasted image 20230112175527.png]]
## 渲染设置
Engine->Rendering
主要包含各种渲染特性的设置

**虚拟纹理 Virtual Texture**
![[Pasted image 20230112180208.png]]
在 Editor->Texture import 中设置自动虚拟纹理大小
![[Pasted image 20230112181729.png]]

**全局光照 GI**->开启 Lumen
![[Pasted image 20230112180257.png]]

**Lumen**
![[Pasted image 20230112180502.png]]

**软/硬光线追踪**
![[Pasted image 20230112180518.png]]

**软光追->生成网格体距离场**
![[Pasted image 20230112180657.png]]

**Nanite**
![[Pasted image 20230112180542.png]]

**后处理->启用自定义深度-模板通道**
![[Pasted image 20230112180839.png]]

**抗锯齿**
Default Setting->
![[Pasted image 20230112180952.png]]

**Strata 材质**
![[Pasted image 20230112181122.png]]
## 一般设置
Engine->General Settings

**帧率**
![[Pasted image 20230112181346.png]]


# 控制台命令
~“键调用控制台命令 

## 技巧
键盘 ↑键可以看到之前输入过的指令
控制台指令并不需要打全名，空格+指令可以模糊搜索
Ctrl + Shift + 逗号，打开 GPUProfile 面
## 调试类

stat fps —— 显示帧率
stat unit —— 显示包括 Draw Call ，游戏逻辑等各种项的消耗
stat UnitGraph —— 显示各个参数的实时曲线图
stat rhi —— 显示各种 GPU 上的消耗细则
stat game —— 显示当前帧的时间信息
stat Engine —— 显示帧数时间，三角面数等
stat scenerendering —— 显示 Drawcall
ShowFlag. Bounds —— 显示包围盒（0 关闭，1 打开）
ShowFlag. Collision—— 显示碰撞盒（0 关闭，1 打开）
r.visualizeOccludedPrimitives —— 查看遮挡剔除（0 关闭，1 打开）
DisableAllScreenMessages/EnableAllScreenMessages —— 关闭/打开屏幕打印信息
ke * rendertextures 高分辨率截图
HighResShot  —— 以当前 viewport 分辨率的倍数进行截图（倍数）
HighResShot 3840x2160 —— 指定分辨率截图（分辨率）【图片存储位置：\Saved\Screenshots\】
stat Hitches —— 或者 stat DumpHitches 记录 log 文件
stat startfile / stat stopfile —— 开始/结束统计性能分析，用于记录某段时间内的性能分析数据。【此时会在路径 Saved/Profiling/UnrealStats 下生成数据文件。】
## 图像类
r.vsync 0—— 垂直同步（0 关闭，1 打开）
r.Tonemapper. Sharpen 3 —— 锐化（锐化强度）
r.Streaming. PoolSize 4096 —— 显存多少分配给了纹理流送池（显存大小，0 表示无上限）
r.ScreenPercentage 50 —— 设置渲染分辨率为默认大小的 50%
r.TonemapperFilm —— 开关后处理效果
ShowFlag. PostProcessing —— 开关后期盒子效果
r.AOSpecularOcclusionMode —— 可以让 skylight 产生的 DFAO 产生更准确的高光（0 关闭，1 打开）
## 游戏类
slomo —— 游戏运行速度（运行速度）