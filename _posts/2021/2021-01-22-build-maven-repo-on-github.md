---
layout: post
title: "在 Github 上建立自己的 Maven Repo"
date: 2021-01-22 15:17:44 +0800
category: deploy
img: cover/maven.jpg
description: 最近用 spring boot 寫的後端專案越來越肥大了，因此團隊考慮將一些功能切分出去變成小專案來維護，也方便之後給其他專案共用，以前自己也曾經作過這種事，不過當時是單純用共享 jar 的方式，每改一版就要重 build 一次並且有用到的專案都要手動去下載，其實是有點麻煩，這次將運用 maven 的機制來部屬自己的 maven repo，讓專案只須要維護 maven 的設定就可以達到共用 library 的效果。
lang: zh-TW
tags: [maven, java]
---

{{page.description}}

## Maven

首先來了解一下 Maven，這是一套由 Apache 開發來針對 Java 開發的套件管理及自動建置工具，就好比 node.js 跟 npm 的關係

maven 透過在專案底下的`pom.xml (Project Object Model)`來管理專案的套件以及建置工作，而預設 maven 會從 [mvnRepository](https://mvnrepository.com/) 這個公共的 repo 空間去抓取 dependency，甚至可以自己手動從上面下載 jar，重點是可以透過設定來指向私人的 repo 來源，也是今天我們的重點

## Github Package

Github package 是一個 package hosting 的服務，可以在 github 上建立自己的 package repo，不過要注意免費版不支援個人的 private package，下面是 github package 的 overview，可以看到支援的 package 的類型

![]({{site.baseurl}}/assets/img/github-packages-overview-diagram.png)

而一般公開的 package 是可以免費使用的，但若有私人使用的需求在 github 就是要收費了，收費方式看下面連結 [github package 收費機制](https://docs.github.com/en/github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-packages)

下面我們來實際使用看看 github package

### setting.xml

首先在 `~/.m2/settings.xml` 下加入以下內容，設定 maven repo 的來源，並加入 authorization，TOKEN 的部分需要取的 github 的 personal access token，步驟可以參考[這裡](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token)

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <activeProfiles>
    <activeProfile>github</activeProfile>
  </activeProfiles>

  <profiles>
    <profile>
      <id>github</id>
      <repositories>
        <repository>
          <id>central</id>
          <url>https://repo1.maven.org/maven2</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
          <id>github</id>
          <name>GitHub OWNER Apache Maven Packages</name>
          <url>https://maven.pkg.github.com/OWNER/REPOSITORY</url>
        </repository>
      </repositories>
    </profile>
  </profiles>

  <servers>
    <server>
      <id>github</id>
      <username>USERNAME</username>
      <password>TOKEN</password>
    </server>
  </servers>
</settings>
```

這個 setting 可以加入 github 為 dependency 的來源，要注意 `<server>` 的 ID 跟 `<repository>` 的 ID 要對好

### pom.xml

接下來在想包裝的子專案下，更改一下 pom 的內容，加入下面內容，一樣記得 ID 必須跟 setting 裡 server 的 ID 對上

```xml
<project>
    ...
    <distributionManagement>
        <repository>
            <id>github</id>
            <name>GitHub OWNER Apache Maven Packages</name>
            <url>https://maven.pkg.github.com/OWNER/REPOSITORY</url>
        </repository>
    </distributionManagement>
    ...
</project>
```

接下來在專案目錄下鍵入指令 `mvn clean deploy`，看到跑完結果之後就可以到 github repo 上檢查看看

![]({{site.baseurl}}/assets/img/mvn-package-on-github.png)

可以發現 package 已經出現囉，接下來只要在想要引用的專案裡加入 dependency 就可以使用了


[主要參考資料](https://docs.github.com/en/packages/guides/configuring-apache-maven-for-use-with-github-packages#authenticating-to-github-packages)