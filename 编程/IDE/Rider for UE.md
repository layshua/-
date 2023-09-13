## 前言

Rider 是从 Rider for unreal engine 测试版本用到现在，不管是功能还是使用舒适度都比我之前使用 VS + 番茄插件好很多真的对，他爱不释手。

Rider 用好了真的能够极大的提高你的工作效率，接下来我会把一些实用的使用技巧分享给大家。

## 功能介绍

### 创建 UE 文件

选中目标文件右键：

![[f0f4a46eefd2bc0980b290f0d8a6aea3_MD5.jpg]]

选择一个类型之后，它就会自动帮你创建：

![[9543c89a3d3b6a5e861e44408995d667_MD5.jpg]]

如果不是像实现接口、Slate 那样，默认第一个就可以了，反正最后都要改。  
如果是像接口那样，点击到对应的地方之后编辑器会自动的帮你创建好你需要的东西，这样创建起来就会很方便，不需要再进入游戏，然后再进行创建文件了。

**过程麻烦？有快捷键滴 =>** Ctrl + Alt + Insert 在当前目录创建文件。


### **解决方案的 Configuration**

![[46ad87e344e5e0d01f70e9ce7ce88d2b_MD5.jpg]]

*   DebugGame 游戏模块开启调试模式，引擎模板开启优化。
*   Development 游戏模块和引擎模块均开启优化
*   Shipping 发行版本（会去掉所有编辑器功能、stat 统计以及 GM 命令等）游戏模块和引擎模块均开启优化

### 控制台输出

![[68d1a43c283a1b0223352862eb337442_MD5.jpg]]

点击之后就可以直接在编译器中打开控制台运行。

**题外话**： 直接在地址上输入 cmd + enter 也能直接打开对应地址的控制台：

![[d42356d90b0bdd1968dc90ae876ec2e7_MD5.jpg]]

![[1b0c284159a4e43a5202236775b5b5d3_MD5.jpg]]


### 刷新项目

**有时候在 Rider 之外往项目中删除或者添加了一些文件，就会导致有个感叹号，需要刷新一下，按下右上角的刷新键就会恢复了**。

![[edfeb6d2cfeae99ca5ba2096142be56b_MD5.jpg]]

## 插件推荐

### git

重要的也就这三个按钮：

![[1bde2a199f1036fda3b2c70bbe58c5ff_MD5.jpg]]

下拉的时候需要注意弹出来的窗口提示，第一个是合并分支，第二个是将当前这个代码全部覆盖之前分支中的代码。

### EzArgs

EzArgs 就是对虚幻自带的 UnrealVS 插件的一个封装，主要功能就是帮助我们加载一些参数，然后跳过一些没有必要的步骤。  
这个插件在虚幻的源码安装路径中可以找到：

![[e1b7d69d61ce182ced348e061d665eb8_MD5.jpg]]

使用 VS 安装上面那个就行了，Rider 在插件市场可以直接搜索到：

![[96bdd4ce0cb0fcd178a84ba1ebcdde9a_MD5.png]]

![[f341257c57fb28fa9e82a936332cbff1_MD5.jpg]]

安装重启之后在工具栏就可以看到：

![[e9dd16ec9b81afcae12feb57658f7a96_MD5.png]]

在这个里面就可以输入一些命令行参数，在启动的时候使用。  
参数命令可以看看官方文档： [命令行参数](https://docs.unrealengine.com/4.26/zh-CN/ProductionPipelines/CommandLineArguments/)，更详细的可以看这个：[UE4 命令行参数 - 可可西 - 博客园](https://www.cnblogs.com/kekec/p/14952261.html)。

这里介绍几个常见的参数：

*   `-skipcompile` ： 启动时不检查模块代码是否再需要编译（一般运行的时候就已经带上了，不需要加）。  
    注意看下方 Run 窗口中的命令行，如果没有，就加上，或者在项目中进行配置。
*   `-game` : 有些时候我们并不需要进入 Editor 模式，只是想进入游戏看看修改代码之后的效果，这个时候就可以输入这个命令直接进入单机游戏模式。
*   `-game -server -log` : 启动本地 ds。
*   `127.0.0.1-game` ： 启动并联网加入本地 ds。
*   `-game -Windowed` 单机以窗口模式运行
*   `-game -FullScreen` 单机以全屏模式运行

### Rider Link

安装了 Rider Link 插件之后，这里的功能就能够使用了，就是可以在编辑器开始游戏了，不用在移动到编辑器中打开进行运行了，这个功能主要在打断点看运行效果的时候会用到。

![[71fd929b4119a4b136d73646a33d8fed_MD5.jpg]]

编辑器中 Log 信息也能在编译器中看到：

![[53106172af254c6c50d1d67d5b7b5d2c_MD5.jpg]]

用处：当项目打断点的时候，看不到编辑器中的打印信息，使用插件之后就在编辑器中直接看到了。

当然，这个 log 也可以在文件中找到：

![[0538fcdd21648f6a0a97f6e1e1729a48_MD5.jpg]]

## 快捷键

像注释、补充头文件、代码补全等这种使用非常频繁的快捷键就不讲了。

### 常用快捷键

<table data-draft-node="block" data-draft-type="table" data-size="small" data-row-style="normal"><tbody><tr><th>功能</th><th>按键</th></tr><tr><td>声明和实现相互切换</td><td>鼠标中键</td></tr><tr><td>格式化</td><td>Ctrl + Alt + L</td></tr><tr><td>在当前目录下创建文件</td><td>Ctrl + Alt + Insert</td></tr><tr><td>切换文件</td><td>Ctrl + Tab</td></tr><tr><td>Build 项目</td><td>Ctrl + F9</td></tr><tr><td>Debug 项目</td><td>Shift + F9</td></tr><tr><td>Run 项目</td><td>Shift + F10</td></tr><tr><td>多光标<br>按同一个光标两次就会取消光标</td><td>Shift + Alt + 左键</td></tr><tr><td>搜索项目所有文件</td><td>Shift + Shift</td></tr><tr><td>搜索项目所有内容</td><td>Ctrl + Shift + F</td></tr><tr><td>搜索当前文件函数并移动到对应位置</td><td>Ctrl + F12</td></tr></tbody></table>

### 生成实现函数

按下 **Alt + Insert**

![[0d6b97d814c22be0475144dc6b882abf_MD5.jpg]]

点击 Select All，然后点击确定一键全给你实现了。

![[f1b59f70422b9406b8d1e0b6edc3e557_MD5.jpg]]

### 断点

<table data-draft-node="block" data-draft-type="table" data-size="small" data-row-style="normal"><tbody><tr><th></th><td></td></tr><tr><th>重新运行项目</th><td>Ctrl + F5</td></tr><tr><th>终止项目</th><td>Ctrl + F2</td></tr><tr><th>项目继续运行</th><td>F9</td></tr><tr><th>单步执行<br>不进入子函数</th><td>F8</td></tr><tr><th>单步执行<br>不进入子函数</th><td>F7</td></tr><tr><th>打断点</th><td>Ctrl + F8</td></tr></tbody></table>

![[d58a5fdb1b2def009fca75f6994052d1_MD5.png]]

左边这个显示所有断点信息，并可以控制它是否启用，有代码预览功能。

![[0729a79090a962496e6598334238a2e6_MD5.jpg]]

右边这个就是将当前所有的断点都屏蔽掉。

注意到上面有个一些参数，可以调，加入说有个循环代码，前 3 次都没有用，就是在浪费时间，只要看 4 之后的代码数据，如果循环代码非常长的话是非常浪费时间的，那么就可以使用下面这个条件判断的功能，直接到第四次：

![[c6fff48705294d47955f32aaecb9dc15_MD5.jpg]]

![[73d74835e3a4e885ae7653930c8efa38_MD5.jpg]]

### 移动到标签所在位置

按下 **F11** 就会在当前位置打下一个标记，可以打在注释上面，这样就可以说明这个标签意思，方便后面查找以及阅读。

按下 **Ctrl + F11** 可以增加一个数字 / 字母标签，可以添加描述。

![[0ef74329aa48ade94d2e380d3e5e9599_MD5.jpg]]

比如说我们这行按下了点击了 **1**， 那么我们再次按下 **Ctrl + 1** 的时候直接跳转到了这个 **1 的位置**。  
这个功能在需要多个位置来回跳转的时候简直就是神器，效率直接 ↑↑↑↑↑。

在打下标签的那一行再按一次 F11 标签就会取消了。

可以在左边的 Bookmarks 界面看到所有的标签以及断点。

![[cae6b9de6a201373e311bd961c54307b_MD5.jpg]]

### 打开导航栏

按下 **Alt + Home** 键，出现当前文件的导航栏：

![[1451f066b135617167378cb486a909dc_MD5.jpg]]

### 打开历史栏（显示最近打开的文件）

按下 **Ctrl + E**, 就会显示出来：

![[a75cb794e455e72b6cee1402b3555d04_MD5.jpg]]

### 打开当前文件的问题清单

按下 **ALT+6** 就可以打开这个界面，处理 Rider 给出的提示。

![[f0db8e7a5c2dfb33c90631d8927cfec2_MD5.jpg]]

并且支持过滤以及排序功能：

![[17fd9ae70d431f0ae1dccd233f8be3f6_MD5.png]]

### 打开 git 变化清单

按下 **ALT+9** 就可以打开这个界面，知道自己在什么地方进行了修改。

![[3d2a12408874554cfedce1e399d72221_MD5.jpg]]

### 关闭窗口 / 划分窗口

我是自定义的一个按键，Ctrl + Alt+ W, 按下之后会关闭其他的窗口，非常方便。 因为平时关网页关聊天框都是用的 Ctrl + W 快捷键，这样设置之后就方便进行记忆，同时也提高了我的效率，不用手动去点击这个按钮了。

![[371eea753959e699b07bd074fc3fa5df_MD5.jpg]]

平时有些时候需要在同一个文件不同函数之间比较，就需要划分成两个窗口，所以我给划分窗口也分配了一个 Alt + E 的快捷键。

### 其他快捷键

<table data-draft-node="block" data-draft-type="table" data-size="small" data-row-style="normal"><tbody><tr><th>移动选中块 / 当前行</th><td>Shift + Alt + ↑ 或 ↓</td></tr><tr><th>光标移动到上下函数位置</th><td>Alt + ↑ 或 ↓</td></tr><tr><th></th><td>Ctrl + Home</td></tr><tr><th>光标移动到文件结束处</th><td>Ctrl + End</td></tr></tbody></table>

## 快捷片段输出

在写代码的时候有些东西很相似又很常见，但是格式太复杂有点浪费时间又不得不写，比如说调用 UE_LOG 进行输出，核心内容就是输出的东西，这个时候可以使用 Live Template 去解决这个问题。

![[31b15598556252a6f519c3f2f575f720_MD5.png]]

*   **uelog : `UE_LOG(LogTemp, Warning, TEXT("%s %s %d ==> $END$"), *FString(__FUNCTION__), *FString(__FILE__), __LINE__);`**

![[05147857d8490c6296b49cc2025f1ea8_MD5.jpg]]

![[a67fa84c5410dec227e43337d1d4b924_MD5.jpg]]

这样打印东西就非常的快速，输出的内容可以根据自己的需求进行修改。
