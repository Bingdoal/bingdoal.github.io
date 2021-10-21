---
layout: post
title: Spring Boot Validation 參數驗證機制
date: 2021-10-20 15:29:01 +0800
category: backend
img: cover/spring-boot.jpg
description: Api 的請求參數一般都需要做合法性的驗證，每個參數都必須確保沒有問題，才不會導致有預期外的資料出現，導致系統發生不可預期的結果，Spring boot 強大的生態下 Validation 的機制自然是有好好地包含在其中了，只要善用各種 Annotation 就可以簡單做到參數驗證，還可以客製化錯誤訊息，下面就簡單介紹下用法
lang: zh-TW
tags: [java, spring boot]
---

{{page.description}}

## 引入依賴
首先當然要先加入 validation 的相關依賴

```xml
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
```

## 基本用法
首先傳入參數必須是個有被事先定義好的類別，就像下面:

```java
@Data
public class UserDto {
    @NotBlank()
    private String name;
    @NotBlank()
    private String password;
    @Size(max = 255, message = "Email 長度不得大於 255")
    @Email(message = "Email 格式不正確")
    private String email;
}
```

並且可以看到在這個類別的屬性上都加上了不同的 annotation 有各自不同的驗證意義，接著在傳入的 API 參數上加上 `@Valid` 的 annotation 就像下面所示，這樣在打這個 API 的時候就會根據設定去做參數的驗證，如果驗證失敗就會拋出 `MethodArgumentNotValidException`

```java
@RestController()
@RequestMapping("/v1/user")
public class UserController {
    @PostMapping("")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void createUser(@RequestBody @Valid UserDto dto) {
        log.info("User: {}", dto);
    }
}
```

特別的是 Validation 不只能夠用在 Controller，在一般的方法中也可以使用，這種情況下驗證失敗則會拋出 `ConstraintViolationException`

```java
@Validated
@Service
public class UserService {
    public void testValid(@Valid UserDto dto){
        log.info("User: {}", dto);
    }
}
```

## 巢狀驗證
使用一陣子之後會發現，如果說今天的 dto 有兩層以上會沒辦法套入第二層的 Validation，以下面為例子，這樣的寫法沒辦法進行 `UserProperty` 的驗證

```java
@Data
public class UserDto {
    private UserProperty property;
}

@Data
public class UserProperty {
    @NotBlank()
    private String name;
    @NotBlank()
    private String password;
    @Size(max = 255, message = "Email 長度不得大於 255")
    @Email(message = "Email 格式不正確")
    private String email;
}
```

解法其實也很簡單，寫法再加上，`@Valid` 驗證就會往下走，而如果希望 property 一定要存在的話也可以加上 `@NotNull` 但不是必要的

```java
@Data
public class UserDto {
    @Valid
    @NotNull
    private UserProperty property;
}
```

## 錯誤訊息
測試過錯誤的輸入之後應該會發現錯誤訊息不是太好看，而且因為是拋出 Exception 所以對於前端來說會收到 500 錯誤，這不符合實際情況，因此這邊要來做一下錯誤處理，用 ExceptionHandler 去抓取驗證相關的 Exception，然後包裝一下錯誤訊息傳給前端，整個流程就完成了

```java
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {
    @Autowired
    private ObjectMapper objectMapper;

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<?> handleConstraintViolationException(ConstraintViolationException ex) {
        ConstraintViolation<?> constraintViolation = ex.getConstraintViolations().iterator().next();
        String defaultMessage = constraintViolation.getMessage();
        JsonNode jsonNode = objectMapper.createObjectNode().put("message", defaultMessage);
        return ResponseEntity.status(400).body(jsonNode);
    }

    @Override
    protected ResponseEntity<Object> handleExceptionInternal(final Exception ex, final Object body, final HttpHeaders headers, final HttpStatus status, final WebRequest request) {
        if (ex instanceof MethodArgumentNotValidException) {
            return handleArgumentInvalid((MethodArgumentNotValidException) ex);
        }
        log.error("Error: ", ex);
        JsonNode jsonNode = objectMapper.createObjectNode().put("message", ex.getLocalizedMessage());
        return ResponseEntity.status(status).body(jsonNode);
    }

    private ResponseEntity<Object> handleArgumentInvalid(MethodArgumentNotValidException ex) {
        String defaultMessage = ex.getBindingResult().getAllErrors().get(0).getDefaultMessage();
        JsonNode jsonNode = objectMapper.createObjectNode().put("message", defaultMessage);
        return ResponseEntity.status(400).body(jsonNode);
    }
}
```

## Annotation 一覽
這邊條列一下Validation 預設提供的常用 Annotation，方便之後使用查找

|             annotation              | 說明                                                                            |
| :---------------------------------: | :------------------------------------------------------------------------------ |
|         `@NotNull`/`@Null`          | 不得為 Null/必須為 Null                                                         |
|    `@AssertFalse`/`@AssertTrue`     | 必須為 False/True                                                               |
|            `@Min`/`@Max`            | 必須為數字類型，並且限制最大最小值                                              |
|     `@DecimalMin`/`@DecimalMax`     | 內容必須為數字，可接受字串，並且限制最大最小值                                  |
|          `@Size(max,min)`           | 限制內容長度，接受字串、Map、List 等等有 size 概念的類別                        |
|             `@NotBlank`             | 必須為字串，必須含有至少一個非空白字元，且不得為 Null                           |
|             `@NotEmpty`             | 接受字串、Map、List 等等有 empty 概念的類別，不得為空或 Null                    |
|     `@Digits(integer,fraction)`     | 內容必須為數字，指定位數的最大長度，`integer` 表整數部分、`fraction` 表小數部分 |
|              `@Email`               | 字串必須為 email 格式                                                           |
|       `@Negative`/`@Positive`       | 必須為數字類型，指定正或負值                                                    |
| `@NegativeOrZero`/`@PositiveOrZero` | 必須為數字類型，指定正或負值，且接受零                                          |
|         `@Pattern(regexp)`          | 字串須符合 regular expression                                                   |
|          `@Future`/`@Past`          | 日期或時間類型，必須為 未來/過去 的時間                                         |
| `@FutureOrPresent`/`@PastOrPresent` | 日期或時間類型，必須為 未來/過去 或當下的時間                                   |

還有由 `hibernate.validator` 另外提供的

|         annotation         | 說明                                                                      |
| :------------------------: | :------------------------------------------------------------------------ |
|     `@Range(min,max)`      | 內容須為數字，可以視為 `@Max` 跟 `@Min` 的合用                            |
|     `@Length(min,max)`     | 專用於 String 的 `@Size`                                                  |
| `@URL(protocol,host,port)` | 必須為字串，判別 URL 格式，且可以指定 protocol、host、port 不指定則為任意 |


其實還有一些 annotation 沒有寫到，但那些基本上沒怎麼用過，大部分的驗證應該都可以透過上面的 annotation 來做到了，如果有比較特殊的需求也有提供客製化驗證的功能，之後的文章會再行介紹

---
## 結語
簡單幾個 annotation 就可以做好資料驗證實在是非常方便，省去大量的時間跟做苦工的部分，這篇簡單介紹基本用法，下篇會再介紹分組驗證以及巢狀驗證的問題
