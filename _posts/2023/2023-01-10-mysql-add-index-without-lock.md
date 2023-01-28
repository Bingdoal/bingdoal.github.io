---
layout: post
title: "[踩雷紀錄] MySQL 新增 index 不鎖表"
date: 2023-01-10 14:15:24 +0800
category: others
img: cover/database.jpg
description: "最近一個需求要加上 index 以增加查詢效率，不過那個表的資料有點太大了，指令一下去整張表完全不能操作，頓時辦公室哀鴻遍野，嚇得我也是不知所措"
lang: zh-TW
tags: [others, database]
published: true
---

{{page.description}}

原來 table 加上 index 的時候會把整張表鎖起來，當表太大張的時候執行時間過就，鎖的時間太久，各種請求都會直接 timeout

所以要改寫一下我們的 SQL 語法，其實很容易

## 原語法

```sql
ALTER TABLE `Table` ADD INDEX `index_name` (`Column`);
```

## 不加鎖語法

```sql
ALTER TABLE `Table` ADD INDEX `index_name` (`Column`) , ALGORITHM=INPLACE, LOCK=NONE;
```

---

## 結語

對於 DB 各種機制尤其是 transaction, index, lock 的機制一直都有點不夠熟悉，找時間要來好好惡補一下
