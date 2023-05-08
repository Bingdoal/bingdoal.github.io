---
layout: post
title: "[踩雷紀錄] nginx: [error] invalid PID number"
date: 2023-05-05 10:41:59 +0800
category: deploy
img: cover/nginx.jpeg
description: "之前在使用 nginx 時有發生過這個錯誤，其實提示訊息蠻明顯的，就簡單記錄一下"
lang: zh-TW
tags: [踩雷紀錄,nginx,deploy]
published: true
---

{{page.description}}

## 問題敘述

問題發生如下

```shell
$ nginx -s reload
nginx: [error] invalid PID number "" in "/run/nginx.pid"
```

看到訊息後我們去找一下這個檔案

```shell
cat /run/nginx.pid

```

結果竟然是空的，看來問題就出在這裡了

## 解決辦法

這裡有兩個做法，第一個就是把他的 PID 塞回去，透過下面指令

```shell
ps -aux | grep nginx
```

找到 master process 的 PID 直接填入檔案

第二個方法就是透過下面指令

```shell
nginx -c <nginx.conf path>
```

重設一下設定檔，指令本身會把 PID 同步回去，就解決啦
