---
layout: post
title: '注解知多少'
subtitle: '关于java的注解你又知道多少呢'
date: 2017-09-03
categories: java
author: yates
cover: 'http://cctv.com'
tags: jdk
---

### 前言
本文将介绍注解的基本结构，语法；使用场景；怎么自定义注解；java中常见的注解等知识
	    
### 注解的基本语法
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-09-03-annotationMore/1.png)
    
- 注解使用@interface关键字定义
        
- @Target是这个注解的作用域，也就是说这个注解可以写在什么地方，有下列作用域范围
    * CONSTRUCTOR（构造方法声明）
    * FIELD（字段声明）
    * LOCAL VARIABLE（局部变量声明）
    * METHOD（方法声明）
    * PACKAGE（包声明）
    * PARAMETER（参数声明）
    * TYPE（类接口）
            
- @Retention是该注解的生命周期，就是说在Java程序从编译，解析到执行，什么时候能够获取得到该注解，有下列生命周期定义
    * SOURCE（只在源码显示，编译时丢弃）
    * CLASS（编译时记录到class中，运行时忽略）
    * RUNTIME（运行时存在，可以通过反射读取）
            
- @Inherited是一个标识性的元注解，它允许子注解继承它
        
-  @Documented，生成javadoc时会包含注解
        
- @Repeatable是可重复的意思。注解的值可以同时取多个
        
- 可以看到方法体内有name，values等成员变量（成员以无参无异常的方式声明，成员变量可以用default指定一个默认值的）
    * 成员类型是受限制的，合法的类型包括基本的数据类型以及String，Class，Annotation,Enumeration等
    * 如果注解只有一个成员，则成员名必须取名为value()，在使用时可以忽略成员名和赋值号（=）
    ![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-09-03-annotationMore/3.png)
    * 注解类可以没有成员，没有成员的注解称为标识注解。
            
- 使用注解的语法：@<注解名>(<成员名1>=<成员值1>,<成员名1>=<成员值1>,…)

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-09-03-annotationMore/2.png)

### Java中常见的注解
#### @Override
我们都知道，java三大特性的继承中，方法是可以覆盖的，如图：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-09-03-annotationMore/4.png)

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-09-03-annotationMore/5.png)

所有类都隐式从object继承，testAno类**目的是覆盖**object的equals方法,但是得到的结果却是错误的，因为我们**重写**object的equals方法时制定了一个非object类型参数，从而引发了这个方法**重载**，而不是重写的效果，当我们引入@Override注解后，会在编译的时候提示报错的信息，因此我么可以使用该注解来确保子类的方法也覆盖超类中的非最终具体方法或抽象方法以及接口方法
    
#### @Deprecated
在开发代码时，有时候代码会变得过时，在新的调用中被其他更合适的代码替换，但是为了兼容以前的对该代码调用的地方的正常运行，以及在未来的版本中被弃用，可以使用该注解，在程序运行中使用过时代码的时候会得到warning提示

#### @FunctionalInterface
随着jdk8的lambda表达式引入，函数式接口（一个函数式接口只有一个抽象方法，由于默认方法有一个实现，所以他们不是抽象的）越来越流行，这些接口可以使用lambda表达式，方法引用，或构造函数引用代替。

```java
public interface Foo {
    public int doSomething();
}

public interface Bar {
    public int doSomething();
    public default int doSomethingElse() {
        return 1;
    }
}
```

因此，下面的对象可以用lambda表达式代替

```java
class FunctionalConsumer {
    public void consumeFoo(Foo foo) {
        System.out.println(foo.doSomething());
    }
    public void consumeBar(Bar bar) {
        System.out.println(bar.doSomething());
    }
}

// lambda调用方法
FunctionalConsumer functionalConsumer = new FunctionalConsumer();
        functionalConsumer.consumeBar(() -> 10);
        functionalConsumer.consumeBar(() -> 20);
```

如果我们错误的将Foo和Bar接口定义为非函数式接口，那么编译器便不会报错直到运行报错，于是可以使用FunctionalInterface注解，编译器便指出错误

#### @SuppressWarnings
警告是编译器组成部分，为开发人员提供反馈，可能的危险行为或在未来的编译器版本中可能会出现的错误，例如在java 泛型类型使用没有关联正式泛型参数。为了忽略某些特定警告，可以使用该注解

#### @SafeVarargs
参数安全类型注解 
同样还有很多著名的第三方注解，如：spring的Autowired，Service等

###注解的提取
标注为生命周期在运行时状态的注解可以通过反射来提取，如下：

```java
@MyIF(author = "muyibeyond",desc = "test anotation class")
public class AnnotationTest {
	@MyIF(author = "muyibeyond",desc = "test anotation method")
	public void method() {
		System.out.println("do something");
	}

	public static void main(String[] args) {
		try {
			// 加载类
			Class c = Class.forName("practice.base.AnnotationTest");
			// 获取是否有对应注解
			boolean classAnoIsExist = c.isAnnotationPresent(MyIF.class);

			if (classAnoIsExist) {
				// 拿到注解实例，解析注解
				MyIF d = (MyIF) c.getAnnotation(MyIF.class);
				System.out.println("编写这个类的作者：" + d.author() + "类描述：" + d.desc());
			}

			// 获取类方法
			Method[] ms = c.getMethods();
			for (Method m : ms) {
				boolean meheodAnoIsExist = m.isAnnotationPresent(MyIF.class);
				if (meheodAnoIsExist) {
					MyIF d=m.getAnnotation(MyIF.class);
					System.out.println("编写这个方法的作者：" + d.author() + "类描述：" + d.desc());
				}
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}
	}
}

@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@interface MyIF{
	String author() default "yates";
	String desc() default "";
}
```

运行结果：

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/2017-09-03-annotationMore/6.png)

### 注解的使用场景
那么应该在什么地方使用注解呢？我们来看看官方的解释
注解是一系列***元数据***，它提供数据用来解释程序代码，但是注解并非是所解释的代码本身的一部分。注解对于代码的运行效果没有直接影响。
注解有许多用处，主要如下： 
- 提供信息给编译器： 编译器可以利用注解来**探测错误和警告信息** 
- 编译阶段时的处理： 软件工具可以用来利用注解信息来**生成代码、Html文档**或者做其它相应处理。 
- 运行时的处理： 某些注解可以在**程序运行**的时候**接受代码的提取**

说白了注解怎么使用取决于你想利用它干什么，通常注解主要给编译器及工具类型的软件用的，还有很重要的一点注解的提取需要借助于 Java 的**反射**技术，反射比较**慢**，所以注解使用时也需要谨慎计较**时间成本**。

### 项目中常见的注解

#### Spring框架

**声明类型**
@Component 组件，没有明确的角色

@Service 在业务逻辑层使用（service层）

@Repository 在数据访问层使用（dao层）

@Controller 在视图层使用，控制器的声明

**注入类型**
@Autowired：由Spring提供

@Inject：由JSR-330提供

@Resource：由JSR-250提供

3.java配置类相关注解

@Configuration 声明当前类为配置类，相当于xml形式的Spring配置（类上）

@Bean 注解在方法上，声明当前方法的返回值为一个bean，替代xml中的方式（方法上）

@Configuration 声明当前类为配置类，其中内部组合了@Component注解，表明这个类是一个bean（类上）

@ComponentScan 用于对Component进行扫描，相当于xml中的（类上）

@WishlyConfiguration 为@Configuration与@ComponentScan的组合注解，可以替代这两个注解

4.切面（AOP）相关注解

Spring支持AspectJ的注解式切面编程。

@Aspect 声明一个切面（类上）
使用@After、@Before、@Around定义建言（advice），可直接将拦截规则（切点）作为参数。

@After 在方法执行之后执行（方法上）
@Before 在方法执行之前执行（方法上）
@Around 在方法执行之前与之后执行（方法上）

@PointCut 声明切点
在java配置类中使用@EnableAspectJAutoProxy注解开启Spring对AspectJ代理的支持（类上）

5.@Bean的属性支持

@Scope 设置Spring容器如何新建Bean实例（方法上，得有@Bean）
其设置类型包括：

Singleton （单例,一个Spring容器中只有一个bean实例，默认模式）,
Protetype （每次调用新建一个bean）,
Request （web项目中，给每个http request新建一个bean）,
Session （web项目中，给每个http session新建一个bean）,
GlobalSession（给每一个 global http session新建一个Bean实例）

@StepScope 在Spring Batch中还有涉及

@PostConstruct 由JSR-250提供，在构造函数执行完之后执行，等价于xml配置文件中bean的initMethod

@PreDestory 由JSR-250提供，在Bean销毁之前执行，等价于xml配置文件中bean的destroyMethod

6.@Value注解

@Value 为属性注入值（属性上）
支持如下方式的注入：
》注入普通字符

@Value("Michael Jackson")
String name;
》注入操作系统属性

@Value("#{systemProperties['os.name']}")
String osName;
》注入表达式结果

@Value("#{ T(java.lang.Math).random() * 100 }")
String randomNumber;
》注入其它bean属性

@Value("#{domeClass.name}")
String name;
》注入文件资源

@Value("classpath:com/hgs/hello/test.txt")
String Resource file;
》注入网站资源

@Value("http://www.cznovel.com")
Resource url;12
》注入配置文件

@Value("${book.name}")
String bookName;
注入配置使用方法：
① 编写配置文件（test.properties）

book.name=《三体》
② @PropertySource 加载配置文件(类上)

@PropertySource("classpath:com/hgs/hello/test/test.propertie")
③ 还需配置一个PropertySourcesPlaceholderConfigurer的bean。

7.环境切换

@Profile 通过设定Environment的ActiveProfiles来设定当前context需要使用的配置环境。（类或方法上）

@Conditional Spring4中可以使用此注解定义条件话的bean，通过实现Condition接口，并重写matches方法，从而决定该bean是否被实例化。（方法上）

8.异步相关

@EnableAsync 配置类中，通过此注解开启对异步任务的支持，叙事性AsyncConfigurer接口（类上）

@Async 在实际执行的bean方法使用该注解来申明其是一个异步任务（方法上或类上所有的方法都将异步，需要@EnableAsync开启异步任务）

9.定时任务相关

@EnableScheduling 在配置类上使用，开启计划任务的支持（类上）

@Scheduled 来申明这是一个任务，包括cron,fixDelay,fixRate等类型（方法上，需先开启计划任务的支持）

10.@Enable*注解说明

这些注解主要用来开启对xxx的支持。
@EnableAspectJAutoProxy 开启对AspectJ自动代理的支持

@EnableAsync 开启异步方法的支持

@EnableScheduling 开启计划任务的支持

@EnableWebMvc 开启Web MVC的配置支持

@EnableConfigurationProperties 开启对@ConfigurationProperties注解配置Bean的支持

@EnableJpaRepositories 开启对SpringData JPA Repository的支持

@EnableTransactionManagement 开启注解式事务的支持

@EnableTransactionManagement 开启注解式事务的支持

@EnableCaching 开启注解式的缓存支持

11.测试相关注解

@RunWith 运行器，Spring中通常用于对JUnit的支持

@RunWith(SpringJUnit4ClassRunner.class)1
@ContextConfiguration 用来加载配置ApplicationContext，其中classes属性用来加载配置类

@ContextConfiguration(classes={TestConfig.class})1
SpringMVC部分

@EnableWebMvc 在配置类中开启Web MVC的配置支持，如一些ViewResolver或者MessageConverter等，若无此句，重写WebMvcConfigurerAdapter方法（用于对SpringMVC的配置）。

@Controller 声明该类为SpringMVC中的Controller

@RequestMapping 用于映射Web请求，包括访问路径和参数（类或方法上）

@ResponseBody 支持将返回值放在response内，而不是一个页面，通常用户返回json数据（返回值旁或方法上）

@RequestBody 允许request的参数在request体中，而不是在直接连接在地址后面。（放在参数前）

@PathVariable 用于接收路径参数，比如@RequestMapping(“/hello/{name}”)申明的路径，将注解放在参数中前，即可获取该值，通常作为Restful的接口实现方法。

@RestController 该注解为一个组合注解，相当于@Controller和@ResponseBody的组合，注解在类上，意味着，该Controller的所有方法都默认加上了@ResponseBody。

@ControllerAdvice 通过该注解，我们可以将对于控制器的全局配置放置在同一个位置，注解了@Controller的类的方法可使用@ExceptionHandler、@InitBinder、@ModelAttribute注解到方法上，
这对所有注解了 @RequestMapping的控制器内的方法有效。

@ExceptionHandler 用于全局处理控制器里的异常

@InitBinder 用来设置WebDataBinder，WebDataBinder用来自动绑定前台请求参数到Model中。

@ModelAttribute 本来的作用是绑定键值对到Model里，在@ControllerAdvice中是让全局的@RequestMapping都能获得在此处设置的键值对。