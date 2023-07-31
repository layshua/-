

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

### Screenspace Distortion 屏幕空间失真

另一个类似的例子是屏幕空间失真效果。

场景中着色器中的失真通常使用“**Scene Color**”节点，但这意味着看不到其后面的透明对象。**一个潜在的解决方案是在渲染透明体后复制屏幕，后面的例子对此进行了介绍。**

In that case, the regular transparent queue can be distorted at least - but multiple layers of distortion still won’t stack.  
在这种情况下，常规透明队列至少可以失真，但**多层失真仍然不会叠加。**

For that we could instead have a feature render the distortion directions/strengths additively into a custom buffer (via a DrawRenderers call). We can then do a fullscreen blit on the camera targets, while sampling the buffer as a global texture to distort UV coords.  
**为此，我们可以让一个功能将失真方向/强度添加到自定义缓冲区中（通过DrawRenderers调用）。然后，我们可以对相机目标进行全屏闪电战，同时将缓冲区采样为全局纹理以扭曲UV坐标。**

### Lens Flares 镜头光斑特效

While URP now supports the [Lens Flares (SRP) component](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@13.1/manual/shared/lens-flare/lens-flare-component.html), older versions of the URP docs also provided a custom [Lens Flare Renderer Feature example](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@12.1/manual/containers/create-custom-renderer-feature-1.html) which could be a good reference if you need to do someting similar with lights.  
虽然URP现在支持Lens Flares（SRP）组件，但旧版本的URP文档还提供了一个自定义的Lens Flare Renderer Feature示例，如果您需要对灯光进行类似的操作，这可能是一个很好的参考。

Specifically, their example makes use of `renderingData.lightData.visibleLights` to loop through those lights, then drawing a quad mesh with a flare texture through `cmd.DrawMesh`.  
具体来说，他们的示例使用 `renderingData.lightData.visibleLights` 循环通过这些灯光，然后通过 `cmd.DrawMesh` 绘制具有光斑纹理的四元网格。

### Internal URP Passes 内部URP通行证

Internally, URP also uses a bunch of Scriptable Render Passes, listed under the [URP package’s Runtime/Passes folder](https://github.com/Unity-Technologies/Graphics/tree/master/Packages/com.unity.render-pipelines.universal/Runtime/Passes). These can use internal functions which we can’t use, but might still be useful to look at to get an idea of how they work.  
在内部，URP还使用一组可脚本化渲染过程，这些过程列在URP包的Runtime/Pases文件夹下。这些pass可以使用我们不能使用的内部函数，但对于了解它们的工作原理可能仍然有用。

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
URP还为以下各项提供渲染器功能和过程：

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
    贴花渲染器功能-另请参见贴花功能文档页面。
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
对于具有相同renderPassEvent值的过程，顺序应与“渲染器功能”列表上显示的顺序相同（如果在单个功能中对多个过程进行排队，则为EnqueuePass的顺序）。

如果需要指定在其他过程之间的特定点运行的过程，也可以提供值的偏移。例如，以下内容将在只有 `RenderPassEvent.BeforeRenderingPostProcessing` （值550）的任何传递之后运行：

```
m_ScriptablePass.renderPassEvent = RenderPassEvent.BeforeRenderingPostProcessing + 1;
// (aka value of 551)
```

只是不要使偏移量太高，BeforeRenderingPostProcessing在枚举中的值为550。因此，这样做 `BeforeRenderingPostProcessing + 50` 就相当于 `AfterRenderingPostProcessing` 。

## Dispose 处置

While not a part of the template, we can add a Dispose method to the feature. The method can be useful for releasing any resources that have been allocated. For example, material instances (see below) or RTHandles (see [RTHandle](#rthandle) section)  
虽然不是模板的一部分，但我们可以向该功能添加Dispose方法。该方法可用于释放已分配的任何资源。例如，材质实例（请参见下文）或RTHandles（请参见RTHandle部分）

In editor, the method is called when removing features, recompling scripts, entering/exiting play mode. (Not too sure when it gets called in builds, probably when changing scenes?)  
在编辑器中，当删除功能、重新编译脚本、进入/退出播放模式时，会调用该方法。（不太确定它何时在构建中被调用，可能是在更改场景时？）

```c
public Shader shader; // expose a Shader field

private Material material;

public override void Create() {
    // Create may be called multiple times... so :
    if (material == null || material.shader != shader){
        // only create material if null or different shader has been assigned

        if (material != null) CoreUtils.Destroy(material);
        // destroy material using previous shader
        
        material = CoreUtils.CreateEngineMaterial(shader);
        // or alternative method that uses the shader name (string):
        //material = CoreUtils.CreateEngineMaterial("Hidden/Internal-DepthNormalsTexture");
        // assumes the required shader is in the build (and variant, if keywords are set)
        // e.g. could add the shader to the "Always Included Shaders" in Project Settings -> Graphics
    }
    m_ScriptablePass = new CustomRenderPass(material, name);
    ...
}

protected override void Dispose(bool disposing) {
    CoreUtils.Destroy(material);
    // (will use DestroyImmediate() or Destroy() depending if we're in editor or not)
}
```

It’s very possible that Unity will automatically clean up some unused resources (e.g. during a Scene change), but it’s still a good practice to handle it ourselves.  
Unity很可能会自动清理一些未使用的资源（例如，在场景更改期间），但自己处理仍然是一个很好的做法。

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
虽然处理这些问题的更好方法是创建多个 URP Assets，将它们分配到 URP 资源上的列表中（可能有多个每个质量的设置），然后使用每个相机组件上的“渲染器（Renderer）”下拉列表来选择应该使用的索引。这样，您就可以选择在每个摄影机的基础上使用哪些渲染器功能（而无需对其进行硬编码）。

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

## OnCameraCleanup OnCamera清理

This method is called once for each camera (every frame) after rendering. You’d use this method to clean up some resources created during the other passes (that aren’t needed across multiple frames)! Some examples below.  
渲染后，对每个摄影机（每帧）调用一次此方法。您可以使用此方法清理在其他过程中创建的一些资源（在多个帧中不需要这些资源）！下面的一些例子。

If you used `CommandBuffer.GetTemporaryRT` somewhere, you’d typically use `CommandBuffer.ReleaseTemporaryRT` in this method.  
如果你在某个地方使用了 `CommandBuffer.GetTemporaryRT` ，你通常会在这个方法中使用 `CommandBuffer.ReleaseTemporaryRT` 。

With the change to RTHandles, you might think to release those in here too but I’ve found this causes glitchy rendering - especially in scene view. It’s better to instead release those in a method called by the feature’s Dispose method (see RTHandle section for an example).  
随着对RTHandles的更改，您可能会考虑在这里也发布这些，但我发现这会导致渲染出现问题，尤其是在场景视图中。相反，最好在功能的Dispose方法调用的方法中发布这些（有关示例，请参阅RTHandle部分）。  
I have seen some examples set private RTHandle fields to `null` in this method, but I don’t think that is required. (It may be to avoid accidently rendering to targets if the feature is called without using it’s Setup function?)  
我已经看到一些例子在这个方法中将私有RTHandle字段设置为 `null` ，但我认为这不是必需的。（如果在不使用设置功能的情况下调用该功能，可能是为了避免意外渲染到目标？）

If your feature relies on shader keywords, you might also enable those during Execute and disable them in OnCameraCleanup. This can either involve using `CommandBuffer.EnableShaderKeyword` and `CommandBuffer.DisableShaderKeyword` or the `CoreUtils.SetKeyword` function which provides a boolean which calls either of those for you. For example, the decal passes such as [DecalScreenSpaceRenderPass](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/Decal/ScreenSpace/DecalScreenSpaceRenderPass.cs) do this.  
如果您的功能依赖于着色器关键字，您也可以在执行期间启用这些关键字，并在OnCameraCleanup中禁用它们。这可以包括使用 `CommandBuffer.EnableShaderKeyword` 和 `CommandBuffer.DisableShaderKeyword` ，也可以包括提供布尔值的 `CoreUtils.SetKeyword` 函数，该函数为您调用这两个函数中的任何一个。例如，贴花过程（如DecadeScreenSpaceRenderPass）会执行此操作。

[DeferredLights](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/DeferredLights.cs) (used by URP’s Deferred Rendering path) appears to dispose some NativeArrays in here.  
DeferredLight（由URP的DeferredRendering路径使用）似乎在此处处置了一些NativeArrays。

## Execute 处决

The Execute function is where most of our custom rendering code goes. This is mostly handled through the [ScriptableRenderContext](https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.html) and [CommandBuffer](https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.html) APIs (or other functions that end up calling those APIs, like ScriptableRenderPass.Blit and the Blitter class which passes a CommandBuffer as a parameter).  
Execute函数是我们大多数自定义呈现代码的所在位置。这主要是通过ScriptableRenderContext和CommandBuffer API（或最终调用这些API的其他函数，如ScriptableRender Pass.Blit和将CommandBuffer作为参数传递的Blitter类）来处理的。

We do not need to call `ScriptableRenderContext.Submit` as URP handles this for us.  
我们不需要调用 `ScriptableRenderContext.Submit` ，因为URP为我们处理此问题。

To properly title things in the [Profiler](https://docs.unity3d.com/Manual/ProfilerWindow.html) & [Frame Debugger](https://docs.unity3d.com/2022.2/Documentation/Manual/frame-debugger-window.html) windows, the usual way to set up the Execute function is like this :  
要在Profiler和Frame Debugger窗口中正确命名，设置Execute函数的常用方法如下：

```cs
// (in Pass)
private ProfilingSampler m_ProfilingSampler;
...
// (constructor, method name should match class name)
public CustomRenderPass(string name) {
    m_ProfilingSampler = new ProfilingSampler(name);
}
...
public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData) {
    CommandBuffer cmd = CommandBufferPool.Get();
    using (new ProfilingScope(cmd, m_ProfilingSampler)) {
        context.ExecuteCommandBuffer(cmd);
        cmd.Clear();
        /*
        Note : should always ExecuteCommandBuffer at least once before using
        ScriptableRenderContext functions (e.g. DrawRenderers) even if you 
        don't queue any commands! This makes sure the frame debugger displays 
        everything under the correct title.
        */

        // Do stuff!
        // Would recommend keeping all your rendering code in this using statement.
        ...
    }
    // Execute Command Buffer one last time and release it
    // (otherwise we get weird recursive list in Frame Debugger)
    context.ExecuteCommandBuffer(cmd);
    cmd.Clear();
    CommandBufferPool.Release(cmd);
}
```

What rendering commands you do in here depends on what the feature is meant to do, but I’ll be providing some common examples below, such as DrawRenderers and Blit calls.  
您在这里执行的渲染命令取决于该功能的用途，但我将在下面提供一些常见的示例，例如DrawRenderers和Blit调用。

Before moving on, it’s very important to understand a few things :  
在继续之前，了解以下几点非常重要：

*   These rendering commands **do not happen straight away** - we are instead queueing up calls that URP will execute later.  
    **这些渲染命令并不是立即发生的，而是在排队等待 URP 稍后执行的调用。**
- 这意味着，例如，您不能编辑材质中的 properties，然后调用渲染命令并再次更改 properties——它将只使用您设置的最后一个值。但是，您可以通过 `CommandBuffer.SetGlobalX` 函数设置全局值（而不是通过 `Shader` 类！）
*   After using command buffer functions. We must then **use `context.ExecuteCommandBuffer(cmd)` to let the feature actually know about them**. If something isn’t working, the first thing is to check that you’re actually executing it!  
    使用 command buffer 函数 s 后。然后我们必须使用 `context.ExecuteCommandBuffer(cmd)` 让功能真正了解它们。如果某个东西不起作用，第一件事就是检查你是否真的在执行它！
*   **Order is important**. If you want command buffer functions to run _before_ a ScriptableRenderContext function (e.g. DrawRenderers), you must `ExecuteCommandBuffer` first. (And ideally `Clear()` it, so you don’t end up calling those commands twice if you execute again later!)  
    顺序很重要。如果希望命令缓冲区函数在 ScriptableRenderContext 函数（例如 DrawRenderers）之前运行，则必须先 `ExecuteCommandBuffer` 。（理想情况下是 `Clear()` ，这样，如果以后再次执行，就不会调用这些命令两次！）
    * 即使您没有专门调用CommandBuffer对象上的函数（例如 `cmd.Blit` ），也要关注需要将CommandBuffer作为参数传递到的任何函数，例如 `Blit(cmd, ...)` 。这意味着它仍将向其中添加命令！


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
        示例：Blit渲染器功能/202.21+分支
    *   2022.2 also introduced a new **Fullscreen Shader Graph** which works with this. There is also a **Fullscreen Pass Renderer Feature**, so if you just need to apply a shader to the camera you can use that instead of a custom feature.  
        2022.2还引入了一种新的全屏着色器图，它可以与此配合使用。还有一个全屏过程渲染器功能，因此如果您只需要将着色器应用于相机，则可以使用该功能而不是自定义功能。

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
        示例：Blit渲染器功能/主分支
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

### Example (Copy Color) 示例（复制颜色）

A blit could also be used to copy the camera colour target to a custom one (initialised similar to `rtTemp` above but renamed).  
blit也可以用于将相机颜色目标复制到自定义目标（初始化类似于上面的 `rtTemp` ，但已重命名）。

This would be similar to what the Opaque Texture does (used by **Scene Color** node), but that always occurs AfterRenderingSkybox so won’t contain transparent objects. With a custom blit feature, we could copy the screen during different events, such as AfterRenderingTransparents.  
这与“不透明纹理”（Opaque Texture）的作用类似（由“场景颜色”（Scene Color）节点使用），但它总是在渲染Skybox之后发生，因此不会包含透明对象。**通过自定义blit功能，我们可以在不同的事件期间复制屏幕，例如AfterRenderingTransparents。  
That way, the texture contains anything rendered in the normal transparent queue.  
这样，纹理包含在正常透明队列中渲染的任何内容。**

In this case you likely wouldn’t specify a material, and since the targets are different only a single call is needed :  
在这种情况下，您可能不会指定材质，并且由于目标不同，因此只需要调用一次：

```c
Blitter.BlitCameraTexture(cmd, rtCamera, rtCustom, Vector2.one);

// Pass as global shader texture
CommandBuffer.SetGlobalTexture("_SomeReference", rtCustom);
// In Shader Graphs could obtain this using Texture2D property,
// set same Reference, untick Exposed.
```

As shown we then pass our custom target (containing the camera copy) as a global texture. We can then sample that in shaders used by objects in the scene.  
如图所示，然后我们将自定义目标（包含摄影机副本）作为全局纹理传递。然后，我们可以在场景中对象使用的着色器中对其进行采样。

However to prevent graphical artifacts, it is important that any objects/shaders that sample the texture are rendered in a **later event**! Can force this by putting the object on a layer removed from the **Default Opaque/Transparent Layer Mask** at the top of the UniversalRenderer, and render that layer with a [DrawRenderers](#drawrenderers) call, or use the RenderObjects feature.  
但是，为了防止图形伪影，在以后的事件中渲染对纹理进行采样的任何对象/着色器都很重要！可以通过将对象放在从UniversalRenderer顶部的默认不透明/透明层遮罩中删除的层上，并使用DrawRenderers调用或使用RenderObjects功能渲染该层来强制执行此操作。

## Full Renderer Feature Example  
完整渲染器功能示例

Here’s a full code example based on snippets mentioned in this post. (And the usage foldout explains what it could be used for)  
下面是一个基于本文中提到的片段的完整代码示例。（用法折页解释了它的用途）

```
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class CustomRendererFeature : ScriptableRendererFeature {

    public class CustomRenderPass : ScriptableRenderPass {

        private Settings settings;
        private FilteringSettings filteringSettings;
        private ProfilingSampler _profilingSampler;
        private List<ShaderTagId> shaderTagsList = new List<ShaderTagId>();
        private RTHandle rtCustomColor, rtTempColor;

        public CustomRenderPass(Settings settings, string name) {
            this.settings = settings;
            filteringSettings = new FilteringSettings(RenderQueueRange.opaque, settings.layerMask);
            
            // Use default tags
            shaderTagsList.Add(new ShaderTagId("SRPDefaultUnlit"));
            shaderTagsList.Add(new ShaderTagId("UniversalForward"));
            shaderTagsList.Add(new ShaderTagId("UniversalForwardOnly"));
            
            _profilingSampler = new ProfilingSampler(name);
        }

        public override void OnCameraSetup(CommandBuffer cmd, ref RenderingData renderingData) {
            var colorDesc = renderingData.cameraData.cameraTargetDescriptor;
            colorDesc.depthBufferBits = 0;

            // Set up temporary color buffer (for blit)
            RenderingUtils.ReAllocateIfNeeded(ref rtTempColor, colorDesc, name: "_TemporaryColorTexture");

            // Set up custom color target buffer (to render objects into)
            if (settings.colorTargetDestinationID != ""){
                RenderingUtils.ReAllocateIfNeeded(ref rtCustomColor, colorDesc, name: settings.colorTargetDestinationID);
            }else{
                // colorDestinationID is blank, use camera target instead
                rtCustomColor = renderingData.cameraData.renderer.cameraColorTargetHandle;
            }

            // Using camera's depth target (that way we can ZTest with scene objects still)
            RTHandle rtCameraDepth = renderingData.cameraData.renderer.cameraDepthTargetHandle;

            ConfigureTarget(rtCustomColor, rtCameraDepth);
            ConfigureClear(ClearFlag.Color, new Color(0,0,0,0));
        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData) {
            CommandBuffer cmd = CommandBufferPool.Get();
            // Set up profiling scope for Profiler & Frame Debugger
            using (new ProfilingScope(cmd, _profilingSampler)) {
                // Command buffer shouldn't contain anything, but apparently need to
                // execute so DrawRenderers call is put under profiling scope title correctly
                context.ExecuteCommandBuffer(cmd);
                cmd.Clear();

                // Draw Renderers to Render Target (set up in OnCameraSetup)
                SortingCriteria sortingCriteria = renderingData.cameraData.defaultOpaqueSortFlags;
                DrawingSettings drawingSettings = CreateDrawingSettings(shaderTagsList, ref renderingData, sortingCriteria);
                if (settings.overrideMaterial != null) {
                    drawingSettings.overrideMaterialPassIndex = settings.overrideMaterialPass;
                    drawingSettings.overrideMaterial = settings.overrideMaterial;
                }
                context.DrawRenderers(renderingData.cullResults, ref drawingSettings, ref filteringSettings);

                // Pass our custom target to shaders as a Global Texture reference
                // In a Shader Graph, you'd obtain this as a Texture2D property with "Exposed" unticked
                if (settings.colorTargetDestinationID != "") 
                    cmd.SetGlobalTexture(settings.colorTargetDestinationID, rtCustomColor);
                
                // Apply material (e.g. Fullscreen Graph) to camera
                if (settings.blitMaterial != null) {
                    RTHandle camTarget = renderingData.cameraData.renderer.cameraColorTargetHandle;
                    if (camTarget != null && rtTempColor != null) {
                        Blitter.BlitCameraTexture(cmd, camTarget, rtTempColor, settings.blitMaterial, 0);
                        Blitter.BlitCameraTexture(cmd, rtTempColor, camTarget);
                    }
                }
            }
            // Execute Command Buffer one last time and release it
            // (otherwise we get weird recursive list in Frame Debugger)
            context.ExecuteCommandBuffer(cmd);
            cmd.Clear();
            CommandBufferPool.Release(cmd);
        }

        public override void OnCameraCleanup(CommandBuffer cmd) {}

        // Cleanup Called by feature below
        public void Dispose() {
            if (settings.colorTargetDestinationID != "")
                rtCustomColor?.Release();
            rtTempColor?.Release();
        }
    }

    // Exposed Settings

    [System.Serializable]
    public class Settings {
        public bool showInSceneView = true;
        public RenderPassEvent _event = RenderPassEvent.AfterRenderingOpaques;

        [Header("Draw Renderers Settings")]
        public LayerMask layerMask = 1;
        public Material overrideMaterial;
        public int overrideMaterialPass;
        public string colorTargetDestinationID = "";

        [Header("Blit Settings")]
        public Material blitMaterial;
    }

    public Settings settings = new Settings();

    // Feature Methods

    private CustomRenderPass m_ScriptablePass;

    public override void Create() {
        m_ScriptablePass = new CustomRenderPass(settings, name);
        m_ScriptablePass.renderPassEvent = settings._event;
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData) {
        CameraType cameraType = renderingData.cameraData.cameraType;
        if (cameraType == CameraType.Preview) return; // Ignore feature for editor/inspector previews & asset thumbnails
        if (!settings.showInSceneView && cameraType == CameraType.SceneView) return;
        renderer.EnqueuePass(m_ScriptablePass);
    }

    protected override void Dispose(bool disposing) {
        m_ScriptablePass.Dispose();
    }
}
```

The feature can be used to render objects with a given material, into a custom buffer (specified by Color Target Destination ID setting). This texture reference can be sampled as a global/unexposed Texture2D in a blit/fullscreen material.  
该功能可用于将具有给定材质的对象渲染到自定义缓冲区中（由“颜色目标目标ID”设置指定）。此纹理参考可以作为blit/fullscreen材质中的全局/未曝光Texture2D进行采样。

For example, feature could be used with materials using the following graphs (click images to view full screen) :  
例如，该功能可以与使用以下图形的材料一起使用（单击图像以查看全屏）：

[![[d077d86ce14e1ca2440f50821d68e42b_MD5.png]]](https://www.cyanilux.com/tutorials/custom-renderer-features/Graph1.png)

[](https://www.cyanilux.com/tutorials/custom-renderer-features/Graph1.png)

Override Material (Unlit Graph)  
覆盖材质（取消图表列表）

[![[bb65d14ecac981b7318cf3db0e6c1c72_MD5.png]]](https://www.cyanilux.com/tutorials/custom-renderer-features/Graph2.png)

[](https://www.cyanilux.com/tutorials/custom-renderer-features/Graph2.png)

Blit Material (Fullscreen Graph)  
Blit材质（全屏图）

Resulting in : 结果是：

[![[f5a164e2d020933148c213d44ba29e9e_MD5.png]]](https://www.cyanilux.com/tutorials/custom-renderer-features/Result.png)

[](https://www.cyanilux.com/tutorials/custom-renderer-features/Result.png)

Cyan objects are on separate Layer, not used by feature’s LayerMask setting. Note the cube doesn’t look great since we used a Fresnel Effect. Alternative outline methods may work better.  
青色对象位于单独的图层上，不由功能的图层遮罩设置使用。请注意，立方体看起来不太好，因为我们使用了菲涅耳效应。替代的大纲方法可能效果更好。

Another similar example is [this Horizon Zero Dawn inspired highlight/glitch effect](https://twitter.com/Cyanilux/status/1575877789403815940) I made a while ago.  
另一个类似的例子是我不久前制作的受地平线零点黎明启发的亮点/故障效果。

## Setting values on features at Runtime  
在运行时设置功能的值

For some effects, you may want to set public or serialised fields/properties on a feature, from a C# Script at runtime.  
对于某些效果，您可能希望在运行时从C#脚本设置功能的公共或串行字段/属性。

If this is only for a **specific** instance of a feature on a single Renderer asset, you should be able to do this quite easily by exposing a public field in your MonoBehaviour. e.g.  
如果这只是针对单个渲染器资源上某个功能的特定实例，那么通过在MonoPhavior中公开一个公共字段，应该可以很容易地做到这一点。例如。

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class SomeScript : MonoBehaviour {

    public CustomRendererFeature feature;
    // This feature uses the same "Settings class" example as shown in other sections
    // If you set public fields on the feature directly, that won't update the pass (unless you call Dispose and Create)

    // Call this method to set Override Material used by feature
    void SetMaterial(Material material, int passIndex){
        feature.settings.overrideMaterial = material;
        feature.settings.overrideMaterialPass = passIndex;
    }

}
```

If you need to do this for **multiple** instances of a feature, you can use an array of those features instead (so `HighlightRendererFeature[]` in this example). Another method could be to use a ScriptableObject to hold the Settings data, which the feature/pass and our SomeScript would have a reference to.  
如果需要对一个功能的多个实例执行此操作，则可以使用这些功能的数组（因此在本例中为 `HighlightRendererFeature[]` ）。另一种方法可以是使用ScriptableObject来保存设置数据，功能/过程和SomeScript将引用这些数据。

You could also try to get the Renderer Asset and loop through the features. But those aren’t public, so might require Reflection, not ideal.  
您也可以尝试获取渲染器资源并循环使用这些功能。但这些都不是公开的，所以可能需要反思，这并不理想。

## Connecting a feature to a URP Volume  
将功能连接到URP卷

While URP doesn’t have an official way to make custom post processing effects for it’s Volume system, it is still possible to create a custom VolumeComponent for it. We could use this to expose fields/properties, which communicates with a custom pass/feature.  
虽然URP没有为其Volume系统制作自定义后处理效果的官方方法，但仍然可以为其创建自定义VolumeComponent。我们可以使用它来公开字段/属性，该字段/属性与自定义传递/功能进行通信。

I won’t be including an example here as [Febucci already has good tutorial of this](https://www.febucci.com/2022/05/custom-post-processing-in-urp/) (though note the code for the pass uses RenderTargetIdentifier and should be converted to RTHandles in 2022+)  
我不会在这里包括一个例子，因为Febucci已经有了很好的教程（不过请注意，通行证的代码使用RenderTargetIdentifier，应该在2022+中转换为RTHandles）

The next section may also provide a small improvement to this, as we can get the ScriptableRenderPass to run without even needing a Renderer Feature. The example below is for a MonoBehaviour but maybe the same thing could work for VolumeComponent.OnEnable/OnDisable.  
下一节还可能对此进行一些小的改进，因为我们甚至可以在不需要渲染器功能的情况下运行ScriptableRenderPass。下面的例子是针对MonoBehavior的，但可能同样的事情也适用于VolumeComponent.OnEnable/OnDisable。

## RenderPipelineManager 渲染管线管理器

While not necessarily the scope of this post, we can also inject code before or after rendering each frame/camera using the events in the [RenderPipelineManager](https://docs.unity3d.com/ScriptReference/Rendering.RenderPipelineManager.html) class.  
虽然不一定是本文的范围，但我们也可以使用RenderPipelineManager类中的事件在渲染每个帧/相机之前或之后注入代码。

I wanted to mention this as it is possible to enqueue Scriptable Render Passes to the renderer here if you don’t want to have to create/assign the Renderer Feature. This is also useful for older versions of URP where the 2D Renderer did not support features.  
我想提到这一点，因为如果您不想创建/指定渲染器功能，可以在此处将可脚本渲染过程排队到渲染器。这对于2D渲染器不支持功能的旧版本URP也很有用。

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

Of course the script here would still need to be put onto an GameObject in the scene - but that could be considered easier than adding a Renderer Feature? 🤷  
当然，这里的脚本仍然需要放在场景中的游戏对象上，但这可以被认为比添加渲染器功能更容易吗？🤷  

## Thanks for reading! 感谢阅读！

If this post helped, consider sharing a link with others!  
如果这篇文章有帮助，可以考虑与其他人共享链接！