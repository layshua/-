## 前言

在 UE5.1 中，官方将原有的输入系统正式替换为增强输入系统。初次使用这个系统时难免会觉得有些复杂，本文档的写作目的就是简单介绍一下增强输入系统的构成以及如何使用它的基础功能。

## EnhancedInput 的优点

如果接触过 EnhancedInput 应该了解这套框架在蓝图中配置一个输入事件的流程要比原本的输入系统复杂。 相应的这些流程也为构建一些较为复杂的输入机制带来了优势：

*   角色能够在不同的情景下切换不同的输入变化（例如切换近战 / 远程 / 不同载具等）
    
*   系统中包含一些默认实现的较为复杂的输入行为（例如按住蓄力 / 双击等）
    
*   将按键输入、轴输入等更多的输入事件都集成在了一个 Input Event 中
    

## EnhancedInput 的基础构成

*   InputAction(IA) 输入动作
    

![](https://img-blog.csdnimg.cn/859bcb41a2d74cc0b188c67453a04129.png)

 老版本的输入设置：

![](https://img-blog.csdnimg.cn/5929fc11679145b5b1ec0811bb5aa8b2.png)

EnhancedInput 中的 IA 相当于老版本中的输入事件的名称，不过它相比老版本多了一些能够自定义设置的参数。

*   Value Type
    

![](https://img-blog.csdnimg.cn/3e4ea0317b5a432f89d4d1021f43ff87.png)

IA 的 value type 共有四种，代表了这一输入事件的输出值类型。

         一般来说 Bool 值代表以前的动作映射，判断按键是否按下。

         float，Vector 等值可以用于判断鼠标，手柄摇杆的输入，可以根据输出值判断手柄或者鼠标的位置。

         Vector3D 可以用于运动传感器的输入，得到传感器的三维坐标。

*   InputMappingContext(IMC) 输入映射上下文环境
    

![](https://img-blog.csdnimg.cn/341c1a3e0181468aa4a4d4ad7c1188b7.png)

 老版本中的按键 mapping

![](https://img-blog.csdnimg.cn/4c2edd1652f34e98ba753c001286e2a2.png)

IMC 的结构类似老版本的的 Mapping，作用也是将按键与输入事件绑定。区别在于老版本是通过输入命名来自定义输入事件，而 EnhancedInput 中是在 IMC 中将按键与用户创建的 InputAction 进行绑定。

*   InputModifier（IM）输入[修改器](https://so.csdn.net/so/search?q=%E4%BF%AE%E6%94%B9%E5%99%A8&spm=1001.2101.3001.7020)
    

![](https://img-blog.csdnimg.cn/a2208c613e7346eeabda95d408303830.png)

顾名思义，IM 就是用于对输入动作的输出值进行限制 / 修改的规则。比如在老版本中判断前进后退的输入是通过修改 Scale 的正负，在 EnhancedInput 中就是通过修改 IM 来实现的。

![](https://img-blog.csdnimg.cn/43a6718a2acf4eb19d365f221288e868.png)

 老版本中通过修改按键 Scale 值来决定输出值的正负

![](https://img-blog.csdnimg.cn/09864d06987b4a949fef14de3f2c167c.png)

*   InputTrigger(IT) 输入[触发器](https://so.csdn.net/so/search?q=%E8%A7%A6%E5%8F%91%E5%99%A8&spm=1001.2101.3001.7020)
    

![](https://img-blog.csdnimg.cn/4d7a49b87cd348419e87ae4895e3e1a6.png)

IT 是用来判断当前输入事件什么时候应该触发的触发规则，比如想做一个蓄力事件：蓄力时间为 1 秒，蓄力一秒后仅触发一次事件，就可以按照下图的方式来进行配置：

![](https://img-blog.csdnimg.cn/a4cc76f535df409fb1f71b8a3603c5f5.png)

## 注意事项

上图中可以看到 IMC 和 IA 中都可以对键位配置 Modifiers 和 Triggers, 这两者有什么区别呢？

IA 中的参数相当于全局，IMC 中的参数只是针对于当前情景，并且两者之间的修改是会叠加的。

如果输入事件在 IA 和 IMC 中的 Modifier 都配置了 negative，则输出值会负负得正变为正值。

## 输入事件绑定

EnhancedInput 系统中绑定输入事件不像之前的系统只需要在项目设置中绑定按键，现在需要创建两种文件进行绑定：InputAction(IA) 和 InputMappingContext(IMC)

![](https://img-blog.csdnimg.cn/359cf652b367484b970a3f0a1e220f68.png)

在创建好需要用到的 IA 后，在 IMC 中对各个 IA 事件进行具体按键的绑定：

![](https://img-blog.csdnimg.cn/11c48489257c4fb29e336ba7e369a019.png)

![](https://img-blog.csdnimg.cn/6975e246164a42b297b0bc3ccf1298a6.png)

根据不同的情景可以创建多个 IMC

![](https://img-blog.csdnimg.cn/27b092fba0f24053b03c349b9d4d90ab.png)

然后在蓝图中倒入对应情景的 IMC 进行导入

![](https://img-blog.csdnimg.cn/423407be97e04c648c37233cae0c739a.png)

导入后就能像之前输入系统一样获取对应的输入事件来 IA 进行逻辑的绑定

![](https://img-blog.csdnimg.cn/87031d06ece746a3b5ed3c124d5dee5d.png)

## 案例：蓄力事件

下图为蓄力事件的配置与在蓝图中[事件触发](https://so.csdn.net/so/search?q=%E4%BA%8B%E4%BB%B6%E8%A7%A6%E5%8F%91&spm=1001.2101.3001.7020)时 Event 不同输出口的触发条件

![](https://img-blog.csdnimg.cn/c4902b3560e74a31a8b3824484dd7c18.png)

![](https://img-blog.csdnimg.cn/f1085713f9e94ef190b62a3f611d5bc3.png)

## 参考文档 / 视频

[https://zhuanlan.zhihu.com/p/470949422](https://zhuanlan.zhihu.com/p/470949422 "https://zhuanlan.zhihu.com/p/470949422")

[https://docs.unrealengine.com/5.1/zh-CN/enhanced-input-in-unreal-engine/](https://docs.unrealengine.com/5.1/zh-CN/enhanced-input-in-unreal-engine/ "https://docs.unrealengine.com/5.1/zh-CN/enhanced-input-in-unreal-engine/")

[https://www.bilibili.com/video/BV14r4y1r7nz/?spm_id_from=333.337.search-card.all.click&vd_source=09360098519d857e12c5a4e268163c45](https://www.bilibili.com/video/BV14r4y1r7nz/?spm_id_from=333.337.search-card.all.click&vd_source=09360098519d857e12c5a4e268163c45 "https://www.bilibili.com/video/BV14r4y1r7nz/?spm_id_from=333.337.search-card.all.click&vd_source=09360098519d857e12c5a4e268163c45")

UE5 引擎自带 VR 模版