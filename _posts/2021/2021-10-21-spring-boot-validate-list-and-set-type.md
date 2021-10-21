---
layout: post
title: "[踩雷紀錄] Spring boot validation 無法驗證 List、Set"
date: 2021-10-21 13:04:13 +0800
category: backend
img: cover/spring-boot.jpg
description: API 參數傳入 List 或是 Set 的例子也是屢見不鮮，但卻發現一般的 Validation 作法沒有辦法驗證到 List 的內容，不確定是不是 Bug，但還是必須要想點辦法來解
lang: zh-TW
tags: [java, spring boot, validation, 踩雷紀錄]
---

{{page.description}}

## 情境
一樣按照前幾篇的例子來看，照以下寫法期望可以去驗證 List 中每個 UserDto 內的資料

```java
@RestController()
@RequestMapping("/v1/user")
public class UserController {
    @PostMapping("")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void createUsers(@RequestBody @Valid List<UserDto> dto) {
    }
}
```

但實際上卻一個驗證都不會執行，如果要讓他生效需要一些其他的做法，大致查到有兩種解法

## 解法 1
在 Controller 的 class 層級加上 `@Validated`，簡單加上一個 annotation 就可以讓驗證生效，只是會改變拋出的 Exception，記得將例外處理事先寫好，如果不知道怎麼寫的可以參考[這裡](https://bingdoal.github.io/backend/2021/10/spring-boot-validate-request-body-and-nest-validate/)

```java
@Validated
@RestController()
@RequestMapping("/v1/user")
public class UserController {
    @PostMapping("")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void createUsers(@RequestBody @Valid List<UserDto> dto) {
    }
}
```

### 分組驗證
不過這個寫法直覺上如果想套用分組驗證的時候，會想跟一般寫法一樣，直接寫在參數裡，但這樣卻是不 work 的

```java
@Validated
@RestController()
@RequestMapping("/v1/user")
public class UserController {
    @PostMapping("")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void createUsers(@RequestBody @Validated(UserDto.Update.class) List<UserDto> dto) {
    }
}
```

必須改為以下寫法，把 annotation 提到 method 層級才會生效，並且參數的 `@Valid` 也不能忘記加上，以上三個 annotation 缺一不可

```java
@Validated
@RestController()
@RequestMapping("/v1/user")
public class UserController {
    @PostMapping("")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    @Validated(UserDto.Update.class)
    public void createUsers(@RequestBody @Valid List<UserDto> dto) {
    }
}
```

## 解法 2
第二種方式呢是自己去實作一個 List 的介面

```java
public class ValidList<E> implements List<E> {
    @Valid
    private final List<E> list;

    public ValidList() {
        this.list = new ArrayList<E>();
    }

    public ValidList(List<E> list) {
        this.list = list;
    }

    public List<E> getList() {
        return list;
    }
    // 以下需要 Override 全部 List 的方法
    @Override
    public int size() {
        return list.size();
    }

    @Override
    public boolean isEmpty() {
        return list.isEmpty();
    }

    ...
}
```

因為要 `@Override` 所有方法的緣故寫起來會有點長，但是只要寫一遍就好，使用上如下，跟一般做驗證的寫法就沒兩樣的，其實概念就是前幾篇講到的巢狀驗證，要使用分組驗證也與一般用法無異，個人其實是比較偏好這種方式的

```java
@RestController()
@RequestMapping("/v1/user")
public class UserController {
    @PostMapping("")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void createUsers(@RequestBody @Valid ValidList<UserDto> dto) {
    }
}
```

上述 `ValidList` 的寫法其實也可以透過 `lombok` 來換個簡潔一點的寫法

```java
@Getter
public class ValidList<E> implements List<E> {
    @Valid
    @Delegate
    private final List<E> list;

    public ValidList() {
        this.list = new ArrayList<E>();
    }

    public ValidList(List<E> list) {
        this.list = list;
    }
}
```

如果有 Set 的需求也可以按照上面作法重新實作一個 Set，這邊就不多寫了

---

## 結語
#### 兩個做法各有利弊，簡單說明一下個人的感覺:
+ 第一種作法可以繼續使用原生的 List 而不用特意去建立另一個實作，使用時也可以很直覺的調用 List 出來，不過缺點就是要加的 annotation 有點多，一個忘了加這個驗證就不會生效，如果又沒有好好測試的話可能就很難注意到
+ 第二種作法則是可以當作一般的類型使用，使用上的體驗是一致的，不過缺點就是必須要記得使用新建的 ValidList

其實兩種做法都可以達到目的，不過第二種作法個人認為是比較容易注意到的，事後需要的步驟越少在團隊中推廣應該也會是比較容易的