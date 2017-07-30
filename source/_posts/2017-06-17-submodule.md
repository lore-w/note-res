---
title: Git Submodule花样作死
tags:
  - Git
  - submodule
date: 2017-06-17 23:52:20
---

因为我的旅行记录站集成了第三方的服务，涉及tocken授权的问题，需要将主题项目从***公有库***迁移到***私有库***，因此需要将Hexo项目中的主题（作为submodule存在）的地址作同步的更新，对于中间遇到的一些问题作了详细的YY，记录如下:

为了方便描述，假设在本地有一个test项目，首先初始化一个git仓库，我们知道.git文件下存放了项目的git信息，`cd`到这个目录下发现有如下8个文件及文件夹

```bash
cd .git
ls
```
| HEAD | config | hooks | objects |
|---------------------------------|
| branches | description | info | refs |

其中config文件存放了一些项目中的本地git配置，如下

```bash
[core]
  repositoryformatversion = 0
  filemode = true
  bare = false
  logallrefupdates = true
  ignorecase = true
  precomposeunicode = true
```


现在我们添加一个submodule，再次进入.git文件下，发现多了两个文件

```bash
cd ../
git submodule add https://github.com/lore-w/2rem.git
Clone into '/Users/lore/Work/Code/test/2rem'...
remote: Counting objects: 19, done.
remote: Total 19 (delta 0), reused 0 (delta 0), pack-reused 19
Unpacking objects: 100% (19/19), done

cd .git
ls
```
| HEAD | config | hooks | info | objects |
|----------------------------------------|
| branches | description | index | modules | refs |

+ 第一个是.git/index文件，记录了主项目现在使用了子项目的哪一次提交。（submodule的其实就是主项目从远程下载子项目，再checkout到`.git/index`记录的commitId的过程）

+ 第二个是.git/modules文件夹，下面存放了子模块的相关信息

现在打开.git/config文件，发现多了submodule的配置

```bash
vim config
```

```bash
[core]
  repositoryformatversion = 0
  filemode = true
  bare = false
  logallrefupdates = true
  ignorecase = true
  precomposeunicode = true
[submodule "2rem"]
  url = https://github.com/lore-w/2rem.git
```

现在如果我需要把子模块的地址从`https://github.com/lore-w/2rem.git`变更成`https://github.com/lore-w/math-calc.git`，那么修改submodule URL最简单的办法就是试着把`/.gitsubmodule`里的submodule地址修改成`https://github.com/lore-w/math-calc.git`，再把`.git/config`的submodule信息删掉，重新init，如下：

```bash
git submodule init
Submodule '2rem' (https://github.com/lore-w/math-calc.git) registered for path '2rem'

```

init成功，此时已经在`.git/config`里重新写入了新的submodule字段，我们检查一下

```bash
cd .git
vim config
```

```bash
[core]
  repositoryformatversion = 0
  filemode = true
  bare = false
  logallrefupdates = true
  ignorecase = true
  precomposeunicode = true
[submodule "2rem"]
  url = https://github.com/lore-w/math-calc.git
```

和我们预想的一样，继续下一步，update

```bash
cd ../
git submodule update
git submodule update
git submodule update
```

执行了3次update都无任何结果，这是因为`.git/modules`文件夹下已经保存了当前项目的子模块信息，并且子模块没有发生更新，因此不会update（这也是为什么现在我把/2rem删除，可以在不重新clone的情况得到一份完整的2rem项目）,既然这样，现在我把`.git/modules`删掉，再update试一次

```bash
cd .git
rm -rf modules/
ls
```

| HEAD | config | hooks | info | objects |
|----------------------------------------|
| branches | description | index |  | refs |

`.git/modules`已删除

```bash
cd ../
git submodule update
fatal: Not a git repository: ../.git/modules/2rem
Unabled to find current revision in submodule path '2rem'
```

这次update失败了，我们先到`/2rem`目录下检查下

```bash
cd 2rem
vim .git //注意这里的.git不是一个文件夹，而是一个文件

```

```bash
gitdir: ../.git/modules/2rem
```

现在知道`/2rem`在其根目录下保存着一份与`.git/modules/2rem`关系的引用，`.git/modules/2rem`不在了，没有办法比对文件的变化，因此更新失败

既然这样，现在我把`/2rem`子模块也删除

```bash
cd ../
rm -rf 2rem
git submodule update
Clone into '/Users/lore/Work/Code/test/2rem'...
error: no such remote ref 1835xxx...9a48
Fetched in submodule path '2rem',but it did not contain 1835xxx...9a48.
Direct fetching of that commit failed.
```
update终于可以从GitHub上重新clone了，但是，紧接着下面报了一个错误，没有找到`1835xxx...9a48`这个commitId的引用

现在我们重新clone一份2rem的代码

```bash
cd ../
git clone https://github.com/lore-w/2rem.git
cd 2rem
git log
```
| commit | author | date | comment |
|----------------------------------|
| 1835xxx...9a48 | xxx | Wed Mar 22 20:42:56 2017 +0800 | fix bug3 |
| xxx | xxx | Fri Mar 17 23:10:45 2017 +0800 | fix bug2 |
| xxx | xxx | Fri Mar 17 22:42:18 2017 +0800 | fix bug1 |

发现最后一次提交正是`1835xxx...9a48`这个commitId，以此推测上面的报错是因为我们更换了submodule的地址，当git把新的submodule更新下来以后，试图把commitId切换到本地记录的submodule的的commitId（即2rem的提交记录`1835xxx...9a48`）这个commitId当然是不存在的

既然是因为没有找到commitId导致的，可不可以把本地记录的submodule commitId修改为`https://github.com/lore-w/math-calc.git`的提交记录，最终在`.git/index`文件下发现了提交ID的记录(共40位)

```bash
4449 5243 0000 0002 0000 0022 5942 c4d3
0000 0000 5942 c4d3 0000 0000 0100 0004
0040 81b3 0000 81a4 0000 01f5 xxxx 1835
xxxx xxxx xxxx xxxx xxxx xxxx xxxx xxxx
9a48 1fb7 779f 869a 000a 2e67 6974 6967
6e6f 7265 0000 0000 0000 0000 5942 c52c
0000 0000 5942 c52c 0000 0000 0100 0004
0040 83e8 0000 81a4 0000 01f5 0000 0014
```
把commitId修改掉，再次执行update操作

```bash
cd ../
git submodule update
error: bad index file sha1 signature
fatal: index file corrupt
```

这次报了一个签名没有通过的错误，因为`.git/index`里不仅仅保存了子模块的Id，还保存了与之对应的签名信息，修改了Id，导致签名被破坏，文件已经损坏。

现在只能把`.git/index`文件删除，重新添加一个submodule，如下

```bash
cd .git
rm -rf index
git submodule add https://github.com/lore-w/math-calc.git
Clone into '/Users/lore/Work/Code/test/math-calc'...
remote: Counting objects: 32, done.
remote: Total 32 (delta 0), reused 0 (delta 0), pack-reused 32
Unpacking objects: 100% (32/32), done
```

**总结，以上都是我推理YY出来的，不对正确性做保证，仅仅是作为一个探索性过程的记录**