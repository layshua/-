# 创建项目

![[50c50e11cb60bc7715aa1a4df6ddc867_MD5.png]]

game空项目，选择CPP

![[4afb80eb55670faa68ae8c2dcce756cf_MD5.png]]

项目目录下新建Plugins,把我们的MultiplayerSessions放里面

![[d19e30bf31e9caac154efe5690c3dc18_MD5.png]]

打开steam插件

![[37734f38a761d31a5bbae9858a10ba0f_MD5.png]]

将官网这一段复制到config中

![[be8e2235851103a8bd39e9d3113d04c2_MD5.png]]

更改另一个config文件

![[bd904f9825285eee0813c3769eb8b82d_MD5.png]]

删除三个文件，用rider不用删

![[64f42a9c7454475cd761bd3bdf55e3e8_MD5.png]]

创建菜单地图，配置一下

![[52dea1d489b560dfdd4501dcb4e61b69_MD5.png]]

项目设置中增加两个地图配置

![[51e2161a717eede1df18ee1ce4a336f2_MD5.png]]

添加蓝图第三人称模板

![[5d6a014600edf180c6e84b0d3b85b1b1_MD5.png]]

将lobby的gamemode改成刚刚下的第三人称默认gameMode，因为第三人称自带同步代码

## 添加免费资产

![[f3083e9a2b9faad96ebed5a3e7bcce27_MD5.png]]

选择这个包，是因为他不支持UE5，我们可以趁机学习怎么在UE5中加入以前版本的资产（现在这个包支持5.0）

![[e2ce7df9c715b4bc0905cd1a317b76dd_MD5.png]]

加入一个专门用于转化资产的空项目

![[cfc08bc1084defdcef1a2d745dcdf5f2_MD5.png]]

选择迁移

![[17d119179394b0e3b94f605fddab1a57_MD5.png]]

![[a31f44b664ef4eefd4fc45facd43abe5_MD5.png]]

再来一些项目

![[342787f502f49acc3a5deaa28513eda1_MD5.png]]

mixamo中下载一些动画

## 动画重定向

![[8f07fd22e8e18940db03f992709e044b_MD5.png]]

新建两个IK Rig

![[289647906a29f4eb313b797e33b27129_MD5.png]]

将右手所有骨骼定义为链

![[766d1853629b18f318a4e0f7452b8375_MD5.png]]

重复这个工作，将所有部分定义完

![[bd884e6752ba760e291a9d373d1640d4_MD5.png]]

新建IK重定向器，这里我们需要小白人的动画，选择小白人

![[c3c74209d6de6a2dcc9a849d309dab9e_MD5.png]]

修改姿势

![[be1e689f1a7f6f5618aa485244ca9bae_MD5.png]]

选择hips之类的作为retarget root，然后导出即可

![[4764c584888e1aece4f6cb7652797ed8_MD5.png]]

![[4f339718935cf2000cf0a139812c3984_MD5.png]]

将需要的资源迁移过去

## 整理项目

![[d15958f9bcda22458668719741de1c50_MD5.png]]

把文件夹整理好

![[a1d70c3059e195bfb27a77b815100a45_MD5.png]]

添加摇臂和摄像机，配置属性

![[18fff04e13e1a63d2070f1895b10a774_MD5.png]]

翻译一些蓝图常说的话，让角色动起来

## Rider快捷键

[【Rider for Unreal Engine】常用功能总结_Tanzq-blog的博客-CSDN博客](https://blog.csdn.net/qq_46276931/article/details/120380827)

# C++动画蓝图

![[0743515fbfe4121ca99c446bad6ff974_MD5.png]]

继承自AnimInstance

![[443fa9578623a9133a868e4cbe7fcea0_MD5.png]]

使用meta语法访问私有变量

![[d1b019f77e69846a3b03e399aede85aa_MD5.png]]

将基础的属性tick放到C++中

![[a9028c101830fd57236e6baf928e2282_MD5.png]]

角色不随Controller旋转，但如果有移动输入，会把角色转向那个方向

# Seamless Travel⭐⭐⭐⭐⭐

![[9deb7905b2729cf29a390389a53da1be_MD5.png]]

![[e9050ba6c99e276ccea5b6b161ebeae3_MD5.png]]

服务器调用ServerTravel，所有客户端都会调用ClientTravel

![[9eee56c98ac035243e715ce4eb2758e7_MD5.png]]

gamemode只存在于服务器上

![[103120b5d6d4603d3db306598d573d02_MD5.png]]

设置转换关卡为这个空关卡，这步不太懂

# 网络同步角色⭐⭐⭐⭐⭐

![[fdd075eb3563b1833af18d9e8c493fc5_MD5.png]]

1. Authority,服务器自己控制的角色
    
2. SimulatedProxy，别的客户端控制的角色
    
3. AutonomousProxy，客户端自己控制的角色
    

![[f2d22bca3dc3ca7a3676831aef3fcf17_MD5.png]]

![[d4862bbdc39eed7d91a3247105e46b1a_MD5.png]]

组件效果

# 创建武器类

![[6e7b8b37e58d5b97b215805899d178bc_MD5.png]]

创建武器类，检查是否Authority

![[60abc6a6e8085dbd441588dad4143b26_MD5.png]]

网络同步的代码，都需要bReplicates =true

![[cd7b0f4f11be901bea50863cda3e4045_MD5.png]]

自定义武器枚举，用UMETA限定编辑器时显示的名称

![[f8b566d1febc0d0828f4274aa1283f64_MD5.png]]

只在服务器上绑定重叠事件

![[6f2fa0ee3ca897d210e4dfe2f1a264f5_MD5.png]]

这样子写，服务器重叠只有服务器可见，客户端重叠也只有服务器可见

# 变量网络复制⭐⭐⭐⭐⭐

![[9eb4128df27e665bf0b050a088703faf_MD5.png]]

给需要网络复制的变量加上宏定义

![[07ffa8ac1005d6969bb2b95bcb517c81_MD5.png]]

接着，重写虚函数，复制这个变量

![[6beaccf278282a45ac15f1bfdfa10a52_MD5.png]]

调用的时候复制变量

![[cccc67f3c98cf1702efd2caec479dc5a_MD5.png]]

书写一个inline函数

![[44ce4d09028c5bc085e2f781b0ddeeb2_MD5.png]]

这样，客户端运行时，会在服务器和所有客户端上都看见这行字

![[4bb45406e448567792be216646eed08e_MD5.png]]

只在owner上复制这个变量，这样做的话，当前客户端和服务器都能看到这行文字（Pickup）

![[db1dd04a3b6c1f00e1453ae0168fe365_MD5.png]]

定义变量复制事件，在变量被复制时执行，这样我们只在捡到武器的客户端上看到这行字

![[c5e0648a4a41f2344c34aacbec763b06_MD5.png]]

OnsphereOverlap仅仅发生在服务器上

![[1e600025f8beb377fdc2c627bd8b7cb9_MD5.png]]

这样，走出去时就不会显示ui了

变量复制只能从服务器传到客户端，不能反向传播

# 手动设置网络延迟⭐⭐⭐⭐⭐

![[ad1e655e065025a494149b26266af69c_MD5.png]]

模拟ping,除此之外还可以模拟丢包等

# 战斗组件

![[613c2d00bb5f6ab88dc66462d9ecb898_MD5.png]]

定义战斗组件

![[98abd188a05f8cbbeacfcd838c2808ef_MD5.png]]

互为朋友，这样角色就能随意访问战斗组件的所有函数，包括private和protected,捡起武器时，要确定是否是服务器。

# 多播拾起武器Rpc(1)⭐⭐⭐⭐⭐

![[355c5d7022d2da046e1aa67cbe0a8dd3_MD5.png]]

reliable意味着会进行确认

![[80645b01042e0c29177bac1542be586f_MD5.png]]

Rider会自动加上Implementation，vs需要自己加

![[088d8e5093ce05427a93bb8a9d0466f4_MD5.png]]

如果是客户端，使用RPC的方式调用这个函数，但是会发现我们的UI没有隐藏

![[df75994cd95f3c6a87a4384d0df2cdc1_MD5.png]]

这是因为setPickupWidget没有在客户端之间广播

![[bd9438f62598079165d849699c81fb85_MD5.png]]

只有服务器上会产生overlap这种事件，一些例如meshAttachToPlayer这种操作，是自动复制的，我们不需要额外书写代码。RPC指远程程序调用，在这个例子中，我们让客户端上的装备事件在服务器上执行

# 多人游戏动画蓝图⭐⭐⭐⭐⭐

![[ea9103380b01aef0eaf4530767a0ce5f_MD5.png]]

这样是行不通的，因为EquippedWeapon并没有在客户端上复制

![[6094d37789f827f0e1940c24555c5414_MD5.png]]

这一段非常难理解，在我们有三个角色的情况下，假设我们在服务器上，操作一个客户端的角色捡起武器，这个角色的EquippedWeapon被设置上了，但这个只在服务器上发生了，服务器上这个客户端的动画蓝图会发生状态改变，播放持枪动画，但这边没有复制到对应客户端上

`shift+👆全选一行`

# 添加下蹲

![[6bde346de3b495fcb70e8d5bbee68da3_MD5.png]]

移动组件内置了Crouch（）以及对应的网络复制函数，按下左Shift，什么都不会发生

![[cd0750a47cef7d67228f3ef7c84f1a3c_MD5.png]]

因为要在移动组件中打开

# 瞄准

![[85139d02dd1a9bb7e7a9e49f7fe6a0e2_MD5.png]]

和蹲基本上一样，但这个变量没有网络同步，无法在别的客户端上看到变化。网络同步后，可以在服务器上瞄准，但无法在客户端上瞄准。

![[9f7489fd6c9e56329f6a31ecd859e7d9_MD5.png]]

需要为客户端编写RPC函数

![[c1709ce8b1907a70bdc45ea5c86dd5ff_MD5.png]]

# Aim Offset⭐⭐⭐⭐⭐

![[fe55d7dcd8b685e2057e8cdf75a4a712_MD5.png]]

手动制作AO动画

![[8eed512e153e57796b1c632aeb2a2e05_MD5.png]]

把AO动画都设置成Additive，选择meshSpace，参考CC动画

![[499f31a57482e3e03231521f27dab03a_MD5.png]]

在这个角色中，spine_01分开上下半身

![[2c2b5a2a5f947843dcff2a8d224d34af_MD5.png]]

选择spine_01

![[178d276012dbad2d006ca34ee8c2eafc_MD5.png]]

使用AO，静止不动的时候，我们希望人物的行为是：

可以朝四周看，超过一定角度之后移动脚

![[2708b5da6537014106c38698e8176606_MD5.png]]

不动时，我们需要yaw和pitch的AO，移动时，我们只需要pitch的AO。

注意这里不能用GetControlRotation,因为一台机器只有一个Controller，在联网编程时要使用GetBaseAimRotation()。

TODO:从服务器上捡起武器，Pitch值同步了，但Yaw值没同步

在客户端上捡起武器能把Pitch同步，但Yaw值不行

课程写到这里，确实是会有这个问题的

# TurnInPlace

![[3e2f746be3b18e4227afe017dd0ec28b_MD5.png]]

新建TurnInPlace枚举，在Character类里处理这个枚举值

![[ea40eadf517d76e118bdb48435cf703e_MD5.png]]

使用分层混合解决只有一个转身动画，但是需要在转身时区分瞄准和不瞄准，填spine_01即可

![[b10a4f5dd8445b0103b80b4764d4ff0a_MD5.png]]

动画飘在天上，可以直接拖下来，加一个key即可。不喜欢动画里有key可以直接导出资产

# 旋转根骨骼

![[5e133752204839e8bbc515f058210508_MD5.png]]

使用旋转根骨骼节点，稍加改动Character.cpp即可实现。

实际上就是人物胶囊体跟着controller在运动，而旋转根骨骼得到了Yaw值将旋转值移回去了，而上半身的AO依然生效，插值一定时间后将StartAimingRotation清零，停止旋转，AO参考的是StartAimingRotaion，所以表现正常。

# 倾斜跑BlendSpace

![[837ac438de472b6f9aed3aab1617d246_MD5.png]]

![[2b07a5e780fd7c44f4d09553de672e9e_MD5.png]]

制作倾斜动画

![[f4192e7fa59918bbc7564fb646b28897_MD5.png]]

自己制作Lean动画

![[2f3255d0f7d5ac138a35063c7d245dba_MD5.png]]

在持械时，我们不需要朝向旋转，我们需要始终看向前方

![[09a0631eeaf66d8abd5df060f40051e1_MD5.png]]

动画蓝图里不需要复制倾斜这种变量，是因为Actor的Rotation是引擎自动复制的，所以在各个客户端上计算出来的结果都是一样的。

![[eb0719bfab5643858aea6deae9226286_MD5.png]]

从-180到180的过渡非常奇怪，因为中间过渡了非常多中间动画，解决办法是插值，不能使用float插值，使用Rotator插值，UE会帮我们找到-180到180的最短路径，得到正确结果。

![[a8075894cec13fc2393c609a41a8463d_MD5.png]]

负值可能会打包出错，因为虚幻在网络传输时会把float变成无符号数，压缩至5Byte。

# 自制八方向移动动画

![[bae15c9e318412879a5d8077e21e4cb1_MD5.png]]

把向前跑动画根骨骼旋转45，spine01反向旋转45即可

# FABRIK IK⭐⭐⭐⭐⭐

P53 FBARIK IK

![[3cc55305e75242c084c9ee853b3a3507_MD5.png]]

在突击步枪的根上加一个socket，命名需要统一

![[1b9ac27ec77cf1b74ca8556a538fb759_MD5.png]]

计算应有位置

![[7f694234812967bf5f103da52b10a291_MD5.png]]

这种情况下End Effector是我们的右手，Solver是我们的左手，RootBone选择leftArm，这将决定我们的算法在哪停止

# 网络更新频率⭐⭐⭐⭐⭐

![[6a0fa5116ca59b1176e465493dfd7281_MD5.png]]

这两个分别是正常情况下网络同步频率和actor不活跃情况下的网络同步频率，在项目中，我们可以手动设置网络延迟，也能手动降低网络同步频率来测试网络不稳定情况下的游戏表现

![[9f37a5e5d46a96929e88c645a3d0bf9c_MD5.png]]

更改服务器tickRate

# 自制八方向动画

![[52af1ca0ad1ed964fff46114142bdb76_MD5.png]]

使用向前跑动画，只旋转spine以上即可

# 添加声音

![[62bced5ac3cc7a8f131ad1151c0fa2da_MD5.png]]

动画通知中的声音会自动复制，不需要额外操作

# 抛物子弹

![[d5092a23f6e91fa73e807f2296f18620_MD5.png]]

基本设置

# 武器开火

![[052eb659bc6edefe917f9b3e446e20ec_MD5.png]]

cpp中接受到是否按下左键的布尔值后，在CombatComponent中处理开火逻辑

![[25bc5ad34d49f8e130affe857b066b01_MD5.png]]

一个蒙太奇即可处理两个开火动画

![[b1c0c79367b160274f07b5d19a8b513e_MD5.png]]

简单选择逻辑

![[40b5fb3669e07351201e96897a171ffe_MD5.png]]

开火动画和声音并没有在网络上复制，这个需要使用MultiCastRPC解决

# 多播武器开火RPC(2)⭐⭐⭐⭐⭐

https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/Actors/RPCs/

![[231bea291655d91787c3935319157301_MD5.png]]

这种情况下，在客户端上捡起武器，只有服务器能看见开火动画和音效

这里不能使用变量复制，变量复制在变量发生变化时自动赋值，我们的自动武器开火时并不会有变量发生变化

![[2515d6d0fbe4dd9bdce9f070020545fd_MD5.png]]

使用MultiCastRPC

![[21040f83deb90c77be2e6f921496ffde_MD5.png]]

服务器和客户端上按下按键，都会触发RPC多播事件，在所有客户端上运行

# 准确发射Projectile⭐⭐⭐⭐⭐

![[70d0dbbfb58410fa2a9cc4f513c8c271_MD5.png]]

UE自带函数，可以获得屏幕中间点的方向和第一个检测到的位置，填充到两个引用传递的参数里，并且返回一个bool。

![[482151f7d3154b5c0b7d123ecaa46fd5_MD5.png]]

我们获得了屏幕中间的世界坐标和朝向，可以利用这个做射线检测，常量可以进行宏定义，减少代码中的数字量

![[726192fe81123f324aa87bb8ca0883d2_MD5.png]]

对于子弹，可以这样定义，好处是可以接收任何派生自这个类的对象

![[cbcec581b8005ef24326bda6364bb5c8_MD5.png]]

写完之后唯一的问题是，我们的射击是基于我们的viewport的，这意味着，simulate结果是错误的。我们要确保只在server上生成Projectile，然后复制到所有Client。

![[b736e933b82cf226dc97489ba766970f_MD5.png]]

只有服务器上能生成子弹，然后复制到所有客户端，将子弹也设置成网络复制的，现在客户端上也有子弹，但是因为目标没有复制到所有客户端上，开枪时子弹方向是不对的

# 网络同步正确方向⭐⭐⭐⭐⭐

![[9ab83f12ea0241e4b221903d76f050f2_MD5.png]]

使用FVector_NetQuantize进行网络传输优化，FVector_NetQuantize是FVector的子类，削减了精度，提高了网络传输速率

# 命中事件网络同步

![[f3298229ffdac4135b4fdb2afb37c30b_MD5.png]]

先声明回调函数，可以点击CollisionBox的Hit，查看需要什么参数，WorldContextObject指的是一个世界中的物体，一般传入this

![[ea8d07bf4ff734429f6511bb681e4e95_MD5.png]]

这样写，特效和声音在所属客户端上播放，我们需要在所有客户端上播放。

Destroyed（）在gameplay过程使用。

我们实现多播特效和声音的最简单方式，是在Super::Destroyed()后书写我们的播放声音和特效逻辑。

对于一个子弹，实际上我们不需要mesh，各种游戏中也没有mesh，只需要一道光柱(Particle)即可

# 弹壳自发光效果

![[71c40d11f7364d2ae69ebf9cd72e4072_MD5.png]]

按住4，创建一个Vector4D，然后保持各个通道都是0，接着把他转化为参数，叫Emissive

![[aaf9ab79480d9274f92b49e74a69ea1a_MD5.png]]

打开这个材质实例的Emissive，我们可以override这个值

![[b38ba4192451829c2e36bbc5c8fdb2c2_MD5.png]]

调成喜欢的颜色

![[08ea4e5a3675f117e340579b6d5061ce_MD5.png]]

对于Casing的生成，我们没有进行变量复制，实际上我们也不需要复制这个变量到所有客户端，只需要本地看到表现，我们需要在他触地之后的一会将它Destroyed。

# 音频编程

![[0c30dd44b4b74cbb04f0479dd1591d1b_MD5.png]]

可以创建一个sound cue,然后将所有cue随机选择

![[19d67ca610ff22fa6795ed0103956599_MD5.png]]

创建音频衰减，可以避免距离很远时仍能听见

# 自制HUD

![[812812a0e2699284784e3bdcbedda147_MD5.png]]

使用GenerateBody使得反射系统能和这个结构合作，USTRUCT使得这个结构能在蓝图中使用

BUG:只有服务器上能瞄准

最后发现蓝图里设置了 combatComponent不能同步

# 修正枪口朝向

![[1d1602d45b91c52e3c241afba7a00aff_MD5.png]]

骨骼上的SocketName不是大小写敏感的，hand_r==Hand_R。计算出右手应有朝向后，在动画蓝图中做解算

![[1efd8d97c5f41da58e4db7d6bce9177d_MD5.png]]

我们只做本地结算，不需要结算其他端上结果。

# 瞄准缩放

![[e72875cb23cad709dd9e323cd69b3c55_MD5.png]]

改变摄像机的景深和光圈值，使得我们在瞄准近处和远处物体时不会出现模糊。

# UE CPP接口⭐⭐⭐⭐⭐

![[4f04d1f54be56558ee3e8bf9ab107b74_MD5.png]]

自动创建的.h里有U开头和I开头的两个接口，在我们使用时，角色应该继承I开头的接口。

[Interfaces](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/GameplayArchitecture/Interfaces/)

![[e26e0741a861682e805d8c5ad1b7ea99_MD5.png]]

询问一个类有没有继承接口的三种方法

![[b62e8e39328aa2879e6a821a426f85b6_MD5.png]]

注意，这里询问是否继承接口，使用U开头的版本。这种情况下，接口里不需要实现任何函数，如果拥有这个接口，我们就把准星改成红色。

# 隐藏人物模型

![[f339fc887041ac612b426e02fefb9f82_MD5.png]]

判断与摄像机的距离，太近时隐藏，超过时恢复。

# 多播受击反应（RPC（3)）⭐⭐⭐⭐⭐

![[6505043ee7cb0a7c981b71e247643780_MD5.png]]

为四个受击动画做叠加处理

![[72c6109ded7f9d7b19fc4d8b76b911db_MD5.png]]

叠加处理后将这四个动画塞进同一个montage里，受击时进行选择即可。服务器可以命中客户端，客户端无法命中服务器，我们需要对这个动画进行multicastRPC。

![[0bda230a2acff7d0273d7464391abb57_MD5.png]]

选择Unreliable，这个不必重要，偶尔丢包不影响，可以提升网络表现。Hit只在服务器上发生，发生时调用MulticastRPC

![[de771dbbc3180751d1bc314b2719e531_MD5.png]]

新增一个通道，使用实际的Mesh进行命中检测，便于实现爆头等伤害数值不同。

![[8d9023a091919a0ea178330e4bdb53f5_MD5.png]]

ECC_1代表我们刚刚创造的碰撞通道，我们可以在Blaster.h中对这个内容进行替换。

# 解决SimulateProxy抖动⭐⭐⭐⭐⭐

P81_Smooth Rotation for Proxies

![[3dec18abc6bf13dbfa24b984af1cc533_MD5.png]]

这个是因为我们本地使用了Rotate Root Bone，我们在网络上并不会以本地的速度tick，所以会出现抖动问题。

![[6f6ebe56ee0bd5d8de8865f07d8ab30a_MD5.png]]

我们需要对simulateProxy进行额外处理，单独写一个函数来进行RotateRootBone。

![[10172202b8053fdf86c970e95e0468af_MD5.png]]

我们不能每帧调用这个函数，因为网络传播tick率要低于本地tick率，每帧调用实际上并不会起作用

![[81d6cc2bca04471da63c54361657097c_MD5.png]]

正确的做法是重写这个函数,角色的运动发生改变时，这个函数会执行一次。

# 全自动开火（Timer）⭐⭐⭐⭐⭐

![[1fb72c37459a982e33b937dbc9b0da3b_MD5.png]]

使用Timer进行全自动开火

# UE_GameFrameWork⭐⭐⭐⭐⭐

![[f9fb8210a18fe33c9110d38454d64560_MD5.png]]

- GameMode只存在于服务器上
    
- GameState存在于服务器和客户端上
    
- PlayerState存在于服务器和客户端上
    
- PlayerController只存在于OwingClient
    
- PAWN存在于服务器和客户端上
    
- HUD/WIDGETS只存在于OwingClient
    

![[046eeb0e6c9e9f9a234f4ec0d0413ec5_MD5.png]]

GameMode定义了默认的类，游戏规则（玩家死亡和重生），比赛状态，热身时间，比赛时间。ServerOnly。

GameState定义了比赛的状态，得分最高的玩家，领先的队伍，队伍得分，PlayerStates数组。

PlayerState定义了玩家得分，击败数，弹药数，属于哪个队伍

![[d0ce06bd6ebf09863571866ce861463f_MD5.png]]

PlayerController可以轻松访问HUD

![[6cfae5eb44547c28a3bf0c6ff5d77da6_MD5.png]]

服务器和客户端类存在情况

# CPP编写UI

![[f0a5b6a01c85f4996ee974cc66ea8045_MD5.png]]

用到了自带的ToText和四舍五入函数

# 造成伤害⭐⭐⭐⭐⭐

![[99234fffb9992e750ebee114fbb697b8_MD5.png]]

使用UE自带的ApplyDamage，需要一个instigator，这个是发起者的意思，接收一个PlayerController，

Causer接收发起伤害的Actor。我们暂时没有自定义伤害类型，这里使用UDamageType::StaticClass即可。

变量复制比RPC效率高得多，应该尽量避免使用RPC。ReceiveDamage只会发生在Server上，在这里进行一次PlayHitReactMontage，OnRep_Health只会发生在Client上，我们在这里也进行一次PlayHitReactMontage，避免了使用RPC

## 事件种类

对于事件有三种：Multicast （多路广播）, Server （服务器）, 以及 Client （客户端）。勾选 多路广播，该事件应在服务器上进行调用，在服务器上执行然后自动转发到客户端。 勾选 Server，该事件应在客户端调用，随后仅在服务器上执行。 勾选 Client，该事件应在服务器调用，随后仅在其所拥有的客户端上执行。

## RPC细节

RPC 函数非常有用，可允许客户端或服务器通过网络连接相互发送消息。 远程调用函数可设置为 Reliable 或 Unreliable，其中 Reliable 调用必定会发生，而 Unreliable 调用可能会在网络繁忙时被丢弃。大多数处理装饰视觉效果的远程调用函数应设置为 Unreliable，以避免过多占用网络。

远程调用函数主要包括 3 种类型：Multicast 广播、Run on Server 在服务端执行 和 Run on owning Client 在客户端执行

# GameMode⭐⭐⭐⭐⭐

![[4f1e35b58550ea5dd0e65e91273998a9_MD5.png]]

继承自gamemode而不是Gamemodebase，因为gamemode有更多的已存在的函数和功能。

![[72634c9e16e74adb118e2383b63c2c94_MD5.png]]

在玩家受伤时与GameMode联系起来

![[b6093ff2b004a85be94a3c41e1686842_MD5.png]]

动画蓝图中对是否死亡做处理，成功在服务器上播放，接下来的目标是在所有客户端上播放

![[6da8ea853c04b71dfa8896e9cf217273_MD5.png]]

将它声明为multicastRPC即可，要保证Reliable

# 使用Timer完成重生

![[48c5c02661d17d5eb5bb5739dcb23534_MD5.png]]

![[7eb6d853cfa4f4a779f4feee871eba72_MD5.png]]

Timer的第二次应用

# DissolveMaterial⭐⭐⭐⭐⭐

P91

![[11af057965744b907d73743ff11157bd_MD5.png]]

使用Mask模式，将得出的结果作为Mesh上的溶解,把溶解程度和发光倍数提升为参数，使用1-x的数据作为UV Data，UV就是材质的XY坐标

![[618970541a94cc3ba8952905d2f50ff8_MD5.png]]

![[80cbb1fb126c9fcc97586c7fec7f90df_MD5.png]]

使用UE自带的Texture制造边缘锐利效果，将这段代码加到角色材质上即可

![[2abc25f1deedf777c720b971d3349f74_MD5.png]]

使用CPP定义Timeline并且使用Curve控制溶解程度

![[12ceac2a6419e7b6fc3b26f17fa6a1b6_MD5.png]]

增加轨道

![[10384a43b3290ad50689550ec985e614_MD5.png]]

使用TEXT设定参数

# 机器人特效

![[0449f03b4b2d30aaf6eb49564e4ff70c_MD5.png]]

设置粒子寿命

# PlayerState⭐⭐⭐⭐⭐

![[47ed8423f0111e194c9d2e405f00e0c7_MD5.png]]

PlayerState储存玩家状态相关的东西，例如分数。Score是PlayerState自带的东西，这意味着他会自动复制。

PlayerState在服务器上在BeginPlay之前就已经初始化完成，在客户端上，需要重写OnRep_PlayerState()赋初始值

![[e42924d2fbf909129a69df7b72d5f46c_MD5.png]]

给空指针加上UPROPERTY,他会是一个真正的空指针，不会发生内存访问错误

![[55649c11866c9dfe08bcf544ce4400f2_MD5.png]]

需要在捡起武器时就设置正确的子弹数量，我们可以重写OnRep_Owner

# 有条件变量复制⭐⭐⭐⭐⭐

![[5a74872f11921d6fe45b420114acef50_MD5.png]]

在复制玩家储备弹药这种变量时，我们无需把他复制到所有客户端，只需要复制到所有者的客户端即可。

![[575b418aabb66fd0a1d3df0c0456b6a1_MD5.png]]

defaultMAX的意义是，我们可以直接查看他等于几，来告诉我们一共有多少个枚举值

![[d369d0c4f3bb870779cd1378dccab627_MD5.png]]

TMAP无法网络复制，需要想出别的办法

# 换子弹问题

![[02055b63427324c28c20d47902cc1c5b_MD5.png]]

最优解

![[c8856b5d964de81fbd8807f912a38b98_MD5.png]]

使用特殊语法初始化倒计时，保证01也会有两位数字

# 同步客户端和服务器时间⭐⭐⭐⭐⭐

![[bb4c2e0c6e5c2f00d274829d1a5cd3c5_MD5.png]]

需要以服务器为主，加上RPC时延

![[5e9a60381281807cb620eee17a00fce0_MD5.png]]

我们无法获得两边的时间，只能使用两个 1/2RTT进行估计，这个技术上被称为非对称往返时间。

![[10769763f3d2cb507353fb5bd6cce422_MD5.png]]

![[1ea8fdc0229fa9855a63650bd4361602_MD5.png]]

需要实现客户端向服务器的询问，服务器对客户端的答复两个RPC函数

# GameMode⭐⭐⭐⭐⭐

p111

![[1716a4695a72e3013b6a192e77550a6e_MD5.png]]

GameMode相比于GameModeBase，多了MatchState以及相关的切换函数

![[5f24b22bf0018b3218b91b150bbadf1f_MD5.png]]

MatchState包括很多状态，我们也可以自定义我们自己的状态。

![[8f2ecd28cc605edfe7a24e7b9447743b_MD5.png]]

在有一大堆变量需要复制时，我们可以不使用变量复制，直接使用一个ServerRPC完成任务。

# BlasterGamestate⭐⭐⭐⭐⭐

![[19a53115a98c69917a1377f86eb5b5b8_MD5.png]]

维护一个数组来表示现在得分最多的玩家

# Make Smoke Trail⭐⭐⭐⭐⭐

![[c689705b465ec1a78185bc744ecceffb_MD5.png]]

为我们的火箭轨迹创建一个Material和MaterialInstance

![[b31f26c3815d077cc47b24694aeea005_MD5.png]]

![[00f56bd8a471284951e4113a866decc6_MD5.png]]

创建Emitter，定义随机的旋转，随机的图案和随机的持续时间，颜色随时间改变。

# 延迟补偿⭐⭐⭐⭐⭐

P159

![[8a4312311f6bb3583707043d75714103_MD5.png]]

从客户端一侧先进行预测，发送这个行为到服务器，再广播给其他客户端

![[058f2d384c4c39c42baddb05ff87f241_MD5.png]]

内插值，在别的客户端，会取两次数据的中间值。

![[d7fc6847c45cd27d4fdd4641da12197b_MD5.png]]

外插值，在下一次值到来时假设目标接着往前走

![[775a09fc494e1e26a80fb814405ff2a6_MD5.png]]

UE同时使用两种插值，并且在需要纠正时使用Rubber-banding.

## Sever-Side Rewind

![[afcd9990e612c89d4a9388d5202bef1f_MD5.png]]

在判断命中事件时，需要和之前Rewind的帧进行对比，判断是否真正命中了目标。

![[41c6ee829a64776ebc08b3520e151874_MD5.png]]

可以从PlayerState中获得Ping，这个值被UE/4，我们需要乘回去。

![[76689a028b565eebcdd74623549e147d_MD5.png]]

1.倒回命中时间，射线检测玩家的碰撞盒子。

2.确认命中后发送请求，重置所有盒子。

# 预测子弹数量

![[bef0dc16ad5b313846e6eca13b797c2e_MD5.png]]

客户端侧预测的小应用

# 服务器倒带实现⭐⭐⭐⭐⭐

![[98ef4d1080808161f5c3d021121b42bd_MD5.png]]

我们需要存下来前几帧角色的位置，精确程度按需求而定

## 构建FramePackage

![[362b14fad5279315eae90b53d399d63d_MD5.png]]

为各个部位制作用于Rewind的碰撞体

![[33ac2d1e4ebfed80663ae1069e4422aa_MD5.png]]

构建一个箱子的信息，frame的信息包括箱子的信息和名称以及对应的骨骼名字，这个使用TMap实现。

![[d94d832b449772a1791f935d3ee8c08b_MD5.png]]

每帧把这个信息保存下来

![[3dbf2366e1456b1b280ef51552404b06_MD5.png]]

最好的解决方案是无环双链表

![[97822aba6e4c69dad4fd324276fb7b6b_MD5.png]]

每帧存储，并且可以规定丢弃时间

## InterpTo原理

![[0d081340fdb3129cd20abce7ab517abc_MD5.png]]

## 记录往返时间

![[2fafea5d98ebe1a4d7d4d01804175637_MD5.png]]

我们需要估计隔一段时间记录一次往返时间，为我们的服务器倒带函数提供参考。

# PostEditProperty

![[e6c833645ca7ff78aa7c5d317810c3f8_MD5.png]]

重写这个方法来保证各种属性的蓝图和C++一致，只在Editor情况下编译这一段

# ProjectileRewind⭐⭐⭐⭐⭐

![[5fa0ba77b4a652ef89f619e5ca942ab5_MD5.png]]

![[c6fc26486d600b9eb67c65efb522971a_MD5.png]]

非常复杂，需要处理Server-Client,Locally-notLocally,useSSR-orNot。这一段是，projectile武器，在收到开火指令后生成子弹的处理。减少ssr的调用次数。

这样处理，在服务器上的结果是一样的。

如果在客户端时，如果不使用SSR，我们会在网络延迟后发射我们的子弹。我们不会生成本地的子弹，会等待服务器返回复制的子弹。

如果使用SSR，我们会在客户端上立刻看见SSR子弹，随后在服务器和别的客户端上看见非复制的非SSR子弹。

这样操作能在客户端上立刻看见结果，提升可玩性，并且也带有正确的服务器结果。

# 作弊与验证⭐⭐⭐⭐⭐

## 作弊方法

### CodeInjection

![[238dcd165712c9752061f60aff14dbc2_MD5.png]]

通过高级语言查看运行时的内存，找到值之后使用poke命令修改这个值。或者使用.dll库注入游戏中

### 网络数据包伪造

![[df47b40fb6be65ff26695a7d770294a2_MD5.png]]

通过截下客户端发给服务器的包，替换其中一些数据为优势数据实现作弊效果。

### 利用Bug

![[00684db16abc26b6af3f6fe80a0591d6_MD5.png]]

例如进入一些不允许进入的区域，你能攻击Boss但Boss不能攻击你。

### MemoryEditing⭐⭐⭐⭐⭐

![[3afa69bf7e507abaeef6f4164e44ac1a_MD5.png]]

利用一个应用程序来监控游戏内存，将一些值修改成优势数据。

## Validation

![[782e46ac8f1ef8ada4454217d9edd0b0_MD5.png]]

UE自带验证函数，用于阻止上文所说的MemoryEditing方法。

对于前两种办法UE会自动阻止，不太可能发生。

## 项目实战

![[e8b78969a99adced8b31b888e2c810af_MD5.png]]

在beginPlay中把开火速度改得超级大

![[293128c770e38d00b99faf91f741cf0b_MD5.png]]

利用UE自带Validate函数，如果返回false，会直接将玩家踢出游戏。

# 记录退出游戏玩家

![[c3f6036fae3df51f3bc5caa4d2637679_MD5.png]]

客户端退出游戏时，应该有一个完整的流程。

![[dab2686a56b8b16bdd83420c48ef2c32_MD5.png]]

角色，GameMode，菜单三个类之间需要配合。