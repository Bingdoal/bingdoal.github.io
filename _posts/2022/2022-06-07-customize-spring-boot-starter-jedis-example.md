---
layout: post
title: 抽離出共用設定，客製化自己的 spring-boot-starter，以 Jedis 為例
date: 2022-06-07 10:42:38 +0800
category: backend
img: cover/spring-boot.png
description: 撰寫程式的時候常常會有一些部分是重複的，這時候不想要重寫一遍 code 或是複製貼上的話，就需要把共用的部分抽離出來，而 spring boot 本身其實就是一個這樣的懶人包的概念，透過各種 starter 幫忙將整個系統架構好，且只需要引入就可以作用，這篇的目的就是來撰寫一個自己的 starter
lang: zh-TW
tags: [spring boot]
published: true
---

{{page.description}}

## Starter

SpringBoot 專案中時常會用到各種各樣的 Starter 像是 `spring-boot-starter-data-jpa`, `spring-boot-starter-validation`, `spring-boot-starter-web` 都是常用到的 starter

而 Starter 的作用其實很簡單，每次 SpringBoot 在啟動的時候會初始化各種實例並進行注入，而 Starter 就是在初始化階段的時候進行各種實例建立，除此之外因為要用到 Starter 就必須加入依賴，所以也可以用來做依賴的集合，簡言之功能如下：

1. 建立初始化實例
2. 依賴集合

## 實作

在微服務架構下我們有很多服務都需要用到 redis 操作，如果每一個服務都要重寫一遍一樣的設定也是蠻麻煩的，於是就來撰寫一個共用的 starter 吧

### pom.xml

首先還是要建立一個基本的 pom.xml，以下的依賴除了 redis 以外對於 starter 來說都是必要的，特別要注意的是專案的 `artifactId` 命名是有規定的，必須是以 `<project-name>-spring-boot-starter` 的格式進行命名

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.6</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>common-spring-boot-starter</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <dependencies>
            <!--starter-basic-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-configuration-processor</artifactId>
                <optional>true</optional>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-autoconfigure</artifactId>
            </dependency>
            <!--redis-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-data-redis</artifactId>
                <exclusions>
                    <exclusion>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-starter-logging</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
            <dependency>
                <groupId>redis.clients</groupId>
                <artifactId>jedis</artifactId>
            </dependency>
    </dependencies>
</project>
```

### Redis config

一般我們在使用 redis 的時候可以引用 `spring-boot-starter-data-redis` 然後直接在 `application.yaml` 寫好設定就會自動注入了，這裡我們要結合 `jedis` 來做連線池的管理，看到下面完整的 config 範例：

```java
@EnableConfigurationProperties({CommonRedisProperties.class})
@Configuration
@ConditionalOnProperty(prefix = "common.redis", value = "enabled", havingValue = "true")
public class CommonRedisConfig {
    @Autowired
    private CommonRedisProperties redisProperties;

    @Bean
    public CommonRedisService commonRedisService() {
        return new CommonRedisService(getCommonJedisPool());
    }

    private JedisPool getCommonJedisPool() {
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setTestOnBorrow(true);
        poolConfig.setMaxTotal(redisProperties.getMaxActive());
        poolConfig.setMaxIdle(redisProperties.getMaxIdle());
        poolConfig.setMinIdle(redisProperties.getMinIdle());
        poolConfig.setMaxWait(Duration.ofSeconds(redisProperties.getMaxWait()));
        poolConfig.setTestWhileIdle(true);
        return new JedisPool(poolConfig,
                redisProperties.getHost(),
                redisProperties.getPort(),
                redisProperties.getTimeout(),
                redisProperties.getUsername(),
                redisProperties.getPassword(),
                redisProperties.getDatabase());
    }
}
```

其中的 `redisProperties` 就是由 `@ConfigurationProperties` 從 `application.yaml` 讀進來的

也可以看到這邊我們轉寫了一個自用的 `CommonRedisService`，方便對於 `JedisPool` 去包裝，不直接把 `JedisPool` 註冊為 `@Bean` 有個原因也是想要避免衝突，如果引用的專案已經有建立過了可以避開，並且直接使用包裝好的 `Service` 也比較好做一些修改

`ConditionalOnProperty` 則是用來做一個設定上的判斷，只有在符合條件的時候，這個 `Configuration` 才會被載入

看一下 `CommonRedisService` 的內容如下：

```java
public class CommonRedisService {
    private final JedisPool jedisPool;

    public CommonRedisService(JedisPool jedisPool) {
        this.jedisPool = jedisPool;
    }

    public Set<String> getAllKey() {
        Jedis jedis = jedisPool.getResource();
        Set<String> set = jedis.keys("*");
        jedis.close();
        return set;
    }

    public String set(String key, String value) {
        Jedis jedis = jedisPool.getResource();
        String set = jedis.set(key, value);// return OK
        jedis.close();
        return set;
    }

    public boolean setTime(String key, long second) {
        Jedis jedis = jedisPool.getResource();
        Long success = jedis.expire(key, second);
        jedis.close();
        return success == 1;
    }

    public String get(String key) {
        Jedis jedis = jedisPool.getResource();
        String value = jedis.get(key);
        jedis.close();
        return value;
    }

    public Long delete(String key) {
        Jedis jedis = jedisPool.getResource();
        Long quantity = jedis.del(key);
        jedis.close();
        return quantity;
    }

    public long getTime(String key) {
        Jedis jedis = jedisPool.getResource();
        Long time = jedis.ttl(key);// -1為永遠有效 -2為過期
        jedis.close();
        return time;
    }

    public Long setKeyAdd(String key) {
        Jedis jedis = jedisPool.getResource();
        Long value = jedis.incr(key);
        jedis.close();
        return value;
    }

    public Long setKeySub(String key) {
        Jedis jedis = jedisPool.getResource();
        Long value = jedis.decr(key);
        jedis.close();
        return value;
    }

    public Long setKeyAddValue(String key, Long value) {
        Jedis jedis = jedisPool.getResource();
        Long newValue = jedis.incrBy(key, value);
        jedis.close();
        return newValue;
    }

    public Long setKeySubValue(String key, Long value) {
        Jedis jedis = jedisPool.getResource();
        Long newValue = jedis.decrBy(key, value);
        jedis.close();
        return newValue;
    }

    public List<String> getRange(String key) {
        Jedis jedis = jedisPool.getResource();
        List<String> values = jedis.lrange(key, 0, -1);
        jedis.close();
        return values;
    }

    public void setRange(String key, long expire, String... values) {
        Jedis jedis = jedisPool.getResource();
        jedis.del(key);
        jedis.lpush(key, values);
        jedis.expire(key, expire);
        jedis.close();
    }
}
```

在 starter 的 service 沒辦法像一般的 spring boot 專案那樣使用 `@Autowired` 來注入依賴，需要手工寫建構子來注入，因為除了指定的 `Configuration` 之外，其他的類別是不會被 spring boot 載入的

+ 而如何指定要載入的 `Configuration` 呢？

必須要在 `resources/META-INF` 路徑下建立一個檔案 `spring.factories` 加入下面內容：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    com.moneymanagement.common.config.CommonConfig, \
    com.moneymanagement.common.config.CommonRedisConfig
```

好，這樣一切都準備就緒了

接下來我們需要把 starter 上到一個 maven repo 以供其他專案引用，這部分可以參考[之前的文章](https://bingdoal.github.io/deploy/2021/01/build-maven-repo-on-github/)

### 引用

接下來在其他專案中的 pom.xml 只要加入:

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>common-spring-boot-starter</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

然後記得在 `application.yaml` 加入設定：

```yaml
common:
  redis:
    enabled: true
    database: 0
    username:
    password:
    timeout: 100000
    host: 127.0.0.1
    port: 6379
    default_expire: 600
    jedis:
      pool:
        max-active: 1000
        max-idle: 8
        max-wait: 1000
        min-idle: 0
```

這些設定會被你的 starter 讀入用來建立 `CommonRedisService`

之後只要在需要的地方加入:

```java
@Autowired
CommonRedisService redisService;
```

就可以直接使用囉

---

## 結語

starter 的建立比想像中還要容易，可以很簡單的整合想要集中的設定，不過部署部分可能還是需要有一個自己用的 maven repo 比較方便
