---
title: Spring 上的 @Scheduled 以及 cron 表示式
date: 2020-11-24 09:30:00 +0800
category: backend
img: cover/spring-boot.png
description: 後端開發中也常會碰到定期任務的需求，這時候 spring 的 Scheduled 就可以派上用場啦，也筆記一下 cron 表示式的內容，不然每次寫都搞不太清楚
layout: post
tags: [spring boot, java, cron]
---

{{page.description}}

## @Scheduled

當需要用到週期性的執行某些任務的時候，就很適合使用 `@Scheduled` 可以看到下面的使用範例

```java
@SpringBootApplication()
@EnableScheduling
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}


@Component
class myTask{
    @Scheduled(fixedDelay = 10 * 1000, initialDelay= 10 * 1000)
    public void everyTenSecondTask(){
        System.out.println("Ten second task.");
    }

    @Scheduled(cron = "0 0 3 * * ?", zone="Asia/Taipei")
    public void everyDayTask(){
        System.out.println("Every day task.");
    }
}
```

首先一定要記得放上 `@Component`，Spring 才會把這個 class 註冊為 Bean，並且在 ` SpringApplication.run` 的類別加上 `@EnableScheduling` 才會啟用

然後可以看到 `@Scheduled` 裡面有兩種寫法 `@Scheduled(fixedDelay = 10 * 1000, initialDelay= 10 * 1000)` 是單純用來執行週期性的任務，以這邊的設置為例的話就是每十秒執行一次，並且會在程式執行十秒後才開始

而 `@Scheduled(cron = "0 0 3 * * ?", zone="Asia/Taipei")` 的寫法呢則會根據設定的 cron expression 來決定執行的時機，使用上的彈性較大，但就不是這麼好理解，以這邊的設置來說的話就是在每天凌晨三點的時候執行，並且是按照台北時區來判斷

## cron expression

那其實重點是在於 cron 的規則撰寫上，要把 cron 運用自如才能更好的發揮 `@Scheduled` 的功能

簡單幾個例子如下:
+ `0 0 12 * * ?`: 每天中午 12 點
+ `0 0 12 1 * ?`: 每個月 1 號中午 12 點
+ `0/5 0 12 * 1 ?`: 1 月每天中午 12 點，每 5 秒

一開始在找資料的時候也是就看到幾個例子，可能套用一下改個數字也是可以用，但總覺得不得其要領，下面詳細介紹他的內容參數

對照上面的表示式來看，依序是
+ `秒 分 時 日 月 周 (年)`: 年是可選參數，不一定要寫，其他都是必須的

然後是一些特別的參數:

| 字元 | 意義|
|:----:|:--------|
| `*`  | 表達任意值|
| `?`  | 只用在 `日` 跟 `周` 的值域，有點表達 don't care 的概念|
| `-`  | 指定範圍，前後接數字: 10-12|
| `,`  | 指定離散的選項: 1,5,6,8|
| `/`  | 指定增量，表達 `每` 的概念: 0/5 意旨從 0 開始每 5 單位|
| `L`  | 用在 `月` 跟 `周` 的值域。在月的話表達最後一天，在周的話前面可以加上數字 3L 表示該月最後一個星期二|
| `W`  | 用在`日`的值域表示距離最近的該月工作日: 15W，距離 15 號最近的工作日，可能往前也可能往後|
| `LW` | 用在`日`的值域，表示最後一周的工作日|
| `#`  | 用在`周`的值域，指定特定周的特定日: "4#2" 表示第二周的星期三|
| `C`  | 用在`日`跟`周`的值域，指某特定個日期的後一天: 在`日`中寫 3C 指該月 3 號的後一天，在`周`中寫 2C 指該周星期一的後一天 |

這邊要特別說一下 `?`，比較特別一點的是一個 cron 表達式之中，一定要存在一個 `?`，也就是 `日` 跟 `周` 不能同時被設定，目的是避免混亂，因為兩者都是指定到日的設定，放入 `?` 的意思也就是說這個參數沒有特別意義的意思