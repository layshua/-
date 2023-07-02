[Command Buffers In Unity - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/news/102132)
[unity的CommandBuffer - 灰信网（软件开发博客聚合） (freesion.com)](https://www.freesion.com/article/7053897071/)

# URP 渲染概念

![[Pasted image 20230702213954.png]]
**Render Pass**：Frame Debug 中看到的每一个过程都有可能提交多个 DrawCall, 这个过程称作 Render Pass。注意这个 Render pass 和 unity shader 中的 pass 有区别. 一个 Render Pass 中可能包含了 unity shader 中多种不同 lightmode 的 pass。
**Render Pass 的作用就是将输入的顶点信息渲染到一个 Render Target 上。 
RenderPass 可以将 GPU 可识别的指令 CommandBuffer 提交到 GPU 中来添加渲染指令并在不同的渲染阶段执行。**


**渲染目标 Render Target**：[[DX12理论 #5 渲染到纹理技术]]是现代图形处理单元 (GPU)的一个特性，它允许 3D 场景被渲染到一个中间内存缓冲区，或渲染目标纹理 (Render-Target-Texture、RTT)，而不是帧缓冲区或后台缓冲区。可以通过像素着色器操纵此 RTT，以便在显示最终图像之前将其他效果应用于最终图像。
Render Target 可以设置为系统内置的纹理 (比如深度、相机纹理)，也可以是临时申请的纹理,，所有的绘制都是不可逆转的，故每一帧绘制前都会清理掉前一帧的绘制结果.

**多目标渲染 MRT**：多目标渲染（MRT: Multiple Render Target)容许应用程序一次渲染到多个缓冲区。利用 MRT 技术，片段着色器可以输出多个颜色，可以用于保存 RGBA 颜色、法线、深度信息或者纹理坐标，每个颜色连接一个颜色缓冲区。在延迟渲染或者前向渲染中, 对应的 Gbuffer 与 Dbuffer 作为 RenderTarget 同时有多个输出。![[Pasted image 20230702213259.png|450]]

**RenderPipeline**：URP 中 `RenderPipelineAsset` 文件会根据用户在其面板上定义的各种信息创建 `RenderPipeline`。而 `RenderPipeline` 则决定了所有 `RenderPass` 的数据来源, 渲染目标, 以及 `RenderPass` 之间的顺序，即所有的渲染流程会在 `RenderPipeline.render()` 下运行。`render()` 的最后会执行 `RenderContex.Submit()`, 将需要传输到 GPU 的数据、设置、以及在 GPU 中需要执行的指令 CommandBuffer 提交给 GPU.

**Renderer**：因为不同的对象的绘制使用到了不同的 RenderPass 进行绘制, 为了更方便的切换不同的管线。URP 在 RenderPipeline 与 RenderPass 之间抽象出了一层 Renderer，**用于管理 RenderPass 集合。**
![[Pasted image 20230702213440.png]]

**Renderer Data**：即 UR Data, 可以对 Render Pipeline 的功能进行一定控制
![[Pasted image 20230702213623.png|450]]

**Render Feature**：是一种 Asset，用于向 URP 渲染器添加额外的 Render pass 并配置其行为。在 Renderer 中定义了基本的 RenderPass 渲染流程，但是这个流程是固定, 如果我们想在不透明物体前增加一个绘制物体法线外扩描边的 RenderPass，或者是想将某一个时刻的渲染结果给保留到一张 RenderTexture 上供后续渲染使用，就需要增加 RenderPass。
自定义的 RenderPass 是通过 RenderFeature 进行设置与添加到渲染队列的，主要的作用是在面板中可视化, 以便用户设置与调整所添加的 RenderPass 的信息。


**Command Buffer**： 是用来存储渲染命令的缓冲区。保存着渲染命令列表，如 `SetRendertarget`, `Drawmesh` 等等，可以设置为在摄像机渲染期间的不同点执行. 
这么说可能会比较生硬, 举几个应用吧.
- 绘制网格: 通过指令在当前缓冲区绘制 mesh.
- 添加一个 pass: 比如希望自定义的 pass 能够在指定阶段渲染到相机或指定贴图, 如在不透明渲染前进行法线外扩描边, 渲染毛发 shader 等需要多 pass 效果;
- 申请渲染 DBuffer 这种可以存储多张贴图供后续渲染使用的 RenderTexture (比如贴花)
- 后处理: 需要获取当前渲染的结果并将其通过后处理 shader 渲染到一张临时材质上, 再将临时材质返回到原管线, 或者直接作为结果
![[Pasted image 20230702220808.png]]
处于跨平台的需要，Unity 对这些底层 API 做了一层封装，产生了 CommandBuffer
# URP 渲染流程
![[Pasted image 20230702204334.png]]

![[Pasted image 20230702204838.png]]

