
# 混合空间八方向
[【UE5】角色动画蓝图，制作八方向混合空间_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1TG4y1V75k/?spm_id_from=333.337.top_right_bar_window_custom_collection.content.click)

混合空间设置如图：
![[Pasted image 20231006093323.png|400]]
![[Pasted image 20231006093239.png]]

动画蓝图中计算 Direction 和 Velocity
![[Pasted image 20231006093631.png]]
# 角度问题
## 坐标系旋转角正负

在左手定则中，**从轴的正方向看向原点**，顺时针旋转方向为正方向；在右手定则中，从轴的正方向看向原点，逆时针旋转方向为正方向。

在UE4中，**坐标系遵循左手定则**，其**Z轴的旋转方向符号是遵循左手定则**的旋转方向，**但X和Y轴却遵循右手定则**：

（1）Z轴，旋转方向遵循左手定则，顺时针为正。
（2）X、Y轴，旋转方向遵循右手定则，逆时针为正。

## 鼠标XY轴

UE4中**鼠标的 XY 轴**遵循左手定则，从+Z 看向原点时，+X 指向右侧，+Y 指向下方。
![[Pasted image 20231006090308.png]]
又因为 UE4的坐标系遵循左手定则，但 XY 轴的旋转方向却遵循右手定则（即逆时针为正），故鼠标往+Y 方向移动时，Pitch（俯仰角）为正值（绕 Y 轴逆时针旋转）视角向上抬。
>注意鼠标 XY 值左上角为原点

需要特别注意的是：因为在BaseGame.ini中，InputPitchScale=-2.5，且PlayerController自带的AddPitchInput接口会将鼠标Pitch输入系数乘以该配置，所以一般**MouseY的轴映射为-1.0**，这样鼠标往上（-Y，Pitch<0）移动时，视角上抬，鼠标往下（+Y，Pitch>0）移动时，视角下移。
# 人物转向问题
TODO：模拟逆水寒自在模式

[虚幻4人物转向问题{User Controller Rotaion Yaw，User Controller Desired Rotation与Orient Rotaion to Movement}-CSDN博客](https://blog.csdn.net/u012249992/article/details/83186907)
![[Pasted image 20231006091114.png]]
![[Pasted image 20231006091124.png]]


通常我们新建第三人称项目时，默认如上图的设置，这个时候按 s 键人正面是面向镜头的，这在一般的 rpg 或 act 游戏都是如此设置，而在绝地求生这类 STG 里按 s 键是背对镜头的，如何设置人物转向就在上面三个 bool 值决定。

`Use Controller Rotaion Yaw` 与 `User Controlle Desired Rotation` 都将人物与镜头视角绑定，即让人物始终跟随镜头转向，区别在于第二个会让转向更加平滑的过度，也可以设置转向的速度，第一个就直接跟鼠标或手柄移动一致转向，如果想让转向更平滑可以设置第二个 bool 值为 true 即可，`Orient Rotaion to Movement` 则在没有 wasd 这种 move 输入的时候不会让人物转向，在人物跑动的时候会有转向效果但是没有往后跑的效果，按 s 键人物正面就会朝着玩家，前两个 bool 值则相反
