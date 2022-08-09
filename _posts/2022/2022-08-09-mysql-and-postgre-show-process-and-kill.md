---
layout: post
title: "[踩雷紀錄] 找出資料庫正在執行的 Query 並刪掉: 以 MySQL 以及 Postgres 為例"
date: 2022-08-09 11:54:58 +0800
category: others
img: cover/database.jpg
description: 最近工作中遇到測試用的資料庫太多人連線導致效能緩慢，查了一下發現有很多閒置的連線沒有被關閉，可能有一些 DBeaver 之類的工具長期佔用連線，於是決定來清掃一番，查了一下資料庫本身就有支援指令可以使用，分別記錄一下 MySQL 跟 PostgreSQL 的操作
lang: zh-TW
tags: [踩雷紀錄, database]
published: true
---

{{page.description}}


## MySQL

```sql
show processlist;
kill <id>
```

MySQL 直接提供特殊的語法供使用，可以列出正在使用中的 process 然後直接拿 id 來 kill

## PostgreSQL

```sql
SELECT * FROM pg_stat_activity
```

PostgreSQL 則是有一個虛擬的表用來查詢，可以查詢到各個連線的 pid 再透過 terminal 自行去結束掉 process