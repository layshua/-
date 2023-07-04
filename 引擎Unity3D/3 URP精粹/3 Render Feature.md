

---
title: 3 Render Feature
aliases: []
tags: []
create_time: 2023-07-02 14:00
uid: 202307021400
banner: "![[Pasted image 20230702140512.png]]"
---

> [!NOTE] 简称
> 下文 Renderer Feature 简称为 RF
> RenderTexture 简称RT


1.  介绍 [URP Renderer Feature | Universal RP | 14.0.8 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/urp-renderer-feature.html)
2.  使用 [Example: How to create a custom rendering effect using the Render Objects Renderer Feature | Universal RP | 14.0.8 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/containers/how-to-custom-effect-render-objects.html)

Render Feature 是一种 Asset，用于向 URP 渲染器添加额外的 Render Pass 并配置其行为。

以下 Render Feature在 URP 中可用：
- [Render Objects 渲染对象](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-features/renderer-feature-render-objects.html)
- [Screen Space Ambient Occlusion  屏幕空间环境光遮蔽](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/post-processing-ssao.html)
- [Decal 贴花](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-feature-decal.html)
- [Screen Space Shadows 屏幕空间阴影](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-feature-screen-space-shadows.html)
- [Full Screen Pass 全屏Pass](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-features/renderer-feature-full-screen-pass.html)

# Render Objects RF
[Render Objects Renderer Feature | Universal RP | 14.0.8 --- 渲染对象渲染器功能](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-features/renderer-feature-render-objects.html)
![[Pasted image 20230702103753.png|500]]
URP 在 DrawOpaqueObjects 和 DrawTransparentObjects Pass 中绘制对象。您可能需要在帧渲染的不同点绘制对象，或者以其他方式解释和写入渲染数据（如 depth 和 stencil）。
Render Objects RF 允许通过特定的重载（overides）在指定的图层、指定的时间来自定义 Draw Objects。

## 透视效果
**实战**：当角色在GameObjects后面时，用不同的材质绘制角色轮廓。
![[character-goes-behind-object.gif]]

使用两个 Render Objects RF（命名为 RF1 和 RF2）：一个用于绘制不被遮挡的颜色，另一个用于绘制被遮挡颜色
1. 创建一个 Layer，命名为 Character
2. 将要渲染的 object 分配给该层，然后将 RF1 的 LayerMask 也设置为该层
3. 创建 Red 材质和 Blue 材质。Object 给与 Red 材质，然后Overides->Blue 材质
4. 我们要实现当角色位于其他游戏对象之后时，渲染器功能才会使用 Blue 材质渲染角色。可以通过深度测试来实现，Depth->DepthTest 设置为 Greater，这样在该 Layer 下深度大的物体绘制在前面。

**当模型复杂时，这样设置可能会发生自透视：**
![[character-depth-test-greater.gif|449]]

## 创建额外的 RF 避免自透视
RF1 的 Event 属性默认为 AfterRenderingOpaques ，Event 属性定义 Unity 从 Render Object RF 注入渲染过程的注入点。在该 RF1 进行渲染之前已经进行了不透明物体的渲染，并将深度值写入了深度缓冲区。执行 RF1 时，Unity 使用“深度测试”属性中指定的条件执行深度测试。

1. Universal Renderer 的 Filtering > Opaque Layer Mask，清除 Character 层旁边的复选标记。![[Pasted image 20230702110513.png|500]]
2. 现在 Unity 不会渲染角色，除非它在游戏对象后面。（因为 RF1 相当于加了一次渲染，虽然渲染器设置的不对该层物体渲染，但是当 RF1 的 Event 触发后，就增加了一次对该层物体的渲染。由于深度测试是 Greater，Zbuffer 默认是无限大，所以在遮挡物体意外是无法通过深度测试的，所以不显示。只有在遮挡物体后面才能通过，显示为 Blue） ![[Pasted image 20230702110612.png|160]]
3. 添加 RF2（虽然都是默认的 Event 条件，但 RF2 执行顺序在 RF1 之后），LayerMask 选择 Character 层，可以发现，都被渲染为 red（RF1 选择了 Equal 作为深度测试条件，所以此时深度缓冲区都是较大值。新建的 RF2 默认是 Less Equal，渲染时与上次 RF1 渲染的重合部分深度相等，上次未显示部位深度小于无限大，所以物体通过深度测试，以本身的 Red 颜色显示出来）![[Pasted image 20230702113031.png]]
4. 关闭 RF1 的深度写入，那么当 RF2 渲染时，遮挡物后面由于角色深度小于遮挡物，不通过测试，所以不会覆盖 RF1 绘制的 Blue。遮挡物外面通过测试，显示为 Red。这样就完成了！![[character-goes-behind-object 1.gif]]

# Full Screen RF
![[Pasted image 20230702134053.png|450]]
**Full Screen PF 允许在预定义的注入点（injection point）注入全屏渲染 Pass，以创建全屏效果。** 

- **Pass Material**：影响全屏 pass 的材质，必须为 Fullscreen Shader Graph 创建的材质。
![[4 后处理#自定义后处理]]
- **Injection Point 注入点：**
    1. **Before Rendering Transparents**：渲染透明体之前，在 skybox pass 之后和 transparents pass 之前添加效果。
    2. **Before Rendering Post Processing**: 渲染后处理前：在 Transparent Pass 之后和 post-processing pass 之前添加效果。
    3. **After Rendering Post Processing**：渲染后处理后：在 post-processing pass 之后和 AfterRendering pass 之前添加效果。

- **Requirements 要求：** 选择以下一个或多个 Pass 以供 RF 使用：  
    - **Depth**：添加 depth prepass（深度预处理） 以启用深度值的使用。 
    - **Normal**: 启用法线矢量数据的使用。
    - **Color**: 将屏幕的颜色数据复制到着色器内部的 `_BlitTexture` 纹理。
    - **Motion**: 启用运动矢量的使用 


# Decal RF
![[Pasted image 20230702102255.png]]

**创建步骤：**
1. URP Renderer 添加 Decal RF
2. 创建一个材质，并为其指定 `Shader Graphs/Decal` 着色器。添加贴图 ![[Pasted image 20230702100652.png|350]]
3. 右键创建贴画投影仪对象 Rendering->Decal Projector
4. 另一种 Decal 的方式：直接创建一个 Quad 当做贴花

**RF设置：**
![[Pasted image 20230702101203.png]]
- **Technique：**
    - **自动（Automatic）**：Unity 根据构建平台自动选择渲染技术。
    - **贴花缓冲区（DBuffer）**：Unity 将贴花渲染到**贴花缓冲区（DBuffer）** 中。在不透明渲染期间，Unity 将 DBuffer 的内容覆盖在不透明对象的顶部。
        - **Surface Data：**
            - Albedo：贴花会影响 BaseColor 和自发光颜色
            - Albedo Normal：贴花会影响基础颜色、自发光颜色和法线。
            - Albedo Normal MAOS：贴花会影响基础颜色、自发光颜色、法线、金属度值、光滑度值和环境光遮蔽值。
        - **限制：**
            - 该技术需要DepthNormal预处理，这使得该技术在实现基于瓦片的渲染的GPU上效率较低。
            - 此技术不适用于粒子和地形细节
    - **屏幕空间（Screen Space）**：使用 Unity 从深度纹理重建的法线**在不透明对象之后渲染贴花**。Unity 将贴花渲染为不透明网格顶部的网格。此技术仅支持法线混合。
        - Normal Blend：
            - Low：Unity在重建法线时获取一个深度样本。
            - Medium：获取三个深度样本
            - High：获取五个深度样本

**注意：**
1. 贴花投影在透明曲面上不起作用。
2. 不支持 SRP Batcher，因为它们使用了 Material property blocks。为了减少绘制调用的数量，可以使用 GPU Instancing 将贴花批处理在一起。如果场景中的贴花使用相同的“材质”，并且“材质”已启用“启用 GPU 实例化”属性，则 Unity 会实例化材质并减少绘制调用的次数。
3. 若要减少贴花所需的材质数量，请将多个贴花纹理放入一个纹理（图集）中。使用贴花投影仪上的 UV 偏移特性来确定要显示图集的哪个部分。


# 自定义 URP
## beginCameraRendering 事件 
Unity 在每帧中渲染每个激活的 Camera 之前引发一个 `beginCameraRendering` 事件。
>如果相机处于失活状态（去掉勾），Unity 不会为此相机引发 `beginCameraRendering` 事件。

订阅此事件的方法时，可以在 Unity 渲染 Camera 之前执行自定义逻辑（比如将额外的 Camera 渲染为 RT，以及将这些纹理用于平面反射或监视摄影机视图等效果。）
[RenderPipelineManager](https://docs.unity3d.com/ScriptReference/Rendering.RenderPipelineManager.html) 类中的其他事件提供了更多自定义 URP 的方法。

**如何为 `beginCameraRendering` 事件订阅方法？**
将下面的脚本拖放到一个 gameobject 即可使用：
```cs
public class URPCallbackExample : MonoBehaviour
{
    //依附的GameObject对象每次激活时调用（打勾）
    private void OnEnable()
    {
        //订阅方法
        //添加 WriteLogMessage 作为 RenderPipelineManager.beginCameraRendering 事件的委托
        RenderPipelineManager.beginCameraRendering += WriteLogMessage;
    }

    //依附的GameObject对象每次失活时调用（去掉勾）
    private void OnDisable()
    {
        //移除 WriteLogMessage 作为 RenderPipelineManager.beginCameraRendering 事件的委托
        RenderPipelineManager.beginCameraRendering -= WriteLogMessage;
    }
    
    // 当此方法是 RenderPipeline.beginCameraRendering 事件的委托时，Unity 每次引发 beginCameraRendering 事件时都会调用此方法
    void WriteLogMessage(ScriptableRenderContext context, Camera camera)
    {
        Debug.Log($"Beginning rendering the camera: {camera.name}");
    }
}
```

## 自定义 Render Feature
创建可编程的 RF，并实现用于配置 `ScriptableRenderPass` 实例并将其注入可编程渲染器的方法。

> [!NOTE] 创建 RenderFeature 脚本更简单的方法
> 右键->Rendering->URP Render Feature

- @ **`CustomRenderFeature` 自定义 RF**
1. **`Create` ：Unity 对以下事件调用此方法：**
    - 首次加载 RF 时
    - 在 RF 的 Inspector 中更改属性时
    - 启用或禁用 RF 时
2. **`AddRenderPasses` ：Unity 每台相机每帧调用一次此方法**。使用此方法可以将 `ScriptableRenderPass` 实例注入到可编程的渲染器中。

- @ **`CustomRenderPass` 自定义 Render Pass**
    1. **`OnCameraSetup`：在执行 render pass 之前被调用。** 它可用于配置 Render Target 和它们的 Clear State，还可以创建临时渲染目标纹理。当为空时，该 Render Pass 将渲染到活动相机的 Render Target。（不要调用 CommandBuffer.SetRenderTarget. 而应该是 `ConfigureTarget` 和 `ConfigureClear`）s
    2. **`Execute`：每帧执行，在这里实现渲染逻辑。** 使用 ` ScriptableRenderContext` 发出绘制命令或执行命令缓冲区。不必调用 submit 指令，渲染管线将在管线中的特定点调用它。
        1. `ProfilingSampler`：CPU 和 GPU 分析采样器的包装器。将此与 `ProfileScope` 一起使用可以评测一段代码。标记 Profiling 后，可在 FrameDebugger 中直接查看标记 Profiling 的对象
        2. `DrawRenderer`：批量绘制对象
    3. **`OnCameraCleanup`** ：清理在此 render pass 执行期间创建的所有已分配资源。

```cs file:RF模板
public class CustomRenderFeature : ScriptableRendererFeature
{
    class CustomRenderPass : ScriptableRenderPass
    {
        // 在执行 render pass 之前被调用。
        public override void OnCameraSetup(CommandBuffer cmd, ref RenderingData renderingData)
        {
        }

        // 每帧执行，这里可以实现渲染逻辑
        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
        {
            //CPU和GPU分析采样器的包装器。将此与ProfileScope一起使用可以评测一段代码。
            //标记Profiling后，可在FrameDebugger中直接查看标记Profiling的对象
            ProfilingSampler mProfilingSampler = new ProfilingSampler("Test1");
            //获取新的命令缓冲区并为其指定一个名称
            
            CommandBuffer cmd = CommandBufferPool.Get("Test1 Cmd");
            
            //ProfilingScope
            using (new ProfilingScope(cmd, mProfilingSampler))
            {
                
                //执行命令缓冲区中的命令
                context.ExecuteCommandBuffer(cmd);
                
                //释放命令缓冲区
                CommandBufferPool.Release(cmd);
            }
        }
        // 清理在此render pass执行期间创建的所有已分配资源。
        public override void OnCameraCleanup(CommandBuffer cmd)
        {
        }
    }

    CustomRenderPass m_ScriptablePass;

    /// <inheritdoc/>
    public override void Create()
    {
        //创建CustomRenderPass实例
        m_ScriptablePass = new CustomRenderPass();

        // 配置render pass插入的位置
        m_ScriptablePass.renderPassEvent = RenderPassEvent.AfterRenderingOpaques;
    }

    // 这里你可以在渲染器中插入一个或多个render pass
    // 在每个摄像机设置一次渲染器时调用此方法。
    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        //入队渲染队列
        renderer.EnqueuePass(m_ScriptablePass);
    }
}
```

![[Pasted image 20230704154156.png]]
打开了管线设置中的 Debug Level 后；可以通过这个参数看见更多的调试信息
![[Pasted image 20230704154208.jpg]]

如 UniversalRenderPipeline.cs 的 `RenderSingleCamera` 方法里：
```c
static void RenderSingleCamera(...)  
{  
   ...  
   asset.debugLevel >= PipelineDebugLevel.Profiling ? ...

```
这段代码在勾选这个设置后可以在FrameDebugger内显示不同的相机名。


**手动创建过程：**
1. 创建脚本，命名为 CustomRenderFeature. cs
2. `using UnityEngine.Rendering.Universal;` 继承 `ScriptableRendererFeature` 类
3. 该脚本类必须实现以下方法：
    -  **`Create` ：Unity 对以下事件调用此方法：**
        - 首次加载 RF 时
        - 启用或禁用 RF 时
        - 在 RF 的 Inspector 中更改属性时
    - **`AddRenderPasses` ：Unity 每台相机每帧调用一次此方法**。使用此方法可以将 `ScriptableRenderPass` 实例注入到可编程的渲染器中。
4. 将创建的 RF 添加到 URP Asset 中。可以看到 Add  Render Feature 多了一个选项 ![[Pasted image 20230702151355.png|250]]

### 可编程 Render Pass

**创建可编程 Render Pass，并将其实例入队（enqueue）渲染队列**

1. 在 `CustomRenderFeature` 类中，声明 `CustomRenderPass` 类并继承 `ScriptableRenderPass` 类
2. 必须实现 `Execute` 方法：Unity 每帧运行 Execute 方法。使用此方法，可以实现自定义渲染功能。我们**在 `Execute` 方法中实现渲染命令**
3. 在 `CustomRenderFeature` 类中，声明一个私有的 `CustomRenderPass` 字段
4. 在 `Create` 方法中，实例化 `_customRenderPass` 对象
5. 在 `AddRenderPasses` 方法中，使用 `renderer. EnqueuePass` 方法将 `_customRenderPass` 放入渲染队列。现在 `CustomRenderFeature` 正在 `CustomRenderPass` 中执行 `Execute` 方法
6. 在 `Execute` 方法中创建 `CommandBuffer` 类型的对象，此对象包含要执行的渲染命令的列表。
```cs file:CustomRenderFeature. cs
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;
//自定义Render Feature
public class CustomRenderFeature : ScriptableRendererFeature
{
    //自定义Render Pass
    private class CustomRenderPass : ScriptableRenderPass
    {
        //每帧执行
        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
        {
            //获取新的命令缓冲区并为其指定一个名称
            CommandBuffer cmd = CommandBufferPool.Get(name: "CustomRenderPass");
            
            //执行命令缓冲区中的命令
            context.ExecuteCommandBuffer(cmd);
            
            //释放命令缓冲区
            CommandBufferPool.Release(cmd);
        }
    }

    private CustomRenderPass _customRenderPass;
    
    public override void Create()
    {
        //创建CustomRenderPass实例
        _customRenderPass = new CustomRenderPass();
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        //入队
        renderer.EnqueuePass(_customRenderPass);
    }
}
```

### 案例
本例中 RF 将镜头光斑绘制为一个 Quad 上的纹理，这里我将两个类分开了。

```cs
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class CustomRenderFeature : ScriptableRendererFeature
{
    private CustomRenderPass _customRenderPass;
    public Material material;
    public Mesh mesh;
    
    public override void Create()
    {
        _customRenderPass = new CustomRenderPass(material, mesh);
        
        //更改渲染顺序，在渲染天空盒之后渲染自定义渲染pass，这样天空盒就不会覆盖渲染的光斑了
        _customRenderPass.renderPassEvent = RenderPassEvent.AfterRenderingSkybox;
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        if(material!=null && mesh!=null)
        {
            renderer.EnqueuePass(_customRenderPass);
        }
    }
}

```


```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class CustomRenderPass : ScriptableRenderPass
{
    private Material _material;
    private Mesh _mesh;

    public CustomRenderPass(Material material, Mesh mesh)
    {
        _material = material;
        _mesh = mesh;
    }
        
    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        //获取新的命令缓冲区并为其指定一个名称
        CommandBuffer cmd = CommandBufferPool.Get(name: "CustomRenderPass");
        
        //获取相机
        Camera camera = renderingData.cameraData.camera;
        //设置投影矩阵，以便 Unity 在屏幕空间中绘制四边形
        cmd.SetViewProjectionMatrices(Matrix4x4.identity, Matrix4x4.identity);
        //比例变量，使用摄像机纵横比作为 y 坐标
        Vector3 scale = new Vector3(1, camera.aspect, 1);
        //在Light的屏幕空间位置为每个Light绘制一个四边形。
        foreach (VisibleLight visibleLight in renderingData.lightData.visibleLights)
        {
            Light light = visibleLight.light;
            
            //将每个光源的位置从世界转换为viewport空间
            Vector3 position = camera.WorldToViewportPoint(light.transform.position) * 2 - Vector3.one;
            //将quad的 z 坐标设置为 0，以便 Uniy 将它们绘制在同一平面上。
            position.z = 0; 
            
            //绘制quad
            cmd.DrawMesh(_mesh, Matrix4x4.TRS(position,Quaternion.identity, scale), _material, 0, 0);
        }
        
        //执行命令缓冲区中的命令
        context.ExecuteCommandBuffer(cmd);
            
        //释放命令缓冲区
        CommandBufferPool.Release(cmd);
    }
}
```

# RTHandle 系统
Render Target 管理是任何渲染管道的重要组成部分。在复杂的渲染管道中，有许多相互依赖的 Rendr Pass 使用许多不同的 RT，因此重要的是要有一个可维护和可扩展的系统，以便轻松管理内存。 
最大的问题之一是当渲染管道使用许多不同的摄影机，每个摄影机都有自己的分辨率时。例如，离屏摄像头或实时反射探针。在这种情况下，如果系统为每个摄影机独立分配 RT，则内存总量将增加到无法管理的级别。
这对于使用许多中间 RT 的复杂渲染管道来说尤其糟糕。Unity 可以使用临时临时渲染纹理 [temporary render textures](https://docs.unity3d.com/ScriptReference/RenderTexture.GetTemporary.html)，但不幸的是，它们不适合这种情况，因为只有当新渲染纹理使用完全相同的属性和分辨率时，临时渲染纹理才能重用内存。这意味着，当使用两种不同的分辨率进行渲染时，Unity 使用的内存总量是所有分辨率的总和。
为了解决渲染纹理内存分配的这些问题，Unity 的 SRP 包含了 RTHandle 系统。
## RTHandle 基本原理
RTHandle 系统是 Unity 的 [RenderTexture](https://docs.unity3d.com/ScriptReference/RenderTexture.html) API 之上的一个抽象层，可以自动 RT 管理。可以可以在使用各种分辨率的摄影机之间重用 RT。
 RTHandle 系统工作基础：
1. 您不再为自己分配具有固定分辨率的 RT。相反，您可以使用与给定分辨率下的全屏相关的比例来声明渲染纹理。RTHandle 系统仅为整个渲染管道分配一次纹理，以便可以将其重新用于不同的摄影机。
2. 现在有了 **reference size** （参考尺寸）的概念。这是应用程序用于渲染的分辨率。**您有责任在渲染管道以特定分辨率渲染每个摄影机之前声明它**。有关如何执行此操作的信息，请参阅 [Updating the RTHandle system](https://docs.unity3d.com/Packages/com.unity.render-pipelines.core@14.0/manual/rthandle-system-fundamentals.html#updating-the-rthandle-system)。
3. 在内部，RTHandle 系统跟踪您声明的最大 reference size。它将其用作渲染纹理的实际大小。最大 reference size 是最大大小。
4. 每次声明新的引用大小用于渲染，RTHandle 系统都会检查它是否大于当前记录的最大引用大小。如果是，RTHandle 系统会在内部重新分配所有渲染纹理以适应新的大小，并用新的大小替换最大的引用大小。