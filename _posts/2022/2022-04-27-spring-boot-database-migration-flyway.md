---
layout: post
title: "在 Spring boot 上利用 Flyway 實現 Database Migration"
date: 2022-04-26 16:16:27 +0800
category: backend
img: cover/spring-boot.jpg
description: "在撰寫後端程式的時候時常需要搭配資料庫的使用，而功能上一定程度也與資料庫是有掛鉤的，那當程式碼有更動的時候資料庫理所當然需要更新，這個更新的機制就被稱為 Migration，可能是從無到有的全部搬移，可能是版本更新導致的變更，在 Spring boot 中有一套工具很好的實現了這個功能，那就是 Flyway"
lang: zh-TW
tags: [backend,spring boot]
published: true
---

{{page.description}}

## 目的
如果有在開發實際上的產品就會了解到，資料庫的修改有時候是不可避免的，但若是直接去 Database 下 Query 的話會發生程式可能與資料庫不同步的情形，尤其若是這套系統需要部屬到多個平台上，那對資料庫的異動若是人工操作那絕對是一場災難，因此才會需要用到 Migration 的機制

那首先簡單介紹下使用 Migration 的目的:
1. 將資料庫的 Schema 變更搬移到不同的環境上
2. 對資料庫的修改進行版本控制
3. 利用 Git 這類版控工具將資料庫的修改與程式的版本同步

## JPA Hibernate 內建 Migration

前面介紹 Spring boot 時有介紹過 Spring 的強大 ORM JPA，而在 JPA 其實有內建簡單的 Migration，只要設定 `application.yaml` 的 `spring.jpa.hibernate.ddl-auto` 這個屬性就可以達到，屬性的值有:

+ `spring.jpa.hibernate.ddl-auto`
  1. `create`: 啟動時自動建立資料庫，如果已經存在會覆蓋，所以資料會遺失
  2. `update`: 啟動時自動更新資料庫，如果不存在會自動建立，有任何修改也會同步更新
  3. `create-drop`: 啟動時自動建立資料庫，如果已經存在會覆蓋，程式關閉則移除

一般來說比較常用的可能會是 `update`，但實際上這些使用起來都有各自的不方便之處，除了功能不完全外，並且無法清楚紀錄每次的變更，`update` 看起來有符合需求，但沒辦法處理刪除或是修改欄位名稱或形態的情況，會直接建立新的並且與舊的共存

而且如果要使用 JPA 的 Migration，那在建立 `@Entity` 的時候就要用相應的 Annotation 詳細設定 Schema 的屬性，這在撰寫程式上要多寫很多東西，而且個人認為可讀性很差，把資料庫的東西夾雜在程式中也不是很好的邏輯

所以在了解到有其他 Migration 的方法之後，建議是不要使用這種方式

## Flyway

那接下來就重點來介紹 Flyway 這套 Migration 的工具，首先要加上依賴:

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
    <version>8.5.9</version>
</dependency>
```

那要先了解到在 Flyway 的 Migration 被分為三種執行方式，分別是:
1. `Versioned Migrations`: 本身帶有版號，執行時會自動檢查資料庫的版本，如果版本不同則只會執行後面版本的 Migration，這是最常用的方式
2. `Undo Migrations`: 本身也帶有版號，顧名思義是用來 rollback 資料庫版本的，通常與 `Versioned Migrations` 相對應
3. `Repeatable Migrations`: 本身不帶有版號，會在每次執行 Migration 時都執行，算是簡單粗暴的方式

而不同行為的 Migration 是用檔名來區分的，各個格式如下圖所示:

![]({{site.baseurl}}/assets/img/flyway-migration.png)

### 自動執行 Flyway Migration

接著來看到 Flyway 的設定，在 `application.yml` 中:

```yaml
spring:
  flyway:
    enabled: true               # 啟用 Flyway Migration
    baseline-version: 1.0       # 基準版本
    baseline-on-migrate: true   # 基準版本是否在 migrate 時執行
    validate-on-migrate: true   # 執行 migrate 時會去驗證資料庫的版本與實際的 Migration 版本是否相同
    out-of-order: false         # 是否允許執行順序不同的 Migration
    target: latest              # 目標版本
    locations: classpath:db/migration # Migration 檔案存放的資料夾
    # 以下是資料庫的連線設定
    url: ${spring.datasource.url}
    user: potgres
    password: 123456
    # 由於是用 postgres 所以需要設定 schema，並且如果不存在會自動建立
    default-schema: ${spring.jpa.properties.hibernate.default_schema}
```

各屬性的功能應該不太需要詳述，只有 `out-of-order` 比較特別一點，如果設定為 true 則會發生一種情況，假設今天執行過了版本 `1` 以及 `3` 的 Migration，事後增加版本 `2`，則 Migration 就不會執行，如果設為 false 的話則會

設定完之後要來準備 Migration 需要用到的執行腳本，基本上是用 SQL 與法來撰寫，位置則放在上面設定的 `classpath:db/migration` 也就是 `resources/db/migration` 之下，簡單寫個範例:

```sql
-- V1.0__create_user_table.sql
CREATE TABLE "user"
(
    "id" bigserial NOT NULL PRIMARY KEY,
    "name" character varying(255) NOT NULL UNIQUE,
    "password" text NOT NULL,
    "email" character varying(255) NOT NULL UNIQUE,
    "creation_time" timestamp with time zone,
    "modification_time" timestamp with time zone
);

```

首先建立一個檔案 `V1.0__create_user_table.sql` 記得檔名的格式是固定的，其中 `V1.0` 是版本，`__` 後面的是 Migration 的描述

然後只要啟動 Spring boot 就會自動執行 Migration 了，非常方便

### 原理簡介

執行完之後到資料庫會看到預期的 Migration 以外，應該還會發現有一個 `flyway_schema_history` 的資料表，裡面就是 Migration 的執行紀錄，避免重複執行，並且有加入 CheckSum 用來驗證 Migration 的正確性，確保不會有人偷改 Migration 的腳本而沒有通知

而 Flyway 的執行原理也就是圍繞在這個資料表來運行

### 手動執行 Flyway

雖然通過設定可以讓 Flyway 自動在啟動時執行，但有時候就是想要手動操作一下，並且可能可以加入一些課製化的功能，所以在進行手動 Migration 之前要先關閉自動執行，設定一下 `spring.flyway.enabled=false`

然後建立一個 `AppListener` 以下是範例 code:

```java
public class AppListener implements ApplicationListener<ContextRefreshedEvent> {
    @Autowired
    private FlywayInfo flywayInfo;

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        flywayMigrate();
    }

    private void flywayMigrate() {
        final Flyway flyway = Flyway.configure()
                .locations(flywayInfo.getLocations())
                .baselineVersion(flywayInfo.getBaselineVersion())
                .baselineOnMigrate(flywayInfo.isBaselineOnMigrate())
                .validateOnMigrate(flywayInfo.isValidateOnMigrate())
                .outOfOrder(flywayInfo.isOutOfOrder())
                .target(MigrationVersion.fromVersion(flywayInfo.getTarget()))
                .dataSource(flywayInfo.getUrl(), flywayInfo.getUser(), flywayInfo.getPassword())
                .defaultSchema(flywayInfo.getDefaultSchema()).load();

        final MigrateResult migrateResult = flyway.migrate();
        final MigrationInfo info = flyway.info().current();
        log.info("\tFlyway Status: TargetVersion={}, state={}, file name=V{}__{}", migrateResult.targetSchemaVersion, info.getState(), info.getVersion(),
                info.getDescription());
    }
}
```

`onApplicationEvent` 是在 Spring boot 的 Application 啟動之後執行的一個 Hook，因此在這裡加入 Flyway Migration 的執行，並且印下客製化的訊息

不過實際上寫手動 Migration 原先的用意是想要在退版的時候可以自動進行 Undo，程式碼如下:

```java
private void flywayMigrate() {
    final Flyway flyway = Flyway.configure()
            .locations(flywayInfo.getLocations())
            .baselineVersion(flywayInfo.getBaselineVersion())
            .baselineOnMigrate(flywayInfo.isBaselineOnMigrate())
            .validateOnMigrate(flywayInfo.isValidateOnMigrate())
            .outOfOrder(flywayInfo.isOutOfOrder())
            .target(MigrationVersion.fromVersion(flywayInfo.getTarget()))
            .dataSource(flywayInfo.getUrl(), flywayInfo.getUser(), flywayInfo.getPassword())
            .defaultSchema(flywayInfo.getDefaultSchema()).load();
    if (flyway.info().current().getVersion().isNewerThan(target)) {
        UndoResult undoResult = flyway.undo();
        final MigrationInfo info = flyway.info().current();
        log.info("\tFlyway Status: TargetVersion={}, state={}, file name=V{}__{}", undoResult.targetSchemaVersion, info.getState(), info.getVersion(),
                info.getDescription());
    } else {
        final MigrateResult migrateResult = flyway.migrate();
        final MigrationInfo info = flyway.info().current();
        log.info("\tFlyway Status: TargetVersion={}, state={}, file name=V{}__{}", migrateResult.targetSchemaVersion, info.getState(), info.getVersion(),
               info.getDescription());
    }
}
```

只不過 Flyway 的 Undo 似乎要付費版才能夠使用😅，非常尷尬的這段程式只好做廢了

---

## 結語

突然來寫 Spring boot Migration 的內容其實是因為要寫 Golang 的 Migration 時才發現當初忘記寫了XD，因此下篇就是來介紹一下 Golang 的 Migration 要怎麼做啦

至於 Flyway 的 Undo 跟 Repeatable 不多做介紹，Undo 要付費沒辦法測試使用，Repeatable 則是想不太到什麼情境可以套用