---
layout: post
title: Kubernetes 初戰(一) Pod、Node、Service、Deployment
date: 2021-02-10 18:28:07 +0800
category: deploy
img: cover/kubernetes.png
description: 接觸容器化的部屬之後當然就是輪到 K8s 登場啦，不過 K8s 的內容實在有夠複雜，因此決定分篇撰寫各個方面，由於筆者也只是初學，所以也不會探討到太深的底層原理，撰寫的視角也是以初學者的角度去看的，可能會有一些誤解，還請不吝指教
lang: zh-TW
tags: [kubernetes, deploy]
---

{{page.description}}

## 簡介

Kubernetes(後簡稱K8s) 是由 Google 開發設計並捐贈給 [Cloud Native Computing Foundation(CNCF)](https://www.cncf.io/)，用於自動化部屬、擴充、管理**容器化的應用程式**的一套系統。

K8s 可以做到幾件事情:
1. 同時部屬多個 container 到一台或是多台機器上
2. 管理並監控多個 container 的狀態。最基本的當 container crash 了，K8s 可以設定自動重啟來確保服務不中斷。
3. 從 container 的角度上做到 Loading balance 跟反代理

簡而言之 K8s 是用於管理 container 的系統，而針對產品開發的各種需求 K8s 將管理 container 的各種功能切分成了許多的元件，這篇會先簡單介紹 Node, Pod, Service, Deployment
> 光搞懂這四個元件就弄的我一個頭兩個大了

## Pod
講到 K8s 一定會先提到 Pod，Pod 為 K8s 的運作最小單位，是 K8s 用來乘載 container 的環境，Pod 中可以含有一個或多個 Container，並且同一個 Pod 中的 container 們共用彼此的檔案系統與 network namespace，意味著在同一個 pod 中的 container 可以用 `localhost:port` 來傳輸資料

![]({{site.baseurl}}/assets/img/k8s-pods.png)

理想上可以將彼此依賴程度高的應用放在同一個 Pod 中，不過筆者目前常用的情境多是一個 Pod 一個 Container，K8s 各個部件的設定方式都是透過 yaml 的格式去撰寫的，下面是一個基本的 Pod yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: express-ts
  labels:
    app: express-ts-label
spec:
  containers:
  - name: express-ts
    image: bingdoal/express-ts
    ports:
    - containerPort: 3000
```
+ Label 用來設定 Pod 的標籤，後面給其他元件用來定位資源

> 有關於 container 的設定中用到的 image 是來自筆者的 [container-demo](https://github.com/Bingdoal/container-demo)，就是一個基本的 express，是另外 side project 的早期實驗品拿來用而已

上面的 yaml 是針對單一 Pod 的設定，是用來建立獨立的 Pod，但多數我們不會這樣單獨使用，主要有幾個問題:
1. 獨立的 pod 若是發生問題時(例如: node failure)，k8s 不會協助恢復其正常的狀態
2. 若 pod 所在的 worker node 因為資源不足或是進入維護狀態時，pod 不會被自動移到其他正常的 node 並重新啟動

因此在 k8s 中，提供了一個更 high level 的抽象概念被稱為 controller ，用來處理上面的問題，而在使用上也應該透過這些 controller 來管理 pod

#### 指令操作

```bash
kubectl create -f ./pod.yaml              # 建立 POD
kubectl get pod                           # 印出所有的 pod 資訊
kubectl edit pod <pod-name>               # 編輯 pod 的設定，也就是剛剛那份 yaml
kubectl describe pod <pod-name>           # 列出該 pod 的詳細資訊
```

後面 Service、Deployment 的 get、edit、create、describe 指令都大同小異，所以後面就只會列出比較特別的指令

於 Pod 來說比較特別的是:

```bash
kubectl port-forward express-ts 3001:3000
```
+ 可以把 pod 的服務幫助 host 在本機上，host 上去之後到 [http://localhost:3001](http://localhost:3001) 可以看到結果

```bash
kubectl exec -it express-ts -- bin/bash
```
+ 類似 docker exec 的概念可以執行 pod 中 container 的 bash

## Node

Node 通常代表的是一個虛擬機或是實體機器，是用來運行 Pod 的環境，只要上面有 Container Engine 跑得起 Pod 就可以加入當作 K8s 的 Node，下面的圖是一個完整的 K8s cluster 架構

![]({{site.baseurl}}/assets/img/components-of-kubernetes.svg)

可以看到一個 K8s 的架構中可以存在多個 node，當運行時資源不足的時候會自動找尋有多餘資源的 Node 部屬 Pod，而在 Node 中又可以分為三個部件 kubelet, kube-proxy, Container Runtime，下面簡單介紹三者內容
1. kubelet
   + 主要的 Node 代理者，主要負責該 Node 上所有 Pod 的狀態以及跟 apiserver 溝通
2. kube-proxy
   + 負責用來代理每個節點間溝通，可以做到簡單的 TCP、UDP、SCTP 的轉流，還可以設置 cluster 內的 DNS
3. Container Runtime
   + 用來運行 Pod 的環境

實際上對於 Node 的理解目前也還很淺，目前實作上也還沒遇到多 Node 的使用，因此只淺談到這裡

## Service
上面提及 Pod 可以透過 `port-forward` 的指令 host 到本機上，但只能在前景執行，而且每個 pod 都要去執行一次也不太合理，所以 Service 這個元件就誕生了，Service 主要可以想成是 Pod 的反代理機制，用來定義 Pod 如何被連線以及存取，下面是 Service 的 yaml 設定:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: express-ts-svc
spec:
  selector:
    app: express-ts-label
  type: NodePort
  ports:
    - protocol: TCP
      port: 3001
      targetPort: 3000
      nodePort: 30390
```
1. NodePort 是 Service 的一種類型，可以將 Service 暴露給外網，其他還有 ClusterIP、LoadBalancer。
2. Selector 可以做出資源定位，根據符合的 label 去抓取資源

## Deployment

一個基本也最常用的 Pod controller，功能可以一次啟用多個 Pod，並且持續監控 Pod 的狀態，一但 Pod 當機無法正常提供服務，會立刻替換備援的 Pod 並且重啟，一般來說都是使用 Deployment 來部屬 Pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: express-ts-depl
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: express-ts-label
    spec:
      containers:
        - name: express-ts
          image: bingdoal/express-ts
          ports:
            - containerPort: 3000
  selector:
    matchLabels:
      app: express-ts-label
```
1. replicas 指同時啟動多少個 pod
2. template 則設定由該 deployment 建立起來的 pod 長什麼樣子

## 資源操作指令指令整理

```bash
# 查看資源
kubectl get <resource>
kubectl describe <resource> <resource-name>
# 創建資源
kucectl create <resource> ...
kubectl create -f <config-yaml>
kubectl apply -f ./
# 修改、刪除資源
kubectl edit <resource> <resource-name>
kubectl delete <resource> <resource-name>
# all
kubectl get all
kubectl delete all --all
```
+ 上面 Pod、Service、Deployment 都可以透過這些指令來進行操作

### 版控
```bash
kubectl rollout history <resource> <resource-name>
```
+ 當對於 K8s 的各個資源進行編輯操作之後，K8s 會紀錄版本間的變更，上面指令可以列出編輯的歷史紀錄

```bash
kubectl rollout undo <resource> <resource-name> --to-reversion=<reversion>
```
+ 印出歷史紀錄後，可以根據版號進行 rollback

## 小結
其實在進到後端跟 DevOps 的領域之前就常聽到 K8s 的盛名，但真正工作接觸後才知道這東西有多複雜，可以感受到這套系統設計的極為精細，本篇只為 K8s 的一開始基本操作，有關的原理探討其實都非常不足，而現階段幾乎都是邊操作邊理解，這也代表自己還有很多不足的地方要多加學習，希望後面能將整個運作原理都補上