---
layout: post
title: "[踩雷紀錄] Java 開發上遇到的 FTP 主動式、被動式"
date: 2021-03-12 15:21:55 +0800
category: backend
img: cover/ftp.jpg
description: 最近一個案子中在 web server 有用到 FTP 的功能，而在開發中卻發現在本地測試時能夠正常運作的 FTP 功能，部屬到線上平台之後卻通通失靈了，查了一下才知道 FTP 有分為兩種不同的模式，而在特定的情況下一些模式是無法作用的。
lang: zh-TW
tags: [java, ftp, protocol, 踩雷紀錄]
---

{{page.description}}

## FTP 模式
首先先理解一下 FTP 連線機制上其實有兩種模式:
1. 主動式 (POST/Active)
2. 被動式 (PASV/Passive)

分別運用在不同的情境之下，在 FTP 中定義兩個 port 通常是 20、21 來進行溝通，其中 21 被視為 Command Port，20 被視為 Data Port，其功用正如其名

### 主動式

最開始的 FTP 設計都是由主動式來進行溝通，溝通的方式如下:
1. 由 client 端(通常是 1024)發出連線請求與 FTP server 的 Command Port 建立連線，並發出命令
2. 同時 client 會開啟一個 Port (通常為 1025)等待接聽由 FTP server Data Port 傳過來的資料
3. server 端回應命令並由 Data Port 傳送資料到 client 端
4. client 端接收到資料後回覆 server 端

整體流程大致如下圖:

![]({{site.baseurl}}/assets/img/ftp-active.png)

那主動式的問題主要會出現在第三步的時候，由於現行的網路架構下，client 端多數是被保護的，可能位在 router 之後或是有防火牆的設計防止外來的連線，也因此才有了被動式的方法誕生。

### 被動式

為了解決在現代網路架構下主動式的連線問題所誕生的被動模式，溝通過程如下:
1. 由 client 端(通常是 1024)發出連線請求與 FTP server 的 Command Port 建立連線，並發出命令
2. 同時，server 端開啟一個大於 1023 的 port 作為 Data Port 等待連線，並在第一步時，將這個 port 傳送給 client
3. client 對 server Data Port 主動發起連線進行資料傳輸

![]({{site.baseurl}}/assets/img/ftp-passive.png)

被動式來說的話變成 server 這邊需要額外多開幾個 port 來進行連線
## 實際問題解析

其實蠻好理解的，既然沒辦法由 server 來連線那就轉由 client 來進行，然而就是這樣的機制造成本地測試與線上使用時預期上的不同

在本地測試時 FTP 以及 web server 皆在同一個 host 之下，而 Java 的 Apache FTP API 預設是使用主動模式進行連線，而部屬到線上的 K8s 環境之後，FTP 以及 web server 被拆開成了兩個不同的微服務部屬在不同的 pod 之內，兩個服務之間的 port 是不能隨意連線的

其實以 K8s 的角度來說的話，也可以在 web server 這邊的微服務開啟 1024 以及 1025 port 應該也可以發起主動連線，不過這邊為了避免往後還需要特別去確認要使用哪個模式，這邊我選擇寫一個判斷來自動切換模式

### Java 解決方案

```java
public class FTPUtils{
    public static FTPClient getFTPClient(
        String host, int port,
        String username, String password
    ){
        FTPClient ftpClient = null;
        try {
            ftpClient = new FTPClient();
            ftpClient.connect(host, port);
            ftpClient.login(username, password);
            ftpClient.setRemoteVerificationEnabled(false);
            ftpClient.enterLocalActiveMode();
            ftpClient.listFiles("/");
            boolean result = FTPReply.isPositiveCompletion(ftpClient.getReplyCode());
            if (!result) {
                ftpClient.enterLocalPassiveMode();
                ftpClient.listFiles("/");
                result = FTPReply.isPositiveCompletion(ftpClient.getReplyCode());
                if (!result) {
                    throw new Exception(ftpClient.getReplyCode());
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return ftpClient;
    }
}
```

其實這個解決方案的靈感來自於著名的 FTP 軟體 [FileZilla](https://filezilla-project.org/)，FileZilla 在每次連線的時候第一件事就是取得根目錄的清單，於是我也效仿這個做法來確認連線模式是否有作用
