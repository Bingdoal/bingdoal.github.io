---
layout: post
title: "[踩雷紀錄] 各種 Annotation 在同個 class 中調用失效"
date: 2022-10-05 11:48:57 +0800
category: backend
img: cover/spring-boot.png
description: 在 spring boot 的框架加護之下，很多的功能都可以透過簡單的 annotation 啟用，像是 @Transactional 或是 @Cacheable 都是常會用到的，不過這些功能卻會在同個 class 中互相調用的時候失效，下面提供幾個解決方案
lang: zh-TW
tags: [踩雷紀錄, spring boot]
published: true
---

{{page.description}}

## 原因

由於 Spring boot 在做依賴注入的時候其實會做一層代理(Proxy)，以便進行像是 AOP 的切片操作

所以如果是從不同的類別透過 Spring boot 注入進來的 class 才會套用到這層代理，內部互相調用是用不到的

也可以得知不只是 Annotation 會失效，就連 AOP 都不會起作用

## 解決方案

+ 範例程式:

```java
@Sl4j
class UserService{
    @Autowired
    private UserDao userDao;
    @Autowired
    private PostDao postDao;

    public void getUserPosts(Long userId) throws Exception {
       List<Post> posts = findPostByUserFromCache(userId);
       log.info("posts: {}", posts);
    }

    @Cacheable("post")
    public List<Post> findPostByUserFromCache(Long userId) throws Exception {
        return postDao.findByUserId(userId);
    }
}
```

### 獨立 Service

最簡單也最推薦的其實就是獨立寫另一個 class 來調用方法

這也可以幫助撰寫程式架構的時候思考每個 service 跟 method 分工的合理性

### AopContext.currentProxy()

另一個做法是透過直接取得被代理過的實例來呼叫方法可以透過 `AopContext.currentProxy()` 來辦到

參考以下程式碼

```java
...

public void getUserPosts(Long userId) throws Exception {
    List<Post> posts = ((UserService)AopContext.currentProxy()).findPostByUserFromCache(userId);
    log.info("posts: {}", posts);
}

...
```

這樣可以拿到該類別被 AOP 代理的實例然後才去調用方法，所以能夠正確套用 AOP 的功能

不過要使用這個方法需要加一個 `@EnableAspectJAutoProxy(exposeProxy = true)` 的 annotation 到 Spring boot 的起始類別，把 AOP 的 Proxy 實例曝露出來

### ApplicationContext.getBean()

這個方法跟上面有點像，一樣是取得正確的實例來調用方法

不過這次我們直接拿 Spring boot 幫我們創建好的 Bean 來使用，看到下面程式

```java
...

@Autowired
private ApplicationContext applicationContext;

public void getUserPosts(Long userId) throws Exception {
       List<Post> posts = applicationContext.getBean(UserService.class).findPostByUserFromCache(userId);
       log.info("posts: {}", posts);
}

...
```

簡單解決，也不需要額外開功能，效果基本上一樣，我是比較偏好這樣寫的

我自己會將這部分在包成一個工具來使用

```java
@Component
public class ContextUtils {
    private static ApplicationContextUtils instance;

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

包成 static method 可以方便我隨時隨地使用，也不需要再額外注入