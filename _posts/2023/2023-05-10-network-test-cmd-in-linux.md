---
layout: post
title: "Linux 上的各種測試目標網路的指令: ping, telnet, curl, netcat"
date: 2023-05-10 10:08:48 +0800
category: linux
img: cover/linux.png
description: "在 linux 也生活一段時間了，常常在架設部署新環境的時候都需要做些網路測試，看看機器網路是否通暢，服務的監聽狀況，確認一切沒問題才能繼續往下進行，這篇就來紀錄一下各種測試用的指令"
lang: zh-TW
tags: [network, linux]
published: true
---

{{page.description}}

## ping

`ping` 應該是在最一開始測試網路最常用的指令，多數系統也都會內建，本身就是非常簡單打一個 [ICMP](https://zh.wikipedia.org/zh-tw/ICMP) 的封包給目標 IP 位址

不過要注意的是對方主機必須有開啟接收 `ping` 指令的功能，也因為就只是拿來測試 ICMP 通不通目前大多都不會用這個指令來做測試，畢竟我們的重點主要會放在自己的服務有沒有正常通順

另一個可能用到的地方是可以直接 `ping <domain name>` 可以看到 domain 解析後的 IP

## telnet

`telnet` 本身也是一種 TCP/IP 的協定，鼎鼎大名的 PTT 就是一個 telnet server，可以嘗試看看直接輸入 `telnet ppt.cc` 就可以看到登入頁了

不過也由於他本身是 TCP/IP 的文字協定解析，個人用 `telnet` 都是拿來測試目標主機的 tcp port 有沒有正常監聽並通暢

使用下面指令

```shell
telnet <ip> <port>
```

連通的話可能會像下面

```shell
$ telnet localhost 6379

Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
```

比較困擾的可能是不知道怎麼跳出，鍵入 `^]` 之後輸入 `q` 就好了

連不通則是出現

```shell
$ telnet localhost 6370

Trying 127.0.0.1...
telnet: connect to address 127.0.0.1: Connection refused
Trying ::1...
telnet: connect to address ::1: Connection refused
telnet: Unable to connect to remote host
```

這個指令本身輕量好安裝又簡單使用，可以用來簡單測試 TCP 的監聽有沒有正常，是我很常用到的指令

## curl

`curl` 相信有在開發網頁應用程式的人都不會陌生，主要是用來發送 HTTP 協定的指令工具，不過其實也支援其他的協定，有機會希望可以詳細介紹一篇，這邊只針對 HTTP 的使用作介紹

```shell
curl [options] <URL>
```

基本 `curl <URL>` 就是直接執行一個 GET，什麼都不帶，http 常用的 options 大概有下面

| 短參數 |    長參數    |                         範例                         |                                 描述                                  |
|:------:|:------------:|:----------------------------------------------------:|:-------------------------------------------------------------------:|
|  `-H`  |  `--header`  |           `-H "Host: https://example.com"`           |                          HTTP 的各種 header                           |
|  `-L`  | `--location` |              `-L "https://example.com"`              |    HTTP 的 url，跟直接寫的差別是如果回覆是 301 會自動跳轉到新的內容    |
|  `-X`  | `--request`  |                     `-X "POST"`                      |                          指定 HTTP 的 method                          |
|  `-u`  |   `--user`   |                  `-u user:password`                  |                      適用於 http basic auth 登入                      |
|  `-d`  |   `--data`   |          `-d 'account=user&password=1234'`           |                        帶入請求的 request body                        |
|  `-F`  |   `--form`   | `-F "user=me" -F "file=@filePath;filename=test.txt"` | 專給 form-data 格式使用，除了帶入內容還可以個別指定屬性，常用在上傳檔案 |
|  `-o`  |  `--output`  |              `-o "~/download/test.txt"`              |                    把請求的回覆存成檔案，常用在下載                    |
|  `-b`  |  `--cookie`  |                 `-b "TOKEN=123456"`                  |                              帶入 cookie                              |
| `-vvv` |    `-vvv`    |               `curl -vvv ifconfig.me`                |            完整印出請求以及回應的詳細資訊，debug 可能會用到            |

另外補充 `curl ifconfig.me` 可以拿到機器的對外 IP

下面是一個常見的基本架構範例

```shell
curl -X POST -L http://localhost/login \
    -H "Content-type: application/json" \
    -d '{"account":"user","password":"1234"}'
```

另外補充如果用 `curl -o` 下載檔案被中斷可以透過下面指令重新續傳，不過路徑不能變喔

```shell
curl -C - -o ~/index.html -L http://localhost/index.html
```

## netcat

netcat 指令使用為 `nc` 可以用來測試 TCP 以及 UDP 基本上這應該是最全能的測試工具，也因為太全能了，功能多到有點難記，甚至可以直接拿來當簡易的 client & server

基本上這個指令是可以寫好幾篇的，這裡就簡單介紹一些就好

### 連線測試

```shell
nc -v 127.0.0.1:8787 # 測試 tcp 連線
echo -n "123" | nc -u -w1 <ip>:<port> # 傳送內容測試 udp 連線，w1 為 timeout 1 秒
```

### 監聽連線

```shell
nc -l 0.0.0.0 8787 # 開啟 8787 port 的 tcp 監聽
nc -lu 0.0.0.0 8787 # 開啟 8787 port 的 udp 監聽
```

不過預設只會接收一個連線可以加上 `-k` 保持連線

### Port scan

nc 還可以用來做簡易的 port scan

```shell
nc -vzn 127.0.0.1 2000-10000 2>&1 | grep succeeded # 掃描 2000-10000 tcp port，並且只輸出成功部分
```

還有更多奇技淫巧，之後有用到再來寫
