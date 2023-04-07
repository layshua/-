http://www.cppblog.com/biao/archive/2013/03/14/198416.html

共有两种库：  
一种是 LIB 包含了函数所在的 DLL 文件和文件中函数位置的信息（入口），代码由运行时加载在进程空间中的 DLL 提供，称为动态链接库 dynamic link library。  
一种是 LIB 包含函数代码本身，在编译时直接将代码加入程序当中，称为静态链接库 static link library。  
共有两种链接方式：  
动态链接使用动态链接库，允许可执行模块（.dll 文件或. exe 文件）仅包含在运行时定位 DLL 函数的可执行代码所需的信息。  
静态链接使用静态链接库，
        TestDLL(123);   //dll 中的函数，在 DllSample.h 中声明  


        return(1);  


上是任意个 obj 文件的集合，obj 文件是 cpp 文件编译生成的。在编译这种静态库工程时，根本不会遇到链接错误；即使有错，也只会在使用这个 l
        TestDLL(123);   //dll 中的函数，在 DllSample.h 中声明  


        return(1);  


b 的 EXT 文件或者 DLL 工程里暴露出来。  
在 VC 中新建一个 static library 类型的工程 Lib，加入 test.cpp 文件和 test.h 文件（头文件内包括函数声明），然后编译，就生成了 Lib.lib 文件。  
别的工程要使用这个 lib 有两种方式：  
（1）在 project->link->Object/Library Module 中加入 Lib.lib 文件（先查询工程目录，再查询系统 Lib 目录）；或者在源代码中加入指令 #pragma comment(lib, “Lib.lib”)。  
（2）将 Lib.lib 拷入工程所在目录，或者执行文件生成的目录，或者系统 Lib 目录中。  
（3）加入相应的头文件 test.h。  
使用 DLL 的方法：  
使用动态链接中的 lib，不是 obj 文件的集合，即里面不会有实际的实现，它只是提供动态链接到 DLL 所需要的信息，这种 lib 可以在编译一个 DLL 工程时由编译器生成。  
创建 DLL 工程的方法（略）。  
**（1）隐式链接**  
第一种方法是：通过 project->link->Object/Library Module 中加入. lib 文件（或者在源代码中加入指令 #pragma comment(lib, “Lib.lib”)），并将. dll 文件置入工程所在目录，然后添加对应的. h 头文件。

![](<assets/1680873938487.png>)

#include "stdafx.h"  

![](<assets/1680873938545.png>)

#include "DLLSample.h"  

![](<assets/1680873938607.png>)

  

![](<assets/1680873938660.png>)

#pragma comment(lib, "DLLSample.lib")    // 你也可以在项目属性中设置库的链接  

![](<assets/1680873938713.png>)

  

![](<assets/1680873938766.png>)

int main()  

![](<assets/1680873938825.png>)

{  

![](<assets/1680873938889.png>)

        TestDLL(123);   //dll 中的函数，在 DllSample.h 中声明  

![](<assets/1680873938951.png>)

        return(1);  

![](<assets/1680873939009.png>)

}  

![](<assets/1680873939063.png>)

**（2）显式链接**  
需要函数指针和 WIN32 API 函数 LoadLibrary、GetProcAddress 装载，使用这种载入方法，不需要. lib 文件和. h 头文件，只需要. dll 文件即可（将. dll 文件置入工程目录中）。

![](<assets/1680873939119.png>)
```c++ nums
```
#include <iostream>  

![](<assets/1680873939173.png>)

#include <windows.h>         // 使用函数和某些特殊变量  

![](<assets/1680873939232.png>)

typedef void (*DLLFunc)(int);  

![](<assets/1680873939287.png>)

int main()  

![](<assets/1680873939342.png>)

{  

![](<assets/1680873939397.png>)

        DLLFunc dllFunc;  

![](<assets/1680873939450.png>)

        HINSTANCE hInstLibrary = LoadLibrary("DLLSample.dll");  

![](<assets/1680873939504.png>)

  

![](<assets/1680873939560.png>)

        if (hInstLibrary == NULL)  

![](<assets/1680873939614.png>)

        {  

![](<assets/1680873939670.png>)

          FreeLibrary(hInstLibrary);  

![](<assets/1680873939730.png>)

        }  

![](<assets/1680873939789.png>)

        dllFunc = (DLLFunc)GetProcAddress(hInstLibrary, "TestDLL");  

![](<assets/1680873939853.png>)

        if (dllFunc == NULL)  

![](<assets/1680873939914.png>)

        {  

![](<assets/1680873939967.png>)

          FreeLibrary(hInstLibrary);  

![](<assets/1680873940023.png>)

        }  

![](<assets/1680873940077.png>)

        dllFunc(123);  

![](<assets/1680873940136.png>)

        std::cin.get();  

![](<assets/1680873940240.png>)

        FreeLibrary(hInstLibrary);  

![](<assets/1680873940304.png>)

        return(1);  

![](<assets/1680873940362.png>)

}  

![](<assets/1680873940421.png>)

  

![](<assets/1680873940475.png>)

LoadLibrary 函数利用一个名称作为参数，获得 DLL 的实例（HINSTANCE 类型是实例的句柄），通常调用该函数后需要查看一下函数返回是否成功，如果不成功则返回 NULL（句柄无效），此时调用函数 FreeLibrary 释放 DLL 获得的内存。  
GetProcAddress 函数利用 DLL 的句柄和函数的名称作为参数，返回相应的函数指针，同时必须使用强转；判断函数指针是否为 NULL，如果是则调用函数 FreeLibrary 释放 DLL 获得的内存。此后，可以使用函数指针来调用实际的函数。  
最后要记得使用 FreeLibrary 函数释放内存。  
注意：应用程序如何找到 DLL 文件？  
使用 LoadLibrary 显式链接，那么在函数的参数中可以指定 DLL 文件的完整路径；如果不指定路径，或者进行隐式链接，Windows 将遵循下面的搜索顺序来定位 DLL：  
（1）包含 EXE 文件的目录  
（2）工程目录  
（3）Windows 系统目录  
（4）Windows 目录  
（5）列在 Path 环境变量中的一系列目录  
From: [http://www.cppblog.com/ming81/archive/2013/03/04/198215.html](http://www.cppblog.com/ming81/archive/2013/03/04/198215.html)  

.h 头文件是编译时必须的，lib 是链接时需要的，dll 是运行时需要的。

附加依赖项的是. lib 不是. dll，若生成了 DLL, 则肯定也生成 LIB 文件。如果要完成源代码的编译和链接，有头文件和 lib 就够了。如果也使动态连接的程序运行起来，有 dll 就够了。在开发和调试阶段，当然最好都有。

.h .lib .dll 三者的关系是：

H 文件作用是: 声明函数接口 

DLL 文件作用是: 函数可执行代码 

当我们在自己的程序中引用了一个 H 文件里的函数, 编链器怎么知道该调用哪个 DLL 文件呢? 这就是 LIB 文件的作用: 告诉链接器 调用的函数在哪个 DLL 中，函数执行代码在 DLL 中的什么位置 ，这也就是为什么需要附加依赖项 .LIB 文件，它起到桥梁的作用。如果生成静态库文件，则没有 DLL ，只有 lib，这时函数可执行代码部分也在 lib 文件中

目前以 lib 后缀的库