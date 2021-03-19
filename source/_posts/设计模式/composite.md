---
title: 合成模式
date: 2021-03-19 20:12:00
author: yancyan
categories: 设计模式
tags:
- 设计模式
---

合成模式属于对象的结构模式，有时又叫做“部分——整体”模式。合成模式将对象组织到树结构中，可以用来描述整体与部分的关系。合成模式可以使客户端将单纯元素与复合元素同等看待。

![](/images/composite_01.png)


从图中可以看出，文件系统是一个树结构，树上长有节点。树的节点有两种，一种是树枝节点，即目录，有内部树结构，在图中涂有颜色；另一种是文件，即树叶节点，没有内部树结构。

显然，可以把目录和文件当做同一种对象同等对待和处理，这也就是合成模式的应用。

合成模式可以不提供父对象的管理方法，但是合成模式必须在合适的地方提供子对象的管理方法，诸如：add()、remove()、以及getChild()等。

合成模式的实现根据所实现接口的区别分为两种形式，分别称为安全式和透明式。

## 安全式
安全的合成模式要求管理聚集的方法只出现在树枝构件类中，而不出现在树叶构件类中。
![](/images/composite_02.png)

这种形式涉及到三个角色：
1. 抽象构件(Component)角色：这是一个抽象角色，它给参加组合的对象定义出公共的接口及其默认行为，可以用来管理所有的子对象。合成对象通常把它所包含的子对象当做类型为Component的对象。在安全式的合成模式里，构件角色并不定义出管理子对象的方法，这一定义由树枝构件对象给出。
2. 树叶构件(Leaf)角色：树叶对象是没有下级子对象的对象，定义出参加组合的原始对象的行为。
3. 树枝构件(Composite)角色：代表参加组合的有下级子对象的对象。树枝构件类给出所有的管理子对象的方法，如add()、remove()以及getChild()。

```java
public interface Component {
    /**
     * 输出组建自身的名称
     */
    public void printStruct(String preStr);
}
// 树枝构件角色
public class Composite implements Component {
    private List<Component> childComponents = new ArrayList<Component>();
    private String name;
    public Composite(String name){
        this.name = name;
    }
    public void addChild(Component child){
        childComponents.add(child);
    }
    public void removeChild(int index){
        childComponents.remove(index);
    }
    public List<Component> getChild(){
        return childComponents;
    }
    @Override
    public void printStruct(String preStr) {
        System.out.println(preStr + "+" + this.name);
        if(this.childComponents != null){
            preStr += "  ";
            for(Component c : childComponents){
                c.printStruct(preStr);
            }
        }
    }
}
// 树叶构件角色
public class Leaf implements Component {
    private String name;
    public Leaf(String name){
        this.name = name;
    }
    @Override
    public void printStruct(String preStr) {
        // TODO Auto-generated method stub
        System.out.println(preStr + "-" + name);
    }
}
```
客户端
```java
public class Client {
    public static void main(String[]args){
        Composite root = new Composite("服装");
        Composite c1 = new Composite("男装");
        Composite c2 = new Composite("女装");
        
        Leaf leaf1 = new Leaf("衬衫");
        Leaf leaf2 = new Leaf("夹克");
        Leaf leaf3 = new Leaf("裙子");
        Leaf leaf4 = new Leaf("套装");
        
        root.addChild(c1);
        root.addChild(c2);
        c1.addChild(leaf1);
        c1.addChild(leaf2);
        c2.addChild(leaf3);
        c2.addChild(leaf4);
        
        root.printStruct("");
    }
}
```
树枝构件类(Composite)给出了addChild()、removeChild()以及getChild()等方法的声明和实现，而树叶构件类则没有给出这些方法的声明或实现。这样的做法是安全的做法，由于这个特点，客户端应用程序不可能错误地调用树叶构件的聚集方法，因为树叶构件没有这些方法，调用会导致编译错误。

## 透明式
透明式的合成模式要求所有的具体构件类，不论树枝构件还是树叶构件，均符合一个固定接口。
![](/images/composite_03.png)

```java
// 抽象构件角色类
public abstract class Component {
    public abstract void printStruct(String preStr);
    public void addChild(Component child){
        throw new UnsupportedOperationException("对象不支持此功能");
    }
    public void removeChild(int index){
        throw new UnsupportedOperationException("对象不支持此功能");
    }
    public List<Component> getChild(){
        throw new UnsupportedOperationException("对象不支持此功能");
    }
}

// 树枝构件角色类，此类将**实现改为继承**，其他地方无变化。
public class Composite extends Component {
    private List<Component> childComponents = new ArrayList<Component>();
    private String name;
    public Composite(String name){
        this.name = name;
    }
    public void addChild(Component child){
        childComponents.add(child);
    }
    public void removeChild(int index){
        childComponents.remove(index);
    }
    public List<Component> getChild(){
        return childComponents;
    }
    @Override
    public void printStruct(String preStr) {
        System.out.println(preStr + "+" + this.name);
        if(this.childComponents != null){
            preStr += "  ";
            for(Component c : childComponents){
                c.printStruct(preStr);
            }
        }
    }

}

// 树叶构件角色类，此类将**实现改为继承**，其他地方无变化。
public class Leaf extends Component {
    private String name;
    public Leaf(String name){
        this.name = name;
    }
    @Override
    public void printStruct(String preStr) {
        // TODO Auto-generated method stub
        System.out.println(preStr + "-" + name);
    }
}
```
Client调用, 不再区分Composite对象和Leaf对象
```java
public class Client {
    public static void main(String[]args){
        Component root = new Composite("服装");
        Component c1 = new Composite("男装");
        Component c2 = new Composite("女装");
        
        Component leaf1 = new Leaf("衬衫");
        Component leaf2 = new Leaf("夹克");
        Component leaf3 = new Leaf("裙子");
        Component leaf4 = new Leaf("套装");
        
        root.addChild(c1);
        root.addChild(c2);
        c1.addChild(leaf1);
        c1.addChild(leaf2);
        c2.addChild(leaf3);
        c2.addChild(leaf4);
        
        root.printStruct("");
    }
}
```

## 两种实现方法的选择
这里所说的安全性合成模式是指：从客户端使用合成模式上看是否更安全，如果是安全的，那么就不会有发生误操作的可能，能访问的方法都是被支持的。

这里所说的透明性合成模式是指：从客户端使用合成模式上，是否需要区分到底是“树枝对象”还是“树叶对象”。如果是透明的，那就不用区分，对于客户而言，都是Compoent对象，具体的类型对于客户端而言是透明的，是无须关心的。