---
layout: post
title: Spring boot JPA 下套用多個資料庫來源
date: 2022-09-12 11:44:58 +0800
category: backend
img: cover/spring-boot.png
description: 最近一個新的專案需要同時動用到兩個不同的資料庫，雖然是同一台機器，但是 JPA 的連線要包含資料庫資訊，也就是說需要可以支援一個以上的資料庫連線才行，這邊就簡單講解下怎麼設定
lang: zh-TW
tags: [spring boot, jpa]
published: true
---

{{page.description}}

## application.yml

以往過去都是透過 application.yml 直接撰寫套用連線設定如下

```yaml
spring:
  datasource:
    host: localhost
    port: 5432
    database: mydb
    password: 123456
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql://${spring.datasource.host}:${spring.datasource.port}/${spring.datasource.database}?useUnicode=true&characterEncoding=utf-8
    username: postgres
    hikari:
      maximum-pool-size: 1000
      minimum-idle: 10
      connection-timeout: 10000
      max-lifetime: 1800000
      idle-timeout: 600000
      auto-commit: true
  jpa:
    database: POSTGRESQL
    show-sql: true
    open-in-view: true
    properties:
      hibernate:
        default_schema: testt
        dialect: org.hibernate.dialect.PostgreSQLDialect
        temp:
          use_jdbc_metadata_defaults: false
```

但是我們現在要切分多個 datasource 那這個結構需要調整一下

```yaml
spring:
  datasource:
    mydb1:
      host: localhost
      port: 5432
      schema: testt
      database: mydb1
      password: 123456
      jdbc-url: jdbc:postgresql://${spring.datasource.host}:${spring.datasource.port}/${spring.datasource.database}?useUnicode=true&characterEncoding=utf-8
    mydb2:
      host: localhost
      port: 5432
      schema: testt
      database: mydb2
      password: 123456
      jdbc-url: jdbc:postgresql://${spring.datasource.host}:${spring.datasource.port}/${spring.datasource.database}?useUnicode=true&characterEncoding=utf-8
```

要注意的是 `url` 要改成 `jdbc-url` 實際的可用屬性可以[參考](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html) `spring.datasource.hikari.` 下的屬性

根據 `spring.datasource` 下一層去切分不同的 datasource 裡面屬性則依照 `spring.datasource.hikari` 去給予，然後加一下 spring 的 configuration，有兩個要加

+ Mydb1Config.java

```java
@EnableTransactionManagement
@EnableJpaRepositories(entityManagerFactoryRef = "mydb1EntityManagerFactory",
    transactionManagerRef = "mydb1TransactionManager",
    basePackages = {"springboot.demo.model.mydb1.dao"})
@Configuration
public class Mydb1Config {

  @Bean("mydb1DataSource")
  @Primary
  @ConfigurationProperties(prefix = "spring.datasource.mydb1")
  public DataSource historyDataSource() {
    return DataSourceBuilder.create().build();
  }

  @Bean("mydb1EntityManagerFactory")
  @Primary
  public LocalContainerEntityManagerFactoryBean historyEntityManagerFactory(
      EntityManagerFactoryBuilder builder,
      @Qualifier("mydb1DataSource") DataSource dataSource
  ) {
    HashMap<String, Object> properties = new HashMap<>();
    properties.put("hibernate.physical_naming_strategy",
        "org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy");
    properties.put("hibernate.implicit_naming_strategy",
        "org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy");

    var bean = builder.dataSource(dataSource)
        .properties(properties)
        .packages("springboot.demo.model.mydb1.entity")
        .persistenceUnit("mydb1").build();

    HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
    vendorAdapter.setShowSql(false);
    bean.setJpaVendorAdapter(vendorAdapter);

    return bean;
  }

  @Bean("mydb1TransactionManager")
  @Primary
  public PlatformTransactionManager historyTransactionManager(
      @Qualifier("mydb1EntityManagerFactory") EntityManagerFactory entityManagerFactory
  ) {
    return new JpaTransactionManager(entityManagerFactory);
  }
}

```

+ Mydb2Config.java

```java
@EnableTransactionManagement
@EnableJpaRepositories(entityManagerFactoryRef = "mydb2EntityManagerFactory",
    transactionManagerRef = "mydb2TransactionManager",
    basePackages = {"springboot.demo.model.mydb2.dao"})
@Configuration
public class Mydb2Config {

  @Bean("mydb2DataSource")
  @ConfigurationProperties(prefix = "spring.datasource.mydb2")
  public DataSource historyDataSource() {
    return DataSourceBuilder.create().build();
  }

  @Bean("mydb2EntityManagerFactory")
  public LocalContainerEntityManagerFactoryBean historyEntityManagerFactory(
      EntityManagerFactoryBuilder builder,
      @Qualifier("mydb2DataSource") DataSource dataSource
  ) {
    HashMap<String, Object> properties = new HashMap<>();
    properties.put("hibernate.physical_naming_strategy",
        "org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy");
    properties.put("hibernate.implicit_naming_strategy",
        "org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy");

    var bean = builder.dataSource(dataSource)
        .properties(properties)
        .packages("springboot.demo.model.mydb2.entity")
        .persistenceUnit("mydb2").build();

    HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
    vendorAdapter.setShowSql(false);
    bean.setJpaVendorAdapter(vendorAdapter);

    return bean;
  }

  @Bean("mydb2TransactionManager")
  public PlatformTransactionManager historyTransactionManager(
      @Qualifier("mydb2EntityManagerFactory") EntityManagerFactory entityManagerFactory
  ) {
    return new JpaTransactionManager(entityManagerFactory);
  }
}

```

其實最重點要注意到的就是，建立各自的 `DataSource` 以及指定各自 `entity` 以及 `dao` 放的 `package` 會根據這裡的設定去進行掃描，然後各種其餘的設定都可以透過 Map 去 put 進去，參數就參考官方使用

特別要注意的是預設這裡的命名策略是完全按照 `entity` 以及 `field` 的名稱而不是 jpa 的預設規則，會去幫忙轉換命名風格成底線命名，因此這裡特別設定的兩個屬性，就是要還原成熟悉的 jpa 使用方式

以及多個 datasource configuration 必須要有一個的 bean 全部都要加上 `@Primary` 

```java
HashMap<String, Object> properties = new HashMap<>();
    properties.put("hibernate.physical_naming_strategy",
        "org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy");
    properties.put("hibernate.implicit_naming_strategy",
        "org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy");
```

這些都搞定之後就跟往常操作沒兩樣囉

