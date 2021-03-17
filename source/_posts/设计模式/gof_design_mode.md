---
title: 简单工厂&工厂方法&抽象工厂
date: 2021-03-17 18:38:00
author: yancyan
categories: 设计模式
tags:
- 设计模式
---
## 简单工厂模式
简单工厂模式是类的创建模式，又叫做静态工厂方法（Static Factory Method）模式。简单工厂模式是由一个工厂对象决定创建出哪一种产品类的实例。

### 简单工厂模式使用场景
拿登录功能来说，假如应用系统需要支持多种登录方式如：口令认证、域认证（口令认证通常是去数据库中验证用户，而域认证则是需要到微软的域中验证用户）。那么自然的做法就是建立一个各种登录方式都适用的接口
```java
public interface Login {
    public boolean verify(String name , String password);
}
public class DomainLogin implements Login {
    @Override
    public boolean verify(String name, String password) {
        // TODO Auto-generated method stub
        return true;
    }
}
public class PasswordLogin implements Login {
    @Override
    public boolean verify(String name, String password) {
        // TODO Auto-generated method stub
        return true;
    }
}

```
还需要一个工厂类LoginManager，根据调用者不同的要求，创建出不同的登录对象并返回。而如果碰到不合法的要求，会返回一个Runtime异常。
```java
public class LoginManager {
    public static Login factory(String type){
        if(type.equals("password")){
            return new PasswordLogin();
        }else if(type.equals("passcode")){
            return new DomainLogin();
        }else{
            throw new RuntimeException();
        }
    }
}
```
《JAVA与模式》一书中使用java.text.DataFormat类作为简单工厂模式的典型例子叙述。
### 简单工厂模式的优点
　　模式的核心是工厂类。这个类含有必要的逻辑判断，可以决定在什么时候创建哪一个登录验证类的实例，而调用者则可以免除直接创建对象的责任。简单工厂模式通过这种做法实现了对责任的分割，当系统引入新的登录方式的时候无需修改调用者。
### 简单工厂模式的缺点
　　这个工厂类集中了所以的创建逻辑，当有复杂的多层次等级结构时，所有的业务逻辑都在这个工厂类中实现。什么时候它不能工作了，整个系统都会受到影响。


## 工厂方法模式
工厂方法模式是类的创建模式，用意是定义一个创建产品对象的工厂接口，将实际创建工作推迟到子类中。
### 工厂方法模式使用场景
拿导出功能来说。如果系统需要支持对数据库中的员工薪资进行导出，支持多种格式（HTML、CSV、PDF等）且每种格式导出的结构有所不同（财务可能需要特定的格式方便核算）。

如果使用简单工厂模式，则工厂类必定过于臃肿。因为简单工厂模式只有一个工厂类，它需要处理所有的创建的逻辑。假如以上需求暂时只支持3种导出的格式以及2种导出的结构，那工厂类则需要6个if else来创建6种不同的类型。如果日后需求不断增加，则后果不堪设想。

　　这时候就需要工厂方法模式来处理以上需求。在工厂方法模式中，核心的工厂类不再负责所有的对象的创建，而是将具体创建的工作交给子类去做。这个核心类则摇身一变，成为了一个抽象工厂角色，仅负责给出具体工厂子类必须实现的接口，而不接触哪一个类应当被实例化这种细节。

这种进一步抽象化的结果，使这种工厂方法模式可以用来允许系统在不修改具体工厂角色的情况下引进新的产品，这一特点无疑使得工厂方法模式具有超过简单工厂模式的优越性。下面就针对以上需求设计UML图：

![](/images/factory_method_design_mode.png)


从上图可以看出，这个使用的工厂方法模式的系统涉及到以下角色：
- 抽象工厂（ExportFactory）角色：担任这个角色的是工厂方法模式的核心，任何在模式中创建对象的工厂类必须实现这个接口。在实际的系统中，这个角色也常常使用抽象类实现。
- 具体工厂（ExportHtmlFactory、ExportPdfFactory）角色：担任这个角色的是实现了抽象工厂接口的具体JAVA类。具体工厂角色含有与业务密切相关的逻辑，并且受到使用者的调用以创建导出类（如：ExportStandardHtmlFile）。
- 抽象导出（ExportFile）角色：工厂方法模式所创建的对象的超类，也就是所有导出类的共同父类或共同拥有的接口。在实际的系统中，这个角色也常常使用抽象类实现。

- 具体导出（ExportStandardHtmlFile等）角色：这个角色实现了抽象导出（ExportFile）角色所声明的接口，工厂方法模式所创建的每一个对象都是某个具体导出角色的实例。
```java
public interface ExportFactory {
    public ExportFile factory(String type);
}
public class ExportHtmlFactory implements ExportFactory{
    @Override
    public ExportFile factory(String type) {
        // TODO Auto-generated method stub
        if("standard".equals(type)){
            return new ExportStandardHtmlFile();
        }else if("financial".equals(type)){
            return new ExportFinancialHtmlFile();
        }else{
            throw new RuntimeException();
        }
    }
}
public class ExportPdfFactory implements ExportFactory {
    @Override
    public ExportFile factory(String type) {
        // TODO Auto-generated method stub
        if("standard".equals(type)){
            return new ExportStandardPdfFile();
        }else if("financial".equals(type)){
            return new ExportFinancialPdfFile();
        }else{
            throw new RuntimeException();
        }
    }
}
public interface ExportFile {
    public boolean export(String data);
}
public class ExportFinancialHtmlFile implements ExportFile{
    @Override
    public boolean export(String data) {
        // TODO Auto-generated method stub
        System.out.println("导出财务版HTML文件");
        return true;
    }
}
public class ExportFinancialPdfFile implements ExportFile{
    @Override
    public boolean export(String data) {
        // TODO Auto-generated method stub
        System.out.println("导出财务版PDF文件");
        return true;
    }
}
```
### 工厂方法和简单工厂模式
工厂方法模式和简单工厂模式在结构上的不同很明显。工厂方法模式的核心是一个抽象工厂类，而简单工厂模式把核心放在一个具体类上。
　　如果系统需要加入一个新的导出类型，那么所需要的就是向系统中加入一个这个导出类以及所对应的工厂类。没有必要修改客户端，也没有必要修改抽象工厂角色或者其他已有的具体工厂角色。对于增加新的导出类型而言，这个系统完全支持“开-闭原则”。



## 抽象工厂模式
抽象工厂模式与工厂方法模式的最大区别就在于，工厂方法模式针对的是一个产品等级结构；而抽象工厂模式则需要面对多个产品等级结构。

### 场景问题
　　举个生活中常见的例子——组装电脑，需要选择一系列的配件，为讨论使用简单点，只考虑选择CPU和主板的问题。

　　事实上，在选择CPU的时候，面临一系列的问题，比如品牌、型号、针脚数目、主频等问题，只有把这些问题都确定下来，才能确定具体的CPU。

　　同样，在选择主板的时候，也有一系列问题，比如品牌、芯片组、集成芯片、总线频率等问题，也只有这些都确定了，才能确定具体的主板。

　　在最终确定这个装机方案之前，还需要整体考虑各个配件之间的兼容性。比如：CPU和主板，如果使用Intel的CPU和AMD的主板是根本无法组装的。因为Intel的CPU针脚数与AMD主板提供的CPU插口不兼容，就是说如果使用Intel的CPU根本就插不到AMD的主板中，所以装机方案是整体性的，里面选择的各个配件之间是有关联的。

　　对于装机工程师而言，他只知道组装一台电脑，需要相应的配件，但是具体使用什么样的配件，还得由客户说了算。也就是说装机工程师只是负责组装，而客户负责选择装配所需要的具体的配件。因此，当装机工程师为不同的客户组装电脑时，只需要根据客户的装机方案，去获取相应的配件，然后组装即可。

#### 简单工厂模式的解决方案

考虑客户的功能，需要选择自己需要的CPU和主板，然后告诉装机工程师自己的选择，接下来就等着装机工程师组装电脑了。
对装机工程师而言，只是知道CPU和主板的接口，而不知道具体实现，很明显可以用上简单工厂模式或工厂方法模式。为了简单，这里选用简单工厂。客户告诉装机工程师自己的选择，然后装机工程师会通过相应的工厂去获取相应的实例对象。

![](/images/abstract_factory_simple_factory.png)


CPU接口与实现类
```java
public interface Cpu {
    public void calculate();
}
public class IntelCpu implements Cpu {
    /**CPU的针脚数*/
    private int pins = 0;
    public  IntelCpu(int pins){
        this.pins = pins;
    }
    @Override
    public void calculate() {
        // TODO Auto-generated method stub
        System.out.println("Intel CPU的针脚数：" + pins);
    }
}
public class AmdCpu implements Cpu {
    private int pins = 0;
    public  AmdCpu(int pins){
        this.pins = pins;
    }
    @Override
    public void calculate() {
        // TODO Auto-generated method stub
        System.out.println("AMD CPU的针脚数：" + pins);
    }
}
```
主板接口与具体实现
```java
public interface Mainboard {
    public void installCPU();
}
public class IntelMainboard implements Mainboard {
    /**
     * CPU插槽的孔数
     */
    private int cpuHoles = 0;
    /**
     * 构造方法，传入CPU插槽的孔数
     * @param cpuHoles
     */
    public IntelMainboard(int cpuHoles){
        this.cpuHoles = cpuHoles;
    }
    @Override
    public void installCPU() {
        // TODO Auto-generated method stub
        System.out.println("Intel主板的CPU插槽孔数是：" + cpuHoles);
    }

}
public class AmdMainboard implements Mainboard {
    /**
     * CPU插槽的孔数
     */
    private int cpuHoles = 0;
    /**
     * 构造方法，传入CPU插槽的孔数
     * @param cpuHoles
     */
    public AmdMainboard(int cpuHoles){
        this.cpuHoles = cpuHoles;
    }
    @Override
    public void installCPU() {
        // TODO Auto-generated method stub
        System.out.println("AMD主板的CPU插槽孔数是：" + cpuHoles);
    }
}
```
CPU与主板工厂类
```java
public class CpuFactory {
    public static Cpu createCpu(int type){
        Cpu cpu = null;
        if(type == 1){
            cpu = new IntelCpu(755);
        }else if(type == 2){
            cpu = new AmdCpu(938);
        }
        return cpu;
    }
}
public class MainboardFactory {
    public static Mainboard createMainboard(int type){
        Mainboard mainboard = null;
        if(type == 1){
            mainboard = new IntelMainboard(755);
        }else if(type == 2){
            mainboard = new AmdMainboard(938);
        }
        return mainboard;
    }
}
```
装机工程师类与客户类运行结果如下：
```java
public class ComputerEngineer {
    /**
     * 定义组装机需要的CPU
     */
    private Cpu cpu = null;
    /**
     * 定义组装机需要的主板
     */
    private Mainboard mainboard = null;
    public void makeComputer(int cpuType , int mainboard){
        /**
         * 组装机器的基本步骤
         */
        //1:首先准备好装机所需要的配件
        prepareHardwares(cpuType, mainboard);
        //2:组装机器
        //3:测试机器
        //4：交付客户
    }
    private void prepareHardwares(int cpuType , int mainboard){
        //这里要去准备CPU和主板的具体实现，为了示例简单，这里只准备这两个
        //可是，装机工程师并不知道如何去创建，怎么办呢？
        
        //直接找相应的工厂获取
        this.cpu = CpuFactory.createCpu(cpuType);
        this.mainboard = MainboardFactory.createMainboard(mainboard);
        
        //测试配件是否好用
        this.cpu.calculate();
        this.mainboard.installCPU();
    }
}
```
上面的实现，虽然通过简单工厂方法解决了：对于装机工程师，只知CPU和主板的接口，而不知道具体实现的问题。但还有一个问题没有解决，那就是这些CPU对象和主板对象其实是有关系的，需要相互匹配的。而上面的实现中，并没有维护这种关联关系，CPU和主板是由客户任意选择，这是有问题的。比如在客户端调用makeComputer时，传入参数为(1,2)，客户选择的是Intel的CPU针脚数为755，而选择的主板是AMD，主板上的CPU插孔是938，根本无法组装，这就是没有维护配件之间的关系造成的。该怎么解决这个问题呢？　

#### 引进抽象工厂模式解决
抽象工厂模式与工厂方法模式的最大区别就在于，工厂方法模式针对的是一个产品等级结构；而抽象工厂模式则需要面对多个产品等级结构。

两个重要的概念：产品族和产品等级

- 产品族：指位于不同产品等级结构中，功能相关联的产品组成的家族。比如AMD的主板、芯片组、CPU组成一个家族，Intel的主板、芯片组、CPU组成一个家族。而这两个家族都来自于三个产品等级：主板、芯片组、CPU。

![](/images/product_and_level.png)

上面所给出的三个不同的等级结构具有平行的结构。因此，如果采用工厂方法模式，势必要使用三个独立的工厂等级结构来对付这三个产品等级结构。由于这三个产品等级结构的相似性，会导致三个平行的工厂等级结构。随着产品等级结构的数目的增加，工厂方法模式所给出的工厂等级结构的数目也会随之增加。如下图：


![](/images/product_level_multi_factory.png)

是否可以使用同一个工厂等级结构来对付这些相同或者极为相似的产品等级结构呢？当然可以的，而且这就是抽象工厂模式的好处。同一个工厂等级结构负责三个不同产品等级结构中的产品对象的创建。

![](/images/product_level_multi_factory_abstract_factory.png)

可以看出，一个工厂等级结构可以创建出分属于不同产品等级结构的一个产品族中的所有对象。显然，这时候抽象工厂模式比简单工厂模式、工厂方法模式更有效率。对应于每一个产品族都有一个具体工厂。而每一个具体工厂负责创建属于同一个产品族，但是分属于不同等级结构的产品。


![](/images/abstract_factory_class_uml.png)


前面示例实现的CPU接口和CPU实现对象，主板接口和主板实现对象，都不需要变化。
前面示例中创建CPU的简单工厂和创建主板的简单工厂，都不再需要。
新加入的抽象工厂类和实现类
```java
public interface AbstractFactory {
    /**
     * 创建CPU对象
     * @return CPU对象
     */
    public Cpu createCpu();
    /**
     * 创建主板对象
     * @return 主板对象
     */
    public Mainboard createMainboard();
}
public class IntelFactory implements AbstractFactory {

    @Override
    public Cpu createCpu() {
        // TODO Auto-generated method stub
        return new IntelCpu(755);
    }

    @Override
    public Mainboard createMainboard() {
        // TODO Auto-generated method stub
        return new IntelMainboard(755);
    }

}

public class AmdFactory implements AbstractFactory {

    @Override
    public Cpu createCpu() {
        // TODO Auto-generated method stub
        return new IntelCpu(938);
    }

    @Override
    public Mainboard createMainboard() {
        // TODO Auto-generated method stub
        return new IntelMainboard(938);
    }

}
```
装机工程师类跟前面的实现相比，主要的变化是：从客户端不再传入选择CPU和主板的参数，而是直接传入客户已经选择好的产品对象。这样就避免了单独去选择CPU和主板所带来的兼容性问题，客户要选就是一套，就是一个系列。
 ```java
 public class ComputerEngineer {
    /**
     * 定义组装机需要的CPU
     */
    private Cpu cpu = null;
    /**
     * 定义组装机需要的主板
     */
    private Mainboard mainboard = null;
    public void makeComputer(AbstractFactory af){
        /**
         * 组装机器的基本步骤
         */
        //1:首先准备好装机所需要的配件
        prepareHardwares(af);
        //2:组装机器
        //3:测试机器
        //4：交付客户
    }
    private void prepareHardwares(AbstractFactory af){
        //这里要去准备CPU和主板的具体实现，为了示例简单，这里只准备这两个
        //可是，装机工程师并不知道如何去创建，怎么办呢？
        
        //直接找相应的工厂获取
        this.cpu = af.createCpu();
        this.mainboard = af.createMainboard();
        
        //测试配件是否好用
        this.cpu.calculate();
        this.mainboard.installCPU();
    }
}
```

抽象工厂的功能是为一系列相关对象或相互依赖的对象创建一个接口。一定要注意，这个接口内的方法不是任意堆砌的，而是一系列相关或相互依赖的方法。比如上面例子中的主板和CPU，都是为了组装一台电脑的相关对象。不同的装机方案，代表一种具体的电脑系列。

#### 在什么情况下应当使用抽象工厂模式
1. 一个系统不应当依赖于产品类实例如何被创建、组合和表达的细节，这对于所有形态的工厂模式都是重要的。
2. 这个系统的产品有多于一个的产品族，而系统只消费其中某一族的产品。
3. 同属于同一个产品族的产品是在一起使用的，这一约束必须在系统的设计中体现出来。（比如：Intel主板必须使用Intel CPU、Intel芯片组）
4. 系统提供一个产品类的库，所有的产品以同样的接口出现，从而使客户端不依赖于实现。


#### 抽象工厂模式的优点
- 分离接口和实现
    客户端使用抽象工厂来创建需要的对象，而客户端根本就不知道具体的实现是谁，客户端只是面向产品的接口编程而已。也就是说，客户端从具体的产品实现中解耦。
- 使切换产品族变得容易


#### 抽象工厂模式的缺点
- 不太容易扩展新的产品
    如果需要给整个产品族添加一个新的产品，那么就需要修改抽象工厂，这样就会导致修改所有的工厂实现类。

