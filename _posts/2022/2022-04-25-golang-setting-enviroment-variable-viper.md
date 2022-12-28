---
layout: post
title: "Golang 環境變數與設定檔: viper"
date: 2022-04-25 15:41:50 +0800
category: backend
img: cover/golang-viper.png
description: "程式部屬時常因執行環境的不同，而需要套用不同的設定，大多數情況我們是透過環境變數來設定，像是 Spring boot 內建讀取 application.yml，或是 Node.js 的 dotenv，這次介紹一個 Golang 的工具 viper，就是類似於前兩者的工具，提供各種環境變數與設定的操作，請看下面的範例"
lang: zh-TW
tags: [golang]
published: true
---

{{page.description}}

## Viper

[官方連結](https://github.com/spf13/viper)

+ 支援 json, toml, yaml, hcl, envfile, properties 格式
+ 支援讀取環境變數
+ 支援遠端讀取設定
+ 支援熱讀取，可以即時更新設定

### 安裝

```bash
go get github.com/spf13/viper
```

## 使用

### 範例設定檔

```yaml
# _assets/env.yaml
name: go-demo
mode: dev
logger: DEBUG
version: "1.0.0"
server:
  port: 8888
```

```yaml
# _assets/env.prod.yaml
mode: prod
```

這裡準備兩份檔案，模擬實際使用情況，可能需要不同的設定

## 範例 code

```golang
package config

var Env *viper.Viper

func init() {
    Env = initViper()
}

func initViper() *viper.Viper {
    v := viper.New()

    // 設定檔的檔名、格式、路徑
    v.SetConfigName("env")
    v.SetConfigType("yaml")
    v.AddConfigPath("_assets/")

    // 執行讀取設定
    err := v.ReadInConfig()
    if err != nil {
        fmt.Println("[Error] Loading config failed: ", err)
        panic(err)
    }

    // 讀取環境變數時以 _ 為分隔符
    v.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))
    // 設定讀取環境變數時的前綴字
    v.SetEnvPrefix("demo")
    // 讀取環境變數
    v.AutomaticEnv()

    //  Print 實際讀取到的設定
    fmt.Printf("%+v\n",v.AllSettings())
    return v
}
```

短短幾行 code 包含了讀取檔案以及自動套入環境變數，並且輸出設定的內容，這樣就可以看到設定的內容

如果需要用環境變數套用，則設定的前綴字以及分隔符號，以這邊為例: `DEMO_MODE`

使用時，可以直接使用 `viper.Get` 的系列方法，例如:

```golang
    fmt.Printf("\n============ Start [%s] version:%s on:%s ============\n",
        config.Env.GetString("name"),
        config.Env.GetString("version"),
        config.Env.GetString("server.port"))
```

多層結構使用上用 `.` 隔開，提供各種 `Get` 的方法，不用額外進行轉換

而如果需要讀取多份設定檔，則可以使用 `viper.MergeInConfig` 方法，我們改寫上面程式

```golang
const envPath = "_assets/"
const envType = "yaml"
const envPrefix = "demo"

func initViper() *viper.Viper {
    v := viper.New()
    v.SetConfigName("env")
    v.SetConfigType(envType)
    v.AddConfigPath(envPath)
    err := v.ReadInConfig()
    if err != nil {
        fmt.Println("[Error] Loading config failed: ", err)
        panic(err)
    }

    // 執行時讀取參數套用不同的設定檔
    if len(os.Args) >= 2 {
        v.SetConfigName("env."+os.Args[1])
         v.SetConfigType(envType)
        v.AddConfigPath(envPath)
        // 合併設定檔
        err := v.MergeInConfig()
        if err != nil {
            fmt.Println("[Error] Merge config failed: ", err)
            panic(err)
        }
    }

    v.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))
    v.SetEnvPrefix(envPrefix)
    v.AutomaticEnv()

    fmt.Printf("%+v\n",v.AllSettings())
    return v
}
```

這邊利用執行程式時的參數，來讀取不同的設定檔，並且合併設定檔，這樣就可以看到不同的設定檔的內容，並且會以參數的設定檔為主去取代預設設定檔的內容

執行時如下:

```bash
go run main.go prod
```
