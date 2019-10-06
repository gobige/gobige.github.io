---
layout: post
title: '设计模式'
subtitle: '设计模式'
date: 2017-09-20
categories: 设计模式
author: yates
cover: 'http://cctv.com'
tags: 设计模式
---

### 设计模式设计原则
- 开闭原则：**对扩展开放，对修改关闭**，当程序需要升级时，不去修改原有代码，而是扩展原有代码，过程中会用到接口和抽象类，所以对事物的**抽象能力**要很强
- 单一职责原则：每个类应该实现单一职责，如若不是，则要进行拆分（对于java中的方法定义也是一样的）
- 里氏替换原则：任何基类可以出现的地方，子类一定可以出现。只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为
- 依赖倒转原则：面向接口编程，依赖于抽象而不依赖于具体。写代码时用到具体类时，不与具体类交互，而与具体类的上层接口交互
- 接口隔离原则：每个接口中不存在子类用不到却必须实现的方法，如果不然，就要将接口拆分
- 迪米特法则：一个类对自己依赖的类知道的越少越好。也就是说无论被依赖的类多么复杂，都应该将逻辑封装在方法的内部，通过public方法提供给外部。这样当被依赖的类变化时，才能最小的影响该类
- 合成复用原则：尽量首先使用合成/聚合的方式，而不是使用继承

[工厂模式](#factory) 


### 创建型模式
#### 工厂模式

<span id = "factory">工厂模式</span>

如同命名一样，当我们需要某个**对象**的时候，直接调用其**工厂对象**，获取对象即可，**不用关心**对象时怎么创建出来的，**具体实现**是怎么样的

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

### 创建型模式
#### 单例模式
该模式涉及到一个单一的类，该类**负责创建自己的对象**，同时确保只有**单个对象被创建**。这个类提供了一种访问其**唯一的对象的方式**，可以直接访问，不需要实例化该类的对象

**优点**

- 在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例（比如管理学院首页页面缓存）。
- 避免对资源的多重占用（比如写文件操作）。

**缺点**

- 没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。

**使用场景**

- 某个应用，网站的一些不常更新的配置，单独从数据库取出，放入内存中，进行配置的读取

单例模式按加载顺序又可分为饿汉模式，懒汉模式

**饿汉式**
```java
class HungrySingleTon {
    private HungrySingleTon(){}

    private static HungrySingleTon hungrySingleTon = new HungrySingleTon();

    public static HungrySingleTon getInstance() {
        return hungrySingleTon;
    }
}
```

**懒汉式**
```java
class lazySingleTonOfDcl {
    private lazySingleTonOfDcl(){}
    // volatile 关键字，限制cpu对指令重排序
    private static volatile lazySingleTonOfDcl singleTon = null;

    public static lazySingleTonOfDcl getInstance() {
        if (singleTon == null) {
            synchronized (lazySingleTonOfDcl.class) {
                if (singleTon == null) {
                    singleTon = new lazySingleTonOfDcl();
                }
            }
        }
        return singleTon;
    }
}
```
在懒加载单例对象时候，由于Java中**new一个对象**并不是一个**原子操作**，编译时singleton = new Singleton(); 语句会被转成多条汇编指令，它们大致做了3件事情

- 给 Singleton 类的实例分配内存空间； 
- 调用私有的构造函数 Singleton()，初始化成员变量； 
- 将 singleton 对象指向分配的内存（执行完此操作 singleton 就不是 null了）

由于Java编译器允许处理器乱序执行（处理器会根据语句执行效率进行**指令重排序**），以及JDK1.5之前的旧的Java内存模型中Cache、寄存器到主内存回写顺序的规定，上面步骤2)和3)的执行顺序是无法确定的，可能是 1) → 2) → 3) 也可能是 1) → 3) → 2) 。如果是后一种情况，在线程 A 执行完步骤 3) 但还没完成 2) 之前，被切换到线程 B 上，此时线程 B 对singleton 第1次判空结果为false，直接取走了singleton使用，但是构造函数却还没有完成所有的初始化工作，就会出错，也就是DCL失效问题。这时我们在加上volatile关键字就能解决这个问题

另外还有其他实现单例模式的方式，如下图
![此处输入图片的描述](![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-09-20-designPattern/1.png))

### 创建型模式
#### 建造者模式
适用于多个**简单**的对象一步一步**构建**成一个**复杂**的对象，一些**基本**部件**不会变**，而其**组合经常变化**的情况

**优点**

- 建造者独立，易扩展。
- 便于控制细节风险

**缺点**

- 产品必须有共同点，范围有限制。
- 如内部变化复杂，会有很多的建造类。

**使用场景**

- 需要生成的对象具有复杂的内部结构
- 需要生成的对象内部属性本身相互依赖。

示例
```java
/**
 * 建造者模式
 */
public class BuilderPattern {
    public static void main(String[] args) {
        Meal chickenMeal = MealBuilder.buildChickenMeal();
        chickenMeal.showItems();
        Meal vegeMeal = MealBuilder.buildVegeMeal();
        vegeMeal.showItems();
    }
}

class MealBuilder {

    public static Meal buildChickenMeal() {
        Meal meal = new Meal();
        pepsi pepsi = new pepsi();
        chickenBurger chickenBurger = new chickenBurger();
        meal.addItem(pepsi);
        meal.addItem(chickenBurger);

        return meal;
    }

    public static Meal buildVegeMeal() {
        Meal meal = new Meal();
        Coke coke = new Coke();
        VegBurger vegBurger = new VegBurger();
        meal.addItem(coke);
        meal.addItem(vegBurger);

        return meal;
    }
}

class Meal {
    private List<Item> items = new ArrayList<Item>();

    public void addItem(Item item) {
        items.add(item);
    }

    public Double getCost() {
        BigDecimal cost = new BigDecimal("0.00");

        for (Item item : items) {
            cost = cost.add(new BigDecimal(item.price()== null?"0.00":item.price().toString()));
        }

        return cost.doubleValue();
    }

    public void showItems () {
        for (Item item : items) {
            System.out.println(item.toString());
        }
    }
}

class Coke extends ColdDrink {
    @Override
    public String name() {
        return "coke";
    }

    @Override
    public Double price() {
        return 3.50;
    }

    @Override
    public String toString() {
        return "name-" + name() +" " + "price-" + price();
    }
}


class pepsi extends ColdDrink {
    @Override
    public String name() {
        return "pepsi";
    }

    @Override
    public Double price() {
        return 3.00;
    }
    @Override
    public String toString() {
        return "name-" + name() +" " + "price-" + price();
    }
}

class chickenBurger extends Burger {
    @Override
    public String name() {
        return "chicken burger";
    }

    @Override
    public Double price() {
        return 11.2;
    }
    @Override
    public String toString() {
        return "name-" + name() +" " + "price-" + price();
    }
}

class VegBurger extends Burger {
    @Override
    public String name() {
        return "vegBurger";
    }

    @Override
    public Double price() {
        return 5.5;
    }
    @Override
    public String toString() {
        return "name-" + name() +" " + "price-" + price();
    }
}

/**
 * 冷饮
 */
abstract class ColdDrink implements Item {
    @Override
    public Packing pack() {
        return new Bottle();
    }
}

/**
 * 汉堡
 */
abstract class Burger implements Item{
    @Override
    public Packing pack() {
        return new Wrapper();
    }
}

/**
 * 组件接口
 */
interface Item{
    String name();
    Double price();
    Packing pack();
}

/**
 * 盒装
 */
class Wrapper implements Packing {
    @Override
    public String pack() {
        return "wrapper";
    }
}

/**
 * 瓶装
 */
class Bottle implements Packing {
    @Override
    public String pack() {
        return "bottle";
    }
}
/**
 * 包装
 */
interface Packing {
    String pack();
}
```


### 创建型模式
#### 原型模式
原型模式用于创建**重复的对象**，同时又能保证**性能**

**优点**

- 性能提高。 
- 逃避构造函数的约束。

**缺点**

- 配备**克隆**方法需要对类的功能进行通盘考虑，这对于全新的类不是很难，但对于已有的类不一定很容易，特别当一个类引用不支持串行化的间接对象，或者引用含有循环结构的时候。
- 必须实现 Cloneable 接口。

**使用场景**

- 资源优化场景。 
- 类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等。 
- 性能和安全要求的场景。 
- 通过 new 产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式。 
- 一个对象多个修改者的场景。 
- 一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用。 
- 在实际项目中，原型模式很少单独出现，一般是和工厂方法模式一起出现，通过 clone 的方法创建一个对象，然后由工厂方法提供给调用者。

示例
```java
/**
 * 原型模式
 */
public class PrototypePattern {
    public static void main(String[] args) {
        BallCache.loadCache();
        try {
            BallCache.getBall("footBall");
        }catch (Exception e){
            e.getMessage();
        }

    }
}

class BallCache {
    private static Hashtable<String,Ball> hashtable = new Hashtable();


    public static Ball getBall(String ballName) throws Exception{
        return (Ball)hashtable.get(ballName).clone();
    }

    public static void loadCache() {
        hashtable.put("footBall",new FootBall());
        hashtable.put("basketBall",new BasketBall());
    }

}

class FootBall extends Ball {
    FootBall() {
        this.wight = 1.1;
        this.color = "white";
    }
}
class BasketBall extends Ball {
    BasketBall() {
        this.wight = 2.1;
        this.color = "red";
    }
}

abstract class Ball implements Cloneable {
    protected String name;
    protected Double wight;
    protected String color;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Double getWight() {
        return wight;
    }

    public void setWight(Double wight) {
        this.wight = wight;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```



### 结构型模式
#### 适配器模式
适配器模式是作为两个**不兼容**的接口之间的桥梁,它结合了两个独立接口的**功能**。这种模式涉及到一个**单一的类**，该类负责加入独立的或不兼容的接口功能

**优点**

- 可以让任何两个没有关联的类一起运行。 
- 提高了类的复用。
- 增加了类的透明度。 
- 灵活性好。

**缺点**

- 过多地使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是 A 接口，其实内部被适配成了b接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。
- 由于JAVA至多继承一个类，所以至多只能适配一个适配者类，而且目标类必须是抽象类。

**使用场景** 

- 有动机地修改一个正常运行的系统的接口，解决正在服役的项目的问题,这时应该考虑使用适配器模式。

示例
```java
/**
 * 适配器模式：类适配模式
 */
public class Adapter extends targetClass implements CAdaptee {

    public String readXmlByStream() {
        return readXml();
    }
}


/**
 * 适配器模式：对象适配模式
 */
class Adapter2 implements CAdaptee {
    // 该对象的方法适配我所需求
    targetClass tc = new targetClass();


    public String readXmlByStream() {
        return tc.readXml();
    }
}

interface CAdaptee {
    String readXmlByStream();
}

class targetClass {
    public String readXml() {
        return "we can read readXml";
    };
}

/**
 * 让这个工具也能读取xml格式数据
 */
class ReadDataTool extends Adapter2{
    public String readJson() {
        return "i can read json data!";
    }

    @Override
    public String readXmlByStream() {
        return super.readXmlByStream();
    }
}
```


### 结构型模式
#### 桥接模式
桥接（Bridge）是用于把**抽象化与实现化解耦**，使得二者可以**独立变化**,通过提供抽象化和实现化之间的桥接结构，来实现二者的解耦

**优点** 

- 抽象和实现的分离
- 优秀的扩展能力
- 实现细节对客户透明

**缺点**

- 桥接模式的引入会增加系统的理解与设计难度，由于**聚合关联关系建立在抽象层**，要求开发者针**对抽象进行设计**与编程

**使用场景**：  

- 如果一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性，**避免在两个层次之间建立静态的继承联系**，通过桥接模式可以使它们在抽象层建立一个关联关系。
- 对于那些不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统，桥接模式尤为适用
- 一个类存在两个独立变化的维度，且这两个维度都需要进行扩展。

示例
```java
public class brigdePattern {
    public static void main(String[] args) {
        xiaoMian xm = new xiaoMian(new hotStyle(), new redColor());
        System.out.println(xm.eat());
        laMian lm = new laMian(new lightStyle(), new greenColor());
        System.out.println(lm.eat());
    }

}

interface style {
    String getName();
}

class hotStyle implements style {
    public String getName() {
        return "hot style";
    }
}

class lightStyle implements style {
    public String getName() {
        return "light style";
    }
}

interface color {
    String getColor();
}

class greenColor implements color {
    public String getColor() {
        return "green";
    }
}

class redColor implements color {
    public String getColor() {
        return "red";
    }
}

abstract class nodle {
    private style style;
    private color color;

    nodle(style style, color color) {
        this.color = color;
        this.style = style;
    }

    public String getStyle() {
        return style.getName();
    }

    public void setStyle(style style) {
        this.style = style;
    }

    public String getColor() {
        return color.getColor();
    }

    abstract String getName();

    abstract String eat();
}

class xiaoMian extends nodle {

    xiaoMian(style style, color color) {
        super(style, color);
    }

    public String getName() {
        return "xiaomian";
    }

    public String eat() {
        return "eat color：" + this.getColor() + "style:" + this.getStyle() + getName();
    }
}

class laMian extends nodle {
    laMian(style style, color color) {
        super(style, color);
    }

    public String getName() {
        return "laMian";
    }

    public String eat() {
        return "eat color：" + this.getColor() + "style:" + this.getStyle() + getName();
    }
}
```


### 结构型模式
#### 过滤器模式
过滤器模式使用不同的标准来**过滤**一组**对象**，通过逻辑运算以解耦的方式把它们连接起来

示例
```java
/**
 * 标准接口
 */
interface Criteria {
    List<Apple> MaterialCriteria(List<Apple> apples);
}

/**
 * 能做饮料的苹果标准
 */
class AppleDrink implements Criteria {
    @Override
    public List<Apple> MaterialCriteria(List<Apple> apples) {
        List<Apple> appleDrinks = new ArrayList<>();
        if (!CollectionUtils.isEmpty(apples)) {
            for (Apple apple : apples) {
                if ("green".equals(apple.getColor()) && 2 > apple.getWight()) {
                    appleDrinks.add(apple);
                }
            }
        }

        return appleDrinks;
    }
}

/**
 * 能做披萨的苹果标准
 */
class ApplePizza implements Criteria {
    @Override
    public List<Apple> MaterialCriteria(List<Apple> apples) {
        List<Apple> applePizzas = new ArrayList<>();
        if (!CollectionUtils.isEmpty(apples)) {
            for (Apple apple : apples) {
                if ("Brazil".equals(apple.getCountry())) {
                    applePizzas.add(apple);
                }
            }
        }

        return applePizzas;
    }
}

/**
 * 符合两个标准的苹果
 */
class AndCriteria implements Criteria {
    private Criteria criteria;
    private Criteria oherCriteria;

    AndCriteria(Criteria criteria,Criteria oherCriteria) {
        this.criteria = criteria;
        this.oherCriteria = oherCriteria;
    }

    @Override
    public List<Apple> MaterialCriteria(List<Apple> apples) {
        List<Apple> criteriaApples = criteria.MaterialCriteria(apples);
        List<Apple> andApples = oherCriteria.MaterialCriteria(criteriaApples);

        return andApples;
    }
}

/**
 * 两个标准都不符合的苹果
 */
class OrCriteria implements Criteria {
    private Criteria criteria;
    private Criteria oherCriteria;

    OrCriteria(Criteria criteria,Criteria oherCriteria) {
        this.criteria = criteria;
        this.oherCriteria = oherCriteria;
    }

    @Override
    public List<Apple> MaterialCriteria(List<Apple> apples) {
        List<Apple> criteriaApples = criteria.MaterialCriteria(apples);
        List<Apple> otherCriterApples = oherCriteria.MaterialCriteria(apples);
        List<Apple> orApples = new ArrayList<>();

        for (Apple apple : criteriaApples) {
            if (!otherCriterApples.contains(apple)) {
                orApples.add(apple);
            }
        }
        return orApples;
    }
}
```

### 结构型模式
#### 组合模式
组合模式（Composite Pattern），又叫**部分整体**模式，是用于把一组相似的对象当作一个单一的对象。组合模式依据树形结构来组合对象，用来表示部分以及整体层次，创建了一个包含自己对象组的类。该类提供了修改相同对象组的方式

**优点** 

- 高层模块调用简单
- 节点自由增加

**缺点**

- 在使用组合模式时，其叶子和树枝的声明都是实现类，而不是接口，违反了依赖倒置原则

**使用场景**：  

- 部分、整体场景，如树形菜单，文件、文件夹的管理

示例
```java
class subject {
    private String content;
    private String author;
    private String time;

    private Collection<comment> comments;
}

class comment {
    private String content;
    private String replyName;
    private String time;

    private Collection<reply> replies;
}

class reply {
    private String content;
    private String replyName;
    private String time;
}
```


### 结构型模式
#### 装饰器模式
装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构，创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能

**优点** 

- 装饰类和被装饰类可以独立发展，不会相互耦合，装饰模式是**继承的一个替代模式**，装饰模式可以**动态扩展**一个实现类的**功能**

**缺点**

- 多层装饰比较复杂

**使用场景**：  

- 扩展一个类的功能
- 动态增加功能，动态撤销

示例
```java
/**
 * 装饰者模式
 */
public class DecoratorPattern {
    public static void main(String[] args) {
        Person england = new EnglandMan();
        england.speak();
        Person china = new Chinese();
        china.speak();
        Person superman = new FlyPersonDecorator(new Chinese());
        superman.speak();
    }
}

interface Person {
    void speak();
}

class FlyPersonDecorator extends PersonDecorator {
    FlyPersonDecorator(Person decoratePerson) {
        super(decoratePerson);
    }

    @Override
    public void speak() {
        super.speak();
        setWing();
    }

    private void setWing() {
        System.out.println("i can fly");
    }
}

abstract class PersonDecorator implements Person {
    protected Person decoratePerson;

    PersonDecorator(Person decoratePerson) {
        this.decoratePerson = decoratePerson;
    }

    @Override
    public void speak() {
        decoratePerson.speak();
    }
}

class EnglandMan implements Person {

    @Override
    public void speak() {
        System.out.println("i will speak english");
    }
}

class Chinese implements Person{
    @Override
    public void speak() {
        System.out.println("i will speak chinese");
    }
}
```


### 结构型模式
#### 外观模式
外观模式（Facade Pattern）隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口，这种模式涉及到一个单一的类，该类提供了客户端请求的简化方法和对现有系统类方法的委托调用。

**优点** 

- 1、减少系统相互依赖。 2、提高灵活性。 3、提高了安全性

**缺点**

- 不符合开闭原则，如果要改东西很麻烦，继承重写都不合适。

**使用场景**：  

- 1、为复杂的模块或子系统提供外界访问的模块。 2、子系统相对独立。 3、预防低水平人员带来的风险。


示例
```java
public class Facade {
    registerClass rc = new registerClass();
    MathClass mc = new MathClass();
    notifyClass nc = new notifyClass();

    public void registerWeibo() {
        rc.register();
        mc.matchFriend();
        nc.notifyUser();
    }
}

class registerClass {
    public String register() {
        return "注册账号";
    }
}

class MathClass {
    public String matchFriend() {
        return "匹配朋友";
    }
}

class notifyClass {
    public String notifyUser() {
        return "短信通知";
    }
}
```


### 结构型模式
#### 享元模式
享元模式（Flyweight Pattern）主要用于减少创建对象的数量，以减少内存占用和提高性能，享元模式尝试**重用**现有的同类**对象**，如果未找到匹配的对象，则创建新对象

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


### 结构型模式
#### 代理模式
在代理模式（Proxy Pattern）中，一个类**代表**另一个类的**功能**

**优点** 

- 职责清晰。
- 高扩展性。
- 智能化。

**缺点**

- 由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢。 
- 实现代理模式需要额外的工作，有些代理模式的实现非常复杂。

**使用场景**：  

- 远程代理。- 虚拟代理。
- Copy-on-Write 代理。 
- 保护（Protect or Access）代理。
- Cache代理。
- 防火墙（Firewall）代理。
- 同步化（Synchronization）代理。
- 智能引用（Smart Reference）代理


示例
```java
public class proxyPattern {
    public static void main(String[] args) {
        ProxyVideo video = new ProxyVideo("Rick And Morty");
        video.display();
    }
}

class ProxyVideo implements video {
    private YoutuBe youtuBe;
    private String videoName;

    ProxyVideo(String videoName) {
        this.videoName = videoName;
    }


    public String getVideoName() {
        return videoName;
    }

    public void setVideoName(String videoName) {
        this.videoName = videoName;
    }

    @Override
    public void display() {
        if (youtuBe == null) {
            youtuBe = new YoutuBe(videoName);
        }
        youtuBe.display();
    }
}

class YoutuBe implements video {
    private String videoName;

    YoutuBe(String videoName) {
        this.videoName = videoName;
    }

    @Override
    public void display() {
        System.out.println("youtube playing " + videoName);
    }
}

interface video {
    void display();
}
```


### 行为型模式
#### 责任链模式
责任链模式（Chain of Responsibility Pattern）为请求创建了一个接收者对象的链，通常每个接收者都包含对另一个接收者的引用，如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者

**优点** 

- 降低耦合度。它将请求的发送者和接收者解耦。
- 简化了对象。使得对象不需要知道链的结构。 
- 增强给对象指派职责的灵活性。通过改变链内的成员或者调动它们的次序，允许动态地新增或者删除责任。
- 增加新的请求处理类很方便

**缺点**

- 不能保证请求一定被接收。
- 系统性能将受到一定影响，而且在进行代码调试时不太方便，可能会造成循环调用。
- 可能不容易观察运行时的特征，有碍于除错。

**使用场景**：  

- 有多个对象可以处理同一个请求，具体哪个对象处理该请求由运行时刻自动确定。
- 在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。
- 可动态指定一组对象处理请求。



示例
```java

@Component
class ChainHandler1 implements BaseChainHandler {

    @Autowired
    private ChainHandler2 nextHandLer;

    @Override
    public List<Object> filter(handleObj obj) {
        List<Object> returnList = new ArrayList<>();
        for (Object o : obj.getItem()) {
            if (o.equals("test")) {
                returnList.add(o);
            }
        }

        return returnList;
    }

    @Override
    public void handle(handleObj obj) {
        List<Object> returnList = this.filter(obj);

        //todo handle returnList
        returnList.stream().forEach(o -> o.getClass());

        nextHandLer.handle(obj);
    }
}


@Component
class ChainHandler2 implements BaseChainHandler {

    @Override
    public List<Object> filter(handleObj obj) {
        List<Object> returnList = new ArrayList<>();
        for (Object o : obj.getItem()) {
            if (o.equals("test2")) {
                returnList.add(o);
            }
        }

        return returnList;
    }

    @Override
    public void handle(handleObj obj) {
        List<Object> returnList = this.filter(obj);

        //todo handle returnList
        returnList.stream().forEach(o -> o.getClass());

    }
}

interface BaseChainHandler {
    List<Object> filter(handleObj obj);

    void handle(handleObj obj);
}

class handleObj {
    List<Object> item;

    public List<Object> getItem() {
        return item;
    }

    public void setItem(List<Object> item) {
        this.item = item;
    }
}
```


### 行为型模式
#### 命令模式
命令模式（Command Pattern）是一种数据驱动的设计模式,请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令

**优点** 

- 降低了系统耦合度。
- 新的命令可以很容易添加到系统中去。

**缺点**

- 使用命令模式可能会导致某些系统有过多的具体命令类

**使用场景**：  

- 认为是命令的地方都可以使用命令模式，比如： 1、GUI 中每一个按钮都是一条命令。 2、模拟 CMD



示例
```java
public class CommandPattern {
    public static void main(String[] args) {
        Stock stock = new Stock("footBall",100);
        AddStock addStock = new AddStock(stock,23);
        ReduceStock reduceStock = new ReduceStock(stock,121);
        Cmd.scanf(addStock);
        Cmd.scanf(reduceStock);
        Cmd.excute();
    }

}

class Cmd {
    private static List<Commond> commonds = new ArrayList<>();

    public static void scanf(Commond commond) {
        commonds.add(commond);
    }

    public static void excute() {
        commonds.stream().forEach(commond -> commond.excute());
    }
}

class Stock {
    private String GoodName;
    private Integer num;

    Stock(String goodName, Integer num) {
        this.GoodName = goodName;
        this.num = num;
    }

    public void add(Integer num) {
        System.out.println(GoodName + " add stock[" + num + "]");
    }

    public void reduce(Integer num) {
        System.out.println(GoodName + " reduce stock[" + num + "]");
    }
}

class AddStock implements Commond {

    private Stock stock;
    private Integer num;

    AddStock(Stock stock,Integer num) {
        this.stock = stock;
        this.num = num;
    }

    @Override
    public void excute() {
        stock.add(num);
    }
}

class ReduceStock implements Commond {

    private Stock stock;
    private Integer num;

    ReduceStock(Stock stock,Integer num) {
        this.stock = stock;
        this.num = num;
    }

    @Override
    public void excute() {
        stock.reduce(num);
    }
}

interface Commond {
    void excute();

```


### 行为型模式
#### 解释器模式
解释器模式（Interpreter Pattern）提供了评估语言的语法或表达式的方式，这种模式实现了一个表达式接口，该接口解释一个特定的上下文。这种模式被用在 SQL 解析、符号处理引擎等

**优点** 

- 可扩展性比较好，灵活。
- 增加了新的解释表达式的方式
- 易于实现简单语法

**缺点**

- 可利用场景比较少。JAVA 中如果碰到可以用 expression4J 代替
- 对于复杂的语法比较难维护。
- 解释器模式会引起类膨胀。
- 解释器模式采用递归调用方法。

**使用场景**：  

- 可以将一个需要解释执行的语言中的句子表示为一个抽象语法树。
- 一些重复出现的问题可以用一种简单的语言来进行表达。
- 一个简单语法需要解释的场景



示例
```java
public class InterpreterPattern {
    public static void main(String[] args) {
        System.out.println("hello,this is bbc news is news?:" + InterpreterPattern.setNewsWord().interpret("hello,this is bbc news"));
        System.out.println("1 hour 30 mills is time?:" + InterpreterPattern.setTimeWord().interpret("1 hour 30 mills"));

    }

    public static Explain setNewsWord() {
        Explain contextExplain = new ContextExplain("news");
        Explain contextExplain2 = new ContextExplain("report");

        return new OrExplain(contextExplain,contextExplain2);
    }

    public static Explain setTimeWord() {
        Explain contextExplain = new ContextExplain("minute");
        Explain contextExplain2 = new ContextExplain("hour");

        return new AndExplain(contextExplain,contextExplain2);
    }
}

class AndExplain implements Explain {
    private Explain orExplain1;
    private Explain orExplain2;

    AndExplain(Explain orExplain1, Explain orExplain2) {
        this.orExplain1 = orExplain1;
        this.orExplain2 = orExplain2;
    }


    @Override
    public boolean interpret(String context) {
        return orExplain1.interpret(context) && orExplain2.interpret(context);
    }
}

class OrExplain implements Explain {
    private Explain orExplain1;
    private Explain orExplain2;

    OrExplain(Explain orExplain1, Explain orExplain2) {
        this.orExplain1 = orExplain1;
        this.orExplain2 = orExplain2;
    }


    @Override
    public boolean interpret(String context) {
        return orExplain1.interpret(context) || orExplain2.interpret(context);
    }
}

class ContextExplain implements Explain {
    private  String data;

    ContextExplain(String data) {
        this.data = data;
    }

    @Override
    public boolean interpret(String context) {
        if (context.contains(data)) {
            return true;
        }

        return false;
    }
}

interface Explain {
    boolean interpret(String context);
}

```



### 行为型模式
#### 迭代器模式
迭代器模式用于顺序访问集合对象的元素，不需要知道集合对象的底层表示

**优点** 

- 它支持以不同的方式遍历一个聚合对象
- 迭代器简化了聚合类
- 在同一个聚合上可以有多个遍历
- 在迭代器模式中，增加新的聚合类和迭代器类都很方便，无须修改原有代码

**缺点**

- 由于迭代器模式将存储数据和遍历数据的职责分离，增加新的聚合类需要对应增加新的迭代器类，类的个数成对增加，这在一定程度上增加了系统的复杂性

**使用场景**：  

- 访问一个聚合对象的内容而无须暴露它的内部表示
- 需要为聚合对象提供多种遍历方式
- 为遍历不同的聚合结构提供一个统一的接口



示例
```java
public class IteratorPattern {
    public static void main(String[] args) {
        MyContainer container = new MyContainer();
        container.add("str1");
        container.add("str7");
        container.add("str5");
        container.add("str2");
        container.add("str3");

        MyIterator iterator = container.getIterator();

        while (iterator.hashNext()) {
            System.out.println(iterator.next());
        }
    }
}

class MyContainer implements Container {
    private String[] strs = new String[10];
    private int size = 0;

    @Override
    public void add(Object object) {
        strs[size++] = (String) object;
    }

    @Override
    public MyIterator getIterator() {
        return new Iterator();
    }

    private class Iterator implements MyIterator {
        int index = 0;
        @Override
        public boolean hashNext() {
            if (index < strs.length) {
                return true;
            }
            return false;
        }

        @Override
        public Object next() {
            if (hashNext()) {
                return strs[index++];
            }

            return null;
        }
    }
}

interface Container {
    MyIterator getIterator();
    void add(Object obj);
}

interface MyIterator {
    boolean hashNext();
    Object next();
}

```


### 行为型模式
#### 中介者模式
中介者模式（Mediator Pattern）是用来**降低**多个**对象和类**之间的通信**复杂性**。这种模式提供了一个中介类，该类通常处理不同类之间的通信，并支持松耦合，使代码易于维护

**优点** 

- 降低了类的复杂度，将一对多转化成了一对一。
- 各个类之间的解耦。
- 符合迪米特原则。

**缺点**

- 中介者会庞大，变得复杂难以维护
- 系统性能将受到一定影响，而且在进行代码调试时不太方便，可能会造成循环调用。
- 可能不容易观察运行时的特征，有碍于除错。

**使用场景**：  

- 系统中对象之间存在比较复杂的引用关系，导致它们之间的依赖关系结构混乱而且难以复用该对象。
- 想通过一个中间类来封装多个类中的行为，而又不想生成太多的子类



示例
```java
public class MediatorPattern {

    public static void main(String[] args) {
        concreteMediator cm = new concreteMediator();
        ColleagueA a = new ColleagueA("tom", cm);
        ColleagueB b = new ColleagueB("jerry", cm);
        cm.setA(a);
        cm.setB(b);
        a.contact("i have something to b");
        b.contact("i busy!! don't intterupt me");
    }
}

abstract class Mediator {
    public abstract void contact(String content, Colleague coll);
}

class concreteMediator extends Mediator {
    ColleagueA a;
    ColleagueB b;

    public ColleagueA getA() {
        return a;
    }

    public void setA(ColleagueA a) {
        this.a = a;
    }

    public ColleagueB getB() {
        return b;
    }

    public void setB(ColleagueB b) {
        this.b = b;
    }

    @Override
    public void contact(String content, Colleague coll) {
        if (coll == a) {
            b.getMes(content);
        } else {
            a.getMes(content);
        }
    }
}


class ColleagueA extends Colleague {
    public ColleagueA(String name, Mediator mediator) {
        super(name, mediator);
    }

    public void getMes(String mes) {
        System.out.println("collA " + name + "get mes " + mes);
    }

    public void contact(String mes) {
        mediator.contact(mes, this);
    }
}

class ColleagueB extends Colleague {
    public ColleagueB(String name, Mediator mediator) {
        super(name, mediator);
    }

    public void getMes(String mes) {
        System.out.println("collB " + name + "get mes " + mes);
    }

    public void contact(String mes) {
        mediator.contact(mes, this);
    }
}

class Colleague {
    protected String name;
    protected Mediator mediator;

    public Colleague(String name, Mediator mediator) {
        this.name = name;
        this.mediator = mediator;
    }
}
```


### 行为型模式
#### 备忘录模式
备忘录模式（Memento Pattern）保存一个对象的某个状态，以便在适当的时候恢复对象

**优点** 

- 给用户提供了一种可以恢复状态的机制，可以使用户能够比较方便地回到某个历史的状态
- 实现了信息的封装，使得用户不需要关心状态的保存细节

**缺点**

- 消耗资源。如果类的成员变量过多，势必会占用比较大的资源，而且每一次保存都会消耗一定的内存

**使用场景**：  

- 需要保存/恢复数据的相关状态场景
- 提供一个可回滚的操作

示例
```java
public class MementoPattern {
    public static void main(String[] args) {
        ArchiveTool archiveTool = new ArchiveTool();
        ArchiveArea archiveArea = new ArchiveArea();

        archiveTool.setState("三打白骨精");
        archiveTool.setState("大战红孩儿");
        archiveArea.add(archiveTool.saveArchiveFile());
        archiveTool.setState("真假美猴王");
        archiveArea.add(archiveTool.saveArchiveFile());
        archiveTool.setState("三借芭蕉扇");
        archiveArea.add(archiveTool.saveArchiveFile());

        int i = 0;
        ArchiveFile file;
        while (true) {
            file = archiveArea.get(i);
            if (file == null) {
                return;
            }
            System.out.println("存档记录--" + file.getState());
            i++;
        }
    }
}

/**
 * 存档区
 */
class ArchiveArea {
    private List<ArchiveFile> archiveFiles = new ArrayList<>();

    public void add(ArchiveFile file) {
        archiveFiles.add(file);
    }

    public ArchiveFile get(Integer index) {
        if (index < archiveFiles.size()) {
            return archiveFiles.get(index);
        }

        return null;
    }
}

/**
 * 存档工具
 */
class ArchiveTool {
    private String state;

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }

    public ArchiveFile saveArchiveFile() {
        return new ArchiveFile(state);
    }

    public void getArchiveFileState(ArchiveFile archiveFile) {
        state = archiveFile.getState();
    }
}

/**
 * 存档文件
 */
class ArchiveFile {
    private String state;

    ArchiveFile(String state) {
        this.state = state;
    }

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }
}
```


### 行为型模式
#### 观察者模式
定义对象间的一种一对多的依赖关系，当一个对象的**状态**发生**改变**时，所有**依赖**于它的对象都得到**通知**并被自动更新

**优点** 

- 观察者和被观察者是抽象耦合的
- 建立一套触发机制。

**缺点**

- 如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间
- 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃
- 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化

**使用场景**：  

- 一个抽象模型有两个方面，其中一个方面依赖于另一个方面。将这些方面封装在独立的对象中使它们可以各自独立地改变和复用
- 一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变，可以降低对象之间的耦合度
- 一个对象必须通知其他对象，而并不知道这些对象是谁
- 需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，可以使用观察者模式创建一种链式触发机制(JAVA 中已经有了对观察者模式的支持类)

示例
```java
public class ObserverPattern {
    
    public static void main(String[] args) {
        Subject subject = new Subject();
        Observer1 observer1 = new Observer1(subject);
        Observer2 observer2 = new Observer2(subject);
        subject.addObserver(observer1);
        subject.addObserver(observer2);
        subject.setState(10);
        subject.setState(100);
    }
}

class Observer1 extends Observer {
    Observer1(Subject subject) {
        this.subject = subject;
    }

    @Override
    void update() {
        System.out.println("观察者一号观察的对象状态改变为：" + subject.getState());
    }
}

class Observer2 extends Observer {
    Observer2(Subject subject) {
        this.subject = subject;
    }

    @Override
    void update() {
        System.out.println("观察者二号观察的对象状态改变为：" + subject.getState());
    }
}

class Subject {
    private List<Observer> observers = new ArrayList<>();

    private Integer state;

    public Integer getState() {
        return state;
    }

    public void setState(Integer state) {
        this.state = state;
        notifyAllObserver();
    }

    public void addObserver(Observer observer) {
        observers.add(observer);
    }

    private void notifyAllObserver() {
        for (Observer observer : observers) {
            observer.update();
        }
    }

}

/**
 * 观察者
 */
abstract class Observer {
    protected Subject subject;
    void update(){};
}
```


### 行为型模式
#### 状态模式
在状态模式（State Pattern）中，类的行为是基于它的状态改变的，在状态模式中，我们创建表示各种状态的对象和一个行为随着状态对象改变而改变的 context 对象

**优点** 

- 封装了转换规则
- 枚举可能的状态，在枚举状态之前需要确定状态种类
- 将所有与某个状态有关的行为放到一个类中，并且可以方便地增加新的状态，只需要改变对象状态即可改变对象的行为
- 允许状态转换逻辑与状态对象合成一体，而不是某一个巨大的条件语句块
- 可以让多个环境对象共享一个状态对象，从而减少系统中对象的个数

**缺点**

- 状态模式的使用必然会增加系统类和对象的个数
- 状态模式的结构与实现都较为复杂，如果使用不当将导致程序结构和代码的混乱
- 状态模式对"开闭原则"的支持并不太好，对于可以切换状态的状态模式，增加新的状态类需要修改那些负责状态转换的源代码，否则无法切换到新增状态，而且修改某个状态类的行为也需修改对应类的源代码

**使用场景**：  

- 行为随状态改变而改变的场景
- 条件、分支语句的代替者。

示例
```java
public class StatePattern {
    public static void main(String[] args) {
        Context context = new Context();
        StartState startState = new StartState();
        EndState endState = new EndState();
        startState.doAction(context);
        endState.doAction(context);
    }
}

class EndState implements State {
    @Override
    public void doAction(Context context) {
        System.out.println("context at end status");
        context.setState(this);
    }
}

class StartState implements State {
    @Override
    public void doAction(Context context) {
        System.out.println("context at start status");
        context.setState(this);
    }
}

interface State {
    void doAction(Context context);
}

class Context {
    private State state;

    Context() {
        state = null;
    }
    public State getState() {
        return state;
    }

    public void setState(State state) {
        this.state = state;
    }
}
```


### 行为型模式
#### 策略模式
在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改

**优点** 
- 算法可以自由切换
- 避免使用多重条件判断
- 扩展性良好。

**缺点**
- 策略类会增多
- 所有策略类都需要对外暴露

**使用场景**：  

- 如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为
- 一个系统需要动态地在几种算法中选择一种
- 如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现

示例
在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法
```java
public class StrategyPattern {

    public static void main(String[] args) {
        Army army1 = new Army(new OffensiveStrategy());
        Army army2 = new Army(new RetreatStrategy());

        army1.excuteStrategy();
        army2.excuteStrategy();
    }
}

class Army {
    private Strategy strategy;

    Army(Strategy strategy) {
        this.strategy = strategy;
    }

    void excuteStrategy() {
        strategy.doOperation();
    }
}

/**
 * 进攻策略
 */
class OffensiveStrategy implements Strategy {
    @Override
    public void doOperation() {
        System.out.println("执行进攻策略");
    }
}

/**
 * 撤退策略
 */
class RetreatStrategy implements Strategy {
    @Override
    public void doOperation() {
        System.out.println("执行撤退策略");
    }
}

interface Strategy {
    void doOperation();
}
```


### 行为型模式
#### 空对象模式
在空对象模式（Null Object Pattern）中，一个空对象取代NULL对象实例的检查。Null对象不是检查空值，而是反应一个不做任何动作的关系。这样的 Null 对象也可以在数据不可用的时候提供默认的行为。

**优点** 

**缺点**

**使用场景**：  

- 需要检查空值的地方

示例
在空对象模式中，我们创建一个指定各种要执行的操作的抽象类和扩展该类的实体类，还创建一个未对该类做任何实现的空对象类，该空对象类将无缝地使用在需要检查空值的地方
```java
ublic class NullObjectPattern {

    public static void main(String[] args) {
        NullObjectPattern nullObjectPattern = new NullObjectPattern();

        Factory factory = nullObjectPattern.new Factory();
        System.out.println(factory.getSubject("haha").getName());
        System.out.println(factory.getSubject("PE").getName());
    }

     class Factory {
        public final String[] names = {"PE","MATH","LITERATURE","POLITICAL"};

        public Subject getSubject(String subject) {
            for (String name : names) {
                if (name.equals(subject)) {
                    return new SecondarySubject(subject);
                }
            }

            return new NullSubject();
        }
    }


    class NullSubject extends Subject {
        @Override
        boolean isNull() {
            return true;
        }

        @Override
        String getName() {
            return "not avilable subject";
        }
    }

    class SecondarySubject extends Subject {
        SecondarySubject( String name) {
            this.name = name;
        }

        @Override
        boolean isNull() {
            return false;
        }

        @Override
        String getName() {
            return name;
        }
    }

    abstract class Subject {
        String name;
        abstract boolean isNull();
        abstract String getName();
    }
}
```


### 行为型模式
#### 模板模式
在模板模式（Template Pattern）中，一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行

**优点** 
- 封装不变部分，扩展可变部分
- 提取公共代码，便于维护
- 行为由父类控制，子类实现

**缺点**
- 每一个不同的实现都需要一个子类来实现，导致类的个数增加，使得系统更加庞大

**使用场景**：  

- 有多个子类共有的方法，且逻辑相同
- 重要的、复杂的方法，可以考虑作为模板方法

示例
定义一个操作中的算法的骨架，而将一些步骤延迟到子类中,为防止恶意操作，一般模板方法都加上 final 关键词
```java
public class TemplatePattern {
    public static void main(String[] args) {
        cooking tofuOfMapo = new TofuOfMapo();
        tofuOfMapo.cookingFood();

        cooking hotChicken = new HotChicken();
        hotChicken.cookingFood();
    }
}

class TofuOfMapo extends cooking{
    @Override
    void prePareCokie() {
        System.out.println("切豆腐");
        System.out.println("宽油");
    }

    @Override
    void cooking() {
        System.out.println("烧酱汁");
        System.out.println("放豆腐");
    }

    @Override
    void cookied() {
        System.out.println("起锅，装盘");
    }
}

class HotChicken extends cooking{
    @Override
    void prePareCokie() {
        System.out.println("杀鸡，放血，拔毛");
        System.out.println("刀工");
    }

    @Override
    void cooking() {
        System.out.println("宽油");
        System.out.println("烧制");
    }

    @Override
    void cookied() {
        System.out.println("起锅，装盘");
    }
}

abstract class cooking {
    abstract void prePareCokie();

    abstract void cooking();

    abstract  void cookied();

    public final void cookingFood() {
        prePareCokie();
        cooking();
        cookied();
    }
}

```


### 行为型模式
#### 访问者模式
在访问者模式（Visitor Pattern）中，我们使用了一个访问者类，它改变了元素类的执行算法。通过这种方式，元素的执行算法可以随着访问者改变而改变,根据模式，元素对象已接受访问者对象，这样访问者对象就可以处理元素对象上的操作

**优点** 
- 符合单一职责原则
- 优秀的扩展性
- 灵活性

**缺点**
- 具体元素对访问者公布细节，违反了迪米特原则
- 具体元素变更比较困难
- 违反了依赖倒置原则，依赖了具体类，没有依赖抽象

**使用场景**：  

- 对象结构中对象对应的类很少改变，但经常需要在此对象结构上定义新的操作
- 需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作"污染"这些对象的类，也不希望在增加新操作时修改这些类

示例

```java
public class VisitorPattern {
    public static void main(String[] args) {
        RealComputerVisitor visitor = new RealComputerVisitor();
        Computer computer = new Computer();
        computer.accept(visitor);

    }
}

class RealComputerVisitor implements ComputerVisitor {
    @Override
    public void visit(Computer computer) {
        System.out.println("visiting computer");
    }

    @Override
    public void visit(KeyBoard keyBoard) {
        System.out.println("visiting keyBoard");
    }

    @Override
    public void visit(Mouse mouse) {
        System.out.println("visiting mouse");
    }

    @Override
    public void visit(Monitor monitor) {
        System.out.println("visiting monitor");
    }
}

class Computer implements ComputerPart {
    ComputerPart[] parts;

    Computer() {
        parts = new ComputerPart[] {new Mouse(),new Monitor(),new KeyBoard()};
    }

    @Override
    public void accept(ComputerVisitor computerVisitor) {
        for (ComputerPart part : parts) {
            part.accept(computerVisitor);
        }
        computerVisitor.visit(this);
    }
}

interface ComputerVisitor {
    void visit(Computer computer);
    void visit(KeyBoard keyBoard);
    void visit(Mouse mouse);
    void visit(Monitor monitor);
}

class Monitor implements ComputerPart {
    @Override
    public void accept(ComputerVisitor computerVisitor) {
        computerVisitor.visit(this);
    }
}

class Mouse implements ComputerPart {
    @Override
    public void accept(ComputerVisitor computerVisitor) {
        computerVisitor.visit(this);
    }
}

class KeyBoard implements ComputerPart {
    @Override
    public void accept(ComputerVisitor computerVisitor) {
        computerVisitor.visit(this);
    }
}

interface ComputerPart {
    void accept(ComputerVisitor computerVisitor);
}
```


TODO 
未完待续