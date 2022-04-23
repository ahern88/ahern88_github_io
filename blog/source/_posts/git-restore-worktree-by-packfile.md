---
title: 使用pack文件恢复Git工作区
comment: true
date: 2022-04-23 21:55:17
categories:
  - Git
tags:
  - Git
  - 工具
---

## 0. 本文介绍

了解过Git原理的同学应该都知道，Git底层是一个kv的文件数据库，Git管理的所有的文件都存在objects文件夹中，而当我们从远端仓库clone数据到本地的过程中主要传递两种数据：

- branch和tag对应的commit id数据列表

- 服务器仓库生成的pack文件（包含所有Git历史提交的数据版本）

那么本文就介绍如何通过pack文件和一个commit id恢复本地工作区，从而去加深了解Git的底层原理。

------

## 1. 新建仓库并加入已有的pack文件

假设我们拿到服务器仓库中的pack文件 pack-598b51485951bbfd3cd893ef230beb0e96ea172c.pack 在/tmp目录下

```bash
cd study_git
git init demo01 # 创建demo01的git目录
cd demo01
cp /tmp/pack-598b51485951bbfd3cd893ef230beb0e96ea172c.pack .

```

------

## 2. 基于pack文件生成pack的idx索引文件

pack中包含了所有的object数据，但它时一个压缩文件，需要有对应idx索引文件才能完成object数据的查找，现在我们有了pack文件，那么我们可以基于```index-pack```命令去生成对应的idx文件，idx文件包含是object对应的hash值和对应的存储位置，可以快速定位。

```bash
cd demo01
git index-pack -o pack-598b51485951bbfd3cd893ef230beb0e96ea172c.idx pack-598b51485951bbfd3cd893ef230beb0e96ea172c.pack
598b51485951bbfd3cd893ef230beb0e96ea172c
ls -l
pack-598b51485951bbfd3cd893ef230beb0e96ea172c.idx
pack-598b51485951bbfd3cd893ef230beb0e96ea172c.pack
```

<!--more-->

这样就在demo01目录下生成了对应的idx文件，现在我们将pack和idx文件转移到objects/pack目录下

```bash
cd demo01
mv pack-* .git/objects/pack/
git cat-file -p 83248373454c38948cc5077e8e613ea276d20018
tree c6861a154c3a91f639db2fbcead9d19c6aa84f0a
parent 43ee8495120f80671b1edc943d6e369b270e303c
author ahern88 <ahern88@163.com> 1650702777 +0800
committer ahern88 <ahern88@163.com> 1650702777 +0800

update 1.txt
```

现在我们完成了pack和idx文件的准备，数据准备好了，但是我们发现工作区中并没有文件，下一步我们要恢复工作区

------

## 3. 恢复工作区

现在object相关的数据在底层都已经存在了，但是现在我们差一个入口，一个commit的入口，只要有了commit id的值，我们就能通过这个commit id找到对应的tree，基于tree就能找到所有相关的blob和tree，就能恢复工作区。

假设我们知道历史的一次提交，产生的commit id为 83248373454c38948cc5077e8e613ea276d20018

```bash
cd demo01
ls  # 发现目前工作区中没有任何文件
git checkout 83248373454c38948cc5077e8e613ea276d20018
注意：正在切换到 '83248373454c38948cc5077e8e613ea276d20018'。

您正处于分离头指针状态。您可以查看、做试验性的修改及提交，并且您可以在切换
回一个分支时，丢弃在此状态下所做的提交而不对分支造成影响。

如果您想要通过创建分支来保留在此状态下所做的提交，您可以通过在 switch 命令
中添加参数 -c 来实现（现在或稍后）。例如：

  git switch -c <新分支名>

或者撤销此操作：

  git switch -

通过将配置变量 advice.detachedHead 设置为 false 来关闭此建议

HEAD 目前位于 8324837 update 1.txt

git checkout -b feature/v1.0
git ls-files --stage
100644 145e9a78fb6b7dfcff4311b0ba7de663c98bb6db 0	1.txt
100644 8d0e41234f24b6da002d962a26c2495ea16a425f 0	2.txt
100644 8d0e41234f24b6da002d962a26c2495ea16a425f 0	3.txt
100644 e5c64e672148327cb9cc9a47600ca01c52babe1c 0	4.txt
100644 87db4918eac1d0263f89c4459bcbcc036536c045 0	5.txt
ls
1.txt 2.txt 3.txt 4.txt 5.txt
```

就这样通过一个commit id和一个pack文件，重建恢复了整个工作区的文件目录，你就又可以开始你的代码之旅了。

------

## 4. 参考资料

- [官方 Pro Git 书籍](https://git-scm.com/book/zh/v2)

- [一文深入浅出学习Git工具](https://ahern88.github.io/2022/04/05/git-in-depth-study/)

