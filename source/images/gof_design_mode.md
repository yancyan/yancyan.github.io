

## 简单工厂模式(静态工厂方法)
简单工厂模式是类的创建模式，又叫做静态工厂方法（Static Factory Method）模式。简单工厂模式是由一个工厂对象决定创建出哪一种产品类的实例。

### 简单工厂模式使用场景
拿登录功能来说，假如应用系统需要支持多种登录方式如：口令认证、域认证（口令认证通常是去数据库中验证用户，而域认证则是需要到微软的域中验证用户）。那么自然的做法就是建立一个各种登录方式都适用的接口
```java
public interface Login {
    //登录验证
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
            throw new RuntimeException("没有找到登录类型");
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


