---
layout: post
title:  "docker 初戰筆記"
date:   2020-11-07 15:00:13 +0800
category: deploy
img: cover/docker.jpg
description: 一直都有聽過 docker 跟容器化的技術，但一直都沒有機會碰到，最近也是工作關係開始接觸到，一些觀念跟操作都還不是很清晰，趕快先把所知先紀錄一下
lang: zh-TW
tags: [docker, deploy]
---

{{page.description}}

## 安裝
首先就請先到 [docker 官網](https://www.docker.com/){:target="_blank"} 去下載並安裝，安裝完應該要重啟，可能還會發生一些問題要裝東裝西的，就照指示一步一步完成吧

安裝完成啟動後會看到這個畫面，工具列右下角還有可愛的鯨魚
![](https://i.imgur.com/2GRGiA4.png)

## 簡介
Docker 的概念就是創造一個虛擬化的容器，這個容器可以將產品包含整個執行環境進行打包，可以幫助產品開發完成後部屬到機器上不會有其他環境的污染，可以運行在獨立乾淨的一個環境之下，確保開發完成的產品在可預期的空間之下執行

也可以搭配 Docker compose 和 K8s 做到 container 的管理還有自動化操作，不過那是後話了，筆者也只接觸過一點點而已，這邊就先簡單筆記一下 docker 的基本概念與操作

## 運作簡介
就如前面所說的，Docker 的運作單位是 **Container**，可以想像成就是一個輕量型的虛擬機，而跟一般的機器一樣，你可以從頭建置你的環境，從灌 OS 開始然後灌你要的工具，設定環境變數...等等

但如果每次都要建置一個環境有點太麻煩了，所以 container 其實可以套用別人已經做好的模板來用，可以幫你建置 OS、開發環境、開發工具.. 都幫你內建在裡面了，你只需要再把你的產品丟進去就好了，而這個模板，就被稱為 **Image**

## Image
Image 作為 Container 的模板存在，大概有下面三種取得方法:
1. 寫 DockerFile 然後下指令 build 出自己客製化的 image
2. 打包現有的 container 成 image
3. 從 [Docker Hub](https://hub.docker.com/){:target="_blank"} 上拉別人建好的 image 下來用

一開始通常都是從 Docker Hub 上拉一些基本款下來，然後灌成自己的環境後打包成 image，但開始有很多產品要部屬之後，就會開始寫 DockerFile 來建置，一方面好管理好維護，裡頭有什麼看 DockerFile 就一目瞭然，一方面也更方便我們執行一些客制化和初始化的行為

## Container
想像成是輕量的 VM，裏頭會包含 OS 再承載你的產品，打包起來後的概念也有點像是一個應用程式，然而環境是完全被隔離的

## DockerFile
可以想成是你想要的 container 的設定檔，可以指定你想基於某個 image 上面去建置，也可加入本地檔案、設置環境變數、執行腳本... 等等，寫好的 DockerFile 可以建置成 image 然後 run 成 container

```dockerfile
FROM node:latest    # 基於 node.js 的 image 去建置，裡面會內建 node.js
RUN mkdir /app      # 建置工作目錄
WORKDIR /app        # 把工作環境指定在 /app 下
ADD ./app /app      # 把本地目錄 ./app 的內容加入到 /app 下
RUN npm install     # 環境建置指令
ENV PORT=8080       # 設定環境變數
EXPOSE 8080         # 開放 8080 port 對外
CMD node index.js   # 容器開始執行指令  (如有多行 CMD 只會以最後一行為主)
```

上面就是一個簡單包含 node.js 的 DockerFile 範例，可以看見這個 DockerFile 的設定，比較要注意的是，`RUN` 跟 `CMD` 這兩個地方會比較匪夷所思一點，感覺在做一樣的事情

事實上呢 `RUN` 的設定是在 DockerFile 建立成 image 的時候就執行了，也就是說他是用來建置 image 的，而不是實際會在 container 上執行的

因此最終建立好的 image 裡面其實是包含我們的專案以及 `npm install` 後的 `node_module`，而最後的 `CMD` 就是 image 被 run 起來成一個 container 之後會執行的腳本，而這邊要注意就是 `CMD` 只能設定一行指令，設定多行的話，只會執行最後一行

DockerFile 也有一個設計是可以加入一個 `.dockerignore` 就像 git 一樣可以略過一些不想被加入的檔案

## 基本流程操作
接下來跑一下基本的流程，基於上面的 DockerFile 往下走，可以先在上面的 DockerFile 同目錄底下建一個 `app` 的資料夾，然後加入這兩個檔案

+ index.js

```javascript
const express = require('express');
const app = express();

app.get('/', function(req, res) {
  res.send("<h1>hello docker</h1>");
});
const server = require('http').Server(app);
const port = process.env.PORT || 8080;

server.listen(port, function() {
  console.log("listening on port: " + port);
});
```

+ package.json

```json
{
  "name": "hello-docker",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

上面弄好之後，鍵入下面指令，幫你建置 image 並且命名 tag 為 hello-docker

```shell
docker build . -t hello-docker
```

可以鍵入下面指令列出所有 image

```shell
docker image ls
```

然後再把我們的 image 執行成我們的 container

```shell
docker run -p 8000:8080 hello-docker
```

這邊有個要講一下的是 `-p 8000:8080` 的參數，是用來把這個 container 的 8080 port，host 到本機的 8000 port，要特別注意前後順序，前面是本機的 port，自己被這個雷到好幾次

成功跑起來之後就到 [http://localhost:8000](http://localhost:8000){:target="_blank"} 去看看結果吧

## 小結
到這邊大概對 Docker 有個基本概念，之後會再深入了解一下，Docker 架構下的其他組件，像是 docker daemon、docker network ...，Docker 有很多東西需要深入了解，然後還有 K8s 要玩，感覺永遠都學不完阿~😵

### 常用指令

+ `docker build . -t <imagename>`: 從 Dockerfile 建立 image 並貼上 tagname
+ `docker pull <reponame>:<version>`: 從 DockerHub 上拉下指定的 image
+ `docker image ls`: 列出所有 image
+ `docker container ls ` / `docker ps`: 列出正在運行的 container
+ `docker run -d <imagename> --name <containername>`: 在背景執行 image 成 container 並指定 container name，不指定會隨機生成
+ `docker container start <containername>`: 啟動 container
+ `docker exec -it <containername> bash`: 執行 container 的 bash
+ `docker container stop <containername>`: 停止 container
+ `docker container rm <containername>`: 移除 container