---
layout: post
title: '设计模式之访问者模式'
subtitle: '访问者模式'
date: 2017-10-14
categories: 设计模式
author: yates
cover: ''
tags: 设计模式
---

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