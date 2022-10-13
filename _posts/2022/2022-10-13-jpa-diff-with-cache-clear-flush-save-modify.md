---
layout: post
title: "[踩雷紀錄] 在 JPA 使用 @Modifying 遇到 cache 問題"
date: 2022-10-13 14:34:43 +0800
category: JPA 是 Spring boot 現行架構下最流行的 ORM 框架，雖然大部分的情況我們都可以直接使用 JPA 提供的介面來進行操作，不過有時候還是會有一些特殊的客製化需求要處理，這次的問題發生在我們撰寫了一個的 @Modifying 的方法，但卻造成後續操作不如預期，之後又陸續踩到各種地雷，特別紀錄一下
img: cover/spring-boot.png
description:
lang: zh-TW
tags: [踩雷紀錄, spring boot]
published: true
---

{{page.description}}

## 問題描述

+ 範例程式

```java
// WalletDao
public interface WalletDao extends JpaRepository<Wallet, Long>, JpaSpecificationExecutor<Wallet>{
    @Modifying
    @Query("UPDATE Wallet w SET w.amount=w.amount-:amount WHERE w.userId=:userId AND w.amount-:amount >= 0")
    int minusCash(Long userId, Long amount)

    Wallet findByUserId(Long userId);
}

// WalletService
@Sl4j
@Service
public class WalletService{
    @Autowired
    private WalletDao walletDao;

    @Transactional
    public void bookOrder(Long userId, Long amount){
        Wallet wallet = walletDao.findByUserId(userId);
        wallet.setModificationTime(LocalDateTime.now());
        walletDao.save(wallet);
        log.info("wallet: {}", wallet);
        int result = walletDao.minusCash(userId, amount);
        if(result <= 0){
            log.error("walletDao.minusCash fail.");
            return;
        }
        wallet = walletDao.findByUserId(userId);
        log.info("wallet: {}", wallet);
    }
}
```

考慮以上範例程式會發現執行情況似乎與預期不符合

假設一開始撈出的 wallet amount 是 1000 然後我想要減少 500

但卻會發現兩次 log 印出來的 wallet 都是 1000，然而實際去看資料庫會發現資料其實有被更新

### 問題原因

原來 JPA 在操作的時候自己內部有先做一層的 cache 每次 select 的時候會先找 cache 中有沒有可用的 entity，有的話會直接回傳

### 第一個問題解決方法

找到原因之後下意識想，那我只要想辦法清除 cache 就可以了吧，於是就改了下面的 code

```java
...
    @Modifying(clearAutomatically = true)
    @Query("UPDATE Wallet w SET w.amount=w.amount-:amount WHERE w.userId=:userId AND w.amount-:amount >= 0")
    int minusCash(Long userId, Long amount)
...
```

然後執行的時候發現 log 沒錯了，真的有取當下更新後的 amount

但是查看資料庫的時候卻發現了第二個問題，那就是 `ModificationTime` 沒有更新到

### 問題原因

原來 JPA 的 cache 裏不只有 select 過的 entity 還一並把 save 執行的 update 以及 insert 一起 cache 起來

為了最後送出的時候可以做 batch 的優化或是合併語法，才不會頻繁跟資料庫做連線請求

### 第二個問題解決方法

把 jpa 的 cache clear 掉之後的確可以解決我們 select 的問題，所以依舊必須清除 cache

因此修改成下面的 code

```java
...
    @Modifying(clearAutomatically = true, flushAutomatically = true)
    @Query("UPDATE Wallet w SET w.amount=w.amount-:amount WHERE w.userId=:userId AND w.amount-:amount >= 0")
    int minusCash(Long userId, Long amount)
...
```

`flushAutomatically` 可以讓這個 `Modifying` 的 method 不只是清除 cache 還會將裡面的 Update 語法實際發送到資料庫實時更新資料

基本上為了避免問題，撰寫額外 `@Modifying` 時應該都要加上這兩個設定比較保險

---

## 結語

雖說利用一些設定可以達到我們的要求，但是經過這次經驗我認為應該用 JPA 的時候就盡可能都用 save 來更新資料，盡量不要寫客製化的 UPDATE 或是 INSERT 語法
