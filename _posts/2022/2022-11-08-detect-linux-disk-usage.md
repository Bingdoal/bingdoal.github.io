---
layout: post
title: "Linux 硬碟空間檢測"
date: 2022-11-08 10:00:18 +0800
category: linux
img: cover/linux.png
description: "長時間運作的伺服器主機常會有 log 寫爆或是備份檔塞爆硬碟的問題，甚至會導致機器直接重新啟動，紀錄一下基本監測硬碟使用率的指令"
lang: zh-TW
tags: [linux]
published: true
---

{{page.description}}

## df
最基本用來檢查硬碟的可用空間指令，直接鍵入 `df`，用來查看機器的總用量是否合理

```bash
$ df

Filesystem     1K-blocks     Used Available Use% Mounted on
devtmpfs         8003076        0   8003076   0% /dev
tmpfs            8032168        0   8032168   0% /dev/shm
tmpfs            8032168    84936   7947232   2% /run
tmpfs            8032168        0   8032168   0% /sys/fs/cgroup
/dev/nvme0n1p2  41930732 29982528  11948204  72% /
/dev/loop2         49152    49152         0 100% /var/lib/snapd/snap/snapd/17336
/dev/loop0         49152    49152         0 100% /var/lib/snapd/snap/snapd/17029
/dev/loop1         64768    64768         0 100% /var/lib/snapd/snap/core20/1634
/dev/loop3         45824    45824         0 100% /var/lib/snapd/snap/certbot/2414
/dev/loop4         45568    45568         0 100% /var/lib/snapd/snap/certbot/2344
/dev/loop6         64768    64768         0 100% /var/lib/snapd/snap/core20/1695
tmpfs            1606432        0   1606432   0% /run/user/1000
```

可以直接看到完整的容量使用率配比，基本上我們會關注的都是 Mounted on 在 `/` 的那行

還有一些可選參數可用
+ `df -h`: 空間單位的顯示會換成 `K,M,G` 的單位，更方便閱讀
+ `df -T`: 額外顯示檔案系統型態

## du

用來單獨顯示個別檔案或是目錄的硬碟使用量，用來定位塞爆你硬碟的元兇

直接鍵入 `du` 的話會列出執行目錄下所有目錄的使用量

一樣有一些可選參數可以用
+ `du -h <path>`: 空間單位的顯示會換成 `K,M,G` 的單位，更方便閱讀
+ `du -a <path>`: 除了目錄外一併列出檔案的佔用量
+ `du -s <path>`: 只顯示跟目錄的用量