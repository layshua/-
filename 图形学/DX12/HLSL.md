# 变量类型
## 标量/向量

**标量类型**

`bool ,int ,float,double` 基本上可 C++ 差不多，所以不再这里讲解；  
`half`:16 位的浮点数

**向量类型**

`float2` ,2D 向量，内部存储着两个浮点类型 float  
`float3` ,3D 向量，内部存储着三个浮点类型 float  
`float4` ,4D 向量，内部存储着四个浮点类型 float  
除此之外还有：  
`int2` ,2D 整数，内部存储着两个类型 int  
`int3` ,3D 整数，内部存储着三个类型 int  
`int4` ,4D 整数，内部存储着四个类型 int  
除此之外还有：  
`bool2` ,2Dbool，内部存储着两个类型 bool  
`bool3` ,3Dbool，内部存储着三个类型 bool  
`bool4`,4Dbool，内部存储着四个类型 bool  
...  
TypeN (2<=N<=4)  
...

**向量初始化**
```c++ nums
float3 v = {1.0f, 2.0f, 3.0f};  
float2 w = float2(x, y);// 内部支持联等  
float4 u = float4(w, 3.0f, 4.0f); // u= (w.x, w.y, 3.0f, 4.0f)  
```
向量的访问  
用数组语法：  
如：`pos[i] = 3.14f; ` 
用字母语法，xyzw，rgba。  
如：  
```c++ nums
pos.x = pos.r = 1.0f;  
pos.y = pos.g = 2.0f;  
pos.z = pos.b = 3.0f;  
pos.w = pos.a = 4.0f;
```

**向量的替换调配**
```c++ nums
float2 pos = {x,y};  
float3 pos = {x,y,z};  
float4 pos = {x,y,z,w};  
//例如  
float4 UV1 = {0.1f, 0.2f, 0.3f, 0.4f};  
float4 UV2 = {0.0f, 0.0f, 0.0f, 0.0f};  
UV2 = UV1 .wyyx; // v = {0.4f, 0.2f, 0.2f,0.10=f}  
UV2 = UV1 .wzyx; // v = {0.4f, 0.3f, 0.2f,0.1f}  
UV2.xy = u; // v = {0.1f,0.2f, 0.0f,0.0f}
```


## **矩阵类型**

**矩阵定义**

`float2x2`:2x2 的矩阵，类型是 float;  
`half3x3`:3x3 矩阵类型是 half  
`int4x4`: 4 × 4 矩阵类型是 int  
`bool3x4`: 3 × 4 矩阵类型是 bool

**矩阵访问**

矩阵的数组语法：`M[i][j] = value`;
除此之外，可以依照访问结构体成员的方式获取元素：
```c++ nums
// 以1作为起始值的索引
M._11 = M._12 = M._13 = M._14 = 0.1f;  
M._21 = M._22 = M._23 = M._24 = 0.2f;  
M._31 = M._32 = M._33 = M._34 = 0.3f;  
M._41 = M._42 = M._43 = M._44 = 0.4f;  

// 以0作为基准值的索引
M._m00 = M._m01 = M._m02 = M._m03 =0.5f;  
M._m10 = M._m11 = M._m12 = M._m13 =0.6f;  
M._m20 = M._m21 = M._m22 = M._m23 =0.7f;  
M._m30 = M._m31 = M._m32 = M._m33 =0.8f;
```

**通过数组单下标获取行向量**

**矩阵初始化**
```c++ nums
float2x2 Mat =float2x2(1.0f,2.0f,3.0f,4.0f);  
int2x2 Mat ={1,2,3,4};
```

**矩阵赋值**
```c++ nums
float3 Nor = normalize(pIn.normalW);  
float3 Tan = normalize(pIn.tangentW -dot(pIn.tangentW, N)*N);// 固定写法  
float3 B = cross(Nor ,Tan);  
float3x3 NTB;  
NTB[0] = Nor ; // float3  
NTB[1] =Tan ; // float3  
NTB[2] = B; // float3

```

**除了利用 float4 与 float4x4 类型来表示 4D 向量与 4x4 矩阵外，我们还可以使用 vector 与 matrix 类型来加以代替：**
```c++ nums
vector u = {1.0f, 2.0f, 3.0f, 4.0f};  
matrix M; // 4x4 矩阵
```

## **数组**
和 C++一样
```c++ nums
float C[8][7];  
half t[n];  
int3 v[42];//42 个 int3
```

## **结构体**

类似 C++ 中的结构体，区别在于不具有成员函数
```c++ nums
struct TastShader  
{  
float3 normal;  
float3 postion;  
float4 diffuse;  
float4 spec;  
};  
TastShader T;  
litColor += T.diffuse;  
dot(lightVec, T.normal);  
float specPower = max(T.spec.a, 1.0f)；
```

## 变量的修饰符
下列关键字可以作为变量声明的修饰符。
1. `static`: 基本上与修饰符 extern 的作用相反，意即以 static 修饰的**着色器变量对于 C++
应用程序而言是不可见的。**

2. `uniform`: 对于经此修饰符标记的变量而言, 此值可以在 C++应用层改变,但**在着色器执行 (处理顶点、像素）的过程中，其值始终保持不变**（其间可将其看作一种常量)。因此，**uniform 变量都是在着色器程序之外进行初始化的（例如，在 C++应用程序中得到初始化)。**
3. `extern`; 经此关键字修饰的变量会**暴露给 C++应用程序端 (即该变量可以被看色器之外的 C++应用代码访问到)。着色器程序中的全局变量被默认为既 uniform 且 extern。**
4. `const`: HLSL 中的 const 关键字与 C++中的意义相同。
5.  `shared`：如果变量以 shared 关键字为前缀，就提示效果框架：变量将在多个效果间被共享。仅全局变量可以以 shared 为前缀。  
6.  `volatile`：如果变量以 volatile 关键字为前缀，就提示效果框架：变量将时常被修改。仅全局变量可以以 volatile 为前缀  

# 关键字与运算符
## 关键字
![[Pasted image 20230418160234.png]]
![[Pasted image 20230418160249.png]]

## 运算符
![[Pasted image 20230418160317.png]]
和 C++的差异：
取模运算符`%`既可以用于整数也可以用于浮点数，必须保证左右操作数有相同的符号。
向量的自增运算就是每个向量加 1
`*` 为向量的分量式乘法，若需要进行矩阵乘法运算，使用 `mul` 函数
比较运算符 `==` 按分量一一对比，返回多维结果
对于二维运算来说，如果所有操作数维度不同，则低维度自动提升为高维度。如果类型不同，低精度自动提升为高精度。比如 x 为 half，y 为 int，若计算 x+y，则 y 被提升为 half。



**着色器关联**

顶点和像素着色器提供的数据供其后的图形管线使用. 输出关联用于指定着色器产生的数据如何链接给下一阶段的输入. 举例来说, 顶点着色器的输出关联将顶点着色程序运算得到的结果链接到像素着色器的输入关联上.

顶点着色器输出关联用于将着色器连接到像素着色器和光栅阶段. POSITION 输出是每个顶点必须输出给光栅器而不暴露给像素着色器的数据,

TEXCOORDn 和 COLORn 表示输出用于像素着色器进行插值.

像素着色器输出关联将其输出颜色绑定给正确的渲染目标. 颜色输出被连接到 alpha 混合阶段. DEPTH 输出关联用于改变当前光栅化位置的目标深度值. 输出关联语义同输入关联的声明相同.

**顶点着色器的输入关联**

**关联符** **描述**  
POSITIONn 位置  
BLENDWEIGHTn 混合宽  
BLENDINDICESn 混合索引  
NORMALn 标准向量  
PSIZEn 点大小  
COLORn 颜色  
TEXCOORDn 纹理坐标  
TANGENTn 切线  
BINORMALn 副法线  
TESSFACTORn 细分因子

**像素着色器输入关联**

**关联符** **描述**  
COLORn 颜色  
TEXCOORDn 纹理坐标  
注意：n 是一个可选的整形变量

