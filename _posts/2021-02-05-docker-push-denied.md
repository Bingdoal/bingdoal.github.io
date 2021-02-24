---
layout: post
title: "[踩雷紀錄] Docker push denied"
date: 2021-02-05 17:19:54 +0800
category: deploy
img: cover/docker.jpg
description: 在做 k8s 的練習的時候碰到的問題，不管怎麼推都推不上去 docker repo，記錄一下踩雷經驗
lang: zh-TW
tags: [踩雷紀錄 ,docker, deploy]
---

{{page.description}}

## 1. 首先看到下圖

`docker push` 執行後跑出 `denied: requested access to the resource is denied` 的錯誤

![]({{site.baseurl}}/assets/img/docker-push-denied.png)

## 2. 登入

第一反應就是沒登入 docker，於是下登入指令

![]({{site.baseurl}}/assets/img/docker-login.png)

就不附圖了，登入後發現情況還是跟上面一樣

## 3. image 改名

上網找了一下資料才知道，原來 image 的 tag name 必須有帳號的前綴才能推上去，於是

```
docker tag express-ts bingdoal/express-ts
```

![]({{site.baseurl}}/assets/img/docker-push-success.png)

成功啦，沒想到連 tag name 都要講究