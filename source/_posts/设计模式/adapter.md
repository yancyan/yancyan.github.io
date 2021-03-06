---
title: 适配器模式
date: 2021-03-16 21:38:00
author: yancyan
categories: 设计模式
tags:
- 设计模式
---

## 介绍
适配器模式把一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作。 适配器模式有类的适配器模式和对象的适配器模式两种不同的形式。

## 类适配器模式
类的适配器模式把适配的类的API转换成为目标类的API。

![](/images/adaptor_01.png)

在上图中可以看出，Adaptee类并没有sampleOperation2()方法，而客户端则期待这个方法。为使客户端能够使用Adaptee类，提供一个中间环节，即类Adapter，把Adaptee的API与Target类的API衔接起来。Adapter与Adaptee是继承关系，这决定了这个适配器模式是类的：

模式所涉及的角色有：

- 目标(Target)角色：这就是所期待得到的接口。注意：由于这里讨论的是类适配器模式，因此目标不可以是类。
- 源(Adaptee)角色：现在需要适配的接口。
- 适配器(Adapter)角色：适配器类是本模式的核心。适配器把源接口转换成目标接口。显然，这一角色不可以是接口，而必须是具体类。

示例代码
```java
public interface Target {
  public void sampleOperation1();
  public void sampleOperation2();
}

public class Adaptee {
  public void sampleOperation1(){}
}

public class Adapter extends Adaptee implements Target {
  /**
   * 由于源类Adaptee没有方法sampleOperation2()
   * 因此适配器补充上这个方法
   */
  @Override
  public void sampleOperation2() {
    //写相关的代码
  }
}
```

## 对象适配器模式
与类的适配器模式一样，对象的适配器模式把被适配的类的API转换成为目标类的API，与类的适配器模式不同的是，对象的适配器模式不是使用继承关系连接到Adaptee类，而是使用委派关系连接到Adaptee类。

![](/images/adaptor_02.png)

示例代码
```java
 public interface Target {
  public void sampleOperation1();
  public void sampleOperation2();
}
public class Adaptee {
  public void sampleOperation1(){}
}
public class Adapter {
  private Adaptee adaptee;
  public Adapter(Adaptee adaptee){
    this.adaptee = adaptee;
  }
  public void sampleOperation1(){
    this.adaptee.sampleOperation1();
  }
  public void sampleOperation2(){
    //写相关的代码
  }
}
```

## 类适配器和对象适配器的权衡
1. 类适配器使用对象继承的方式，是静态的定义方式；而对象适配器使用对象组合的方式，是动态组合的方式。
2. 对于类适配器，由于适配器直接继承了Adaptee，使得适配器不能和Adaptee的子类一起工作。
3. 对于对象适配器，一个适配器可以把多种不同的源适配到同一个目标。换言之，同一个适配器可以把源类和它的子类都适配到目标接口。
4. 对于类适配器，适配器可以重定义Adaptee的部分行为，相当于子类覆盖父类的部分实现方法。
5. 对于对象适配器，要重定义Adaptee的行为比较困难，这种情况下，需要定义Adaptee的子类来实现重定义，然后让适配器组合子类。虽然重定义Adaptee的行为比较困难，但是想要增加一些新的行为则方便的很，而且新增加的行为可同时适用于所有的源。
6. 对于类适配器，仅仅引入了一个对象，并不需要额外的引用来间接得到Adaptee。 对于对象适配器，需要额外的引用来间接得到Adaptee。

> 建议尽量使用对象适配器的实现方式，多用合成/聚合、少用继承。当然，具体问题具体分析，根据需要来选用实现方式，最适合的才是最好的。

## 适配器模式的优点
- 更好的复用性
  系统需要使用现有的类，而此类的接口不符合系统的需要。那么通过适配器模式就可以让这些功能得到更好的复用。
- 更好的扩展性
  在实现适配器功能的时候，可以调用自己开发的功能，从而自然地扩展系统的功能。

## 适配器模式的缺点
过多的使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是A接口，其实内部被适配成了B接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。