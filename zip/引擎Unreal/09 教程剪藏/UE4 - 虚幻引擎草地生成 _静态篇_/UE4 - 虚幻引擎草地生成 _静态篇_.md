## 效果预览

![](1677316202165.png)

![](1677316202201.png)

## 一、刷草仔初试

刷草可以直接用 Foliage 的植被笔刷来刷上去。

也可以通过使用 "LandscapeGrass Type" 来加载进 land layer 材质中，后续刷材质在地形上植被也跟随出现就更加的方便，但是遇到特殊情况还是需要笔刷单独刷控制更精细。

![](1677316202491.png)

![](1677316202617.png)

![](1677316202685.png)

![](1677316202718.png)

![](1677316202849.png)

## 二、刷草开始

先使用上面的方法让植被把地形铺满，调试材质。这次直接用了[风筝男孩工程](https://link.zhihu.com/?target=https%3A//www.unrealengine.com/id/login/api/login%3Fclient_id%3D43e2dea89b054198a703f6199bee6d5b%26redirect_uri%3Dhttps%253A%252F%252Fwww.unrealengine.com%252Fmarketplace%252Fproduct%252Fkite-demo%253Flang%253Dzh-CN%2526sessionInvalidated%253Dtrue%26prompt%3Dpass_through)里的材质套用了还有 [Lucen](https://link.zhihu.com/?target=https%3A//www.youtube.com/channel/UCMCjGkbqKVkjJN2sVWocz0w)，主要分享下草地材质应该有那些部分的配备与制作。

草的 mesh 也是工程中的那个 SM_Field grass

*   **地面随机颜色分布**

地面使用以 **world position xy 两个轴为 UV** 当纹理坐标采样贴图来随机植被颜色，做出更为自然的草地颜色。

![](1677316202928.png)

*   **风力模拟 <Normal>**

根据 camera 制作一个 depth 通道来混合风力吹动草地和静止的状态。

添加到 custom uv1 通道

![](1677316202982.png)

草地后面会使用 position offset，法线会失去作用，因此关闭 tangent space normal，后面都自己赋值新的法线。

![](1677316203057.png)

草的发线一般都设置为全部向上 <0,0,1> 这样高光更加有序，这里就**把 <0,0,1> 从 Local 空间转到 world 空间**，使用 Foliage 工具分布植被时对齐物体表面法线的位移信息来纠正法线方向，并存储到 custom uv2，3 中。

![](1677316203093.png)

下面制作一个法线贴图位移动画，链接到基础色上查看一下。嗯，效果有是有了，但是平铺在世界上的，没有基于物体的法线。

![](1677316203162.png)

现在就使用之前存在自定义 uv 中的法线方向，再叉乘一下，**定义出一个物体空间坐标矩阵**。使用 texture normal 的向量和矩阵做一个**线性变换** [[1]](#ref_1) 就把平铺的法线匹配到了物体的方向上。

![](1677316203248.png)

![](1677316203355.png)

*   **风力模拟 <vertex>**

一般情况可以直接使用 ue4 自带的 grass wind，但是后来的版本去掉了 wind size 的可调节端口，这里再函数中加入了两个 input 来控制风吹动的频率。

![](1677316203460.png)

![](1677316203713.png)

*   **人物交互**

这里直接用角色的位置与草地位置算一个推出去的向量就好啦。

![](1677316203769.png)

最终效果的话只能说一般吧，后续会单独就交互，草地燃烧来写一个文章。

最近终于忙完了，又可以继续做点小东西啦。

对了！菜狗求职，哪个游戏公司收了我吧，优先考虑成都，看了我的文章觉得我可以的求勾搭。:(

![](1677316203833.png)

草地资源链接：[https://pan.baidu.com/s/1wVVO_km-5yX-TzPp22rtDQ](https://link.zhihu.com/?target=https%3A//pan.baidu.com/s/1wVVO_km-5yX-TzPp22rtDQ)  
提取码：zo5c

## 参考

1.  [^](#ref_1_0) 线性变换，向量乘矩阵 [https://charlesliuyx.github.io/2017/10/06/%E3%80%90%E7%9B%B4%E8%A7%82%E8%AF%A6%E8%A7%A3%E3%80%91%E7%BA%BF%E6%80%A7%E4%BB%A3%E6%95%B0%E7%9A%84%E6%9C%AC%E8%B4%A8/](https://charlesliuyx.github.io/2017/10/06/%E3%80%90%E7%9B%B4%E8%A7%82%E8%AF%A6%E8%A7%A3%E3%80%91%E7%BA%BF%E6%80%A7%E4%BB%A3%E6%95%B0%E7%9A%84%E6%9C%AC%E8%B4%A8/)