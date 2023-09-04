---
layout: post
title: "Spring boot JPA 讀寫分離"
date: 2023-08-09 11:14:59 +0800
category: backend
img: cover/spring-boot.png
description: "之前寫過 JPA 連接多資料庫來源的做法，這次要來寫的是讀寫分離的設定，實際上也可以透過多資料庫來源的做法來操作，不過使用上會變成讀與寫需要使用不同的類別，理想上是希望可以只套用到方法，這樣可以都用同一個 DAO 來操作，根據需求不同來決定是否存取主資料庫，下面也附上之前的文章可以先參考看看"
lang: zh-TW
tags: [spring boot, jpa]
published: true
---

{{page.description}}

[Spring boot JPA 下套用多個資料庫來源](https://bingdoal.github.io/backend/2022/09/spring-boot-jpa-multiple-datasource/)

## 實作一

如同前言所說希望可以只套用方法，在我需要的方法去使用，那最理想的方式就是透過 `annotation` 去標記，為此我們要先做點設定準備

yaml 設定的部分就像之前的多資料庫來源方式一樣，多一層去分開不同的資料來源，下面是 Configuration

```java
@EnableTransactionManagement
@Configuration
public class DataSourceConfig {
    @Bean("master")
    @ConfigurationProperties(prefix = "spring.datasource.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().build();
    }


    @Bean("slave")
    @ConfigurationProperties(prefix = "spring.datasource.slave")
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean("routingDataSource")
    public DataSource dataSource() {
        Map<Object, Object> map = new HashMap<>();
        map.put("master", masterDataSource());
        map.put("slave", slaveDataSource());

        AbstractRoutingDataSource routing = new AbstractRoutingDataSource() {
            @Override
            protected Object determineCurrentLookupKey() {
                return TransactionSynchronizationManager.isCurrentTransactionReadOnly()
                    ? "slave" : "master";
            }
        };
        routing.setTargetDataSources(map);
        routing.setDefaultTargetDataSource(map.get("master"));
        return routing;
    }
}

```

spring boot 有提供 `AbstractRoutingDataSource` 的介面來幫助我們做資料源的切換，這裡是去抓 `@Transactional` 的 `readOnly` 狀態來判斷，也可以用其他方式替代，只要可以控管

使用上如下

```java
class UserDao extends JpaRepository<User, Long> {
    @Transactional(readOnly=true)
    List<User> findAllBySlave();
}
```

這樣在調用這個方法的時候就會去抓 slave 的資料庫

不過因為 `@Transactional` 本身還包含有其他功能，所以後來我們轉為用其他方式來做資料源的切換

## 實作二

因為不想讓 `@Transactional` 有其他副作用，所以後來決定用自己定義的 `annotation` 來實做這件事，不過要寫的 code 會稍微多一點

先定義 `annotation`，以及資料源的型態

```java
@Inherited
@Documented
@Retention(value = RetentionPolicy.RUNTIME)
@Target(value = ElementType.METHOD)
public @interface DataSourceType {

  DataSourceKey value() default DataSourceKey.MASTER;
}
```

```java
public enum DataSourceKey {
  MASTER, SLAVE
}
```

另外還要處理切換資料源的地方，這邊需要用到 AOP 以及 ThreadLocal 的幫助

```java
public class RoutingDataSourceContext implements AutoCloseable {

  private static final ThreadLocal<DataSourceKey> contextHolder = new ThreadLocal<>();

  public RoutingDataSourceContext(DataSourceKey routingType) {
    setRoutingType(routingType);
  }

  public static void setRoutingType(DataSourceKey routingType) {
    contextHolder.set(Objects.requireNonNullElse(routingType, DataSourceKey.MASTER));
  }

  public static DataSourceKey getRoutingType() {
    return contextHolder.get();
  }

  public static void clear() {
    contextHolder.remove();
  }

  @Override
  public void close() throws Exception {
    clear();
  }
}
```

```java
@Aspect
@Slf4j
@Component
public class DynamicDataSourceAspect {

  @Pointcut("execution(* com.demo.dao.*.*(..)) ")
  public void pointCut() {

  }

  @Before("pointCut()")
  public void switchDataSource(JoinPoint point) {
    MethodSignature signature = (MethodSignature) point.getSignature();
    Method method = signature.getMethod();
    DataSourceType dbSource = method.getAnnotation(DataSourceType.class);
    if (dbSource != null) {
      RoutingDataSourceContext.setRoutingType(dbSource.value());
    } else {
      RoutingDataSourceContext.setRoutingType(DataSourceKey.MASTER);
    }
  }
}
```

Configuration 則改成下面這樣

```java
@EnableTransactionManagement
@Configuration
public class DataSourceConfig {

    ...

    @Primary
    @Bean("routingDataSource")
    public DataSource dataSource() {
        Map<Object, Object> map = new HashMap<>();
        map.put(DataSourceKey.MASTER, masterDataSource());
        map.put(DataSourceKey.SLAVE, slaveDataSource());

        AbstractRoutingDataSource routing = new AbstractRoutingDataSource() {
            @Override
            protected Object determineCurrentLookupKey() {
                return RoutingDataSourceContext.getRoutingType();
            }
        };
        routing.setTargetDataSources(map);
        routing.setDefaultTargetDataSource(map.get(DataSourceKey.MASTER));
        return routing;
    }
}
```

用到 ThreadLocal 可以確保每個請求的資料源控制是分開的，並用 AOP 來動態判斷當前方法的 DataSourceType，來做到方法的動態切換，也不會跟 `@Transactional` 混用，目前用下來的體驗還蠻理想的
