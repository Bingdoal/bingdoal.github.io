---
layout: post
title: "GraphQL in Spring boot(一) 基本 Query 操作"
date: 2021-12-30 12:09:25 +0800
category: backend
img: cover/graphql-in-spring.png
description: "GraphQL 原先是由 Facebook 內部開發，後來開放出來給大家一起使用的一種標準，目的是更簡潔有規劃性的操作 Query，跟 RESTful API 比較起來，目前感受到最大的優點是可以讓前端更有彈性的去選擇資料，不過不確定是不是會犧牲一點效能，可能需要更深入使用跟分析後才能得知，本篇先介紹最基本的 Query 之後會再慢慢新增其他的內容"
lang: zh-TW
tags: [backend,spring boot,graphql]
published: true
---

{{page.description}}

本系列會著重在 GraphQL 在 Spring boot 上的實作，所以對 GraphQL 的介紹只會簡單帶過

## 簡介

GraphQL 是一種類似於 SQL 的語法，透過 Server 端預先定義好 Schema 讓 Client 端送出需求的 Field 以及參數然後返還資料給 Client 的互動模式

互動上像是不同型態的 RESTful API，而目前多數的實作方式也是透過 RESTful API Post 然後在 Request body 中帶 GraphQL 的語法

主要看點在於高彈性的 Field 選擇，把資料的選擇權交給前端

下面是一個簡單的範例，左邊是輸入的語法，右邊則是回傳

![]({{site.baseurl}}/assets/img/graphql-hello-world.png)

複雜一點像這樣，可以任意挑選需要的欄位來 query

![]({{site.baseurl}}/assets/img/graphql-get-user-all.png)

![]({{site.baseurl}}/assets/img/graphql-get-user-ex-email.png)

語法上有點像 SQL 的 SELECT 但又更簡潔好懂

### 設定

#### 依賴引用
```xml
<dependency>
    <groupId>com.graphql-java-kickstart</groupId>
    <artifactId>graphql-spring-boot-starter</artifactId>
    <version>12.0.0</version>
</dependency>
```

一個依賴全包好 Spring boot 全家桶就是這麼方便

#### application.yaml
這邊只寫上我用到的基本設定，詳細設定請看 [graphql-java-kickstart github](https://github.com/graphql-java-kickstart/graphql-spring-boot)

```yaml
graphql:
  graphiql:
    enabled: true
  servlet:
    enabled: true
    cors-enabled: true
    exception-handlers-enabled: true
  playground:
    enabled: false
  voyager:
    enabled: true
```

### 基本範例
設定完成後我們就來個基本範例
#### Schema
首先在 `src/main/resources` 下新增 `graphql` 的目錄，然後在裡面新增 `schema.graphqls` 內容如下

```graphql
type User{
    id: ID!
    name:String
    email:String
}
type Query{
    userList:[User!]!
}
```

這邊定義了一個 `User` 的 `Object Type` 就像是一個 class 這樣表達一種資料類型，而 `type Query` 是一種特別的類型，這是用來定義 Query 的 Operation 的，也就是可以執行的語法功能，而不是資料類型，就像是對於 `function` 的定義

`!` 的意思是 `Not Null`，這個會在 GraphQL 這層去檢查，讓前端看到這個 Schema 的時候就可以了解這個屬性有沒有可能是 `Null`

`ID` 是一個特別的基本型態，不過其實沒有實際定義說 `ID` 一定要長怎樣，回傳是 Int 或是 String 都會被接受

#### Query
寫好以上 Schema 後會發現程式還沒有辦法執行，那是因為雖然已經定義了 `userList` 的 Schema 了，但還沒有定義實作，因此我們要在專案中加入以下程式碼

```java
@Component
@Slf4j
public class UserQuery implements GraphQLQueryResolver {
    @Autowired
    private UserDao userDao;

    public List<User> getUserList() {
        return userDao.findAll();
    }
}
```

這邊 Spring boot 會自己去抓與 operation 對應的 method 名稱，這邊就是 `userList` 對應到 `getUserList`

注意要寫上 `@Component` 這樣才會被註冊成 Spring boot 中的 Bean

#### Graphiql
啟動之後我們可以到 [http://localhost:8080/graphiql](http://localhost:8080/graphiql) 下會有一個簡單的 GraphQL 的互動介面，在這邊可以測試你的 GraphQL 有沒有正確運作，回傳是不是自己想要的

![]({{site.baseurl}}/assets/img/graphiql.png)

旁邊點開還有文件可以看，也就是寫好了 Schema 就寫好了文件，這也是 GraphQL 的優點吧

![]({{site.baseurl}}/assets/img/graphiql-schema.png)

不過我自己在開發時候的時候會有遇到沒辦法看到 Graphiql 頁面的情況

+ 事先有設定 Spring boot 的 `WebSecurity` 機制，那可能會被攔下來沒辦法存取，這問題不大，只要修改一下權限就好了

```java
@Override
    protected void configure(final HttpSecurity httpSecurity) throws Exception {
    ...
    httpSecurity.authorizeRequests()
            .antMatchers("/vendor/**").permitAll()
            .antMatchers("/graphiql").permitAll()
            .antMatchers("/graphql").permitAll();
    ...
}
```

+ CSP(Content Security Policy) 政策無法渲染頁面
    這是用來阻擋一些不安全的前端寫法而設置的，而 Graphiql 的 UI 介面可能有一些 Legacy code，導致會發生這個問題

    首先這個設定通常也是在 `WebSecurity` 設定的，那可以直接拿掉就好，另一種方式呢是直接去關掉瀏覽器的 CSP 設定，雖然可以用了但這樣一來瀏覽其他網站就會有安全上的疑慮，所以這邊我建議安裝一個 chrome 的 extension

    [Disable Content-Security-Policy](https://chrome.google.com/webstore/detail/disable-content-security/ieelmcmcagommplceebfedjlakkhpden)

    顧名思義就是用來關掉 CSP 的但是他只會作用在當前的網頁上，並且可以隨時開關十分方便

#### Voyager
根據上面的 `application.yaml` 設定會發現還有一個 Voyager 功能被啟用，可以到 [http://localhost:8080/Voyager](http://localhost:8080/Voyager)

![]({{site.baseurl}}/assets/img/voyager.png)

會看到完整的 GraphQL Schema 關聯圖，一目瞭然非常容易理解，搭配上 Graphiql 基本上已經是很完整的文件跟測試平台了

#### Resolver
上面是 Query 一個簡單的結構，而當結構稍微複雜一點，像是關聯式資料庫常出現的外部關聯

```graphql
type Post{
    id: ID!
    content:String
    author:User!
    creationTime: String!
}

type Query{
    postList:[Post!]!
}
```

一般像是 RESTful 的處理都是直接做好 Join 後全部回傳，而 GraphQL 的宗旨就是，沒用到的就不處理

所以實作方面會變成下面這樣

```java
@Component
@Slf4j
public class PostQuery implements GraphQLQueryResolver {
    @Autowired
    private PostDao postDao;

    public List<Post> getPostList() {
        return postDao.findAll();
    }
}

@Slf4j
@Component
public class PostResolver implements GraphQLResolver<Post> {
    @Autowired
    private UserDao userDao;

    public User getAuthor(Post post) {
        return userDao.findById(post.getAuthorId()).get();
    }
}
```
多了一個 `Resolver` 來處理主要 Query 附帶的其他子 Query，一樣 Spring boot 會根據定義好的 Field name 去抓相應的方法，像這邊就是 `author` 對應 `getAuthor`


 + `Resolver` 也能用在一般的 Field 上，甚至還能帶參數

先在 schema 上加入 field 參數
```graphql
type Post{
    id: ID!
    content:String
    author:User!
    creationTime(loc:String): String!
}
```

然後加入 Resolver
```java
@Slf4j
@Component
public class PostResolver implements GraphQLResolver<Post> {
    ...

    public String creationTime(Post post, String loc) {
        if(loc != null){
            return post.getCreationTime().atZone(ZoneId.of(loc))
                                            .toLocalDateTime().toString();
        } else {
            return post.getCreationTime().toString();
        }
    }
}
```

其實概念上就有點像是 Getter 一樣，只不過要記得處理好沒有帶參數的時候

---
## 結語

目前看下來 GraphQL 這樣的效能表現感覺會蠻差的，不確定是不是 Spring boot 的實作問題，又或是 GraphQL 跟關聯式資料庫的相性不太好，也或許會有相應的解決辦法，等待以後研究，還有如果遇到層層遞迴的結構，那是不是會直接炸掉，很多問題等待釐清，離正式用在產品上可能還有一段路要走