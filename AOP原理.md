```java
*
* 日志切面类 LogAspects
*  通知方法
*      前置通知:
*      后置通知: 方法正常、异常结束都执行
*      返回通知：正常返回 后执行
*      异常通知：
*      环绕通知：动态代理手动推进目标方法执行
*  给切面类的目标方法标注上何时运行（通知注解）
*  将切面类和业务逻辑类（加入到容器中）
*  告诉spring 谁是 切面类
*  启用基于注解的AOP功能 @EnableAspectJAutoProxy
*  joinPoint 只能出现在参数的第一个位置
*
*  主要步骤： 
*   将业务逻辑类和切面类都加入到容器中，告诉Spring那个是切面类
*  在切面类的方法上加上需要的（5种）通知注解
*  开启基于注解的AOP模式
```

### AOP原理

@EnableAspectJAutoProxy 

1. ```Java
   注解实现使用：@Import({AspectJAutoProxyRegistrar.class}) //给容器中导AspectJAutoProxyRegistrar组件
   //利用AspectJAutoProxyRegistrar（实现了ImportBeanDefinitionRegistrar） 自定义给容器中注册bean ，给容器中注册一个AnnotationAwareAspectJAutoProxyCreator（自动代理创建器）
   ```

2. ```Java
   //类关系
   AnnotationAwareAspectJAutoProxyCreator，
   	-> AspectJAwareAdvisorAutoProxyCreator
   		-> AbstractAdvisorAutoProxyCreator
   			-> AbstractAutoProxyCreator
   				-> ProxyProcessorSupport,implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware 
   SmartInstantiationAwareBeanPostProcessor
   	-> InstantiationAwareBeanPostProcessor
   		-> BeanPostProcessor //后置处理器（在bean初始化前后做的事）
   BeanFactoryAware //自动装配
   ```

3. ```java
   AbstractAutoProxyCreator
   	- setBeanFactory() 自动装配相关
       - postProcessBeforeInstantiation() 后置处理器相关
   ```



### AOP流程

1. 传入主配置类，创建IOC容器

2. 注册配置类，调用refresh，刷新容器

3. refresh中调用：registerBeanPostProcessors(beanFactory); 注册bean的后置处理器，来拦截bean的创建！！！

   1. 先获取IOC容器中已经定义了的需要创建对象的所有beanPostProcessor
   2. 给容器中加别的beanPostProcessor  internalAutoProxyCreator
   3. 并对后置处理器进行优先级的区分，优先 PriorityOrdered的接口的
   4. 再给容器中注册实现了  Ordered 接口的BeanPostProcessor
   5. 注册没实现优先级接口的BeanPostProcessor
   6. 注册 BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中
       1. 创建bean的实例
       2. populateBean，给bean的各种属性赋值
       3. initializeBean：初始化bean：
           	1. invokeAwareMethods()，处理Aware接口的方法回调
           	2. 应用后置处理器，初始化之前的方法
           	3. invokeInitMethods，执行自定义的初始化方法
           	4. 应用后置处理器，初始化之后的方法
      4. BeanPostProcessor创建成功（AnnotationAwareAspectJAutoProxyCreator）-》
   7. 把BeanPostProcessor 注册到BeanFactory中beanFactory.addBeanPostProcessor(postProcessor);

   4. 


容器中保存了组件的代理对象（增强后的对象），这个对象保存了详细信息（增强器，目标对象）

cglib：通过一个拦截器链，来顺序执行 

- 正常执行：前置通知-》目标方法-》后置通知-》返回通知
- 出现异常：前置通知-》目标方法-》后置通知-》异常通知

