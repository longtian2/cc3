# Spring AOP工作机制 #

## 前言 ##

我们先来了解一下，JVM在运行java程序时，JVM 是如何执行的，或者说 JVM 都在干什么？

当你运行 java 程序的时候，JVM会创建一个主线程，这个主线程以包含main()方法的类作为入口，开始执行程序代码。每一个线程在内存中都会维护一个属于自己的栈(Stack)，记录着整个程序执行的过程。栈里的每一个元素称为栈帧(Stack Frame)，栈帧表示着某个方法调用，会记录方法调用的信息；实际上我们在代码中调用一个方法的时候，在内存中就对应着一个栈帧的入栈和出栈。

至此，我们大致掌握了程序在JVM上运行的机制。那么，在某个特定的时间点，一个Main线程内的栈会呈现如下图所示的情况：

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-stack.png)

## AOP 设计思想 ##

**连接点（Joinpoint）**

在方法调用切面编程可切入的点，或者叫“时机”。

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-stack2.png)

**切面（Aspect）**

连接点横向切分成若干个面，即为编程的切面。切面编程是通过后面的建议来实现。

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-stack3.png)

**切入点(Pointcut)**

选择具体的切面执行特定的逻辑，该切面即为切入点。须检查类和方法是否匹配。

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-stack4.png)

**建议(Advice)**

针对切面的特定处理，即为建议。可分为方法调用前建议、方法调用后建议、方法调用前后建议、方法调用异常建议。

**建议者(Advisor)**

维护着建议与切入点的关系，即为建议者。

**代理工厂(ProxyFactory)**

通过动态代理实现建议者的织入，通过代理类先调用建议逻辑，再执行被切入的方法。

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-proxy.png)

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-proxy2.png)

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-proxy3.png)

通过上面关键知识点，我们大致逻辑了Spring AOP的工作机制，如下图：

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-work.png)

以上图片都引自[《Spring设计思想》AOP设计基本原理](https://blog.csdn.net/luanlouis/article/details/51095702)一文。

## Spring AOP 实现 ##

通过经典的实例来了解基于代理的AOP实现，其他实现方法请参考 [Spring AOP四种实现方式Demo详解与相关知识探究](https://blog.csdn.net/zhangliangzi/article/details/52334964)

（1）可睡觉的接口，任何可以睡觉的人或机器都可以实现它。

	public interface Sleepable {  
	    public void sleep();  
	}  

（2）接口实现类，“Me”可以睡觉，“Me”就实现可以睡觉的接口。

	public class Me implements Sleepable{  
	    public void sleep() {  
	        System.out.println("\n睡觉！不休息哪里有力气学习！\n");  
	    }  
	}  

（3）Me关注于睡觉的逻辑，但是睡觉需要其他功能辅助，比如睡前脱衣服，起床脱衣服，这里开始就需要AOP替“Me”完成！解耦！首先需要一个SleepHelper类。因为一个是切入点前执行、一个是切入点之后执行，所以实现对应接口。

	public class SleepHelper implements MethodBeforeAdvice, AfterReturningAdvice {  
	  
	    public void before(Method arg0, Object[] arg1, Object arg2) throws Throwable {  
	        System.out.println("睡觉前要脱衣服！");  
	    }  
	  
	    public void afterReturning(Object arg0, Method arg1, Object[] arg2, Object arg3) throws Throwable {  
	        System.out.println("起床后要穿衣服！");  
	    }  
	  
	}  

（4）最关键的来了，Spring核心配置文件application.xml配置AOP。


	<?xml version="1.0" encoding="UTF-8"?>  
	<beans xmlns="http://www.springframework.org/schema/beans"  
	<span style="white-space:pre">    </span>xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
	<span style="white-space:pre">    </span>xmlns:aop="http://www.springframework.org/schema/aop"  
	<span style="white-space:pre">    </span>xsi:schemaLocation="http://www.springframework.org/schema/beans  
	<span style="white-space:pre">    </span>http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
	<span style="white-space:pre">    </span>http://www.springframework.org/schema/aop  
	<span style="white-space:pre">    </span>http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">  
	     
	   <!-- 定义被代理者 -->  
	   <bean id="me" class="com.springAOP.bean.Me"></bean>  
	     
	   <!-- 定义通知内容，也就是切入点执行前后需要做的事情 -->  
	   <bean id="sleepHelper" class="com.springAOP.bean.SleepHelper"></bean>  
	     
	   <!-- 定义切入点位置 -->  
	   <bean id="sleepPointcut" class="org.springframework.aop.support.JdkRegexpMethodPointcut">  
	        <property name="pattern" value=".*sleep"></property>  
	   </bean>  
	     
	   <!-- 使切入点与通知相关联，完成切面配置 -->  
	   <bean id="sleepHelperAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">  
	        <property name="advice" ref="sleepHelper"></property>         
	        <property name="pointcut" ref="sleepPointcut"></property>  
	   </bean>  
	     
	   <!-- 设置代理 -->  
	   <bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">  
	        <!-- 代理的对象，有睡觉能力 -->  
	        <property name="target" ref="me"></property>  
	        <!-- 使用切面 -->  
	        <property name="interceptorNames" value="sleepHelperAdvisor"></property>  
	        <!-- 代理接口，睡觉接口 -->  
	        <property name="proxyInterfaces" value="com.springAOP.bean.Sleepable"></property>   
	   </bean>  
	      
	</beans>  

接下来，来看看Spring AOP实现核心类的UML图，该图出自[Spring AOP 实现原理](https://blog.csdn.net/moreevan/article/details/11977115/)：

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-uml.png)

Spring提供了两种方式来生成代理对象: JDKProxy和Cglib，具体使用哪种方式生成由AopProxyFactory根据AdvisedSupport对象的配置来决定。默认的策略是如果目标类是接口，则使用JDK动态代理技术，否则使用Cglib来生成代理。

当代理对象生成后，那切面建议者是如何织入的？

以JDK动态代理举例，我们知道InvocationHandler是JDK动态代理的核心，生成的代理对象的方法调用都会委托到InvocationHandler.invoke()方法。我们看到JdkDynamicAopProxy类实现了InvocationHandler，通过分析这个类中实现的invoke()方法即可了解Spring AOP是如何织入建议者的。

invoke（）方法的核心逻辑是通过AdvisedSupport.getInterceptorsAndDynamicInterceptionAdvice()方法得到拦截器链（interception chain），如果得到的拦截器链为空，则直接反射调用目标方法，否则创建ReflectiveMethodInvocation，调用其proceed方法，触发拦截器链的执行。

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-invoke.png)

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-proceed.png)

其中，拦截器就是建议者，拦截器链就是建议者链。AdvisedSupport.getInterceptorsAndDynamicInterceptionAdvice()方法的核心逻辑是通过AdvisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice()方法得到拦截器链（interception chain），AdvisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice()方法才是真正的将建议者转换成拦截器的核心方法。这里强调一下，拦截器（Interceptor）只是建议（Advice）的扩展，本质也是建议，换而言之，拦截器（Interceptor）继承于建议（Advice）。

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-advised.png)

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-aop-advice.png)



参考文献：

   《Spring 揭秘》 王福强

   https://blog.csdn.net/moreevan/article/details/11977115/

   https://blog.csdn.net/zhangliangzi/article/details/52334964

   https://blog.csdn.net/luanlouis/article/details/51095702

联系方式：

https://github.com/longtian2

**如有用请不吝打赏**

![](https://github.com/longtian2/cc3/blob/master/images/wechat_pay.png)
