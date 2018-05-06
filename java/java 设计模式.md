# java 23种设计模式 #

设计模式是指针对软件开发过程中重复发生的问题的解决办法。使用设计模式是为了可重用代码，让代码更容易被他人理解、保证代码可靠性。

## 设计模式中的设计原则 ##

**1.开闭原则（Open Close Principle）**

   开闭原则说的是对扩展开放，对修改关闭。按照开闭原则在程序代码需要进行扩展的时候不能对原有的程序进行修改而是要对程序实现热插拔的效果。这样一来程序的易复用和可扩展性就能得到大大提升。

**2.里氏代换原则（Liskov Substitution Principle）**

   里氏代换原则（LSP）中说，任何基类出现的地方，子类一定可以出现。LSP是继承复用的基石，只有当扩展类可以替换掉基类，软件的单位不受到影响时，基类才能真正的被复用，而扩展类也能够在基类的基础上增加新的行为。LSP是开闭原则的补充。实现开闭原则的关键步骤就是抽象化。而基类与扩展类的继承关系就是抽象化的具体实现。所以LSP是对实现抽象化的具体步骤的规范。

**3. 依赖倒转原则（Dependence Inversion Principle）**

   依赖倒转原则是开闭原则的基础，具体是说要针对接口编程而不是针对具体实现编程。为交互对象之间的松耦合设计而努力。

**4. 接口隔离原则（Interface Segregation Principle）**

   接口隔离原则是说要使用多个隔离的接口，而不要使用单个接口。其作用是降低对象之间的耦合度。

**5. 迪米特法则（Demeter Principle）**

   迪米特法则也叫最少知道原则。意思是对象之间应该尽量少的知道对方的信息。使得对象之间在一定程度上是相对独立的，作用也是降低对象之间的依赖程度。

**6. 合成复用原则（Composite Reuse Principle）**

意思就是多用组合少用继承。 更加灵活易于变化。

**设计模式原则小结：**

其实以上提到的各种原则目标大概只有一个：降低对象之间的耦合，增加程序的可复用性、可扩展性、可维护性。设计模式就是软件设计的一种思想，从大型软件架构出发，为了可复用和可升级而努力。


## 创建型设计模式： ##

**静态工厂模式**：由3个类组成，抽象的产品类；具体实现的产品类；具体实现的工厂类，提供不同的静态方法创建不同的产品。

**工厂方法模式**：由4个类组成，抽象的产品类；具体实现的产品类；抽象的工厂类；具体实现的工厂类，由具体的工厂实现类提供方法创建不同的产品。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/factory.png)

**抽象工厂模式**：有6个类组成，抽象的产品类A；具体实现的产品类A；抽象的产品类B；具体实现的产品类B；抽象的工厂类，包含抽象产品类A的创建方法，和抽象产品类B的创建方法；具体实现的工厂类，由具体的工厂实现类提供方法创建不同的产品。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/abstract factory.png)

**建造者模式**：由4个类组成，具体的产品类；抽象的建造者类；具体实现的建造者类；负责监督产品的组装的具体的监理类，监理类包含被抽象的建造者。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/builder.png)

**原型模式**：由2个类组成，继承Cloneable 接口的原型接口；具体实现的原型类。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/prototype.png)

**单例模式**：由1个类组成，私有的构造函数，并且包含一个私有的静态变量，提供一个静态方法获取实例。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/singleton.png)

## 结构型设计模式： ##

**适配器模式**：由3个类组成，抽象的目标对象类；具体实现目标对象的适配器类，适配器包含被适配对象（或者基础被适配对象）；被适配对象。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/adapter.png)


**桥接模式**：由4个类组成，抽象的实现器类；具体的实现器类；抽象的抽象化类，包含抽象的实现器；具体实现的抽象化类。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/bridge.png)

**组合模式**：由3个类组成，抽象的组合类；实现抽象组合类的叶子节点类；实现抽象组合类的非叶子节点类，且包含一个存放抽象组合类的集合。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/composite.png)

**装饰模式**：由4个类组成，抽象的被装饰类；具体实现的被装饰类；实现抽象的被装饰类的装饰器类，并包含抽象的被装饰类；具体实现的装饰器类。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/decorator.png)

**门面模式**：由2个类组成，抽象的门面类；具体实现门面类。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/facade.png)

**享元模式**：由3个类组成，抽象的轻量化类；具体实现的可以分享的类；轻量化工厂类，包含一个名称与抽象的轻量化类的映射关系集合。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/flyweight.png)

**代理模式**：由3个类组成，抽象的被代理对象类；具体实现的被代理对象类；实现被代理对象，并包含一个被代理对象类型的变量的代理类，代理对象先处理，处理不了的由被代理对象处理。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/proxy.png)

## 行为型设计模式： ##


**责任链模式**：由2个类组成，抽象的处理者类，并且包含当前抽象处理者类的变量；具体的处理者类，完成当前处理任务或者交由下一个处理者继续处理。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/chain.png)

**命令模式**：由4个类组成，抽象的命令类；具体实现的命令类，其中包含具体处理命令的接收器类；完成具体命令的接收器类；触发命令的触发器类，其中包含抽象的命令类。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/command.png)

**解释器模式**：由4个类组成，抽象的表达式类；实现抽象表达式类的终结符表达式类；实现抽象表达式类的非终结符表达式类，并包含抽象表达式类；上下文解释类，为解释器进行语法分析提供必要的信息。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/interpreter.png)

**迭代器模式**：由4个类组成，抽象的迭代器类，并包含一个是否存在下一个的抽象方法；具体实现的迭代器类；抽象的集合类，并包含一个指定迭代器的抽象方法，具体实现的集合类。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/iterator.png)

**中介者模式**：由4个类组成，抽象的中介者类，包含一个抽象的同事类集合；具体实现的中介者类；抽象的同事类，包含一个抽象的中介者类型变量；具体实现的同事类。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/mediator.png)

**备忘录模式**：由3个类组成，具体的被备忘的类，包含一个指定备忘器的方法，和一个从备忘器恢复数据的方法；具体的备忘器类，包含一个构造函数为被备忘类的入参；负责人类，包含一个备忘器类型变量。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/memento.png)

**观察者模式**：由4个类组成，抽象的观察对象类，包含一个抽象的观察者类的集合，并提供注册观察者、删除观察者和通知观察者抽象方法；具体实现的观察对象类；抽象的观察者类，定义接收观察对象发送通知处理的抽象方法；具体实现的观察者类，完成通知方法的实现。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/observer.png)

**状态模式**：由3个类组成，抽象的状态类；具体实现的状态类；上下文类，包含一个抽象状态类型的变量。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/state.png)

**策略模式**：由3个类组成，抽象的策略类；具体实现的策略类；上下文类，包含一个抽象策略类型的变量。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/strategy.png)

**访问者模式**：由5个类组成，抽象的元素类，提供接收访问者的抽象方法；具体实现的元素类；抽象的访问者类，提供访问元素的抽象方法；具体实现的访问者类；具体的元素对象结果，包含一个抽象元素类型的集合，和一个访问者类型的变量。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/visitor.png)

**模板模式**：由2个类组成，抽象的模板类，一个已实现的模板方法，和多个抽象的方法；具体的模板类，实现抽象方法。

![](https://github.com/longtian2/cc3/blob/master/images/java design pattern/template.png)

参考文献：

   《图解设计模式》 [日] 结城浩 著  杨文轩译

   https://www.cnblogs.com/wxisme/p/4693094.html


联系方式：

https://github.com/longtian2

**如有用请不吝打赏**

![](https://github.com/longtian2/cc3/blob/master/images/wechat_pay.png)