---
title: 桥接模式
date: 2021-03-16 21:38:00
author: yancyan
categories: 设计模式
tags:
- 设计模式
---

桥梁模式是对象的结构模式。又称为柄体(Handle and Body)模式或接口(Interface)模式。桥梁模式的用意是“将抽象化(Abstraction)与实现化(Implementation)脱耦，使得二者可以独立地变化”。

- 抽象化
从众多的事物中抽取出共同的、本质性的特征，而舍弃其非本质的特征，就是抽象化。共同特征是指那些能把一类事物与他类事物区分开来的特征，这些具有区分作用的特征又称本质特征。在抽象时，同与不同，决定于从什么角度上来抽象。抽象的角度取决于分析问题的目的。

- 实现化
抽象化给出的具体实现，就是实现化。

- 脱耦
所谓耦合，就是两个实体的行为的某种强关联。脱耦是指将它们之间的强关联改换成弱关联。 强关联，就是在编译时期已经确定的，无法在运行时期动态改变的关联；所谓弱关联，就是可以动态地确定并且可以在运行时期动态地改变的关联。显然，在Java语言中，继承关系是强关联，而聚合关系是弱关联。因此，桥梁模式中的所谓脱耦，就是指在一个软件系统的抽象化和实现化之间使用聚合关系而不是继承关系。

![](/images/bridge_01.png)

可以看出，这个系统含有两个等级结构：
1. 由抽象化角色和修正抽象化角色组成的抽象化等级结构。
2. 由实现化角色和两个具体实现化角色所组成的实现化等级结构。

桥梁模式所涉及的角色有：
- 抽象化(Abstraction)角色：抽象化给出的定义，并保存一个对实现化对象的引用。
- 修正抽象化(RefinedAbstraction)角色：扩展抽象化角色，改变和修正父类对抽象化的定义。
- 实现化(Implementor)角色：这个角色给出实现化角色的接口，但不给出具体的实现。必须指出的是，这个接口不一定和抽象化角色的接口定义相同，实际上，这两个接口可以非常不一样。实现化角色应当只给出底层操作，而抽象化角色应当只给出基于底层操作的更高一层的操作。
- 具体实现化(ConcreteImplementor)角色：这个角色给出实现化角色接口的具体实现。

```java
// 抽象化角色类
public abstract class Abstraction {
    
    protected Implementor impl;
    
    public Abstraction(Implementor impl){
        this.impl = impl;
    }
    //示例方法
    public void operation(){
        
        impl.operationImpl();
    }
}
// 修正抽象化角色
public class RefinedAbstraction extends Abstraction {

    public RefinedAbstraction(Implementor impl) {
        super(impl);
    }
    //其他的操作方法
    public void otherOperation(){

    }
}

// 实现化角色
public abstract class Implementor {
    /**
     * 示例方法，实现抽象部分需要的某些具体功能
     */
    public abstract void operationImpl();
}

// 具体实现化角色
public class ConcreteImplementorA extends Implementor {

    @Override
    public void operationImpl() {
        //具体操作
    }

}
public class ConcreteImplementorB extends Implementor {

    @Override
    public void operationImpl() {
        //具体操作
    }

}
```