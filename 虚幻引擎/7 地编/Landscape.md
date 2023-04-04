# 1 创建地形
高度图格式：PNG或RAW16

导入后法线和软件中地形不一致，是由于尺寸设置不一致
[[案例灵感]]
软件中长宽8km
**顶视图** 按住鼠标滚轮测量距离：接近4km，通过缩放调整xy轴，调到8km
![[Pasted image 20221216104617.png]]
软件中海拔为1505m
**左视图**：测量海拔，调整z轴到相近高度
![[Pasted image 20221216104735.png]]

成功
![[Pasted image 20221216105048.png]]

# 2 地貌管理
组件的LOD根据需求调节
![[Pasted image 20221216111256.png]]

### 移动组件
![[Pasted image 20221216111719.png]]![[Pasted image 20221216111828.png|300]]
选择地形》移动》右下角切换目标关卡》点击选中的地形
![[Pasted image 20221216111812.png]]

### 样条曲线
绘制：按住ctrl+左键
连接：shift选择+绘制
平滑：R

绘制地形：
![[Pasted image 20221216113137.png|300]]![[Pasted image 20221216113205.png|200]]

创建沟渠；
选择所有点，移动位置到陆地下面，选择所有分段》仅选中将地形变形
![[Pasted image 20221216113430.png]]

样条点和曲线都可以添加网格体
![[Pasted image 20221216113845.png]]
# 3 地形雕刻
雕刻：按住shift反向

可视性：混合模式选Masked，材质赋给地形，按左键即可挖洞，按住shift恢复
![[Pasted image 20221216115437.png|300]]

选择：遮罩，不会被笔刷影响

# 4 Landmass
必须开启编辑图层

开启曲线编辑：alt添加新点
![[Pasted image 20221216121101.png]]

后续
https://www.bilibili.com/video/BV1mD4y1D7D6?p=6&vd_source=02e3d219e0c32801f6b50c2266e6a7be