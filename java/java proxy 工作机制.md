# java 动态代理 #

## 前言 ##

代理模式是设计模式的一种，提供了对目标对象的间接访问方式，即通过代理访问目标对象。

![](https://github.com/longtian2/cc3/blob/master/images/java-proxy-demo.png)

## 静态代理 ##

静态代理就是通过编程的方式来实现代理。代理类通过实现与目标对象相同的接口，并在类中维护一个目标对象的变量。

![](https://github.com/longtian2/cc3/blob/master/images/java%20design%20pattern/proxy.png)

静态代理因为需要开发人员手动编程来实现，如果需要被代理的对象越来越多，开发并维护这些代理类便变得困难了。于是乎，聪明的人类发明了动态代理，动态代理就是让程序自动生成代理类，不再让傲娇的开发人员手动编程了，顺应了开发人员懒惰的天性。动态代理有两种实现方式，一种是实现 JDK 提供的 InvocationHandler 接口的 invoke() 方法，通过Proxy.newProxyInstance() 方法来获得代理对象；另一种是实现 CGLIB 提供的 MethodInterceptor 接口的 intercept() 方法，通过Enhancer.create() 方法来获得代理对象。

## JDK动态代理 ##

**JDK动态代理是针对接口的代理**。通过 System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true") 来生成代理类的class文件。

	interface pro {
	    void text();
	}

	class proxyed implements pro {
	    @Override
	    public void text() {
	        System.err.println("本方法");
	    }
	}
	
	public class JavaProxy implements InvocationHandler {
	        private Object source;

	        public JavaProxy(Object source) {
	            super();
	            this.source = source;
	        }

	        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	            System.out.println("before");
	            Object invoke = method.invoke(source, args);
	            System.out.println("after");
	            return invoke;
	        }

	        public Object getProxy(){
	            return Proxy.newProxyInstance(getClass().getClassLoader(), source.getClass().getInterfaces(), this);
	        }

	        public static void main(String[] args) throws IllegalAccessException, InvocationTargetException, InstantiationException, NoSuchMethodException {
	            //第一种，自己写
	            //1.设置saveGeneratedFiles值为true则生成 class字节码文件方便分析
	            System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
	            //2.获取动态代理类
	            Class proxyClazz = Proxy.getProxyClass(pro.class.getClassLoader(),pro.class);
	            //3.获得代理类的构造函数，并传入参数类型InvocationHandler.class
	            Constructor constructor = proxyClazz.getConstructor(InvocationHandler.class);
	            //4.通过构造函数来创建动态代理对象，将自定义的InvocationHandler实例传入
	            pro iHello = (pro) constructor.newInstance(new JavaProxy(new proxyed()));
	            //5.通过代理对象调用目标方法
	            iHello.text();

	            //第二种，调用JDK提供的方法，实现了2~4步
	            iHello = (pro) Proxy.newProxyInstance(JavaProxy.class.getClassLoader(),proxyed.class.getInterfaces(),new JavaProxy(new proxyed()));
				iHello.text();
	        }
	}

我们来梳理一下其工作机制，我们以程序中的第二种方式作为切入口。首先，**我们来看Proxy.newProxyInstance()方法的实现，该方法的核心是逻辑是生成代理类、获取代理类的构造器、通过构造器创建对象，获取代理类的构造器、通过构造器创建对象就是JDK提供的反射机制**。

![](https://github.com/longtian2/cc3/blob/master/images/java-proxy-instance.png)

重点看一下代理类是怎么生成的，继续查看Proxy.getProxyClass0() 方法，发现生成的代理类会放入内存（即放入WeakCache中）缓存起来，避免重复生成代理类的问题。

![](https://github.com/longtian2/cc3/blob/master/images/java-proxy-class.png)

继续深挖代码，WeakCache.get() 方法最终都会调用Proxy.ProxyClassFactory.apply() 方法，Proxy.ProxyClassFactory.apply()方法通过ProxyGenerator.generateProxyClass() 方法返回的字节流完成代理类的加载。

![](https://github.com/longtian2/cc3/blob/master/images/java-proxy-class2.png)

![](https://github.com/longtian2/cc3/blob/master/images/java-proxy-class3.png)

由ProxyGenerator.generateProxyClass() 方法负责生成class文件

![](https://github.com/longtian2/cc3/blob/master/images/java-proxy-class4.png)

最后，来看一下生成的代理类，发现代理类继续Proxy并实现被代理接口(Pro)，所有的方法调用InvocationHandler.invoke()方法，这里就会回调之前实现InvocationHandler接口的invoke方法，譬如JavaProxy。**再调用被代理类时是通过Method.invoke()方法调用的，是采用的JDK反射机制**。

	final class $Proxy0 extends Proxy implements Pro {
        //fields    
        private static Method m1;
        private static Method m2;
        private static Method m3;
        private static Method m0;

        public $Proxy0(InvocationHandler var1) throws  {
            super(var1);
        }

        public final boolean equals(Object var1) throws  {
            try {
                return ((Boolean)super.h.invoke(this, m1, new Object[]{var1})).booleanValue();
            } catch (RuntimeException | Error var3) {
                throw var3;
            } catch (Throwable var4) {
                throw new UndeclaredThrowableException(var4);
            }
        }

        public final String toString() throws  {
            try {
                return (String)super.h.invoke(this, m2, (Object[])null);
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }

        public final void text() throws  {
            try {
                //实际就是调用代理类的invoke方法 
                super.h.invoke(this, m3, (Object[])null);
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }

        public final int hashCode() throws  {
            try {
                return ((Integer)super.h.invoke(this, m0, (Object[])null)).intValue();
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }

        static {
            try {
                //这里每个方法对象 和类的实际方法绑定
                m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
                m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
                m3 = Class.forName("spring.commons.api.study.CreateModel.pro").getMethod("text", new Class[0]);
                m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
            } catch (NoSuchMethodException var2) {
                throw new NoSuchMethodError(var2.getMessage());
            } catch (ClassNotFoundException var3) {
                throw new NoClassDefFoundError(var3.getMessage());
            }
        }
    }


## CGLIB 动态代理 ##

**CGLIB动态代理不仅可以实现接口的代理，还可以实现类的代理。重点强调一下，被final修饰的方法是不能被代理的**。通过 System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "D:\\code") 来生成代理类的class文件。


	public class PersonService {
	  
	    //该方法不能被子类覆盖
	    public final void getPerson(String code) {
	        System.out.println("PersonService:getPerson>>" + code);
	    }
	
	    public void setPerson() {
	        System.out.println("PersonService:setPerson");
	    }
	}

	public class CglibProxyIntercepter implements MethodInterceptor {
	    @Override
	    public Object intercept(Object sub, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
	        System.out.println("执行前...");
	        Object object = methodProxy.invokeSuper(sub, objects);
	        System.out.println("执行后...");
	        return object;
	    }
	}

	public class Test {
	    public static void main(String[] args) {
	        //代理类class文件存入本地磁盘
	        System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "D:\\code");
	        Enhancer enhancer = new Enhancer();
	        enhancer.setSuperclass(PersonService.class);
	        enhancer.setCallback(new CglibProxyIntercepter());
	        PersonService proxy= (PersonService)  enhancer.create();
	        proxy.setPerson();
	        proxy.getPerson("1"); 
	    } 
	}

再来梳理一下CGLIB 的工作机制，首先，Enhancer.create() 方法就是切入口，代理对象就是由该方法返回的。 Enhancer.create()调用了Enhancer.createHelper()方法，Enhancer.createHelper()方法主要确定类名的前缀。

![](https://github.com/longtian2/cc3/blob/master/images/java-cglib-create.png)

![](https://github.com/longtian2/cc3/blob/master/images/java-cglib-create2.png)

AbstractClassGenerator.create()方法先从缓存中查找，未查找到被代理类的对象则先生成代理类，然后将生成的代理类放入缓存中，并通过JDK反射机制完成对象的创建。

![](https://github.com/longtian2/cc3/blob/master/images/java-cglib-create3.png)

![](https://github.com/longtian2/cc3/blob/master/images/java-cglib-reflect.png)

DefaultGeneratorStrategy.generate()生成class文件

![](https://github.com/longtian2/cc3/blob/master/images/java-cglib-class.png)

DebuggingClassWriter.toByteArray() 生成 class文件的字节流

![](https://github.com/longtian2/cc3/blob/master/images/java-cglib-class2.png)

最后，来看一下CGLIB生成的代理类，发现生成了3个class文件，其中一个主要class文件就是代理类的class文件，该类继承被代理类（PersonService）并实现net.sf.cglib.proxy.Factroy接口，所有非final修饰的方法都会调用MethodInterceptor.intercept()方法，这里就会回调到之前实现MethodInterceptor接口的intercept方法，譬如CglibProxyIntercepter。另外两个class文件是FastClass机制所需的文件，都继承FastClass类。FastClass并不是跟代理类一块生成的，而是在第一次执行MethodProxy invoke/invokeSuper时生成的并放在了缓存中。这里两个FastClass类不是都需要用，实际上使用哪个FastClass类取决于MethodProxy.invoke()方法或MethodProxy.invokeSuper()方法。

![](https://github.com/longtian2/cc3/blob/master/images/java-cglib-class3.png)

PersonService$$EnhancerByCGLIB$$824b571f就是代理类

	public class PersonService$$EnhancerByCGLIB$$824b571f extends PersonService implements Factory {
	    private boolean CGLIB$BOUND;
	    private static final ThreadLocal CGLIB$THREAD_CALLBACKS;
	    private static final Callback[] CGLIB$STATIC_CALLBACKS;
	    private MethodInterceptor CGLIB$CALLBACK_0;
	    private static final Method CGLIB$setPerson$0$Method;
	    private static final MethodProxy CGLIB$setPerson$0$Proxy;
	    private static final Object[] CGLIB$emptyArgs;
	    private static final Method CGLIB$finalize$1$Method;
	    private static final MethodProxy CGLIB$finalize$1$Proxy;
	    private static final Method CGLIB$equals$2$Method;
	    private static final MethodProxy CGLIB$equals$2$Proxy;
	    private static final Method CGLIB$toString$3$Method;
	    private static final MethodProxy CGLIB$toString$3$Proxy;
	    private static final Method CGLIB$hashCode$4$Method;
	    private static final MethodProxy CGLIB$hashCode$4$Proxy;
	    private static final Method CGLIB$clone$5$Method;
	    private static final MethodProxy CGLIB$clone$5$Proxy;
	
	    static void CGLIB$STATICHOOK1() {
	        CGLIB$THREAD_CALLBACKS = new ThreadLocal();
	        CGLIB$emptyArgs = new Object[0];
	        Class var0 = Class.forName("com.wlqq.complaint.controller.web.PersonService$$EnhancerByCGLIB$$824b571f");
	        Class var1;
	        Method[] var10000 = ReflectUtils.findMethods(new String[]{"finalize", "()V", "equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;"}, (var1 = Class.forName("java.lang.Object")).getDeclaredMethods());
	        CGLIB$finalize$1$Method = var10000[0];
	        CGLIB$finalize$1$Proxy = MethodProxy.create(var1, var0, "()V", "finalize", "CGLIB$finalize$1");
	        CGLIB$equals$2$Method = var10000[1];
	        CGLIB$equals$2$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/Object;)Z", "equals", "CGLIB$equals$2");
	        CGLIB$toString$3$Method = var10000[2];
	        CGLIB$toString$3$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/String;", "toString", "CGLIB$toString$3");
	        CGLIB$hashCode$4$Method = var10000[3];
	        CGLIB$hashCode$4$Proxy = MethodProxy.create(var1, var0, "()I", "hashCode", "CGLIB$hashCode$4");
	        CGLIB$clone$5$Method = var10000[4];
	        CGLIB$clone$5$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/Object;", "clone", "CGLIB$clone$5");
			
			//重点看这里1
	        CGLIB$setPerson$0$Method = ReflectUtils.findMethods(new String[]{"setPerson", "()V"}, (var1 = Class.forName("com.wlqq.complaint.controller.web.PersonService")).getDeclaredMethods())[0];
	        CGLIB$setPerson$0$Proxy = MethodProxy.create(var1, var0, "()V", "setPerson", "CGLIB$setPerson$0");
	    }
	
		//重点看这里2
	    final void CGLIB$setPerson$0() {
	        super.setPerson();
	    }

		//重点看这里3
	    public final void setPerson() {
	        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
	        if(this.CGLIB$CALLBACK_0 == null) {
	            CGLIB$BIND_CALLBACKS(this);
	            var10000 = this.CGLIB$CALLBACK_0;
	        }
	
	        if(var10000 != null) {
	            var10000.intercept(this, CGLIB$setPerson$0$Method, CGLIB$emptyArgs, CGLIB$setPerson$0$Proxy);
	        } else {
	            super.setPerson();
	        }
	    }
	
	    final void CGLIB$finalize$1() throws Throwable {
	        super.finalize();
	    }
	
	    protected final void finalize() throws Throwable {
	        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
	        if(this.CGLIB$CALLBACK_0 == null) {
	            CGLIB$BIND_CALLBACKS(this);
	            var10000 = this.CGLIB$CALLBACK_0;
	        }
	
	        if(var10000 != null) {
	            var10000.intercept(this, CGLIB$finalize$1$Method, CGLIB$emptyArgs, CGLIB$finalize$1$Proxy);
	        } else {
	            super.finalize();
	        }
	    }
	
	    final boolean CGLIB$equals$2(Object var1) {
	        return super.equals(var1);
	    }
	
	    public final boolean equals(Object var1) {
	        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
	        if(this.CGLIB$CALLBACK_0 == null) {
	            CGLIB$BIND_CALLBACKS(this);
	            var10000 = this.CGLIB$CALLBACK_0;
	        }
	
	        if(var10000 != null) {
	            Object var2 = var10000.intercept(this, CGLIB$equals$2$Method, new Object[]{var1}, CGLIB$equals$2$Proxy);
	            return var2 == null?false:((Boolean)var2).booleanValue();
	        } else {
	            return super.equals(var1);
	        }
	    }
	
	    final String CGLIB$toString$3() {
	        return super.toString();
	    }
	
	    public final String toString() {
	        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
	        if(this.CGLIB$CALLBACK_0 == null) {
	            CGLIB$BIND_CALLBACKS(this);
	            var10000 = this.CGLIB$CALLBACK_0;
	        }
	
	        return var10000 != null?(String)var10000.intercept(this, CGLIB$toString$3$Method, CGLIB$emptyArgs, CGLIB$toString$3$Proxy):super.toString();
	    }
	
	    final int CGLIB$hashCode$4() {
	        return super.hashCode();
	    }
	
	    public final int hashCode() {
	        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
	        if(this.CGLIB$CALLBACK_0 == null) {
	            CGLIB$BIND_CALLBACKS(this);
	            var10000 = this.CGLIB$CALLBACK_0;
	        }
	
	        if(var10000 != null) {
	            Object var1 = var10000.intercept(this, CGLIB$hashCode$4$Method, CGLIB$emptyArgs, CGLIB$hashCode$4$Proxy);
	            return var1 == null?0:((Number)var1).intValue();
	        } else {
	            return super.hashCode();
	        }
	    }
	
	    final Object CGLIB$clone$5() throws CloneNotSupportedException {
	        return super.clone();
	    }
	
	    protected final Object clone() throws CloneNotSupportedException {
	        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
	        if(this.CGLIB$CALLBACK_0 == null) {
	            CGLIB$BIND_CALLBACKS(this);
	            var10000 = this.CGLIB$CALLBACK_0;
	        }
	
	        return var10000 != null?var10000.intercept(this, CGLIB$clone$5$Method, CGLIB$emptyArgs, CGLIB$clone$5$Proxy):super.clone();
	    }
	
	    public static MethodProxy CGLIB$findMethodProxy(Signature var0) {
	        String var10000 = var0.toString();
	        switch(var10000.hashCode()) {
	        case -1902447170:
	            if(var10000.equals("setPerson()V")) {
	                return CGLIB$setPerson$0$Proxy;
	            }
	            break;
	        case -1574182249:
	            if(var10000.equals("finalize()V")) {
	                return CGLIB$finalize$1$Proxy;
	            }
	            break;
	        case -508378822:
	            if(var10000.equals("clone()Ljava/lang/Object;")) {
	                return CGLIB$clone$5$Proxy;
	            }
	            break;
	        case 1826985398:
	            if(var10000.equals("equals(Ljava/lang/Object;)Z")) {
	                return CGLIB$equals$2$Proxy;
	            }
	            break;
	        case 1913648695:
	            if(var10000.equals("toString()Ljava/lang/String;")) {
	                return CGLIB$toString$3$Proxy;
	            }
	            break;
	        case 1984935277:
	            if(var10000.equals("hashCode()I")) {
	                return CGLIB$hashCode$4$Proxy;
	            }
	        }
	
	        return null;
	    }
	
	    public PersonService$$EnhancerByCGLIB$$824b571f() {
	        CGLIB$BIND_CALLBACKS(this);
	    }
	
	    public static void CGLIB$SET_THREAD_CALLBACKS(Callback[] var0) {
	        CGLIB$THREAD_CALLBACKS.set(var0);
	    }
	
	    public static void CGLIB$SET_STATIC_CALLBACKS(Callback[] var0) {
	        CGLIB$STATIC_CALLBACKS = var0;
	    }
	
	    private static final void CGLIB$BIND_CALLBACKS(Object var0) {
	        PersonService$$EnhancerByCGLIB$$824b571f var1 = (PersonService$$EnhancerByCGLIB$$824b571f)var0;
	        if(!var1.CGLIB$BOUND) {
	            var1.CGLIB$BOUND = true;
	            Object var10000 = CGLIB$THREAD_CALLBACKS.get();
	            if(var10000 == null) {
	                var10000 = CGLIB$STATIC_CALLBACKS;
	                if(CGLIB$STATIC_CALLBACKS == null) {
	                    return;
	                }
	            }
	
	            var1.CGLIB$CALLBACK_0 = (MethodInterceptor)((Callback[])var10000)[0];
	        }
	
	    }
	
	    public Object newInstance(Callback[] var1) {
	        CGLIB$SET_THREAD_CALLBACKS(var1);
	        PersonService$$EnhancerByCGLIB$$824b571f var10000 = new PersonService$$EnhancerByCGLIB$$824b571f();
	        CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
	        return var10000;
	    }
	
	    public Object newInstance(Callback var1) {
	        CGLIB$SET_THREAD_CALLBACKS(new Callback[]{var1});
	        PersonService$$EnhancerByCGLIB$$824b571f var10000 = new PersonService$$EnhancerByCGLIB$$824b571f();
	        CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
	        return var10000;
	    }
	
	    public Object newInstance(Class[] var1, Object[] var2, Callback[] var3) {
	        CGLIB$SET_THREAD_CALLBACKS(var3);
	        PersonService$$EnhancerByCGLIB$$824b571f var10000 = new PersonService$$EnhancerByCGLIB$$824b571f;
	        switch(var1.length) {
	        case 0:
	            var10000.<init>();
	            CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
	            return var10000;
	        default:
	            throw new IllegalArgumentException("Constructor not found");
	        }
	    }
	
	    public Callback getCallback(int var1) {
	        CGLIB$BIND_CALLBACKS(this);
	        MethodInterceptor var10000;
	        switch(var1) {
	        case 0:
	            var10000 = this.CGLIB$CALLBACK_0;
	            break;
	        default:
	            var10000 = null;
	        }
	
	        return var10000;
	    }
	
	    public void setCallback(int var1, Callback var2) {
	        switch(var1) {
	        case 0:
	            this.CGLIB$CALLBACK_0 = (MethodInterceptor)var2;
	        default:
	        }
	    }
	
	    public Callback[] getCallbacks() {
	        CGLIB$BIND_CALLBACKS(this);
	        return new Callback[]{this.CGLIB$CALLBACK_0};
	    }
	
	    public void setCallbacks(Callback[] var1) {
	        this.CGLIB$CALLBACK_0 = (MethodInterceptor)var1[0];
	    }
	
	    static {
	        CGLIB$STATICHOOK1();
	    }
	}

通过生成的class文件我们也清晰的看到并没有生成final修饰的getPerson()方法信息，只生成了setPerson()方法信息，注意了这里生成了两个关于setPerson()方法的信息。CGLIB$setPerson$0()是直接调用的父类，即直接调用的被代理对象的方法；setPerson()是被final修饰的，判断了当前有没有MethodInterceptor的回调变量，存在则会调用MethodInterceptor.intercept()方法，不存在则直接调用被代理对象的方法。

回到我们实现MethodInterceptor接口的CglibProxyIntercepter类，MethodProxy.invokeSuper()方法决定使用PersonService$$EnhancerByCGLIB$$824b571f$$FastClassByCGLIB$$d07d4322类的invoke()方法。

![](https://github.com/longtian2/cc3/blob/master/images/java-cglib-invokeSuper.png)

![](https://github.com/longtian2/cc3/blob/master/images/java-cglib-invoke.png)

PersonService$$EnhancerByCGLIB$$824b571f$$FastClassByCGLIB$$d07d4322类的invoke()方法，**该方法把代理类的方法全部对应上数组索引，即通过方法的比对就能快速的实现方法的调用，这与JDK的反射形成鲜明的对比**。

	public Object invoke(int var1, Object var2, Object[] var3) throws InvocationTargetException {
        824b571f var10000 = (824b571f)var2;
        int var10001 = var1;

        try {
            switch(var10001) {
            case 0:
                return new Boolean(var10000.equals(var3[0]));
            case 1:
                return var10000.toString();
            case 2:
                return new Integer(var10000.hashCode());
            case 3:
                return var10000.newInstance((Class[])var3[0], (Object[])var3[1], (Callback[])var3[2]);
            case 4:
                return var10000.newInstance((Callback)var3[0]);
            case 5:
                return var10000.newInstance((Callback[])var3[0]);
            case 6:
                var10000.setCallback(((Number)var3[0]).intValue(), (Callback)var3[1]);
                return null;
            case 7:
                var10000.setCallbacks((Callback[])var3[0]);
                return null;
            case 8:
                var10000.setPerson();
                return null;
            case 9:
                return var10000.getCallback(((Number)var3[0]).intValue());
            case 10:
                return var10000.getCallbacks();
            case 11:
                824b571f.CGLIB$SET_STATIC_CALLBACKS((Callback[])var3[0]);
                return null;
            case 12:
                824b571f.CGLIB$SET_THREAD_CALLBACKS((Callback[])var3[0]);
                return null;
            case 13:
                return 824b571f.CGLIB$findMethodProxy((Signature)var3[0]);
            case 14:
                824b571f.CGLIB$STATICHOOK1();
                return null;
            case 15:
                var10000.CGLIB$setPerson$0();
                return null;
            case 16:
                return new Integer(var10000.CGLIB$hashCode$4());
            case 17:
                var10000.CGLIB$finalize$1();
                return null;
            case 18:
                return var10000.CGLIB$toString$3();
            case 19:
                return var10000.CGLIB$clone$5();
            case 20:
                return new Boolean(var10000.CGLIB$equals$2(var3[0]));
            case 21:
                var10000.getPerson();
                return null;
            case 22:
                var10000.wait();
                return null;
            case 23:
                var10000.wait(((Number)var3[0]).longValue(), ((Number)var3[1]).intValue());
                return null;
            case 24:
                var10000.wait(((Number)var3[0]).longValue());
                return null;
            case 25:
                return var10000.getClass();
            case 26:
                var10000.notify();
                return null;
            case 27:
                var10000.notifyAll();
                return null;
            }
        } catch (Throwable var4) {
            throw new InvocationTargetException(var4);
        }

        throw new IllegalArgumentException("Cannot find matching method/constructor");
    }

再来看看PersonService$$FastClassByCGLIB$$4b1bc63类的invoke()方法。该方法采用相同的机制实现。

	public Object invoke(int var1, Object var2, Object[] var3) throws InvocationTargetException {
        PersonService var10000 = (PersonService)var2;
        int var10001 = var1;

        try {
            switch(var10001) {
            case 0:
                var10000.setPerson();
                return null;
            case 1:
                var10000.getPerson();
                return null;
            case 2:
                var10000.wait();
                return null;
            case 3:
                var10000.wait(((Number)var3[0]).longValue(), ((Number)var3[1]).intValue());
                return null;
            case 4:
                var10000.wait(((Number)var3[0]).longValue());
                return null;
            case 5:
                return new Boolean(var10000.equals(var3[0]));
            case 6:
                return var10000.toString();
            case 7:
                return new Integer(var10000.hashCode());
            case 8:
                return var10000.getClass();
            case 9:
                var10000.notify();
                return null;
            case 10:
                var10000.notifyAll();
                return null;
            }
        } catch (Throwable var4) {
            throw new InvocationTargetException(var4);
        }

        throw new IllegalArgumentException("Cannot find matching method/constructor");
    }

怎样才能使用到该方法勒？只有我们实现MethodInterceptor接口的CglibProxyIntercepter类，采用MethodProxy.invoke()方法调用被代理对象才会进入到PersonService$$FastClassByCGLIB$$4b1bc63类的invoke()方法。

**这里强调一下MethodInterceptor.intercept(Object sub, Method method, Object[] objects, MethodProxy methodProxy)方法中的sub是代理类的对象，如果直接通过MethodProxy.invoke(sub,objects)是会循环拦截导致栈溢出的**。原因很简单，看看调用链就明白了：MethodProxy.invoke() --> PersonService$$FastClassByCGLIB$$4b1bc63.invoke()--> var10000.setPerson() --> PersonService$$EnhancerByCGLIB$$824b571f.setPerson() --> MethodInterceptor.intercept() --> MethodProxy.invoke()


最后我们总结一下JDK动态代理和Gglib动态代理的区别：

**1.JDK动态代理是实现了被代理对象的接口，Cglib是继承了被代理对象。**

**2.JDK和Cglib都是在运行期生成字节码，JDK是直接写Class字节码，Cglib使用ASM框架写Class字节码，Cglib代理实现更复杂，生成代理类比JDK效率低。**

**3.JDK调用代理方法，是通过反射机制调用，Cglib是通过FastClass机制直接调用方法，Cglib执行效率更高。**


参考文献：

   https://www.cnblogs.com/zhangxinly/p/6974283.html

   https://www.cnblogs.com/monkey0307/p/8328821.html

联系方式：

https://github.com/longtian2

**如有用请不吝打赏**

![](https://github.com/longtian2/cc3/blob/master/images/wechat_pay.png)