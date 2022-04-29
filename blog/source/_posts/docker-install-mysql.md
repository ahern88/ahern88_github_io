---
title: 使用Docker安装MySQL
comment: true
date: 2022-04-26 20:46:26
categories:
  - 数据库
tags:
  - Docker
  - MySQL
typora-root-url: ../../source
---

## 安装Docker

一定要去官网下载Docker，目前一般会安装Docker Destop版本，也自带命令行工具。

下载地址：[docker-desktop](https://www.docker.com/products/docker-desktop/)

安装后检查下docker pull是否能从hub上拉取镜像：

```bash
➜ docker pull mysql:latest
Error response from daemon: Get "https://registry-1.docker.io/v2/": EOF
```

很遗憾，直接报错了，一般这个是网络问题，使用```dig @114.114.114.114 registry-1.docker.io``` 查看DNS解析情况

```bash
➜ dig @114.114.114.114 registry-1.docker.io

; <<>> DiG 9.10.6 <<>> @114.114.114.114 registry-1.docker.io
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17618
;; flags: qr rd ra; QUERY: 1, ANSWER: 8, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;registry-1.docker.io.		IN	A

;; ANSWER SECTION:
registry-1.docker.io.	53	IN	A	52.21.28.242
registry-1.docker.io.	53	IN	A	52.5.157.114
registry-1.docker.io.	53	IN	A	34.230.238.103
registry-1.docker.io.	53	IN	A	54.197.112.205
registry-1.docker.io.	53	IN	A	34.237.244.67
registry-1.docker.io.	53	IN	A	34.236.198.185
registry-1.docker.io.	53	IN	A	54.85.133.123
registry-1.docker.io.	53	IN	A	52.202.132.224

;; Query time: 10 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: Tue Apr 26 20:43:45 CST 2022
;; MSG SIZE  rcvd: 177
```

选取一个一个DNS记录，配置配置hosts，在```/etc/hosts```中增加记录

```bash
registry-1.docker.io 54.85.133.123
```

再测试```docker pull```发现已经可以工作。

## 拉取镜像启动容器

1）拉取镜像：查找mysql镜像，https://hub.docker.com/_/mysql?tab=tags

```bash
➜ docker pull mysql:latest                 
latest: Pulling from library/mysql
4be315f6562f: Pull complete 
96e2eb237a1b: Pull complete 
8aa3ac85066b: Pull complete 
ac7e524f6c89: Pull complete 
f6a88631064f: Pull complete 
15bb3ec3ff50: Pull complete 
ae65dc337dcb: Pull complete 
654aa78d12d6: Pull complete 
6dd1a07a253d: Pull complete 
a32905dc9e58: Pull complete 
152d41026e44: Pull complete 
42e0f73ebe32: Pull complete 
Digest: sha256:fc77d54cacef90ad3d75964837fad0f2a9a368b69e7d799665a3f4e90e600c2d
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest
```

2）启动容器

```bash
# 查找镜像
➜ docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
mysql        latest    f2ad9f23df82   6 days ago   521MB

➜ docker run --name mysql8 -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=ahern88 -v /data/mysql:/var/lib/mysql mysql:latest
2053595f4a2338a61ff727ceb9dad5c99d02229feab2823dfee852f8595b9aa2
```

3）检查数据库

使用客户端工具或代码连接，测试安装是否成功。
