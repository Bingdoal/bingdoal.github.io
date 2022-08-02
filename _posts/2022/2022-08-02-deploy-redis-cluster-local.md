---
layout: post
title: 在本地部署 redis cluster
date: 2022-08-02 13:58:34 +0800
category: others
img: cover/redis-cover.png
description: Redis 是一個常用來作為暫存資料的記憶體資料庫，由於是資料是在記憶體操作的因此斷電就會遺失資料，為求穩定大型系統都會有 cluster 的建置防止單點錯誤，一般來說 cluster 會把不同的 node 建立在不同的機器上，這篇是為了在本地進行測試才會在本地建置環境
lang: zh-TW
tags: [redis]
published: true
---

{{page.description}}

## 事前準備
1. redis 的命令列工具: redis-cli, redis-server
2. redis cluster node 的設定檔:

```conf
# rediscluster.conf
cluster-enabled yes
cluster-config-file node.conf
cluster-node-timeout 5000
protected-mode no
appendonly yes
```

## 直接建 redis cluster 在本地

由於 redis server 啟動時會在目錄下建立各種設定跟暫存資料，所以不同的 redis server 必須在不同的目錄啟動

我自己的目錄結構如下:

```
├─7001
├─7002
├─7003
├─7002
├─7004
├─7005
└─rediscluster.conf
```

目錄名稱代表的是 node 的 port 這樣比較好區分，需要用到 6 個 node 的原因是 cluster 的最小基數是 3，而每個 cluster node 必須要有至少一個 replica，所以最低需要有 6 個 node

接著 `cd` 到每個目錄下執行下面指令:

```sh
redis-server ../rediscluster.conf --port 7001 --cluster-announce-bus-port 17001 --daemonize yes
```

記得 port 要每個目錄不一樣，執行完之後用下面指令確認一下有沒有正確跑起來

```sh
ps -aux | grep redis
```

這時候雖然每個 redis server 都已經起來了，但他們彼此並不知道對方的存在，cluster 還沒有建立起來，這時候需要進行一個初始化

```sh
echo "yes" | redis-cli --cluster create 127.0.0.1:7001 \
127.0.0.1:7002 \
127.0.0.1:7003 \
127.0.0.1:7004 \
127.0.0.1:7005 \
127.0.0.1:7006 --cluster-replicas 1
```

看到下面訊息的話表示初始化成功了

```sh
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

這個時候可以實際來操作看看能不能運行，操作下面指令

```sh
redis-cli -p 7001 -c cluster info # 查看 cluster 狀態
redis-cli -p 7001 -c cluster nodes # 查看 cluster 的 node
```

1. `info` 可以用來確認 cluster 有沒有運行成功，確認一下 `cluster_state:ok` 以及 `cluster_known_nodes:6` 是正確的
2. `nodes` 則可以看到 cluster 內的 node 有哪些，確認所有的 node 都在裡面，也可以看到哪些是 master 哪些是 slave

然後可以用下面指令連接到 cluster 進行操作

```sh
redis-cli -p 7001 -c
```

之後的 set get 的操作就跟單台 redis 沒有兩樣了

---

## 結語

這篇說明要怎麼在本地架設 redis cluster，下篇會說明怎麼用 spring boot 進行連接以及踩到的雷