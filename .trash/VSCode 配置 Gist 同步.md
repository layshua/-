[(31条消息) VSCode配置 GitHub的Gist 同步设置和插件（简明教程）_vscode github插件_王张飞的博客-CSDN博客](https://blog.csdn.net/RNG_uzi_/article/details/120892625)

Settings Sync

gist:98d1902df98f32ee57d1082c41d4ba37

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

## 写在前面：

在本地电脑上的 [vscode](https://so.csdn.net/so/search?q=vscode&spm=1001.2101.3001.7020) 上配置了很全面的扩展（插件），由于后期可能要在不同的电脑设备上操作，所以希望可以将本地 vscode 上的扩展进行同步。  
在网上搜了很多关于 vscode 同步扩展（插件）到 [github](https://so.csdn.net/so/search?q=github&spm=1001.2101.3001.7020) 博客，但是基本都不是很全面，所以将自己成功操作的步骤进行总结，方便大家进行操作。

## （一）同步上传：VSCode 配置 GitHub 的 Gist 同步设置和插件信息

**第一步：github 上生成 Gist ID**  
因为我已经创建过 gist 了，所以在创建的话，第二步直接点右上角的 “+” 号。如果是第一次创建的话，在界面上会有提示: `creat new gist`  

![](<images/1686098156261.png>)

  

![](<images/1686098156380.png>)

  

![](<images/1686098156405.png>)

**第二步：配置 GitHub 获取令牌**  

![](<images/1686098156454.png>)

![](<images/1686098156504.png>)

  

![](<images/1686098156534.png>)

  

![](<images/1686098156571.png>)

  

![](<images/1686098156613.png>)

  

![](<images/1686098156646.png>)

![](<images/1686098156692.png>)

  
**第三步：配置 Vscode 先下载 setting sync 插件**  

![](<images/1686098156724.png>)

  
按住`Ctrl + Shift + P`，在输入框中输入`sync advanced options`  

![](<images/1686098156764.png>)

  

![](<images/1686098156820.png>)

  

![](<images/1686098156888.png>)

OK！到现在我们已经将所有文件配置完成了，接下来就是将本地的设置和扩展（插件）进行上传到 github 了

还是先按住`Ctrl + Shift + P`，在输入框中输入`sync update`  

![](<images/1686098156930.png>)

  
然后命令窗中出现如下界面，说明大功告成！  

![](<images/1686098156978.png>)

## （二）导入到本地：将配置文件导入到本地的 Vscode

（还没有在新电脑上尝试，挖个坑 0.0）  
我们还是先按住`Ctrl + Shift + P`，在输入框中输入`sync download settings`  

![](<images/1686098157010.png>)