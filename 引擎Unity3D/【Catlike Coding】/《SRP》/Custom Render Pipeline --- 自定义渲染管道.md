Taking Control of Rendering  
控制渲染

*   Create a render pipeline asset and instance.  
    创建渲染管道资源和实例。
*   Render a camera's view.  
    渲染摄影机的视图。
*   Perform culling, filtering, and sorting.  
    执行剔除、筛选和排序。
*   Separate opaque, transparent, and invalid passes.  
    分离不透明、透明和无效的过程。
*   Work with more than one camera.  
    使用多台相机。

sr-annote { all: unset; }

This is the first part of a tutorial series about creating a [custom scriptable render pipeline](https://catlikecoding.com/unity/tutorials/custom-srp/). It covers the initial creation of a bare-bones render pipeline that we will expand in the future.  
这是关于创建自定义可脚本化渲染管道的系列教程的第一部分。它涵盖了我们将在未来扩展的基本渲染管道的初始创建。

This series assumes that you've worked through at least the [Object Management](https://catlikecoding.com/unity/tutorials/object-management/) series and the [Procedural Grid](https://catlikecoding.com/unity/tutorials/procedural-grid/) tutorial.  
本系列假设您至少完成了对象管理系列和过程网格教程。

This tutorial is made with Unity 2019.2.6f1.  
本教程是用Unity 2019.2.6f1制作的。

![](<images/1686828521660.png>)

Rendering with a custom render pipeline.  
使用自定义渲染管道进行渲染。

## A new Render Pipeline  
新的渲染管道

To render anything, Unity has to determine what shapes have to be drawn, where, when, and with what settings. This can get very complex, depending on how many effects are involved. Lights, shadows, transparency, image effects, volumetric effects, and so on all have to be dealt with in the correct order to arrive at the final image. This is what a render pipeline does.  
要渲染任何东西，Unity必须确定必须绘制什么形状，在哪里、何时以及使用什么设置。这可能会变得非常复杂，这取决于所涉及的影响有多少。灯光、阴影、透明度、图像效果、体积效果等等都必须按照正确的顺序进行处理才能得到最终图像。这就是渲染管道的作用。

In the past Unity only supported a few built-in ways to render things. Unity 2018 introduced scriptable render pipelines—RPs for short—making it possible to do whatever we want, while still being able to rely on Unity for fundamental steps like culling. Unity 2018 also added two experimental RPs made with this new approach: the Lightweight RP and the High Definition RP. In Unity 2019 the Lightweight RP is no longer experimental and got rebranded to the Universal RP in Unity 2019.3.  
在过去，Unity只支持一些内置的渲染方式。Unity 2018引入了可编写脚本的渲染管道RP，使我们可以随心所欲，同时仍然可以依靠Unity来执行基本步骤，如剔除。Unity 2018还增加了两种采用这种新方法制造的实验RP：轻量级RP和高清RP。在Unity 2019中，轻量级RP不再是实验性的，并在Unity 2019.3中更名为通用RP。

The Universal RP is destined to replace the current legacy RP as the default. The idea is that it is a one-size-fits-most RP that will also be fairly easy to customize. Rather than customizing that RP this series will create an entire RP from scratch.  
通用RP将取代当前遗留RP作为默认RP。这个想法是，它是一个适合大多数RP的尺寸，也很容易定制。这个系列将从头开始创建一个完整的RP，而不是自定义该RP。

This tutorial lays the foundation with a minimal RP that draws unlit shapes using forward rendering. Once that's working, we can extend our pipeline in later tutorials, adding lighting, shadows, different rendering methods, and more advanced features.  
本教程为使用正向渲染绘制未发光形状的最小RP奠定了基础。一旦成功，我们可以在后面的教程中扩展我们的管道，添加照明、阴影、不同的渲染方法和更高级的功能。

### Project Setup 项目设置

Create a new 3D project in Unity 2019.2.6 or later. We'll create our own pipeline, so don't select one of the RP project templates. Once the project is open you can go to the package manager and remove all packages that you don't need. We'll only use the _Unity UI_ package in this tutorial to experiment with drawing the UI, so you can keep that one.  
在Unity 2019.2.6或更高版本中创建一个新的3D项目。我们将创建自己的管道，所以不要选择RP项目模板之一。项目打开后，您可以转到包管理器，删除所有不需要的包。我们将只使用本教程中的包来尝试绘制UI，因此您可以保留该包。

We're going to exclusively work in linear color space, but Unity 2019.2 still uses gamma space as the default. Go to the player settings via _Edit / Project Settings_ and then _Player_, then switch _Color Space_ under the _Other Settings_ section to _Linear_.  
我们将只在线性颜色空间中工作，但Unity 2019.2仍然使用伽马空间作为默认空间。通过和转到播放器设置，然后在部分下切换至。

![](<images/1686828522253.png>)

Color space set to linear.  
颜色空间设置为线性。

Fill the default scene with a few objects, using a mix of standard, unlit opaque and transparent materials. The _Unlit/Transparent_ shader only works with a texture, so [here](https://catlikecoding.com/unity/tutorials/custom-srp/custom-render-pipeline/a-new-render-pipeline/sphere-alpha-map.png) is a UV sphere map for that.  
使用标准、未发光的不透明和透明材质的混合，使用一些对象填充默认场景。着色器仅适用于纹理，因此这里有一个UV球体贴图。

![](<images/1686828522810.png>)

UV sphere alpha map, on black background.  
UV球体alpha贴图，在黑色背景上。

I put a few cubes in my test scene, all of which are opaque. The red ones use a material with the _Standard_ shader while the green and yellow ones use a material with the _Unlit/Color_ shader. The blue spheres use the _Standard_ shader with _Rendering Mode_ set to _Transparent_, while the white spheres use the _Unlit/Transparent_ shader.  
我在测试场景中放置了几个立方体，所有这些立方体都是不透明的。红色的将材质用于着色器，而绿色和黄色的将材质与着色器一起使用。蓝色球体使用设置为的着色器，而白色球体使用着色器。

![](<images/1686828523389.png>)

Test scene. 测试场景。

### Pipeline Asset 管道资产

Currently, Unity uses the default render pipeline. To replace it with a custom render pipeline we first have to create an asset type for it. We'll use roughly the same folder structure that Unity uses for the Universal RP. Create a _Custom RP_ asset folder with a _Runtime_ child folder. Put a new C# script in there for the `**CustomRenderPipelineAsset**` type.  
目前，Unity使用默认的渲染管道。要用自定义渲染管道替换它，我们首先必须为它创建一个资产类型。我们将使用与Unity用于通用RP大致相同的文件夹结构。创建一个带有子文件夹的资产文件夹。为 `**CustomRenderPipelineAsset**` 类型放入一个新的C#脚本。

![](<images/1686828523939.png>)

Folder structure. 文件夹结构。

The asset type must extend `[RenderPipelineAsset](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderPipelineAsset.html)` from the `UnityEngine.Rendering` namespace.  
资产类型必须从 `UnityEngine.Rendering` 命名空间扩展 `[RenderPipelineAsset](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderPipelineAsset.html)` 。

```
using UnityEngine;
using UnityEngine.Rendering;

public class CustomRenderPipelineAsset : RenderPipelineAsset {}
```

The main purpose of the RP asset is to give Unity a way to get a hold of a pipeline object instance that is responsible for rendering. The asset itself is just a handle and a place to store settings. We don't have any settings yet, so all we have to do is give Unity a way to get our pipeline object instance. That's done by overriding the abstract `CreatePipeline` method, which should return a `[RenderPipeline](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderPipeline.html)` instance. But we haven't defined a custom RP type yet, so begin by returning `**null**`.  
RP资产的主要目的是为Unity提供一种获取负责渲染的管道对象实例的方法。资源本身只是一个句柄和存储设置的地方。我们还没有任何设置，所以我们所要做的就是给Unity一种获取管道对象实例的方法。这是通过重写抽象的 `CreatePipeline` 方法来完成的，该方法应该返回 [RenderPipeline](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderPipeline.html) 实例。但是我们还没有定义自定义RP类型，所以从返回 `**null**` 开始。

The `CreatePipeline` method is defined with the `**protected**` access modifier, which means that only the class that defined the method—which is `[RenderPipelineAsset](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderPipelineAsset.html)`—and those that extend it can access it.  
`CreatePipeline` 方法是用 `**protected**` 访问修饰符定义的，这意味着只有定义了#2方法的类和扩展它的类才能访问它。

```
protected override RenderPipeline CreatePipeline () {
		return null;
	}
```

Now we need to add an asset of this type to our project. To make that possible, add a `[CreateAssetMenu](http://docs.unity3d.com/Documentation/ScriptReference/CreateAssetMenuAttribute.html)` attribute to `**CustomRenderPipelineAsset**`.  
现在我们需要将这种类型的资产添加到我们的项目中。要实现这一点，请在 `**CustomRenderPipelineAsset**` 中添加 `[CreateAssetMenu](http://docs.unity3d.com/Documentation/ScriptReference/CreateAssetMenuAttribute.html)` 属性。

```
[CreateAssetMenu]
public class CustomRenderPipelineAsset : RenderPipelineAsset { … }
```

That puts an entry in the _Asset / Create_ menu. Let's be tidy and put it in a _Rendering_ submenu. We do that by setting the `menuName` property of the attribute to _Rendering/Custom Render Pipeline_. This property can be set directly after the attribute type, within round brackets.  
这会在菜单中放入一个条目。让我们整理一下，把它放在一个子菜单里。我们通过将属性的 `menuName` 属性设置为来实现这一点。此属性可以直接设置在属性类型后面的圆括号内。

```
[CreateAssetMenu(menuName = "Rendering/Custom Render Pipeline")]
public class CustomRenderPipelineAsset : RenderPipelineAsset { … }
```

Use the new menu item to add the asset to the project, then go to the _Graphics_ project settings and select it under _Scriptable Render Pipeline Settings_.  
使用新菜单项将资源添加到项目中，然后转到项目设置并在下进行选择。

![](<images/1686828524477.png>)

Custom RP selected. 已选择自定义RP。

Replacing the default RP changed a few things. First, a lot of options have disappeared from the graphics settings, which is mentioned in an info panel. Second, we've disabled the default RP without providing a valid replacement, so nothing gets rendered anymore. The game window, scene window, and material previews are no longer functional. If you open the frame debugger—via _Window / Analysis / Frame Debugger_—and enable it, you will see that indeed nothing gets drawn in the game window.  
替换默认RP改变了一些事情。首先，很多选项已经从图形设置中消失，这在信息面板中提到。其次，我们在没有提供有效替换的情况下禁用了默认RP，因此不再渲染任何内容。游戏窗口、场景窗口和材质预览不再起作用。如果您通过-打开框架调试器并启用它，您将看到游戏窗口中实际上没有绘制任何内容。

### Render Pipeline Instance  
渲染管道实例

Create a `**CustomRenderPipeline**` class and put its script file in the same folder as `**CustomRenderPipelineAsset**`. This will be the type used for the RP instance that our asset returns, thus it must extend `[RenderPipeline](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderPipeline.html)`.  
创建一个 `**CustomRenderPipeline**` 类，并将其脚本文件放在与 `**CustomRenderPipelineAsset**` 相同的文件夹中。这将是我们的资产返回的RP实例所使用的类型，因此它必须扩展#2。

```
using UnityEngine;
using UnityEngine.Rendering;

public class CustomRenderPipeline : RenderPipeline {}
```

`[RenderPipeline](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderPipeline.html)` defines a protected abstract `Render` method that we have to override to create a concrete pipeline. It has two parameters: a `[ScriptableRenderContext](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.ScriptableRenderContext.html)` and a `[Camera](http://docs.unity3d.com/Documentation/ScriptReference/Camera.html)` array. Leave the method empty for now.  
`[RenderPipeline](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderPipeline.html)` 定义了一个受保护的抽象 `Render` 方法，我们必须重写该方法才能创建具体的管道。它有两个参数：一个#2数组和一个#3数组。暂时将该方法保留为空。

```
protected override void Render ( ScriptableRenderContext context, Camera[] cameras ) {}
```

Make `**CustomRenderPipelineAsset**.CreatePipeline` return a new instance of `**CustomRenderPipeline**`. That will get us a valid and functional pipeline, although it doesn't render anything yet.  
使 `**CustomRenderPipelineAsset**.CreatePipeline` 返回 `**CustomRenderPipeline**` 的新实例。这将为我们提供一个有效且实用的管道，尽管它还没有呈现任何内容。

```
protected override RenderPipeline CreatePipeline () {
		return new CustomRenderPipeline();
	}
```

## Rendering 翻译

Each frame Unity invokes `Render` on the RP instance. It passes along a context struct that provides a connection to the native engine, which we can use for rendering. It also passes an array of cameras, as there can be multiple active cameras in the scene. It is the RP's responsibility to render all those cameras in the order that they are provided.  
每个帧Unity在RP实例上调用 `Render` 。它传递了一个上下文结构，该结构提供了与本机引擎的连接，我们可以使用本机引擎进行渲染。它还经过一组摄影机，因为场景中可能有多个活动摄影机。RP有责任按照提供的顺序渲染所有这些相机。

### Camera Renderer 摄影机渲染器

Each camera gets rendered independently. So rather than have `**CustomRenderPipeline**` render all camera's we'll forward that responsibility to a new class dedicated to rendering one camera. Name it `**CameraRenderer**` and give it a public `Render` method with a context and a camera parameter. Let's store these parameters in fields for convenience.  
每个摄影机都会独立渲染。因此，与其让 `**CustomRenderPipeline**` 渲染所有相机，我们将把这一责任转交给一个专门渲染一台相机的新类。将其命名为 `**CameraRenderer**` ，并为其提供一个带有上下文和相机参数的公共 `Render` 方法。为了方便起见，让我们将这些参数存储在字段中。

```
using UnityEngine;
using UnityEngine.Rendering;

public class CameraRenderer {

	ScriptableRenderContext context;

	Camera camera;

	public void Render (ScriptableRenderContext context, Camera camera) {
		this.context = context;
		this.camera = camera;
	}
}
```

Have `**CustomRenderPipeline**` create an instance of the renderer when it gets created, then use it to render all cameras in a loop.  
让 `**CustomRenderPipeline**` 在创建渲染器实例时创建该实例，然后使用它来渲染循环中的所有摄影机。

```
CameraRenderer renderer = new CameraRenderer();

	protected override void Render ( ScriptableRenderContext context, Camera[] cameras ) {
		foreach (Camera camera in cameras) {
			renderer.Render(context, camera);
		}
	}
```

Our camera renderer is roughly equivalent to the scriptable renderers of the Universal RP. This approach will make it simple to support different rendering approaches per camera in the future, for example one for the first-person view and one for a 3D map overlay, or forward vs. deferred rendering. But for now we'll render all cameras the same way.  
我们的相机渲染器大致相当于Universal RP的可脚本化渲染器。这种方法将使未来每个相机支持不同的渲染方法变得简单，例如，一种用于第一人称视图，另一种用于3D地图覆盖，或者前向渲染与延迟渲染。但现在我们将以相同的方式渲染所有摄影机。

### Drawing the Skybox 绘制Skybox

The job of `**CameraRenderer**.Render` is to draw all geometry that its camera can see. Isolate that specific task in a separate `DrawVisibleGeometry` method for clarity. We'll begin by having it draw the default the skybox, which can be done by invoking `DrawSkybox` on the context with the camera as an argument.  
`**CameraRenderer**.Render` 的任务是绘制其相机可以看到的所有几何体。为了清晰起见，用单独的 `DrawVisibleGeometry` 方法隔离该特定任务。我们将首先让它绘制默认的skybox，这可以通过调用上下文中的#2来完成，并将相机作为参数。

```
public void Render (ScriptableRenderContext context, Camera camera) {
		this.context = context;
		this.camera = camera;

		DrawVisibleGeometry();
	}

	void DrawVisibleGeometry () {
		context.DrawSkybox(camera);
	}
```

This does not yet make the skybox appear. That's because the commands that we issue to the context are buffered. We have to submit the queued work for execution, by invoking `Submit` on the context. Let's do this in a separate `Submit` method, invoked after `DrawVisibleGeometry`.  
这还没有使skybox出现。这是因为我们向上下文发出的命令是缓冲的。我们必须通过调用上下文上的 `Submit` 来提交排队的工作以供执行。让我们在一个单独的 `Submit` 方法中执行此操作，该方法在 `DrawVisibleGeometry` 之后调用。

```
public void Render (ScriptableRenderContext context, Camera camera) {
		this.context = context;
		this.camera = camera;

		DrawVisibleGeometry();
		Submit();
	}

	void Submit () {
		context.Submit();
	}
```

The skybox finally appears in both the game and scene window. You can also see an entry for it in the frame debugger when you enable it. It's listed as _Camera.RenderSkybox_, which has a single _Draw Mesh_ item under it, which represents the actual draw call. This corresponds to the rendering of the game window. The frame debugger doesn't report drawing in other windows.  
skybox最终出现在游戏和场景窗口中。启用它时，您还可以在框架调试器中看到它的条目。它列为，下面有一个项目，表示实际的绘制调用。这与游戏窗口的渲染相对应。框架调试器不报告其他窗口中的绘图。

![](<images/1686828525074.png>)

  

![](<images/1686828525626.png>)

Skybox gets drawn. Skybox被抽中。

Note that the orientation of the camera currently doesn't affect how the skybox gets rendered. We pass the camera to `DrawSkybox`, but that's only used to determine whether the skybox should be drawn at all, which is controlled via the camera's clear flags.  
请注意，摄影机的方向当前不会影响skybox的渲染方式。我们将相机传递给 `DrawSkybox` ，但这仅用于确定是否应该绘制skybox，这是通过相机的清晰标志控制的。

To correctly render the skybox—and the entire scene—we have to set up the view-projection matrix. This transformation matrix combines the camera's position and orientation—the view matrix—with the camera's perspective or orthographic projection—the projection matrix. It is known in shaders as _unity_MatrixVP_, one of the shader properties used when geometry gets drawn. You can inspect this matrix in the frame debugger's _ShaderProperties_ section when a draw call is selected.  
为了正确渲染skybox和整个场景，我们必须设置视图投影矩阵。该变换矩阵将摄影机的位置和方向（视图矩阵）与摄影机的透视或正交投影（投影矩阵）相结合。它在着色器中被称为，是绘制几何体时使用的着色器属性之一。当选择绘制调用时，可以在框架调试器的部分中检查此矩阵。

At the moment, the _unity_MatrixVP_ matrix is always the same. We have to apply the camera's properties to the context, via the `SetupCameraProperties` method. That sets up the matrix as well as some other properties. Do this before invoking `DrawVisibleGeometry`, in a separate `Setup` method.  
目前，矩阵总是相同的。我们必须通过 `SetupCameraProperties` 方法将相机的属性应用于上下文。这设置了矩阵以及其他一些属性。在调用 `DrawVisibleGeometry` 之前，在单独的 `Setup` 方法中执行此操作。

```
public void Render (ScriptableRenderContext context, Camera camera) {
		this.context = context;
		this.camera = camera;

		Setup();
		DrawVisibleGeometry();
		Submit();
	}

	void Setup () {
		context.SetupCameraProperties(camera);
	}
```

![](<images/1686828526171.png>)

Skybox, correctly aligned.  
Skybox，正确对齐。

### Command Buffers 命令缓冲区

The context delays the actual rendering until we submit it. Before that, we configure it and add commands to it for later execution. Some tasks—like drawing the skybox—can be issued via a dedicated method, but other commands have to be issued indirectly, via a separate command buffer. We need such a buffer to draw the other geometry in the scene.  
上下文会延迟实际渲染，直到我们提交它。在此之前，我们会对其进行配置并向其添加命令以供稍后执行。一些任务，如绘制skybox，可以通过专用方法发出，但其他命令必须通过单独的命令缓冲区间接发出。我们需要这样一个缓冲区来绘制场景中的其他几何体。

To get a buffer we have to create a new `[CommandBuffer](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.CommandBuffer.html)` object instance. We need only one buffer, so create one by default for `**CameraRenderer**` and store a reference to it in a field. Also give the buffer a name so we can recognize it in the frame debugger. _Render Camera_ will do.  
为了获得缓冲区，我们必须创建一个新的 `[CommandBuffer](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.CommandBuffer.html)` 对象实例。我们只需要一个缓冲区，所以默认情况下为 `**CameraRenderer**` 创建一个缓冲区时，并在字段中存储对它的引用。还要给缓冲区一个名称，这样我们就可以在帧调试器中识别它。会的。

```
const string bufferName = "Render Camera";

	CommandBuffer buffer = new CommandBuffer {
		name = bufferName
	};
```

We can use command buffers to inject profiler samples, which will show up both in the profiler and the frame debugger. This is done by invoking `BeginSample` and `EndSample` at the appropriate points, which is at the beginning of `Setup` and `Submit` in our case. Both methods must be provided with the same sample name, for which we'll use the buffer's name.  
我们可以使用命令缓冲区来注入探查器样本，这些样本将显示在探查器和框架调试器中。这是通过在适当的点调用 `BeginSample` 和 `EndSample` 来完成的，在我们的例子中，这是在#2和#3的开头。两个方法必须提供相同的样本名称，我们将使用缓冲区的名称。

```
void Setup () {
		buffer.BeginSample(bufferName);
		context.SetupCameraProperties(camera);
	}

	void Submit () {
		buffer.EndSample(bufferName);
		context.Submit();
	}
```

To execute the buffer, invoke `ExecuteCommandBuffer` on the context with the buffer as an argument. That copies the commands from the buffer but doesn't clear it, we have to do that explicitly afterwards if we want to reuse it. Because execution and clearing is always done together it's handy to add a method that does both.  
若要执行缓冲区，请在上下文中调用 `ExecuteCommandBuffer` ，并将缓冲区作为参数。它从缓冲区复制命令，但不清除它，如果我们想重用它，我们必须在之后显式地执行。因为执行和清除总是一起完成的，所以添加一个同时执行这两项的方法很方便。

```
void Setup () {
		buffer.BeginSample(bufferName);
		ExecuteBuffer();
		context.SetupCameraProperties(camera);
	}

	void Submit () {
		buffer.EndSample(bufferName);
		ExecuteBuffer();
		context.Submit();
	}

	void ExecuteBuffer () {
		context.ExecuteCommandBuffer(buffer);
		buffer.Clear();
	}
```

The _Camera.RenderSkyBox_ sample now gets nested inside _Render Camera_.  
样本现在嵌套在里面。

![](<images/1686828526730.png>)

Render camera sample. 渲染摄影机示例。

### Clearing the Render Target  
清除渲染目标

Whatever we draw ends up getting rendered to the camera's render target, which is the frame buffer by default but could also be a render texture. Whatever was drawn to that target earlier is still there, which could interfere with the image that we are rendering now. To guarantee proper rendering we have to clear the render target to get rid of its old contents. That's done by invoking `ClearRenderTarget` on the command buffer, which belongs in the `Setup` method.  
我们绘制的任何内容最终都会渲染到摄影机的渲染目标，默认情况下，渲染目标是帧缓冲区，但也可能是渲染纹理。之前被吸引到那个目标的东西仍然存在，这可能会干扰我们现在渲染的图像。为了保证正确的渲染，我们必须清除渲染目标以清除其旧内容。这是通过调用命令缓冲区上的 `ClearRenderTarget` 来完成的，该缓冲区属于 `Setup` 方法。

`[CommandBuffer](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.CommandBuffer.html).ClearRenderTarget` requires at least three arguments. The first two indicate whether the depth and color data should be cleared, which is true for both. The third argument is the color used to clearing, for which we'll use `[Color](http://docs.unity3d.com/Documentation/ScriptReference/Color.html).clear`.  
`[CommandBuffer](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.CommandBuffer.html).ClearRenderTarget` 至少需要三个参数。前两个指示是否应该清除深度和颜色数据，这两个都是正确的。第三个参数是用于清除的颜色，我们将使用 `[Color](http://docs.unity3d.com/Documentation/ScriptReference/Color.html).clear` 。

```
void Setup () {
		buffer.BeginSample(bufferName);
		buffer.ClearRenderTarget(true, true, Color.clear);
		ExecuteBuffer();
		context.SetupCameraProperties(camera);
	}
```

![](<images/1686828527261.png>)

Clearing, with nested sample.  
清除，带有嵌套样本。

The frame debugger now shows a _Draw GL_ entry for the clear action, which shows up nested in an additional level of _Render Camera_. That happens because `ClearRenderTarget` wraps the clearing in a sample with the command buffer's name. We can get rid of the redundant nesting by clearing before beginning our own sample. That results in two adjacent _Render Camera_ sample scopes, which get merged.  
框架调试器现在显示清除操作的条目，该条目嵌套在的附加级别中。之所以会发生这种情况，是因为 `ClearRenderTarget` 使用命令缓冲区的名称将清除封装在一个示例中。我们可以在开始自己的样本之前通过清除来消除多余的嵌套。这将导致两个相邻的示例作用域合并。

```
void Setup () {
		buffer.ClearRenderTarget(true, true, Color.clear);
		buffer.BeginSample(bufferName);
		
		ExecuteBuffer();
		context.SetupCameraProperties(camera);
	}
```

![](<images/1686828527829.png>)

Clearing, without nesting.  
清除，不嵌套。

The _Draw GL_ entry represent drawing a full-screen quad with the _Hidden/InternalClear_ shader that writes to the render target, which isn't the most efficient way to clear it. This approach is used because we're clearing before setting up the camera properties. If we swap the order of those two steps we get the quick way to clear.  
该条目表示使用写入渲染目标的着色器绘制全屏四边形，这不是清除它的最有效方法。之所以使用此方法，是因为我们在设置相机属性之前进行清除。如果我们交换这两个步骤的顺序，我们可以快速清除。

```
void Setup () {
		context.SetupCameraProperties(camera);
		buffer.ClearRenderTarget(true, true, Color.clear);
		buffer.BeginSample(bufferName);
		ExecuteBuffer();
		
	}
```

![](<images/1686828528412.png>)

Correct clearing. 正确清理。

Now we see _Clear (color+Z+stencil)_, which indicates that both the color and depth buffers get cleared. Z represents the depth buffer and the stencil data is part the same buffer.  
现在我们看到了，这表明颜色和深度缓冲区都被清除了。Z表示深度缓冲区，模具数据是同一缓冲区的一部分。

### Culling 剔除

We're currently seeing the skybox, but not any of the objects that we put in the scene. Rather than drawing every object, we're only going to render those that are visible to the camera. We do that by starting with all objects with renderer components in the scene and then culling those that fall outside of the view frustum of the camera.  
我们目前看到的是天空盒子，但没有看到我们放在场景中的任何物体。与其绘制每个对象，我们只渲染那些对摄影机可见的对象。我们要做到这一点，首先从场景中具有渲染器组件的所有对象开始，然后剔除那些位于摄影机视锥之外的对象。

Figuring out what can be culled requires us to keep track of multiple camera settings and matrices, for which we can use the `[ScriptableCullingParameters](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.ScriptableCullingParameters.html)` struct. Instead of filling it ourselves, we can invoke `TryGetCullingParameters` on the camera. It returns whether the parameters could be successfully retrieved, as it might fail for degenerate camera settings. To get hold of the parameter data we have to supply it as an output argument, by writing `**out**` in front of it. Do this in a separate `Cull` method that returns either success or failure.  
要想找出可以剔除的内容，我们需要跟踪多个相机设置和矩阵，为此我们可以使用 `[ScriptableCullingParameters](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.ScriptableCullingParameters.html)` 结构。我们可以在摄像机上调用 `TryGetCullingParameters` ，而不是自己填充。它返回是否可以成功检索参数，因为它可能会因退化的相机设置而失败。要获取参数数据，我们必须将其作为输出参数提供，方法是在其前面写入#2。在返回成功或失败的单独#3方法中执行此操作。

```
bool Cull () {
		ScriptableCullingParameters p
		if (camera.TryGetCullingParameters(out p)) {
			return true;
		}
		return false;
	}
```

It is possible to inline the variable declaration inside the argument list when used as an output argument, so let's do that.  
当用作输出参数时，可以在参数列表中内联变量声明，所以让我们这样做。

```
bool Cull () {
		
		if (camera.TryGetCullingParameters(out ScriptableCullingParameters p)) {
			return true;
		}
		return false;
	}
```

Invoke `Cull` before `Setup` in `Render` and abort if it failed.  
在#2中的 `Setup` 之前调用 `Cull` ，如果失败则中止。

```
public void Render (ScriptableRenderContext context, Camera camera) {
		this.context = context;
		this.camera = camera;

		if (!Cull()) {
			return;
		}

		Setup();
		DrawVisibleGeometry();
		Submit();
	}
```

Actual culling is done by invoking `Cull` on the context, which produces a `[CullingResults](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.CullingResults.html)` struct. Do this in `Cull` if successful and store the results in a field. In this case we have to pass the culling parameters as a reference argument, by writing `**ref**` in front of it.  
实际的剔除是通过调用上下文上的 `Cull` 来完成的，这将生成 `[CullingResults](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.CullingResults.html)` 结构。如果成功，请在#2中执行此操作，并将结果存储在字段中。在这种情况下，我们必须通过在前面写入 `**ref**` 来传递剔除参数作为引用参数。

```
CullingResults cullingResults;

	…
	
	bool Cull () {
		if (camera.TryGetCullingParameters(out ScriptableCullingParameters p)) {
			cullingResults = context.Cull(ref p);
			return true;
		}
		return false;
	}
```

### Drawing Geometry 绘制几何图形

Once we know what is visible we can move on to rendering those things. That is done by invoking `DrawRenderers` on the context with the culling results as an argument, telling it which renderers to use. Besides that, we have to supply drawing settings and filtering settings. Both are structs—`[DrawingSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.DrawingSettings.html)` and `[FilteringSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.FilteringSettings.html)`—for which we'll initially use their default constructors. Both have to be passed by reference. Do this in `DrawVisibleGeometry`, before drawing the skybox.  
一旦我们知道了什么是可见的，我们就可以继续渲染这些东西。这是通过调用上下文上的 `DrawRenderers` 来完成的，并将剔除结果作为参数，告诉它要使用哪些渲染器。除此之外，我们还必须提供绘图设置和过滤设置。这两个结构都是 `[DrawingSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.DrawingSettings.html)` 和 `[FilteringSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.FilteringSettings.html)` ，我们最初将使用它们的默认构造函数。两者都必须通过引用传递。在绘制skybox之前，请在#3中执行此操作。

```
void DrawVisibleGeometry () {
		var drawingSettings = new DrawingSettings();
		var filteringSettings = new FilteringSettings();

		context.DrawRenderers(
			cullingResults, ref drawingSettings, ref filteringSettings
		);

		context.DrawSkybox(camera);
	}
```

We don't see anything yet because we also have to indicate which kind of shader passes are allowed. As we only support unlit shaders in this tutorial we have to fetch the shader tag ID for the _SRPDefaultUnlit_ pass, which we can do once and cache it in a static field.  
我们还没有看到任何东西，因为我们还必须指出允许哪种着色器过程。由于我们在本教程中只支持未照明的着色器，因此我们必须获取过程的着色器标记ID，我们可以执行一次，并将其缓存在静态字段中。

```
static ShaderTagId unlitShaderTagId = new ShaderTagId("SRPDefaultUnlit");
```

Provide it as the first argument of the `[DrawingSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.DrawingSettings.html)` constructor, along with a new `[SortingSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.SortingSettings.html)` struct value. Pass the camera to the constructor of the sorting settings, as it's used to determine whether orthographic or distance-based sorting applies.  
将其作为 `[DrawingSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.DrawingSettings.html)` 构造函数的第一个参数提供，并提供一个新的 `[SortingSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.SortingSettings.html)` 结构值。将相机传递给排序设置的构造函数，因为它用于确定应用正交排序还是基于距离的排序。

```
void DrawVisibleGeometry () {
		var sortingSettings = new SortingSettings(camera);
		var drawingSettings = new DrawingSettings(
			unlitShaderTagId, sortingSettings
		);
		…
	}
```

Besides that we also have to indicate which render queues are allowed. Pass `[RenderQueueRange](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderQueueRange.html).all` to the `[FilteringSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.FilteringSettings.html)` constructor so we include everything.  
除此之外，我们还必须指出哪些渲染队列是允许的。将 `[RenderQueueRange](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderQueueRange.html).all` 传递给 `[FilteringSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.FilteringSettings.html)` 构造函数，以便我们包含所有内容。

```
var filteringSettings = new FilteringSettings(RenderQueueRange.all);
```

![](<images/1686828528980.png>)

  

![](<images/1686828529512.png>)

Drawing unlit geometry. 绘制未照明的几何图形。

Only the visible objects that use the unlit shader get drawn. All the draw calls are listed in the frame debugger, grouped under _RenderLoop.Draw_. There's something weird going on with transparent objects, but let's first look at the order in which the objects are drawn. That's shown by the frame debugger and you can step through the draw calls by selecting one after the other or using the arrow keys.  
仅绘制使用未照明着色器的可见对象。所有绘制调用都列在框架调试器中，分组在下。透明对象有一些奇怪的地方，但让我们首先看看对象的绘制顺序。这由框架调试器显示，您可以通过一个接一个地选择或使用箭头键来逐步完成绘制调用。

 <video src="" control></video>

 

Stepping through the frame debugger.  
逐步通过帧调试器。

The drawing order is haphazard. We can force a specific draw order by setting the `criteria` property of the sorting settings. Let's use `[SortingCriteria](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.SortingCriteria.html).CommonOpaque`.  
绘图顺序杂乱无章。我们可以通过设置排序设置的 `criteria` 属性来强制执行特定的绘制顺序。让我们使用 `[SortingCriteria](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.SortingCriteria.html).CommonOpaque` 。

```
var sortingSettings = new SortingSettings(camera) {
			criteria = SortingCriteria.CommonOpaque
		};
```

<video src="" control></video>

  

![](<images/1686828530043.png>)

Common opaque sorting. 常见的不透明排序。

Objects now get more-or-less drawn front-to-back, which is ideal for opaque objects. If something ends up drawn behind something else its hidden fragments can be skipped, which speeds up rendering. The common opaque sorting option also takes some other criteria into consideration, including the render queue and materials.  
现在，对象或多或少都是从前到后绘制的，这对于不透明对象来说非常理想。如果某个东西最终被绘制在其他东西后面，则可以跳过其隐藏的片段，从而加快渲染速度。常见的不透明排序选项还考虑了其他一些标准，包括渲染队列和材质。

### Drawing Opaque and Transparent Geometry Separately  
分别绘制不透明和透明几何图形

The frame debugger shows us that transparent objects get drawn, but the skybox gets drawn over everything that doesn't end up in front of an opaque object. The skybox gets drawn after the opaque geometry so all its hidden fragments can get skipped, but it overwrites transparent geometry. That happens because transparent shaders do not write to the depth buffer. They don't hide whatever's behind them, because we can see through them. The solution is to first drawn opaque objects, then the skybox, and only then transparent objects.  
框架调试器向我们展示了绘制透明对象，但天空框会绘制在所有没有出现在不透明对象前面的对象上。skybox是在不透明几何体之后绘制的，因此可以跳过其所有隐藏片段，但它会覆盖透明几何体。之所以会发生这种情况，是因为透明着色器不会写入深度缓冲区。他们不会隐藏背后的一切，因为我们可以看穿他们。解决方案是先绘制不透明对象，然后绘制天空框，然后再绘制透明对象。

We can eliminate the transparent objects from the initial `DrawRenderers` invocation by switching to `[RenderQueueRange](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderQueueRange.html).opaque`.  
通过切换到 `[RenderQueueRange](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderQueueRange.html).opaque` ，我们可以从最初的 `DrawRenderers` 调用中消除透明对象。

```
var filteringSettings = new FilteringSettings(RenderQueueRange.opaque);
```

Then after drawing the skybox invoke `DrawRenderers` again. But before doing so change the render queue range to `[RenderQueueRange](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderQueueRange.html).transparent`. Also change the sorting criteria to `[SortingCriteria](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.SortingCriteria.html).CommonTransparent` and again set the sorting of the drawing settings. That reverses the draw order of the transparent objects.  
然后在绘制skybox之后再次调用 `DrawRenderers` 。但在此之前，请将渲染队列范围更改为 `[RenderQueueRange](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.RenderQueueRange.html).transparent` 。同时将排序条件更改为#2，然后再次设置图形设置的排序。反转透明对象的绘制顺序。

```
context.DrawSkybox(camera);

		sortingSettings.criteria = SortingCriteria.CommonTransparent;
		drawingSettings.sortingSettings = sortingSettings;
		filteringSettings.renderQueueRange = RenderQueueRange.transparent;

		context.DrawRenderers(
			cullingResults, ref drawingSettings, ref filteringSettings
		);
```

<video src="" control></video>

  

![](<images/1686828530596.png>)

Opaque, then skybox, then transparent.  
不透明，然后是天空盒子，然后是透明的。

## Editor Rendering 编辑器渲染

Our RP correctly draws unlit objects, but there are a few things that we can do to improve the experience of working with it in the Unity editor.  
我们的RP正确地绘制了未照明的对象，但我们可以做一些事情来改善在Unity编辑器中使用它的体验。

### Drawing Legacy Shaders 绘制传统着色器

Because our pipeline only supports unlit shaders passes, objects that use different passes are not rendered, making them invisible. While this is correct, it hides the fact that some objects in the scene use the wrong shader. So let's render them anyway, but separately.  
因为我们的管道只支持未照明的着色器过程，所以使用不同过程的对象不会被渲染，从而使它们不可见。虽然这是正确的，但它隐藏了场景中某些对象使用错误着色器的事实。所以，让我们无论如何都渲染它们，但要分别渲染。

If someone were to start with a default Unity project and later switch to our RP then they might have objects with the wrong shader in their scenes. To cover all Unity's default shaders we have to use shaders tag IDs for the _Always_, _ForwardBase_, _PrepassBase_, _Vertex_, _VertexLMRGBM_, and _VertexLM_ passes. Keep track of these in a static array.  
如果有人从默认的Unity项目开始，然后切换到我们的RP，那么他们的场景中可能有使用错误着色器的对象。为了覆盖所有Unity的默认着色器，我们必须为、、和过程使用着色器标记ID。在一个静态数组中跟踪这些。

```
static ShaderTagId[] legacyShaderTagIds = {
		new ShaderTagId("Always"),
		new ShaderTagId("ForwardBase"),
		new ShaderTagId("PrepassBase"),
		new ShaderTagId("Vertex"),
		new ShaderTagId("VertexLMRGBM"),
		new ShaderTagId("VertexLM")
	};
```

Draw all unsupported shaders in a separate method after the visible geometry, starting with just the first pass. As these are invalid passes the results will be wrong anyway so we don't care about the other settings. We can get default filtering settings via the `[FilteringSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.FilteringSettings.html).defaultValue` property.  
在可见几何体之后以单独的方法绘制所有不受支持的着色器，仅从第一个过程开始。由于这些都是无效的过程，结果无论如何都是错误的，所以我们不在乎其他设置。我们可以通过 `[FilteringSettings](http://docs.unity3d.com/Documentation/ScriptReference/Rendering.FilteringSettings.html).defaultValue` 属性获取默认筛选设置。

```
public void Render (ScriptableRenderContext context, Camera camera) {
		…

		Setup();
		DrawVisibleGeometry();
		DrawUnsupportedShaders();
		Submit();
	}

	…

	void DrawUnsupportedShaders () {
		var drawingSettings = new DrawingSettings(
			legacyShaderTagIds[0], new SortingSettings(camera)
		);
		var filteringSettings = FilteringSettings.defaultValue;
		context.DrawRenderers(
			cullingResults, ref drawingSettings, ref filteringSettings
		);
	}
```

We can draw multiple passes by invoking `SetShaderPassName` on the drawing settings with a draw order index and tag as arguments. Do this for all passes in the array, starting at the second as we already set the first pass when constructing the drawing settings.  
我们可以通过调用绘图设置上的 `SetShaderPassName` ，使用绘图顺序索引和标记作为参数来绘制多个过程。对阵列中的所有过程执行此操作，从第二个过程开始，因为我们在构建图形设置时已经设置了第一个过程。

```
var drawingSettings = new DrawingSettings(
			legacyShaderTagIds[0], new SortingSettings(camera)
		);
		for (int i = 1; i < legacyShaderTagIds.Length; i++) {
			drawingSettings.SetShaderPassName(i, legacyShaderTagIds[i]);
		}
```

![](<images/1686828531164.png>)

Standard shader renders black.  
标准着色器渲染为黑色。

Objects rendered with the standard shader show up, but they're now solid black because our RP hasn't set up the required shader properties for them.  
使用标准着色器渲染的对象会显示出来，但它们现在是纯黑色的，因为我们的RP尚未为它们设置所需的着色器属性。

### Error Material 错误材料

To clearly indicate which objects use unsupported shaders we'll draw them with Unity's error shader. Construct a new material with that shader as an argument, which we can find by invoking `[Shader](http://docs.unity3d.com/Documentation/ScriptReference/Shader.html).Find` with the _Hidden/InternalErrorShader_ string as an argument. Cache the material via a static field so we won't create a new one each frame. Then assign it to the `overrideMaterial` property of the drawing settings.  
为了清楚地指示哪些对象使用不受支持的着色器，我们将使用Unity的错误着色器绘制它们。以该着色器为参数构建一个新材质，我们可以通过以字符串为参数调用 `[Shader](http://docs.unity3d.com/Documentation/ScriptReference/Shader.html).Find` 来找到它。通过静态场缓存材质，这样我们就不会在每帧中创建一个新的材质。然后将其指定给图形设置的 `overrideMaterial` 特性。

```
static Material errorMaterial;

	…

	void DrawUnsupportedShaders () {
		if (errorMaterial == null) {
			errorMaterial =
				new Material(Shader.Find("Hidden/InternalErrorShader"));
		}
		var drawingSettings = new DrawingSettings(
			legacyShaderTagIds[0], new SortingSettings(camera)
		) {
			overrideMaterial = errorMaterial
		};
		…
	}
```

![](<images/1686828531757.png>)

Rendered with magenta error shader.  
使用品红色错误着色器渲染。

Now all invalid objects are visible and obviously wrong.  
现在，所有无效的对象都是可见的，而且显然是错误的。

### Partial Class 分部类别

Drawing invalid objects is useful for development but is not meant for released apps. So let's put all editor-only code for `**CameraRenderer**` in a separate partial class file. Begin by duplicating the original _CameraRenderer_ script asset and renaming it to _CameraRenderer.Editor_.  
绘制无效对象对开发很有用，但不适用于已发布的应用程序。因此，让我们将 `**CameraRenderer**` 的所有编辑器专用代码放在一个单独的分部类文件中。首先复制原始脚本资源并将其重命名为。

![](<images/1686828532328.png>)

One class, two script assets.  
一个类，两个脚本资源。

Then turn the original `**CameraRenderer**` into a partial class and remove the tag array, error material, and `DrawUnsupportedShaders` method from it.  
然后将原始 `**CameraRenderer**` 转换为分部类，并从中删除标记数组、错误材料和 `DrawUnsupportedShaders` 方法。

```
public partial class CameraRenderer { … }
```

Clean the other partial class file so it only contains what we removed from the other.  
清理另一个分部类文件，使其仅包含我们从另一个中删除的内容。

```
using UnityEngine;
using UnityEngine.Rendering;

partial class CameraRenderer {

	static ShaderTagId[] legacyShaderTagIds = {	… };

	static Material errorMaterial;

	void DrawUnsupportedShaders () { … }
}
```

The content of the editor part only needs to exist in the editor, so make it conditional on _UNITY_EDITOR_.  
编辑器部分的内容只需要存在于编辑器中，因此使其成为条件。

```
partial class CameraRenderer {

#if UNITY_EDITOR

	static ShaderTagId[] legacyShaderTagIds = { … }
	};

	static Material errorMaterial;

	void DrawUnsupportedShaders () { … }

#endif
}
```

However, making a build will fail at this point, because the other part always contains the invocation of `DrawUnsupportedShaders`, which now only exists while in the editor. To solve this we make that method partial as well. We do that by always declaring the method signature with `**partial**` in front of it, similar to an abstract method declaration. We can do that in any part of the class definition, so let's put it in the editor part. The full method declaration must be marked with `**partial**` as well.  
然而，此时构建将失败，因为另一部分始终包含 `DrawUnsupportedShaders` 的调用，而 `DrawUnsupportedShaders` 现在只存在于编辑器中。为了解决这个问题，我们也使该方法具有部分性。我们通过总是在方法签名前面声明 `**partial**` 来实现这一点，类似于抽象方法声明。我们可以在类定义的任何部分做到这一点，所以让我们把它放在编辑器部分。完整的方法声明也必须用 `**partial**` 标记。

```
partial void DrawUnsupportedShaders ();

#if UNITY_EDITOR

	…

	partial void DrawUnsupportedShaders () { … }

#endif
```

Compilation for a build now succeeds. The compiler will strip out the invocation of all partial methods that didn't end up with a full declaration.  
编译生成现在成功。编译器将去掉所有未以完整声明结束的分部方法的调用。

### Drawing Gizmos 绘制Gizmo

Currently our RP doesn't draw gizmos, neither in the scene window nor in the game window if they are enabled there.  
目前，我们的RP不绘制小控件，无论是在场景窗口中还是在游戏窗口中，如果它们在那里启用的话。

![](<images/1686828532924.png>)

Scene without gizmos. 没有小控件的场景。

We can check whether gizmos should be drawn by invoking `UnityEditor.[Handles](http://docs.unity3d.com/Documentation/ScriptReference/Handles.html).ShouldRenderGizmos`. If so, we have to invoke `DrawGizmos` on the context with the camera as an argument, plus a second argument to indicate which gizmo subset should be drawn. There are two subsets, for before and after image effects. As we don't support image effects at this point we'll invoke both. Do this in a new editor-only `DrawGizmos` method.  
我们可以通过调用 `UnityEditor.[Handles](http://docs.unity3d.com/Documentation/ScriptReference/Handles.html).ShouldRenderGizmos` 来检查是否应该绘制小控件。如果是这样，我们必须在上下文中调用 `DrawGizmos` ，将摄影机作为参数，再加上第二个参数来指示应该绘制哪个gizmo子集。有两个子集，用于前后图像效果。由于我们目前不支持图像效果，我们将同时调用这两种效果。在新的仅限编辑器的#2方法中执行此操作。

```
using UnityEditor;
using UnityEngine;
using UnityEngine.Rendering;

partial class CameraRenderer {
	
	partial void DrawGizmos ();

	partial void DrawUnsupportedShaders ();

#if UNITY_EDITOR

	…

	partial void DrawGizmos () {
		if (Handles.ShouldRenderGizmos()) {
			context.DrawGizmos(camera, GizmoSubset.PreImageEffects);
			context.DrawGizmos(camera, GizmoSubset.PostImageEffects);
		}
	}

	partial void DrawUnsupportedShaders () { … }

#endif
}
```

The gizmos should be drawn after everything else.  
小控件应该在绘制完所有其他内容之后绘制。

```
public void Render (ScriptableRenderContext context, Camera camera) {
		…

		Setup();
		DrawVisibleGeometry();
		DrawUnsupportedShaders();
		DrawGizmos();
		Submit();
	}
```

![](<images/1686828533567.png>)

Scene with gizmos. 带有小控件的场景。

### Drawing Unity UI 绘图Unity UI

Another thing that requires our attention is Unity's in-game user interface. For example, create a simple UI by adding a button via _GameObject / UI / Button_. It will show up in the game window, but not the scene window.  
另一件需要我们注意的事情是Unity的游戏内用户界面。例如，通过添加一个按钮来创建一个简单的UI。它将显示在游戏窗口中，但不会显示在场景窗口中。

![](<images/1686828534226.png>)

UI button in game window.  
游戏窗口中的UI按钮。

The frame debugger shows us that the UI is rendered separately, not by our RP.  
框架调试器向我们表明，UI是单独呈现的，而不是由我们的RP呈现的。

![](<images/1686828534759.png>)

UI in frame debugger.  
框架调试器中的UI。

At least, that's the case when the _Render Mode_ of the canvas component is set to _Screen Space - Overlay_, which is the default. Changing it to _Screen Space - Camera_ and using the main camera as its _Render Camera_ will make it part of the transparent geometry.  
至少，当画布组件的of设置为时，情况就是这样，这是默认值。将其更改为并使用主摄影机将使其成为透明几何体的一部分。

![](<images/1686828535375.png>)

Screen-space-camera UI in frame debugger.  
帧调试器中的屏幕空间相机UI。

The UI always uses the _World Space_ mode when it gets rendered in the scene window, which is why it usually ends up very large. But while we can edit the UI via the scene window it doesn't get drawn.  
UI在场景窗口中渲染时始终使用该模式，这就是为什么它通常会非常大。但是，虽然我们可以通过场景窗口编辑用户界面，但它不会被绘制出来。

![](<images/1686828535990.png>)

UI invisible in scene window.  
UI在场景窗口中不可见。

We have to explicitly add the UI to the world geometry when rendering for the scene window, by invoking `[ScriptableRenderContext](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.ScriptableRenderContext.html).EmitWorldGeometryForSceneView` with the camera as an argument. Do this in a new editor-only `PrepareForSceneWindow` method. We're rendering with the scene camera when its `cameraType` property is equal to `[CameraType](http://docs.unity3d.com/Documentation/ScriptReference/CameraType.html).[SceneView](http://docs.unity3d.com/Documentation/ScriptReference/SceneView.html)`.  
在为场景窗口进行渲染时，我们必须通过以相机为参数调用 `[ScriptableRenderContext](http://docs.unity3d.com/Documentation/ScriptReference/Experimental.Rendering.ScriptableRenderContext.html).EmitWorldGeometryForSceneView` ，将UI显式添加到世界几何体中。在新的仅限编辑器的 `PrepareForSceneWindow` 方法中执行此操作。当场景摄影机的#2属性等于#3时，我们使用场景摄影机进行渲染。

```
partial void PrepareForSceneWindow ();

#if UNITY_EDITOR

	…

	partial void PrepareForSceneWindow () {
		if (camera.cameraType == CameraType.SceneView) {
			ScriptableRenderContext.EmitWorldGeometryForSceneView(camera);
		}
	}
```

As that might add geometry to the scene it has to be done before culling.  
因为这可能会将几何体添加到场景中，所以必须在剔除之前完成。

```
PrepareForSceneWindow();
		if (!Cull()) {
			return;
		}
```

![](<images/1686828536590.png>)

UI visible in scene window.  
UI在场景窗口中可见。

## Multiple Cameras 多个摄像头

It is possible to have more that one active camera in the scene. If so, we have to make sure that they work together.  
场景中可以有多个活动摄影机。如果是这样的话，我们必须确保他们共同努力。

### Two Cameras 两个摄像头

Each camera has a _Depth_ value, which is −1 for the default main camera. They get rendered in increasing order of depth. To see this, duplicate the _Main Camera_, rename it to _Secondary Camera_, and set its _Depth_ to 0. It's also a good idea to give it another tag, as _MainCamera_ is supposed to be used by only a single camera.  
每个摄影机都有一个值，对于默认主摄影机，该值为−1。它们按深度的递增顺序进行渲染。若要查看此信息，请复制，将其重命名为，并将其设置为0。给它另一个标签也是个好主意，因为它应该只由一台相机使用。

![](<images/1686828537209.png>)

Both cameras grouped in a single sample scope.  
两个摄影机都分组在一个采样范围内。

The scene now gets rendered twice. The resulting image is still the same because the render target gets cleared in between. The frame debugger shows this, but because adjacent sample scopes with the same name get merged we end up with a single _Render Camera_ scope.  
场景现在渲染两次。生成的图像仍然相同，因为渲染目标会在两者之间清除。框架调试器显示了这一点，但由于具有相同名称的相邻示例作用域被合并，我们最终只能得到一个作用域。

It's clearer if each camera gets its own scope. To make that possible, add an editor-only `PrepareBuffer` method that makes the buffer's name equal to the camera's.  
如果每个相机都有自己的范围，就会更清晰。为了实现这一点，添加一个仅限编辑器的 `PrepareBuffer` 方法，使缓冲区的名称与相机的名称相等。

```
partial void PrepareBuffer ();

#if UNITY_EDITOR

	…
	
	partial void PrepareBuffer () {
		buffer.name = camera.name;
	}

#endif
```

Invoke it before we prepare the scene window.  
在准备场景窗口之前调用它。

```
PrepareBuffer();
		PrepareForSceneWindow();
```

![](<images/1686828537773.png>)

Separate samples per camera.  
每个摄像头单独取样。

### Dealing with Changing Buffer Names  
处理更改缓冲区名称

Although the frame debugger now shows a separate sample hierarchy per camera, when we enter play mode Unity's console will get filled with messages warning us that _BeginSample_ and _EndSample_ counts must match. It gets confused because we're using different names for the samples and their buffer. Besides that, we also end up allocating memory each time we access the camera's `name` property, so we don't want to do that in builds.  
尽管帧调试器现在为每个相机显示单独的采样层次结构，但当我们进入播放模式时，Unity的控制台将充满警告我们和计数必须匹配的消息。这会让人感到困惑，因为我们对样本及其缓冲区使用了不同的名称。除此之外，每次访问相机的 `name` 属性时，我们都会分配内存，所以我们不想在构建中这样做。

To tackle both issues we'll add a `SampleName` string property. If we're in the editor we set it in `PrepareBuffer` along with the buffer's name, otherwise it's simply a constant alias for the _Render Camera_ string.  
为了解决这两个问题，我们将添加 `SampleName` 字符串属性。如果我们在编辑器中，我们将其与缓冲区的名称一起设置在 `PrepareBuffer` 中，否则它只是字符串的一个常量别名。

```
#if UNITY_EDITOR

	…

	string SampleName { get; set; }
	
	…
	
	partial void PrepareBuffer () {
		buffer.name = SampleName = camera.name;
	}

#else

	const string SampleName = bufferName;

#endif
```

Use `SampleName` for the sample in `Setup` and `Submit`.  
使用 `SampleName` 作为 `Setup` 和 `Submit` 中的样本。

```
void Setup () {
		context.SetupCameraProperties(camera);
		buffer.ClearRenderTarget(true, true, Color.clear);
		buffer.BeginSample(SampleName);
		ExecuteBuffer();
	}

	void Submit () {
		buffer.EndSample(SampleName);
		ExecuteBuffer();
		context.Submit();
	}
```

We can see the difference by checking the profiler—opened via _Window / Analysis / Profiler_—and playing in the editor first. Switch to _Hierarchy_ mode and sort by the _GC Alloc_ column. You'll see an entry for two invocations of _GC.Alloc_, allocating 100 bytes in total, which is causes by the retrieval of the camera names. Further down you'll see those names show up as samples: _Main Camera_ and _Secondary Camera_.  
我们可以通过检查通过-打开的探查器并首先在编辑器中播放来查看差异。切换到模式并按列排序。您将看到的两个调用的条目，总共分配了100个字节，这是由于检索相机名称造成的。再往下看，您会看到这些名称显示为示例：和。

![](<images/1686828538334.png>)

Profiler with separate samples and 100B allocations.  
具有单独样本和100B分配的探查器。

Next, make a build with _Development Build_ and _Autoconnect Profiler_ enabled. Play the build and make sure that the profiler is connected and recording. In this case we don't get the 100 bytes of allocation and we get the single _Render Camera_ sample instead.  
接下来，使用并启用生成。播放生成并确保探查器已连接并正在录制。在这种情况下，我们没有得到100字节的分配，而是得到单个样本。

![](<images/1686828538875.png>)

Profiling build. 正在分析生成。

We can make it clear that we're allocating memory only in the editor and not in builds by wrapping the camera name retrieval in a profiler sample named _Editor Only_. In this case we need to invoke `[Profiler](http://docs.unity3d.com/Documentation/ScriptReference/Profiling.Profiler.html).BeginSample` and `[Profiler](http://docs.unity3d.com/Documentation/ScriptReference/Profiling.Profiler.html).EndSample` from the `UnityEngine.Profiling` namespace. Only `BeginSample` needs to be passed the name.  
我们可以通过将相机名称检索封装在名为的探查器示例中来明确我们只在编辑器中分配内存，而不是在构建中分配内存。在这种情况下，我们需要从 `UnityEngine.Profiling` 命名空间调用 `[Profiler](http://docs.unity3d.com/Documentation/ScriptReference/Profiling.Profiler.html).BeginSample` 和 `[Profiler](http://docs.unity3d.com/Documentation/ScriptReference/Profiling.Profiler.html).EndSample` 。只有 `BeginSample` 需要传递名称。

```
using UnityEditor;
using UnityEngine;
using UnityEngine.Profiling;
using UnityEngine.Rendering;

partial class CameraRenderer {

	…
	
#if UNITY_EDITOR

	…

	partial void PrepareBuffer () {
		Profiler.BeginSample("Editor Only");
		buffer.name = SampleName = camera.name;
		Profiler.EndSample();
	}

#else

	string SampleName => bufferName;

#endif
}
```

![](<images/1686828539490.png>)

Editor-only allocations made obvious.  
编辑专用的分配是显而易见的。

### Layers 图层

Cameras can also be configured to only see things on certain layers. This is done by adjusting their _Culling Mask_. To see this in action let's move all objects that use the standard shader to the _Ignore Raycast_ layer.  
相机也可以被配置为只看到某些层上的东西。这是通过调整它们来完成的。若要看到这一点，让我们将使用标准着色器的所有对象移动到该层。

![](<images/1686828540061.png>)

Layer switched to _Ignore Raycast_. 层切换到。

Exclude that layer from the culling mask of _Main Camera_.  
从的剔除遮罩中排除该层。

![](<images/1686828540608.png>)

Culling the _Ignore Raycast_ layer. 剔除图层。

And make it the only layer seen by _Secondary Camera_.  
并使其成为唯一被看到的层。

![](<images/1686828541225.png>)

Culling everything but the _Ignore Raycast_ layer.  
剔除除图层以外的所有内容。

Because _Secondary Camera_ renders last we end up seeing only the invalid objects.  
因为渲染在最后，所以我们最终只能看到无效的对象。

![](<images/1686828541792.png>)

Only _Ignore Raycast_ layer visible in game window.  
只有在游戏窗口中可见的层。

### Clear Flags 清除标志

We can combine the results of both cameras by adjusting the clear flags of the second one that gets rendered. They're defined by a `[CameraClearFlags](http://docs.unity3d.com/Documentation/ScriptReference/CameraClearFlags.html)` enum which we can retrieve via the camera's `clearFlags` property. Do this in `Setup` before clearing.  
我们可以通过调整渲染的第二个摄影机的清除标志来组合两个摄影机的结果。它们由 `[CameraClearFlags](http://docs.unity3d.com/Documentation/ScriptReference/CameraClearFlags.html)` 枚举定义，我们可以通过相机的 `clearFlags` 属性检索该枚举。在清除之前在#2中执行此操作。

```
void Setup () {
		context.SetupCameraProperties(camera);
		CameraClearFlags flags = camera.clearFlags;
		buffer.ClearRenderTarget(true, true, Color.clear);
		buffer.BeginSample(SampleName);
		ExecuteBuffer();
	}
```

The `[CameraClearFlags](http://docs.unity3d.com/Documentation/ScriptReference/CameraClearFlags.html)` enum defines four values. From 1 to 4 they are `Skybox`, `Color`, `Depth`, and `Nothing`. These aren't actually independent flag values but represent a decreasing amount of clearing. The depth buffer has to be cleared in all cases except the last one, so when the flags value is at most `Depth`.  
`[CameraClearFlags](http://docs.unity3d.com/Documentation/ScriptReference/CameraClearFlags.html)` 枚举定义了四个值。从1到4，它们分别是#1、#2、#3和#4。这些实际上并不是独立的标志值，但表示清除量在减少。除最后一种情况外，所有情况下都必须清除深度缓冲区，因此当标志值最多为 `Depth` 时。

```
buffer.ClearRenderTarget(
			flags <= CameraClearFlags.Depth, true, Color.clear
		);
```

We only really need to clear the color buffer when flags are set to `Color`, because in the case of `Skybox` we end up replacing all previous color data anyway.  
我们只需要在标志设置为 `Color` 时清除颜色缓冲区，因为在 `Skybox` 的情况下，我们最终会替换所有以前的颜色数据。

```
buffer.ClearRenderTarget(
			flags <= CameraClearFlags.Depth,
			flags == CameraClearFlags.Color,
			Color.clear
		);
```

And if we're clearing to a solid color we have to use the camera's background color. But because we're rendering in linear color space we have to convert that color to linear space, so we end up needing `camera.backgroundColor.linear`. In all other cases the color doesn't matter, so we can suffice with `[Color](http://docs.unity3d.com/Documentation/ScriptReference/Color.html).clear`.  
如果我们要清除为纯色，我们必须使用相机的背景色。但因为我们在线性颜色空间中渲染，我们必须将该颜色转换为线性空间，所以我们最终需要 `camera.backgroundColor.linear` 。在所有其他情况下，颜色并不重要，所以我们可以用 `[Color](http://docs.unity3d.com/Documentation/ScriptReference/Color.html).clear` 就足够了。

```
buffer.ClearRenderTarget(
			flags <= CameraClearFlags.Depth,
			flags == CameraClearFlags.Color,
			flags == CameraClearFlags.Color ?
				camera.backgroundColor.linear : Color.clear
		);
```

Because _Main Camera_ is the first to render, its _Clear Flags_ should be set to either `Skybox` or `Color`. When the frame debugger is enabled we always begin with a clear buffer, but this is not guaranteed in general.  
因为是第一个渲染的，所以应将其设置为 `Skybox` 或 `Color` 。当启用帧调试器时，我们总是从清除缓冲区开始，但这通常不能保证。

The clear flags of _Secondary Camera_ determines how the rendering of both cameras gets combined. In the case of skybox or color the previous results get completely replaced. When only depth is cleared _Secondary Camera_ renders as normal except that it doesn't draw a skybox, so the previous results show up as the background. When nothing gets cleared the depth buffer is retained, so unlit objects end up occluding invalid objects as if they were drawn by the same camera. However, transparent objects drawn by the previous camera have no depth information, so are drawn over, just like the skybox did earlier.  
的清除标志决定了如何组合两个摄影机的渲染。在skybox或颜色的情况下，以前的结果将被完全替换。如果只清除了深度，则渲染为正常，除非它没有绘制天空框，因此以前的结果显示为背景。当没有清除任何内容时，深度缓冲区将被保留，因此未照明的对象最终会遮挡无效对象，就好像它们是由同一相机绘制的一样。然而，上一个相机绘制的透明对象没有深度信息，因此会被绘制，就像之前的skybox一样。

![](<images/1686828542355.png>)

  

![](<images/1686828542919.png>)

  

![](<images/1686828543524.png>)

Clear color, depth-only, and nothing.  
颜色清晰，只有深度，什么都没有。

By adjusting the camera's _Viewport Rect_ it is also possible to reduce the rendered area to only a fraction of the entire render target. The rest of the render target remains unaffected. In this case clearing happens with the _Hidden/InternalClear_ shader. The stencil buffer is used to limit rendering to the viewport area.  
通过调整摄影机的，还可以将渲染区域减少到整个渲染目标的一小部分。渲染目标的其余部分不受影响。在这种情况下，着色器会进行清除。模具缓冲区用于限制渲染到视口区域。

![](<images/1686828544121.png>)

Reduced viewport of secondary camera, clearing color.  
辅助摄影机的缩小视口，清除颜色。

Note that rendering more than one camera per frame means culling, setup, sorting, etc. has to be done multiple times as well. Using one camera per unique point of view is typically the most efficient approach.  
请注意，每帧渲染一个以上的摄影机意味着还必须多次进行剔除、设置、排序等操作。每个独特的视点使用一台摄像机通常是最有效的方法。

The next tutorial is [Draw Calls](https://catlikecoding.com/unity/tutorials/custom-srp/draw-calls/).  
下一个教程是绘制调用。

[license 许可证](https://catlikecoding.com/unity/tutorials/license/) [repository](https://bitbucket.org/catlikecodingunitytutorials/custom-srp-01-custom-render-pipeline/) [PDF](https://catlikecoding.com/unity/tutorials/custom-srp/custom-render-pipeline/Custom-Render-Pipeline.pdf)