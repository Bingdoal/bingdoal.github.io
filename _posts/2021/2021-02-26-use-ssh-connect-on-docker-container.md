---
layout: post
title: 想用 SSH 連線到 Docker container
date: 2021-02-26 14:54:23 +0800
category: deploy
img: cover/docker.jpg
description: 今天接到一個有趣的需求，是需要可以 ssh 連線到 container 上操作，一般都是直接用 docker exec 操作，不過可能會有需要給外部人員進來操作或看一些紀錄，ssh 遠端操作是最常見的作法，下面列出如何在 container 中設定 ssh 連線
lang: zh-TW
tags: [docker, deploy]
---

{{page.description}}

# 安裝 ssh
首先 container 版的 os 是沒有 ssh server 的，所以先安裝:

```bash
apt update
apt install openssh-server -y
```

# 設定密碼
然後 container 版 os 的 root 也是沒有密碼的，所以也得先設定一下:

```bash
passwd root
# Enter new UNIX password:
# Retype new UNIX password:
```

# 設定 ssh

因為預設 openssh server 有些功能沒有開，所以即使有帳號密碼也沒辦法連線，需要改一下設定檔:

```bash
echo "Port 22" >> /etc/ssh/sshd_config
echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
```

直接用 echo 的方式加入上面三行，然後再重啟 ssh，就可以連上啦

```bash
/etc/init.d/ssh restart
```

# Dockerfile

如果想在 container 一啟動就可以用 ssh 連線的話可以參考下面 Dockerfile 的寫法，不過應該只適用 `Ubuntu`:

```Dockerfile
FROM ubuntu:18.04

RUN apt update
RUN apt install openssh-server -y
RUN echo 'root:password' | chpasswd
RUN echo "Port 22" >> /etc/ssh/sshd_config
RUN echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
RUN echo "PermitRootLogin yes" >> /etc/ssh/sshd_config

EXPOSE 22

CMD /etc/init.d/ssh restart
```