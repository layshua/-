# 工程文件结构
![[Pasted image 20230602225358.png]]
![[Pasted image 20230602225421.png]]

只要有 Assets 文件就可以打开，其他由 unity 自动生成。

# 基本操作
![[Pasted image 20230602231416.png]]

Move to view：ctrl+alt+F   ，让物体移动到 Scene 视角中心
Align with View：ctrl+shift+F，选中摄像机后使用，可以让摄像机视角变为 Scene 中的视角


# 资源类型
![[Pasted image 20230602232944.png]]
# 关闭脚本自动重新编译
1. Unity 的修改
Edit->Prefrence->Asset Pipeline
![[Pasted image 20230607161139.jpg]]

2. Rider 的修改
修改方法： Settings - Languages&Frameworks - Unity Engine - 取消勾选 Automatically refresh assets in Unity
![[Pasted image 20230607161142.jpg]]
3. 想要刷新的时候，手动 ctrl+r