---
layout: post
title: 在 Docker container 中使用 Cron
date: 2021-04-14 12:10:38 +0800
category: others
img: cover/docker.jpg
description: 接續上一篇介紹 Cron，這一篇要來在 container 之中使用 Cron，雖然一般來說應該不太會遇到這個情況，但最近剛好就被我遇上了，因此順便記錄一下
lang: zh-TW
tags: [docker, cron]
---

{{page.description}}

其實說穿了也沒什麼特別的技巧，就是撰寫一份專用的 Dockerfile 就解決了，不過首先要先準備好需要的 crontab

## crontab
```bash
# 第一行與最後一行的空行不能移除，才能正常執行 crontab
* * * * * echo "Hello world"
#
```

## Dockerfile
```dockerfile
FROM ubuntu:18.04
COPY crontab /mycron
RUN chmod 777 /mycron
RUN apt update && apt install cron -y
RUN crontab /mycron
CMD cron start && tail -f /var/mail/mail
```

這樣就可以執行我們預先寫好的 crontab 並且列出 log，cron 會將執行的輸出送到 mail 因此可以讀取 mail 來拿到 log