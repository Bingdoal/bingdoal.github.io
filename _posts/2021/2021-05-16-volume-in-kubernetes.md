---
layout: post
title: Kubernetes 初戰(二) 儲存空間 Volume
date: 2021-05-16 09:00:26 +0800
category: deploy
img: cover/kubernetes.png
description: 在一般的觀念裡，container 相當合適作為 stateless application 之用(例如：api service)，但由於 stateful service 的需求也是存在的(例如：各種資料庫、檔案存放伺服器…等等)，因此 Kubernetes 就額外增加了一些相關的機制，讓 pod 也開始能夠承載 stateful application，本篇就簡單接紹 K8s 裡的 Volume 機制，看看是如何將資料保存下來
lang: zh-TW
tags: [kubernetes, deploy]
---

{{page.description}}

## Volume
為了保存 container 的狀態而誕生的機制就是 Volume，在 Docker 中其實也有這個機制可以看看之前的文章 [利用 Docker volume 儲存 container 的資料](https://bingdoal.github.io/deploy/2021/01/docker-container-store-data-at-local/) 簡單來說 K8s 的 Volume 主要是為了解決:
1. 資料在 container 中無法存續，重啟 container 後將會遺失
2. pod 內的多個 container 有共享檔案的需求

K8s 支援幾種不同的 Volume Type 下面會各自介紹，詳細也可見[官方文件](https://kubernetes.io/docs/concepts/storage/volumes/):

### emptyDir
簡單的 Volume 掛載可以直接在 Pod 中設定，下面是 EmptyDir 的設定:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: empty-dir-nginx
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /tmp/conf
      name: empty-dir
  volumes:
  - name: empty-dir
    emptyDir: {}
```

`emptyDir` 顧名思義一開始會是一個空的目錄用來暫存或是共享資料，雖說是一種 Volume 但是在 Pod 被移除的時候資料依舊會被一並刪除，主要用途是當 Pod 崩潰被 Deploy 這類的 controller 重啟時會保留，可以看見 `emptyDir` 只有指定 mountPath 而沒有設定機器(Node)上的存放位置，就是因為沒有這個需求

`emptyDir` 主要會應用在:
+ 暫存資料
+ 多 container 的共用資料

+ 這邊要注意的是 `volumeMounts` 裡的 `name` 要跟 `volumes` 的 `name` 對應上，`volumes` 裡是定義這個 volume 的 spec 然後在 `volumeMounts` 裡進行掛載

### hostPath
第二個來看到的 Volume 種類是 hostPath，以下是 yaml 的設定方式:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: host-path-nginx
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /tmp/conf
      name: host-path
  volumes:
  - name: host-path
    hostPath:
      path: /tmp/host_path
      type: DirectoryOrCreate
```

`hostPath` 是一個很直覺的 Volume 種類，就是將 container 內的指定路徑跟實體機(Node)上的指定路徑綁定在一起，Docker 的 Volume 也比較像這個模式，而 `hostPath` 設定裡還有一個 type 可以做一些路徑的檢查如下表:

| type                | 描述                              |
| ------------------- | --------------------------------- |
| `(空)`              | 不做任何檢查                      |
| `DirectoryOrCreate` | 路徑必須為目錄若不存在則建立      |
| `Directory`         | 路徑必須為已存在目錄              |
| `FileOrCreate`      | 路徑必須為檔案若不存在則建立      |
| `File`              | 路徑必須為已存在檔案              |
| `Socket`            | 路徑必須為已存在的 UNIX socket    |
| `CharDevice`        | 路徑必須為已存在 character device |
| `BlockDevice`       | 路徑必須為已存在的 block device   |

實際上後三個 type 筆者也不確定是用在什麼情境，詳情可以看[官方文件原文](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)

跟 `emptyDir` 的差異上來說，可以事先在 `hostPath` 裡放入資料再進行掛載，且 Pod 重啟資料也不會遺失，符合 stateful 的 service 使用情境

但通常來說不會直接用 `hostPath` 來設定 Volume，會再透過 `Persistent Volumes (PV)` 以及 `Persistent Volume Claims (PVC)` 來進行 Volume 的管理，大多用在測試階段才會先用 `hostPath` 來使用

+ 如果說需要在一個 Volume 下掛載多個路徑也可以透過 `subPath` 的方式，如下所設定:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: host-path-nginx
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /tmp/conf
      name: host-path
      subPath: sub1
    - mountPath: /tmp/conf
      name: host-path
      subPath: sub2
  volumes:
  - name: host-path
    hostPath:
      path: /tmp/host_path
      type: DirectoryOrCreate
```

照這樣設定可以在同一個 `hostPath` 之下掛載兩個目錄，而不會把資料都混在一起
### 其他
其他的 Volume Type 則主要是用於雲端上的儲存空間，可以透過 K8s 直接將 Volume 掛載到雲端上，如下面的三種:
1. gcePersistentDisk
2. awsElasticBlockStore
3. azureDisk

分別在各個平台上都有各自的 Volume Type，K8s 這方面的支援也真的是很到位，不過目前筆者還沒有用到雲端儲存空間的經驗這邊就不細談了

## 結論

此篇簡單講解 k8s 的 Volume 在 Pod 上的設定還有種類，下次預計會介紹下 `Persistent Volumes (PV)` 和 `Persistent Volume Claims (PVC)`，更完整得補完 K8s 在資料保存方面的機制
