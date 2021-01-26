1.spring启动

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("springStart.xml");
```

2.进入ClassPathXmlApplicationContext的构造函数中

```java
public ClassPathXmlApplicationContext(
      String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
      throws BeansException {
	 // 调用父类的一些方法.前期可以忽略过去
   super(parent);
   // configLocations对象为构造方法中传入的springStart.xml文件.然后通过该方法进行加载对应的xml文件
   setConfigLocations(configLocations);
   if (refresh) {
     // 进入spring核心方法
      refresh();
   }
}
```

3.开始解析refresh()方法

```java
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      // 准备一些前置配置
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      // 生成对应的bean工厂对象
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      // 准备一些bean工厂的配置,设置一些值,忽略一些接口之类的.总体来说也是偏向配置.前期不需要太着重查看
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         // bean工厂后强化处理类.单独启动spring,该方法实现为空.***但是在springboot项目中,springboot启动的时候,该方法就开始进行相关自动装配的内容了***
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         // 执行已经实例化且注册过的BeanFactoryPostProcessors  也很重要,springboot自动装配也是在这里							开始进行相关操作的.总之自动装配基本都是跟BeanFactoryPostProcessors有关.而且spring的扫描包扫描操作也都是在这里的
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         // 注意注意注意 此时是要注册BeanPostProcessors相关操作,而这之前的两个方法都是针对												BeanFactoryPostProcessors在进行操作的,springIOC中的对象有两个阶段,一个阶段是先实例化对象,也就是在内存中开辟空间,后一个阶段则是初始化对象,也就是给实例化出来的对象,进行赋值相关操作.目前这个方法是进行实例化且注册对象,而不是实例化且执行对象.可以看对应的方法注释
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         initMessageSource();

         // Initialize event multicaster for this context.
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         onRefresh();

         // Check for listener beans and register them.
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         // 要完成所有非懒加载的单例的对象实例化,属性填充,初始化前后置增强,以及初始化工作
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         finishRefresh();
      }

      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // Destroy already created singletons to avoid dangling resources.
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
```

4.bean工厂创建之前的准备刷新操作,主要是设置一些常规参数(不太太重要),getEnvironment().validateRequiredProperties();获取系统环境信息对象,可以debug模式下看看对应的对象的数据情况

```java
protected void prepareRefresh() {
   // Switch to active.
   this.startupDate = System.currentTimeMillis();
   this.closed.set(false);
   this.active.set(true);

   if (logger.isDebugEnabled()) {
      if (logger.isTraceEnabled()) {
         logger.trace("Refreshing " + this);
      }
      else {
         logger.debug("Refreshing " + getDisplayName());
      }
   }

   // Initialize any placeholder property sources in the context environment.
   initPropertySources();

   // Validate that all properties marked as required are resolvable:
   // see ConfigurablePropertyResolver#setRequiredProperties
   // 获取当前环境信息
   getEnvironment().validateRequiredProperties();

   // Store pre-refresh ApplicationListeners...
   if (this.earlyApplicationListeners == null) {
      this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
   }
   else {
      // Reset local application listeners to pre-refresh state.
      this.applicationListeners.clear();
      this.applicationListeners.addAll(this.earlyApplicationListeners);
   }

   // Allow for the collection of early ApplicationEvents,
   // to be published once the multicaster is available...
   // 这个很重要,但是目前么有讲到
   this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

5.开始创建bean工厂对象.执行      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

该方法总体的讲,就是创建好工厂对象,同时给bean工厂对象设置一些属性,**并且将bean对象的定义属性参数(BeanDefinition)加载到工厂当中去!**

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
   // 比较重要,需要进去看看
   refreshBeanFactory();
   return getBeanFactory();
}

/**
	 * This implementation performs an actual refresh of this context's underlying
	 * bean factory, shutting down the previous bean factory (if any) and
	 * initializing a fresh bean factory for the next phase of the context's lifecycle.
	 此实现对此上下文的基础bean工厂执行实际的刷新，关闭前一个bean工厂（如果有），并为上下文生命周期的下一阶段			初始化一个新bean工厂。
	 */
	@Override
	protected final void refreshBeanFactory() throws BeansException {
    // 判断是否有bean工厂
		if (hasBeanFactory()) {
      // 销毁对应的bean对象
			destroyBeans();
      // 关闭bean工厂
			closeBeanFactory();
		}
		try {
      // 开始真正创建对应的bean工厂
			DefaultListableBeanFactory beanFactory = createBeanFactory();
      // 设置工厂ID
			beanFactory.setSerializationId(getId());
      // 一些个性化设置
			customizeBeanFactory(beanFactory);
      // 很重要,加载bean定义参数  Definitions(定义).常规理解就是以前的spring.xml中配置的那些标签,也可						能是相关注解 内部的方法命名还是比较明显的,没有其他骚操作掺杂在里面
			loadBeanDefinitions(beanFactory);
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
```

6.执行         postProcessBeanFactory(beanFactory); 单纯spring启动,该方法为空

7.执行         invokeBeanFactoryPostProcessors(beanFactory);

```java
/**
 * Instantiate and invoke all registered BeanFactoryPostProcessor beans,
 * respecting explicit order if given.
 * <p>Must be called before singleton instantiation.
 实例化并调用所有已注册的BeanFactoryPostProcessor Bean，*遵循显式顺序（如果给定的话）。 * <p>必须在单例实例化之前调用。
 
 注意是实例化并且执行调用invoke对应的BeanFactoryPostProcessor
 */
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
   PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());

   // Detect a LoadTimeWeaver and prepare for weaving, if found in the meantime
   // (e.g. through an @Bean method registered by ConfigurationClassPostProcessor)
   if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
      beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
      beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
   }
}
```

8.执行         registerBeanPostProcessors(beanFactory);   此时最好再参考BeanPostProcessor.java接口.该接口中提供了两个方法.postProcessBeforeInitialization()初始化前的后期处理;     postProcessAfterInitialization()初始化后的后期处理;

为什么要在此处讲BeanPostProcessor.java接口呢.因为BeanPostProcessor是针对已经实例化且注册好了的对象,才能进行相关初始化后处理加强操作.得先有对象呀,才能进行后续一系列初始化啊.你没实例化对象出来,你初始化啥呢?   ***Initialization=初始化.但是但是你难道不好奇什么时候才会真正的实例化bean对象么?!***

**再注意一点,此时也是针对BeanPostProcessor bean对象进行实例化并且注册操作,什么对象?BeanPostProcessor对象,不是真正需要用的bean对象哦!特别是spring.xml中配置的<bean>上的对象数据.不是这个时候实例化的呀,是后面核心方法才会进行实例化操作的!!!**

spring工厂中大致的操作是,先注册,然后执行操作,执行操作必须先进行实例化对象,然后初始化对象操作.

```java
/**
 * Instantiate and register all BeanPostProcessor beans,
 * respecting explicit order if given.
 * <p>Must be called before any instantiation of application beans.
 实例化并注册所有BeanPostProcessor Bean，*遵循显式顺序（如果给定）。 * <p>必须在应用程序bean的任何实例化之前调用。
 
 注意是实例化并且进行注册操作.而不是实例化并且invoke执行调用操作
 */
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
   PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
}
```

9.执行initMessageSource();国际化操作.主要解决i18n前端问题

10.执行initApplicationEventMulticaster(); 看注释:为此上下文初始化事件多播器.需要理解spring使用了一个观察者设计模式.该模式主要针对spring中的一系列监听器,为了让监听器在不同的阶段做不同的处理工作.

11.onRefresh(); 空方法,留给子类去实现

12.执行registerListeners();  在步骤10中,创建了一个多播器,现在就要讲对应的监听器注册到多播器中去.

13.重灾区来了!!!最麻烦的他来了!!!执行finishBeanFactoryInitialization(beanFactory);在此之前的所有操作中,都是准备bean工厂,准备工厂后处理强化器,国际化,多播器,监听器,以及注册bean对象初始化强化器等等一系列操作.那么这个方法就是开始真正的去实例化我们项目中需要用到的bean对象了.(推测:在实例化bean对象之后,bean后置强化器才能开始执行!)

```java
/**
 * Finish the initialization of this context's bean factory,
 * initializing all remaining singleton beans.
 *完成该上下文的bean工厂的初始化，*初始化所有剩余的单例bean。
 */
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
   // Initialize conversion service for this context.
   // 一些准备工作
   if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
         beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
      beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
   }

   // Register a default embedded value resolver if no bean post-processor
   // (such as a PropertyPlaceholderConfigurer bean) registered any before:
   // at this point, primarily for resolution in annotation attribute values.
   if (!beanFactory.hasEmbeddedValueResolver()) {
      beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
   }

   // Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
   String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
   for (String weaverAwareName : weaverAwareNames) {
      getBean(weaverAwareName);
   }

   // Stop using the temporary ClassLoader for type matching.
   beanFactory.setTempClassLoader(null);

   // Allow for caching all bean definition metadata, not expecting further changes.
   beanFactory.freezeConfiguration();

   // Instantiate all remaining (non-lazy-init) singletons.
   // 前面都是常规操作,前期不用过度关注,这个方法才是真正开始干活的!!!
   beanFactory.preInstantiateSingletons();
}
```

13.2执行上面的beanFactory.preInstantiateSingletons();代码

```java
public void preInstantiateSingletons() throws BeansException {
   if (logger.isTraceEnabled()) {
      logger.trace("Pre-instantiating singletons in " + this);
   }

   // Iterate over a copy to allow for init methods which in turn register new bean definitions. 遍历一个副本以允许使用init方法，这些方法依次注册新的bean定义。 
   // While this may not be part of the regular factory bootstrap, it does otherwise work fine. 虽然这可能不是常规工厂引导程序的一部分，但可以正常运行。
  	// this.beanDefinitionNames将之前通过beanDefinitionReader读取解析xml文件中的bean定义信息数组转成集合,然后开始一系列操作
   List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

   // Trigger initialization of all non-lazy singleton beans...
   for (String beanName : beanNames) {
      // 获取对应每个bean的定义信息的对象.
      RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
      // 判断是否是抽象属性,判断是单例对象属性,判断是否是懒加载
      // 该if判断逻辑为:不是抽象&&是单例&&不是懒加载,则进入if语句内部执行
      if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
         // isFactoryBean()-->这里引入FactoryBean概念,注意FactoryBean与BeanFactory的区别,spring中又两种创建对象的方式,一种是FactoryBean,一种是BeanFactory. BeanFactory是指只能通过工厂去创建,而FactoryBean是直接创建那种独特的复杂的bean对象,只需要一个该对象即可,可以直接new出来的那种,复杂的bean对象也就不需要专门去通过bean工厂创建.
         if (isFactoryBean(beanName)) {
            Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
            if (bean instanceof FactoryBean) {
               final FactoryBean<?> factory = (FactoryBean<?>) bean;
               boolean isEagerInit;
               if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                  isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                              ((SmartFactoryBean<?>) factory)::isEagerInit,
                        getAccessControlContext());
               }
               else {
                  isEagerInit = (factory instanceof SmartFactoryBean &&
                        ((SmartFactoryBean<?>) factory).isEagerInit());
               }
               if (isEagerInit) {
                  getBean(beanName);
               }
            }
         }
         // 因为一般都不是创建独特的复杂的bean,所以不会走FactoryBean相关逻辑,而开始执行getBean()方法.
         else {
            getBean(beanName);
         }
      }
   }

   // Trigger post-initialization callback for all applicable beans...
   for (String beanName : beanNames) {
      Object singletonInstance = getSingleton(beanName);
      if (singletonInstance instanceof SmartInitializingSingleton) {
         final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
         if (System.getSecurityManager() != null) {
            AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
               smartSingleton.afterSingletonsInstantiated();
               return null;
            }, getAccessControlContext());
         }
         else {
            smartSingleton.afterSingletonsInstantiated();
         }
      }
   }
}
```

13.2.1 执行getBean()方法,传参为BeanName.注意这个时候会去执行doGetBean(); 在spring中,只要是方法名前缀为do的方法.都是具体实际干活的方法.

```java
@Override
public Object getBean(String name) throws BeansException {
   return doGetBean(name, null, null, false);
}
```

13.2.1.1 doGetBean()的具体执行代码如下.因为业务逻辑很复杂,只能通过具体的人工注释来进行挑选重点

```java
/**
 * Return an instance, which may be shared or independent, of the specified bean.
 * @param name the name of the bean to retrieve
 * @param requiredType the required type of the bean to retrieve
 * @param args arguments to use when creating a bean instance using explicit arguments
 * (only applied when creating a new instance as opposed to retrieving an existing one)
 * @param typeCheckOnly whether the instance is obtained for a type check,
 * not for actual use
 * @return an instance of the bean
 * @throws BeansException if the bean could not be created
 */
@SuppressWarnings("unchecked")
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
      @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

   final String beanName = transformedBeanName(name);
   Object bean;

   // Eagerly check singleton cache for manually registered singletons.
   // 认真检查单例缓存中是否有手动注册的单例。 没有的话就开始执行spring的实例化动作
   Object sharedInstance = getSingleton(beanName);
   if (sharedInstance != null && args == null) {
      if (logger.isTraceEnabled()) {
         if (isSingletonCurrentlyInCreation(beanName)) {
            logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
                  "' that is not fully initialized yet - a consequence of a circular reference");
         }
         else {
            logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
         }
      }
      bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
   }

   else {
      // 开始执行具体创建,不会走上面的if内部逻辑
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
         if (parentBeanFactory instanceof AbstractBeanFactory) {
            return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
                  nameToLookup, requiredType, args, typeCheckOnly);
         }
         else if (args != null) {
            // Delegation to parent with explicit args.
            return (T) parentBeanFactory.getBean(nameToLookup, args);
         }
         else if (requiredType != null) {
            // No args -> delegate to standard getBean method.
            return parentBeanFactory.getBean(nameToLookup, requiredType);
         }
         else {
            return (T) parentBeanFactory.getBean(nameToLookup);
         }
      }

      if (!typeCheckOnly) {
         // 注意这里,对bean并进行一个基本的标志.然后开始走下面的try部分代码
         markBeanAsCreated(beanName);
      }

      try {
         final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
         checkMergedBeanDefinition(mbd, beanName, args);

         // Guarantee initialization of beans that the current bean depends on.
         // depends on 在spring.xml中,bean标签里面有个属性是depends on ,该属性表示为是否bean的创建顺序.
         String[] dependsOn = mbd.getDependsOn();
         if (dependsOn != null) {
            for (String dep : dependsOn) {
               if (isDependent(beanName, dep)) {
                  throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                        "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
               }
               registerDependentBean(dep, beanName);
               try {
                  getBean(dep);
               }
               catch (NoSuchBeanDefinitionException ex) {
                  throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                        "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
               }
            }
         }

         // Create bean instance.
         // 上面是在判断属性中是否是有创建顺序的逻辑,这里判断属性中是否为单例对象 也就是<bean scope="singleton"> 如果有,则进入if内部.
         if (mbd.isSingleton()) {
           // 采用JDK8,语法糖方式来执行代码
            sharedInstance = getSingleton(beanName, () -> {
               try {
                  // 这里肯定会执行createBean()方法来创建对象.也就是说,这里才是在创建bean对象.注意,需要提前进入该方法的实现类中进行打断点,不然方法进不去!
                  return createBean(beanName, mbd, args);
               }
               catch (BeansException ex) {
                  // Explicitly remove instance from singleton cache: It might have been put there
                  // eagerly by the creation process, to allow for circular reference resolution.
                  // Also remove any beans that received a temporary reference to the bean.
                  destroySingleton(beanName);
                  throw ex;
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
               Object scopedInstance = scope.get(beanName, () -> {
                  beforePrototypeCreation(beanName);
                  try {
                     return createBean(beanName, mbd, args);
                  }
                  finally {
                     afterPrototypeCreation(beanName);
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
   if (requiredType != null && !requiredType.isInstance(bean)) {
      try {
         T convertedBean = getTypeConverter().convertIfNecessary(bean, requiredType);
         if (convertedBean == null) {
            throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
         }
         return convertedBean;
      }
      catch (TypeMismatchException ex) {
         if (logger.isTraceEnabled()) {
            logger.trace("Failed to convert bean '" + name + "' to required type '" +
                  ClassUtils.getQualifiedName(requiredType) + "'", ex);
         }
         throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
      }
   }
   return (T) bean;
}
```

13.2.1.1.1 注意,这里是上面方法中的createBean的具体实现类方法,需要提前在实现类中打断点.在该方法中寻找一个doCreateBean()方法,在此方法之前的代码都是一些处理操作,前期不用太关注.找到doCreateBean()即可.继续往里面走.

```java
/**
 * Central method of this class: creates a bean instance,
 * populates the bean instance, applies post-processors, etc.
 * @see #doCreateBean
 */
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
      throws BeanCreationException {

   if (logger.isTraceEnabled()) {
      logger.trace("Creating instance of bean '" + beanName + "'");
   }
   RootBeanDefinition mbdToUse = mbd;

   // Make sure bean class is actually resolved at this point, and
   // clone the bean definition in case of a dynamically resolved Class
   // which cannot be stored in the shared merged bean definition.
   Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
   if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
      mbdToUse = new RootBeanDefinition(mbd);
      mbdToUse.setBeanClass(resolvedClass);
   }

   // Prepare method overrides.
   try {
      mbdToUse.prepareMethodOverrides();
   }
   catch (BeanDefinitionValidationException ex) {
      throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
            beanName, "Validation of method overrides failed", ex);
   }

   try {
      // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
      Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
      if (bean != null) {
         return bean;
      }
   }
   catch (Throwable ex) {
      throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
            "BeanPostProcessor before instantiation of bean failed", ex);
   }

   try {
      // 进入该方法!!!
      Object beanInstance = doCreateBean(beanName, mbdToUse, args);
      if (logger.isTraceEnabled()) {
         logger.trace("Finished creating instance of bean '" + beanName + "'");
      }
      return beanInstance;
   }
   catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
      // A previously detected exception with proper bean creation context already,
      // or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
      throw ex;
   }
   catch (Throwable ex) {
      throw new BeanCreationException(
            mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
   }
}
```

13.2.1.1.1.1 继续执行doCreateBean()方法.此时需要确定一个思想,那就是spring中在实例化对象的时候,肯定是通过反射的方式来创建对象的!!!在该方法中,会调用后续一系列的业务逻辑,然后通过反射的方式实例化出对象,同时在该方法中,还会继续针对实例化出的对象,进行一个属性填充操作,然后再进行初始化!!!!!这里还会进行初始化操作!!!!初始化!!!但是前提一定是已经实例化对象出来了!!!

实例化调用createBeanInstance(beanName, mbd, args);

属性填充调用populateBean(beanName, mbd, instanceWrapper);

初始化调用initializeBean(beanName, exposedObject, mbd);;

```java
/**
 * Actually create the specified bean. Pre-creation processing has already happened
 * at this point, e.g. checking {@code postProcessBeforeInstantiation} callbacks.
 * <p>Differentiates between default bean instantiation, use of a
 * factory method, and autowiring a constructor.
 * @param beanName the name of the bean
 * @param mbd the merged bean definition for the bean
 * @param args explicit arguments to use for constructor or factory method invocation
 * @return a new instance of the bean
 * @throws BeanCreationException if the bean could not be created
 * @see #instantiateBean
 * @see #instantiateUsingFactoryMethod
 * @see #autowireConstructor
 */
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
      throws BeanCreationException {

   // Instantiate the bean. 注意该注释
   BeanWrapper instanceWrapper = null;
   // 判断是否是单例的
   if (mbd.isSingleton()) {
      instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
   }
   // 重点来了!!!createBeanInstance().创建bean的实例化.进去进去进去!!!
   if (instanceWrapper == null) {
      instanceWrapper = createBeanInstance(beanName, mbd, args);
   }
  // 至此,对象终于实例化出来了.继续搞事情,搞什么呢,搞初始化啊!!!
   final Object bean = instanceWrapper.getWrappedInstance();
   Class<?> beanType = instanceWrapper.getWrappedClass();
   if (beanType != NullBean.class) {
      mbd.resolvedTargetType = beanType;
   }

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
      if (logger.isTraceEnabled()) {
         logger.trace("Eagerly caching bean '" + beanName +
               "' to allow for resolving potential circular references");
      }
      addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
   }

   // Initialize the bean instance.
   // 初始化当前实例对象.这可是官方注释喔!!!
   Object exposedObject = bean;
   try {
      // 具体开始执行属性填充操作!!!
      populateBean(beanName, mbd, instanceWrapper);
      // 这里才是真正的初始化操作喔!!!
      exposedObject = initializeBean(beanName, exposedObject, mbd);
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
            Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
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
                     "'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.");
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
```

13.2.1.1.1.1.1 继续进入到createBeanInstance()方法中,看看注释.总体的讲该方法就是在具体的创建一个bean对象了.重点

```java
/**
 * Create a new instance for the specified bean, using an appropriate instantiation strategy:
 *使用适当的实例化策略为指定的bean创建一个新实例：
 通俗翻译是 创建一个实例 对于一个指定的bean对象,也就是说在这里执行一个具体的创建bean对象的过程了!!!
 * factory method, constructor autowiring, or simple instantiation.
 * @param beanName the name of the bean
 * @param mbd the bean definition for the bean
 * @param args explicit arguments to use for constructor or factory method invocation
 * @return a BeanWrapper for the new instance
 * @see #obtainFromSupplier
 * @see #instantiateUsingFactoryMethod
 * @see #autowireConstructor
 * @see #instantiateBean
 */
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
   // Make sure bean class is actually resolved at this point.
   Class<?> beanClass = resolveBeanClass(mbd, beanName);

   if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
      throw new BeanCreationException(mbd.getResourceDescription(), beanName,
            "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
   }

   Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
   if (instanceSupplier != null) {
      return obtainFromSupplier(instanceSupplier, beanName);
   }

   if (mbd.getFactoryMethodName() != null) {
      return instantiateUsingFactoryMethod(beanName, mbd, args);
   }

   // Shortcut when re-creating the same bean...
   boolean resolved = false;
   boolean autowireNecessary = false;
   if (args == null) {
      synchronized (mbd.constructorArgumentLock) {
         if (mbd.resolvedConstructorOrFactoryMethod != null) {
            resolved = true;
            autowireNecessary = mbd.constructorArgumentsResolved;
         }
      }
   }
   if (resolved) {
      if (autowireNecessary) {
         return autowireConstructor(beanName, mbd, null, null);
      }
      else {
         return instantiateBean(beanName, mbd);
      }
   }

   // Candidate constructors for autowiring?
   // 一顿操作猛如虎,也不知道在干啥,但是有构造器了,构造器,构造器.有了构造器,就能执行最后的实例化方法了!那就直接找对应的instance方法
   Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
   if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
         mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
      return autowireConstructor(beanName, mbd, ctors, args);
   }

   // Preferred constructors for default construction?
   ctors = mbd.getPreferredConstructors();
   if (ctors != null) {
      return autowireConstructor(beanName, mbd, ctors, null);
   }

   // No special handling: simply use no-arg constructor.
   // 找到了,instantiateBean(),继续进去进去
   return instantiateBean(beanName, mbd);
}
```

13.2.1.1.1.1.1.1 执行instantiateBean(beanName, mbd);方法

```java
/**
 * Instantiate the given bean using its default constructor.
  使用默认的构造方法来实例化当前我们的对象!
 * @param beanName the name of the bean
 * @param mbd the bean definition for the bean
 * @return a BeanWrapper for the new instance
 */
protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
   try {
      Object beanInstance;
      final BeanFactory parent = this;
      if (System.getSecurityManager() != null) {
         beanInstance = AccessController.doPrivileged((PrivilegedAction<Object>) () ->
               // 进入到instantiate()方法中去!!!
               getInstantiationStrategy().instantiate(mbd, beanName, parent),
               getAccessControlContext());
      }
      else {
         beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
      }
      BeanWrapper bw = new BeanWrapperImpl(beanInstance);
      initBeanWrapper(bw);
      return bw;
   }
   catch (Throwable ex) {
      throw new BeanCreationException(
            mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
   }
}
```

13.2.1.1.1.1.1.1.1 进入对应的instantiate(mbd, beanName, parent)中去. 里面的业务逻辑也很复杂,但是只要找到一个方法即可,那就是BeanUtils.instantiateClass(constructorToUse);方法 通过bean工具类中的instantiateClass()方法,传入构造器作为参数来执行

```java
public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
   // Don't override the class with CGLIB if no overrides.
   if (!bd.hasMethodOverrides()) {
      Constructor<?> constructorToUse;
      synchronized (bd.constructorArgumentLock) {
         constructorToUse = (Constructor<?>) bd.resolvedConstructorOrFactoryMethod;
         if (constructorToUse == null) {
            final Class<?> clazz = bd.getBeanClass();
            if (clazz.isInterface()) {
               throw new BeanInstantiationException(clazz, "Specified class is an interface");
            }
            try {
               if (System.getSecurityManager() != null) {
                  constructorToUse = AccessController.doPrivileged(
                        (PrivilegedExceptionAction<Constructor<?>>) clazz::getDeclaredConstructor);
               }
               else {
                  constructorToUse = clazz.getDeclaredConstructor();
               }
               bd.resolvedConstructorOrFactoryMethod = constructorToUse;
            }
            catch (Throwable ex) {
               throw new BeanInstantiationException(clazz, "No default constructor found", ex);
            }
         }
      }
      return BeanUtils.instantiateClass(constructorToUse);
   }
   else {
      // Must generate CGLIB subclass.
      return instantiateWithMethodInjection(bd, beanName, owner);
   }
}
```

13.2.1.1.1.1.1.1.1.1 执行BeanUtils.instantiateClass(constructorToUse);方法..找到找到找到ctor.newInstance(argsWithDefaultValues);方法 当最终执行完构造器.newInstance()方法的时候,对象就真的真的真的真的创建出来了!!!!!!!怎么创建出来的?就是通过反射的方式来创建的!

对象实例化出来了对吧,那么该初始化了吧.开始返回返回返回,返回到哪里呢.返回到13.2.1.1.1.1执行doCreateBean()当中,在方法中寻找populateBean()方法进行属性填充操作.

```java
/**
 * Convenience method to instantiate a class using the given constructor.
 * <p>Note that this method tries to set the constructor accessible if given a
 * non-accessible (that is, non-public) constructor, and supports Kotlin classes
 * with optional parameters and default values.
 * @param ctor the constructor to instantiate
 * @param args the constructor arguments to apply (use {@code null} for an unspecified
 * parameter, Kotlin optional parameters and Java primitive types are supported)
 * @return the new instance
 * @throws BeanInstantiationException if the bean cannot be instantiated
 * @see Constructor#newInstance
 */
public static <T> T instantiateClass(Constructor<T> ctor, Object... args) throws BeanInstantiationException {
   Assert.notNull(ctor, "Constructor must not be null");
   try {
      // 设置为可访问的.也就是暴力访问,防止private修饰
      ReflectionUtils.makeAccessible(ctor);
      if (KotlinDetector.isKotlinReflectPresent() && KotlinDetector.isKotlinType(ctor.getDeclaringClass())) {
         return KotlinDelegate.instantiateClass(ctor, args);
      }
      else {
         Class<?>[] parameterTypes = ctor.getParameterTypes();
         Assert.isTrue(args.length <= parameterTypes.length, "Can't specify more arguments than constructor parameters");
         Object[] argsWithDefaultValues = new Object[args.length];
         for (int i = 0 ; i < args.length; i++) {
            if (args[i] == null) {
               Class<?> parameterType = parameterTypes[i];
               argsWithDefaultValues[i] = (parameterType.isPrimitive() ? DEFAULT_TYPE_VALUES.get(parameterType) : null);
            }
            else {
               argsWithDefaultValues[i] = args[i];
            }
         }
         // 完结撒花,完结撒花.完结撒花!!!!!!!bean对象终于被实例化创建出来了!!!!但是好像还没有进行初始化诶,继续继续继续.
         return ctor.newInstance(argsWithDefaultValues);
      }
   }
   catch (InstantiationException ex) {
      throw new BeanInstantiationException(ctor, "Is it an abstract class?", ex);
   }
   catch (IllegalAccessException ex) {
      throw new BeanInstantiationException(ctor, "Is the constructor accessible?", ex);
   }
   catch (IllegalArgumentException ex) {
      throw new BeanInstantiationException(ctor, "Illegal arguments for constructor", ex);
   }
   catch (InvocationTargetException ex) {
      throw new BeanInstantiationException(ctor, "Constructor threw exception", ex.getTargetException());
   }
}
```

13.2.1.1.1.1.2  在方法doCreateBean()中执行populateBean()方法,开始属性填充操作

```java
/**
 * Populate the bean instance in the given BeanWrapper with the property values
 *使用属性值填充给定BeanWrapper中的bean实例
 也就是说根据具体的配置数据,来对当前对象的一个属性填充操作
 * from the bean definition.
 * @param beanName the name of the bean
 * @param mbd the bean definition for the bean
 * @param bw the BeanWrapper with bean instance
 */
@SuppressWarnings("deprecation")  // for postProcessPropertyValues
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
   if (bw == null) {
      if (mbd.hasPropertyValues()) {
         throw new BeanCreationException(
               mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
      }
      else {
         // Skip property population phase for null instance.
         return;
      }
   }

   // Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
   // state of the bean before properties are set. This can be used, for example,
   // to support styles of field injection.
   if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
      for (BeanPostProcessor bp : getBeanPostProcessors()) {
         if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
               return;
            }
         }
      }
   }

   PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

   int resolvedAutowireMode = mbd.getResolvedAutowireMode();
   if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
      MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
      // Add property values based on autowire by name if applicable.
      if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
         autowireByName(beanName, mbd, bw, newPvs);
      }
      // Add property values based on autowire by type if applicable.
      if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
         autowireByType(beanName, mbd, bw, newPvs);
      }
      pvs = newPvs;
   }

   boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
   boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

   PropertyDescriptor[] filteredPds = null;
   if (hasInstAwareBpps) {
      if (pvs == null) {
         pvs = mbd.getPropertyValues();
      }
      for (BeanPostProcessor bp : getBeanPostProcessors()) {
         if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
               if (filteredPds == null) {
                  filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
               }
               pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
               if (pvsToUse == null) {
                  return;
               }
            }
            pvs = pvsToUse;
         }
      }
   }
   if (needsDepCheck) {
      if (filteredPds == null) {
         filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
      }
      checkDependencies(beanName, mbd, filteredPds, pvs);
   }

   if (pvs != null) {
      applyPropertyValues(beanName, mbd, bw, pvs);
   }
}
```

13.2.1.1.1.1.3 在方法doCreateBean()中执行initializeBean()方法,开始初始化操作!!!这里才是初始化,上面的方法是属性填充操作

```java
/**
 * Initialize the given bean instance, applying factory callbacks
 * as well as init methods and bean post processors.
 * <p>Called from {@link #createBean} for traditionally defined beans,
 * and from {@link #initializeBean} for existing bean instances.
 * @param beanName the bean name in the factory (for debugging purposes)
 * @param bean the new bean instance we may need to initialize
 * @param mbd the bean definition that the bean was created with
 * (can also be {@code null}, if given an existing bean instance)
 * @return the initialized bean instance (potentially wrapped)
 * @see BeanNameAware
 * @see BeanClassLoaderAware
 * @see BeanFactoryAware
 * @see #applyBeanPostProcessorsBeforeInitialization
 * @see #invokeInitMethods
 * @see #applyBeanPostProcessorsAfterInitialization
 */
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
   if (System.getSecurityManager() != null) {
      AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
         invokeAwareMethods(beanName, bean);
         return null;
      }, getAccessControlContext());
   }
   else {
      invokeAwareMethods(beanName, bean);
   }

   Object wrappedBean = bean;
   if (mbd == null || !mbd.isSynthetic()) {
      // 注意这个方法!!!是不是很眼熟 apply是应用的意思.BeanPostProcessorsBeforeInitialization是什么呢???在这之前是不是注册了很多很多的BeanPostProcessors.而在BeanPostProcessors.java接口中就有两个方法,一个是before,一个是after.
     // 这里是调用BeanPostProcessors.java接口的实现类的applyBeanPostProcessorsBeforeInitialization()来进行一个初始化前置处理增强逻辑,
      wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
   }

   try {
     // 执行完初始化前置增强业务逻辑,如果在定义的对象当中有一个初始化的方法.那么就是在这里执行的!!!意思这里才是真正的初始化功能!!!在此之前执行初始化前置增强,然后执行初始化,那么执行完初始化之后,是不是还有后置增强呢?
      invokeInitMethods(beanName, wrappedBean, mbd);
   }
   catch (Throwable ex) {
      throw new BeanCreationException(
            (mbd != null ? mbd.getResourceDescription() : null),
            beanName, "Invocation of init method failed", ex);
   }
   if (mbd == null || !mbd.isSynthetic()) {
      // 这里也要注意,对,这里就是在初始化完成之后,进行一个后置增强操作的!后置处理完成之后,回顾一下,对象已经实例化出来了,也进行了属性填充,而且前后置的增强操作也都完成了,意思真正的对象已经完全完全处理好了!!!也就是说会返回到refresh()中去了,至此整个finishBeanFactoryInitialization(beanFactory);全部执行完毕了!
      wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
   }

   return wrappedBean;
}
```

14.执行最后的finishRefresh();方法,该方法基本就是一些后续的清场工作了.

完结!撒花!撒花!撒花!
