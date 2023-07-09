![[c1dab9753b93cf462f447f82c529fda8_MD5.jpg]]

## 阴影图片

*   **性能消耗：最低。**
*   **纯手绘：**一般就一个椭圆底，放角色脚下。
*   **跳跃**：阴影底会有 Scale 变化。

## 平面阴影（顶点投射阴影）

*   **性能消耗：中等。（多 Pass 渲染）**
*   **假阴影**：不需要接受阴影的物体。（可以自己调阴影颜色）
*   **原理**：[使用顶点投射的方法制作实时阴影 - 喵喵 Mya 的文章 - 知乎](https://zhuanlan.zhihu.com/p/31504088)、 [【Unity Shader】平面阴影（Planar Shadow） - 红果 Ygg 的文章 - 知乎](https://zhuanlan.zhihu.com/p/266174847) （URP）

## 阴影贴图（ShadowMap） - Unity 内置方案

*   **性能消耗：高。（多 Pass 渲染）**
*   **实时阴影**：需要一个接受阴影的物体（接受阴影的物体需要自己采样阴影）
*   **边缘锯齿严重**（阴影贴图分辨率低，在对阴影贴图采样时，多个不同的顶点对同一个像素采样，导致生成锯齿）。
*   **阴影级联**：生成和使用多张（不同精细度的）阴影贴图，解决锯齿问题。（4 级级联用的 3 张阴影图）
*   **没有办法做类似平面阴影的阴影衰减**：接受阴影物体通过阴影贴图弄得阴影不知道原物体位置。
*   **原理**：[方向阴影（Directional Shadow）](https://edu.uwa4d.com/lesson-detail/282/1311/0?isPreview=true)，阴影贴图本质是一张深度图。

![[057b9b537dd2af3b072832b71a43550d_MD5.jpg]]

## 光照贴图（烘焙阴影）

*   **使用光照贴图的好处**：烘焙阴影在最大距离外也不会被剔除。
*   **混合阴影**：光照贴图和阴影贴图是可以混用的。（最大距离内用实时阴影，最大距离外用烘焙阴影）
*   **光照探针**：运动物体从静止物体处接受阴影投射。
*   **原理**：[烘焙阴影（ShadowMask）](https://edu.uwa4d.com/lesson-detail/282/1313/0?isPreview=0)

## 阴影体积（ShadowVolume）

*   **没有阴影锯齿问题**。
*   效率没有 ShadowMap 高。（需要几何计算）
*   更详细的介绍：[阴影锥 (Shadow Volume)](https://blog.csdn.net/zjull/article/details/11819923?spm=1001.2101.3001.6650.5&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-5-11819923-blog-18219431.t0_edu_mix&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-5-11819923-blog-18219431.t0_edu_mix&utm_relevant_index=6)

## 补充资料

*   [点光和聚光灯阴影](https://edu.uwa4d.com/lesson-detail/282/1350/0?isPreview=0)
*   [屏幕空间深度阴影贴图（Screen Space Deep Shadow Maps） - 凯奥斯的文章 - 知乎](https://zhuanlan.zhihu.com/p/79062045)

Deep Shadow Maps 主要应用在头发和烟雾渲染中，对于增强二者的体积感起到了很关键的作用。本文结合 Deep Shadow Maps 和 Screen Space Shadow Maps 技术，在屏幕空间结算阴影，主要用在头发渲染上。

### 投影器

*   实际上就是一个接受阴影的 shader。
*   相关使用细节：[UNITY 如何实现投影阴影效果](https://www.freesion.com/article/324239269/)