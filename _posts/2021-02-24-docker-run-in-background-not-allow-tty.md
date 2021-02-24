---
layout: post
title: [踩雷紀錄] Docker container 背景執行 tty 類的程式
date: 2021-02-24 14:05:38 +0800
category: deploy
img: cover/docker.jpg
description: 最近在部屬包裝在 Karaf 的產品的時候遇到的問題，不管怎麼包 container 都無法正常啟動，記錄一下踩雷的經驗
lang: zh-TW
tags: [踩雷紀錄,docker,deploy]
---

{{page.description}}

## Container 運行機制

Docker 的 container 啟動時會執行一個程式或是指令，而當指令執行完畢之後便會結束 container，而程式的結束判斷依據似乎是`是否進入 TTY 模式`，也就是自由輸入指令的 terminal mode

因此像是 Karaf 這種執行後會開啟一個終端的程式，在初始化完之後，進入終端操作介面，但我們需要的其實是在背景執行中的 feature

![]({{site.baseurl}}/assets/img/Karaf-console.png)

當時感到疑惑的地方就是當用，下面指令執行的時候一切正常
```shell
docker run -it <image>
```

但是換成下面背景執行就怎麼都跑不起來
```shell
docker run -d <image>
```
## 解法

解法其實也簡單暴力，把 karaf 丟到背景執行，然後無限 sleep
```dockerfile
CMD ./karaf &;sleep infinity
```

不過 karaf 其實有自帶背景執行的執行擋，可以直接
```dockerfile
CMD ./start;sleep infinity
```

或是寫成一個 shell script，直接執行 shell script 就好