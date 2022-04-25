---
layout: post
title: "Linux 環境下在背景執行程式"
date: 2021-03-03 17:28:41 +0800
category: linux
img: cover/linux.png
description: 在 Linux bash 操作下應該很常會有這種需求，尤其是在 ssh 連線下需要執行程式的同時去做其他事，如果為此多開 terminal 去連線也蠻蠢的，下面記錄一下 linux 背景執行的一些指令
lang: zh-TW
tags: [linux]
---

{{page.description}}


# 背景執行
```bash
<cmd> &
# Output
# [1] 135
```
把 cmd 放到背景執行，並輸出 PID

然後藉由下面指令，查看目前在背景執行的 process
```bash
jobs -l
```

如果想要把背景程式搬回到前景可以透過
```bash
fg # 將第一個背景程式搬到前景
fg %2 # 將第二個背景程式搬到前景
```
如果想要把前景在轉回背景請先鍵入 `ctrl+z` 暫停 process 然後輸入

```bash
bg # 將第一個暫停的程式回到背景執行
bg %2 # 將第二個暫停的程式回到背景執行
```

如果要結束背景程式請直接 kill

```bash
kill -9 <PID>
```

## output

如果希望背景程式的 stdout 可以被查看，可以將 stdout 寫入到檔案中
```bash
<cmd> &> output.log &
```

這樣該程式的輸出就會被持續寫入到 output.log 中了

## 延伸閱讀

如果說需要在離開 ssh 連線之後還能持續運行的話可以[參考](https://bingdoal.github.io/linux/2022/03/linux-run-in-background-after-exit-ssh/)