
# ddx, ddy
`ddx`，`ddy` 的定义  
GPU 在像素化的基本单位是 2x2，那么在这个 2x2 像素块当中，右侧的像素对应的 fragment 的 x 坐标减去左侧的像素对应的 fragment 的 x 坐标就是 ddx；下侧像素对应的 fragment 的坐标 y 减去上侧像素对应的 fragment 的坐标 y 就是 ddy。  
ddx 和 ddy 代表了相邻两个像素在设备坐标系当中的距离，据此可以判断应该使用哪一层的贴图 LOD（如果贴图支持 LOD，也就是 MIPS）。
**常见使用：**  
**边缘突出**  
ddx (v) = 该像素点右边的 v 值 - 该像素点的 v 值  
ddy (v) = 该像素点下面的 v 值 - 该像素点的 v 值
![[Pasted image 20230722233358.png]]
![[Pasted image 20230722233401.png]]
**边缘亮化**  

![[Pasted image 20230722233410.png]]
![[Pasted image 20230722233412.png]]