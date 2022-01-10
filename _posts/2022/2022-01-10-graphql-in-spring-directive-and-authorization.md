---
layout: post
title: "GraphQL in Spring Boot(四) 彈性附加功能 Directive 與 Authentication 實現"
date: 2022-01-10 11:51:38 +0800
category: backend
img: cover/graphql-in-spring.png
description: "Directive 可以想成是一種對於 Schema 的額外修飾，可以用來實現一些額外的檢查或是資料的轉換，有種額外插件的感覺，除了在 Server 端可以實現之外在 Client 端也有相應的功能，並且本篇會間單介紹下如何使用 Directive 達到 Authentication 的功能"
lang: zh-TW
tags: [backend,spring boot,graphql]
published: true
---

{{page.description}}

沒看過前幾篇的可以點這邊:
+ [GraphQL in Spring boot(一) 基本 Query 操作](https://bingdoal.github.io/backend/2021/12/graphql-in-spring-boot-basic-query/)
+ [GraphQL in Spring boot(二) Mutation 與 Scalar Type](https://bingdoal.github.io/backend/2022/01/graphql-in-spring-mutation-and-scalar-type/)
+ [GraphQL in Spring Boot(三) DataLoader 解決 Resolver 重複執行問題](https://bingdoal.github.io/backend/2022/01/graphql-in-spring-dataloader/)

## Schema
+ 用一個簡單的大寫轉換作為範例，Query `hello` 回傳 `world`

```graphql
directive @uppercase on FIELD_DEFINITION

type Query{
    hello: String! @uppercase
}
```

+ 先宣告好 `directive` 的關鍵字以及適用範圍，可宣告的適用範圍如下:

![]({{site.baseurl}}/assets/img/graphql-directive-on.png)

## 實作

```java
@Configuration
public class UppercaseConfig {
    @Bean
    public SchemaDirective uppercaseDirective() {
        return new SchemaDirective("uppercase", new SchemaDirectiveWiring() {
            @Override
            public GraphQLFieldDefinition onField(SchemaDirectiveWiringEnvironment<GraphQLFieldDefinition> env) {
                DataFetcher dataFetcher = DataFetcherFactories.wrapDataFetcher(env.getFieldDataFetcher(),
                        (dataFetchingEnvironment, value) -> {
                            if (value == null) {
                                return null;
                            }
                            return ((String) value).toUpperCase();
                        });
                return env.setFieldDataFetcher(dataFetcher);
            }
        });
    }
}
```

+ 註冊一個 `SchemaDirective` 的 `Bean` 並寫好對應的 Directive name，根據適用範圍 Override 不同的 method 去進行操作，操作發生在 `DataFetcherFactories.wrapDataFetcher` 的 callback 當中

+ 操作輸出如下，簡單轉換成大寫內容了

![]({{site.baseurl}}/assets/img/graphql-directive-output-1.png)

+ 相關的應用像是日期格式轉換或是資料格式檢查都可以用上

### 帶參數
+ 或是 Directive 也可以帶入參數某種程度上達到共用或是增加彈性

```graphql
directive @caseConvert(convert: String) on FIELD_DEFINITION

type Query{
    hello: String! @caseConvert(convert: "UPPER")
}
```

+ 取參數的方式稍微迂迴一點

```java
@Configuration
public class CaseConvertConfig {
    @Bean
    public SchemaDirective caseConvertDirective() {
        final String directiveName = "caseConvert";
        return new SchemaDirective(directiveName, new SchemaDirectiveWiring() {
            @Override
            public GraphQLFieldDefinition onField(SchemaDirectiveWiringEnvironment<GraphQLFieldDefinition> env) {
                GraphQLDirective directive = env.getDirective(directiveName);
                DataFetcher dataFetcher = DataFetcherFactories.wrapDataFetcher(env.getFieldDataFetcher(),
                        (dataFetchingEnvironment, value) -> {
                            if (value == null) {
                                return null;
                            }
                            StringValue stringValue = ((StringValue) directive.getArgument("convert")
                                    .getArgumentValue().getValue());
                            if (stringValue == null) {
                                return null;
                            }

                            String convert = stringValue.getValue();
                            switch (convert) {
                                case "LOWER":
                                    return ((String) value).toLowerCase();
                                case "UPPER":
                                    return ((String) value).toUpperCase();
                            }
                            return null;
                        });
                return env.setFieldDataFetcher(dataFetcher);
            }
        });
    }
}
```

## Authentication

Directive 其中最想用來處理的問題就是 Authentication 了，以往 RESTful API 的作法通常是在 URL 上做驗證，而 GraphQL 是以 Query 為單位，因此 Authentication 也應該做在 Query 上，因此 Directive 就派上用場了

+ Schema

```graphql
directive @isAuthenticated on FIELD_DEFINITION

type Query{
    hello: String! @isAuthenticated
}
```

+ Java 實作

```java
@Configuration
public class AuthenticationConfig {
    @Autowired
    private JwtTokenService jwtTokenService;

    @Bean
    public SchemaDirective isAuthenticatedDirective() {
        return new SchemaDirective("isAuthenticated", new SchemaDirectiveWiring() {
            @Override
            public GraphQLFieldDefinition onField(SchemaDirectiveWiringEnvironment<GraphQLFieldDefinition> env) {
                DataFetcher dataFetcher = DataFetcherFactories.wrapDataFetcher(env.getFieldDataFetcher(),
                        (dataFetchingEnvironment, value) -> {
                            GraphQLServletContext context = dataFetchingEnvironment.getContext();
                            try {
                                String token = context.getHttpServletRequest()
                                        .getHeader("Authorization")
                                        .replaceFirst("Bearer ", "");
                                jwtTokenService.validateToken(token);
                            } catch (Exception e) {
                                throw new BadCredentialsException(e.getMessage());
                            }
                            return value;
                        });
                return env.setFieldDataFetcher(dataFetcher);
            }
        });
    }
}
```

最主要是要透過 `GraphQLServletContext` 拿到 Request 的 Header，才能拿 Token 去驗證合法性，其他如果要設定詳細權限也可以透過參數的方式帶入，這部分的實作就端看每個系統的需求了

---

## 結語

Directive 賦予 GraphQL Schema 更詳細的描述內容，並且提供額外功能，不僅僅讓前端看到 Schema 的時候可以更了解細節，也可以在 GraphQL 這層加上許多特異功能，往後應該會頻繁使用到