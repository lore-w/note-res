---
title: Git Submodule 进阶
tags:
  - Git
  - submodule
date: 2017-06-17 23:52:20
---

最近因为我的旅行记录站集成了第三方的服务，涉及tocken授权的问题，需要将主题仓库从公有库迁移到私有库，所以需要将Hexo项目中的主题（作为submodule存在）的地址作同步的更新，对于中间遇到的一些问题作了详细的分析，记录如下。
<!-- more -->

首先在本地test文件下下初始化一个git仓库，我们知道.git文件下存放了项目的提交信息，cd到这个目录下发现有如下8个文件及文件夹
![](/images/2017/submodule-1.png)
其中config文件存放了一些项目中的本地git配置，如下
![](/images/2017/submodule-2.png)
现在我们添加一个submodule，再次进入.git文件下，发现多了两个文件
![](/images/2017/submodule-3.png)
其中index文件存放了主项目现在使用了子项目的哪一次提交，（submodule的原理就是主项目从远程下载子项目，然后checkout到主项目记录的commitId）modules文件下存放了子项目的信息

现在打开。git下的config文件，发现多了submodule的配置
![](/images/2017/submodule-4.png)

index问价下记录了主项目现在引用的子项目commitId

把config里的submodule信息删掉，如下

现在重新初始化submodule的信息
![](/images/2017/submodule-5.png)

update回发现不能update
![](/images/2017/submodule-6.png)

这是因为。git／modules下的文件，现在把它删掉，重新update
![](/images/2017/submodule-7.png)

![](/images/2017/submodule-8.png)
![](/images/2017/submodule-9.png)
![](/images/2017/submodule-10.png)
![](/images/2017/submodule-11.png)