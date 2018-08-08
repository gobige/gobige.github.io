---
layout: post
title: '设计模式之享元模式'
subtitle: '享元模式'
date: 2017-09-30
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 享元模式
---

### 结构型模式
#### 享元模式
享元模式（Flyweight Pattern）主要用于减少创建对象的数量，以减少内存占用和提高性能，享元模式尝试重用现有的同类对象，如果未找到匹配的对象，则创建新对象

**优点** 

- 大大减少对象的创建，降低系统的内存，使效率提高

**缺点**

- 提高了系统的复杂度，需要分离出外部状态和内部状态，而且外部状态具有固有化的性质，不应该随着内部状态的变化而变化，否则会造成系统的混乱

**使用场景**：  

-  1、系统有大量相似对象。 2、需要缓冲池的场景。


示例
```java
public class FlyweightPattern {
    private static final ExpressCompanyEnum express[] = ExpressCompanyEnum.values();

    public static void main(String[] args) {
        for (int i = 0; i < 1000; i++) {
            DeliverFare deliverFare = CacheFactory.getDeliverFare(getRandomExpressCompany());
            deliverFare.setUnitNum(getDeliverUnitNum());
            deliverFare.caculate();
        }
    }

    private static ExpressCompanyEnum getRandomExpressCompany() {
        return express[(int)(Math.random()*express.length)];
    }

    private static int getDeliverUnitNum() {
        return (int)(Math.random() * 10);
    }

}

class CacheFactory {
    private static Map<String,DeliverFare> deliverFareMap = new HashMap<>();

    public static DeliverFare getDeliverFare(ExpressCompanyEnum companyEnum) {
        if (deliverFareMap.containsKey(companyEnum.getKey())) {
            System.out.println("获取以前对象");
            return deliverFareMap.get(companyEnum.getKey());
        }else {
            DeliverFare deliverFare = new DeliverFare(companyEnum);
            deliverFareMap.put(companyEnum.getKey(),deliverFare);
            System.out.println("创建新对象");
            return deliverFare;
        }
    }
}

class DeliverFare implements ExpressDelivery {
    private Integer unitNum;
    private ExpressCompanyEnum company;
    DeliverFare(ExpressCompanyEnum company) {
        this.company = company;
    }

    @Override
    public void caculate() {
        System.out.println(company.getValue() + "计算运费，货物数量：" + unitNum);
    }

    public Integer getUnitNum() {
        return unitNum;
    }

    public void setUnitNum(Integer unitNum) {
        this.unitNum = unitNum;
    }

    public ExpressCompanyEnum getCompany() {
        return company;
    }

    public void setCompany(ExpressCompanyEnum company) {
        this.company = company;
    }
}

enum ExpressCompanyEnum {
    SHUNFENG("shunfeng","顺丰"),
    ZHONGTONG("zhongtong","中通"),
        JD("jd","京东");

    ExpressCompanyEnum(String key,String value) {
        this.key = key;
        this.value = value;
    }
    private String key;
    private String value;

    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}

interface ExpressDelivery {
    void caculate();
}
```