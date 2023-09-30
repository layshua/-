

## 1.UE5.2 源码版获取

1. 关联 Epic 账号和 Github 账号。因为 Epic 的 UE 引擎源代码库是私有库，需要加入 Epic 仓库才能拉取源代码，详细操作见官网链接：[GitHub 上的虚幻引擎 - Unreal Engine](https://www.unrealengine.com/zh-CN/ue-on-github "GitHub上的虚幻引擎 - Unreal Engine")

[![](https://csdnimg.cn/release/blog_editor_html/release2.3.6/ckeditor/plugins/CsdnLink/icons/icon-default.png?t=N7T8)](https://www.unrealengine.com/zh-CN/ue-on-github "GitHub上的虚幻引擎 - Unreal Engine")

[https://www.unrealengine.com/zh-CN/ue-on-github](https://www.unrealengine.com/zh-CN/ue-on-github "GitHub上的虚幻引擎 - Unreal Engine")

使用魔法，从 Github 上拉取下源码后，你会得到这样的一个文件目录：

![](https://img-blog.csdnimg.cn/e613b41df70e4252a115513db06832d5.png)

2. 找到 Setup.[bat 文件](https://so.csdn.net/so/search?q=bat%E6%96%87%E4%BB%B6&spm=1001.2101.3001.7020)，双击运行，开始下载各类依赖项，此环节只需保持网络畅通即可。别忘记提前给 UE 留出足够大的空间。（具体留根据版本不同可以自行百度。4.26 差不多 150-200G，5.2 差不多 200-300G）。这一过程差不多要下载 20G 左右的数据，主要是各类模板和第三方库之类的。

![](https://img-blog.csdnimg.cn/1859387981c341bd8cae6a840a2b65ac.png)

## 2.UE5 源码编译 (推荐使用 [VS2022](https://so.csdn.net/so/search?q=VS2022&spm=1001.2101.3001.7020)) 

1. 双击运行 GenerateProjectFiles.bat 文件生成 VS 工程文件。建议使用 VS2022，下载完工程文件后会在当前文件目录下生成 UE5.sln 文件。此时如果你缺少了一些必要的工具包，VS2022 会在解决方案栏上方提示你安装它们来获取完整体验。根据提示安装完成后，打开 UE5.sln 准备进行编译。

2. 将 UE5 设置为启动项，然后**调试** -> **开始执行。**之后就会开始漫长的调试过程

![](https://img-blog.csdnimg.cn/f7fbb2d907384fadbe8e0728ad4f31b0.png)

UE5 大概需要编译 6000 + 个文件。编译完成后可以在 Engine/Binaries/Win64 目录下找到刚生成的 **UnrealEditor.exe** 文件，双击运行。

![](https://img-blog.csdnimg.cn/fce435c1923d4a44a5710ed9834bd594.png)

## 3.UE5 源码版新建并打开项目

根据需求创建 UE 游戏相关模板即可。注意，如果要在 Android 平台上运行测试，建议如下选项：

**目标平台：移动平台**

**质量预设：可缩放**

不然在之后的打包时间会非常漫长

![](https://img-blog.csdnimg.cn/8d93e370b7e2466f9934ada9539969bd.png)

创建项目后，源码版不会打开 UE 编辑器界面，而是会跳出 VS 界面，此时如果要打开项目，选中所创建的项目，然后开始执行即可，等待编译完成后，刚刚创建的项目就会打开。

![](https://img-blog.csdnimg.cn/4275085eff924f47a6f1e2ea910aa771.png)

## 4.Android 平台打包（这一阶段才是重头戏）

官方文档有很详细的教程，建议刚开始先参考官方文档一步一步来 [https://docs.unrealengine.com/5.2/zh-CN/android-support-for-unreal-engine/](https://docs.unrealengine.com/5.2/zh-CN/android-support-for-unreal-engine/ "https://docs.unrealengine.com/5.2/zh-CN/android-support-for-unreal-engine/")

这里我来补充一些官方教程之外的东西。

在 **Android Studio 设置向导（Android Studio Setup Wizard）**之前如果出现一个弹窗，点击 Cancel，然后选择 Custom，之后的操作都是**默认选项**。

#### 1. **一般情况下，建议所有安装路径都选择默认路径，这样你就不用到处修改 SDK 路径**，但是如果你像博主一样**用户名里面含有中文**（下次长记性了）

![](https://img-blog.csdnimg.cn/8ed9a2e8e9d44382bf6a90bfd84b637c.png)

这一步它会提示你的路径里面含有非 ASCII 码，所以不能安装下用命名目录下。**此时可以修改安装路径。之后会讲到需要修改路径的位置。**

#### 2. 下面这一步虽然教程上说的是对于 UE 4.27、5.0 和 5.1 是必须的，但是如果你是 UE5.2，建议也完成这一步。

![](https://img-blog.csdnimg.cn/fa90d1d605c04ac5bc1048bba518e900.png)

如果你没找到 8.0 版本，勾选右下角的 **Show Package Details** 就可以下载 8.0 版本了。

![](https://img-blog.csdnimg.cn/6382729615b0470aae38db3b3fcb8463.png)

#### 3. 此时你就需要修改一下 SetupAndroid.bat 文件了。

①按照官网教程找到文件后，以文本文件格式打开文件，如果在上一步你没有修改默认路径，此时按照官网教程，将 latest 修改为 8.0 即可。但是如果你修改了路径，你需要**将 = 号后面的路径修改为你当前 sdkmanager.bat 的路径，参考我的路径：**

![](https://img-blog.csdnimg.cn/0f25e69daf0f4a709a6e1c5d1fd73411.png)

![](https://img-blog.csdnimg.cn/eaa4ee42744d47eb8565e480be094be4.png)

不然的话，要是双击 SetupAndroid.bat 文件，小黑窗会出现以下字样：

![](https://img-blog.csdnimg.cn/165e44809a8147f79a9d032807203fd7.png)

②如果你双击运行文件，出现以下字样：

![](https://img-blog.csdnimg.cn/becb2a4893374239ab916cee01ead015.png)

这就是第二个要修改的位置，找到最上面一句，set ANDROID_LOCAL，将其改为你在**第一步更换默认路径之后的文件位置。**

![](https://img-blog.csdnimg.cn/1baf4f9dfa294ce99dcfcda2fbe6b7c4.png)

③更改 Android 组件版本信息，在 SetupAndroid.bat 文件中找到以下语句：

![](https://img-blog.csdnimg.cn/e18d4d1b7e4f4994b978b00ddaab0d53.png)

实际上这段话表示的是此版本引擎发布 Android 所需组件的最低编译要求，批处理文件会按照这几项去要求 Android Studio 下载 UE 所需的组件。以上是博主修改好的参数，如果按照默认给你的参数，UE 版本可能与 Android 所需组件不匹配，打包时可能会报错。

建议此时的参数按照上图博主的参数进行修改（UE5.2）。如果你是其他版本，建议参考一下官网对应 UE 引擎需要的配置。

以上步骤都完成后，**记得保存文件，重启计算机**。然后**双击 SetupAndroid.bat 文件运行**，命令行窗口会开始执行批处理文件，**出现：成功：指定的值得到保存 请按任意键继续...。**表示这一步就 OK 了。没出现官方文档里的许可协议也没关系。

#### 4. 添加阿里云的国内镜像仓库地址配置，防止后面打包时下载依赖包失败。

先找到模板文件，参考文件目录 **C:\Program Files\Android\Android Studio\plugins\android\lib\templates\gradle-projects\NewAndroidProject\root** 找到 **build.gradle.ftl**

这个文件默认是不能够写入内容的，需要先修改写入权限，**右击 build.gradle.ftl→属性→安全→点击编辑→在组或用户名栏里选择你访问系统的用户身份 (User)→勾选权限栏里的完全控制的允许→然后全部点确定保存**。

用记事本打开 **build.gradle.ftl**，在目标位置添加配置

```
maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
```

![](https://img-blog.csdnimg.cn/0195eb05c91c4aa489484679dddd1cbd.png)

#### 5. 参考其他博主的经验

你需要在之前更改的路径下面添加一个文件，没更改过则使用默认路径，新建一个 repositories.cfg 文件。**有一说一，不知道这个文件有没有用，不过以防万一吧还是加着吧。**

![](https://img-blog.csdnimg.cn/11358bbd698444f5972b319ce7b8dbcf.png)

#### 6. 开始 UE 编辑器内部的设置

打开编辑器，**编辑 -> 项目设置 ->Android SDK，**如果是空白状态，需要手动配置一下。

![](https://img-blog.csdnimg.cn/2b56bc40e8e24fa7b4b7022cd4ebb84f.png)

**第一行参数：**

使用的默认路径：{User}/AppDate/Local/Android/Sdk

修改了默认路径，则填写你修改过后的路径。

**第二行参数：**

使用的默认路径：{User}/AppDate/Local/Android/Sdk/ndk / 版本号

修改了默认路径。则在修改路径下找到 ndk 下的带版本号的文件夹

**第三行参数：**

使用了默认路径：C：/Program Files/Android/Android Studio/jre

第四行参数：

可以在 Android Studio 查看你下载的 SDK 的版本号

![](https://img-blog.csdnimg.cn/add09e9ce76847858b2d1b555761a726.png)

这里建议使用 **30.0.0** 这个版本。所以**第四行参数**：**android-30**。**第五行参数：android-25**。也是在上述同一界面查看 ndk 版本。

上述步骤完成后，来到 Android 界面，点击下图两个位置的**立即配置。**然后将图一的 **Package game data inside apk 勾选。**

![](https://img-blog.csdnimg.cn/0ba707f04d424b5a87eb0b54217a7ea0.png)

![](https://img-blog.csdnimg.cn/9b9db58b86b447e88ee2795c3e8f6cbd.png)

#### 6. 打包！

![](https://img-blog.csdnimg.cn/874c02b71dbc46ec9498e2a577a65e53.png)

选择一个文件夹存储打包后的文件，之后会在对应文件夹生成 apk 文件，将此文件发送给手机下载后，即可开玩！

![](https://img-blog.csdnimg.cn/58ac14c9a3944a3f9438fc534c498826.png)