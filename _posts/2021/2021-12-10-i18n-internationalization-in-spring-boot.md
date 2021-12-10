---
layout: post
title: "Spring boot 的 I18n 設定與進一步包裝"
date: 2021-12-10 15:45:18 +0800
category: backend
img: cover/spring-boot.jpg
description: 在專案中加入多國語言設定是很常見的需求，Spring boot 當然也有相應的支援，透過簡易的設定就可以套用
lang: zh-TW
tags: [java, spring boot, i18n]
published: true
---

{{page.description}}

## 依賴引用

Spring boot 的 i18n 支援是含在 `spring-context-support` 裡的，理論上當在使用 Spring boot 的時候應該都會有才是

## application.yaml
`basename` 代表語系檔的路徑以及檔名，依照這裡的設定須將語系檔放在 `resource/i18n` 之下，且命名為 `message.properties`
```yaml
messages:
  basename: i18n/messages
  encoding: UTF-8
  cache-duration: 3600
```

## 語系檔範例

不同語系需放在不同檔案，命名上用上面設定的 `basename` 後面加上 `_語系` 來區隔

{% raw %}
```yaml
# message.properties
user.controller.not.found.by.id=[預設] 使用者 ID [{0}] 不存在

# message_en_US.properties
user.controller.not.found.by.id=UseID [{0}] do not exist.

# message_zh_TW.properties
user.controller.not.found.by.id=使用者 ID [{0}] 不存在
```

`{0}` 是用來帶入參數的，讓訊息可以做到某種程度的客製化

{% endraw %}

## 套用
注入 `MessageSource` 的資源，透過 key 值去對應訊息，如果後面帶入的語系不存在就會用預設的 `message.properties` 內容

```java
@Slf4j
@RestController
public class DemoController {

    @Autowired
    private MessageSource messageSource;
    @Autowired
    private UserDao userDao;

    @GetMapping("/{userId}")
    @ResponseStatus(HttpStatus.OK)
    public User getOneUser(@PathVariable("userId") Long userId) throws Exception {
        Optional<User> userOption = userDao.findById(userId);
        if (userOption.isEmpty()) {
            String msg = messageSource.getMessage(
                "user.controller.not.found.by.id",
                new String[]{userId.toString()},
                Locale.TAIWAN);
            log.error(msg);
        }
        return userOption.get();
    }
}
```

## 進階使用

上面的範例是直接寫入 `Locale.TAIWAN` 來套用語系，實際情境下通常是透過 API 帶入語系參數來決定訊息的語系，因此可以改寫成

```java
...
if (userOption.isEmpty()) {
    String msg = messageSource.getMessage(
        "user.controller.not.found.by.id",
        new String[]{userId.toString()},
        LocaleContextHolder.getLocale())
    log.error(msg);
);
...
```

`LocaleContextHolder.getLocale()` 會從 request 的 header 欄位 `Accept-Language` 來解析語系，如果語系找不到會先抓主機的預設語系，若預設語系也不存在才會去抓 `message.properties`

### 進一步包裝

每次使用都要像上面寫法注入後呼叫 `getMessage` 參數的帶法也不是這麼方便，而且語系的參數基本上就是帶入 `LocaleContextHolder.getLocale()`

秉持著不願意多寫重複的 code，於是決定多做一層包裝，寫成類似 builder 的方式來讓撰寫可以更簡潔，加入例外處理當輸入不存在的 key 值時可以直接回傳 key 而不會導致 crash，其實最主要是可以在 unit test 的時候不用去管 i18n 的套用

```java
@Slf4j
public class I18nDto {
    private I18nDto(String key) {
        this.key = key;
        this.locale = LocaleContextHolder.getLocale();
    }

    private String key;
    private List<String> args = new ArrayList<>();
    private Locale locale;

    public static I18nDto key(String key) {
        return new I18nDto(key);
    }

    public I18nDto args(Object... objects) {
        for (Object object : objects) {
            args.add(Objects.toString(object));
        }
        return this;
    }

    @Override
    public String toString() {
        String str = key;
        try {
            MessageSource messageSource = StaticApplicationContext.getBean(MessageSource.class);
            str = messageSource.getMessage(key, args.toArray(), locale);
        } catch (Exception ex) {
            log.warn("I18N toString: {}", ex.getMessage(), ex);
        }
        return str;
    }
}
```

由於這個 class 不會被建立成 Spring boot 的 `Bean` 因此沒辦法用自動注入的方式取得 `MessageSource`，採用靜態方法去拿到 `ApplicationContext` 下的 `MessageSource`

```java
@Component
public class StaticApplicationContext {
    private static StaticApplicationContext instance;

    @Autowired
    private ApplicationContext applicationContext;

    @PostConstruct
    public void registerInstance() {
        instance = this;
    }

    public static <T> T getBean(Class<T> clazz) {
        return instance.applicationContext.getBean(clazz);
    }
}
```

之後調用時只需要

```java
...
if (userOption.isEmpty()) {
    String msg = I18nDto.key("user.controller.not.found.by.id")
                        .args(userId).toString();
    log.error(msg);
);
...
```

語法上簡潔許多，調用時也不需要額外注入資源