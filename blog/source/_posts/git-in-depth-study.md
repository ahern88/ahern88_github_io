---
title: 一文深入浅出学习Git工具
comment: true
date: 2022-04-05 17:05:20
categories:
  - Git
tags:
  - Git
  - 工具
typora-root-url: ../../source
---

> Git也许在很多人看来只是一款版本控制软件，其实不然，Linus在设计Git时给它的定位并不是版本控制，而是一个包含版本控制能力的文件系统，为什么说它是一款文件系统？在了解Git的实现原理的时候你就会明白为啥它的定位是文件系统。

## 0. Git安装

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

<!--more-->

------

## 1. Git命令

### 1.1 init

init命令的原理很简单，其原理就是在当前目录下创建 .git 文件夹，并生成如下文件结构，这个过程相当于初始化git内置的文件数据库

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

### 1.2 clone

先看clone的整个过程
![Clone流程图](./images/git-in-depth-study/Git原理-Clone原理.drawio.png)

git clone其实本质包含三步操作

1. 先执行init初始化本地数据库

2. 通过指定的url去远程仓库拉取数据，分两个数据请求

   - 请求refs数据（先发送/info/refs?service=git-upload-pack）

     ```bash
     GET /xxxx/xxx.git/info/refs?service=git-upload-pack HTTP/1.1
     Host: git.xxxx.com
     Authorization: Basic xxxxxxxxxx==
     User-Agent: git/2.35.GIT
     Accept: */*
     Accept-Encoding: deflate, gzip
     Accept-Language: zh-CN, *;q=0.9
     Pragma: no-cache
     Git-Protocol: version=2
     Connection: keep-alive
     
     HTTP/1.1 200 OK
     Server: nginx
     Date: Tue, 12 Apr 2022 17:11:30 GMT
     Content-Type: application/x-git-upload-pack-advertisement
     Content-Length: 140
     Cache-Control: no-cache
     Content-Encoding: gzip
     Strict-Transport-Security: max-age=31536000
     Referrer-Policy: strict-origin-when-cross-origin
     Proxy-Connection: keep-alive
     
     001e# service=git-upload-pack
     0000000eversion 2
     0015agent=git/2.29.0
     000cls-refs
     0019fetch=shallow filter
     0012server-option
     0017object-format=sha1
     0000
     ```

   - 请求pack数据（再发送2次/info/git-upload-pack获取服务端的数据，第1次获取refs信息，第2次获取object压缩文件）

     第一次请求获取refs信息，ls-refs命令

     ```bash
     POST /xxxx/xxxx.git/git-upload-pack HTTP/1.1
     Host: git.xxxx.com
     Authorization: Basic xxxxxxxx==
     User-Agent: git/2.35.GIT
     Accept-Encoding: deflate, gzip
     Content-Type: application/x-git-upload-pack-request
     Accept: application/x-git-upload-pack-result
     Git-Protocol: version=2
     Content-Length: 166
     Connection: keep-alive
     
     0014command=ls-refs
     0016agent=git/2.35.GIT0016object-format=sha100010009peel
     000csymrefs
     0014ref-prefix HEAD
     001bref-prefix refs/heads/
     001aref-prefix refs/tags/
     0000
     
     HTTP/1.1 200 OK
     Server: nginx
     Date: Tue, 12 Apr 2022 17:11:30 GMT
     Content-Type: application/x-git-upload-pack-result
     Transfer-Encoding: chunked
     Cache-Control: no-cache
     Strict-Transport-Security: max-age=31536000
     Referrer-Policy: strict-origin-when-cross-origin
     Proxy-Connection: keep-alive
     
     0053527a1cfca15f69a4fc70a5a8ab0a4780a08d54cd HEAD symref-target:refs/heads/develop
     003ec4a3084ab692ac60c21c9b1fd91d66f6b33ca1ff refs/heads/2.0.0
     003e2e4de8da6ddbe3d3449f09759074c45632578308 refs/heads/2.0.1
     ......
     003e2e4de8da6ddbe3d3449f09759074c45632578308 refs/heads/5.0.1
     0000
     # 0053和003e是数据类型，如branch、tag等
     ```

     第二次获取object的压缩信息

     ```bash
     POST /xxxxx/xxx.git/git-upload-pack HTTP/1.1
     Host: git.xxxxx.com
     Authorization: Basic xxxxxxxxxx==
     User-Agent: git/2.35.GIT
     Accept-Encoding: deflate, gzip
     Content-Type: application/x-git-upload-pack-request
     Accept: application/x-git-upload-pack-result
     Git-Protocol: version=2
     Content-Encoding: gzip
     Content-Length: 1893
     Connection: keep-alive
     
     0011command=fetch0016agent=git/2.35.GIT0016object-format=sha10001000dthin-pack000dofs-delta0032want 527a1cfca15f69a4fc70a5a8ab0a4780a08d54cd
     0032want c4a3084ab692ac60c21c9b1fd91d66f6b33ca1ff
     0032want 2e4de8da6ddbe3d3449f09759074c45632578308
     0032want 3791b07056cd2bd04be6acaea51cd886587aa826
     0032want dadaba7ca933af40ce703604986e53b5c15a2145
     ......
     0032want 5f7fd8f722527c8cfcdeda7a4ff51d052c568725
     0032want 2317be07ef3a7a083c8ad8e42605ebdd0f5e24dc
     0009done
     0000
     
     HTTP/1.1 200 OK
     Server: nginx
     Date: Tue, 12 Apr 2022 17:11:30 GMT
     Content-Type: application/x-git-upload-pack-result
     Transfer-Encoding: chunked
     Cache-Control: no-cache
     Strict-Transport-Security: max-age=31536000
     Referrer-Policy: strict-origin-when-cross-origin
     Proxy-Connection: keep-alive
     
     000dpackfile
     0026Enumerating objects: 7015, done.
     0010PACK     2005gxËÁ	1@Ñ{ªÈ]dL±I²»ueÉ^¬Ã:¼{²±+ðôáÁïk­:#2Y@,`#Å5GÀQrbu¡µ»Î5CBG¾·I meï1K f¥ )Íç¢­ËªiëyÐ_O×¾ìeÚ¢ó)åè@ïL2F}uz¯,êõ¼¿oÞõp?
     xA
     ......
     ¨½xuÊ; EÑUXËXÌdx ágfÀÂÕk,M,ÏÍ5k¦iáÞBæ} ³B¬BNºoÒçawt$Vðn¹L­Q»Fþ
     ÝCÝ
     RÛ.ûÂ­®iXÖ¥nÑý¼!gj0006­0000
     ```

3. 基于objects文件夹的数据在本地工作区中恢复当前版本的文件

使用index-pack能基于已有的pack文件生成idx文件

```bash
$ git index-pack -o pack-168897a15d76f859db48b05e57fa62ed51da13c7.idx /xxx_repo/.git/objects/pack/pack-168897a15d76f859db48b05e57fa62ed51da13c7.pack
```

最后通过生成的pack文件，通过commit对象获取到tree对象，从tree对象恢复index文件，并checkout出最新的工作区文件

### 1.3 add

git add命令执行后会做如下两件事

- 将当前工作区中的文件计算hash（采用sha1）后写入object数据库中（这里只包含内容，不包含文件名）

  ```bash
  ➜  demo05 git:(master) ✗ tree .git
  ├── objects
  │   ├── 1d
  │   │   └── f95118185e2c64bad7b70cb734d64b8f156de8
  │   ├── 42
  │   │   └── 8e2d41d463864e5f6f691bb8b25c42e6e70817
  │   ├── 5d
  │   │   └── b502c2cb71c9ecae6fdba29e019f3990fba40b
  │   ├── f5
  │   │   └── 69a1819e3e3f3c1099c3e8a203ca60fcc44c08
  ➜  demo05 git:(master) git cat-file -t f569a1819e3e3f3c1099c3e8a203ca60fcc44c08
  blob
  ➜  demo05 git:(master) git cat-file -p f569a1819e3e3f3c1099c3e8a203ca60fcc44c08
  hello abcdefg12312312
  ```

- 将当前文件列表写入到index索引文件中（包含文件类型、权限、hash值、文件名等）

  ```bash
  ➜  demo05 git:(master) ✗ git ls-files --stage
  100644 84b20b931e64957dc817031e1221f8e79af9ab19 0	com/test3.txt
  100644 1df95118185e2c64bad7b70cb734d64b8f156de8 0	test.txt
  100644 f569a1819e3e3f3c1099c3e8a203ca60fcc44c08 0	test2.txt
  ```

> 注意：add 操作并不会为文件夹创建tree对象，而是在下面commit操作完成tree对象的生成。

### 1.4 commit

```bash
➜  demo05 git:(master) ✗ git commit -m 'init demo'
[master（根提交） 014dc87] init demo
 3 files changed, 3 insertions(+)
 create mode 100644 com/test3.txt
 create mode 100644 test.txt
 create mode 100644 test2.txt
```

commit操作会为当前提交生成commit对象，并在logs中记录log信息，解析index暂存区的文件结构，为文件夹生成tree对象，将此过程产生的对象存入object数据库中。

其中logs文件夹的信息主要为 git reflog 服务，可以看到git的操作日志（如删除分支、切换分支等操作）

每个commit对象会指向一个tree对象和一到多个parent对象，指向的tree对象包含了当前commit对应的所有文件对象。

### 1.5 fetch

### 1.6 pull

pull命令简单理解就是 fetch + merge，先通过fetch将远程仓库中最新的代码同步到 本地远程分支中，然后将本地远程分支的代码合并到对象的本地分支中

### 1.7 push

push命令和pull命令相反，它是向git服务器提交新的数据。其中也是包含两步，先发送 info/refs?service=git-receive-pack 命令获取服务器上最新的分支情况，然后在本地生成pack数据后通过 git-receive-pack 接口发送过去。

- 通过 info/refs?service=git-receive-pack 获取服务器最新分支情况

  ```bash
  GET /xxxxx.git/info/refs?service=git-receive-pack HTTP/1.1
  Host: git.xxxxx.com
  Authorization: Basic xxxxxx==
  User-Agent: git/2.35.GIT
  Accept: */*
  Accept-Encoding: deflate, gzip
  Accept-Language: zh-CN, *;q=0.9
  Pragma: no-cache
  Connection: keep-alive
  
  001f# service=git-receive-pack
  000000beb817c1ddbfb186fac42fad989df3595a1431396c refs/heads/PRODreport-status report-status-v2 delete-refs side-band-64k quiet atomic ofs-delta push-options object-format=sha1 agent=git/2.29.0
  003db817c1ddbfb186fac42fad989df3595a1431396c refs/heads/TEST
  00405ff1597364ec484ce9434550342358fa299bd421 refs/heads/develop
  004eb817c1ddbfb186fac42fad989df3595a1431396c refs/tags/å¤©æº_å¤©æºV20210316
  0000
  ```

- 调用 git-receive-pack 发送本地增量数据

  ```bash
  POST /xxxx.git/git-receive-pack HTTP/1.1
  Host: git.xxxxx.com
  Authorization: Basic xxxxxx==
  User-Agent: git/2.35.GIT
  Accept-Encoding: deflate, gzip
  Content-Type: application/x-git-receive-pack-request
  Accept: application/x-git-receive-pack-result
  Content-Length: 480
  Connection: keep-alive
  
  00c4d1cb34c00645790a3bd92fe4e35428a1e455000f 64fd98d5f8e293feae96f9deb4b3b969df2681b9 refs/heads/develop  report-status-v2 side-band-64k object-format=sha1 agent=git/2.35.GIT0000PACK      
  xËM
  !@á½§p_¢F¡^Ecd:?Lû·^ »Çoª¶WAç°SSÎ,5Vñ*ØÙsì+rËæ(§nÃ6'I "ÎP°¶ì»b S
   º)×÷ÓYÏ-%{ÿÅÓEd_ÖÅ ÞR°7H æ«ë2þ5ëhe¨­×òjÓ{6Ì(<àþðJûÔg{V|%1¾L®Exkfjf`&¢ò2QÑ}ñqÛÊ¯~±×ò~8Ñk' ¨¥7{ùÎ¢#ùîÞéæß{x»ÎxqÂ´Ä. _RîðyWñýÀ_¾øÄÃ/·
  
  0047000eunpack ok
  0030ok refs/heads/develop
  00000006
  0085To create a merge request for develop, visit:
    http://git.xxxxx.com/xxxx/-/merge_re0050quests/new?merge_request%5Bsource_branch%5D=develop
  
  0000
  ```

  

### 1.8 remote

### 1.9 branch

### 1.10 tag

### 1.11 merge

### 1.12 rebase

------

## 2. Git底层命令

### 2.1 ls-files

```bash
➜  demo04 git:(master) ✗ git ls-files -s
100644 defcc8403294ad3772ef80c7f9f0863f2e56be47 1	Test.java
100644 5c5afc55543e5d2b68c38fdd966a3cfc9234d715 2	Test.java
100644 a0f09dfc6ae374ced7de33884abd2e4817d7eff9 3	Test.java
100644 faa29b2a19af8a3ab5682c228aa5b2b7b7db8561 0	text001.txt
100644 92df0d24f248938ee3ab8867af2d486c58ade458 0	text002.txt
```

### 2.2 cat-file

### 2.3 hash-object

### 2.4 write-tree

------

## 3. Git内部原理

### 3.1 Git的四大对象

git底层包含四大对象，blob、tree、commit、tag，每个对象都存储在 .git/objects/ 下，通过类型+内容计算出sha1的40位长度的摘要（其中前2位为文件夹名称，后38位为文件名）

#### 3.1.1 blob对象

```bash
➜  demo05 git:(master) git cat-file -t f569a1819e3e3f3c1099c3e8a203ca60fcc44c08
blob
➜  demo05 git:(master) git cat-file -p f569a1819e3e3f3c1099c3e8a203ca60fcc44c08
hello abcdefg12312312
```

工作区中的每个文件都会产生一个或多个版本的blob对象，当使用add命令或hash-object命令的时候会将内容写入到object数据库中，类型为blob。

#### 3.1.2 tree对象

```bash
➜  demo05 git:(master) git cat-file -t f6aef1915baafe2567427da5d687a4b8ebfac983
tree
➜  demo05 git:(master) git cat-file -p f6aef1915baafe2567427da5d687a4b8ebfac983
040000 tree e9543a8cde4b92f8a6ad4a07ab5979c04d23fab8	com
100644 blob 1df95118185e2c64bad7b70cb734d64b8f156de8	test.txt
100644 blob f569a1819e3e3f3c1099c3e8a203ca60fcc44c08	test2.txt
```

tree对象可以理解为是文件夹的对象，整个工作区文件夹相当于根tree，其中的子文件为blob对象，而子文件夹为tree对象，依次能枚举出所有文件和文件夹的结构。

#### 3.1.3 commit对象

```bash
➜  demo05 git:(master) git cat-file -t 014dc87c3af9af64b96688d0e408cd2d718a8805
commit
➜  demo05 git:(master) git cat-file -p 014dc87c3af9af64b96688d0e408cd2d718a8805
tree f6aef1915baafe2567427da5d687a4b8ebfac983
author ahern88 <ahern88@163.com> 1649775779 +0800
committer ahern88 <ahern88@163.com> 1649775779 +0800

init demo
```

commit对象包含一个或多个parent的commit指向 和 tree对象（这个是整个工作区文件夹的快照，联合index文件能恢复整个工作区）。

#### 3.1.4 tag对象

```bash
➜  demo05 git:(master) git cat-file -t 20033db439a593a1866ab312ed9bb100ddfefa96
tag
➜  demo05 git:(master) git cat-file -p 20033db439a593a1866ab312ed9bb100ddfefa96
object 014dc87c3af9af64b96688d0e408cd2d718a8805
type commit
tag v1.0.tag
tagger ahern88 <ahern88@163.com> 1649778880 +0800

test my tag
➜  demo05 git:(master) git cat-file -t 014dc87c3af9af64b96688d0e408cd2d718a8805
commit
```

其中标签分轻量级标签和附注标签

- 轻量级标签：不产生tag对象，tag文件直接指向一个commit对象
- 附注标签：产生tag对象，tag文件指向tag对象，其中tag对象包含详细信息（标签信息和作者等信息）和commit对象的指向

------

## 4. Git扩展功能

### 4.1 submodule

### 4.2 lfs

### 4.3 worktree

------

## 5. Git源码环境搭建

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

------

## 6. 技巧

------

## 7. 参考资料

- [Git Reference](https://git-scm.com/docs) 
- [Pro Git Book](https://git-scm.com/book/en/v2)
