Spring容器的创建



#### refresh()

- prepareRefresh() 刷新前的预处理
  - initPropertySources() 初始化一些属性设置，子类自定义个性化的属性设置方法
  - getEnvironment().validateRequiredProperties(); 校验属性的合法性
  - this.earlyApplicationEvents = new LinkedHashSet<ApplicationEvent>(); 保存容器中的一些事件
- obtainFreshBeanFactory()：获取beanFactory
  - efreshBeanFactory(); 刷新Beanfactory：创建一个DefaultListableBeanFactory
  - getBeanFactory(); 得到刚刚创建的beanFactory
  - 
- prepareBeanFactory(beanFactory); beanFactory的预处理设置
  - 设置beanFactory的类加载器，支持表达式解析器
  - 添加部分BeanPostProcessor（ApplicationContextAwareProcessor）
  - 设置忽略的自动装配的接口EnvironmentAware、xxx
  - 注册可以解析的自动装配，我们能在任何组件中自动注入，BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext
  - beanFactory.addBeanPostProcessor(new ApplicationListenerDetector
  - 添加编译时的aspectj支持
  - 给beanFactory中注册可用的组件 environment（）
  - 添加系统属性systemproperties
- postProcessBeanFactory(beanFactory);  进行后置处理工作
  - 子类通过重写这个方法在beanFactory 创建并预准备完成以后做进一步的设置
- invokeBeanFactoryPostProcessors(beanFactory); 执行BeanFactoryPostProcessors 
  - BeanFactory的后置处理器，在beanfactory标准初始化之后执行的
  - 两个接口 BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor（额外添加组件）
  - 执行获取所有的BeanFactoryPostProcessor-》获取所有的BeanDefinitionRegistryPostProcessor -》按优先级排序、首先执行实现了PriorityOrdered接口的BeanDefinitionRegistryPostProcessor 
  - 再执行实现 了 Ordered 接口的 BeanDefinitionRegistryPostProcessor -》postProcessor.postProcessBeanDefinitionRegistry(registry);
  - 最后执行其余没实现任何优先级的BeanDefinitionRegistryPostProcessor 
  - 再执行BeanFactoryPostProcessor的方法
    - 剩下的逻辑和上面BeanDefinitionRegistryPostProcessor 的一样
  - 
- registerBeanPostProcessors(beanFactory);  bean的后置处理器，拦截bean的创建过程（BeanPostProcessor ： 实现类）
  - 获取所有的BeanPostProcessor，后置处理器都默认都有PriorityOrdered、Ordered 优先级
  - 依次按照优先级顺序执行
  - 最后注册ApplicationListenerDetector，创建完成后检查是否有bean是实现自InstantiationAwareBeanPostProcessor这些接口，然后注册进去
- initMessageSource(); 初始化  MessageSource 组件（国际化，消息绑定，消息解析）
  - 获取beanfactory
  - 查看是否有类型为 messageSource的组件，有就直接赋值，没有就创建一个默认的DelegatingMessageSource
  - 把创建好的messageSource注册到容器中，获取国际化配置信息的时候，可以自动注入messageSource组件
- initApplicationEventMulticaster(); 初始化事件派发器
  - 拿到beanfactory，获取工厂中applicationEventMulticaster的ApplicationEventMulticaster
  - 上一步没有配置， 就创建一个简单的SimpleApplicationEventMulticaster
  - 将创建的applicationEventMulticaster 添加到 工厂中
- onRefresh(); 留给子类（子容器）
  - 子类重写这些方法，在容器刷新的时候自定义逻辑
- registerListeners(); 给容器中将所有的ApplicationListener注册进来
  - 从容器中拿到所有的ApplicationListener
  - 将每个监听器添加到事件派发器中：getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
  - 派发之前步骤产生的事件
- finishBeanFactoryInitialization(beanFactory); 初始化所有剩下的单实例bean
  - beanFactory.preInstantiateSingletons();初始化单实例bean
    - 获取容器中的所有bean，进行初始化操作
    - 拿到bean的定义信息：RootBeanDefinition类型
    - bean不是抽象的、是单实例的、不是懒加载的
      1. 判断是否是FactoryBean（工厂bean），是否实现FactoryBean接口的bean
      2. 不是工厂bean，利用getBean(beanName); 创建对象（doGetBean（）-》获取缓存中的bean）
      3. 在缓存中获取不到，就开始创建对象流程
      4. 首先标记bean已经被创建
      5. 获取bean的定义信息
      6. 获取bean的依赖的其他bean，如果有就先创建所依赖的bean
      7. 启动单实例的bean的的创建流程（拿到bean的定义-》如果有后置处理器先执行处理器（两种后置处理器））
      8. 
    - 
  - 
- 
- 



### spring 启动过程总结

```java
1）、Spring容器在启动的时候，先会保存所有注册进来的Bean的定义信息；
	1）、xml注册bean；<bean>
	2）、注解注册Bean；@Service、@Component、@Bean、xxx
2）、Spring容器会合适的时机创建这些Bean
	1）、用到这个bean的时候；利用getBean创建bean；创建好以后保存在容器中；
	2）、统一创建剩下所有的bean的时候；finishBeanFactoryInitialization()；
3）、后置处理器；BeanPostProcessor
	1）、每一个bean创建完成，都会使用各种后置处理器进行处理；来增强bean的功能；
		AutowiredAnnotationBeanPostProcessor:处理自动注入
		AnnotationAwareAspectJAutoProxyCreator:来做AOP功能；
		xxx....
		增强的功能注解：
		AsyncAnnotationBeanPostProcessor
		....
4）、事件驱动模型；
	ApplicationListener；事件监听；
	ApplicationEventMulticaster；事件派发：
```