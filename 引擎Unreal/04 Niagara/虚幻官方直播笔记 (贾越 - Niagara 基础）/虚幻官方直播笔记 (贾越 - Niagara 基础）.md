## 【Niagara 基础 1】

最重要的理解部分：

![](1673936189951.png)

![](1673936189984.png)

*   材质在粒子中不显示，打开材质以下选项：
![[Pasted image 20230119232315.png]]
*   复杂特效调试时很卡的话，关掉以下选项：
![[Pasted image 20230119234331.png]]
![](1673936190372.png)

## 【Niagara 基础 2】

*   慢速观察特效命令：~ 加输入 slomo(0.1 是十分之一的速度，2 是两倍速）

![](1673936190417.png)

*   在 niagara system 里面添加自定义属性的时候，要先把它拖到例如 particle spawn 中，让它先参与运算，这样才能在 Bindings 中找到该属性：

![](1673936190608.png)

*   要用 Beam 时，要记得加上 spawn beam 和 beam emitter setup :

![](1673936190642.png)

*   user 是 open 的一个 system 参数:

![](1673936190687.png)

*   键盘的；号键是 Debug Camera 快捷键，F 键是 Freeze Camera 快捷键：

![](1673936190717.png)

*   Beam 和 Ribbon 的关系: Beam 就是利用 Ribbon 的一种方式

## 【Niagara 基础 3】

*   没有太多色彩倾向的贴图，绿色通道会更加精确一些。
*   在 ribbon 上做一个材质，并且纹理不受长度影响，就应修改 UV0TillingDistance

![](1673936190761.png)

*   视口增加百分之五的小技巧：

![](1673936190805.png)

*   paragon 项目的贴图特效资源基本都够你用。

*   size random 技巧

![](1673936190834.png)

*   cup 粒子是每个都去算每个都去检测的状态，改成 GPU 会省很多

![](1673936190877.png)