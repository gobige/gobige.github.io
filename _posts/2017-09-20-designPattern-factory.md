---
layout: post
title: '设计模式之工厂模式'
subtitle: '工厂模式'
date: 2017-09-20
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 工厂模式
---

### 设计模式设计原则
- 开闭原则：**对扩展开放，对修改关闭**，当程序需要升级时，不去修改原有代码，而是扩展原有代码，过程中会用到接口和抽象类，所以对事物的**抽象能力**要很强
- 单一职责原则：每个类应该实现单一职责，如若不是，则要进行拆分（对于java中的方法定义也是一样的）
- 里氏替换原则：任何基类可以出现的地方，子类一定可以出现。只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为
- 依赖倒转原则：面向接口编程，依赖于抽象而不依赖于具体。写代码时用到具体类时，不与具体类交互，而与具体类的上层接口交互
- 接口隔离原则：每个接口中不存在子类用不到却必须实现的方法，如果不然，就要将接口拆分
- 迪米特法则：一个类对自己依赖的类知道的越少越好。也就是说无论被依赖的类多么复杂，都应该将逻辑封装在方法的内部，通过public方法提供给外部。这样当被依赖的类变化时，才能最小的影响该类
- 合成复用原则：尽量首先使用合成/聚合的方式，而不是使用继承

### 创建型模式
#### 工厂模式
如同命名一样，当我们需要某个对象的时候，直接调用其工厂对象，获取对象即可，不用关心对象时怎么创建出来的，具体实现是怎么样的

**优点**

- 只需直到创建对象的名字
- 只需关系接口本身
- 增加一个产品只需增加一个工厂类
- 复杂对象适合使用工厂模式

**缺点**

- 每次增加一个对象时，都需要增加一个具体类和对象实现工厂，使得系统中类的个数成倍增加，在一定程度上增加了系统的复杂度，同时也增加了系统具体类的依赖

**使用场景**

- 如spring的jdbc不同数据库访问

**示例**
根据不同的订单类型和订单id返回不同的订单
```java
/**
 * 工厂模式
 */
public class FactoryPattern {

    public Order getOrder(Integer orderId, OrderTypeEnum orderTypeEnum) {
        if (OrderTypeEnum.GOODSORDERTYPE == orderTypeEnum) {
            // 返回商品订单
            return new GoodsOrder();
        } else if(OrderTypeEnum.SERVICEORDERTYPE == orderTypeEnum) {
            // 返回服务订单
            return new ServicesOrder();
        } else if (OrderTypeEnum.AFTERSALEORDERTYPE == orderTypeEnum) {
            // 返回售后订单
            return new AfsOrder();
        } else {
            return null;
        }
    }
}


/**
 * 售后订单的model
 */
class AfsOrder extends Order {
    /**
     * 订单类型
     */
    private Integer orderType;
    /**
     * 客服姓名
     */
    private String servicerName;
}

/**
 * 服务订单的model
 */
class ServicesOrder extends Order {
    /**
     * 订单类型
     */
    private Integer orderType;
    /**
     * 服务内容
     */
    private String serviceContent;
}

/**
 * 商品订单model
 */
class GoodsOrder extends Order {
    /**
     * 订单类型
     */
    private Integer orderType;
    /**
     * 商品名称
     */
    private String goodsName;
}

class Order {
    /**
     * 订单ID
     */
    private Integer orderId;
    /**
     * 订单金额
     */
    private Double orderFee;
    /**
     *
     */
    private Date createTime;
}

enum OrderTypeEnum {
    GOODSORDERTYPE(1,"商品订单"),
    SERVICEORDERTYPE(2,"服务订单"),
    AFTERSALEORDERTYPE(3,"售后订单");

    OrderTypeEnum(Integer key,String value) {
        this.key = key;
        this.value = value;
    }
    private Integer key;
    private String value;

    public Integer getKey() {
        return key;
    }

    public void setKey(Integer key) {
        this.key = key;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}
```


TODO 
未完待续