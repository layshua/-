#### 文章目录

*   [前言](#_2)
*   [步骤](#_6)
*   *   [1. 下载软件](#1_7)
    *   [2. 下载 node](#2node_14)
    *   [3. 开始操作](#3_31)
    *   [4. 中文](#4_91)
*   [下载现成的](#_118)

## 前言

[StartUML](https://so.csdn.net/so/search?q=StartUML&spm=1001.2101.3001.7020) 这款软件虽然是免费的，但经常弹出注册窗口很烦，所以就有了本文

如果觉得麻烦，可以直接去文末下载我已经弄好的（本文已经所介绍的为旧版，最新版可到文末获取）

## 步骤

### 1. 下载软件

软件直接去官网下载最新版本即可，[点击](https://staruml.io/download)去官网下载

![](https://img-blog.csdnimg.cn/51476e4699b44a88858242ee7ae6c6bf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
目前最新是 5.0.1 版本

安装过程就不说了，正常安装即可

### 2. 下载 node

然后就是下载 Node，点击[这里](http://nodejs.cn/download/)去官网下载  

![](https://img-blog.csdnimg.cn/8fb1b18433004053b8bd1f86bf6f3e0c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

自己选择合适自己的版本下载安装即可，现在电脑一般为 64 位，就下载图中的 64 位即可

下载完一路点击确认安装即可

然后 win+R 快捷键，输入 cmd，进入控制台  

![](https://img-blog.csdnimg.cn/3f4bfd3e64f04178baafd134d7f389e9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
输入 node -v，打印出版本则安装成功  

![](https://img-blog.csdnimg.cn/d9d15682d542432c98ab37f0048f26ea.png)

  
**注意，如果安装的目录需要管理员权限，那么你必须也要用管理员权限运行 cmd 控制台窗口，否则必然失败！**

方法就是右键已经打开的 cmd 图标，然后右键[命令提示符](https://so.csdn.net/so/search?q=%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6&spm=1001.2101.3001.7020)，点击以管理员身份运行  

![](https://img-blog.csdnimg.cn/28ad4c05771b4ce883632e1cfc065049.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  

![](https://img-blog.csdnimg.cn/7e2b00f8becf4b00966b1741d75e33f7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 3. 开始操作

找到 StartUML 的安装路径，进入 resources 文件夹  

![](https://img-blog.csdnimg.cn/2759f485ea4f43eda98d1f023aced1e8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

然后在控制台中进入这个文件夹  

![](https://img-blog.csdnimg.cn/2b37a7af9049448da2ddbdf241776143.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
先下载工具包

```
npm install asar -g
```

![](https://img-blog.csdnimg.cn/da2c26b50aba471297739197244c8483.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
然后解压

```
asar extract app.asar app
```

![](https://img-blog.csdnimg.cn/2574861336324a43b76c762369a28cbb.png)

  

![](https://img-blog.csdnimg.cn/f82881a7b2aa446d9eac6c607caa5958.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
回到文件夹，多了解压文件夹 app

此时可以删除掉 app.asar  

![](https://img-blog.csdnimg.cn/b4c324ec01d342f6be0fbc70a34bfe72.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
然后进入下图文件夹，找到 license-manager.js  

![](https://img-blog.csdnimg.cn/5666765dd7ee4a8fbba11a677dbee72c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
然后记事本打开即可  

![](https://img-blog.csdnimg.cn/da20514082bd4db4bd9dbffb96576151.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
找到图中的那一段代码，改成图中绿色框中所示，即注释掉注册框

```
//setStatus(this, false)
        //UnregisteredDialog.showDialog()
        setStatus(this, true)
```

保存后退出

来到下图位置，找到 application.js 文件  

![](https://img-blog.csdnimg.cn/df0f0c98e4fb4ee9b88c5b0b49dd79d0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
用记事本打开即可，ctrl+F 搜索 autoUpdater.check，找到下图位置  

![](https://img-blog.csdnimg.cn/433d98a5d5f74dcdaefb4ec6442af82c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
在图中两个位置，加上 //, 即绿色框所示

保存后退出

回到 cmd 控制台窗口，输入

```
asar pack app app.asar
```

![](https://img-blog.csdnimg.cn/bad65fdd3974456fb6dc8d0a1d6b1c0f.png)

  
回到文件夹 resource，打包成功 app.asar  

![](https://img-blog.csdnimg.cn/2e0153dc091545109a9d02b166c9e9b2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
删除 app 文件夹即可，**注意，如果还需要汉化的话，可暂时不用删除**  

![](https://img-blog.csdnimg.cn/4451da9b26ab4332b8552b344ebb5397.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
现在打开 startUML 软件，就发现不再弹对话框了

至此，安装完成

### 4. 中文

我没找到比较好的中文语言包，所以自能自己搞了

还是来到上面的 app 文件夹里面

在网上看到的路径分布：

*   常量字符串，主要在 src/strings.js 中。（这些会作为常量主要在 src 中的各个模块使用）
*   菜单，主要在 resources/default/menus/*.json 中。（扩展和插件目录的 menus 应该也会被识别为菜单项）
*   首选项，主要在 resources/default/preferences/default.json 中。（扩展和插件目录的 preferences 应该也会被识别为首选项）
*   规则验证提醒消息，主要在 resources/default/rules.js 中的 message:  
    提醒消息中。（扩展和插件目录的 rules.js 应该也会被识别为规则）
*   窗口页面，主要在 src\static\html-contents。（StarUML 应该是 Electron 这个 js 框架编写，其窗口是用 HTML 编写的）
*   其他文件，包括 Dialog, toast（这两个直接用 vscode 搜索 Dialog, toast 就可以），以及一些 js 文件中的英文等等

这里以 resources/default/menus/win32.json 为例  

![](https://img-blog.csdnimg.cn/339283e3c1f24e4c8ea34284ead78845.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

  
记事本打开即可  

![](https://img-blog.csdnimg.cn/1fffa961a5c14d109ce9a4a0cf44af3d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

这些字符是不是很熟悉！将 label 后面的英文改为中文即可，然后就是运行下面的指令进行重新打包

```
asar pack app app.asar
```

打开软件，成功！  

![](https://img-blog.csdnimg.cn/7f687037fe084803886a67e59965a730.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5L2Z6K-GLQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

## 下载现成的

可到本文末进入微信公众号回复：**StarUML**

将得到的文件，点击运行解压后，得到下面文件：

![](https://img-blog.csdnimg.cn/b8cf6d10892e4b39a19d3d3e15d0c94c.png)

包含最新的软件安装包，以及一个实用的脚本

安装好后，直接双击`install.bat`这个文件，即可完成软件的注册、中文翻译。