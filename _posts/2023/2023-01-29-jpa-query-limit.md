---
layout: post
title: "[踩雷紀錄] JPA 限制查詢筆數"
date: 2023-01-28 15:12:20 +0800
category: backend
img: cover/spring-boot.png
description: "JPA 的功能強大，能利用介面名稱以及回傳類型建立相應的查詢語句，如果不夠用還可以透過 JPQL 自行組裝，不過最近遇到需要取用查詢結果排序後第一筆的時候遇到一些困難，紀錄一下解法"
lang: zh-TW
tags: [spring boot]
published: true
---

{{page.description}}

## 問題描述

因需求的 query 蠻複雜的所以是採用 JPQL 撰寫，但是撰寫的時候發現沒辦法用 `LIMIT` 如下面舉例

```java
@Query("SELECT u FROM User u WHERE u.status = 1 ORDER BY createdAt DESC LIMIT 1")
public User findFirst();
```

上面的寫法是沒辦法執行的，JPQL 是不支援 `LIMIT` 語法的，令人有些納悶

## 解決辦法

### Native Query

最簡單暴力的解決辦法就是直接改用原生語法

```java
@Query(value = "SELECT * FROM `user` u WHERE u.status = 1 ORDER BY created_at DESC LIMIT 1", nativeQuery = true)
public User findFirst();
```

雖然可以簡單解決問題，但是寫原生語法是盡可能想避免的做法

### Pageable 做為參數帶入

JPQL 無法直接寫 `LIMIT` 但可以透過傳入 `Pageable` 的物件來實現，我們可以改寫 `DAO` 的方法參數

```java
@Query("SELECT u FROM User u WHERE u.status = 1 ORDER BY createdAt LIMIT 1")
public User findFirst(Pageable pageable);
```

使用的時候帶入參數

```java
userDao.findFirst(PageRequest.of(0, 1, Sort.by("createdAt").descending()));
```

這才是比較符合 JPA 用法的做法，不過由於 Pageable 的參數有三個，通常會再包裝一層在 service 中調用

## 補充

其實如果不需要寫到客製化語法，透過 JPA 自己產生的實作是可以套用 `LIMIT` 的，如下範例

```java
public List<User> findTop10ByOrderByCreatedAtDesc();
```

這樣可以產出能使用的語法，只是 method 會長一點

---

## 結語

JPA 的確是一個非常好用的 ORM 工具，在方便以及高度客製化中都提供方法實作，取得了很好的平衡，不過有時候這種便利的套裝也會引起一些問題跟誤用，需要對工具多加了解才能得心應手，發揮最大的效用
