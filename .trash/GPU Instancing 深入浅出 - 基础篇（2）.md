本章节主要是让大家对 Unity3d 中的 GPU-Instancing 的原理进行一个整体的了解。

跟着文章由浅入深，相信只要看完整个系列的文章就能轻松搞定在大规模的程序开发中使用 GPU Instancing 的问题，本文主要来源一个小众《元宇宙》项目。

对应 Demo 用例下载：链接: [https://pan.baidu.com/s/1qoiiHGRbe_skt6Nercub8Q?pwd=9hf0](https://pan.baidu.com/s/1qoiiHGRbe_skt6Nercub8Q?pwd=9hf0) 提取码: 9hf0  
对应 Demo 用例下载：链接： https://pan.baidu.com/s/1qoiiHGRbe_skt6Nercub8Q?pwd=9hf0 提取码： 9hf0

GitHub: 下载地址

[https://github.com/xinxun/TestGPUInstancing](https://github.com/xinxun/TestGPUInstancing)

## GPU Instanceing 相关章节传送门

[梅川依福：GPU Instancing 深入浅出 - 基础篇（1）](https://zhuanlan.zhihu.com/p/523702434)

[梅川依福：GPU Instancing 深入浅出 - 基础篇（2）](https://zhuanlan.zhihu.com/p/523765931)

[梅川依福：GPU Instancing 深入浅出 - 基础篇（3）](https://zhuanlan.zhihu.com/p/523924945)

[梅川依福：GPU Instancing 深入浅出 - 中级篇（1）](https://zhuanlan.zhihu.com/p/524195324)

[梅川依福：GPU Instancing 深入浅出 - 中级篇（2）](https://zhuanlan.zhihu.com/p/524285662)

## 一、什么是 Draw Call

### 1、官方手册解读

老样子先看相应 [Optimizing draw calls - Unity 手册](https://docs.unity.cn/cn/current/Manual/optimizing-draw-calls.html)

![[aca47d199d3ff143e0c3bb4d1b1f0747_MD5.jpg]]

优化 Draw calls 优化 Draw calls

要在屏幕上绘制几何图形，unity 会调用图形 API 进行处理。一个 Draw call 会告诉图形 API 需要绘制什么以及使用什么方式进行绘制。每个 Draw call 包含了图形 API 所需要的纹理，阴影以及缓冲区的绘制信息。大量的 Draw call 会消耗大量的资源，但 Draw call 的准备通阶段要比 Draw call 本身消耗更多的资源。

## 二、一次 DrawCall 在做什么

以上是为自己的翻译，我们可以通过以下的 Gif 来感受一下 GPU 与 CPU 的通信，简单的来说 Draw call 其实是 CPU 与 GPU 的通信方式，他们通过 CommandBuffer 作为通信的 “信道”，其实每一次 GPU 与 CPU 的通信并没有我们想得的那么简单叫一个 Draw Call 其中的过程有很多步骤，所以 Unity3D 又叫作 Batch（批次）。

如下图所示：

![[1473df82a90f4da4a4e2ed7d59b7be0f_MD5.gif]]

GPU 渲染速度远远高于 CPU 提交命令的速度，如果一帧中间 DrawCall 数量太多，CPU 就会在设置渲染状态 - 提交 drawcall 上花费大量时间，造成性能问题，这里的性能问题其实是 GPU 在等待 CPU 的处理。

![[0d437877aa1505267e7a098e77ac4150_MD5.png]]

### 1、举个例子（测试用例 1）

我们可以通过 Unity3D 的一个案例来说明

想要用例下载见百度网盘：

链接: [https://pan.baidu.com/s/1qoiiHGRbe_skt6Nercub8Q?pwd=9hf0](https://pan.baidu.com/s/1qoiiHGRbe_skt6Nercub8Q?pwd=9hf0) 提取码: 9hf0  
链接: https://pan.baidu.com/s/1qoiiHGRbe_skt6Nercub8Q?pwd=9hf0 提取码: 9hf0

创建 Unity 工程，在场景中创建对象，并使用如下代码

```
using UnityEngine;
public class CreateCube : MonoBehaviour
{
    [SerializeField]
    private GameObject _instanceGo;//需要实例化对象
    [SerializeField]
    private int _instanceCount;//需要实例化个数
    [SerializeField]
    private bool _bRandPos = false;//是否随机的显示对象
    // Start is called before the first frame update
    void Start()
    {
        for (int i = 0; i < _instanceCount; i++)
        {
            Vector3 pos = new Vector3(i * 1.5f, 0, 0);
            GameObject pGO = GameObject.Instantiate<GameObject>(_instanceGo);
            pGO.transform.SetParent(gameObject.transform);
            if(_bRandPos)
            {
                pGO.transform.localPosition = Random.insideUnitSphere * 10.0f;
            }
            else
            {
                pGO.transform.localPosition = pos;
            }          
        }
    }
}
```

创建一个 NormalCubeCreate 的空节点挂上以上代码

并创建一个 Cube 把 Cube 制作成 prefab 后拖放到 NormalCubeCreate 中的 Instance Go 属性上中

![[07775b79c582a749ba06f90569fcfaed_MD5.jpg]]

运行后的效果

![[a557d5273a8a1426a806827c10dda6ea_MD5.jpg]]

### 2、用例分析

通过 Statistics 我们可以看到 Batches 为 12 这里的 Batches 为 Draw Call 的次数

![[377556e6383f0c02a2563d85d916bf95_MD5.jpg]]

打开 Frame Debug: Frame Debug 可以显示每一帧渲染时 CPU 与 GPU 的一些绘制信息

![[4838286226a41ff7f0bd5d07f4e22647_MD5.jpg]]

相应 的 Frame Debug 下的数据显示

![[35f9b802b26f570dab19b59994c90c75_MD5.jpg]]

通过以上的测试我们在 RenderForward.RenderLoopJob 中看到 10 个 Draw Mesh NormalCube(Clone) 的提交，相当于在同一帧中 CPU 与 GPU 提交了 10 次的绘制（Draw call），每画一个 Cube 就有一次 DrawCall 的调用。

那有没有更优的解决方案呢，如一次就把这十个对象都绘制上，当然有的，但是是有前提的那就是我们说的材质和网格需要相同（但不完全如此总归是有此约束）。

## 三、更优的 Draw Call 处理

### 1、优化方案

![[0dfcfcc657453aea0ab51fea328c8f3d_MD5.gif]]

通过以上的 GIF 我们可以这么理解，我们可以把需要绘制的相同内容同时放到 CommandBuffer 中再通知 GPU 进行绘制，这样可以有效的优化每绘制一个对象就调用一个转态转换让 GPU 进行显示效率要高。

![[3fa0efbc4be5c65064b9675c38788442_MD5.png]]

### 2、举个例子

我们可以通过 Unity3D 的一个案例来说明

创建 Unity 工程，在场景中创建对象，并使用测试用例 1 的代码

重新创建一个 Prefab 命名 InstanceCube, 在所使用的 Material 中选择 Enable GPU Instancing

![[1447fb5a92792400170255c67e407944_MD5.jpg]]

创建一个 GameObject 重名称为 InstanceCubeCreate 挂上 CreateCube 组件，把 InstanceCube

![[d9e5c32633fab054fc24bf6dad30709d_MD5.jpg]]

运行效果如下

![[c5ba71d0555da2889772a66b7612ad5e_MD5.jpg]]

### 2、用例分析

从 Statistics 中我们可以看到 Batches 变成了 3，而 RenderForward.RenderLoop.Job 为 1，我们仅仅只是在 Cube 的材质中打开了 Enable GPU Instancing 就达到了我们想要的效果（Batch 从 12 变成了 3）。其中 Frame Debug 中显示的 Draw Mesh(Instanced) 和官方文档中提到的

![[541859cc0a89493cdf98c6001286b5c2_MD5.png]]

表现完全一至看来是 GPU Instancing 起到了正向的作用。

## 四、总结

经过以上的使用我们初小掌握了 GPU Instancing 的使用，在 Material 中只要把开 “Enale GPU Instancing” 就有如上的运行效果，原来 GPU Instancing 如此的简单。似乎到此咱们就结束了相应的课程。

![[e7026373b15023409867f814202fea55_MD5.jpg]]

咱只是入了个门，路漫漫兮......

在下面的章节中我会给大家介绍，GPU Instancing 中的一些限制，这些限制我们要如何绕过去？当然方法很多。