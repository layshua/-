**目录**

[一、下载插件](#t0)

[二、插件配置](#t1)

[注意](#t2)

[三、插件使用](#t3) 

**欢迎大家来学习和使用 LeetCode 力扣插件！！！**

这几天准备刷算法题，突然发现了 [IDEA 插件](https://so.csdn.net/so/search?q=IDEA%E6%8F%92%E4%BB%B6&spm=1001.2101.3001.7020)商城里面有 LeetCode 力扣的插件，于是就下载下来尝试着使用了一下，发现用着很方便，尤其是对于我们这种经常使用 IDEA 编程工具的这部分来说，是非常的节省时间。插件的具体样式如下图所示：

![[085282c1d2d8be08e333a3f6c97e6060_MD5.png]]

像这样的插件，点击想要刷的题，右边会自动弹出题目要求和代码编译器。直接在右边写算法代码了，不需要再去浏览器中搜索网址进行算法题的编写和学习。 例如：我们点击最基础的 **两数之和** 来做一个示范，如下图所示：

![[0328af45955834dfa9d80a27b51aff8a_MD5.png]]

像这样的插件如何使用呢，接下来进行详细的讲解：

# 一、下载插件

打开 idea 编译软件，点击设置 -- 进入插件商城 -- 搜索 leetcode 插件 -- 点击下载就可以了（点击右上角的 File-->Setting-->Plugins）。如下图所示：

![[884d88951a19414ffa0f72a3f262f5f0_MD5.png]]

# 二、插件配置

插件安装之后，继续点击 File-->Setting-->Tools->leetcode plugin，如下图所示：

![[62dab419bef612b16384392bbb8ccde0_MD5.png]]

打开这个窗口之后，我们接下来的工作就是进行完成对插件配置的工作了。具体的操作全部展示在下图中：

![[76c6d12b953cccf1ca95c816b3999d46_MD5.png]]

## **注意** 

如果要是想专门刷算法题，我的建议是：重新建一个项目来专门存放算法源代码。

关于部分参数的定义：

*   ${question.title}    题目标题    示例: 两数之和
*   ${question.titleSlug}    题目标记    示例: two-sum
*   ${question.frontendQuestionId}    题目编号
*   ${question.content}    题目描述
*   ${question.code}    题目代码
*   $!velocityTool.camelCaseName(str)    转换字符为大驼峰样式（开头字母大写）
*   $!velocityTool.smallCamelCaseName(str)    转换字符为小驼峰样式（开头字母小写）
*   $!velocityTool.snakeCaseName(str)    转换字符为蛇形样式
*   $!velocityTool.leftPadZeros(str,n)    在字符串的左边填充 0，使字符串的长度至少为 n
*   $!velocityTool.date()    获取当前时间 

# 三、插件使用 

点击 leetcode，进入题目列表，并且点击登录按钮，进行账号登陆。如下图所示：

![[7a0eb8d4f690158eb91e33f8e0d9ca0f_MD5.png]]

将自己的账户登陆上去之后， 就可以进行对算法题的编写和解答了。点击一个题目（如 **两数之和**），然后在右边编写代码，点击运行，就可以与逆行相关的代码了。具体流程如下图所示：

![[6ca138710f511447206117d288f2d42d_MD5.png]]

根据题目提示，编写相应的代码：

![[58c4a59bfbb4a2d86111a00bfe64a374_MD5.png]]

点击运行按钮运行程序：

![[21c10d268237ddc86b8768c42fe391f4_MD5.png]]



最后，下面的执行窗口就会显示代码的执行结果，来判断代码是否正确：

![[c784f1fb15398a200f6d6fcd68c738b1_MD5.png]]

**到这里，LeetCode 插件的教程已经结束了，希望大家坚持刷算法题！！！**