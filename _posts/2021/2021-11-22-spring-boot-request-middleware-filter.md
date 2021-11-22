---
layout: post
title: "Spring boot Filter 機制，攔截請求與回應"
date: 2021-11-22 10:21:34 +0800
category: backend
img: cover/spring-boot.jpg
description: 這次要介紹的 Filter 是用來針對 Http 的請求與回應在途中攔截下來做一些處理，運作邏輯上有一點像先前寫過的 AOP，但 AOP 是以 Method 為視角去攔截，而 Filter 則是以 Servlet 的層級來攔截，應用的場景會稍有不同
lang: zh-TW
tags: [java, spring boot]
---

{{page.description}}

## Filter

首先來撰寫一個簡單的 Filter 將收到的請求以及回應的各種資訊記錄下來

```java
@Slf4j
public class OnceFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req,
                                    HttpServletResponse res,
                                    FilterChain chain) throws ServletException, IOException {
        ContentCachingRequestWrapper reqWrapper = new ContentCachingRequestWrapper(req);
        final String queryString =
            req.getQueryString() == null ? "" : "?" + req.getQueryString();

        log.info("\t[Request] {} {}{}", req.getMethod(), req.getRequestURI(), queryString);
        if (reqWrapper.getContentLength() > 0) {
            log.info("\t[Request] Body: {}", new String(reqWrapper.getContentAsByteArray()));
        }

        chain.doFilter(req, res);

        ContentCachingResponseWrapper resWrapper = new ContentCachingResponseWrapper(res);
        if (resWrapper.getContentSize() > 0) {
            log.info("\t[Response] Body: {}", new String(resWrapper.getContentAsByteArray()));
        }
    }
}
```

一般的 Filter 其實也可以透過 `Implements Filter` 來實作，而 Spring 多提供了一個 `OncePerRequestFilter` 用以確保每個 Request 只會經過一次

而一個請求在多個 Filter 間串接起來就被稱為 `FilterChain`，`chain.doFilter` 就是指往下一個 Filter 前進，並且在那之後才收到 Response 的物件，在 `chain.doFilter` 執行之前 Response 是拿不到東西的

並且由於 Servlet 容器不允許多次取得內容資訊，所以透過 `ContentCachingRequestWrapper` 讓我們能夠多次呼叫資料主體來取得 Request 以及 Response Body

## 註冊
將上面的 Filter 完成後應該會發現實際上還沒有作用，Spring boot 的物件基本上都需要通過註冊成為 Bean 的過程來啟用，Filter 也不例外，下面介紹三種 Filter 註冊的方式

### FilterRegistrationBean
首先是比較正規透過 Configuration 的方式來註冊完整的 Filter 屬性

```java
@Configuration
public class AppConfig {
    @Bean
    public FilterRegistrationBean filterRegistration() {
        FilterRegistrationBean<OnceFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new OnceFilter());
        bean.addUrlPatterns("/*");
        bean.setOrder(1);
        bean.setName("onceFilter");
        bean.setEnabled(true);
        return bean;
    }
}
```
可以針對需要的 Url 來 Filter，如果有多個 Filter 也可以指定順序(數字越小越優先)以及啟用與否

### WebFilter
不想多寫一個 Configuration 來註冊的話可以透過 `@WebFilter` 來取代，一樣可以指定 Url，執行順序則可以透過 `@Order` 來給定，只不過 `@WebFilter` 不是由 Spring 提供的，因此預設不會被掃描到，必須在進入點的主類別額外加上 `@ServletComponentScan` 才可以使用

```java
@SpringBootApplication()
@ServletComponentScan()
public class Application {
    SpringApplication.run(Application.class, args);
}

@WebFilter(urlPatterns = "/*", filterName = "onceFilter")
@Order(1)
@Slf4j
public class OnceFilter extends OncePerRequestFilter {
    ...
}
```

### Component
如果還是嫌棄上述的作法太麻煩，最簡單的作法可以直接加上 `@Component` 直接將物件在 Spring boot 啟用時註冊成 Bean，預設上的 Url 就是全部，如果需要排序也可以加上 `@Order`，實際上沒有什麼特殊用途的話應該是蠻堪用了

```java
@Component
@Order(1)
@Slf4j
public class OnceFilter extends OncePerRequestFilter {
    ...
}
```