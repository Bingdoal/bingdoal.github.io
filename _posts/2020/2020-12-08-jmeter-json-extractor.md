---
title: Jmeter JSON Extractor 快速解析 API 的 Response
date: 2020-12-08 09:30:00 +0800
category: others
img: cover/jmeter.png
description: 在 RESTful API 的測試情境之下，常使用 JSON 為溝通的格式，在測試的時候需要驗證資料完整性，或是將結果擷取出來帶入其他 API 之中都是常會遇到的情境，今天就來整理一下 Jmeter JSON Extractor 的使用吧
layout: post
tags: [test, jmeter]
redirect_from:
- /test/2020/12/jmeter-json-extractor/
---

{{page.description}}

# JSON Extractor
新版 Jmeter 中內建在 `Post Processor` 中可以馬上使用，使用上輸入的 JSON 為 `Sample` 結束後的結果資料，依據 JSON Path expressions 來解析內容

## 屬性

![]({{site.baseurl}}/assets/img/jmeter-json-extractor-ui.png)

+ Names of created variables: 從 JSON Extractor 提取出來的屬性存放的變數名稱，可以是已經存在的或是不存在的都可以
+ JSON Path expression: 特有的表達式，用來擷取需要的資料
+ Match No.: 若表達式的條件節取出來是多結果，可以選擇要第幾個結果。-1 為全部、0 為隨機，預設是全部
+ Compute concatenation var (suffix_ALL): 若最後選取到多結果，預設的儲存方式為，suffix_1、suffix_2、suffix_3 ...，若勾選這個選項，則會多一個 suffix_ALL 裡頭會放全部的結果，suffix 為第一個輸入的變數名稱
+ Default Value: 表示設定變數的預設值

## JSON Path expression

### 運算元
+ `$`: 整個 Json 的根節點，基本上是所有 Path expression 的開頭
+ `*`: match 所有內容
+ `..`: 掃描底下所有元素
+ `.<name>`: 取得 name 底下的元素
+ `['<name>' (, '<name>')]`: 一次取得兩個 name 的元素
+ `[<number> (, <number>)]`: 用在 array 取得指定的 index 元素
+ `[start:end]`: 用在 array 取得指訂區間的元素
+ `[?(<filter expression>)]`: 篩選需要的元素
+ `@`: 與 `?` 一起使用，指當前節點的位置

### Filter
+ 常見的等於、大於、不等於那類的就不列了
+ `=~`: 接 regular expression
+ `in`: 後面接一個 array
+ `nin`: 後面接一個 array: 表示 not in
+ `subsetof`: 後面接一個 array: 表示子集
+ `anyof`: Github 有寫但我始終不了解用法，原文 => `left has an intersection with right [?(@.sizes anyof ['M', 'L'])]`
+ `noneof`: not anyof
+ `size`: 後面要接元素: 表示左右的元素長度相同(array 或 string)
+ `empty`: 表示為空元素(array 或 string)

### 函式
+ `min()`: 找最小值
+ `max()`: 找最大值
+ `avg()`: 取平均值
+ `stddev()`: 算標準差
+ `length()`: 長度
+ `sum()`: 總和

## 示例

+ `$.content..id`: 取得 content 下所有的 id，要注意一點 `..` 是深層掃描，如果底下層還有 id 的屬性也會一並被截取到
+ `$.content[*].id`: 這個寫法就可以只抓到下一層的所有 id
+ `$.content[?(@.id < 10)]`: 取 id 小於 10 的所有元素

更多用法就參考 Github 吧 [Jayway JsonPath Github](https://github.com/json-path/JsonPath)

也可以到 [Jayway JsonPath Evaluator](http://jsonpath.herokuapp.com/) 去玩玩看 Path expression 是否如預期

## Debug Sampler
在 Jmeter 中使用的話可以用 `Debug Sampler` 查看最後變數取得的值，來確認有沒有正常運行
