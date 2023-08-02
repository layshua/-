
> [!NOTE] Title
> Unity： UV 坐标原点为左下角
> UE：UV 坐标原点为左上角

# 遮罩
## 1 UV 坐标系转换成笛卡尔直角坐标系
```cs
float4 frag(Varyings i) : SV_Target  
{  
    i.uv = (i.uv - 0.5)*2;
    //或
    i.uv = i.uv * 2 - 1;
}
```
![[Pasted image 20221215123919.png]]
![[Pasted image 20221215123942.png]]
## 2 圆形遮罩
**方法一：**
![[Pasted image 20221215130852.png]]
不能用 power 节点代替 multiply，因为 power 节点不支持负数运算，小于 0 的数都会被 clamp 到0。

**方法二：**
![[Pasted image 20221215131923.png]]
UE 材质中的 Sine 函数，取值范围是-1~1，周期为 1（不是 2pai）。
![[Pasted image 20221215131719.png|]]

**方形遮罩**使用这两个节点相乘即可：
![[Pasted image 20221215142944.png]]

**方法三：SphereMask**
![[Pasted image 20221215132224.png]]

**RadialGradientExponential（指数径向渐变）**函数
![[Pasted image 20230118202614.jpg]] ![[Pasted image 20230118202557.jpg]]
**UV（矢量 2）（UVs (Vector 2)）**
用于控制渐变所在的位置及其涵盖 0-1 空间的程度。

**中心点（矢量 2）（CenterPosition (Vector2)）**
基于 0-1 的渐变中心位置偏移。

**半径（标量）（Radius (Scalar)）**
源自中心的径向渐变的大小。默认值 0.5 使渐变边缘位于纹理空间边缘附近。

**密度（标量）（Density (Scalar)）**
调整此函数所产生的渐变的硬度。这个数值越大，意味着渐变越清晰。

**反转密度（布尔值）（Invert Density (Boolean)）**
对于渐变，将白色反转为黑色，并将黑色反转为白色。

**DiamondGradient（钻石型渐变）**
![[Pasted image 20230118202708.jpg]] ![[Pasted image 20230118202702.jpg]]
**衰减（标量）（Falloff (Scalar)）**
通过控制渐变从白色变为黑色的速度，提高渐变对比度。
## 3 线性遮罩
![[Pasted image 20221215133043.png]]
![[Pasted image 20221215133053.png]]
![[Pasted image 20221215145815.png]]

## 4 世界坐标 mask

![[042402eab80e213473b1cba5fbad412b_MD5.webp]]

```cs
//x方向
float mask = saturate(pow(abs(frac(rebuildPosWS.x*_ScanSpace-_Time.y*_TimeSpeed)),_ScanPower))*_ScanScale;
//y方向
float mask = saturate(pow(abs(frac(rebuildPosWS.y*_ScanSpace-_Time.y*_TimeSpeed)),_ScanPower))*_ScanScale;
//z方向
float mask = saturate(pow(abs(frac(rebuildPosWS.z*_ScanSpace-_Time.y*_TimeSpeed)),_ScanPower))*_ScanScale;
```

# 扫线

![[2023771458.gif|300]]

```c
float flow=saturate(pow(1-abs(frac(i.positionWS.y*0.3-_Time.y*0.2)-0.5),10)*0.3);  
float4 flowcolor=flow*_FlowLightColor;
```

## 世界坐标画线
重建的世界坐标取小数+取余数，即可

得到世界坐标后，因为我们的坐标轴取值范围是从 -∞到 +∞，而颜色的范围只是 0-1 之间，如果对世界坐标使用 frac 取小数就可以得到只在 0-0.99 的值了
```c
return float4(frac(rebuildPosWS),1);
```

![[61fb3efd59dc2c6cf9b00ff81ee5c9b4_MD5.webp]]

取余数

对于在 0-1 之间均匀变换的我们想得到它的边界位置，所以直接来个 step 函数，我们只取 0.98 到 1.0 之间的值为 1，其他的值为 0，我们把它输出出来。是不是有那味了，线框就直接出来了。

![[89da9b0159e843562f581518306b433d_MD5.webp]]

但是这个线框的颜色红蓝绿（为什么是红蓝绿？是因为它对应 XYZ 三个轴向）很乱而且不好看，我们想要自由控制颜色，我们定义三个颜色，去分别控制 XYZ 方向的线框颜色，并把它输出。

![[248dd835198f5ef05b6c64552a6603ca_MD5.webp]]



# 法线
##  多面显示
xyz 面分别显示对应图片，此时-x-y-z 面颜色值为-1黑色
![[Pasted image 20221003172541.png|300]] ![[Pasted image 20221003172610.png]]
六面显示 ![[Pasted image 20221003174221.png]]
