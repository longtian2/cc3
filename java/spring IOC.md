
## 名词解释 ##

**IOC（Inversion Of Control）**:控制反转

**DI（Dependency Injection）**:依赖注入

## 设计思想 ##

IOC是一种叫做“控制反转”的设计思想。

1、较浅的层次——从名字上解析 

“控制”就是指对象的创建、维护、销毁等生命周期的控制，这个过程一般是由我们的程序去主动控制的，如使用new关键字去创建一个对象（创建），在使用过程中保持引用（维护），在失去全部引用后由GC去回收对象（销毁）。 

“反转”就是指对 对象的创建、维护、销毁等生命周期的控制由程序控制改为由IOC容器控制，需要某个对象时就直接通过名字去IOC容器中获取。

2、更深的层次——提到DI，依赖注入，是IOC的一种重要实现 

一个对象的创建往往会涉及到其他对象的创建，比如一个对象A的成员变量持有着另一个对象B的引用，这就是依赖，A依赖于B。IOC机制既然负责了对象的创建，那么这个依赖关系也就必须由IOC容器负责起来。负责的方式就是DI——依赖注入，通过将依赖关系写入配置文件，然后在创建有依赖关系的对象时，由IOC容器注入依赖的对象，如在创建A时，检查到有依赖关系，IOC容器就把A依赖的对象B创建后注入到A中（组装，通过反射机制实现），然后把A返回给对象请求者，完成工作。

3、IOC的意义何在？ 

IOC并没有实现更多的功能，但它的存在使我们不需要很多代码、不需要考虑对象间复杂的耦合关系就能从IOC容器中获取合适的对象，而且提供了对对象的可靠的管理，极大地降低了开发的复杂性。

## Spring IOC 原理  ##

IoC 对于 Spring 框架来说，就是由 Spring 来负责控制对象的生命周期和对象间的关系。
换而言之，IOC主要存在两个职责：Bean 象的构建管理；和 Bean 对象间的依赖绑定。

首先，依赖对象是通过什么方式注入到被依赖对象的？

**接口注入**。从注入方式的使用上来说，接口注入是现在不甚提倡的一种方式，基本处于“退
役状态”。因为它强制被注入对象实现不必要的接口，带有侵入性。而构造方法注入和setter
方法注入则不需要如此。

**构造方法注入**。 这种注入方式的优点就是，对象在构造完成之后，即已进入就绪状态，可以
马上使用。缺点就是，当依赖对象比较多的时候，构造方法的参数列表会比较长。而通过反
射构造对象的时候，对相同类型的参数的处理会比较困难，维护和使用上也比较麻烦。而且
在Java中，构造方法无法被继承，无法设置默认值。对于非必须的依赖处理，可能需要引入多
个构造方法，而参数数量的变动可能造成维护上的不便。

**setter方法注入**。因为方法可以命名， 所以setter方法注入在描述性上要比构造方法注入好一些。
另外， setter方法可以被继承，允许设置默认值，而且有良好的IDE支持。缺点当然就是对象无
法在构造完成后马上进入就绪状态。

综上所述，构造方法注入和setter方法注入因为其侵入性较弱，且易于理解和使用，所以是现在使
用最多的注入方式；而接口注入因为侵入性较强，近年来已经不流行了。

其次，了解依赖注入的方式后，对象间依赖关系是如何绑定的？

1、**直接编码方式**

2、**配置文件方式**

3、**注解方式**


**Spring 提供两种容器类型：BeanFactory 和 ApplicationContext。**

BeanFactory 是基础类型的IOC容器，提供完整的IOC服务支持。如果没有特殊指定，**默认采用延迟初始化策略（lazy-load）**。只有当客户端对象需要访问容器中的某个受管对象的时候，才对该受管对象进行初始化以及依赖注入操作。所以，相对来说，容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求不是很严格的场景，BeanFactory是比较合适的IOC容器选择。

ApplicationContext 在BeanFactory 的基础上构建，是相对比较高级的容器实现，除了拥有BeanFactory 的所有支持，ApplicationContext 还提供了其他高级特性，比如事件发布、国际化信息支持等。ApplicationContext 所管理的对象，在该类型容器启动之后，**默认全部初始化并绑定完成**。所以，相对于BeanFactory来说，ApplicationContext 要去更多的系统资源，同时，因为在启动时就完成所有对象的初始化，容器启动时间较BeanFactory会长一下。在那些系统资源充足，并且要求更多功能的场景中，ApplicationContext 类型的容器是比较合适的选择。

![](https://github.com/longtian2/cc3/blob/master/images/spring-ioc.png)

BeanFactory只是一个接口，我们最终需要一个该接口的实现来进行实际的Bean的管理，DefaultListableBeanFactory就是这么一个比较通用的BeanFactory实现类。 DefaultListableBeanFactory除了间接地实现了BeanFactory接口，还实现了BeanDefinitionRegistry接口，该接口才是在BeanFactory的实现中担当Bean注册管理的角色。基本上，BeanFactory接口只定义如何访问容器内管理的Bean的方法，各个BeanFactory的具体实现类负责具体Bean的注册以及管理工作。BeanDefinitionRegistry接口定义抽象了Bean的注册逻辑。通常情况下，具体的BeanFactory实现类会实现这个接口来管理Bean的注册。

Spring的IoC容器支持两种配置文件格式： Properties文件格式和XML文件格式。当然，如果你愿意也可以引入自己的文件格式，前提是真的需要。采用外部配置文件时，Spring的IoC容器有一个统一的处理方式。通常情况下，需要根据不同的外部配置文件格式，给出相应的BeanDefinitionReader实现类，由BeanDefinitionReader的相应实现类负责将相应的配置文件内容读取并映射到BeanDefinition，然后将映射后的BeanDefinition注册到一个BeanDefinitionRegistry，之后，BeanDefinitionRegistry即完成Bean的注册和加载。当然，大部分工作，包括解析文件格式、装配BeanDefinition之类的工作，都是由BeanDefinitionReader的相应实现类来做的， BeanDefinitionRegistry只不过负责保管而已。PropertiesBeanDefinitionReader类用于Properties配置文件的加载，XmlBeanDefinitionReader类用于XML配置文件的加载。

每一个受管的对象，在容器中都会有一个BeanDefinition的实例（instance）与之相对应，该BeanDefinition的实例负责保存对象的所有必要信息，包括其对应的对象的class类型、是否是抽象类、构造方法参数以及其他属性等。当客户端向BeanFactory请求相应对象的时候， BeanFactory会通过这些信息为客户端返回一个完备可用的对象实例。 RootBeanDefinition 和 ChildBeanDefinition是BeanDefinition的两个主要实现类。

以XML配置实现举例

	public static void main(String[] args)
	{
		DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();
		BeanFactory container = (BeanFactory)bindViaXMLFile(beanRegistry);
		FXNewsProvider newsProvider = ➥
		(FXNewsProvider)container.getBean("djNewsProvider");
		newsProvider.getAndPersistNews(); 
	}

	public static BeanFactory bindViaXMLFile(BeanDefinitionRegistry registry)
	{ 
		XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(registry);
		reader.loadBeanDefinitions("classpath:../news-config.xml");
		return (BeanFactory)registry; 
		// 或者直接
		//return new XmlBeanFactory(new ClassPathResource("../news-config.xml"));
	}

Resource：定位xml文件

XmlBeanDefinitionReader：读取xml文件内容，转换成BeanDefinition

BeanDefinitionRegistry：并注册BeanDefinition

BeanFactory：完成依赖注入

参考文献：

   《Spring 揭秘》 王福强

   https://blog.csdn.net/zhangliangzi/article/details/51550912


联系方式：

https://github.com/longtian2

**如有用请不吝打赏**

![](https://github.com/longtian2/cc3/blob/master/images/wechat_pay.png)