---
layout: post
title: "Kubernetes 初戰(三) Persistent Volumes、Persistent Volume Claims"
date: 2021-05-17 09:07:51 +0800
category: deploy
img: cover/kubernetes.png
description: 繼上篇簡介過 K8s 的 Volume 之後，本篇想要來著墨在實際上運用 Volume 時是怎麼使用的，因為通常不會像上篇介紹的方式直接寫在 Pod 的設定內，而會透過 Persistent Volumes(PV) 及 Persistent Volume Claims(PVC) 來管理和設定
lang: zh-TW
tags: [kubernetes, deploy]
---

{{page.description}}

# 前言
Persistent Volumes(PV) 及 Persistent Volume Claims(PVC) 的概念是用來將 Pod 以及 Volume 分開，將 Volume 抽象化並從 Pod 的設定中抽離，可以讓專門的 storage 管理者來設定並管理 PV 及 PVC

## Persistent Volumes (PV)
PV 有點像是 storage 的 cluster，一個 PV 可以包含多個 Volume Type，PV 的設定如下:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-test
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /tmp/pv-test
```

+ `accessModes` 有三種模式:
    1. `ReadWriteOnce (RWO)`: 允許被單一 Node 掛載為 read/write 模式
    2. `ReadOnlyMany (ROM)`: 允許被多個 Node 掛載為 read 模式
    3. `ReadWriteMany (RWM)`: 允許被多個 Node 掛載為 read/write 模式

+ `persistentVolumeReclaimPolicy` 是指當 PV 被刪除時對於原有資料的處理，有三種可選:
    1. `Retain`: 保留資料
    2. `Delete`: 刪除資料(預設)
    3. `Recycle`: 也是刪除資料，不過實際上的差異筆者其實沒有很理解😅，所以這邊附上[官方文件連結](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

## Persistent Volume Claims(PVC)
PVC 是用來向 PV 發出存取得請求的，充當 PV 以及 Pod 之間的連結，當 Pod 不再需要 Volume 的時候也只要移除 PVC 就好，也可以有一些 Volume 的操作策略，設定如下:

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-test
spec:
  accessModes:
    - ReadWriteOnce
  selector:
    matchLabels:
      name: pv-test
  resources:
    requests:
      storage: 1Gi
```

上述可以設定一個基本的 PVC 出來，透過 selector 的方式取得 PV 資源，以及需求的資源當然不能超過 PV 設定的保留上限

可以透過 `kubectl get pvc` 來查看部屬好的 PVC，看 status 為 Bound 則表示部屬成功，有成功與 PV 綁定起來

## 在 Pod 的設定中如何使用
當 Volume 透過 PV 來設定管理，透過 PVC 來請求資源，那 Pod 就是透過 PVC 來綁定 Volume，關係上會像是 `Pod <-> PVC <-> PV`，所以必須確認 PVC 有確實綁好 PV，而 Pod 的設定其實也不複雜就是把 Volume 的設定換成 PVC 如下:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-nginx
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /tmp/conf
      name: pvc
  volumes:
  - name: pvc
    persistentVolumeClaim:
      claimName: pvc-test
```

# 結語
其實會發現 PVC 的內容很少，好像沒有必要特別抽離出來管理，主要應該是在使用雲端儲存空間時，有很多的設定細節才能體現出 PVC 的優點，不過 PV 的好處就顯而易見了，可以管理 Volume 的使用上限還有刪除 Volume 的策略