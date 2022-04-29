---
layout: post
title: "在 Golang 利用 golang-migrate 實現 database migration"
date: 2022-04-27 11:47:50 +0800
category: backend
img: cover/golang.png
description: "前篇介紹了 Spring Boot 的 Migration 用法，也簡介了 Migration 的用途，那這篇就介紹了在 Golang 中使用 golang-migrate 實現 database migration"
lang: zh-TW
tags: [golang]
published: true
---

{{page.description}}

## 目的
怕有讀者是只想看 golang 的，就再把 Migration 的用途貼一遍

如果有在開發實際上的產品就會了解到，資料庫的修改有時候是不可避免的，但若是直接去 Database 下 Query 的話會發生程式可能與資料庫不同步的情形，尤其若是這套系統需要部屬到多個平台上，那對資料庫的異動若是人工操作那絕對是一場災難，因此才會需要用到 Migration 的機制

那首先簡單介紹下使用 Migration 的目的:
1. 將資料庫的 Schema 變更搬移到不同的環境上
2. 對資料庫的修改進行版本控制
3. 利用 Git 這類版控工具將資料庫的修改與程式的版本同步

## golang-migrate

[官方 Github](https://github.com/golang-migrate/migrate)

### 安裝

```bash
go get github.com/golang-migrate/migrate/v4
```

官方有提供 CLI 與在程式內執行的方式，但筆者比較偏好將 Migration 與程式的版控結合在一起，所以這邊就只介紹在程式中運行的部分

### 範例程式

首先在路徑 `_assets/db/migration` 下面準備好 Migration 的執行腳本，要特別注意檔名的部分一定要按照格式 `版號_敘述.up/down.sql`，up 表示在進版的時候執行，down 則反

```sql
-- 1_create_user_table.up.sql
BEGIN;

CREATE TABLE "user"
(
    "id" bigserial NOT NULL PRIMARY KEY,
    "name" character varying(255) NOT NULL UNIQUE,
    "password" text NOT NULL,
    "email" character varying(255) NOT NULL UNIQUE,
    "creation_time" timestamp with time zone,
    "modification_time" timestamp with time zone
);

COMMIT;

-- 1_drop_user_table.down.sql
BEGIN;

DROP TABLE IF EXISTS "user";

COMMIT;
```

然後接下來是 golang 的部分:

```golang
package main

import (
    "github.com/golang-migrate/migrate/v4"

    _ "github.com/golang-migrate/migrate/v4/database/postgres"
    _ "github.com/golang-migrate/migrate/v4/source/file"
)

func main() {
    up := true

    m, err := migrate.New(
        "file://_assets/db/migration",
        "postgres://postgres:postgres@localhost:5432/mydb?sslmode=disable",
    )

    if err != nil {
       panic(err)
    }
    if up {
        err = m.Up()
    } else {
        err = m.Down()
    }

    if err != nil {
       panic(err)
    }
}
```

要特別注意到必須加上 `import _ "github.com/golang-migrate/migrate/v4/database/postgres"` 以及 `import _ "github.com/golang-migrate/migrate/v4/source/file"`，這樣在執行的時候才找的到相應的驅動

其實用起來非常簡單，只要把 Migration 的腳本放在 `_assets/db/migration` 下面，然後 `migrate.New` 指定好相應的路徑以及資料庫的連線，就可以開始使用了

`m.Up()` 會進版到最新，`m.Down()` 則會退到最低，執行進版後可以到資料庫觀察，應該會發現多了一個 table 是 `schema_migrations` 就是用來記錄現在 migration 的版本，不過這不像 spring boot 的 flyway 有加入驗證功能，也就是說如果在執行之後有人修改 Migration 的腳本的話是沒有辦法知道的

### 實際設計使用

不過實際上使用起來還是會希望可以控制 Migration 的版本，並且跟整個 Server 起來的時候同步運行，因此我們可以把 Migration 加入到我們前幾天寫的 gorm 連線之中，還沒看過的可以點[這邊](https://bingdoal.github.io/backend/2022/04/golang-database-operation-gorm/)

```golang
// 詳細看前幾天的程式
...

var DB *gorm.DB

func init() {
	DB = connectDB()
	if config.Env.GetBool("migration.enabled") {
		migration()
	}
}

func migration() {
	migration := newMigration()
	target := config.Env.GetString("migration.target")
	if target == "latest" {
		migration.Up()
	} else {
		migration.To(config.Env.GetUint("migration.target"))
	}
}
```

利用環境變數設定來決定 Migration 的目標版本，並且在確認資料庫連線暢通後進行 Migration

然後我們來看看 `newMigration` 是怎麼運作的:

```golang
type Migration struct {
	client *migrate.Migrate
}

var dbUrl = fmt.Sprintf("postgres://%s:%s@%s:%d/%s?sslmode=disable&search_path=%s",
	config.Env.GetString("postgres.user"),
	config.Env.GetString("postgres.password"),
	config.Env.GetString("postgres.host"),
	config.Env.GetInt("postgres.port"),
	config.Env.GetString("postgres.database"),
	config.Env.GetString("postgres.schema"),
)

func newMigration() *Migration {
	m := Migration{}
	path := "file://_assets/db/migration"
	var err error
	m.client, err = migrate.New(path, dbUrl)
	if err != nil {
		panic(err)
	}
	return &m
}

func (m *Migration) To(targetVersion uint) {
	if err := m.client.Migrate(targetVersion); err != nil && err != migrate.ErrNoChange {
		panic(err)
	}
	afterVersion, _, _ := m.client.Version()
	fmt.Printf("Migration to version:%d success", afterVersion)
}

func (m *Migration) Up() {
	if err := m.client.Up(); err != nil && err != migrate.ErrNoChange {
		panic(err)
	}
	afterVersion, _, _ := m.client.Version()
	fmt.Printf("Migration up version:%d success", afterVersion)
}

func (m *Migration) Down() {
	if err := m.client.Down(); err != nil && err != migrate.ErrNoChange {
		panic(err)
	}
	version, _, _ := m.client.Version()
	fmt.Printf("Migration down version:%d success", version)
}
```

簡單用一個結構來進行包裝，分裝成 `To`、`Up`、`Down` 三個方法來使用，不過 `Down` 目前沒想到哪裡可以用上

---

## 結語

有一個這麼方便的 Migration 真是太好了，基本想要的功能都具備了，就差驗證版本了，這點就差 Flyway 一點了，有點可惜，不過已經算是有充分的功能可以套用在產品上了，這點就是我們的目標了