---
layout: post
title: "Spring boot Validation 進階操作: 客製化驗證、手動驗證"
date: 2021-10-28 14:19:23 +0800
category: backend
img: cover/spring-boot.png
description: 前幾篇說到基本的 Validation 該怎麼使用，不過實際情況下的商業邏輯往往是無比複雜的，一些內容的驗證可能不是這麼簡單的規則就可以帶過了，因此這篇就來說明下，怎麼在 Validation 的框架下建立自己的驗證方式，另外如果不想要用自動驗證的方式去處理，想要加點自己的驗證前後邏輯又該怎麼做呢
lang: zh-TW
tags: [java, spring boot, validation]
---

{{page.description}}

沒看過前幾篇的可以點這邊:

+ [Spring boot Validation 參數驗證機制](https://bingdoal.github.io/backend/2021/10/spring-boot-validate-request-body-and-nest-validate/)
+ [Spring boot Validation 分組驗證功能](https://bingdoal.github.io/backend/2021/10/spring-boot-validation-group-use-different-rule/)
+ [[踩雷紀錄] Spring boot validation 無法驗證 List、Set](https://bingdoal.github.io/backend/2021/10/spring-boot-validate-list-and-set-type/)

## Annotation

首先要先建立自己的驗證用 annotation，如下的寫法

```java
@Target({FIELD, PARAMETER})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = PasswordRuleValidator.class)
public @interface PasswordRule {
    int upperLetter() default 1;
    int lowerLetter() default 1;
    int number() default 1;
    int nonAlphaNum() default 0;

    String message() default "Invalid password";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

其中 `message`、`groups`、`payload` 是 Validation 固定會呼叫的屬性務必要加上，而其他屬性就可以自己根據需求客製化了，這邊用的例子是一個簡單的密碼規則驗證，可以設定大小寫字母、數字、特殊符號的各個數量

先寫完 annotation 會發現 `PasswordRuleValidator` 是不存在的，所以接下來就要自己來建立 validator 了

## Validator

建立一個 validator 去繼承 `ConstraintValidator`，泛型中帶入 annotation 跟預計傳入的參數型別，接著可以從 `initialize` 裡取得 annotation 的各個設定值，然後藉由 `isValid` 來實作驗證機制

```java
public class PasswordRuleValidator implements ConstraintValidator<PasswordRule, String> {
    private PasswordRule passwordRule;

    @Override
    public void initialize(PasswordRule constraintAnnotation) {
        ConstraintValidator.super.initialize(constraintAnnotation);
        this.passwordRule = constraintAnnotation;
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        int numberCount = 0;
        int lowerCount = 0;
        int upperCount = 0;
        int nonAlphaNumCount = 0;
        for (int i = 0; i < value.length(); i++) {
            char ch = value.charAt(i);
            if (Character.isDigit(ch)) {
                numberCount++;
            } else if (Character.isUpperCase(ch)) {
                upperCount++;
            } else if (Character.isLowerCase(ch)) {
                lowerCount++;
            } else {
                nonAlphaNumCount++;
            }
        }
        return numberCount >= passwordRule.number() &&
                lowerCount >= passwordRule.lowerLetter() &&
                upperCount >= passwordRule.upperLetter() &&
                nonAlphaNumCount >= passwordRule.nonAlphaNum();
    }
}
```

這邊簡單舉個計算字元類型的範例，整個過程可以自行客製化，這樣一來就完整整個驗證機制的建立了，接下來只要在需要的屬性上面加上 annotation，跟以往的使用體驗完全一樣，驗證機制就會自己套用了

```java
@Data
public class UserDto{
    @NotBlank()
    private String name;

    @NotBlank()
    @PasswordRule()
    private String password;

    @NotBlank()
    @Email()
    private String email;
}
```

## 手動進行驗證

有時候驗證失敗可能不想只是簡單的拋出錯誤訊息給前端，這時候需要自己加上驗證失敗的邏輯作法如下

```java
@Slf4j
@RestController()
@RequestMapping("/v1/user")
public class UserController {
    @Autowired
    Validator validator;

    @PostMapping("")
    @ResponseStatus(HttpStatus.OK)
    public void createUser(@RequestBody UserDto userDto) {
        Set<ConstraintViolation<UserDto>> validateSet = validator.validate(userDto);
        if (!validateSet.isEmpty()) {
            for (ConstraintViolation<UserDto> violation : validateSet) {
                log.info("violation message: {}", violation.getMessage());
            }
            return;
        }

        ...
    }
}
```

首先要注入 `Validator` 的資源，利用這個物件來進行資料驗證，並且要注意不要啟動自動驗證的程序，也就是要把 `@Valid` 跟 `@Validated` 都移除掉，經過 `validator.validate` 驗證過後會回傳驗證結果的集合，只有發生驗證錯誤才會收集起來，如果想要加上分組驗證機制的話只要在參數之中加入 `interface` 就會正常套用了

```java
Set<ConstraintViolation<UserDto>> validateSet = validator.validate(userDto, UserDto.Create.class);
```

## 其他補充

### 快速驗證失敗

一般在進行驗證的時候，其實會將所有參數都跑完，最後回傳所有驗證結果，但是實際上我們比較希望跑到第一個錯誤就回傳，可以節省一些效能，跟沒必要的操作，設定方式如下

```java
@Configuration
public class AppConfig {
    @Bean
    public LocalValidatorFactoryBean localValidatorFactoryBean() {
        LocalValidatorFactoryBean bean = new LocalValidatorFactoryBean();
        bean.getValidationPropertyMap().put("hibernate.validator.fail_fast", "true");
        return bean;
    }
}
```

---

## 結語

Validation 除了方便之外，提供的彈性也是非常好的，可以在很高的程度上進行客製化，有了手動驗證的方式甚至自己重新覆寫整個自動驗證都不是什麼困難事
