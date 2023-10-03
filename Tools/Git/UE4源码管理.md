[【UE4】使用Git源码管理_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1nf4y1w77X/?spm_id_from=333.337.search-card.all.click&vd_source=9d1c0e05a6ea12167d6e82752c7bc22a)
[【UE4】虚幻的版本控制 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/136163257)
# 所需工具
**Git**：是一个开源的分布式版本控制系统。GitHub因为只支持Git 作为唯一的版本库格式进行托管所以名为GitHub。

**GitLFS**：Git大文件存储（LFS）在Git内部用文本指针替换了大文件，例如音频样本，视频，数据集和图形。(为项目中资源而下载的Git扩展，不然GitHub不允许大于100M的文件上传。) **目前windous端已经集成在git中**
注意：如果使用LFS上传，那么其他人pull仓库必须要也配置好LFS

**GitHub Desktop**：可以简化开发流程，让我们专注于有意义的事情，不用在工具的使用上浪费太多时间。(使用Git真的变得非常非常简单。)
# ue连接git
![[Pasted image 20230110141652.png]]
UE5在右下角就就可以进行源码管理：
![[Pasted image 20230114175649.png]]
![[Pasted image 20230110141837.png]]
**初始化项目->提示成功后接受设置**
项目文件夹下出现.git文件夹

**大文件超过100mb要启用GitLFS**
# 远程库的创建
这里远程库选择使用阿里云效
创建新的代码仓库，获取SSH： git@codeup.aliyun.com:63a5d66f8d9a873a30aae912/UE4project.git

打开.git文件夹，进入git bash
```c
git remote add origin git@codeup.aliyun.com:63a5d66f8d9a873a30aae912/UE4project.git
```
# git的使用
## 远程库
本地库 pull 远程库，获取远程库内容
本地库 push 远程库，提交本地库内容

```c++
git push -u origin "master"
// “”内容为分支名称
```

成功：
![[Pasted image 20230110145321.png]]
## 本地库：提交(迁入)、迁出
新创建的内容右上角有问号提示，点击保存后变成+
？：没有添加到源码管理
+：添加到源码管理

![[Pasted image 20230110143412.png]]————》![[Pasted image 20230110143451.png]]

√：迁出，我们需要对其提交
![[Pasted image 20230110144023.png]]
单个内容的提交：
![[Pasted image 20230110143559.png]]
![[Pasted image 20230110143628.png]]

将所有待提交文件提交：
![[Pasted image 20230110143735.png]]



## 查看历史/查看对比
![[Pasted image 20230110144527.png]]
![[Pasted image 20230110144424.png]]
![[Pasted image 20230110144449.png]]
## 恢复
回退版本
![[Pasted image 20230110144613.png]]

# .gitignore
ue默认给个忽略文件
```c++
Binaries
DerivedDataCache
Intermediate
Saved
.vscode
.vs
.idea
*.VC.db
*.opensdf
*.opendb
*.sdf
*.sln
*.suo
*.xcodeproj
*.xcworkspace
```

# github桌面端
add local repository

初始化Git LFS
![[Pasted image 20230114172310.png]]
然后push

![[Pasted image 20230114172605.png]]