---
layout: post
title: "在 Linux 上週期性執行任務 - Cron"
date: 2021-04-14 10:05:21 +0800
category: linux
img: cover/linux.png
description: 操作 linux 時常會需要一些週期性的任務，例行的資料備份、測試腳本等等，這篇簡介一下 linux 上常用的例行任務工具 Cron
lang: zh-TW
tags: [linux, cron]
---

{{page.description}}

## Cron

是一款運行在 Unix like 環境下的週期性任務管理系統，可以基於時間點設定任務的執行，可以設定在特定的時間、日期、間隔之下執行任務

cron 的執行是基於 crontab 的指定格式來運作的，crontab 的文件會被一個 crond 的 daemon 持續監控確認是否需要執行任務，而每一項 crontab 的任務也被稱作 cron job

### crond daemon

crond 會週期性的去檢查 crontab 的任務內容是否須被執行，以每分鐘為週期去檢查以下路徑的 crontab

+ `/etc/crontab`: 為系統任務時間表
+ `/etc/cron.d/`: 包含系統層面的任務表，不同用戶間共用
+ `/var/spool/cron/`: 用戶透過 crontab 指令創建的任務表，會依照不同用戶分開

### crontab

crontab 指令是用來維護跟查看該用戶的任務表，一般不建議直接去修改任務表因此可以透過 crontab 的指令去操作，crontab 指令編輯的任務表會被存放在 `/var/spool/cron/` 路徑下

#### 指令

+ `crontab -e`: 編輯 crontab
+ `crontab -l`: 列出正在套用的 crontab 任務
+ `crontab -r`: 清除所有的 crontab 任務
+ `crontab <filename>`: 套用檔案裡撰寫的 crontab 任務
+ `crontab -u <user> <command>`: 切換不同用戶使用 crontab 指令

#### 語法

+ 用戶文件: 也就是透過 `crontab -e` 編輯的內容

```bash
# ┌──分鐘（0 - 59）
# │ ┌──小時（0 - 23）
# │ │ ┌──日（1 - 31）
# │ │ │ ┌─月（1 - 12）
# │ │ │ │ ┌─星期（0 - 6 => 周日到周六）
# │ │ │ │ │
# * * * * * 執行的任務
```

+ 系統文件: 被存放在 `/etc/crontab` 或 `/etc/cron.d/` 下的文件

```bash
# ┌──分鐘（0 - 59）
# │ ┌──小時（0 - 23）
# │ │ ┌──日（1 - 31）
# │ │ │ ┌─月（1 - 12）
# │ │ │ │ ┌─星期（0 - 6 => 周日到周六）
# │ │ │ │ │
# * * * * * 執行用戶 執行的任務
```

+ crontab 的任務執行是在符合時間表達式的條件時去執行任務，例如:
  + `1 * * * *`: 每天每小時的 1 分
  + `1,3,5,7 * * * *`: 每天每小時的 1,3,5,7 分
  + `1-7 * * * *`: 每天每小時的 1~7 分，每分鐘都會執行
  + `*/3 * * * *`: 每天每小時每 3 分鐘執行

+ 還有一些既定好的參數可以不用自己寫:
  + `@reboot`: 僅在開機執行一次
  + `@yearly` 或 `@annually`: 等同於 `0 0 1 1 *`
  + `@monthly`: 等同於 `0 0 1 * *`
  + `@weekly`: 等同於 `0 0 * * 0`
  + `@daily` 或 `@midnight`: 等同於 `0 0 * * *`
  + `@hourly`: 等同於 `0 * * * *`

這裡有一個線上的編輯器可以幫助撰寫 crontab 格式文件: [crontab.guru](https://crontab.guru/)

### 重新啟動服務

設定完 crontab 文件之後，還需要重啟 cron 服務才能真正套用，可以透過以下方式重啟服務:

+ `/etc/init.d/cron restart`
+ `sudo service cron restart`
+ `sudo systemctl restart cron`

### Cron Log

為了確認 crontab 有被正常運作可以透過一些方式來確認:

1. `grep CRON /var/log/syslog`: 直接查看相關的 syslog
2. `* * * * * command >> /cron.log 2>&1`: 透過 stdout 直接輸出訊息到指定檔案
3. 查看 `/var/mail/{user}`: 預設 crontab 會把結果發送 mail 給用戶

### 其他

#### Mail 通知

crontab 預設會將輸出 mail 給使用者，也可以透過設定送出給其他人，也可以設定空字串表示不發送

```bash
# 輸出送給 root
MAILTO=root
* * * * * echo "Hello"
```

#### 執行的 shell 環境

```bash
SHELL=/bin/sh
* * * * * echo "Hello"
```

#### 指定環境變數

有時候需要執行一些程式需要事先設定環境變數，例如執行 node.js

```bash
PATH=/bin:/usr/bin:/usr/local/bin
* * * * * node /test.js
```

#### 權限設定

可能由於一些安全性的考量，會希望只有特定的使用者可以使用 crontab，可以透過設定以下檔案來達到功效

+ `/etc/cron.allow`: 如果存在此檔案，列在之中的帳號才能使用 crontab
+ `/etc/cron.deny`: 如果存在此檔案，列在之中的帳號則都不能使用 crontab
+ 如果兩個檔案都不存在，則只有 root 可以使用 crontab

## 參考資料

+ [Cron - Wikipedia](https://zh.wikipedia.org/wiki/Cron)
+ [Crontab.guru](https://crontab.guru/)
+ [Linux 設定 crontab 例行性工作排程教學與範例 - G.T.Wang](https://blog.gtwang.org/linux/linux-crontab-cron-job-tutorial-and-examples/)
+ [Ubuntu 排程設定：Crontab 排程使用教學– 甲寬網路科技](https://jqnets.com/blog/ubuntu-%E6%8E%92%E7%A8%8B%E8%A8%AD%E5%AE%9A-%EF%BC%9Acrontab-%E6%8E%92%E7%A8%8B%E4%BD%BF%E7%94%A8%E6%95%99%E5%AD%B8/)
