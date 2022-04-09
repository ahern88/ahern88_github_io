---
title: 一文深入浅出学习Git工具
comment: true
date: 2022-04-05 17:05:20
categories:
  - Git
tags:
  - Git
  - 工具
---

> Git也许在很多人看来是一款版本控制软件，其实不然，Linus在设计Git时给它的定位并不是版本控制，而是一个包含版本控制能力的文件系统，为什么说它是一款文件系统？在了解Git的实现原理的时候你就会明白为啥它的定位是文件系统。

## Git安装

git分客户端和服务端，一般用户都为客户端安装，服务器主要用来管理远端仓库，一般用git web或gitlab安装。

mac安装git

```bash
$ brew install git
```

检查git是否安装完成

```bash
$ git --version
git version 2.32.0 (Apple Git-132)
```

## Git命令

### git init

init命令的原理很简单，其原理就是在当前目录下创建 .git 文件夹，并生成如下文件结构

```text
├── HEAD
├── config
├── description
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   ├── push-to-checkout.sample
│   └── update.sample
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```



### git clone

### git add

### git commit

### git fetch

### git pull

### git post

### git remote

### git branch

### git tag

### git merge

### git rebase



## Git底层命令

### git ls-files

```bash
➜  demo04 git:(master) ✗ git ls-files -s
100644 defcc8403294ad3772ef80c7f9f0863f2e56be47 1	Test.java
100644 5c5afc55543e5d2b68c38fdd966a3cfc9234d715 2	Test.java
100644 a0f09dfc6ae374ced7de33884abd2e4817d7eff9 3	Test.java
100644 faa29b2a19af8a3ab5682c228aa5b2b7b7db8561 0	text001.txt
100644 92df0d24f248938ee3ab8867af2d486c58ade458 0	text002.txt
```

### git cat-file

### git hash-object



## Git原理

### Git对象

#### Blob

#### Tree

#### Commit



## Git扩展功能

### Submodule

### LFS

### WorkTree



## Git源码环境搭建

安装openssl和asciidoc（非必须）

```bash
$ brew install openssl@3
$ export OPENSSLDIR=/usr/local/opt/openssl@3
$ cd git-master
$ make configure
$ ./configure --prefix=/opt/git
$ make all doc
$ sudo make install install-doc install-html
```



## 技巧

## 参考资料

- [Git Reference](https://git-scm.com/docs) 
- [Pro Git Book](https://git-scm.com/book/en/v2)
