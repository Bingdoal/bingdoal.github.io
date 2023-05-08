---
layout: post
title: "[踩雷紀錄] nginx: open() "..." failed(13:Permission denied) ..."
date: 2023-05-08 10:47:41 +0800
category: deploy
img: cover/nginx.jpeg
description: "這次的問題是發生在 nginx 安裝好並且正常啟動之後，要訪問頁面卻跑出 403，去查看一下 error.log 才看到下面的錯誤訊息，紀錄一下怎麼解決與排查"
lang: zh-TW
tags: [踩雷紀錄,nginx,deploy]
published: true
---

## 問題描述

{{page.description}}

```shell
open() "/data/www/index.html" failed (13: Permission denied), client: 192.168.1.144, server: localhost, request: "GET / HTTP/1.1", host: "localhost"
```

訊息是寫沒有訪問檔案的權限，想想應該很直觀很好解決，下面來排查一下

## 解決方案

### 檔案不存在

第一先看看這個訪問的路徑存不存在，不存在的話當然會有問題，log 也有印出路徑在哪，檢查一下就好

### 使用者權限問題

權限問題分為兩個部分，一個是使用者是不是正確的，另一個是使用者是否有正確的權限

使用者身份問題可以看到 `nginx.conf` 應該前幾行就有寫了

```conf
user root;
```

通常我是直接寫 root，那 nginx 會嘗試用 root 執行，這樣檔案權限比較不會有問題

如果情況不允許用 root 執行，也可以檢查一下檔案權限，到路徑下執行下面指令修改權限

```shell
chmod -R 777 /data/www/index.html
```

暴力改成 777，改完重啟 nginx 應該就可以了

### SELinux 的保護機制問題

如果上面方法都試過了，還是不行，那有可能是 SELinux 在作祟，可以先用下面指令看看 SELinux 的模式

```shell
getenforce
```

如果顯示 `Enforcing` 那 SELinux 就會根據設定的規則阻擋外部訪問資源，可以使用下面指令暫時關閉

```shell
setenforce 0
```

但下次重啟就會再次開啟，可以透過修改文件 `/etc/selinux/config`

```
SELINUX=disabled
```

改完檔案下次重啟就不會再開了

關於 SELinux 的細節可以參考[鳥哥](https://linux.vbird.org/linux_server/rocky9/0140selinux.php)
