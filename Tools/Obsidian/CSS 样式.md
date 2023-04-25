
---
title: CSS 样式
aliases: 
tags: []
create_time: 2023-04-25 22:57
banner: "![[1678031998694.png]]"
banner_y: 0.4625
banner_x: 0.51302
---

## 001：url 对应的地址

```
background: url("app://local/...绝对地址.../..图片..")；
```

我只找到绝对地址的位置，在地址前加 “app://local/”，而且要将 Win 复制地址中的斜杠改成 “/”；

## 002：更改预览模式下的 YAML front matter 样式

![](1678031998748.png)

```
.frontmatter-container {...}
```

将中括号中的 "..." 替换成你想要的 CSS 样式

![](1678031998788.png)

## 003. 普通字体颜色

```
/* 普通字体 */
body {
  color: #eef7f2;  /*这里就是普通字体的颜色*/
}
```

## 004.# 标题的样式（.cm-header-1 的数字 “1” 就表示一个 “#” 的样式）

```
.cm-header-1 {
  text-shadow: 2px 2px 2px rgba(0, 0, 0, 0.8);   /*--------字体阴影--*/
  color: rgb(25, 130, 180);                      /*--------字体颜色--*/
  background-color: #def700;                    /*--------背景颜色--*/     
  border-radius: 5px;                           /*--------四边倒圆角--*/
  padding: 2px 5px;                             /*--------文字距边缘的距离--*/

}
```

![](1678031998829.png)

.cm-header-1 的数字 “1” 就表示一个 “#” 的样式；

.cm-header-2 的数字 “2” 就表示两个 “#” 的样式（即“##”）；

以此类推，改变数字就可以；

## 005. 标题前端的箭头

```
.collapse-indicator.collapse-icon {       /*--------这是未折叠的样式----*/
content: url("data:image/svg+xml,<svg></svg>")        /*-----将<svg></svg>替换成你要的SVG图标吧---*/
}
```

折叠前后的样式，我是将内容替换成我的 SVG 图形，可以去网上找 SVG 图标，替换到这个位置；

要注意的是，不知为何，颜色只能用 "rgb（X,X,X)"，使用 "#XXXXXX“会不显示的。

```
.cm-fold-indicator.is-collapsed .collapse-indicator.collapse-icon,
.is-collapsed .collapse-indicator.collapse-icon {                    /*--------这是折叠后的样式----*/
content: url("data:image/svg+xml,<svg></svg>")        /*-----将<svg></svg>替换成你要的SVG图标吧---*/
}
```

## 006. 引用内容的样式，文字与框框，我只能分开实现；

```
/* 引用框中的文字颜色 */
span.cm-quote.cm-quote-1 {
  color: rgb(156, 246, 0);
}

/*编辑模式/引用文本的外框*/

div:not(.CodeMirror-activeline)>.HyperMD-quote {
  background-color: rgb(49, 48, 49);
  border-radius: 0 8px 8px 8px;
  border-left: 4px solid #41b349;
  border-top: 4px solid #41b349;
  border-right: 2px solid #41b349;
  border-bottom: 2px solid #41b349;
}
```

![](1678031998861.png)

## 007. 无序列表与有序列表，开头的样式 + 后边整体

由于我只会修改头部加尾部的样式，所以分成两段；

首先头部分为有序，无序

有序样式，如下代码

```
.HyperMD-list-line .cm-formatting-list-ol {...} /*这里修改背景颜色之类的*/
span.cm-formatting.cm-formatting-list.cm-formatting-list-ol {...}/*这里仅仅修改字体颜色比较好*/
```

无序样式，如下代码

```
.HyperMD-list-line .cm-formatting-list-ul {...} /*这里修改背景颜色之类的*/
span.cm-formatting.cm-formatting-list.cm-formatting-list-ul {...}/*这里仅仅修改字体颜色比较好*/
```

其次是尾部的一整条，如下代码

```
.HyperMD-list-line .cm-list-1 {   /*-----这是整体后面的所有内容-----*/
  color: #fecc11;
  text-shadow: 2px 2px 2px rgba(0, 0, 0, 0.8);
  background-color: #00ffa6b6;
  padding: 0px 5px;
}
```

这里最后的”.cm-list-1“的数字”1“就表示第一层列表；依次类推；

”-“+ 空格 是第一层列表；

”TAB 键 “+”-“+ 空格 是第二层列表；

”TAB 键 “+”TAB 键 “+”-“+ 空格 是第三层列表；

![](1678031998893.png)

```
此前我也照网上修改过”.cm-formatting-list-ul+span{。。。}“，
但是在其中，增加别的内容后就断掉了，把+改了就不识别了，不知道为什么？（请大佬指教）由于它识别的是以一个span把？
```

## 008. 链接的样式，

```
/* 链接cm-link中括号中的内容 */
.cm-s-obsidian span.cm-link {
  color: #37dab2;
}

/* 链接cm-link 小括号中的内容*/
span.cm-string.cm-url {
  color: rgb(255, 225, 0);
  cursor: pointer;
}

/* 链接cm-link 触摸小括号内容后*/
span.cm-string.cm-url:hover {
  background-color: rgb(89, 136, 8);
  color: #54ff7c;
}
```

![](1678031998935.png)

## 009. 粗体与斜体

```
/* 加粗cm-strong */
.cm-strong {
  color: #27f5b6;
  text-shadow: 1px 1px 5px #fecc11;
}

/* 斜体---斜体cm-em */
.cm-em {
  color: rgb(45, 215, 240);
  text-shadow: 2px 2px 5px #fecc11;
}
```

![](1678031998968.png)

## 010. 高亮文本

```
.cm-s-obsidian span.cm-formatting-highlight,
.cm-s-obsidian span.cm-highlight {
  background-color: rgb(255, 170, 0);
  color: rgb(0, 106, 255);
  text-shadow: 1px 1px 2px #00ffff;
}
```

![](1678031999007.png)

## 011. 标签的样式

```
.tag:not(.token) {   /*正常状态下*/
 background-color: rgba(0, 242, 255, 0.8);  /*背景颜色*/
 color: white;  /*字体颜色*/
 padding: 1px 8px;  /*字体距边缘*/
 margin: 0px 1px;  /*标签与标签的间距*/
 cursor: pointer;   /*鼠标变点击样式*/
 border-radius: 5px;  /*倒圆角*/
 border: 1px solid #f6f6f6;   /*边框颜色*/
}

.tag:not(.token):hover {       /*鼠标触摸后*/
 color: rgb(0, 255, 153);
 background-color: rgba(0, 174, 255, 0.8);
 text-shadow: 0 0 2px rgb(255, 255, 255);
 box-shadow: 0 0 5px 1px rgb(60, 124, 201);
 border: 1px solid #f6f6f6;
  ;
}
```

![](1678031999044.png)

## 012. 文件与文件夹名称前加一个图标做区分，与本文的 005 一样，替换一个 svg；

下述的 SVG 来自于 [iconfont - 阿里巴巴矢量图标库](https://link.zhihu.com/?target=https%3A//www.iconfont.cn/)，**如有侵权，请联络，立即删除；**

```
/* 文件-----前面的图标 */
.nav-folder-children .nav-file-title-content:first-child::before {
  content: url("data:image/svg+xml,<svg viewBox='0 0 1024 1024' xmlns='http://www.w3.org/2000/svg' width='20' height='20'><path d='M788.5 509L627 833.8l9.9 124 104.7-67.1L903 566l-114.5-57zM845.4 394.4l-38 76.4 114.6 57 38-76.4-114.6-57zM160 896c-17.7 0-32-14.3-32-32V160c0-17.7 14.3-32 32-32h543.1c17.7 0 32 14.3 32 32v256c0 17.7 14.3 32 32 32s32-14.3 32-32V128c0-35.3-28.7-64-64-64H128c-35.4 0-64 28.7-64 64v768c0 35.3 28.7 64 64 64h416.6c17.7 0 32-14.3 32-32s-14.3-32-32-32H160z' p-id='3493' fill='rgb(255, 170, 0)'></path><path d='M623.1 255H239c-17.7 0-32 14.3-32 32s14.3 32 32 32h384.1c17.7 0 32-14.3 32-32 0.1-17.7-14.3-32-32-32zM623.1 447H239c-17.7 0-32 14.3-32 32s14.3 32 32 32h384.1c17.7 0 32-14.3 32-32 0.1-17.7-14.3-32-32-32zM431.1 639H239c-17.7 0-32 14.3-32 32s14.3 32 32 32h192.1c17.7 0 32-14.3 32-32s-14.3-32-32-32z' p-id='3494' fill='rgb(121,85,72)'></path></svg>");

}


/* 文件夹----前面的图标 */
.nav-folder-children .nav-folder-title-content::before {
  /* content: url("data:image/svg+xml,<svg viewBox='0 0 1024 1024'  xmlns='http://www.w3.org/2000/svg' width='25' height='25'><path d='M912 208H427.872l-50.368-94.176A63.936 63.936 0 0 0 321.056 80H112c-35.296 0-64 28.704-64 64v736c0 35.296 28.704 64 64 64h800c35.296 0 64-28.704 64-64v-608c0-35.296-28.704-64-64-64z m-800-64h209.056l68.448 128H912v97.984c-0.416 0-0.8-0.128-1.216-0.128H113.248c-0.416 0-0.8 0.128-1.248 0.128V144z m0 736v-96l1.248-350.144 798.752 1.216V784h0.064v96H112z' fill='rgb(121,85,72)'></path></svg>"); */
  content: url("data:image/svg+xml,<svg t='1656895780474' class='icon' viewBox='0 0 1322 1024' version='1.1' xmlns='http://www.w3.org/2000/svg' p-id='6924' width='22' height='22'><path d='M0.644034 173.878853C143.842294 430.462765 120.871924 733.476146 230.347727 999.834683c7.8197 19.54925 37.632307-9.774625 41.053426-20.037982C335.424948 771.108453 342.755917 537.983642 413.621949 334.182706c44.963276-129.513784 128.536321-141.732065 260.005031-152.484153a332.337257 332.337257 0 0 1 223.350186 7.8197l2.932387 130.979978c0 6.353506 8.308431 5.864775 12.218282 5.376044 153.950347-20.526713 379.255458-93.836402 304.479575 132.446171A1605.970921 1605.970921 0 0 1 1100.778101 684.60302c-95.302596 175.454522-185.717879 220.90653-377.789264 244.36563l-482.377754 60.113945c-15.150669 0-43.985813 38.121038-16.616863 34.69992l579.146544-71.843496c64.023795-7.8197 195.492504 2.932388 255.117718-48.873126s63.535064-173.499597 86.505433-232.63608c56.692826-146.619378 162.74751-262.448687 176.920716-420.797615 0-8.308431-4.887313-10.263356-11.72955-9.285894-53.760439 7.330969-234.591005 54.737901-285.419056 37.632307-94.813865-31.767532-72.820958-72.820958-97.746252-152.972884a7.8197 7.8197 0 0 0-7.8197-5.864775c-156.394003-17.105594-347.976657 48.873126-501.438274 73.309689-10.263356 0-27.368951 14.173207-28.835144 25.414025a3880.526207 3880.526207 0 0 1-66.956183 437.414478c-19.060519 88.460358-48.873126 395.872321-107.520877 150.040497C158.0155 579.525799 155.083113 351.2883 40.231266 144.066246c-7.330969-13.195744-48.873126 14.661938-38.609769 30.301338z' p-id='6925' fill='rgb(18, 105, 171)'></path><path d='M17.749628 155.307065C216.663251 124.516996 415.576874 98.125508 614.979229 70.756557c83.084314-10.263356 195.492504-44.963276 279.554281-32.744994 188.650266 28.835144 151.017959 105.565952 157.371465 240.45578 0 16.128132 42.030888-7.330969 41.053426-23.947832-8.797163-185.229148 21.015444-273.200775-181.808029-251.207868-293.238756 31.767532-586.477512 71.354764-876.783881 117.295502-15.150669 0-43.497082 39.098501-16.616863 34.69992z' p-id='6926'  fill='rgb(67, 178, 131)'></path></svg>");

}
```

注意！！颜色只能用 "rgb（X,X,X)"，倘若使用 "#XXXXXX“会不显示的。一般是 fill="..." 中的内容吧

![](1678031999081.png)

## 013. 右下角状态栏，增加淡出效果，转载于 ----> [社区大佬的回复](https://link.zhihu.com/?target=https%3A//forum.obsidian.md/t/meta-post-common-css-hacks/1978/10)

```
/* 右下状态栏淡出 */
/* auto fades status bar items */
.status-bar:not(:hover) .status-bar-item {
  opacity: 0.25;
  transition: opacity .25s ease-in-out;
}
```

## 014. 左边功能栏的图标的颜色

```
.side-dock-ribbon-action {
  color: rgb(156, 246, 0);
  transition: color .25s ease-in-out;
}

.side-dock-ribbon-action:hover {
  color: rgb(255, 166, 0);
}
```

![](1678031999119.png)

## 015. 滚动条的颜色

```
body:not(.native-scrollbars) ::-webkit-scrollbar {
  background-color: rgb(0, 255, 128);
}
/* 滚动条 */

body:not(.native-scrollbars) ::-webkit-scrollbar-thumb {
  background-color: rgb(156, 246, 0);
  border: 2px solid rgb(0, 0, 0);
}
/* 滚动条，动的那部分 */
```

## 2022-07-09 补充【被选中文件底色 & 鼠标置于文件名上时的信息】

**被选中文件底色**：目前我找到的是分为两部分改，上面只改背景，下面只改文字；

```
body.folder-style-change-options-colorful .nav-folder.mod-root>.nav-folder-children .nav-file-title.is-active {

 background-image: linear-gradient(120deg,rgb(0, 157, 255), rgb(251, 105, 0), rgb(0, 255, 94), rgb(0, 157, 255)) !important;

}

body.folder-style-change-options-colorful .nav-folder.mod-root>.nav-folder-children .nav-file-title.is-active .nav-file-title-content {
  
color: rgb(255, 255, 255);
text-shadow: 1px 1px 1px rgb(0, 0, 0) !important;
}
```

![](1678031999155.png)

**鼠标置于文件名上时的信息**：只改文件栏目的浮动窗口的话，“.tooltip.mod-right”；

```
.tooltip.mod-right {
  /*---仅仅左侧功能栏和文件栏--*/

  text-shadow: 1px 1px 3px rgb(0, 0, 0);

  color: white;

  box-shadow: 0 0 5px 1px rgb(128, 167, 216);

  border: 2px solid #f6f6f6;

  font-weight: bold;

 /* overflow: hidden; *//*一般会有个小三角，不会改它的话，可以把这个启动*/

}
```

一般会有个小三角，具体改法参考：[css border 三角形阴影 (不规则图形阴影) & 多重边框的制作](https://link.zhihu.com/?target=http%3A//t.zoukankan.com/sspeng-p-6443542.html)

比较不好弄，我也是一知半解；不过可以把它裁剪掉，用【overflow: hidden;】。

