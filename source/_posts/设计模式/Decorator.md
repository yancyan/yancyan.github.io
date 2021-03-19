---
title: 装饰器模式
date: 2021-03-19 20:12:00
author: yancyan
categories: 设计模式
tags:
- 设计模式
---

装饰模式又名包装(Wrapper)模式。装饰模式以对客户端透明的方式扩展对象的功能，是继承关系的一个替代方案。装饰模式以对客户透明的方式动态地给一个对象附加上更多的责任。装饰模式可以在不使用创造更多子类的情况下，将对象的功能加以扩展。

## 类结构
![](/images/decorator_01.png)

在装饰模式中的角色有：

- 抽象构件(Component)角色：给出一个抽象接口，以规范准备接收附加责任的对象。
- 具体构件(ConcreteComponent)角色：定义一个将要接收附加责任的类。
- 装饰(Decorator)角色：持有一个构件(Component)对象的实例，并定义一个与抽象构件接口一致的接口。
- 具体装饰(ConcreteDecorator)角色：负责给构件对象“贴上”附加的责任。

## 编程模型示例
```java
// 抽象构件角色
public interface Component {
    public void sampleOperation();
}

// 具体构件角色
public class ConcreteComponent implements Component {
    @Override
    public void sampleOperation() {
        // 写相关的业务代码
    }
}

// 装饰角色
public class Decorator implements Component{
    private Component component;
    public Decorator(Component component){
        this.component = component;
    }
    @Override
    public void sampleOperation() {
        // 委派给构件
        component.sampleOperation();
    }
}

// 具体装饰角色
public class ConcreteDecoratorA extends Decorator {
    public ConcreteDecoratorA(Component component) {
        super(component);
    }
    @Override
    public void sampleOperation() {
　　　　　super.sampleOperation();
        // 写相关的业务代码
    }
} 
public class ConcreteDecoratorB extends Decorator {
    public ConcreteDecoratorB(Component component) {
        super(component);
    }
    @Override
    public void sampleOperation() {
　　　　  super.sampleOperation();
        // 写相关的业务代码
    }
}
```

## 装饰模式简化
如果只有一个ConcreteComponent类，那么可以考虑去掉抽象的Component类（接口），把Decorator作为一个ConcreteComponent子类。如下图所示：
![](/images/decorator_02.png)

如果只有一个ConcreteDecorator类，那么就没有必要建立一个单独的Decorator类，而可以把Decorator和ConcreteDecorator的责任合并成一个类。甚至在只有两个ConcreteDecorator类的情况下，都可以这样做。如下图所示：
![](/images/decorator_03.png)

## 装饰模式的优缺点
### 装饰模式的优点
1. 装饰模式与继承关系的目的都是要扩展对象的功能，但是装饰模式可以提供比继承更多的灵活性。装饰模式允许系统动态决定“贴上”一个需要的“装饰”，或者除掉一个不需要的“装饰”。继承关系则不同，继承关系是静态的，它在系统运行前就决定了。
2. 通过使用不同的具体装饰类以及这些装饰类的排列组合，设计师可以创造出很多不同行为的组合。

### 装饰模式的缺点
由于使用装饰模式，可以比使用继承关系需要较少数目的类。使用较少的类，当然使设计比较易于进行。但是，在另一方面，使用装饰模式会产生比使用继承关系更多的对象。更多的对象会使得查错变得困难，特别是这些对象看上去都很相像。

## JAVA I/O库中的应用
Java I/O库需要很多性能的各种组合，如果这些性能都是用继承的方法实现的，那么每一种组合都需要一个类，这样就会造成大量性能重复的类出现。而如果采用装饰模式，那么类的数目就会大大减少，性能的重复也可以减至最少。因此装饰模式是Java I/O库的基本模式。

Java I/O库的对象结构图如下，由于Java I/O的对象众多，因此只画出InputStream的部分。
![](/images/decorator_04.png)

根据上图可以看出：

- 抽象构件(Component)角色：由InputStream扮演。这是一个抽象类，为各种子类型提供统一的接口。

- 具体构件(ConcreteComponent)角色：由ByteArrayInputStream、FileInputStream、PipedInputStream、StringBufferInputStream等类扮演。它们实现了抽象构件角色所规定的接口。

- 抽象装饰(Decorator)角色：由FilterInputStream扮演。它实现了InputStream所规定的接口。

- 具体装饰(ConcreteDecorator)角色：由几个类扮演，分别是BufferedInputStream、DataInputStream以及两个不常用到的类LineNumberInputStream、PushbackInputStream。

## 半透明的装饰模式
装饰模式和适配器模式都是“包装模式(Wrapper Pattern)”，它们都是通过封装其他对象达到设计的目的的，但是它们的形态有很大区别。

理想的装饰模式在对被装饰对象进行功能增强的同时，要求具体构件角色、装饰角色的接口与抽象构件角色的接口完全一致。而适配器模式则不然，一般而言，适配器模式并不要求对源对象的功能进行增强，但是会改变源对象的接口，以便和目标接口相符合。

装饰模式有透明和半透明两种，这两种的区别就在于装饰角色的接口与抽象构件角色的接口是否完全一致。透明的装饰模式也就是理想的装饰模式，要求具体构件角色、装饰角色的接口与抽象构件角色的接口完全一致。相反，如果装饰角色的接口与抽象构件角色接口不一致，**也就是说装饰角色的接口比抽象构件角色的接口宽的话，装饰角色实际上已经成了一个适配器角色**，这种装饰模式也是可以接受的，称为“半透明”的装饰模式


