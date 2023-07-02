[Command Buffers In Unity - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/news/102132)
[unity的CommandBuffer - 灰信网（软件开发博客聚合） (freesion.com)](https://www.freesion.com/article/7053897071/)

# URP 渲染流程
![[Pasted image 20230702204334.png]]

![[Pasted image 20230702204748.png]]


![[Pasted image 20230702204838.png]]

**渲染目标 Render Target**：[[DX12理论 #5 渲染到纹理技术]]是现代图形处理单元 (GPU)的一个特性，它允许 3D 场景被渲染到一个中间内存缓冲区，或渲染目标纹理 (Render-Target-Texture、RTT)，而不是帧缓冲区或后台缓冲区。可以通过像素着色器操纵此 RTT，以便在显示最终图像之前将其他效果应用于最终图像。
Render Target 可以设置为系统内置的纹理 (比如深度、相机纹理)，也可以是临时申请的纹理,，所有的绘制都是不可逆转的，故每一帧绘制前都会清理掉前一帧的绘制结果.

**多目标渲染 MRT**：多目标渲染（MRT: Multiple Render Target)容许应用程序一次渲染到多个缓冲区。利用 MRT 技术，片段着色器可以输出多个颜色，可以用于保存 RGBA 颜色、法线、深度信息或者纹理坐标，每个颜色连接一个颜色缓冲区。在延迟渲染或者前向渲染中, 对应的 Gbuffer 与 Dbuffer 作为 RenderTarget 同时有多个输出。![[Pasted image 20230702213259.png|450]]


**RenderPipeline**：URP 中 `RenderPipelineAsset` 文件会根据用户在其面板上定义的各种信息创建 `RenderPipeline`。而 `RenderPipeline` 则决定了所有 `RenderPass` 的数据来源, 渲染目标, 以及 `RenderPass` 之间的顺序，即所有的渲染流程会在 `RenderPipeline.render()` 下运行。`render()` 的最后会执行 `RenderContex.Submit()`, 将需要传输到 GPU 的数据、设置、以及在 GPU 中需要执行的指令 CommandBuffer 提交给 GPU.

**Renderer**：因为不同的对象的绘制使用到了不同的 RenderPass 进行绘制, 为了更方便的切换不同的管线。URP 在 RenderPipeline 与 RenderPass 之间抽象出了一层 Renderer，**用于管理 RenderPass 集合。**
![[Pasted image 20230702213440.png]]

**Renderer Data**：即 UR Data, 可以对 Render Pipeline 的功能进行一定控制
![[Pasted image 20230702213623.png|450]]

**Render Feature**：是一种 Asset，用于向 URP 渲染器添加额外的渲染过程并配置其行为。在 Renderer 中定义了基本的 RenderPass 渲染流程，但是这个流程是固定, 如果我们想在不透明物体前增加一个绘制物体法线外扩描边的 RenderPass,
或者是想将某一个时刻的渲染结果给保留到一张 RenderTexture 上供后续渲染使用. 就需要增加 RenderPass.
自定义的 RenderPass 是通过 RenderFeature 进行设置与添加到渲染队列的. 主要的作用是在面板中可视化, 以便用户设置与调整所添加的 RenderPass 的信息


![[Pasted image 20230702210043.png]]

![[Pasted image 20230702210113.png]]

![[Pasted image 20230702210129.png]]