---
layout: post
title: "Kubernetes 初戰(四) StatefulSet 以及 Deployment 的差異"
date: 2021-06-16 09:27:48 +0800
category: deploy
img: cover/kubernetes.png
description: 前幾篇討論過的 Deployment 是屬於偏向 stateless 的 Pod 部屬，雖然可以透過設定 Volume 來保存某些狀態，但實際使用上還是有些不同，如果說需要建立 stateful 的服務的話，k8s 有設置 StatefulSet 的資源來作這方面的機制，本篇主要是撰寫 StatefulSet 的實作以及與 Deployment 的差異
lang: zh-TW
tags: [kubernetes, deploy]
---

{{page.description}}

## StatefulSet

根據官方給予的使用 StatefulSet 的情境建議
1. 穩定且唯一的網路辨識(Stable, unique network identifiers.)
2. 穩定且持久化儲存(Stable, persistent storage.)
3. 有序且優雅的部屬及擴充(Ordered, graceful deployment and scaling.)
4. 有序並自動滾動更新(Ordered, automated rolling updates.)

簡單說需要持久化並確保資料保存，以及需要固定網路位置的服務適合使用 StatefulSet，主要強調在於各種穩定性

並且從第一點及第二點可以知道 StatefulSet 必須搭配一個 Service 以及 PVC 來設定一個唯一的網路辨識以及持久化的儲存機制

### yaml
由於 StatefulSet 的特性，所以在 StatefulSet 的 yaml 中必須要有 Service 以及 Volume 的設定，這邊的 Volume 藉由外部的 PVC 來加入，PVC 的設定可以看[上篇](https://bingdoal.github.io/deploy/2021/05/pv-and-pvc-in-kubernetes/)的介紹

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: nginx-svc
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: http
        volumeMounts:
        - mountPath: /tmp/conf
          name: nginx-volume
      volumes:
      - name: nginx-volume
        persistentVolumeClaim:
          claimName: nginx-pvc
```

記得在設定 StatefulSet 之前要先將 PVC 以及 Service 設定好

## StatefulSet vs Deployment

### 部署 或是 Scale out
當部署 StatefulSet 的時候，replica 數量如果大於一，Pod 不會像 Deployment 的部署一次全部產生，而是會逐一有序的產生，對應到上述 StatefulSet 的特性中強調的有序性，會等到前面順序的 Pod 部署完成才部署下一個 Pod

### 刪除 或是 Scale in
而當要進行 Pod 的刪除或是進行 Scale in 時，則是反向依序刪除，從最後建立的 Pod 開始逐一移除，在完成後面順序的刪除後才會繼續刪除下一個 Pod

並且在刪除 StatefulSet 的時候預設 Volume 的資料並不會被刪除

### Volume
雖然 StatefulSet 跟 Deployment 都可以設定 Volume 但兩者還是有些不同，Deployment 中的所有 Pod 都會共享一個 Volume，同時會有多個 Pod 在存取，而在 StatefulSet 則是每個 Pod 都會有個獨立的 PVC 以確保彼此不互相影響

### Pod
從上面 Scale in 的策略可以看出來 StatefulSet 的 Pod 在刪除或更新時會等到前一個 Pod 完全停止才啟動新的 Pod，而不會像 Deployment 會在同止 Pod 的同時啟動新的 Pod

## 結語
簡單說的話，只要是需要儲存狀態的服務，都比較建議使用 StatefulSet，像是資料庫類的服務，比較可以確保資料的穩定性