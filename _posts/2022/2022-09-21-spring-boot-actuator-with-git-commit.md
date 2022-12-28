---
layout: post
title: 利用 Actuator 監控服務狀態並整合 git 版控確認版本
date: 2022-09-21 10:22:05 +0800
category: backend
img: cover/spring-boot.png
description: 當服務部署到機器上運行時，我們需要確認一下服務的狀態，簡單的做法就是開個 API 給外部打來確認，Spring boot 則已經有包好的套件可以用了，簡單來介紹下 Actuator 的使用，並在最後介紹一個相關的好用工具
lang: zh-TW
tags: [spring boot]
published: true
---

{{page.description}}

## Spring boot Actuator

最基本的讓我們先加入依賴

```yaml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

然後就好了，對就這麼簡單

加上依賴之後就會自動幫忙加上幾個 API 用來確認服務狀態

### Endpoint

以下是 Actuator 有提供的 API，預設只有提供 `health` 以及 `info` 這兩個，其他可以透過設定開啟

| Method | Path                       | 描述                                                                                             |
| :----: | -------------------------- | ------------------------------------------------------------------------------------------------ |
|  GET   | `/actuator`                | 取得 Actuator 有開啟的 endpoint                                                                  |
|  GET   | `/actuator/health`         | 確認服務活著，最基本會回傳一個 `{"status":"UP"}`，可以一定程度做些客製訊息                       |
|  GET   | `/actuator/info`           | 用來查看 properties 在 info 下的內容，可以寫一些基本的服務資訊，一些套件會把資訊放在下面方便查看 |
|  GET   | `/actuator/mappings`       | 取得所有可用的 API 以及相應的 controller，感覺很好用但個人在使用上有遇到一些問題，目前還沒解決   |
|  GET   | `/actuator/env`           | 取得所有 properties 讀進來的值，有趣的是會幫忙把 PASSWORD 之類的敏感字屏蔽掉                     |
|  GET   | `/actuator/configprops`    | 取得所有被 `@ConfigurationProperties` 修飾的 class 及其內容，以便得知有沒有被正確 binding 到     |
|  GET   | `/actuator/conditions`     | 查看 `@ConditionalOnProperty` 修飾的 class 哪些有達成條件被建立                                  |
|  GET   | `/actuator/beans`          | 可以看到所有 Spring Boot 啟動後建立的所有 Bean 以及其依賴關係                                    |
|  GET   | `/actuator/caches`         | 用來查看運作中的 cacheManager                                                                    |
|  GET   | `/actuator/scheduledtasks` | 查看所有被定義的 `@Schedule` 任務相關資訊                                                        |
|  GET   | `/actuator/loggers`        | 看看 logger 的各個 package level 在哪裡                                                          |
|  GET   | `/actuator/heapdump`       | 會抓一個大檔案下來，紀錄 java 的 heap 細節                                                       |
|  GET   | `/actuator/threaddump`     | 查看服務中使用到的 thread 以及各種詳細內容，意外有趣的資訊                                       |
|  GET   | `/actuator/metrics`        | 可以監控服務運作的各項指標，像是 cpu memory 之類的，有用過 Prometheus 的話應該很熟悉             |
|  POST  | `/actuator/shutdown`       | 唯一用 POST 的方法，用來關閉服務，不過沒看人開過感覺很危險                                       |
|  GET   | `/actuator/auditevent`     | 查看 audit 的事件，搭配 Spring security 使用，個人沒有使用過                                     |

基本 Actuator 提供的介面就如以上，有些套件會去額外擴充一些內容就以後碰到再說了

### application.yml

接下來就是各種設定用來開啟或是關閉 endpoint 或是針對細節的調整

```yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,configprops,env # 表示開啟這幾個 endpoint

...

management:
  endpoints:
    web:
      exposure:
        include: "*" # 也可以用 wildcard 直接全開
        exclude: beans # 然後透過 exclude 去除不想要的

...

management:
  endpoint:
    shutdown:
      enabled: true # 以上設定預設都是不開啟 shutdown 必須額外設定，多一層保護

...

management:
  endpoints:
    web:
      base-path: /management # 也可以修改基礎的 path

```

還有其他可以更詳細的設定，個人有用到的就是跟 `health` 相關的，可以列出更詳細的啟動細節

```yml
management:
  endpoint:
    health:
      show-details: ALWAYS
```

基本上每個 endpoint 都有一些設定可以玩，之後有碰到再說

### Git info

接下來介紹一個好用的工具，應該每個開發者都會遇到部署的版本跟預期的不符合的問題

多數時候我們都是手動押版號，看到版號就可以確定版本，但有時候在快速修正迭代版本的時候一直改版號也很麻煩

甚至也會有忘記改或是改錯的情況發生，不過我們其實日常就在使用一個絕對可靠的版本控制 Git

因此我們希望可以將 Git 的版本資訊帶到程式中查看，那就能夠確保運行的版本了

那首先除了 Actuator 讓我們加入以下的額外 plugin

```xml
 <plugins>
    <plugin>
        <groupId>pl.project13.maven</groupId>
        <artifactId>git-commit-id-plugin</artifactId>
        <configuration>
          <useNativeGit>true</useNativeGit>
          <dateFormat>yyyy-MM-dd HH:mm:ss Z</dateFormat>
        </configuration>
    </plugin>
</plugins>
```

然後就好了，太簡單了吧

接下來去訪問 `/actuator/info` 就能看到詳細的 Git 資訊，包含 branch, commit id 以及 commit 的時間

上面有做一些設定像是 `dateFormat` 就是修改輸出的時間格式

`useNativeGit` 則是讓他透過本地的 git 設定進行操作，才可以通過 private repo 的驗證問題

---

## 結語

這種通用型的套件把一切都包好好，真的是使用 Spring boot 最大的優點，什麼都不用做，設定就完事了
