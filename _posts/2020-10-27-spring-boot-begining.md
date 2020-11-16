---
layout: post
title:  "Spring Boot 初戰"
date:   2020-10-27 15:00:13 +0800
category: backend
img: cover/spring-boot.jpg
description: spring boot 初學，快速掌握 Spring 的基本 MVC 架構
lang: zh-TW
tags: [java, spring boot]
---
# Spring Boot 初學用法介紹

Spring 框架下透過各種 annotation 來定義各種設定，下面會介紹常用的幾種功能  
## 進入點
```java
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```
+ Spring 的進入點就如上面這段 code  
+ 其中 `@SpringBootApplication` 這個 annotation 是一堆 annotation 的綜合體如下

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters={@Filter(type=CUSTOM, classes={TypeExcludeFilter.class}), @Filter(type=CUSTOM, classes={AutoConfigurationExcludeFilter.class})})
@Target(value={TYPE})
@Retention(value=RUNTIME)
@Documented
@Inherited
```

+ 要注意的是，所有的 Controller、Model、Servivce 等等 Spring 相關的 annotion 都只能使用在與進入點同目錄或子目錄下，才能被 Spring 給掃描到。

而如果有特別理由需要放不同目錄下的話可以參考下面作法，命令其額外去掃描其他目錄

```java
@SpringBootApplication(
    scanBasePackages={
        "bingdoal.controller",
        "bingdoal.model"
    }
)
```

## Controller
+ Spring 的路由是由 Controller 決定的，下面來了解如何建置 Controller  

```java
@Controller
class User{
    ...
}
@RestController
class User{
    ...
}
```

+ `@Controller` 和 `@RestController` 都可以用來指定 class 為 Controller，區別為是否包含 View 的回傳，單純的資料交換的話使用 `@RestController` 就好了  

+ 用 annotation 的好處是，只要在任意的 class 開頭定義好，Spring 啟動後就會自動去找，不需要一個一個 import 也不用為了路徑去改變專案結構

### RequestMapping

+ `@RequestMapping` 這個 annotation 決定詳細的路徑以及 header、body、method 等等訊息在內，主要的 request 設定其實都仰賴它  

+ 而 Spring 後來又推出以 method 分類的 annotation 包含:  
    + `@GetMapping`
    + `@PostMapping`
    + `@DeleteMapping`
    + `@PutMapping`
    + `@PatchMapping`

```java
@RequestMapping(value = "/user", method=RequestMethod.GET)
public User getUser(@RequestParam(value="id",defaultValue="0") int id){
    return userDAO.findAUser(id);
}
```

+ 上面的 `@RequestMapping` 就等價於 `@GetMapping`，`value`/`path` 定義了路徑，`method` 定義方法，也可以額外定義 `header`  

+ `@RequestParam` 則是參數傳遞的方法，下面會一併提到

#### 參數傳遞
+ request 的參數取得大致透過幾種方式其對應的 annotation 如下
    + url: `/user/{id}` => `@PathVariable("id")`
    + query: `/user?id=` => `@RequestParam("id")`
    + jsonBody => `@RequestBody`
    + formData => `@RequestPart("name")`
  
```java
@GetMapping(value = "/{id}")
public Object getUser(@PathVariable("id") int id) {
    if (id > 0 && id <= users.length) {
        return memberService.findById(id);
    } else {
        return null;
    }
}
```

```java
@GetMapping("")
    public Object getMember(@RequestParam(value = "id", defaultValue = "0") int id) {
        if (id <= 0) {
            return memberService.findAll();
        } else {
            return memberService.findById(id);
        }
    }
```

#### 根路由用法
這邊特別講一下路由的設定，`@RequestMapping` 的 annotation 比較特別，不只可以用在 method 上來設定觸發的功能，也可以宣告在 class 上用來定義根路由，例如:

```java
@RestController
@RequestMapping("/user")
public class MemberController {

    @GetMapping("")
    public String getMember() {
        return "all member";
    }
    
    @GetMapping("/info")
    public String getMemberInfo(){
        return "member info";
    }
}
```
這樣設定下，在 MemberController 中的路由都會從 `/user` 開始，藉由這種方式達到封裝路由的效果，而 `@GetMapping` 這類的 annotation 則沒有這種用法，各個方法的 mapping 都只能用在 method 的宣告上

## View
要用到畫面之前要先引入官方推薦的模板引擎 thymeleaf

```xml
<dependency>         
    <groupId>org.springframework.boot</groupId>            
    <artifactId>spring-boot-starter-thymeleaf</artifactId>      
</dependency>
```
相關設定:

```
spring.thymeleaf.prefix=classpath:/templates/
spring.resources.static-locations=classpath:/static/
spring.thymeleaf.cache=false
spring.resources.cache-period=0
```
然後在專案結構的 `resources` 資料夾底下創建 `templates` 和 `static` 兩個資料夾，templates 用來放 html，static 則是放 css 與 js 等等靜態資源。

code:

```java
@Controller
@RequestMapping("/view")
public class ViewController {
    @GetMapping("")
    public String index(){
        return "index";
    }
}
```
然後回傳的值會去尋找在 templates 裡有同樣檔名的 html。

要注意 html 的開頭需要宣告  `<html xmlns:th="http://www.thymeleaf.org">` 才能使用 thymeleaf 的特殊語法，像是引用資源

```html
<script th:src="@{/js/vue.js}"></script>
<link rel="stylesheet" th:href="@{/css/bootstrap.min.css}">
```

或是從後端直接傳值的方法

```java
@Controller
@RequestMapping("/view")
public class ViewController {
    @GetMapping("")
    public String index(HttpServletRequest request, HttpSession session){
        request.setAttribute("test","dasdasd");
        session.setAttribute("test","sadasdasdsadsd");
        return "index";
    }
}
```

```html
<h1>hello <p th:text="${test}"></p></h1>
<h1>hello <p th:text="${session.test}"></p></h1>
```

其實還有蠻多操作的，詳細請看下面連結  
[thymeleaf 一些基本教學](https://zhuanlan.zhihu.com/p/103089477){:target='_blank'}

## Model、Repository、Service
+ 這一部分有蠻多的東西，這邊只能淺談一下，還有很多東西待研究

相關設定:

```
spring.jpa.hibernate.ddl-auto=none
spring.datasource.initialization-mode=always
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/{databasename}
spring.datasource.username= {username}
spring.datasource.password= {passsword}
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
```
這邊我用的是 PostgreSql，就請代入自己使用的 DB 和其他設置
如果想看 JPA 最後組合出的 SQL 語法也可以加入:

```
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

### Model

```java
@Entity
@Table(name = "member")
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    @Column(name = "name")
    private String name;
    
    public Member() {

    }
    ...

}
```

+ 重點一樣放在幾個 annotation，大致上就如同字面意思，應該不難理解，如果變數名稱與實際的 column name 相同，則不需要設定 `@Column` 這個 annotation。

+ 這邊要提到的是 `@GeneratedValue(strategy = GenerationType.IDENTITY)` 這行，指的是 id 的產生方法，網路上大部分教學都是 `GenerationType.AUTO` 但這邊範例使用的是 PostgreSql，id 的產生方法上與一般 MySql 不太一樣，所以設定的值也不相同  
  
### Repository
Spring 這邊多了一層 Repository 用來定義 model 的方法，下面有三個介面裡面有實作一些基本的方法
+ `@CrudRepository` => 提供基本 CRUD 的方法
+ `@PagingAndSortingRepository` => 繼承上面，並加入分頁與排序
+ `@JpaRepository` => 繼承上面，並加入更多額外功能，例如批次操作

使用上 `@JpaRepository` 最齊全，但事實上用不到這麼多功能，反而會造成系統負擔，所以應該根據情境使用 `@CrudRepository` 和 `@PagingAndSortingRepository`

```java
@Repository
public interface MemberRepo extends CrudRepository<Member, Integer> {}
```

+ 泛型中，第一個參數是 model，第二個則是 ID 的類型
+ 而繼承了 `@CrudRepository` 就已經提供了基本的 `find`、`save`、`delete` 等功能了

在撰寫自己的客製方法上，這邊 Spring 的設計就十分有趣了，他會根據你下的關鍵字自動幫你產生方法，例如:

```java
@Repository
public interface MemberRepo extends JpaRepository<Member, Integer> {
    Member findByName(String name);
    Member findByNameLike(String name);
}
```

而如果想要撰寫更複雜的 query 的話，也有能完全自訂義的做法，就是透過 `@Query` 這個 annotation，在裡面撰寫 JPQL 或者原生的 SQL 語法

這邊建議是寫 JPQL，這樣如果系統有更換資料庫的需求，才可以一併通用

```java
@Query("SELECT m FROM Member m WHERE m.name=:name")
Member findByName(String name);
```
這一段就是自訂義了 findByName 這個方法，JPQL 是針對 JPA 的實體查詢，所以 FROM 後面放的不是 table name 而是 Entity name，預設則是 class 的名稱，`:name` 則是會替換成下面傳入的參數 `name`，也可以寫成:

```java
@Query("SELECT m FROM Member m WHERE m.name=?1")
Member findByName(String name);
```
參數則會按照數字順序代入  

參考資料:  
[詳細擴充方法撰寫](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods){:target='_blank'}  
[JPQL 簡介](https://openhome.cc/Gossip/EJB3Gossip/JPQLABC.html){:target='_blank'}  
[Spring Data JPA 自訂更新/刪除](https://blog.jren.cc/2020/03/11/spring-data-jpa-define-update-and-remove/){:target='_blank'}  

### Service

```java
@Service
public class MemberService{
    @Autowired
    private MemberRepo memberRepo;

    public List<Member> findAll() {
        return (List<Member>) memberRepo.findAll();
    }

    public Member findById(int id) {
        return memberRepo.findById(id).get();
    }

    public String insert(Member member) {
        if (!memberRepo.existsById(member.getId())) {
            memberRepo.save(member);
            return "insert success";
        }
        return "insert failed";
    }

    public String update(Member member) {
        if (memberRepo.existsById(member.getId())) {
            memberRepo.save(member);
            return "update success";
        }
        return "update failed";
    }

    public void delete(Member member) {
        memberRepo.deleteById(member.getId());
    }
}
```
+ 這邊寫到 Service 的部分，開頭一定也是要寫個 `@Service` annotation，宣告 `@Autowired` 的 annotation 可以讓 Spring 自動注入 Repository 的資源

+ 基本上看到這邊也有概念了，Spring 不依賴原生方法去匯入資源以及創造實體，而是透過各種 annotation 去自動產生與掃描

在 Controller 之中使用也是如此:  

```java
@RestController
@RequestMapping("/user")
public class MemberController {

    @Autowired
    private MemberService memberService;

    @GetMapping("")
    public Object getMember(@RequestParam(value = "id", defaultValue = "0") int id) {
        if (id <= 0) {
            return memberService.findAll();
        } else {
            return memberService.findById(id);
        }
    }
    
    @PostMapping("")
    public String insert(@RequestBody Member member) {
        return memberService.insert(member);
    }

    @PutMapping("")
    public String update(@RequestBody Member member) {
        return memberService.update(member);
    }

    @DeleteMapping("")
    public void delete(@RequestBody Member member) {
        memberService.delete(member);
    }
}
```

透過 `@Autowired` 資源會自動注入，而不需要自己創建實體
### 關聯查詢
這邊的邏輯有點混亂，要再多加研究跟實際運用
#### OneToOne
+ 被關聯的 model 要在`@OneToOne` 的 annotation 中寫上 mappedBy 而後面的值是放入關聯 model 的**變數名稱**

```java
@Entity
@Table(name = "member")
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String name;

    @OneToOne(mappedBy = "member")
    private MemberInfo memberInfo;
    public Member() {

    }
    ...

}
```

+ 在關聯的 model 中呢，除了要宣告 `@OneToOne`裡面需要設定存取權限 `cascade` 並且設定關聯的對象，這邊要注意一點，被設定在 `@JoinColumn` 中的關聯欄位，不能在 model 中再次設定變數，否則會報錯

```java
@Entity
@Table(name = "member_info")
public class MemberInfo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    // private int member_id  <= 不能宣告已經設定關聯的變數
    private String phone;
    private String address;
    private String email;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "member_id", referencedColumnName = "id", nullable = false)
    private Member member;

    public MemberInfo() {
    }
    ...
}
```
#### OneToMany
邏輯基本上與 `OneToOne` 大同小異

```java
@Entity
@Table(name = "member")
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String name;

    @OneToMany(mappedBy = "member")
    private List<ShopCard> shopCards;
    
    public Member() {

    }
    ...
}
```

```java
@Entity
@Table(name = "shop_card")
public class ShopCard {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String item;

    @ManyToOne()
    @JoinColumn(name = "member_id", referencedColumnName = "id", nullable = false)
    private Member member;

    public ShopCard() {

    }
    ...
}

```

spring boot 實際上還有很多操作需要研究，這邊就簡單說明到這裡，之後再多針對單一主題去做說明跟深入  