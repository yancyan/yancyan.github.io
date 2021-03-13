---
title: JIT - 方法内联
date: 2021-03-10 22:38:00
author: yancyan
categories: 虚拟机
tags:
- 编译优化
---

## 介绍
先通过一段代码，体会编译优化的手段

```java
 // 原始代码
 static class B{
        int value;
        final int get() {
            return value;
        }
    }
    public void foo() {
        y = b.get();
        // ... do stuff...
        z = b.get();
        sum = y + z;
    }
```
方法内联的主要目的有两个
- 去除方法调用的成本（查找方法版本，建立栈帧等）
- 为其他优化建立良好的基础

方法内联膨胀以后可以更大范围进行后续优化，因此各编译器也会优先进行方法内联，上述代码内联后的代码如下
```java
// 方法内联
public void foo() {
    y = b.value;
    // ... do stuff...
    z = b.value;
    sum = y + z;
}
```
这一步进行了冗余访问消除，而如果中间代码不会改变`b.get()`的值，那么`z = b.value`可以直接替换称`z = y`，即为公共子表达式消除，优化以后的代码如下
```java
// 公共子表达式消除
public void foo() {
    y = b.value;
    // ... do stuff...
    z = y;
    sum = y + z;
}
```
这一步可以看到没有使用z变量，它与`y`变量相等，因此可以采用复写传播，代码如下
```java
// 复写传播
public void foo() {
    y = b.value;
    // ... do stuff...
    y = y;
    sum = y + y;
}
```
再进行无用代码消除，无用代码是可能永远不会被执行的代码，也可能是完全没有意义的代码，优化如下
```java
// 无用代码消除
public void foo() {
    y = b.value;
    // ... do stuff...
    sum = y + y;
}
```


## 方法内联
方法内联，就是在编译过程中遇到方法调用时，将目标方法纳入到编译范围内，并取代原方法调用的编译优化手段

内联优化的示例
```java
 public int  add(int a, int b , int c, int d){
        return add(a, b) + add(c, d);
        }

public int add(int a, int b){
        return a + b;
        }
```
内联之后,明显减少了二次调用函数带来的函数调用开销
```java
 public int  add(int a, int b , int c, int d){
        return add(a, b) + add(c, d);
        }

public int add(int a, int b){
        return a + b;
        }
```
## 方法内联条件
对于即时编译器来说，内联越多，编译时间就越长，且生成的机器码也越长。在JVM里，编译生成的机器码都被放到Code Cache中，这个区域是有大小限制的（XX:ReservedCodeCacheSize可调）。

所以方法内联不能无限制进行，需要满足特定条件

- 由 -XX:CompileCommand中的inline指令指定的方法和由@ForceInline（此注解仅限于JDK的内部方法）注解的方法，会被强制内联；对应的，由-XX:CompileCommand中的dontinline指令或者exclude指令指定的方法和由@DontInline注解的方法，则始终不会被内联。
- 如果调用字节码对应的符号引用未被解析、目标方法所在的类未初始化、目标方法是native方法，都将导致该方法无法内联。
- C2不支持内联超过9层的调用（-XX:MaxInlineLivel可调）以及一层的直接递归调用（-XX:MaxRecursiveInlineLevel可调）
- JIT将根据方法调用指令所在程序的路径的热度，目标方法的调用次数（-XX:CompileThreshold可调）和大小以及当前IR图的代销来决定方法调用能否内联。

来自《深入理解Java虚拟机》一书
> 只有使用`invokespecial`指令调用的私有方法、实例构造器、父类方法和使用`invokestatic`指令调用的静态方法才会在编译器进行解析（最多除去被final修饰的方法，尽管它使用invokevirtual指令，但也是非虚方法），其他Java方法都必须在运行时进行方法接收者的多态选择，简而言之，Java中的默认的实例方法都是虚方法。

## 去虚化
对于静态方法，JIT可以明确目标方法，但是对于需要动态绑定的虚方法调用，就需要对其进行去虚化，去虚化分为**完全去虚化**和**条件去虚化**。
### 完全去虚化

完全去虚化是通过类型推到或者类层次分析（CHA），识别虚方法调用的唯一目标方法，从而转换为直接调用的优化手段，它的关键在于证明虚方法调用的目标方法是唯一的。

- 基于类型推导的完全去虚化将通过数据流分析推到出调用者的动态类型，从而确定具体的目标方法。
- 基于类层次分析的完全去虚化是通过分析Java虚拟机中所有已被加载的类，判断某个抽象方法或者接口方法是否仅有一个实现。

JVM的为当前编译结果注册若干个假设，假设某抽象类只有一个子类，或某类没有子类等类似假设。在这之后，每当新的类被加载，JVM就会重新验证这些假设项，如果某个假设不再成立，那么JVM就会对其所属的编译结果进行去优化，退到解释执行的状态。事实上，即使方法是一个子类方法，即时编译器仍需为其添加假设，这是因为JVM并不能保证没有重写了方法的下级子类。所以，对于一些没有扩展子类必要的类，我们可以加上final关键字，这样JVM就不会再为其增加子类相关的假设，减少类加载时所需的验证内容。

如果向CHA查询出来的结果确实存在多个版本的目标方法，即使编译器还会尝试使用**内联缓存（InlineCache）**的方式缩减开销，

内联缓存是建立在目标方法正常入口之前的缓存，工作原理大致：

未调用，内联缓存状态为空，第一次调用，缓存记录下方法接受者的版本信息并每次调用都比较接收者版本，如果后续的版本信息都是一样的，这是就是**单态内联缓存（Monomorphic Inline Cache）**。

通过该缓存调用比用不内联的非虚方法调用仅多了一次类型判断的开销而已，但是如果真的虚方法的多态性，就退化成**超多态内联缓存（Megamorphic Inline Cache）**，其开销相当于真正查找虚方法表来进行方法分派。

### 条件去虚化

条件去虚化是通过向代码中添加若干个类型比较，将虚方法调用转换为若干个直接调用。即将调用者的动态类型，依次与JVM所搜集的类型Profile中记录的类型作比较，如果匹配，则直接调用该记录类型所对应的目标方法。

```java
public static int test(BinaryOp op) {
return op.apply(2, 1);
}
```
假设编译时类型Profile记录了调用者的两个类型Sub和Add，那么即时编译器可以根据此条件去虚化，依次比较调用者的动态类型是否为Sub或者Add，并内联相应的方法。如下是伪代码：
```java
public static int test(BinaryOp op) {
if (op.getClass() == Sub.class) {
return 2 - 1; // inlined Sub.apply
} else if (op.getClass() == Add.class) {
return 2 + 1; // inlined Add.apply
} else {
... // 当匹配不到类型Profile中的类型怎么办？
}
}
```
遍历类型Profile未匹配到调用者的动态类型，那么即时编译器有两种选择
- 类型Profile是完整的，也就是说，所有出现过的动态类型都被记录到类型Profile中了，那么即时编译器可以让程序去优化，重新收集类型Profile。
- 类型Profile是不完整的，也就是指某些出现过的动态类型未被记录到类型Profile中，那么重新收集也没有特别的作用，此时，即时编译器可以让程序进行原本的虚调用，通过内联缓存进行调用或者通过方法表进行动态绑定。（在C2中，如果类型Profile是不完整的，即时编译器不会进行条件去虚化，而是直接使用内联缓存或者方法表）