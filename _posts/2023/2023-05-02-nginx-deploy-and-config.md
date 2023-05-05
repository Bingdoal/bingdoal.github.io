---
layout: post
title: "Nginx 基本安裝與設定"
date: 2023-05-02 16:29:28 +0800
category: deploy
img: cover/nginx.jpeg
description: "nginx 是一個常見的 web server，主要用於作為反代理以及負載平衡器，不過本身也可以做一些請求的前處理，以供後面的程式使用，簡單紀錄下安裝與設定，也會用 netcat 做個簡單的測試"
lang: zh-TW
tags: [nginx,deploy]
published: true
---

{{page.description}}

## 安裝

每個系統的安裝方式都各不相同，這裡就直接附上[官方](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)內容吧

安裝完之後應該會自動起動，可以直接訪問本地連結 [http://localhost:8080](http://localhost:8080)

沒意外會看到下面畫面

![nginx test]({{site.baseurl}}/assets/img/nginx-index.png)

## 基本操作

下面列出一些常用的 nginx 指令

```shell
nginx -t # 測試設定檔有沒有問題，也可以查看設定檔的路徑
nginx -c <nginx.conf path> # 設定預設設定檔的路徑
nginx # 啟動 nginx
nginx -s reload # 不重啟 nginx 的情況下重新載入設定檔
nginx -s stop # 關閉 nginx
nginx -V # 秀出 nginx 的各種資訊，各種檔案路徑一覽無遺
```

可能會好奇上面的畫面是從哪裡來的，可以透過 `nginx -V` 找到 `--prefix` 的參數路徑，路徑下的 `index.html` 就是了

## 設定

設定檔路徑可以透過 `nginx -t` 或是 `nginx -V` 找到

下面是一個稍作修改的預設 `nginx.conf`，有去除掉一些不重要的註解還有開啟 log 紀錄

```conf

#user  nobody;
worker_processes  1;

error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       8080;
        server_name  localhost;s

        #charset koi8-r;

        access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }

    include servers/*;
}
```

除了開啟 log 之外可以注意到最後一行我加上了 `include servers/*;`，可以想成是把 `servers` 這個資料夾下的 `.conf` 內容包含進這裡面，這樣可以方便做設定的分類

參數應該都蠻好理解的，特別說明下 location，這裡的 root 指的是靜態網頁放的位置，這邊只有寫 html 是因為有個預設的路徑，可以透過 `nginx -V` 找到 `--prefix` 的路徑就是了

另外可以注意到在寫 access_log 的部分有寫一堆變數來帶入內容，[這裡](https://www.javatpoint.com/nginx-variables) 有列出所有的 nginx 變數可以用在設定之中

### proxy

nginx 也常作為 proxy server 下面來客製化一個簡單的設定檔在 `servers/` 裏面

```conf
# jekyll.conf
server {
    listen 8888;
    server_name localhost;
    location / {
        proxy_pass http://127.0.0.1:4000;
    }
}
```

透過設定去做反代理到我部落格的測試服務上，只要透過訪問 [http://localhost:8888](http://localhost:8888) 就可以連接到本地 4000 port 的 jekyll 服務

接著我們還可以在 nginx 中間轉介的過程加點料

像是

```conf
# api-server.conf
server {
    listen 8888;
    server_name localhost;
    location / {
        add_header 'Access-Control-Allow-Origin' '*';
        proxy_set_header REMOTE-IP $remote_addr;
        proxy_pass http://127.0.0.1:3000;
    }
}
```

在前後端分離的架構下，透過 nginx proxy 到後端 api 是常見的做法

但是在瀏覽器的規範下前端時常會遇到 [CORS](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS) 的問題，這時候可以透過設定 header `Access-Control-Allow-Origin` 來避免

`proxy_set_header REMOTE-IP $remote_addr;` 在一些場景下我們後端需要拿到使用者 IP，但在透過 nginx proxy 的情況下，後端程式只能拿到 nginx 的 IP，這時候可以透過設定的方式從 header 中拿到使用者 IP，基本上只要有透過 nginx 都會設定一下，才能讓後端拿到正確的使用者 IP

### load balance

另一個常用場景是負載平衡，nginx load balancer 有三種模式

+ round-robin：預設的輪詢方式
+ least-connected：當連線進來時會導向連線較少的 server
+ ip-hash：依據 IP 的 hash 結果來分配到不同 server

下面是個簡單範例

```conf
upstream api {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
}

server {
    listen       8888;
    server_name  localhost;

    location / {
        add_header 'Access-Control-Allow-Origin' '*';
        proxy_set_header REMOTE-IP $remote_addr;
        proxy_pass http://api;
    }
}
```

如果想要更改模式只要加在 `upstream` 裏面

```conf
upstream api {
    least_conn;
    server 127.0.0.1:3000 weight=3;
    server 127.0.0.1:3001 weight=2;
}

upstream api {
    ip_hash;
    server 127.0.0.1:3000 weight=3;
    server 127.0.0.1:3001 weight=2;
}
```

另外設定 weight 就是權重的概念，越高的會乘載越多的連線

---

## 結語

nginx 是個輕量好設定的代理伺服器，也可以做到簡單的負載均衡，但是也別忘了，他本身就是一個 web server，可以直接用來訪問靜態網頁，或是把 SPA 打包好後放在目錄下使用，功能一應俱全，基本上是架站必備啊
