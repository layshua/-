[Metadata - Obsidian Help](https://help.obsidian.md/Editing+and+formatting/Metadata)

*   需要给一个笔记取多个名称（别名）。
    *   不想让标签出现在正文中，破坏正文格式。
    *   想给笔记加上诸如创建时间、笔记进度等各种属性，方便查找和管理笔记。



## 1. 什么是 YAML front matter？

[YAML](https://link.zhihu.com/?target=https%3A//baike.baidu.com/item/YAML/1067697) ：是一种表达数据序列化的格式。

**Front matter**：直译为 “前置内容”，它是基于 YAML 格式的纯文本内容，放置在文档开头，用于标明文档的各种属性（元信息）。



## 2. Front matter 的格式

**“前置内容” 当然得放在最前面！**

```
---
key: value
key: [one, two, three]
key:
- 1
- 2
- 3
---
```

*   用 `---` 标明 Front matter 的起始位置。
*   其有效内容以 “**键值对**” 的形式呈现：

*   `key`（键）：属性名
*   `value`（值）：属性值

*   一个键可以对应一个值也可以对应多个值，其表示方法如上所示。
*   **注意：**Front matter 里的冒号是英文 “:”，而且冒号后**要加空格**才能让键值对生效。

## 3. Front matter 的用法

我将基于本文开头提到的问题，展示目前在 Obsidian 中，我对 Front matter 的应用。（以我写的这篇文档的 front matter 为例）

当前的 Obsidian 包含三个原生的 key：`tags`、`aliases` 和 `cssclass`。除此之外，我们还可以人为添加 key，比如 `time`、`progress` 和 `简介`。

```
---
aliases:
- YAML front matter
tags:
- Obsidian
time: 2023-01-12 09:51
progress: 进行中
简介: Obsidian 中 YAML front matter 的用法。
---
```

*   `aliases`：别名  
    我给这篇笔记取了一个别名叫 “YAML front matter”，方便我在其他笔记中引用这篇笔记。
*   `tags`：标签  
    为了防止标签出现在正文中，破坏文章结构，这里我将它放到了 Front matter 中，而且这么做还有利于统一查看和管理一篇笔记的标签。
*   `time`：创建时间  
    创建这篇笔记的日期和具体时间，结合模板使用。（后面会讲到）
*   `progress`：笔记进度  
    共有 “未开始”、“进行中”、“已完成” 和“已放弃”四种属性值，用于任务管理。
*   `简介`：用一句话介绍笔记内容。

如果要在[阅读视图](https://www.zhihu.com/search?q=%E9%98%85%E8%AF%BB%E8%A7%86%E5%9B%BE&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2845528185%7D)下查看 Front matter 的话，首先需要在 “设置—编辑器—显示” 中打开“显示 Front-matter”，然后将文档切换为“阅读视图”，就能在文档开头看到“Metadate”（[元数据](https://www.zhihu.com/search?q=%E5%85%83%E6%95%B0%E6%8D%AE&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2845528185%7D)）了。

![](<assets/1682424385569.png>)

如果我需要为每一篇笔记都加上这些属性，显然就得用到模板功能，于是我设计了一个 “通用模板”：

![](<assets/1682424386769.png>)

*   `time`：这里用到了模板功能的变量特性。

*   模板文件中的 `{{date}}` 字段在插入该模板时会被替换为当前日期；
*   [模板文件](https://www.zhihu.com/search?q=%E6%A8%A1%E6%9D%BF%E6%96%87%E4%BB%B6&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2845528185%7D)中的 `{{time}}` 字段在插入该模板时会被替换为当前时间。
*   前文 Metadata 中的 time 属性值就是一个例子。

*   `progress`：分为四个类型，用于任务管理。  
    具体任务管理模式见：任务看板——Obsidian 白板功能的妙用（后续更新）

现在我写任何一篇笔记，都会先插入 “通用模板”，依次来为我的笔记添加别名、标签等各种属性，以此来规范笔记的基础内容。

* * *

**总结：**在 Obsidian 中，Front matter 可以为笔记添加属性，并以元数据的形式呈现，不会干扰正文内容。借此配合 “模板” 功能，就能很方便地管理和搜索笔记。

![](<assets/1682424388810.png>)

画画的阿杜

这个教程有教，可以看看 [https://www.bilibili.com/video/BV14T41157Vt?t=190.5](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV14T41157Vt%3Ft%3D190.5)