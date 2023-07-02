

## SRP 底层

#### 一. Scriptable Culling

我们先思考一个问题：调用`ScriptableRenderContex.Cull()`的时候底层会发生什么

##### (1) Shadow Culling

先介绍阴影的 Culling 过程。首先，Unity 会遍历场景中实时的 Light，如果勾选了 CastShadow，那么这个 Light 会遍历场景中的物体，如果物体也勾选了 CastShadow，那么就会产生阴影。

底层：**每个 Cast Shadow 的 Real-time Light 会分配一个 Shadow 的 Job，每一个单独的 Job 完成对 Cast Shadow 物体的 Cull**

##### (2) Dynamic Objects Culling

我们再来看 Unity 对动态物体的 Culling 过程。Unity 为场景中所有挂载了 Renderer Component 的物体维护了一个 Renderer 的 List，**IndexList 存储了当前所有可见的 Renderer 在 List 中的下标**。

【生成 IndexList】

List 中的 Renderer 会分组，每一组会分配一个 **Cull Job**，每个 Cull Job 独自执行完 Cull 之后，所有 Cull Job 的结果合并，生成 IndexList。**这个过程不需要引入锁**。

【ExtractRenderNodeQueue】

Unity 为了保证渲染的速度，引入了 RenderNode。对于所有 IndexList 中对应的 Renderer 对象，**把 Renderer 里所有引用类型的数据展开，拷贝到一个 struct 中，这个 struct 就是 RenderNode**。**RenderNode 在内存中是连续的**，**RenderNode 组成一个队列，称为 RenderNodeQueue**，**方便做多线程渲染**。

（注：因为 RenderNodeQueue 是由 IndexList 生成的，所以 RenderNode 对应的 Renderer 是场景中可见的）

#### 二. Scriptable Draw

现在，我们知道了动态物体 Culling 的结果就是 RenderNodeQueue。Unity 接下来就要进行绘制了，先介绍一些绘制过程中的函数。

```
CommandBuffer.Blit();
CommandBuffer.DrawMesh();
ScriptableRenderContext.DrawRenderers();
ScriptableRenderContext.DrawShadows();
......
```

这些函数有什么关系呢？我们写着往下看

##### ExecuteCommandBuffer & DrawRenderers

有 4 个跟 Command 相关的 List：

```
dynamic_array<ShadowDrawingSettings> m_DrawShadowCommands;
dynamic_array<DrawRenderersCommand> m_DrawRenderersCommands;
dynamic_array<RenderingCommandBuffer*> m_COmmandBuffers;
dynamic_array<Command> m_Commands;
```

1.  调用 DrawRenderers 时，会产生 1 个 DrawRenderersCommand 添加到对应 List 中，同时也会产生一个 Commands 添加到对应 List 中，用来记录这个 Command 的类型及其在 List 中的下标。
    
2.  调用 ExecuteCommandBuffer 时，会产生 1 个 CommandBuffer 添加到对应 List 中，同上。
    

**m_Commands 存储了所有 command 队列中的每一个 command 的类型和下标**

#### 三. Scriptable Render Loop

在插入 Command 之后，继续调用`ScriptableRenderContext.Submit()`就会执行 Render Loop

##### (1) PrepareDrawRenderersCommand()

我们先看 Renderer，material，pass 之间的关系

一个 Renderer 可以有多个材质 (有很多材质插槽)，一个材质可以有多个 Pass

在 Culling 的过程中，我们已经得到了 RenderNodeQueue。然后我们要通过 sort 决定 Draw 的顺序

【执行过程】

1.  **通过 RenderNode 找到 Materials，通过每一个 Material 找到 Pass，通过每一个 Pass 生成一个`ScriptableLoopObjectData`，这个`SctiptableLoopObjectData`中有一个标识，标识它是不是 SRP batcher 兼容的。**
    
2.  对所有的`ScriptableLoopObjectData`进行排序
    

##### (2) Scriptable Render Loop

【CommandBuffer】

遍历提交的 m_Commands，找到所有的 command 的类型和下标，然后 Execute

【ScriptableLoopObjectData】

遍历`ScriptableLoopObjectData`，找到兼容 SRP batcher 的，然后扔到 SRP batcher 渲染器进行渲染，剩余的会交给传统的 Draw 渲染路径进行渲染。

##### (5) SRP batcher

**SRP Batcher 就是**

1.  把调用 draw call 前，一大堆 CPU 的设置工作给一口气处理了，增加了效率。
    
2.  把材质的属性数据直接永久放入到显卡的 CBUFFER 里，那只要数据不变，CPU 就可以
    

不需要把这些数据重新做设置工作。节省了 CPU 调用，增加了效率。

3.  用专用的代码将引擎的属性（比如 objects transform）直接放入到 GPU 显存，这个专用的代码是不是更快更强呢，

官方是这样么说的，用的词语是 quickly，就是快。

4.  SRP Batcher 并没有减少 drawcalls，而仅仅是提高了效率。相当于一个人减肥了，减去了多余的脂肪和水分，但是器官结构啥的一个没少。总之就是有用。

**SRP Batcher 的工作原理**

SRP Batcher 诞生的原因：

在一个 Drawcall 被一个新的 material 使用的时候，有很多工作要做。

所以如果场景有越多的 materials，就会有越多的 CPU 必须使用去设置 GPU 数据。

传统的方法是减少 DrawCalls 的数量去优化 CPU 渲染性能。

因为 Unity 必须在调用 drawcall 前设置很多东西。

并且真正的 CPU 消耗来自那些设置工作，而不是 GPU drawcall 本身。

**Drawcall 只是一些 Unity 向 GPU command buffer 发送的 bytes。**

## 补充

1.  **SRP batcher 是工作在 CPU 层面的，它做的事情就是减少 SetPass Call**。Unity 在很久以前就把 Draw Call 和 SetPass Call 做了区分：Draw Call 本身就是调用一个图形的 API，它本身的开销并不耗。而开销高是高在我们做切换渲染状态的时候要提前为显卡准备非常多的数据，也就是 SetPass Call 的工作，准备这些数据往往来说是开销比较高的。评判标准：不管是默认管线还是 SRP，SetPass Call 最好都不要超过 150，Draw Call 的话可以高一些。
    
2.  **Vaulkan**：SRP batcher 在 CPU 层面的开销，比较可以关注的一个点是 android 上的 vaulkan，它已经越来越成熟，有不少项目在立项阶段把 vaulkan 作为首选的 API。其实使用了 vaulkan 的话，会有一个明显的发现就是，**vaulkan 在 CPU 上的开销要远远小于 OpenGL**。所以推荐！！！
    
3.  **SRP batcher 和 GPU Instance 用的技术是差不多**，如果大家是想绘制单一的物体（像草这样的），推荐大家使用 GPU Instance。但是如果想做正常的场景渲染，比如说场景里的 material 多于 5 个，SRP batcher 的速度要比我们手动做 GPU Instance 要划算的多的。
    

