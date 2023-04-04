## 描述

将当页面转换为 Markdown 并将其中的图片保存到本地，适合 Obsidian、Typora 用户使用。

## 地址

[https://simpread.ksria.cn/plugins/details/BvjrJA6eh5](https://simpread.ksria.cn/plugins/details/BvjrJA6eh5)

## 视频

<video src="https://user-images.githubusercontent.com/81074/173170371-d7915fbb-4843-43eb-a1ab-51ba39c87f42.mp4" control></video>

## 快捷键

`a t`

## 前提

此功能依赖于 **同步助手 1.0.2 版**，关于此版本的详细说明 [请看这里](https://zhuanlan.zhihu.com/p/525999909)。

## Obsidian 用户

如果你是 Obsidian 用户，且在使用 [导入到 Obsidian 插件](https://github.com/Kenshin/simpread/discussions/2902) 此功能也内置到了此插件中，可直接在此插件中使用，详细说明 [请看这里](https://github.com/Kenshin/simpread/discussions/2902#discussioncomment-2705604)。

## 使用

支持手动导出或加入自动化时导出，详细说明 [请看这里](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-2921855)。

## 选项
![[Pasted image 20230403202728.png]]
随便进入一个页面 e.g. [https://sspai.com/post/71576](https://sspai.com/post/71576) 并进入 **阅读模式 → 右下角 → 动作 → 插件触发器**

按照下图所示即可查看此插件的选项页。（注意：如果你的界面没有插件触发器，请根据此 [解决方案](https://github.com/Kenshin/simpread/discussions/2342) 修复。

[![](<assets/1680524038623.png>)

## 功能

1.  是否添加 Metadata，关于 Metadata 占位符 [请看这里](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-2920472)。

[![](<assets/1680524039628.png>)

2.  导出位置

[![](<assets/1680524040999.png>)

3.  [以 idx-title 的形式导出](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-2926944)
    
    [![](<assets/1680524041526.png>)
    
4.  [图片导出位置](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-3920696)
    
    [![](<assets/1680524042075.png>)
    
5.  [图片导出类型](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-3920120)
    
    [![](<assets/1680524042601.png>)
    

## 更新日志

### 1.1.2

*   `Add` 新的字符，主要针对双链笔记用户去掉了一些特别字符，包括：`[` `]` `#` [#5035](https://github.com/Kenshin/simpread/issues/5035)

### 1.1.1

*   `Fix` 修复非高级账户权限时导出异常问题。

### 1.1.0

此功能是 [同步助手 1.1.3](https://github.com/Kenshin/simpread/discussions/4049#discussioncomment-4192563) 增加的新功能，只能在 1.1.3 中使用。

*   `Add` [图片导出位置](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-3920696) 与 Typora 保存一直的四种类型。
    
*   `Add` [图片导出类型](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-3920120)，专门针对个别页面无法成功导出图片情况下，可使用 `Base64` 方案。
    

### 1.0.2

*   `Fix` 当安装了 [题图](https://github.com/Kenshin/simpread/discussions/2487) 插件并且使用 [Live Editor](https://github.com/Kenshin/simpread/discussions/2852) 修改标题后，导入时仍使用旧标题的错误。

### 1.0.1

*   `Fix` 当标题出现特殊字符时无法使用的错误。

### 1.0.0

*   快捷键 `a e` 支持
    
*   [自定义 Metadata](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-2920472)
    
*   [设定导出目录](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-2920749)
    
*   [以 idx-title 的形式导出](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-2926944)
    

## 关联

*   [利用 Hazel 将 textbundle 修改为 Obsidian 可以预览的文件](https://github.com/Kenshin/simpread/discussions/3241)
    
*   [巧用 Textbundle 格式让 Obsidian 更好的支持「包含图片子文件夹的 Markdown」](https://github.com/Kenshin/simpread/discussions/2360)
    

## 引申

安装插件或带有插件触发器的插件失效的 [解决方案](https://github.com/Kenshin/simpread/discussions/2342)。

 

# Metadata 用法

Metadata 只能用于全文导出，且内置了如下占位符

<table role="table"><thead><tr><th>占位符</th><th>描述</th></tr></thead><tbody><tr><td><code>{{url}}</code></td><td>当前页面的 URL</td></tr><tr><td><code>{{title}}</code></td><td>当存在稍后读时为稍后读标题，否则为原网页标题</td></tr><tr><td><code>{{org_title}}</code></td><td>原网页标题</td></tr><tr><td><code>{{un_title}}</code></td><td>稍后读标题，不存在稍后读时为 untitled</td></tr><tr><td><code>{{desc}}</code></td><td>当前页面的描述</td></tr><tr><td><code>{{cover}}</code></td><td>当前页面的题图，没有的话用第一张图替代</td></tr><tr><td><code>{{note}}</code></td><td>当前稍后读的备注</td></tr><tr><td><code>{{date}}</code></td><td>导入到 Obsidian 的时间</td></tr><tr><td><code>{{timestamp}}</code></td><td>导入到 Obsdian 的时间戳</td></tr><tr><td><code>{{now|yyyyddmmHHMMss}}</code></td><td>日期格式化 <a href="https://github.com/Kenshin/simpread/discussions/2902#discussioncomment-2590775" data-hovercard-type="discussion" data-hovercard-url="/Kenshin/simpread/discussions/2902/hovercard?comment_id=2590775">详细说明</a></td></tr><tr><td><code>{{tags}}</code></td><td>如果当前文章是稍后读的话，为当前稍后读的标签</td></tr><tr><td><code>{{#|tag| }}</code></td><td>定制标签的前 / 后缀 <a href="https://github.com/Kenshin/simpread/discussions/2902#discussioncomment-2590798" data-hovercard-type="discussion" data-hovercard-url="/Kenshin/simpread/discussions/2902/hovercard?comment_id=2590798">详细说明</a></td></tr></tbody></table>

 

# md + assets 格式

为什么要使用 `md + assets` 格式以及使用它们的优势是什么，简单的做了下介绍。

## Markdown 的「问题」

Markdown 包含的是在线图片，当图片失效时就无法显示，所以简悦引入了 `离线 Markdown` 方案。

## 离线 Markdown

Markdown 可以包含离线图片，也就是将图片转为 Base64 编码形式，存储在 Markdown 中以防止图片 404 的方案，如下图所示

[![](<assets/1680524043152.png>)

虽然避免了图片 404 问题，但因为 `.md` 过大会导致大部分 Markdown 都没法正常使用，我测试过：vs code、Obsidian、Typora 目前能较好的预览这种方式的只有 vs code。

## [Textbundle](http://textbundle.org/)

**同步助手 1.0.1** 中引入了导出 `.textbundle` 格式，大部分 Markdown 都支持这种格式，如：Bear、FANotes（下图所示）

[![](<assets/1680524043678.png>)

但更多的 Markdown 编辑器，如：Obsidian、Typora 均不支持此方式。

## md + assets 格式

Obsidian、Typora 虽然不支持 `.textbundle` 格式，但它们支持 `md + assets` 格式，如下图所示

[![](<assets/1680524044298.png>)

所以 **同步助手 1.0.2** 中引入了 `md + assets` 格式。

 

# 导出位置

[![](<assets/1680524044836.png>)

通常情况下，建议设置路径，如果不设置路径的话，需要 **在同步助手端设置增强导出的路径**，详细说明 [请看这里](https://github.com/Kenshin/simpread/discussions/2958#discussioncomment-2713824)。

增强导出的路径设置如下：

```
{"extension":"assets", "path":"/Users/***/Documents/Obsidian/SimpRead"}
```

## 注意

1.  **同步助手端设置增强导出的路径** 的优先级大于插件端，也就是说同时设定路径的话，以 **同步助手端设置增强导出的路径** 为主。
    
2.  此路径为绝对地址路径。
    

 

# 使用

## 手动方式

进入阅读模式，右下角触发器 → 动作 → 插件触发器 → 导出 md + assets 格式

[![](<assets/1680524045365.png>)

## 自动方式（当加入稍后读后自动导出）

安装 [自动化辅助插件](https://github.com/Kenshin/simpread/discussions/3596) 然后按照下图设置，即可。

[![](<assets/1680524045893.png>)

### 视频

<video src="https://user-images.githubusercontent.com/81074/173170371-d7915fbb-4843-43eb-a1ab-51ba39c87f42.mp4" control></video>

 

# 以 `idx-title` 的形式导出

当用户配置了 [知识库](https://kb.simpread.pro/#/page/%E5%BB%BA%E7%AB%8B%E7%9F%A5%E8%AF%86%E5%BA%93) 方案时，建议开启此功能，这样当加入稍后读后导出时，会以 `idx-title` 为标题。

[![](<assets/1680524046420.png>)

导出后的标题

[![](<assets/1680524046968.png>)

 

# 图片的导出类型

[![](<assets/1680524048501.png>)

支持 `HTTP` 与 `Base64` ，前者为默认值。

## `HTTP` 方案

将当前阅读模式的图片直接发送到同步助手中，在同步助手中保存到本地。

## `Base64` 方案

将当前阅读模式的图片先转换为 base64 格式，然后直接通过同步助手保存到本地。

一般用于：某些网页有一些限制导致无法使用 `HTTP` 保存图片，当使用后会出现下图所示的提示。

<video src="https://user-images.githubusercontent.com/81074/202834601-ee1271ed-aa76-4125-8050-442d00bd2d9f.mp4" control></video>

**注意**

不建议使用此方式，因为「前端转换图片」会比较慢。只有在无法使用 `HTTP` 方案时再使用。

# 关联

*   [通过简悦导出 B 站笔记到 Obsidian 时报错 #4510](https://github.com/Kenshin/simpread/issues/4510)
    
*   [towardsdatascience 无法使用 assets 方案导入 #4620](https://github.com/Kenshin/simpread/issues/4620)
    

 

# 图片导出位置

[![](<assets/1680524049028.png>)

图片的导出位置在结构上与 Typora 保持一致，包含四种类型：

1.  `/${filename}/assets`
    
    [![](<assets/1680524049556.png>)
    
2.  `./${filename}.assets` → 保存到 `标题.assets` 文件夹下
    
    [
    
    ![](<assets/1680524050794.png>)
    
    ](https://user-images.githubusercontent.com/81074/202833837-ff849bf8-e1d4-4f13-b558-2a362ed78faa.png)
3.  `./assets` → 无论是多少个文章，将图片统一全部都保持到 `assets` 文件夹下面
    
    [
    
    ![](<assets/1680524052178.png>)
    
    ](https://user-images.githubusercontent.com/81074/202833875-69a7d7cc-2e25-4898-ae79-05293382cf96.png)
4.  `custom` → 将图片保存本地的任意目录，详细说明 [请看这里](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-3930228)。
    

 

# 绝对路径与相对路径

[![](<assets/1680524053531.png>)

此功能仅当选择 [方案 ➍ 时](https://github.com/Kenshin/simpread/discussions/4041#discussioncomment-3930228) 可用

[![](<assets/1680524054522.png>)

## 使用绝对路径（启用此功能）

在 Typora 中预览效果

[![](<assets/1680524056755.png>)

## 使用相对路径（禁用此功能）

在 Obsidian 中预览效果

[![](<assets/1680524058509.png>)