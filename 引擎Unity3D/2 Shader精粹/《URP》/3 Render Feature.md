1.  介绍 [URP Renderer Feature | Universal RP | 14.0.8 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/urp-renderer-feature.html)
2.  使用 [Example: How to create a custom rendering effect using the Render Objects Renderer Feature | Universal RP | 14.0.8 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/containers/how-to-custom-effect-render-objects.html)

Render Feature 是一种 Asset，用于向 URP 渲染器添加额外的渲染过程并配置其行为。

以下 Render Feature在 URP 中可用：

- [Render Objects 渲染对象](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-features/renderer-feature-render-objects.html)
- [Screen Space Ambient Occlusion  屏幕空间环境光遮蔽](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/post-processing-ssao.html)
- [Decal 贴花](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-feature-decal.html)
- [Screen Space Shadows 屏幕空间阴影](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-feature-screen-space-shadows.html)
- [Full Screen Pass 全屏Pass](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-features/renderer-feature-full-screen-pass.html)


> [!NOTE] 简称
> 下文Renderer Feature 简称为 RF

# Render Objects
[Render Objects Renderer Feature | Universal RP | 14.0.8 --- 渲染对象渲染器功能](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-features/renderer-feature-render-objects.html)
![[Pasted image 20230702103753.png|500]]
URP 在 DrawOpaqueObjects 和 DrawTransparent Objects 过程中绘制对象。您可能需要在帧渲染的不同点绘制对象，或者以其他方式解释和写入渲染数据（如 depth 和 stencil）。Render Objects RT 允许通过特定的重载（overides）在指定的图层、指定的时间来自定义 Draw Objects。

**实战**：当角色在GameObjects后面时，用不同的材质绘制角色轮廓。
![[character-goes-behind-object.gif]]

使用两个 Render Objects RF：一个用于绘制不被遮挡的颜色，另一个用于绘制被遮挡颜色
1. 创建一个 Layer，命名为 Character
2. 将要渲染的 object 分配给该层，然后将 RF 的 LayerMask 也设置为该层
3. 创建 Red 材质和 Blue 材质。Object 给与 Red 材质，然后Overides->Blue 材质
4. 我们要实现当角色位于其他游戏对象之后时，渲染器功能才会使用 Blue 材质渲染角色。可以通过深度测试来实现，Depth->DepthTest 设置为 Greater，这样在该 Layer 下深度大的物体绘制在前面。

当模型复杂时，这样设置可能会发生自透视：
![[character-depth-test-greater.gif|449]]

## 创建额外的 RF 避免自透视
RF 的 Event 属性默认为 AfterRenderingOpaques ，Event 属性定义 Unity 从 Render Object RF 注入渲染过程的注入点。在该 RF 进行渲染之前已经进行了不透明物体的渲染，并将深度值写入了深度缓冲区。执行 RF 时，Unity 使用“深度测试”属性中指定的条件执行深度测试。

1. Universal Renderer 的 Filtering > Opaque Layer Mask，清除 Character 层旁边的复选标记。![[Pasted image 20230702110513.png|500]]
2. 现在 Unity 不会渲染角色，除非它在游戏对象后面。 ![[Pasted image 20230702110612.png|160]]
3. 

# Decal
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


# 屏幕空间阴影
URP14 暂时无法使用
![[Pasted image 20230702102742.png]]

计算受主定向光影响的不透明对象的屏幕空间阴影，并在场景中绘制这些阴影。若要渲染屏幕空间阴影，URP 需要一个额外的 Render Target。这增加了应用程序所需的内存量，但如果项目使用前向渲染，屏幕空间阴影可以提高运行时资源强度。这是因为如果使用屏幕空间阴影，URP 不需要多次访问级联阴影贴图。

