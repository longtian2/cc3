# Spring Bean初始化过程 #

## 基础知识 ##

Spring对Bean的生命周期提供了多种扩展入口，可以划分为以下4类型： 

1、Bean自身的方法：

　　通过配置文件中<bean>的init-method和destroy-method指定Bean类本身的方法，可以扩展自定义的Bean对象的初始化和销毁逻辑。

2、Bean级生命周期接口方法：

　 通过Bean类实现 BeanNameAware、BeanFactoryAware接口的方法是可以获取到Bean类的BeanName，和创建该Bean的BeanFactory；

   通过Bean类实现 InitializingBean、DiposableBean接口的方法，可以扩展自定义的Bean对象的初始化和销毁逻辑。

3、容器级生命周期接口方法：

　　通过Bean类实现 InstantiationAwareBeanPostProcessor 和 BeanPostProcessor 接口的方法，可以在Bean实例化前后、初始化前后扩展自定义逻辑。

 
4、后处理器接口方法：

　　通过Bean类实现 BeanFactoryPostProcessor接口的方法，可以在Bean在加载完成之后，还没有进入实例化之前，修改Bean的信息。BeanFactoryPostProcessor接口实现包括 AspectJWeavingEnabler, ConfigurationClassPostProcessor, CustomAutowireConfigurer等等，都是非常有用的工厂后处理器。
 
接下来通过示例来了解Bean的生命周期过程

	public class Person implements BeanFactoryAware, BeanNameAware, InitializingBean, DisposableBean {
	
	    private String name;
	    private String address;
	    private String phone;
	
	    private BeanFactory beanFactory;
	    private String beanName;
	
	    public Person() {
	        System.out.println("【构造器】调用Person的构造器实例化");
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        System.out.println("【注入属性】注入属性name");
	        this.name = name;
	    }
	
	    public String getAddress() {
	        return address;
	    }
	
	    public void setAddress(String address) {
	        System.out.println("【注入属性】注入属性address");
	        this.address = address;
	    }
	
	    public String getPhone() {
	        return phone;
	    }
	
	    public void setPhone(String phone) {
	        System.out.println("【注入属性】注入属性phone");
	        this.phone = phone;
	    }
	
	    @Override
	    public String toString() {
	        return "com.Person [address=" + address + ", name=" + name + ", phone=" + phone + "]";
	    }
	
	    // 这是BeanFactoryAware接口方法
	    @Override
	    public void setBeanFactory(BeanFactory arg) throws BeansException {
	        System.out.println("【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()");
	        this.beanFactory = arg;
	    }
	
	    // 这是BeanNameAware接口方法
	    @Override
	    public void setBeanName(String arg) {
	        System.out.println("【BeanNameAware接口】调用BeanNameAware.setBeanName()");
	        this.beanName = arg;
	    }
	
	    // 这是InitializingBean接口方法
	    @Override
	    public void afterPropertiesSet() throws Exception {
	        System.out.println("【InitializingBean接口】调用InitializingBean.afterPropertiesSet()");
	    }
	
	    // 这是DiposibleBean接口方法
	    @Override
	    public void destroy() throws Exception {
	        System.out.println("【DiposibleBean接口】调用DiposibleBean.destory()");
	    }
	
	    // 通过<bean>的init-method属性指定的初始化方法
	    public void myInit() {
	        System.out.println("【init-method】调用<bean>的init-method属性指定的初始化方法");
	    }
	
	    // 通过<bean>的destroy-method属性指定的初始化方法
	    public void myDestory() {
	        System.out.println("【destroy-method】调用<bean>的destroy-method属性指定的初始化方法");
	    }
	}


	public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
	
	    public MyBeanFactoryPostProcessor() {
	        super();
	        System.out.println("这是BeanFactoryPostProcessor实现类构造器！！");
	    }
	
	    @Override
	    public void postProcessBeanFactory(ConfigurableListableBeanFactory arg) throws BeansException {
	        System.out.println("BeanFactoryPostProcessor调用postProcessBeanFactory方法");
	        BeanDefinition bd = arg.getBeanDefinition("person");
	        bd.getPropertyValues().addPropertyValue("phone", "");
	    }
	}


	public class MyInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessorAdapter {
	
	    public MyInstantiationAwareBeanPostProcessor() {
	        super();
	        System.out.println("这是InstantiationAwareBeanPostProcessorAdapter实现类构造器！！");
	    }
	
	    // 实例化Bean之前调用，指调用构造函数之前
	    @Override
	    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) {
	        System.out.println(beanName + " : InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法1");
	        return null;
	    }
	
	    // 初始化Bean之前调用，指对象实例化，并依赖注入完成之后，初始化方法之前。
	    @Override
	    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
	
	        System.out.println(beanName + " : InstantiationAwareBeanPostProcessor调用postProcessBeforeInitialization方法2");
	        return bean;
	    }
	
	    // 实例化Bean之后调用，指调用构造函数之后
	    @Override
	    public boolean postProcessAfterInstantiation(Object bean, String beanName) {
	        System.out.println(beanName + " : InstantiationAwareBeanPostProcessor调用postProcessAfterInstantiation方法3");
	        return true;
	    }
	
	    // 初始化Bean之后调用，指对象实例化，并依赖注入完成之后，初始化方法之后。
	    @Override
	    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
	        System.out.println(beanName + " : InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法4");
	        return bean;
	    }
	
	    // 设置属性时调用
	    @Override
	    public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean,
	            String beanName) throws BeansException {
	        System.out.println(beanName + " : InstantiationAwareBeanPostProcessor调用postProcessPropertyValues方法");
	        return pvs;
	    }
	}

	public class MyBeanPostProcessor implements BeanPostProcessor {
	
	    public MyBeanPostProcessor() {
	        super();
	        System.out.println("这是BeanPostProcessor实现类构造器！！");
	    }
	
	    @Override
	    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
	        System.out.println(beanName + " : BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改！");
	        return bean;
	    }
	
	    @Override
	    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
	        System.out.println(beanName + " : BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改！");
	        return bean;
	    }
	}

重点强调一下，以下4个方法长的很像，稍不注意就整错了，然后就开始怀疑人生了。

**postProcessBeforeInstantiation()和postProcessAfterInstantiation()方法是在Bean实例化前后调用，指的是Bean构造函数被调用前后。**

**postProcessBeforeInitialization()和postProcessAfterInitialization()方法是在Bean初始化前后调用，指的是Bean实例化并完成依赖注入，在初始化方法被调用前后。**

其中，InstantiationAwareBeanPostProcessor 接口的父接口是BeanPostProcessor，postProcessBeforeInitialization()和postProcessAfterInitialization()方法就是来自于父接口BeanPostProcessor。因此，实现InstantiationAwareBeanPostProcessor接口的postProcessBeforeInitialization()和postProcessAfterInitialization()方法，跟实现BeanPostProcessor是等价的。

输入日志情况：

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-bean-init-log-1.png)

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-bean-init-log-2.png)

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-bean-init-log-3.png)

根据日志我们能得出如下结论：

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-bean-init.png)

## 源码分析 ##

AbstractBeanFactory.getBean()方法，获取Bean的入口，即IOC容器的第二阶段，Bean的初始化就是从这里开始。

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-code-beanFactory-getBean.png)

AbstractBeanFactory.doGetBean()方法，该方法很重要，主要职责是：如果内存中存在单例的Bean，则直接返回；如果内存中不存在，则根据对Bean的Scope的不同创建对象。

	protected <T> T doGetBean(
			final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
			throws BeansException {

		final String beanName = transformedBeanName(name);
		Object bean;

		// Eagerly check singleton cache for manually registered singletons.
		Object sharedInstance = getSingleton(beanName);
		if (sharedInstance != null && args == null) {
			if (logger.isDebugEnabled()) {
				if (isSingletonCurrentlyInCreation(beanName)) {
					logger.debug("Returning eagerly cached instance of singleton bean '" + beanName +
							"' that is not fully initialized yet - a consequence of a circular reference");
				}
				else {
					logger.debug("Returning cached instance of singleton bean '" + beanName + "'");
				}
			}
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
		}

		else {
			// Fail if we're already creating this bean instance:
			// We're assumably within a circular reference.
			if (isPrototypeCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(beanName);
			}

			// Check if bean definition exists in this factory.
			BeanFactory parentBeanFactory = getParentBeanFactory();
			if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
				// Not found -> check parent.
				String nameToLookup = originalBeanName(name);
				if (args != null) {
					// Delegation to parent with explicit args.
					return (T) parentBeanFactory.getBean(nameToLookup, args);
				}
				else {
					// No args -> delegate to standard getBean method.
					return parentBeanFactory.getBean(nameToLookup, requiredType);
				}
			}

			if (!typeCheckOnly) {
				markBeanAsCreated(beanName);
			}

			try {
				final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
				checkMergedBeanDefinition(mbd, beanName, args);

				// Guarantee initialization of beans that the current bean depends on.
				String[] dependsOn = mbd.getDependsOn();
				if (dependsOn != null) {
					for (String dep : dependsOn) {
						if (isDependent(beanName, dep)) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
						}
						registerDependentBean(dep, beanName);
						getBean(dep);
					}
				}

				// Create bean instance.
				if (mbd.isSingleton()) {
					sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
						@Override
						public Object getObject() throws BeansException {
							try {
								return createBean(beanName, mbd, args);
							}
							catch (BeansException ex) {
								// Explicitly remove instance from singleton cache: It might have been put there
								// eagerly by the creation process, to allow for circular reference resolution.
								// Also remove any beans that received a temporary reference to the bean.
								destroySingleton(beanName);
								throw ex;
							}
						}
					});
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}

				else if (mbd.isPrototype()) {
					// It's a prototype -> create a new instance.
					Object prototypeInstance = null;
					try {
						beforePrototypeCreation(beanName);
						prototypeInstance = createBean(beanName, mbd, args);
					}
					finally {
						afterPrototypeCreation(beanName);
					}
					bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
				}

				else {
					String scopeName = mbd.getScope();
					final Scope scope = this.scopes.get(scopeName);
					if (scope == null) {
						throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
					}
					try {
						Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
							@Override
							public Object getObject() throws BeansException {
								beforePrototypeCreation(beanName);
								try {
									return createBean(beanName, mbd, args);
								}
								finally {
									afterPrototypeCreation(beanName);
								}
							}
						});
						bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
					}
					catch (IllegalStateException ex) {
						throw new BeanCreationException(beanName,
								"Scope '" + scopeName + "' is not active for the current thread; consider " +
								"defining a scoped proxy for this bean if you intend to refer to it from a singleton",
								ex);
					}
				}
			}
			catch (BeansException ex) {
				cleanupAfterBeanCreationFailure(beanName);
				throw ex;
			}
		}

		// Check if required type matches the type of the actual bean instance.
		if (requiredType != null && bean != null && !requiredType.isAssignableFrom(bean.getClass())) {
			try {
				return getTypeConverter().convertIfNecessary(bean, requiredType);
			}
			catch (TypeMismatchException ex) {
				if (logger.isDebugEnabled()) {
					logger.debug("Failed to convert bean '" + name + "' to required type '" +
							ClassUtils.getQualifiedName(requiredType) + "'", ex);
				}
				throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
			}
		}
		return (T) bean;
	}


AbstractAutowireCapableBeanFactory.createBean()方法，该方法是真正的创建Bean的方法的前奏，主要是提供一个工厂后置处理器返回对象的入口。即InstantiationAwareBeanPostProcessor接口的postProcessBeforeInstantiation()方法的扩展调用。

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-code-beanFactory-createBean.png)

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-code-beanFactory-proxyBean.png)

**AbstractAutowireCapableBeanFactory.doCreateBean()方法，该方法调用了两个主要的方法createBeanInstance()方法 和  initializeBean()方法，分别完成实例化和初始化动作。**

	protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)
			throws BeanCreationException {

		// Instantiate the bean.
		BeanWrapper instanceWrapper = null;
		if (mbd.isSingleton()) {
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
		final Object bean = (instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null);
		Class<?> beanType = (instanceWrapper != null ? instanceWrapper.getWrappedClass() : null);
		mbd.resolvedTargetType = beanType;

		// Allow post-processors to modify the merged bean definition.
		synchronized (mbd.postProcessingLock) {
			if (!mbd.postProcessed) {
				try {
					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Post-processing of merged bean definition failed", ex);
				}
				mbd.postProcessed = true;
			}
		}

		// Eagerly cache singletons to be able to resolve circular references
		// even when triggered by lifecycle interfaces like BeanFactoryAware.
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isDebugEnabled()) {
				logger.debug("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, new ObjectFactory<Object>() {
				@Override
				public Object getObject() throws BeansException {
					return getEarlyBeanReference(beanName, mbd, bean);
				}
			});
		}

		// Initialize the bean instance.
		Object exposedObject = bean;
		try {
			populateBean(beanName, mbd, instanceWrapper);
			if (exposedObject != null) {
				exposedObject = initializeBean(beanName, exposedObject, mbd);
			}
		}
		catch (Throwable ex) {
			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
				throw (BeanCreationException) ex;
			}
			else {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
			}
		}

		if (earlySingletonExposure) {
			Object earlySingletonReference = getSingleton(beanName, false);
			if (earlySingletonReference != null) {
				if (exposedObject == bean) {
					exposedObject = earlySingletonReference;
				}
				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
					String[] dependentBeans = getDependentBeans(beanName);
					Set<String> actualDependentBeans = new LinkedHashSet<String>(dependentBeans.length);
					for (String dependentBean : dependentBeans) {
						if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
							actualDependentBeans.add(dependentBean);
						}
					}
					if (!actualDependentBeans.isEmpty()) {
						throw new BeanCurrentlyInCreationException(beanName,
								"Bean with name '" + beanName + "' has been injected into other beans [" +
								StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
								"] in its raw version as part of a circular reference, but has eventually been " +
								"wrapped. This means that said other beans do not use the final version of the " +
								"bean. This is often the result of over-eager type matching - consider using " +
								"'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
					}
				}
			}
		}

		// Register bean as disposable.
		try {
			registerDisposableBeanIfNecessary(beanName, bean, mbd);
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
		}

		return exposedObject;
	}


## 实例化阶段 ##

AbstractAutowireCapableBeanFactory.createBeanInstance()方法

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-code-beanFactory-beanInstance.png)

AbstractAutowireCapableBeanFactory.instantiateBean()方法

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-code-beanFactory-instantiate.png)

SimpleInstantiationStrategy.instantiate()方法

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-code-strategy-instantiate.png)

-------------------------------------------------------------------------------------------------------------
## 初始化阶段 ##

AbstractAutowireCapableBeanFactory.initializeBean()方法，从该方法我们可以清楚的看到类初始化的方法调用顺序。

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-code-beanFactory-initializeBean.png)

BeanNameAware、BeanFactoryAware接口的注入

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-code-beanFactory-initializeBean-1.png)

BeanPostProcessor的postProcessBeforeInitialization()方法调用

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-code-beanFactory-initializeBean-2.png)

InitializingBean接口的afterPropertiesSet()方法调用，以及自定义的init方法调用

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-code-beanFactory-initializeBean-3.png)

BeanPostProcessor的postProcessAfterInitialization()方法调用

![](https://github.com/longtian2/cc3/blob/master/images/spring/spring-code-beanFactory-initializeBean-4.png)


参考文献：

   《Spring 揭秘》 王福强

   http://uule.iteye.com/blog/2094609

联系方式：

https://github.com/longtian2

**如有用请不吝打赏**

![](https://github.com/longtian2/cc3/blob/master/images/wechat_pay.png)