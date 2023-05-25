---
layout: post
title: "Linux 指令查看使用中的連線 port: netstat, lsof"
date: 2023-05-25 14:08:02 +0800
category: linux
img: cover/linux.png
description: "前面介紹怎麼測試網路，這篇來寫怎麼檢測自己環境的連線 port，還蠻常會遇到啟動程式時發現 port 被佔用，要找出是誰占用的情形發生，這邊介紹兩個指令 netstat, lsof"
lang: zh-TW
tags: [network, linux]
published: true
---

{{page.description}}

## netstat

這應該是最常見的指令，也是蠻多 Linux 有內建的指令，直接鍵入 `netstat` 可以秀出所有的連線，不過通常太多了，難以閱讀，可以透過下面參數來篩出要的資訊，常用參數的有下面

| 參數  | 說明                                                          |
|:-----:|:--------------------------------------------------------------|
| `-t`  | TCP                                                           |
| `-u`  | UDP                                                           |
| `-l`  | 監聽狀態                                                      |
| `-e`  | 建立連線狀態                                                  |
| `-p`  | 顯示使用的 process，看不到其他使用者的行程，常搭配 `sudo` 使用˝ |
| `-c`  | 持續更新                                                      |
| `-ie` | 顯示網路介面卡，等同於 `ifconfig`                              |

常搭配 `grep` 使用，找到需要的 port 或是 process

## lsof

`lsof` 指令主要是拿來查看開啟檔案的行程的，但也可以用來查看網路資訊

```shell
lsof -i tcp
lsof -i tcp:80
lsof -i tcp:1001-2000
lsof -i tcp -s tcp:listen
```

應該不用多解釋指令意義，應該還蠻好懂的，想查看 udp 就把 tcp 換成 udp 就好了
