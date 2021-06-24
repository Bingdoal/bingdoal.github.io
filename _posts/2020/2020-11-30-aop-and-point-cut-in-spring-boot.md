---
title: AOP 與 Pointcut 淺談
date: 2020-11-30 09:30:00 +0800
category: backend
img: cover/spring-boot.jpg
description: 用 Spring boot 撰寫後端時，常會有事前檢查、事後日誌的需求，若在每個 Api 中都做一次那就太蠢了，於是這邊介紹一下 Spring boot 中十分方便的 AOP 機制，可以輕鬆達成日誌的統一撰寫，也可以降低程式碼的耦合性
layout: post
tags: [spring boot, java]
---

{{page.description}}

# AOP (Aspect-Oriented Programming)
從字面上的話看不太懂是什麼意思，其實簡單來說是指在一般流程中加入一些關注點，從需要的角度去執行程式。也就是在一般流程中，插入特定的切入點，術語上稱為 Cross-cutting concerns

![]({{site.baseurl}}/assets/img/cross-cutting-concerns.jpg)

看圖應該很容易理解的，就是從一般的程式流程中間橫切進去執行日誌紀錄、安全檢查等等，如果這種作法要按照一般撰寫來實現的話，維護上會是場災難，想想要在每個 Api 的 function 中加入各自的 log 機制、安全檢查，不光是加上這個功能就很麻煩，後續更改維護都很費工夫，會讓人為因素大幅上升，就各方面來說都不是理想的做法

也因此就有依靠框架之力的必要了，AOP 的機制也是為此而生

> 有點像前端框架 lifecycle 概念

## 使用

使用前先在 `pom.xml` 加入

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
</dependencies>
```

## Advice
AOP 橫切入的時機點被稱為 Advice，大致有下面五種:

1. `@Before`: 執行前
2. `@AfterReturning`: 執行後正常回傳
3. `@AfterThrowing`: 執行後拋出例外處理
4. `@After`: 不管執行後的行為如何都會觸發
5. `@Around`: 最全能的 Advice，可以在方法中自由撰寫切點跟執行內容

# Pointcut 表示式
用來指定需要加入 Advice 的方法們，有自己的規則跟語法，源自於 [AspectJ](https://www.eclipse.org/aspectj/) 這個 package 的用法，可用的語法有下面:

<!-- TODO:把 PointCut 細節弄懂 -->
1. `execution`: 直接用來表示要指定的方法，可以用一些表示法指定複數方法
2. `within`: 包含在指定條件下的方法們
3. `this`: 指定某個類別的方法們，如果指定的是 `interface` 則會作用於所有實作這個介面的類別
4. `target`: 與 `this` 很像，詳細差異筆者也還不太了解
5. `args`: 指定方法的參數傳入型態
6. `@target`: 指定有被宣告特定 `annotation` 的物件，套用到此物件的所有方法
7. `@args`: 指定有特定 `annotation` 的傳入參數
8. `@within`: 指定用到 `annotation` 的類別或方法們
9. `@annotation`: 指定用到特定 `annotation` 的方法們

以上的表示法都可以混用，可以加入邏輯運算式 `&&`、`||` 和 `!` 來串起整個表達式，所以上面那些表達式可以想作是回傳 boolean 的 function

> 這邊 `this` 跟 `target` 的差異沒很理解，而且絕大部分情況還是都用 `execution` 來定義
> `this` 跟 `target` 的差異可以看這邊 [Spring AOP target() vs this()](https://stackoverflow.com/questions/11924685/spring-aop-target-vs-this)

表達式的格式如下:
```java
execution(
    modifiers-pattern? // 方法的存取權限，不設定表示所有權限
    ret-type-pattern // 回傳類別
    declaring-type-pattern? // 方法的路徑
    name-pattern(param-pattern) // 方法名稱與傳入的參數類型
    throws-pattern? // throw 的 Exception 類型
)
// ? 表示是可選參數
```

## 舉例一下
```java
// 表示在 TestController 底下所有 public 的方法，並且不限制傳入參數的類型與數量，也不限制回傳，下面兩個寫法是等價的
@Pointcut("execution(public * com.xyz.someapp.control.TestController.*(..))")
@Pointcut("within(public * com.xyz.someapp.control.TestController)")

// 表示在 TestEntity 下，所有 set 開頭的 public 方法
@Pointcut("execution(public * com.xyz.someapp.control.TestEntity.set*(..))")

// 所有 get 開頭無參數的 public 方法
@Pointcut("execution(public * get*())")

//所有 get 跟 set 開頭無參數的 public 方法
@Pointcut("execution(public * get*()) || execution(public * set*())")
```


# 實例用法
```java
@Aspect
@Component
class TestAop{
    @Before("execution(public * com.xyz.someapp.*.*(..))")
    public void logBefore(JoinPoint joinPoint){
        System.out.println("Before " + joinPoint.getSignature().toString() + " execution.");
    }
}
```

記得 class 必須加上 `@Component` 才會被 Spring boot 被註冊為 Bean，這段程式碼會在 `com.xyz.someapp` 下的所有類別的所有 public 方法執行前，印出類別及方法名稱

## `@AfterReturning`
```java
@Aspect
class TestAop{
    @AfterReturning("execution(public * com.xyz.someapp.*.*(..))", returning="returnValue")
    public void logAfterReturning(JoinPoint joinPoint, Object returnValue){
        System.out.println("After " + joinPoint.getSignature().toString() + " execution. Then return " + Objects.toString(returnValue));
    }
}
```
`AfterReturning` 可以利用 `returning` 的參數來指定回傳的名稱

## `@AfterThrowing`
```java
@Aspect
class TestAop{
    @AfterThrowing("execution(public * com.xyz.someapp.*.*(..))", throwing="ex")
    public void logAfterThrowing(JoinPoint joinPoint, Exception ex){
        System.out.println(joinPoint.getSignature().toString() + " throwing: " + ex.toString());
    }
}
```
同樣的 `AfterThrowing` 也可以利用 `throwing` 的參數來指定拋出的例外名稱

## `@Around`
```java
@Aspect
class TestAop{
    @Around("execution(public * com.xyz.someapp.*.*(..))")
    public void around(ProceedingJoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        // Before
        try{
            Object result = joinPoint.proceed(args);
            // AfterReturning
        }catch(Exception ex){
            // AfterThrowing
        }
        // After
    }
}
```
`Around` 比較特別一點，沒有指定特定的時間點，而是可以自己選擇流程控制，如上面程式碼註解所寫的樣子，可以自己定義什麼時候執行，藉由傳入的特別參數 `ProceedingJoinPoint`，可以決定在哪裡拋出例外、取得結果等等，算是萬用的 `Pointcut`

## `@Pointcut`
除了上面的特定的時機點以外，還額外提供了一個 annotation `@Pointcut`，用意是在於集中註冊需要用到 AOP 機制的方法，例如:

```java
@Aspect
class TestAop{
    @Pointcut("execution(public * com.xyz.someapp.*.*(..))")
    public void includeSomeappAllMethod(){}

    @Before("includeSomeappAllMethod()")
    public void logBefore(JoinPoint joinPoint){ ... }

    @AfterReturning("includeSomeappAllMethod()", returning="returnValue")
    public void logAfterReturning(JoinPoint joinPoint, Object returnValue){ ... }

    @AfterThrowing("includeSomeappAllMethod()", throwing="ex")
    public void logAfterThrowing(JoinPoint joinPoint, Exception ex){ ... }

    @Around("includeSomeappAllMethod()")
    public void around(ProceedingJoinPoint joinPoint){ ... }
}
```

這會等價於上面的那些寫法，而且可以統一管理要加入切點的方法，不必每次都去改每個 Pointcut 的表達式