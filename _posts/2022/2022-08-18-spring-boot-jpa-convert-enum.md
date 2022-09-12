---
layout: post
title: Spring boot JPA 解析 Entity 的 Enum 屬性
date: 2022-08-18 12:21:04 +0800
category: backend
img: cover/spring-boot.png
description: 撰寫系統不乏用到 Enum 的場景，但是通常不會希望直接存 Enum 的名稱在資料庫中，雖然可能可以增加資料庫的可讀性，但一方面也增加儲存空間的浪費，一方面可能需要儲存的是值，因此簡單紀錄下 JPA 在這部分的轉換是怎麼做的
lang: zh-TW
tags: [spring boot, jpa]
published: true
---

{{page.description}}

## 範例場景

+ 給定一個假設的場景是用在電商的訂單記錄

```sql
CREATE TABLE "order" (
    "id" bigserial NOT NULL PRIMARY KEY,
    "user_id" bigint NOT NULL,
    "item" character varying(255) NOT NULL,
    "status" int NOT NULL,
    "creation_time" timestamp with time zone,
    "modification_time" timestamp with time zone
);

COMMENT ON COLUMN "order"."status" is '1:訂單成功 2:訂單處理中 3:訂單處理完成 -1:訂單取消';
```

+ 那在 Spring boot 中我們的 Entity 會寫成

```java
@Entity
@data
public class Order{
    private Long id;
    private Long userId;
    private String item;
    private OrderStatus status;
}

@Getter
@AllArgsConstructor
public enum OrderStatus{
    SUCCESS(1),
    PROCESS(2),
    COMPLETE(3),
    CANCEL(-1)
    ;

    private final int value;
}
```

依照上面的 code 直接去新增資料也不會有錯誤，但會發現假設我是設定 `OrderStatus.SUCCESS` 資料庫裡面卻是寫入 `0` 而不是 `1`

原因就是預設會寫入的是 Enum 的順序而不是 value，而在 `find` 的時候也會用順序來轉換成相應的 Enum，不過這不符合我們的需求

## 解法

### Enum name

首先說明如果資料庫要儲存的是 Enum name 的方法，在某些時候我們可以只存 Enum name 就好了，單純用來識別並增加可讀性

+ 那要做的改動不大，如下

```java
@Entity
@data
public class Order{
    private Long id;
    private Long userId;
    private String item;
    @Enumerated(EnumType.String)
    private OrderStatus status;
}
```

簡單用個 annotation 就可以用 Enum name 來轉換

### Enum value

而比較常見的是我們需要的其實是 Enum 的 value，但 Enum 本身並不一定要有 value 的欄位，所以這個我們需要客製化一下

+ 我們先準備好以下的介面跟抽象類別

```java
public interface EnumBase<T> {
  T getValue();
}

@Converter
public abstract class ConverterBase<T extends Enum<T> & EnumBase<E>, E> implements
    AttributeConverter<T, E> {
  private final Class<T> clazz;

  public ConverterBase(Class<T> clazz) {
    this.clazz = clazz;
  }

  @Override
  public E convertToDatabaseColumn(T attribute) {
    return attribute != null ? attribute.getValue() : null;
  }

  @Override
  public T convertToEntityAttribute(E dbData) {
    T[] enums = clazz.getEnumConstants();
    for (T e : enums) {
      if (e.getValue().equals(dbData)) {
        return e;
      }
    }
    return null;
  }
}

```

+ 然後改造一下 Enum 跟 Entity

```java
@Getter
@AllArgsConstructor
public enum OrderStatus implements EnumBase<Integer> {
  SUCCESS(1),
  PROCESS(2),
  COMPLETE(3),
  CANCEL(-1);

  private final int value;

  public static class Converter extends ConverterBase<OrderStatus, Integer> {
    public Converter() {
      super(OrderStatus.class);
    }
  }
}

@EqualsAndHashCode(callSuper = true)
@Entity
@Data
public class Order extends EntityBase {
  private Long id;
  private Long userId;
  private String item;
  @Convert(converter = OrderStatus.Converter.class)
  private OrderStatus status;
}
```

經過 Converter 的轉換之後就可以取之 value 存之 value 了

