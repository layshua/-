

### Screen Blur 屏幕模糊

We can use a custom feature to apply blur operations to the screen. This can involve either blitting back to the screen, or to a custom buffer - which could then be sampled in UI shaders for example to make them appear to blur what is behind them.  
我们可以使用自定义功能对屏幕应用模糊操作。这可能涉及到闪电式传输回屏幕，也可能涉及到自定义缓冲区——例如，可以在UI着色器中对其进行采样，使其看起来模糊了背后的内容。

There are multiple ways to blur in a shader, such as Box Blur, Gaussian, Kawase. If you use those as search terms you should be able to find example implementations in Unity shaders. These can be applied to the screen by using a [Blit](#blit) (may require shader to be modified depending on the blit method used).  
着色器中有多种模糊方式，例如Box blur、Gaussian、Kawase。如果使用这些作为搜索术语，您应该能够在Unity着色器中找到示例实现。这些可以通过使用Blit应用于屏幕（根据使用的Blit方法，可能需要修改着色器）。

Some examples (May be for older URP versions. Also check licenses before use) :  
一些示例（可能适用于较旧的URP版本。使用前也要检查许可证）：

*   [https://gist.github.com/Refsa/e006f1a8d3a974ae88cb7ecd93bf306b](https://gist.github.com/Refsa/e006f1a8d3a974ae88cb7ecd93bf306b)
*   [https://github.com/sebastianhein/urp_kawase_blur](https://github.com/sebastianhein/urp_kawase_blur)
*   [https://alexanderameye.github.io/notes/scriptable-render-passes/](https://alexanderameye.github.io/notes/scriptable-render-passes/)
*   [https://danielilett.com/2023-06-01-tut6-6-gaussian-blur/](https://danielilett.com/2023-06-01-tut6-6-gaussian-blur/)




### Internal URP Passes 内部URP通行证

Internally, URP also uses a bunch of Scriptable Render Passes, listed under the [URP package’s Runtime/Passes folder](https://github.com/Unity-Technologies/Graphics/tree/master/Packages/com.unity.render-pipelines.universal/Runtime/Passes). These can use internal functions which we can’t use, but might still be useful to look at to get an idea of how they work.  
在内部，URP还使用一组可脚本化Render Pass，这些过程列在URP包的Runtime/Pases文件夹下。这些pass可以使用我们不能使用的内部函数，但对于了解它们的工作原理可能仍然有用。

For example, here’s a few :  
例如，以下是一些：

*   [DepthOnlyPass](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/Passes/DepthOnlyPass.cs), [DepthNormalOnlyPass](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/Passes/DepthNormalOnlyPass.cs), or [CopyDepthPass](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/Passes/CopyDepthPass.cs)  
    DepthOnly通行证、DepthNormalOnly通行证或CopyDepthPass
    *   sets up the Camera Depth (and Normals) Textures (e.g. used by the **Scene Depth** node and SSAO feature)  
        设置“摄影机深度（和法线）纹理”（例如，“场景深度”节点和SSAO功能使用的纹理）
*   [CopyColorPass CopyColorPass](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/Passes/CopyColorPass.cs)
    *   sets up the Camera Opaque Texture (used by the **Scene Color** node)  
        设置“摄影机不透明纹理”（由“场景颜色”节点使用）
*   [MainLightShadowCasterPass](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/Passes/MainLightShadowCasterPass.cs), [AdditionalLightsShadowCasterPass](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/Passes/AdditionalLightsShadowCasterPass.cs)  
    主LightShadowCasterPass，附加LightShadowCaster Pass
    *   renders objects into shadowmaps  
        将对象渲染到阴影贴图中

URP also provides Renderer Features and Passes for :  
URP还为以下各项提供Renderer Feature和过程：

*   [RenderObjectsPass](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/Passes/RenderObjectsPass.cs) - used by the [Render Objects feature](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@15.0/manual/renderer-features/renderer-feature-render-objects.html)  
    RenderObjectsPass-由“渲染对象”功能使用
    *   (see section above for example usages)  
        （有关用法示例，请参阅上面的部分）
*   [ScreenSpaceAmbientOcclusionPass](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/Passes/ScreenSpaceAmbientOcclusionPass.cs) - used by the [SSAO feature](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@15.0/manual/post-processing-ssao.html)  
    ScreenSpaceAmbientOcclusionPass-SSAO功能使用
*   [ScreenSpaceShadowsPass](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/RendererFeatures/ScreenSpaceShadows.cs) - used by [Screen Space Shadows feature](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@15.0/manual/renderer-feature-screen-space-shadows.html).  
    屏幕空间阴影通行证-由屏幕空间阴影功能使用。
    *   This feature actually enqueues _two_ passes - ScreenSpaceShadowsPostPass as well, used to turn off the screen space shadows keyword for the transparent pass.  
        该功能实际上将两个过程排队——ScreenSpaceShadowsPostPass，用于关闭透明过程的screen-space shadows关键字。
*   (2022.2+) [FullscreenRenderPass](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/RendererFeatures/FullScreenPassRendererFeature.cs) - used by [Fullscreen Pass feature](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@15.0/manual/renderer-features/renderer-feature-full-screen-pass.html).  
    （2022.2+）FullscreenRenderPass-由FullscreenPass功能使用。
    *   Allows you to draw/blit material/shader effects (using the new **Fullscreen Shader Graph**) to the screen, as mentioned earlier.  
        允许您在屏幕上绘制/blit材质/着色器效果（使用新的Fullscreen shader Graph），如前所述。
*   [DecalRendererFeature](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/RendererFeatures/DecalRendererFeature.cs) - Also see [Decals feature docs page](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@15.0/manual/renderer-feature-decal.html).  
    贴花Renderer Feature-另请参见贴花功能文档页面。
    *   Uses various passes depending on the technique selected. Can find them under [Runtime/Decal folder](https://github.com/Unity-Technologies/Graphics/tree/master/Packages/com.unity.render-pipelines.universal/Runtime/Decal).  
        根据所选的技术使用不同的过程。可以在Runtime/Dateal文件夹下找到它们。

The github links here are for the “master” branch so may not be accurate to the version you’re using. For example, you may want to switch to “2022.2/staging” to view the code for that release. Be aware that for older versions, you need to remove the `Packages/` from the URL or you’ll see a 404.  
这里的github链接是针对“master”分支的，因此可能与您使用的版本不准确。例如，您可能希望切换到“2022.2/staring”以查看该版本的代码。请注意，对于旧版本，您需要从URL中删除 `Packages/` ，否则您将看到404。



### RenderPassEvent RenderPassEvent

它使用 `RenderPassEvent` 枚举，其中包含以下条目/值：

```c
BeforeRendering = 0
BeforeRenderingShadows = 50
AfterRenderingShadows = 100
BeforeRenderingPrePasses = 150
AfterRenderingPrePasses = 200
BeforeRenderingGbuffer = 210
AfterRenderingGbuffer = 220
BeforeRenderingDeferredLights = 230
AfterRenderingDeferredLights = 240
BeforeRenderingOpaques = 250
AfterRenderingOpaques = 300
BeforeRenderingSkybox = 350
AfterRenderingSkybox = 400
BeforeRenderingTransparents = 450
AfterRenderingTransparents = 500
BeforeRenderingPostProcessing = 550
AfterRenderingPostProcessing = 600
AfterRendering = 1000
```

请注意，在 `BeforeRenderingPrePasses` 事件（值为150）之前，不会设置摄影机矩阵和 stereo rendering。

通常，您将使用枚举本身设置字段：

```
m_ScriptablePass.renderPassEvent = RenderPassEvent.AfterRenderingOpaques;
// (aka value of 300)
```

For passes with the same renderPassEvent value, the order should be the same as it appears on the Renderer Features list (or order of EnqueuePass if enqueuing multiple passes in a single feature).  
对于具有相同renderPassEvent值的过程，顺序应与“Renderer Feature”列表上显示的顺序相同（如果在单个功能中对多个过程进行排队，则为EnqueuePass的顺序）。

如果需要指定在其他过程之间的特定点运行的过程，也可以提供值的偏移。例如，以下内容将在只有 `RenderPassEvent.BeforeRenderingPostProcessing` （值550）的任何传递之后运行：

```
m_ScriptablePass.renderPassEvent = RenderPassEvent.BeforeRenderingPostProcessing + 1;
// (aka value of 551)
```

只是不要使偏移量太高，BeforeRenderingPostProcessing在枚举中的值为550。因此，这样做 `BeforeRenderingPostProcessing + 50` 就相当于 `AfterRenderingPostProcessing` 。



## AddRenderPasses AddRenderPass


此方法负责将ScriptableRenderPass与URP的Renderer注入/排队。默认情况下，它已经有 `renderer.EnqueuePass(m_ScriptablePass)` 了，这可能就是我们在这里所需要的。但是，如果需要，可以将多个通行证排队。

**模板中的注释提到，每个相机调用一次该方法，但请注意，这也是每个帧/更新，因此避免在此处创建/实例化任何内容（可以使用Create方法）。**

Also note that by default it would enqueue the pass for _all_ cameras - including ones used by the Unity Editor. In order to avoid this, we can test the camera type and return before enququeing (or can check later during Execute to prevent that function running, if you prefer).  
**还要注意，默认情况下，它会将所有相机的通行证排队**，包括Unity Editor使用的相机。为了避免这种情况，我们可以在查询之前测试相机类型并返回（如果您愿意，也可以稍后在执行过程中检查以阻止该函数运行）。

```c
public bool showInSceneView;

public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData) {
    if (renderingData.cameraData.isPreviewCamera) return;
    // Ignore feature for editor/inspector previews & asset thumbnails
    if (renderingData.cameraData.isSceneViewCamera) return;
    // Ignore feature for scene view
    // If the feature uses camera targets, you may want to expose a bool/tickbox instead, e.g.
    if (!showInSceneView && renderingData.cameraData.isSceneViewCamera) return;

    // (could alternatively use "cameraData.cameraType == CameraType enum" for these)

    if (renderingData.cameraData.camera != Camera.main) return;
    // Ignore all cameras except the camera tagged as MainCamera
    // Though may be better to use Multiple Renderer Assets (see below)
    
    renderer.EnqueuePass(m_ScriptablePass);
}
```

As shown, we can also test against `Camera.main` if you only want the feature to run on the Main Camera. If you only want it to run on a different specific camera you could potentially set a Camera field at runtime (can’t during editor as assets can’t serialise scene references).  
如图所示，如果您只想在主摄像头上运行该功能，我们也可以针对 `Camera.main` 进行测试。如果只希望它在不同的特定摄影机上运行，则可以在运行时设置“摄影机”字段（**在编辑器期间不能设置，因为资源无法序列化场景引用**）。  
Though a better way to handle these would be to create multiple “Renderer” assets (e.g. Universal Renderers), assign them to the list on the **URP Asset(s)** (may have multiple per quality setting), then use the **Renderer** dropdown on each **Camera** component to select which index it should use. That way, you can choose which renderer features are used on a per-camera basis (without it being hardcoded).  
虽然处理这些问题的更好方法是创建多个 URP Assets，将它们分配到 URP 资源上的列表中（可能有多个每个质量的设置），然后使用每个相机组件上的“渲染器（Renderer）”下拉列表来选择应该使用的索引。这样，您就可以选择在每个摄影机的基础上使用哪些Renderer Feature（而无需对其进行硬编码）。

In 2021.2+ you should avoid accessing camera targets here (e.g. `cameraColorTargetHandle` / `cameraDepthTargetHandle`, or older `cameraColorTarget` / `cameraDepthTarget` on the `ScriptableRenderer` param) as these may not have been allocated yet! We can either obtain those targets directly in the passes Execute method, or add the SetupRenderPasses method (see below)  
在2021.2+中，您应该避免访问此处的相机目标（例如，#4参数上的 `cameraColorTargetHandle` / `cameraDepthTargetHandle` 或旧的 `cameraColorTarget` / `cameraDepthTarget` ），因为这些目标可能尚未分配！我们可以直接在passesExecute方法中获得这些目标，也可以添加SetupRenderPass方法（见下文）

### ConfigureInput 配置输入

We can also call `ConfigureInput` in this method, which allows us to request URP to generate certain textures (via the `ScriptableRenderPassInput` enum) :  
我们也可以在这个方法中调用 `ConfigureInput` ，这允许我们请求URP生成某些纹理（通过 `ScriptableRenderPassInput` 枚举）：

*   `ScriptableRenderPassInput.Depth`
    *   Make URP generate `_CameraDepthTexture`, even if it’s not enabled on the URP Asset.  
        使URP生成 `_CameraDepthTexture` ，即使它在URP资产上未启用。
*   `ScriptableRenderPassInput.Normal`
    *   Make URP generate `_CameraNormalsTexture` 使URP生成 `_CameraNormalsTexture`
*   `ScriptableRenderPassInput.Color`
    *   Make URP generate `_CameraOpaqueTexture`, even if it’s not enabled on the URP Asset.  
        使URP生成 `_CameraOpaqueTexture` ，即使它在URP资产上未启用。
*   `ScriptableRenderPassInput.Motion` (2022.1+)
    *   Make URP generate `_MotionVectorTexture` and `_MotionVectorDepthTexture`  
        使URP生成 `_MotionVectorTexture` 和 `_MotionVectorDepthTexture`


这似乎只适用于通用渲染器。2D渲染器似乎忽略了它。

These textures are generated at usual events, so won’t necessarily be ready for the event the pass is enqueued at. `_CameraOpaqueTexture` would only be used in the BeforeRenderingTransparents event or later for example.  
这些纹理是在通常的事件中生成的，因此不一定为过程排队的事件做好准备。例如， `_CameraOpaqueTexture` 只会在 BeforeRenderingTransparents 事件或更早使用。  
However the depth texture will be generated using a DepthPrepass if using the BeforeRenderingOpaques event and CopyDepth when using AfterRenderingOpaques (assuming it isn’t already using a prepass for other reasons). Can always check the [Frame Debugger window](https://docs.unity3d.com/2022.2/Documentation/Manual/frame-debugger-window.html) to see the order of everything!  
**但是，如果使用BeforeRenderingOpaques事件，则深度纹理将使用DepthPrepass生成，如果使用AfterRenderingOpaques事件，将使用CopyDepth生成（假设由于其他原因尚未使用预处理）。可以随时查看“框架调试器”窗口以查看所有内容的顺序！**

If you need multiple of these inputs, don’t call `ConfigureInput` multiple times (that will just override the value). As the enum has the `[Flags]` attribute, you can use the `|` operator to combine them instead. Example below.  
如果您需要这些输入中的多个，请不要多次调用 `ConfigureInput` （这只会覆盖值）。由于枚举具有 `[Flags]` 属性，因此可以使用 `|` 运算符来组合它们。以下示例。

```
public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData) {

    // Tell URP to generate the Camera Depth Texture
    m_ScriptablePass.ConfigureInput(ScriptableRenderPassInput.Depth);

    // Tell URP to generate the Camera Normals and Depth Textures
    // m_ScriptablePass.ConfigureInput(ScriptableRenderPassInput.Normal | ScriptableRenderPassInput.Depth);

    renderer.EnqueuePass(m_ScriptablePass);
}
```



## OnCameraSetup OnCamera设置

This method is responsible for **configuring the render targets** that will be used. By default if you do nothing, URP will **already configure the pass to use the camera colour and depth targets** for you.  
此方法负责配置将要使用的渲染目标。默认情况下，如果您什么都不做，URP将已经为您配置过程以使用相机颜色和深度目标。

But in the cases that we want to specify our **own targets**, we can use one of the `ConfigureTarget` function overloads from the ScriptableRenderPass class.  
但是，在我们想要指定自己的目标的情况下，我们可以使用ScriptableRenderPass类中的 `ConfigureTarget` 函数重载之一。

### RTHandle RT手柄

RTHandles are the way to handle render targets in Unity 2022+. Typically we allocate one inside OnCameraSetup using `RenderingUtils.ReAllocateIfNeeded` :  
RTHandles是在Unity 2022+中处理渲染目标的方法。通常，我们使用 `RenderingUtils.ReAllocateIfNeeded` 在OnCameraSetup中分配一个：

```c
// To create a Color Target :
var colorDesc = renderingData.cameraData.cameraTargetDescriptor;
colorDesc.depthBufferBits = 0; // must set to 0 to specify a colour target
// to use a different format, set .colorFormat or .graphicsFormat
RenderingUtils.ReAllocateIfNeeded(ref colorTarget, colorDesc, 
    name: settings.colorDestinationID);

// To create a Depth Target :
var depthDesc = renderingData.cameraData.cameraTargetDescriptor;
depthDesc.depthBufferBits = 32; // should be default anyway
RenderingUtils.ReAllocateIfNeeded(ref depthTarget, depthDesc,
    name: settings.depthDestinationID);
```

There is also `RTHandles.Alloc` (various overloads, see [docs](https://docs.unity3d.com/Packages/com.unity.render-pipelines.core@14.0/api/UnityEngine.Rendering.RTHandles.html#methods)). This should only run once, so can do a null check :  
还有 `RTHandles.Alloc` （各种重载，请参阅文档）。这应该只运行一次，因此可以执行null检查：

```
if (colorTarget == null) {
    colorTarget = RTHandles.Alloc(Vector2.one, colorDesc,
        name: settings.colorDestinationID);
}
```

There’s a lot of parameters for some of these methods, but they have default values so we don’t need to specify all of them. If you want to override/set a specific parameter, can use `<param name>:<value>`, such as `filterMode:FilterMode.Bilinear`, `wrapMode:TextureWrapMode.Clamp`, etc.  
其中一些方法有很多参数，但它们有默认值，所以我们不需要指定所有参数。如果要覆盖/设置特定参数，可以使用 `<param name>:<value>` ，如 `filterMode:FilterMode.Bilinear` 、 `wrapMode:TextureWrapMode.Clamp` 等。

There are also Alloc overrides to create an RTHandle from a RenderTargetIdentifier, RenderTexture, or Texture object, so you can _technically_ still use those types with the new system.  
还有Alloc覆盖可以从RenderTargetIdentifier、RenderTexture或Texture对象创建RTHandle，因此从技术上讲，您仍然可以在新系统中使用这些类型。

RTHandles also need to be released when they are no longer needed (by calling `.Release();` on it). You may want to do this in a function called by the Dispose function of the feature. e.g.  
RTHandles在不再需要时也需要释放（通过调用它的 `.Release();` ）。您可能希望在功能的Dispose函数调用的函数中执行此操作。例如。

```cs
public class CustomRendererFeature : ScriptableRendererFeature {
    class CustomRenderPass : ScriptableRenderPass {
        ...
        public void ReleaseTargets() {
            colorTarget?.Release();
            depthTarget?.Release();
        }
    }
    ...
    protected override void Dispose(bool disposing) {
        m_ScriptablePass.ReleaseTargets();
    }
}
```

If you aren’t familiar, the `?` here is the null-propagation operator - which means if the object is null, the function won’t be called. This helps avoid potential NullPointerExceptions if the feature is disposed without the targets being set (e.g. feature is added to list but disabled).  
如果您不熟悉，这里的 `?` 是null传播运算符，这意味着如果对象为null，则不会调用该函数。这有助于避免潜在的NullPointerExceptions，如果在未设置目标的情况下处理功能（例如，将功能添加到列表中但禁用）。

In case it’s useful, the docs for [RTHandle System Fundamentals](https://docs.unity3d.com/Packages/com.unity.render-pipelines.core@15.0/manual/rthandle-system-fundamentals.html) and [Using the RTHandle system](https://docs.unity3d.com/Packages/com.unity.render-pipelines.core@15.0/manual/rthandle-system-using.html) may provide additional info.  
如果有用，RTHandle系统基础知识和使用RTHandle的文档可能会提供其他信息。

### ConfigureTarget ConfigureTarget

These take either one or two parameters, the first being the colour target (like the colours you see on the screen in Scene/Game view) and the second being an optional depth target (like the depth buffer, used for ZWrite & ZTest. Can also contain bits for Stencil values).  
它们采用一个或两个参数，第一个是颜色目标（如场景/游戏视图中屏幕上的颜色），第二个是可选的深度目标（如深度缓冲区，用于ZWrite和ZTest。也可以包含Stencil值的位）。  
These parameters need to be of the type `RTHandle` (or `RenderTargetIdentifier` but those functions are deprecated as of Unity 2022).  
这些参数的类型需要为 `RTHandle` （或 `RenderTargetIdentifier` ，但自Unity 2022以来，这些函数已被弃用）。

```c
private RTHandle colorTarget, depthTarget;

public override void OnCameraSetup(CommandBuffer cmd, ref RenderingData renderingData) {
    var colorDesc = renderingData.cameraData.cameraTargetDescriptor;
    colorDesc.depthBufferBits = 0; // must set to 0 to specify a colour target
    // to use a different format, set .colorFormat or .graphicsFormat
    
    if (settings.colorDestinationID != ""){
        RenderingUtils.ReAllocateIfNeeded(ref colorTarget, colorDesc, name: settings.colorDestinationID);
        // if you need to specify texture filter and wrap modes :
        // RenderingUtils.ReAllocateIfNeeded(ref colorTarget, colorDesc, FilterMode.Point, TextureWrapMode.Clamp, name: settings.colorDestinationID);
    }else{
        colorTarget = renderingData.cameraData.renderer.cameraColorTargetHandle;
    }

    var depthDesc = renderingData.cameraData.cameraTargetDescriptor;
    depthDesc.depthBufferBits = 32; // should be default anyway
    if (settings.depthDestinationID != ""){
        RenderingUtils.ReAllocateIfNeeded(ref depthTarget, depthDesc, name: settings.depthDestinationID);
    }else{
        depthTarget = renderingData.cameraData.renderer.cameraDepthTargetHandle;
    }
    
    //ConfigureTarget(colorTarget);
    // Later rendering commands will render into colorTarget
    // No depth target, so ZWrite, ZTest and Stencil operations will not do anything

    // OR 

    ConfigureTarget(colorTarget, depthTarget);
    // Later rendering commands will render into colorTarget
    // and ZWrite/ZTest/Stencil based on depthTarget

    ConfigureClear(ClearFlag.Color, Color.black);
    // Set all pixels in the target to black
}
```

The first param of `ConfigureTarget` can also be an _array_ of colour targets, which sets up what is known as Multi-Target Rendering (MRT) assuming the target platform supports it. That allows you to render objects into multiple buffers at the same time by having the fragment shader use `SV_Target0`, `SV_Target1`, `SV_Target2`, etc in the fragment shader output (rather than just `SV_Target`). An example of this is Deferred Rendering, which sets up multiple “gbuffer” targets and configures them using this.  
**`ConfigureTarget` 的第一个参数也可以是一组颜色目标，假设目标平台支持，它会设置所谓的多目标渲染（MRT）。这允许您通过让片段着色器在片段着色器输出中使用 `SV_Target0` 、 `SV_Target1` 、 `SV_Target2` 等（而不仅仅是 #4 ），同时将对象渲染到多个缓冲区中。** 延迟渲染就是一个例子，它设置了多个“gbuffer”目标，并使用它进行配置。

### ConfigureClear ConfigureClear

In the above example you might also notice `ConfigureClear` being used, which allows us to clear the render target.  
在上面的示例中，您可能还会注意到正在使用 `ConfigureClear` ，这使我们可以清除渲染目标。

By default, RTHandle targets are uninitalised and may contain data from the previous frame/camera. This behaviour isn’t typically wanted, so we can use this function to set the entire texture to a particular colour (e.g. black) before rendering that camera.  
默认情况下，RTHandle目标未初始化，并且可能包含来自前一帧/摄影机的数据。这种行为通常是不需要的，所以我们可以在渲染相机之前使用此函数将整个纹理设置为特定颜色（例如黑色）。  
It can also clear depth and stencil values - this is specified by the first parameter of type [ClearFlag](https://docs.unity3d.com/Packages/com.unity.render-pipelines.core@14.0/api/UnityEngine.Rendering.ClearFlag.html) (enum with `[Flags]`). `Stencil` was added in 2021.2. In older versions that than, `Depth` cleared both.  
它还可以清除深度和模具值-这是由ClearFlag类型的第一个参数指定的（带有 `[Flags]` 的枚举） `Stencil` 于2021.2年添加。在比的旧版本中，#2清除了两者。

The function can be used even without ConfigureTarget, which would apply the clear to the camera targets.  
即使没有ConfigureTarget，也可以使用该功能，ConfigureTarget会将清除应用于相机目标。  
Typically you wouldn’t clear the camera’s colour (URP kinda does this for us anyway with the Background Color / Skybox) but for some effects clearing the depth or stencil values at a specific event could probably be useful.  
通常情况下，你不会清除相机的颜色（无论如何，URP在使用背景色/天空框时都会为我们这样做），但对于某些效果，在特定事件中清除深度或模板值可能会很有用。



## DrawRenderers DrawRenderers

As the method name suggests, this allows us to draw renderers to the current render target (specified in OnCameraSetup by ConfigureTarget as previously discussed). Can see the function’s params/overloads in the Unity docs for [ScriptableRenderContext.DrawRenderers](https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.DrawRenderers.html)  
正如方法名称所示，这允许我们将渲染器绘制到当前渲染目标（如前所述，由ConfigureTarget在OnCameraSetup中指定）。可以在ScriptableRenderContent.DrawRenderers的Unity文档中查看函数的参数/重载

The **DrawingSettings** struct parameter allows us to configure how the renderers will be drawn. It’s typically created using the `CreateDrawingSettings` method of the ScriptableRenderPass class (which calls the same method in [RenderingUtils](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/RenderingUtils.cs)). After this, can set properties on it.  
DrawingSettings结构参数允许我们配置绘制渲染器的方式。它通常是使用ScriptableRenderPass类的 `CreateDrawingSettings` 方法创建的（该类在RenderingUtils中调用相同的方法）。之后，可以对其设置属性。

Can see which properties are available in the Unity docs for [DrawingSettings](https://docs.unity3d.com/ScriptReference/Rendering.DrawingSettings.html). For example :  
可以在Unity文档中查看哪些属性可用于DrawingSettings。例如：

*   A commonly used one is specifying an `overrideMaterial`. This will override the material completely, including previous property values.  
    一个常用的方法是指定 `overrideMaterial` 。这将完全覆盖材质，包括以前的特性值。
*   In 2022.2+ we can now specify an `overrideShader`, which is similar to the concept of [Replacement Shaders](https://docs.unity3d.com/Manual/SL-ShaderReplacement.html) in the Built-in RP. Existing property values won’t be overridden. However, note that it does not support SRPBatcher and BatchRendererGroups so will be more expensive.  
    在2022.2+中，我们现在可以指定 `overrideShader` ，这类似于内置RP中的替换着色器的概念。现有属性值不会被覆盖。但是，**请注意，它不支持SRPBatcher和BatchRendererGroups，因此成本会更高。**

Unity will automatically get renderers from the cullResults, but we can specify filters in the **FilteringSettings** struct parameter. For creating this, see the [FilteringSettings constructor](https://docs.unity3d.com/ScriptReference/Rendering.FilteringSettings-ctor.html). If you don’t want to filter anything, can use `FilteringSettings.defaultValue`.  
Unity将自动从cullResults中获取渲染器，但我们可以在FilteringSettings结构参数中指定过滤器。有关创建此项的信息，请参阅FilteringSettings构造函数。如果您不想过滤任何内容，可以使用 `FilteringSettings.defaultValue` 。

Below is an example of using DrawRenderers to render any **opaque** objects, filtered by a LayerMask and specifying an Override Material (& pass index). Settings is a serialised class in the feature, as set up in the [Create section](#create).  
以下是使用DrawRenderers渲染任何不透明对象的示例，这些对象由LayerMask过滤并指定“覆盖材质”（&pass索引）。Settings是功能中的一个串行类，如在Create部分中所设置的。

如果改为绘制透明对象，则需要使用 `RenderQueueRange.transparent` 和 `SortingCriteria.CommonTransparent` 。（RenderPassEvent可能至少设置为AfterRenderingSkybox）

```cs
// in pass
private Settings settings;
private FilteringSettings filteringSettings;
private List<ShaderTagId> shaderTagsList = new List<ShaderTagId>();
private ProfilingSampler _profilingSampler;

// (constructor)
public CustomRenderPass(Settings settings, string name) {
    this.settings = settings;
    _profilingSampler = new ProfilingSampler(name);
    filteringSettings = new FilteringSettings(RenderQueueRange.opaque, settings.layerMask);
    // Use URP's default shader tags
    shaderTagsList.Add(new ShaderTagId("SRPDefaultUnlit"));
    shaderTagsList.Add(new ShaderTagId("UniversalForward"));
    shaderTagsList.Add(new ShaderTagId("UniversalForwardOnly"));
}
...
public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData) {
    CommandBuffer cmd = CommandBufferPool.Get();
    using (new ProfilingScope(cmd, m_ProfilingSampler)) {
        context.ExecuteCommandBuffer(cmd);
        cmd.Clear();
        
        // Draw Renderers to current Render Target (set in OnCameraSetup)
        SortingCriteria sortingCriteria = renderingData.cameraData.defaultOpaqueSortFlags;
        DrawingSettings drawingSettings = CreateDrawingSettings(shaderTagsList, ref renderingData, sortingCriteria);
        if (settings.overrideMaterial != null) {
            drawingSettings.overrideMaterialPassIndex = settings.overrideMaterialPass;
            drawingSettings.overrideMaterial = settings.overrideMaterial;
        }
        context.DrawRenderers(renderingData.cullResults, ref drawingSettings, ref filteringSettings);
    }
    context.ExecuteCommandBuffer(cmd);
    cmd.Clear();
    CommandBufferPool.Release(cmd);
}
```

## Blit 布利特

A blit is used to copy pixels from a “source” texture to a “destination” target. To do this, it draws a fullscreen quad (or triangle). It optionally allows you to specify a material if you want to use a custom shader, otherwise it’ll use a built-in one (specifically [Hidden/Universal/CoreBlit](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Shaders/Utils/CoreBlit.shader)).  
blit用于将像素从“源”纹理复制到“目标”目标。为此，它绘制了一个全屏四边形（或三角形）。如果要使用自定义着色器，则可以选择指定材质，否则将使用内置着色器（特别是Hidden/Universal/CoreBlit）。

There are a number of ways to handle blits. For Unity 2022+ versions we should now use the [Blitter API](https://docs.unity3d.com/Packages/com.unity.render-pipelines.core@14.0/api/UnityEngine.Rendering.Blitter.html). Most commonly :  
有很多方法可以处理闪电战。对于Unity 2022+版本，我们现在应该使用Blitter API。最常见的情况是：

*   `Blitter.BlitCameraTexture`
    *   Draws a fullscreen triangle. A specific vertex shader is required. See [Blit.hlsl](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blit.hlsl) and example link below.  
        绘制全屏三角形。需要特定的顶点着色器。请参阅下面的Blit.hlsl和示例链接。
    *   Passes “source” into `_BlitTexture` reference.  
        将“源”传递到 `_BlitTexture` 引用中。
    *   Example : [Blit Renderer Feature / 2022.1+ branch](https://github.com/Cyanilux/URP_BlitRenderFeature/blob/2022.1/Blit.cs)  
        示例：BlitRenderer Feature/202.21+分支
    *   2022.2 also introduced a new **Fullscreen Shader Graph** which works with this. There is also a **Fullscreen Pass Renderer Feature**, so if you just need to apply a shader to the camera you can use that instead of a custom feature.  
        2022.2还引入了一种新的全屏着色器图，它可以与此配合使用。还有一个全屏过程Renderer Feature，因此如果您只需要将着色器应用于相机，则可以使用该功能而不是自定义功能。

For older versions, these methods are available :  
对于旧版本，可以使用以下方法：

*   [`CommandBuffer.Blit`](https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.Blit.html)
    *   Draws a fullscreen quad  
        绘制全屏四边形
    *   Is probably more documented / used in tutorials. But this does not work with XR (Single Pass Instanced at least), so should be avoided.  
        可能更多地记录/使用在教程中。但这不适用于XR（至少是单次实例化），因此应该避免。
    *   Passes “source” into `_MainTex` reference.  
        将“源”传递到 `_MainTex` 引用中。
    *   Example : [Blit Renderer Feature / master branch](https://github.com/Cyanilux/URP_BlitRenderFeature/blob/master/Blit.cs)  
        示例：BlitRenderer Feature/主分支
        *   Note, this example uses the `ScriptableRenderPass.Blit` function, but that then calls `CommandBuffer.Blit` when using `RenderTargetIdentifier` params.  
            注意，本例使用 `ScriptableRenderPass.Blit` 函数，但当使用 `RenderTargetIdentifier` 参数时，它会调用 `CommandBuffer.Blit` 。
*   [`CommandBuffer.DrawMesh`](https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.DrawMesh.html) with `RenderingUtils.fullscreenMesh` (quad)  
    `CommandBuffer.DrawMesh` 与 `RenderingUtils.fullscreenMesh` （四元）
    *   Can work with XR. Likely the recommended method for versions prior to 2022.  
        可以使用XR。可能是2022年之前版本的推荐方法。
    *   Would need to pass source into shader yourself. e.g. using `cmd.SetGlobalTexture(`_SomeReference`, source);`  
        需要自己将源传递到着色器中。例如使用 `cmd.SetGlobalTexture(` _SomeReference `, source);`
    *   Would require a specific vertex shader (output vertex positions as-is, rather than using `TransformObjectToHClip`), unless you also override the View/Projection matrices to identity, then switch back after.  
        将需要特定的顶点着色器（按原样输出顶点位置，而不是使用 `TransformObjectToHClip` ），除非您也将“视图/投影”矩阵覆盖为标识，然后再切换回来。
    *   Example : [Blit Renderer Feature / cmd-DrawMesh branch](https://github.com/Cyanilux/URP_BlitRenderFeature/blob/cmd-drawMesh/Blit.cs)  
        示例：Blit Renderer Feature/cmd DrawMesh分支
*   `CoreUtils.DrawFullScreen`
    *   Draws a fullscreen triangle  
        绘制全屏三角形
    *   (Has various overloads, see the [CoreUtils class](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.core/Runtime/Utilities/CoreUtils.cs))  
        （具有各种重载，请参阅CoreUtils类）
    *   Similar to above but uses [`CommandBuffer.DrawProcedural`](https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.DrawProcedural.html). Would require a specific vertex shader (e.g. see [Blit.hlsl](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blit.hlsl)) to make sure it actually is fullscreen rather than rendering it in the world.  
        与上面类似，但使用 `CommandBuffer.DrawProcedural` 。需要一个特定的顶点着色器（例如，请参见Blit.hlsl）来确保它实际上是全屏的，而不是在世界上渲染它。

### Example 实例

When rendering we need to make sure we do not _read_ and _write_ to the same texture/target as this can cause “unintended behaviour” (to quote the `CommandBuffer.Blit` documentation). Because of this, if the source/destination needs to be the same, we actually need to instead use **two blits** with an additional target in-between.  
**渲染时，我们需要确保不会对同一纹理/目标进行读取和写入，因为这可能会导致“意外行为”（引用 `CommandBuffer.Blit` 文档）。因此，如果源/目的地需要相同，我们实际上需要使用两个blit，中间有一个额外的目标。**

In older versions this would typically be a “Temporary Render Texture” but with the change to RTHandles that’s not really used anymore. But I’ve still named it temp, and by using `RenderingUtils.ReAllocateIfNeeded`, the texture won’t be set up multiple times if the feature is used multiple times (or if other features use the same `_TemporaryColorTexture` reference) - so in a way, it still acts as a temporary texture, kinda.  
在旧版本中，这通常是一个“临时渲染纹理”，但随着对RTHandles的更改，它不再真正使用。但我仍然将其命名为temp，通过使用 `RenderingUtils.ReAllocateIfNeeded` ，如果该功能被多次使用（或者如果其他功能使用相同的 `_TemporaryColorTexture` 引用），则纹理不会被多次设置，因此在某种程度上，它仍然充当临时纹理。

```cs
// In pass :
private RTHandle rtTemp;
...
// OnCameraSetup
RenderingUtils.ReAllocateIfNeeded(ref rtTemp, desc, name: "_TemporaryColorTexture");
...
// Execute (inside using statement)
RTHandle rtCamera = renderer.cameraColorTargetHandle;
Blitter.BlitCameraTexture(cmd, rtCamera, rtTemp, settings.blitMaterial, settings.blitMaterialPassIndex);
Blitter.BlitCameraTexture(cmd, rtTemp, rtCamera, Vector2.one);
...
// Should also clean-up our allocated RTHandle, so :
public void ReleaseTargets() {
    temp?.Release();
}
...
// In feature :
protected override void Dispose(bool disposing) {
    blitPass.ReleaseTargets();
}
```

Of course some effects may require multiple passes/blits anyway, such as two-pass blurs. This would be the same as the above, but we’d specify the material in both with different pass indices :  
当然，有些效果可能需要多次传球/闪电战，比如两次传球模糊。这与上面的相同，但我们会指定具有不同通过指数的两种材料：

```
Blitter.BlitCameraTexture(cmd, rtCamera, rtTemp, settings.blurMaterial, 0);
Blitter.BlitCameraTexture(cmd, rtTemp, rtCamera, settings.blurMaterial, 1);
```




## RenderPipelineManager 渲染管线管理器
**在不需要 Renderer Feature. 的情况下运行 ScriptableRenderPass**

虽然不一定是本文的范围，但我们也可以使用RenderPipelineManager类中的事件在渲染每个帧/相机之前或之后注入代码。

I wanted to mention this as it is possible to enqueue Scriptable Render Passes to the renderer here if you don’t want to have to create/assign the Renderer Feature. This is also useful for older versions of URP where the 2D Renderer did not support features.  
我想提到这一点，因为如果您不想创建/指定 Renderer Feature，可以在此处将可脚本Render Pass排队到渲染器。这对于2D 渲染器不支持功能的旧版本 URP 也很有用。

Here’s an example of this. Note that it still uses the Settings and CustomRenderPass classes nested inside a CustomRendererFeature, but they could also be separate - the feature isn’t used! The feature used here is the same as the [Full Code Example](#full-example) from earlier.  
这是一个例子。请注意，它仍然使用嵌套在CustomRendererFeature中的Settings和CustomRenderPass类，但它们也可以是单独的-没有使用该功能！这里使用的功能与前面的完整代码示例相同。

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class SomeScript : MonoBehaviour {

    public CustomRendererFeature.Settings settings;
    private CustomRendererFeature.CustomRenderPass m_ScriptablePass;
    // or just CustomRenderPass if not nested
    // if nested, make sure CustomRenderPass is marked as public

    private void OnEnable(){
        // Setup same way as in CustomRendererFeature.Create
        m_ScriptablePass = new CustomRendererFeature.CustomRenderPass(settings, "Example");
        m_ScriptablePass.renderPassEvent = settings.renderPassEvent;

        // Register method
        RenderPipelineManager.beginCameraRendering += BeginCameraRendering;
    }
    
    private void OnDisable(){
        // same as CustomRendererFeature.Dispose
        m_ScriptablePass.ReleaseTargets();
        m_ScriptablePass = null;

        // Unregister method
        RenderPipelineManager.beginCameraRendering -= BeginCameraRendering;
    }

    // if fields are changed in editor, update pass by disposing & recreate
    private void OnValidate(){
        if (m_ScriptablePass != null) {
            OnDisable();
            OnEnable();
        }
    }

    private void BeginCameraRendering(ScriptableRenderContext context, Camera camera) {
        // Similar to CustomRendererFeature.AddRenderPasses
        CameraType cameraType = camera.cameraType;
        if (cameraType == CameraType.Preview) return; // Ignore feature for editor/inspector previews & asset thumbnails
        if (!settings.showInSceneView && cameraType == CameraType.SceneView) return;

        ScriptableRenderer renderer = camera.GetUniversalAdditionalCameraData().scriptableRenderer;
        renderer.EnqueuePass(m_ScriptablePass);
    }
}
```

Note that while the pass is enqueued at the beginning of the camera render, the blit will still occur later as set by the RenderPassEvent.  
请注意，虽然过程在摄影机渲染开始时排队，但blit仍将在稍后发生，如RenderPassEvent所设置的那样。

