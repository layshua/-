
构建后运行报错
Unable to parse Build/Output.framework.js.gz! This can happen if build compression was enabled but web server hosting the content was misconfigured to not serve the file with HTTP Response Header "Content-Encoding: gzip" present. Check browser Console and Devtools Network tab to debug
![[Pasted image 20230606172742.png]]
解决办法：PlayerSetting->Player->Publishing Setting

把DecomPression Fallback勾选上

![](https://img-blog.csdnimg.cn/6b6384c133544c829dce1e4ab7e25d94.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6JKZ5Y-M55y855yL5LiW55WM,size_20,color_FFFFFF,t_70,g_se,x_16)

这个时候发布出来就能正常运行，但是它不是全屏的：