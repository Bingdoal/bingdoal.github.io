---
layout: post
title: Spring boot Validation 分組驗證功能
date: 2021-10-21 09:12:04 +0800
category: backend
img: cover/spring-boot.jpg
description: 上篇介紹到 spring boot 簡單好用的參數驗證機制，但一般 API 常會分成 Create 跟 Update 兩種功能，兩種功能提供的參數大致上都會相同但會有一些驗證上的差異，如果要為了分開驗證去多寫一個類別也稍嫌有點麻煩且不好維護，所以 Validation 就提供了分組驗證的機制，下面介紹一下要如何來使用
lang: zh-TW
tags: [java, spring boot, validation]
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

## 預設組別驗證
在上面的例子中會發現 `@Email` 跟 `@Size` 是沒有被設定組別的，預想情況應該是每個組別都會去執行驗證，但實際測試後就會發現其實是都不執行，對於 Validation 的機制來說他們其實都屬於一個 `Default` 的組別，所以這邊我們可以做點更改

```java
@Data
public class UserDto {
    ...

    public interface Create extend Default{}
    public interface Update extend Default{}
}
```

讓用來分組的 interface 去繼承 `Default`，就能讓這些組別去套用預設沒有被分組的 annotation


## 巢狀驗證
如同上篇也有提到的巢狀驗證，如果在分組的情況下巢狀驗證該怎麼進行呢，其實完全一樣，不需更改任何地方，只要加上 `@Valid` 就好，也許在嘗試的時候會發現 `@Validated` 其實是不允許被寫在 class 的 field 層的所以也只能用 `@Valid` 來標記

```java
@Data
public class UserDto {
    @Valid
    private UserProperty property;

    public interface Create{}
    public interface Update{}
}

@Data
public class UserProperty{
    @NotBlank(groups = UserDto.Create.class)
    @Size(max = 255)
    private String name;

    @NotBlank(groups = UserDto.Create.class)
    @Null(groups = Update.class)
    private String password;

    @NotBlank(groups = UserDto.Create.class)
    @Size(max = 255, groups={UserDto.Create.class, UserDto.Update.class})
    @Email()
    private String email;
}
```

而當巢狀驗證設定好之後，從最外層傳入的 group 便會一路傳下去，到最後要驗證的屬性就會按照該 group 去做驗證，不過必須注意設定的 group 一定要是同一個來源，使用上其實相當直覺

```java
@PostMapping("")
@ResponseStatus(HttpStatus.CREATED)
public void createUser(@RequestBody @Validated(UserDto.Create.class) UserDto userDco){
}
```

---

## 結語

這個 Validation 的機制設計上實在具有巧思，用 interface 的方式傳入而不是單傳字串更能分辨不同 dto 間的驗證，最棒的是還會有 IDE 的語法提示