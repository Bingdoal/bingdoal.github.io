---
layout: post
title: 利用 Docker volume 儲存 container 的資料在本地端
date: 2021-01-13 16:14:12 +0800
category: deploy
img: cover/docker.jpg
description: 常在使用 container 建置開發環境後常會有一個困擾，就是當 container 被刪除的時候裡面的資料也會一併消失，或是 container 之間要共用一些檔案的時候都會有一些困難點，於是就有 docker volume 的機制出來了
lang: zh-TW
tags: [docker, deploy]
---

{{page.description}}

# Docker volume

由於 container 本身是一個封閉的環境，所以整個環境被刪除的時候裡頭的資料也無法倖免，為了讓我們可以保留部分資料，像是 log, report 或者 config 之類需要被保留的文件，於是有了 volume 這一個概念

其實說穿了就是在 container 裡面建立一個與本地端連結的路徑，像是 windows 的分享資料夾的概念，以下就直接以實作來看看吧

## 實作
+ 建立一個 container 並指定本地端的 `C:/.volume` 與 container 的 `/storage` 連結

```
docker run -it -v C:/.volume:/storage ubuntu:18.04 bin/bash
```

+ 也可以不指定連結位置，docker 會自動產生鏈結的資料夾

```
docker run -it -v /storage ubuntu:18.04 bin/bash
```

+ 透過指令查看 volume 位置
{% raw %}
```
docker inspect -f "{{range .Mounts}}{{.Source}}{{end}}" <container_id>
```
{% endraw %}
+ 如果在 Windows 上建立的話會發現實際上路徑並不存在，因為 Window 版的 docker 實際上是透過 WSL 去執行的，以 Windows 來說的話會放在 `\\wsl$\docker-desktop-data\version-pack-data\community\docker\volumes` 裡面，是個十分神奇的位置

+ 也可以加在 Dockerfile 裡頭，其實跟指令沒什麼差異

```
VOLUME "C:/.volume" "/storage"
```
# 其他傳遞檔案的方法

有時候只是臨時需要來回傳個檔案，在建立 container 的時候並沒有事先掛上 volume，那也可以透過下面的方式來做到檔案傳遞:

```
docker cp <container>:<source-path> <dest-path>
docker cp <source-path> <container>:<dest-path>
```

簡單指令可以在 container 跟本地之間複製檔案

## 結語

volume 使用起來非常方便，可以直接建立 container 與本地端的資料共用甚至 container 之間也可以共用 volume，往後產出的 log 或是 report 也不用透過 api 打出來到實體機器，可以直接放在 volume 底下