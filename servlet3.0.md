## servlet3.0

#### 原生注解

@WebService

@WebListener

。。。

### 共享库和运行时插件

1. Servlet容器启动会扫描每个jar包的ServletContainerInitalizer的实现

2. 提供ServletContainerInitializer的实现类；文件路径必须绑定在，META-INF/services/javax.servlet.ServletContainerInitializer文件的内容就是ServletContainerInitializer实现类的全类名（xxx.xxx.xxxServlet）；
3. @HandlersType: 使用
4. 实现 ServletContainerInitializer 可以在其中注册web的3大组件



### springMVC 与Tomcat整合

1、web容器在启动的时候，会扫描每个jar包下的META-INF/services/javax.servlet.ServletContainerInitializer
2、加载这个文件指定的类SpringServletContainerInitializer
3、spring的应用一启动会加载感兴趣的WebApplicationInitializer接口的下的所有组件；
4、并且为WebApplicationInitializer组件创建对象（组件不是接口，不是抽象类）
​	1）、AbstractContextLoaderInitializer：创建根容器；createRootApplicationContext()；
​	2）、AbstractDispatcherServletInitializer：
​			创建一个web的ioc容器；createServletApplicationContext();
​			创建了DispatcherServlet；createDispatcherServlet()；
​			将创建的DispatcherServlet添加到ServletContext中；
​				getServletMappings();
​	3）、AbstractAnnotationConfigDispatcherServletInitializer：注解方式配置的DispatcherServlet初始化器
​			创建根容器：createRootApplicationContext()
​					getRootConfigClasses();传入一个配置类
​			创建web的ioc容器： createServletApplicationContext();
​					获取配置类；getServletConfigClasses();
​	
总结：
​	以注解方式来启动SpringMVC；继承AbstractAnnotationConfigDispatcherServletInitializer；
实现抽象方法指定DispatcherServlet的配置信息；

实例如下：

web容器启动的时候创建对象，调用方法来初始化前端控制器

```java
// web.xml的配置内容
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    //获取根容器（spring配置文件）
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[]{RootConfig.class};
    }

    //web容器，springMVC
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{AppConfig.class};
    }

    //dispatchServlet的映射信息
    //  / 拦截所有请求（静态资源，js,png） ,不包括jsp
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```

SpringMVC的配置文件

```java
//springMVC 的子容器， 禁用默认规则
@ComponentScan(value = "com.wzy",
        includeFilters = {
        @ComponentScan.Filter(type=FilterType.ANNOTATION,
            classes = Controller.class)},
        useDefaultFilters = false)
public class AppConfig {
}
```

Spring 的配置文件

```java
/**
 * spring配置文件  排除 扫描controller
 */
@ComponentScan(value = "com.wzy",excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,value = {Controller.class})
})
public class RootConfig {
}
```



#### 定制SpringMVC

1. ```Java
   @EnableWebMvc //开启springMVC的定制功能
   ```

2. 添加配置组件（视图解析器、视图映射、静态资源映射、拦截器）

```Java
//视图解析器配置
@Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        //默认所有的页面都从 /WEB-INF/ xxx.jsp
        //registry.jsp("/WEB-INF/",".jsp");
        registry.jsp();
    }

    //静态资源配置
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {

        configurer.enable();
    }
	//拦截器配置
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyIntercepter()).addPathPatterns("/**");
    }
```

interceptor实现

```java
public class MyIntercepter implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle");
    }

    //页面响应以后执行
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("afterCompletion");
    }
}
```

#### 异步请求

**原生异步请求**

1. 配置WebServlet(value="/async", asyncSupported=true) 设置支持异步模式，继承HTTPServlet
2. 开启异步模式，req.startAsync() 获取异步AsyncContext
3. AsyncContext.start(Runnable) 进行异步处理
4. 调用AsyncContext.complete(); 说明异步处理执行完毕
5. 得到响应对象response = AsyncContext.getResponse();
6. 使用response 写数据

**SpringMVC 异步请求**

controller实例

- 控制器返回Callable

1. Spring异步处理，将task提交到TaskExecutor，使用一个隔离的线程池中的线程执行任务
2. DispatchServlet和所有的Filter推出Web容器，但是response保持打开状态
3. Callable 响应之后，**SpringMVC将请求重新派发给容器**，恢复之前的处理（请求发送了两次，可以添加一个interceptor 来观察拦截的请求）
4. Callable返回的结果，SpringMVC继续进行接下来的操作

```java
@ResponseBody
@RequestMapping("/async01")
public Callable<String> async01(){

    System.out.println("主"+Thread.currentThread() +"\t"+System.currentTimeMillis());
    Callable<String> callable = new Callable<String>() {

        @Override
        public String call() throws Exception {
            System.out.println(Thread.currentThread() +"\t"+System.currentTimeMillis());
            Thread.sleep(3000);
            System.out.println(Thread.currentThread() +"\t"+System.currentTimeMillis());
            return "async,ok";
        }
    };
    System.out.println("主"+Thread.currentThread() +"\t"+System.currentTimeMillis());
    return callable;
}
```

- 返回DeferredResult

  1. 创建控制器，处理过程创建DeferredResult对象，并设置超时时间，和失败时返回值
  2. 使用一个队列保存DeferredResult对象，并调用DeferredResult.setResult();设置结果
  3. 当DeferredResult结果被设置完成，并且未超时，则会返回设置的结果

  ```java
  @ResponseBody
  	@RequestMapping("/createOrder")
  	public DeferredResult<Object> createOrder(){
  		DeferredResult<Object> deferredResult = new DeferredResult<>((long)3000, "create fail...");
  			
  		DeferredResultQueue.save(deferredResult);
  		
  		return deferredResult;
  	}
  	
  	@ResponseBody
  	@RequestMapping("/create") //装作是另一个线程
  	public String create(){
  		//创建订单
  		String order = UUID.randomUUID().toString();
  		DeferredResult<Object> deferredResult = DeferredResultQueue.get();
  		deferredResult.setResult(order);
  		return "success===>"+order;
  	}
  ```

  ```java
  public class DeferredResultQueue {
  	private static Queue<DeferredResult<Object>> queue = new ConcurrentLinkedQueue<DeferredResult<Object>>();
  	public static void save(DeferredResult<Object> deferredResult){
  		queue.add(deferredResult);
  	}	
  	public static DeferredResult<Object> get( ){
  		return queue.poll();
  	}
  }
  ```


