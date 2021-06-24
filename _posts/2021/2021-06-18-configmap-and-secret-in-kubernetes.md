---
layout: post
title: "Kubernetes 初戰(五) 環境設定 ConfigMap、Secret"
date: 2021-06-18 13:42:19 +0800
category: deploy
img: cover/kubernetes.png
description: 過去使用 docker 在包裝應用程式時會一併把環境變數寫入到 Dockerfile 中作為服務的設定，但是難道每次更改設定都要重 build 一個 image 嗎，k8s 有提供比較方便的做法，那就是本篇要介紹到的 ConfigMap 與 Secret，可以直接從外部資源來設定容器內的環境變數
lang: zh-TW
tags: [kubernetes, deploy]
---

{{page.description}}


## ConfigMap

ConfigMap 是 k8s 中用來設定環境變數的其中一個資源，主要用於設定一些較不敏感的資料，直接以明文撰寫就好

### yaml

```yaml
## configmap.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-cm
data:
  TEST_VAR: testt
```

```yaml
## pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      env:
        - name: TEST_VAR
          valueFrom:
            configMapKeyRef:
              name: nginx-cm
              key: TEST_VAR
```

設定完後可以到 pod 裡直接輸入 `env` 查看環境變數有沒有被正確設定

## Secret

Secret 用法跟 ConfigMap 基本差不多，不過是用來設定一些敏感資料的，常見的例如:
+ 帳號密碼
+ 一些 token 或是 ssh key 等等

### yaml

```yaml
##secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: nginx-secret
type: Opaque
data:
  USERNAME: YWRtaW4gDQo=
  PASSWORD: YWRtaW4gDQo=
```

可以看到這邊設定的內容是經過編碼的，其實就是 `admin` 經過 base64 轉換後的內容

```yaml
## pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      env:
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: nginx-secret
              key: USERNAME
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
              name: nginx-secret
              key: PASSWORD
```

這樣設定之後一樣我們進到 pod 裡輸入 `env` 可以看到 secret 被解開後的值

## 結語

至此五篇拖了很長的時間才完成XD，而 k8s 初戰系列預計就到這邊了，接下來會是進階系列，其實也沒什麼明確的分水嶺，只是目前只是跟著團隊部署時在使用，沒太深入各種內容，單就以有使用到的表面的功能做點介紹跟紀錄，之後再針對一些議題去做各個資源的討論，只是不知道會拖稿多久XD