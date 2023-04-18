有的时候我们在使用 UE4 的时候，需要设计出别样的 Shader, 往往 UE4 内部的材质无法满足，这个时候，就需要我们手写 Shader，了解 HLSL 就至关重要，现在我们总结一下 HLSL 语言基础:

## **变量类型**

**标量类型**

bool ,int ,float,double 基本上可 C++ 差不多，所以不再这里讲解；  
half:16 位的浮点数

**向量类型**

float2 ,2D 向量，内部存储着两个浮点类型 float  
float3 ,3D 向量，内部存储着三个浮点类型 float  
float4 ,4D 向量，内部存储着四个浮点类型 float  
除此之外还有：  
int2 ,2D 整数，内部存储着两个类型 int  
int3 ,3D 整数，内部存储着三个类型 int  
int4 ,4D 整数，内部存储着四个类型 int  
除此之外还有：  
bool2 ,2Dbool，内部存储着两个类型 bool  
bool3 ,3Dbool，内部存储着三个类型 bool  
bool4,4Dbool，内部存储着四个类型 bool  
...  
TypeN (2<=N<=4)  
...

**向量初始化**

float3 v = {1.0f, 2.0f, 3.0f};  
float2 w = float2(x, y);// 内部支持联等  
float4 u = float4(w, 3.0f, 4.0f); // u= (w.x, w.y, 3.0f, 4.0f)  
向量的访问  
用数组语法：  
如：pos[i] = 3.14f;  
用字母语法，xyzw，rgba。  
如：  
pos.x = pos.r = 1.0f;  
pos.y = pos.g = 2.0f;  
pos.z = pos.b = 3.0f;  
pos.w = pos.a = 4.0f;

**向量的替换调配**

float2 pos = {x,y};  
float3 pos = {x,y,z};  
float4 pos = {x,y,z,w};  
例如  
float4 UV1 = {0.1f, 0.2f, 0.3f, 0.4f};  
float4 UV2 = {0.0f, 0.0f, 0.0f, 0.0f};  
UV2 = UV1 .wyyx; // v = {0.4f, 0.2f, 0.2f,0.10=f}  
UV2 = UV1 .wzyx; // v = {0.4f, 0.3f, 0.2f,0.1f}  
UV2.xy = u; // v = {0.1f,0.2f, 0.0f,0.0f}

## **矩阵类型**

**矩阵定义**

语法：Typexn xxx 例如：  
float2x2:2x2 的矩阵，类型是 float;  
half3x3:3x3 矩阵类型是 half  
int4x4: 4 × 4 矩阵类型是 int  
bool3x4: 3 × 4 矩阵类型是 bool

**矩阵访问**

矩阵的数组语法：M[i][j] = value;

M._11 = M._12 = M._13 = M._14 = 0.1f;  
M._21 = M._22 = M._23 = M._24 = 0.2f;  
M._31 = M._32 = M._33 = M._34 = 0.3f;  
M._41 = M._42 = M._43 = M._44 = 0.4f;  
or  
M._m00 = M._m01 = M._m02 = M._m03 =0.5f;  
M._m10 = M._m11 = M._m12 = M._m13 =0.6f;  
M._m20 = M._m21 = M._m22 = M._m23 =0.7f;  
M._m30 = M._m31 = M._m32 = M._m33 =0.8f;

矩阵初始化

float2x2 Mat =float2x2(1.0f,2.0f,3.0f,4.0f);  
int2x2 Mat ={1,2,3,4};

**矩阵赋值**

float3 Nor = normalize(pIn.normalW);  
float3 Tan = normalize(pIn.tangentW -dot(pIn.tangentW, N)*N);// 固定写法  
float3 B = cross(Nor ,Tan);  
float3x3 NTB;  
NTB[0] = Nor ; // float3  
NTB[1] =Tan ; // float3  
NTB[2] = B; // float3

**其他定义**

vector u = {1.0f, 2.0f, 3.0f, 4.0f};  
matrix M; // 4x4 矩阵

**数组**

float C[8][7];  
half t[n];  
int3 v[42];//42 个 int3

**结构体**

HLSL 中的结构体类似 C 中的结构体，不能含有函数成员，访问成员只需要根据下标即可。

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

**和变量相关的关键字**

typedef，static，uniform，extern，const，shared;  
typedef:  
HLSL 的 typedef 关键字功能和 C++ 里的完全一样。例如，我们可以给类型 vector<float, 3>  
typedef vector<float, 3> point;  
typedef const float CFLOAT;  
typedef float point2[2];  
用下面的语法命名：  
static：  
变量默认是 extern 外部可见。一旦声明了 static 后变量就是内部变量，不会暴露在着色器外面  
uniform：  
uniform——如果变量以 uniform 关键字为前缀，就意味着此变量在着色器外面被初始化，比如被 C++ 应用程序初始化，然后再输入进着色器。  
extern:  
extern——如果变量以 extern 关键字为前缀，就意味着该变量可在着色器外被访问，比如被 C++ 应用程序。仅全局变量可以以 extern 关键字为前缀。不是 static 的全局变量默认就是 extern。  
shared：  
shared——如果变量以 shared 关键字为前缀，就提示效果框架：变量将在多个效果间被共享。仅全局变量可以以 shared 为前缀。  
volatile：  
volatile——如果变量以 volatile 关键字为前缀，就提示效果框架：变量将时常被修改。仅全局变量可以以 volatile 为前缀  
const：  
const——HLSL 中的 const 关键字和 C++ 里的意思一样。也就是说，如果变量以 const 为前缀，那此变量就是常量，并且不能被改变。

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

## **其他教程地址**

[人宅：人宅系列教程](https://zhuanlan.zhihu.com/p/60117613)