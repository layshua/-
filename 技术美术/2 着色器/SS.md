# 原理
Subsurface Scatting
次表面散射可以使用**光线追踪**实现实时，游戏中更多的是对次表面散射的“快速模拟”。

![[02 PBR理论#^c36ln4]]
![[02 PBR理论#^lxo1ef]]

BSSRDF：
![[Pasted image 20230723134543.png|500]]


# 实现
SSS=背光+扰动+扩散+...

![[Pasted image 20230723135132.png]]
> $dot (N,+L)$ 是正光，$dot (V,-L)$ 是背光

