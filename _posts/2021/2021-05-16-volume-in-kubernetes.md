---
layout: post
title: Kubernetes 初戰(二) Volume
date: 2021-05-16 09:00:26 +0800
category: deploy
img: cover/kubernetes.png
description: 在一般的觀念裡，container 相當合適作為 stateless application 之用(例如：api service)，但由於 stateful service 的需求也是存在的(例如：各種資料庫、檔案存放伺服器…等等)，因此 Kubernetes 就額外增加了一些相關的機制，讓 pod 也開始能夠承載 stateful application，本篇就簡單接紹 K8s 裡的 Volume
lang: zh-TW
tags: [kubernetes, deploy]
---

{{page.description}}
