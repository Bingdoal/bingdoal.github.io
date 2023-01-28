---
layout: post
title: "JPA 原生的複雜查詢 Specification"
date: 2023-01-28 13:41:31 +0800
category: backend
img: cover/spring-boot.png
description: "之前有介紹過 QueryDSL 這個好用的套件用來做到複雜查詢以及自動匹配 filter 的功能，後來發現其實 JPA 也有內建對於複雜查詢的使用框架，雖然自己本身還是比較常用 QueryDSL 不過也紀錄一下 JPA Specification 的用法"
lang: zh-TW
tags: [spring boot]
published: true
---

{{page.description}}

之前的 QueryDSL 可以看[這邊][querydsl-article]

## JpaSpecificationExecutor

首先要來認識到 JPA 用來複雜查詢的介面 `JpaSpecificationExecutor`

```java
public interface JpaSpecificationExecutor<T> {
  Optional<T> findOne(@Nullable Specification<T> spec);

  List<T> findAll(@Nullable Specification<T> spec);

  Page<T> findAll(@Nullable Specification<T> spec, Pageable pageable);

  List<T> findAll(@Nullable Specification<T> spec, Sort sort);

  long count(@Nullable Specification<T> spec);
}
```

基本上就是通過 `Specification` 的 class 做為參數用來查詢單筆或多筆的資料

一樣就像其他介面一下只要在 `DAO` 直接繼承 spring boot 就會幫忙建立以及注入了

## 程式範例

下面是簡單的建立 `Specification` 的範例

```java
@Service
public class UserDaoService {

  @Autowired
  private UserDao userDao;

  public List<User> findWithCondition(String name, String email) {
    Specification<User> spec = (root, query, builder) -> {
      List<Predicate> predicates = new ArrayList<>();
      if (name != null) {
        Expression<String> nameExp = root.get("name").as(String.class);
        predicates.add(builder.equal(nameExp, name));
      }
      if (email != null) {
        Expression<String> emailExp = root.get("email").as(String.class);
        predicates.add(builder.equal(emailExp, email));
      }
      query.where(builder.and(predicates.toArray(new Predicate[0])));
      return query.getRestriction();
    };
    return userDao.findAll(spec);
  }
}
```

`Specification` 本身也是個介面，裡面只有一個方法需要實作，所以可以用 lambda 簡寫

```java
public interface Specification<T> extends Serializable {

  ...

  @Nullable
  Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder);
}
```

基本的邏輯做法就是用 `root` 取用 model 指定 type，`builder` 進行邏輯處理，`query` 就是用來組合語句，除了 `where` 還有 `order by` 跟 `group by` 可以用

---

## 結語

其實用法也不難，不過比起來個人還是更喜歡用 QueryDSL，主要是 QueryDSL 會自動產出相應得類別用來查詢，`Specification` 雖然有類似的做法，但要自己手動建立就稍嫌麻煩了，QueryDSL 的引用成本也不高

[querydsl-article]: https://bingdoal.github.io/backend/2021/08/spring-boot-jpa-and-querydsl/