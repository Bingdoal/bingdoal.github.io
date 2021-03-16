---
layout: post
title:  "利用 Docker Compose 一次啟用多個服務"
date:   2020-12-25 09:00:00 +0800
category: deploy
img: cover/docker.jpg
description: 之前寫過 docker 的初戰簡介學會了一步步透過 Dockerfile 來建立需要的 Image，但每個服務都要寫一份 Dockerfile，每份都要個別維護實在有點麻煩，今天就來看看怎麼利用 docker-compose 來一次維護多個服務吧
lang: zh-TW
tags: [docker, deploy]
---

{{page.description}}

# 前言

之前介紹過怎麼一步步把一個 Dockerfile 建立成 Image 然後 run 成一個 Container，試想今天如果一個產品上可能包含三到五個服務都還蠻正常的，那每次都要去更改這三到五個 Dockerfile，然後一個個建立成 Image 再一個個 run 成 Container

想想就覺得麻煩，而且幾個服務之間可能還需要彼此關聯，分開在不同的地方維護的話也會增加不必要的成本，因此就出現了 Docker Compose 這個方便的工具啦

# 安裝

在 Windows 環境下的話安裝完 Docker Desktop 之後就會內建在裡面了

# `docker-compose.yml`

跟 Dockerfile 類似，在我們使用 docker-compose 之前需要先寫一份設定檔或稱作是描述檔，這份描述檔的檔名為 `docker-compose.yml`，一個簡單的範例如下:

```yaml
version: '2'
services:
  db:
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
  adminer:
    image: adminer
    restart: always
    ports:
      - 8000:8080
```

然後在同個目錄下輸入指令

```bash
docker-compose up -d
```

可以一次啟動以上的全部服務，按照上面的啟動的話，到 [http://localhost:8000](http://localhost:8000) 可以看到 mysql admin 的服務

也可以指定想要啟動的服務，其他指令跟 docker 用法也都大同小異像是

```bash
docker-compose ps
docker-compose images
docker-compose logs
docker-compose stop
docker-compose rm
docker-compose exec
...
```

使用情境應該大致好很理解，需要注意的是 docker-compose 的指令必須使用在與 `docker-compose.yml` 同個目錄下才會有作用，也只會作用在該 `docker-compose.yml` 所描述的範疇之下

使用 docker-compose 的最大好處就是只需要維護一份描述檔就好，並且可以透過 docker-compose 的指令統一管理，所以重點就應該是在描述檔該怎麼寫了

```yml
version:
services:
    myservice: # 自己取的 service 名稱
        container_name: my_first_service  # 建立起來的 container 名稱，若沒取會按照 service name 後面加流水號
        image: my_image # 建立 container 用的 image
        volumes: my_volume:/home/myservice # 指定 volumes 的位置
        ports:
            - "8000:8080" # port 轉發的設定，記得前面的 8000 是本機 port，後面的 8080 是 container 的 port
        restart: always # container crash 時是否重啟，參數有 no, always, on-failure, unless-stopped
        # on-failure: 5 , unless-stopped: 5 這兩個參數需要在後面補充一個 int，表示重啟特定次數
        hostname: myservice # 如果其他服務需要用到該服務的接口，由於無法取得 container 啟動後的 ip 可以藉由 hostname 來達到這個效果
        environment: # 設定環境變數
            - PORT: 8080
    mysecondservice:
        container_name: my_second_service
        image: my_image2
        depends_on: # 表示該服務必須依賴以下的服務，docker-compose 會優先啟動下面的服務才執行這個 container
            - "myservice"
        environment:
            - API_SERVER: myservice

volumes: # 設定該 docker-compose 的 volume
    my_volume:
        external: true
```

以上就是個人常用到的參數啦，其他更詳細的 `docker-compose.yml` 語法可以參考以下:

[Github - compose-spec](https://github.com/compose-spec/compose-spec/blob/master/spec.md)
[docker docs - Compose file version 2 reference](https://docs.docker.com/compose/compose-file/compose-file-v2/)

# 結

看完以上可以發現其實，`docker-compose.yml` 也不能完全取代 `Dockerfile`，像是 `ADD`, `WORKDIR`, `RUN` 等等的語法都沒有對應的可以使用，所以我們可以理解到 `docker-compose` 的目的在於統一管理 `container` 執行面的狀態，而 image 的建置狀態則交給 `Dockerfile`，`Dockerfile` 在建立完 `image` 之後就可以不用動了，讓 `Dockerfile` 的概念更接近應用程式的設定，而 `docker-compose` 則是環境的設定，讓兩者的關係明確，分工獨立