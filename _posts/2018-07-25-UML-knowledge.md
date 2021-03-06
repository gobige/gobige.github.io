---
layout: post
title: 'UML相关知识'
subtitle: 'UML相关知识'
date: 2018-07-25
categories: UML
author: yates
cover: ''
tags: UML
---
 
#### UML图

**用例图**
用例图从用户角度描述系统功能，并指出各功能的操作者。
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/1.png) 

**类图**
类图使我们更直观的了解一个系统的体系结构。类图可以图形化的描述一个系统的设计部分
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/2.png) 

**对象图**
对象图时类图的实例，不同之处在于对象图显示类的多个对象实例，且只能在系统某一时间段存在

**状态图**
状态图描述一个实体基于事件反应的动态行为，显示了该实体如何根据当前所处状态对不同的事件作出反应的。
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/3.png) 

**活动图**
活动图记录单个操作或方法的逻辑，或单个业务流程的逻辑。
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/4.png) 

**顺序图**
顺序图描述对象之间动态的交互关系，主要体现对象之间进行消息传递的时间顺序。
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/5.png)  

**协作图**
协作图用于显示组件及其交互关系的空间组织结构，它并不侧重于交互的顺序。
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/6.png) 

**组件图**
组件图描述代码部件的物理结构及各部件之间的依赖关系，组件图有助于分析和理解部件之间的相互影响程度。
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/7.png) 

**部署图**
部署图描述系统中硬件和软件的物理配置情况和系统体系结构
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/8.png) 

#### 常用UML开发工具

**rational rose**
功能强大，作图，数据库设计，UML建模，生成代码等

**visio**
主要是作图，流程，系统，复杂信息可视化，UML支持很少

**powerDesigner**
建模，数据库建模支持强大，作图偏弱，UML支持，分析，生成代码

**StarUML**
生成类图和其他类型的统一建模UML图表工具，支持双向工程，可导入rose文件

#### 用例和用例图

**包含关系**：两个用例之间，一个用例行为包含另一个用例的行为
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/9.png) 
**扩展关系**：扩展用例对基本用例的扩展，没有自用力参与，也可完成一个完整的功能
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/10.png) 
**泛化关系**：指一般和特殊的关系。多个用例共同拥有一种类似的结构和行为，共性抽象为付永利
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/11.png) 
**分组关系**：多个用例组织起来

#### 类图和对象图

**类图**
主要包括名称，属性，操作，静态
**对象图**
参与交互的各个对象在交互过程中某一时刻的状态
**接口**
描述类的部分行为的一组操作，也是一个类提供给另一个类的一组操作，抽象操作，用于标识说明行为。
**抽象方法**
包含一种或多种抽象方法的类，其他类可对它进行扩充。

#### 类之间关系
**依赖关系**
表示两个或多个模型元素之间，其中一个元素的某些改变可能会影响或提供消息给其他元素。

- 使用依赖。使用者使用提供者所提供的服务实现它的行为。
- 抽象依赖。使用者和提供者之间关系。
    - 跟踪依赖。不同模型中元素之间的连接。
    - 精化依赖。两个不同语义层次上元素之间映射。
    - 派生依赖。一个实例可以从另一个实例中导出。
- 授权依赖。表示一个事物访问另一个事物能力。
    - 访问依赖。一个包访问另一个包的内容
    - 导入依赖。一个包访问另一个包内容并被访问包的组成部分增加别名
    - 友元依赖。一个元素访问另一个元素，不管被访问元素是否具有可见性
- 绑定依赖。表明对目标模板使用给定实际参数进行实例化。

**泛化关系**。一种存在于一般元素和特殊元素之间的分类。
**关联关系**。指明一个事物的对象与另一个事物的对象之间的结构联系。
**实现关系**。用于将一种模型元素与另一种模型元素连接起来。泛化也是连接，泛化时同一语义层元素连接；实现是将不同语义层内元素连接起来。

#### 顺序图和协作图
**顺序图**
强调消息时间顺序的交互图，描述对象之间传递消息的时间顺序，表示用例中行为的顺序。

- 角色。可以是人或其他系统或子系统
- 对象。和类图对象定义一致。
- 生命线。代表顺序图中对象在一段时间内的存在。
- 激活器。代表顺序图中对象执行一项操作的时期，表示时间段的符号。
- 消息。对象之间某种形式的通信，垂直生命线之间，带有箭头的线并附以消息表达式方式表示。
    - 同步消息 发送者发送一个消息且接收者做好准备接收这个消息
    - 异步消息 不管接收者是否做好了接收准备都发送的消息
    - 返回消息 表示从过程调用返回

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/12.png) 

**协作图**
用于描述系统的行为，描述如何由系统成分协作实现的图。

- 活动者。活动者发出主动操作的对象，负责发送初始消息，启动一个操作。
- 对象。类的实例，负责发送和接收消息，一个协作代表了为了完成某个目标而共同工作的一组对象。
- 链接。表示两个对象共享一个消息，位于对象之间或参与者与对象之间。
- 消息。与顺序图中消息基本类似。
    - 序列化。消息前面添加序列号，消息按照执行顺序排序。
    - 控制点条件。根据消息表达式计算结果来限制消息的发送。
    - 创建实例。用来在协作图中创建对象实例
    - 发送多对象的消息。
    - 返回结果。要求某个对象进行计算并返回结果的值。
    - 构造型。在现有的UML元素的基础上创建新的元素。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/13.png) 

顺序图和协作图相同点：都是用对象和消息描述系统的动态行为的。表示出对象间的交互作用，规定了发送，接收对象的责任，支持所有消息类型，可相互转换
顺序图和协作图区别：顺序图清楚表示了交互作用中的事件顺序，但没有明确表示对象间关系。顺序图可反映对象生命周期，协作图没有。协作图清楚表达了对象间关系。顺序图强调消息时间顺序交互，协作图强调发送和接收消息对象间组织结构交互。协作图能反映动作路径，消息必须有顺序号，顺序图没有。

#### 状态图和活动图
用于在UML中建立动态模型，描述系统随时间变化的行为，行为抽取系统瞬间值变化来描述。

**状态图**
通过建立类对象的生存周期模型描述对象随时间变化的动态行为

- 状态。定义对象在其生命周期中条件和状况。
    - 名称。讲一个状态和其他状态区分的文本字符串
    - 进入/退出动作。表示进入/退出这个状态所执行的动作。
    - 内部转换。在不退出状态情况下内部处理。
    - 子状态。嵌套另一个状态中的状态称为子状态。
        - 顺序子状态。
        - 并发子状态。
    - 延迟事件。是其处理过程被推迟的事件。
- 转换。对象状态之间的转换，包括事件和动作
    - 源状态。对象在被激发前所处状态
    - 触发事件。引起转换的事件，诱因。是信号，事件，条件变化或时间表达式
    - 监护条件。转移触发事件发生时，对监护条件求值。
    - 动作。转移发生时，执行它对应的动作。
    - 目标状态。转移从一个状态到的另一个状态。
    
![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/14.png) 

**活动图**
用于描述活动流程，描述业务过程的工作流。一个过程或操作工作步骤。

- 动作状态。对象动作状态时最小单位。
    - 原子性。不能分解成更小的部分
    - 不可中断性。一旦开始就必须运行到结束
    - 瞬时性。动作状态所占用的极短处理时间。
- 活动状态。可分割的动作，可被分解成其他子活动或动作状态，可中断，占有有限时间。
- 转移。表示两个状态间一种关系，表示对象在当前状态中执行动作，并在某个特定事件发生或某个特定条件满足时进入后继状态。
- 分支。描述基于某个条件的可选择路径。
- 分叉和汇合。对运行时可能会存在两个或多个并发运行的控制流
- 泳道。将活动图中活动划分为若干组，并把每一组指定给负责这组活动的业务组织，对象。
- 对象流。包含对象间活动转移和动作的依赖关系。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/15.png) 

#### 组件图和部署图
**组件图**
表示组件类型的组织以及各组件之间依赖关系的图。

- 组件。时系统中遵从从一组接口且提供实现的一个物理部件，开发和运行时类的物理实现。比如程序源代码，子系统，动态链接库
    - 实施组件。DLL,EXE,ActiveX控件和javabean组件。
    - 工作产品组件。开发过程的产物，产生实施组件的源代码文件及数据文件，不直接参与可执行系统
    - 执行组件。作为一个正在执行的系统的结果创建。
    - 与类区别：类表示逻辑抽象，组件表示存在于计算机中物理抽象2类可以直接拥有属性，操作；组件仅拥有只能通过接口访问的操作。
- 接口。一组用于描述类或组件的一个服务的操作，是一个被命名的操作的集合。
    - 导出接口。为其他组件提供服务的接口
    - 导入接口。在组建中用到其他组件提供的接口
- 关系。代表事物之间的联系。
    - 依赖关系。组件依赖外部提供服务
    - 实现关系。组件向外提供的服务。

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/15.png) 

**部署图**
描述系统硬件的物理拓扑结构以及在此结构上运行的软件。表示运行时过程节点结构，组件实例及对象结构的图。

- 节点。存在于运行时并代表一项计算资源的物理元素至少拥有一些内存能力，通常拥有处理能力。
- 组件。表示上面所说的组件，组件是被节点执行的事物，表示逻辑元素的物理模块，节点表示组件物理部署
- 关系
    - 依赖
    - 泛化
    - 关联
    - 实现

![此处输入图片的描述](http://yatesblog.oss-cn-shenzhen.aliyuncs.com/img/uml/17.png) 