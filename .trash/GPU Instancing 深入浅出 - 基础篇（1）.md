本章节主要是让大家对 Unity3d 中的 GPU-Instancing 是干什么用的有一个整体的了解，以及通过对官方手册的解读全方面的了解对 GPU Instancing。

深入浅出的 GPU Instancing 文章：会分成三个部分进行，

基础篇：了解 GPU instancing 的基础使用，小白可从此开始

中级篇：写一个自定义的 GPU instancing Shader，并让他变的更有趣，有一定经验的开发可以从这里开始。

高级篇：在 GPU Instancing 的 instance 中使用 GPU 动画， 以及 GPU 草场的应用，元宇宙小伙伴可以从这里开始。

跟着文章由浅入深，相信只要看完整个系列的文章就能轻松搞定在大规模的程序开发中使用 GPU Instancing 的问题，本文主要来源一个小众《元宇宙》项目。

## GPU Instanceing 相关章节传送门

[梅川依福：GPU Instancing 深入浅出 - 基础篇（1）](https://zhuanlan.zhihu.com/p/523702434)

[梅川依福：GPU Instancing 深入浅出 - 基础篇（2）](https://zhuanlan.zhihu.com/p/523765931)

[梅川依福：GPU Instancing 深入浅出 - 基础篇（3）](https://zhuanlan.zhihu.com/p/523924945)

[梅川依福：GPU Instancing 深入浅出 - 中级篇（1）](https://zhuanlan.zhihu.com/p/524195324)

[梅川依福：GPU Instancing 深入浅出 - 中级篇（2）](https://zhuanlan.zhihu.com/p/524285662)

## **一、GPU Instancing 定义**

### **1、对官网手册的解读**

[https://docs.unity.cn/cn/current/Manual/GPUInstancing.html](https://docs.unity.cn/cn/current/Manual/GPUInstancing.html)

![](<images/1686897011443.png>)

翻译：GPU Instancing 是一种 Draw call 的优化方案，使用一个 Draw call 就能渲染具有多个相同材质的网格对象。而这些网格的每个 copy 称为一个实例。此技术在一个场景中对于需要绘制多个相同对象来说是一个行之有效办法，例如树木或灌木丛的绘制。

GPU Instancing 在同一个 Draw call 中渲染完全相同的网格。可以通过添加变量来减少重复的外观，每个实例可以具有不同的属性，例如颜色或缩放。在 Frame Debugger 中如有 Draw calls 显示多个实例时会显示 “Darw Mesh(Instanced)”。

![](<images/1686897011498.png>)

以上是 GPU Instancing 的官方说明，翻译时加入了自己的理解，不到位之处尽请谅解。

### 2、苍白文章的感知

从官方手册的解读中，我们似乎可以这么理解：即只有使用相同网格和相同材质的物体渲染时，才可以使用 GPU Instancing 技术， 这样看来，我们使用 GPU Instancing 似乎只能渲染出来一堆网格和材质都一样的物体？那除了告诉用户 “看！我能渲染出这么多一样的东西，厉不厉害？！” 之外，似乎毫无乐趣可言。事实如此么？

那具体是什么效果还是用一些用例来看看吧。

## 二、GPU Instancing 可达到的效果

### 1、相同 Mesh 个性化显示

![](<images/1686897011541.png>)

### 2、多个 Mesh Instancing 效果

![](<images/1686897011577.png>)

### 3、大型效果场景的显示

![](<images/1686897011611.png>)

### 4、GPU Instancing 的高级动作应用

![](<images/1686897011654.png>)

### 5、GPU 在植被中的应用

![](<images/1686897011709.png>)

看完以上效果不知道有没有想试试手的感觉，GPU Instancing 技术的应用其实很广，所以不要小看官网中那些苍白的文字描述，到 AssetStore 中看看。

说了这么多展示了这么多，不防继续来看官方手册关于 GPU Instancing 的一些约束，

## **三、需要注意的事**

### **1、在 SRP Batcher 时如何使用 GPU Instancing 技术**

其实不是不支持而是 SRP 有 SRP Batcher，以下官网说明，官网偷偷的告诉我们如果非要在 SRP 下使用 GPU instance 的话那可以使用 Graphics.DrawMeshInstanced，其实是直接 draw a mesh on screen。

![](<images/1686897011788.png>)

### 2、SkinnedMeshRenderers 不支持 GPU Instancing 技术

![](<images/1686897011839.png>)

其实本质资源是 GPU Instancing 仅支持 MeshRender，不直接支持 SkinnedMeshRender 的 Instance，想想如果支持了那不是所有蒙皮动画都可以 Instance，正常来说对于 SkinnedMeshRender 其实有办法支持，以下是使用 GPU 的顶点动画的方案进行的支持。

[https://github.com/Unity-Technologies/Animation-Instancing](https://github.com/Unity-Technologies/Animation-Instancing)

当然支持 Animation-Instancing 是有一定牺牲。

### **3、Lighting 对 GPU Instance 是支持的**

说了很多不可以，同时 Unity 抛砖引玉的说到了 Lighting 对 GPU instanceing 对象的不离不弃

![](<images/1686897011895.png>)

从以上的信息可以看出 GPU Instancing 出的对象可以受光照的影响，看起来不错，只要设置一下就好了。

## **四、总结**

看完以上的介绍，我们对 GPU instancing 有一个大体的了解，从苍白的文档中看不出什么效果，后面的章节我们会由浅入深的对 GPU Instancing 从基础篇一直讲到高级的应用。