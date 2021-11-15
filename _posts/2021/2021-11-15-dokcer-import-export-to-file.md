---
layout: post
title: "匯出/匯入 Docker image"
date: 2021-11-15 15:16:00 +0800
category: deploy
img: cover/docker.jpg
description: 以往 container image 的上傳與下載來源都是 hub，但其實 Docker 能夠把 image 存成本地檔案透過檔案共享的方式也可以把 image 傳送出去
lang: zh-TW
tags: [docker, deploy]
---

{{page.description}}

## Export

```sh
docker export <container-name> > container-export.tar
```

可以直接針對 container 進行匯出，注意 export 出來的是 tar 格式

## Import

```sh
cat container-export.tar | docker import - <container-name>
```

匯入的作法比較特別一點，需要透過 pipe 的方式去接收檔案資料


## Save
除了 Export 以外其實還有 Save 的指令，類似於 export，不過差別在於，`save` 是匯出該 container 的 image，而不是 container 本身，也就是在建立之後的操作並不會被匯出

```sh
docker save <container-name> > container-export.tar
```

## Load
如果檔案是由 save 輸出的，請用 load 來匯入

```sh
docker load < container-export.tar
```

---

## 結語
一開始在使用 docker 的時候其實就一直在想，難道一定要透過 hub 才能夠分享 image 嗎，這樣如果只是想分享給同事其實也不是太方便，事實上是可以透過檔案來傳送的