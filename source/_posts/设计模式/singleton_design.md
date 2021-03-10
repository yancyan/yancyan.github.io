---
title: 单例设计模式
date: 2021-03-10 21:38:00
author: yancyan
categories: 虚拟机
tags:
- 设计模式
- 单例模式
---

## Singleton
单例模式：一个类有且仅有一个实例
### 饿汉式
```java
public class SingleTon{
private static SingleTon singleton = new SingleTon();
private SingleTon(){
}
public static SingleTon getInstance(){
    return singleton;
}}
```
### 懒汉式（**线程不安全**）
```java
public class SimpleSingleTon {
    private static SimpleSingleTon singleton = null;
    private SimpleSingleTon(){
    }
    public static SimpleSingleTon getInstance(){
        if (singleton == null){
            singleton = new SimpleSingleTon();
        }
        return singleton;
    }
}
```
加入同步改造，依然线程不安全，由于指令重排的问题，可能导致其他线程获取到未进行初始化的实例对象.
```java
public class SimpleSingleTon {
    private static SimpleSingleTon singleton = null;
    private SimpleSingleTon(){
    }
    public static  SimpleSingleTon getInstance(){
        if (singleton == null){
            synchronized(SimpleSingleTon.class){
                singleton = new SimpleSingleTon();
            }
        }
        return singleton;
    }
}
```
双重检测方案：使用`volatile`的内存屏障禁止指令重排序,
```java
public class SimpleSingleTon {
    private volatile static SimpleSingleTon singleton = null;
    private SimpleSingleTon(){
    }
    public static  SimpleSingleTon getInstance(){
        if (singleton == null){
            synchronized(SimpleSingleTon.class){
                if (singleton == null){
                    singleton = new SimpleSingleTon();
                } 
            }
        }
        return singleton;
    }
}
```

### 静态内部类
```java
public class SimpleSingleTon {
    private SimpleSingleTon(){
    }
    private static class LazySingleTon{
        private static  final  SimpleSingleTon INSTANCE = new SimpleSingleTon();
    }
    public static SimpleSingleTon getInstance(){
       return LazySingleTon.INSTANCE;
    }
}
```

### 枚举
```java
public enum Type {
    HIGH,
    LOW;
}
```
通过`javap -c Type.class`
```text
public final class com.web.mvc.test.Type extends java.lang.Enum<com.web.mvc.test.Type> {
  public static final com.web.mvc.test.Type HIGH;

  public static final com.web.mvc.test.Type LOW;

  public static com.web.mvc.test.Type[] values();
    Code:
       0: getstatic     #1                  // Field $VALUES:[Lcom/web/mvc/test/Type;
       3: invokevirtual #2                  // Method "[Lcom/web/mvc/test/Type;".clone:()Ljava/lang/Object;
       6: checkcast     #3                  // class "[Lcom/web/mvc/test/Type;"
       9: areturn

  public static com.web.mvc.test.Type valueOf(java.lang.String);
    Code:
       0: ldc           #4                  // class com/web/mvc/test/Type
       2: aload_0
       3: invokestatic  #5                  // Method java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/St
ring;)Ljava/lang/Enum;
       6: checkcast     #4                  // class com/web/mvc/test/Type
       9: areturn

  static {};
    Code:
       0: new           #4                  // class com/web/mvc/test/Type
       3: dup
       4: ldc           #7                  // String HIGH
       6: iconst_0
       7: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
      10: putstatic     #9                  // Field HIGH:Lcom/web/mvc/test/Type;
      13: new           #4                  // class com/web/mvc/test/Type
      16: dup
      17: ldc           #10                 // String LOW
      19: iconst_1
      20: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
      23: putstatic     #11                 // Field LOW:Lcom/web/mvc/test/Type;
      26: iconst_2
      27: anewarray     #4                  // class com/web/mvc/test/Type
      30: dup
      31: iconst_0
      32: getstatic     #9                  // Field HIGH:Lcom/web/mvc/test/Type;
      35: aastore
      36: dup
      37: iconst_1
      38: getstatic     #11                 // Field LOW:Lcom/web/mvc/test/Type;
      41: aastore
      42: putstatic     #1                  // Field $VALUES:[Lcom/web/mvc/test/Type;
      45: return
}
```
可见枚举类也是通过静态特性帮助进行初始化属性值