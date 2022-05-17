---
layout: post
title: "Golang 資料庫操作: gorm"
date: 2022-04-26 09:02:18 +0800
category: backend
img: cover/golang.png
description: "後端程式不免會牽涉到資料庫的 CRUD，這次我們將試著介紹一個常見的 Golang ORM 函式庫 gorm，介紹一些基本操作還有設定，以及一些自己的使用習慣與心得"
lang: zh-TW
tags: [golang]
published: true
---

{{page.description}}

## GORM

簡單易懂的名字，官方連結在這裡：[GORM](https://gorm.io/)，本身也是一個開源專案，官方文件也寫得非常好，甚至是有全中文的翻譯，推薦使用。

因此這邊只進行一些簡單介紹，更多使用請看官方文件應該是綽綽有餘

### 安裝

```bash
go get -u gorm.io/gorm
go get -u gorm.io/driver/postgres
```

driver 請按照個人使用的 DB 去安裝，我這裡示範的是 Postgres

### DB 連線

首先我們要進行 DB 的連線初始化，以下是範例 code，`config.Env` 的用法可以參考[上一篇](https://bingdoal.github.io/backend/2022/04/golang-setting-enviroment-variable-viper/)

```golang
var dsn = fmt.Sprintf(
    "host=%s user=%s password=%s dbname=%s port=%d sslmode=disable",
    config.Env.Get("postgres.host"),
    config.Env.Get("postgres.user"),
    config.Env.GetString("postgres.password"),
    config.Env.Get("postgres.database"),
    config.Env.GetInt("postgres.port"),
)

var pgcon = postgres.New(postgres.Config{
    DSN:                  dsn,
    PreferSimpleProtocol: true,
})

func connectDB() *gorm.DB {
    db, err := gorm.Open(pgcon, &gorm.Config{
        NamingStrategy: schema.NamingStrategy{
            // table name 為單數形式
            SingularTable: true,
        },
    })
    if err != nil {
        fmt.Println("Connect DB failed: ", err)
        panic(err)
    }
    // 如果沒有 schema 就自動建立，並設為 search_path
    db.Exec(fmt.Sprintf("CREATE SCHEMA IF NOT EXISTS %s", config.Env.Get("postgres.schema")))
    db.Exec(fmt.Sprintf("SET search_path='%s'", config.Env.Get("postgres.schema")))

    return db.Session(&gorm.Session{PrepareStmt: true})
}

var DB *gorm.DB

func init() {
    DB = connectDB()
}
```

由於 Postgres 有 schema 的特殊設計，所以在連線的時候有事先做一點處理，預先建立 schema，並設為 search_path，初始化之後提供 public 的 `DB` 供外部使用

### Model

接下來 ORM 最重要的就是 table schema 跟 model 的鍊結了，以下是範例 code:

```golang
type User struct {
    ID       uint64 `json:"id" gorm:"primarykey"`
    Name     string `json:"name"`
    Email    string `json:"email"`
    Password string `json:"password"`
}

func main(){
    users := []User{}
    // Select * From user
    if err:=DB.Find(&users).Error; err != nil {
        fmt.Println("Get users failed: ", err)
        panic(err)
    }
    fmt.Println("Get users: ", users)
}
```

由於我們在連線時設定了 `SingularTable: true` 所以這邊預設對應的 table 名稱就會是 user，gorm 會根據提供的 model 自動轉換語法，這裡只是簡單示例它的便利性，更多 Query 的用法參考[官方文件](https://gorm.io/docs/query.html)應該更詳盡

基本上可以自由組合出自己想要的任何語法

#### Model 設計

官方文件有的東西就不多做說明了，這邊主要想要分享下對於 gorm 模式的使用心得

由於操作基本上都是透過 `gorm.DB` 也就是建立連線的 Session 來做，為了切割不同 Model 的方法，一般來說可能會進行 Service 的封裝，不過這邊應該還在 Dao 的範疇內

golang 的設計上錯誤處理是由回傳的 error 來套用，因此盡可能的減少回傳參數才是比較好的設計，可以看到 `gorm.DB` 的操作也是將結果直接放在 Model 上

以上原因產生了下面的範例程式:

```golang
type User struct {
    ID       uint64 `json:"id" gorm:"primarykey"`
    Name     string `json:"name"`
    Email    string `json:"email"`
    Password string `json:"password"`
}

func (user *User) model() *gorm.DB { return postgres.DB.Model(user) }

func (user *User) Create() error {
    return user.model().Create(user).Error
}

func (user *User) Update() error {
    return user.model().Updates(user).Error
}

func (user *User) FindOne() error {
    return user.model().Where("id = ?", user.ID).First(user).Error
}

func (user *User) Delete() error {
    return user.model().Delete(user).Error
}

type Users []User

func (users *Users) model() *gorm.DB { return postgres.DB.Model(users) }

func (users *Users) FindAll() error {
    return users.model().Find(&users).Error
}

```

這邊的設計會偏向把 Dao 也跟 Model 做在一起，這樣可以減少回傳的參數，並且把 Dao 與 Model 放在一起也是將相同的操作邏輯放在一起，並且隔離了不同 Model 的操作，還額外設計了複數型的 Model 可以用來加上批次操作

不過這樣寫的壞處是不容易抽換實作，這在測試的時候會有些困難，這方面還要思考怎麼解決

### Hooks

另一個比較會用到的應該就是 Model Hook，gorm 有留下一些 Hook 的方法讓我們可以簡單的對 Model 操作進行監控

```golang
type BaseModel struct {
    ID               uint64    `json:"id" gorm:"primarykey"`
    CreationTime     time.Time `json:"creationTime"`
    ModificationTime time.Time `json:"modificationTime"`
}

func (model *BaseModel) BeforeCreate(db *gorm.DB) (err error) {
    model.CreationTime = time.Now()
    model.ModificationTime = time.Now()
    return
}

func (model *BaseModel) BeforeUpdate(db *gorm.DB) (err error) {
    model.ModificationTime = time.Now()
    return
}

type User struct {
    base.BaseModel
    Name     string `json:"name"`
    Email    string `json:"email"`
    Password string `json:"password"`
}
```

這邊利用到 golang 的繼承以及 gorm 的 Hook，幫剛剛的 Model 套上一層基本的 Model，藉由這樣的設計在每次更新跟建立的時候都會去自動更新時間


---

## 結語

gorm 真的是個強大好用的 ORM 工具，操作簡單好上手，官方文件寫的也非常詳盡易懂，雖然這邊只有講到基本的運用，不過更多的都可以在官方文件找到

而像是關聯的設定或是多表查詢，這些在 Postgres 通常會用 View 來解決，基本上不太喜歡透過 ORM 來做過於複雜的操作

除以上的介紹之外，其實 gorm 還有提供 Migration 的功能，但是個人不是很偏好這種 ORM 提供的 Migration，都只會支援更新以及建立，如果有太複雜的修改或是移除都沒辦法辦到，也沒有版本的管理，所以之後會再來介紹其他的 Migration 工具