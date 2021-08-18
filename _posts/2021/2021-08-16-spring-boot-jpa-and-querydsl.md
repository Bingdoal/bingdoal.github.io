---
layout: post
title: "Spring boot JPA 的擴充函式庫 QueryDSL，輕鬆處理 GET filter"
date: 2021-08-16 14:55:14 +0800
category: backend
img: cover/spring-boot.jpg
description: spring boot 提供了強大的 JPA 操作，用簡單的宣告就可以組合出大多的查詢語法，但有些較複雜的操作還是得寫 JPQL 的語法，雖然也是很便利強大，但還是希望能有更模組化的方法，QueryDSL 就是用來解決這個問題的
lang: zh-TW
tags: [java, spring boot]
---

{{page.description}}

# 前言

工作使用 Spring boot 也一段時間了，完全能感受到這個生態系的強大，幾乎所有想到的功能都有很好的現有解決方案，這次會用到 QueryDSL 想要解決的問題最主要是對於每個 Get 的 filter 功能，由於可能會有多種不同的資料需要作 filter，若是手動去撰寫每個接口要耗費的工也是不小，而透過 QueryDSL 的神奇方法，只要透過一點設定就可以自動套用 filter 了

不過 QueryDSL 可不只是用來作 filter 方便而已，QueryDSL 最主要是以模組化的方式來組裝客製的 query，下面就開始介紹吧

## JPA

那在介紹 QueryDSL 之前來簡單複習一下 JPA 的作法
```java
@Repository
public interface UserDao extends JpaRepository<User, Long> {
    User findAll();

    @Query("SELECT m FROM User m WHERE m.name=:name")
    User findByName(String name);
}
```

JPA 除了可以用 method 的命名來生成想要的 query 外，也可以透過 `@Query` 來撰寫 JPQL 的語法來寫客製的查詢，其實這兩種寫法基本上已經涵蓋了所有的查詢了

不過用 method 的命名沒辦法作到太複雜的操作，JPQL 語法又不好維護，因此才會需要用到 QueryDSL

## QueryDSL

### 安裝

要使用之前要加入 dependency:
```xml
<dependencies>
    <!-- QueryDSL 函式庫 -->
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-apt</artifactId>
        <scope>provided</scope>
    </dependency>
</dependencies>

<!-- 自動生成 Query Entity -->
<build>
    <plugins>
        <plugin>
            <groupId>com.mysema.maven</groupId>
            <artifactId>apt-maven-plugin</artifactId>
            <version>1.1.3</version>
            <executions>
                <execution>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>process</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>.apt_generated</outputDirectory>
                        <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

QueryDSL 的操作都會建立在由 `apt-maven-plugin` 生成的 Query Entity 物件來進行，舉個例子:

像一般操作一樣先建立一個 Entity
```java
@Entity
@Data
public class User{
    private Long id;
    private String name;
    private String password;
    private String description;
}
```

然後透過指令 `mvn clean compile` plugin 會自動偵測 `@Entity` 並且在 `.apt_generated/` 產生 `QUser` 的類別內容如下，是用來幫助 QueryDSL 操作的

```java
@Generated("com.querydsl.codegen.EntitySerializer")
public class QUser extends EntityPathBase<User> {
    private static final long serialVersionUID = 309447108L;
    public static final QUser user = new QUser("user");
    public final QEntityBase _super = new QEntityBase(this);
    public final StringPath description = createString("description");
    //inherited
    public final NumberPath<Long> id = _super.id;
    public final StringPath name = createString("name");
    public final StringPath password = createString("password");
    public QUser(String variable) {
        super(User.class, forVariable(variable));
    }
    public QUser(Path<? extends User> path) {
        super(path.getType(), path.getMetadata());
    }
    public QUser(PathMetadata metadata) {
        super(User.class, metadata);
    }
}
```

### QueryDSL 操作

在進行操作之前要先註冊一個物件 `JPAQueryFactory` 這是用來幫助組裝 query 語句的工具，需在 Config 下註冊成 Bean 讓 spring boot 幫我們注入 `EntityManager`:

```java
@Bean
JPAQueryFactory jpaQueryFactory(EntityManager entityManager) {
    return new JPAQueryFactory(entityManager);
}
```

使用上的就像下面寫的，透過提供的介面可以自由組裝想要的語句，並且是透過物件跟方法去組裝，維護跟修改上都會比較方便快速

```java
@Service
public class UserDaoService {
    @Autowired
    private JPAQueryFactory queryFactory;

    private User findBy(String name, String password) {
        return queryFactory
                .selectFrom(QUser.user)
                .where(
                        QUser.user.name.eq(name).eq(QUser.user.password.eq(password))
                ).fetchFirst();
    }
}
```

### 自動化產生 filter
這部分其實才是我使用 QueryDSL 的重點😂

直接來看到實作吧，原本的 Dao 只需要繼承 `JpaRepository`，而為了使用 QueryDSL `Predicate` 提供的強大 filter 功能，需要額外繼承 `QuerydslPredicateExecutor` 和 `QuerydslBinderCustomizer`，然後去 override `customize` 這個方法，用來定義那些屬性希望可以用作 filter 的屬性

```java
public interface UserDao extends JpaRepository<User, Long>,
QuerydslPredicateExecutor<User>, QuerydslBinderCustomizer<QUser> {
    @Override
    default void customize(final QuerydslBindings bindings, final QUser qEntity) {
        bindings.excludeUnlistedProperties(true);
        // 自訂需要 filter 的屬性
        bindings.including(qEntity.name, qEntity.description);
        bindings.bind(String.class).first((SingleValueBinding<StringPath, String>) StringExpression::containsIgnoreCase);
    }
}
```

接著在使用上只需要在 API 的承接參數加上 `@QuerydslPredicate(root = User.class)` 就會自動轉換成 `Predicate` 類別帶入到 `findAll` 中

```java
@RestController()
@RequestMapping("/v1/user")
public class UserController {

    @Autowired
    private UserDao userDao;

    @GetMapping("")
    @ResponseStatus(HttpStatus.OK)
    public Page<User> getAllUserPage(@QuerydslPredicate(root = User.class) Predicate predicate,
                                     final Pageable pageable) {
        return userDao.findAll(predicate, pageable);
    }
}
```

外部呼叫 API 的時候只要帶在 GET 的 Url 之後就會自動產生 filter 後的結果了:
```bash
$ curl --request GET http://localhost:8080/v1/user?name=test
```

上面的 Controller 寫法還順便加入了 `Pageable` 的用法:
```bash
$ curl --request GET http://localhost:8080/v1/user?name=test&page=1&size=10
```

# 結語

能夠自動組建 filter 的語句真的十分強大，以往都要手動去解析 filter 的參數，這真的是福音，讓後端撰寫上更加快速且易於維護，能夠用類別方法去組件查詢語句也是很強大的功能，能夠處理複雜操作卻又不會不易維護