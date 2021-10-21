---
layout: post
title: Spring boot 分組驗證功能
date: 2021-10-21 09:12:04 +0800
category: backend
img: cover/spring-boot.jpg
description: 上篇介紹到 spring boot 簡單好用的參數驗證機制，但一般 API 常會分成 Create 跟 Update 兩種功能，兩種功能提供的參數大致上都會相同但會有一些驗證上的差異，如果要為了分開驗證去多寫一個類別也稍嫌有點麻煩且不好維護，所以 Validation 就提供了分組驗證的機制，下面介紹一下要如何來使用
lang: zh-TW
tags: [java, spring boot]
---

{{page.description}}

## 分組驗證設定
分組驗證上呢我們要先宣告幾個不同的 interface 來用作區別，作用有點像 Enum，然後在各個 annotation 中指定到 `groups` 的參數中，這樣就可以指定在該 group 的時候會去做這個驗證，也可以指定多個 group 給單一個 annotation

```java
@Data
public class UserDto {
    @NotBlank(groups = Create.class)
    @Size(max = 255, message = "{validation.user.name.length.max}")
    private String name;

    @NotBlank(groups = Create.class)
    @Null(groups = Update.class)
    private String password;

    @NotBlank(groups = Create.class)
    @Size(max = 255, message = "{validation.user.email.length.max}", groups={Create.class, Update.class})
    @Email()
    private String email;

    public interface Create{}
    public interface Update{}
}
```

## 分組驗證啟用
設定完成之後呢只要在需驗證的參數前改用 `@Validated(UserDto.Create.class)` 這種形式去指定這次驗證的組別就可以了，需注意 `@Valid` 不具備帶入 group 的功能，只能用 `@Validate`，`@Validate` 本身也是 `@Valid` 的封裝，基本上功能是一樣的

```java
@PostMapping("")
@ResponseStatus(HttpStatus.CREATED)
public void createUser(@RequestBody @Validated(UserDto.Create.class) UserDto userDco){
}

@PutMapping("/{userId}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void modifyUser(@PathVariable("userId") Long userId, @RequestBody @Validated(UserDto.Update.class) UserDto userDuo) {
}
```

---

## 結語

這個 Validation 的機制設計上實在具有巧思，用 interface 的方式傳入而不是單傳字串更能分辨不同 dto 間的驗證，最棒的是還會有 IDE 的語法提示