## SpringMVC工作流程

- 常用组件：
  DispatcherServlet：处理用户请求，作统一的响应
  HandlerMapping ： 根据url和method找到具体的controller
  Handler(Controller)：对具体的用户请求进行处理。
  HandlerAdapter
  View：接收model对象、Request对象、Response对象，渲染结果输出给response
  ![](https://alcor-1306883605.cos.ap-shanghai.myqcloud.com/my/DzW9S9-20230218204558673.png)

#### 具体流程：

1. 用户通过浏览器发送http请求到DispatcherServlet前端控制器
2. 前端控制器把用户请求发送给HandlerMapping映射器。
3. 映射器通过请求，找到负责处理该请求的处理器，并交给前端控制器。
4. 前端控制器根据处理链中的处理器，找到能够执行该处理的HandlerAdaptor
5. 然后HandlerAdapter会调用对应的Controller去执行
6. Controller执行后返回一个ModelAndView对象，并返回给HandlerAdaptor
7. 返回给前端控制器
8. 前端控制器把把结果返回。

## IOC

- ApplicationContext
  - ClassPathXmlApplicationContext
    - 基于xml配置文件的类路径
  - FileSystemXmlApplicationContext
    - 基于文件xml配置文件在系统中的路径
  - AnnotationConfigApplicationContext
    - 基于注解的context
- refresh() （初始化核心代码
- prepareRefresh()
  - 记录容器时间，标记为已启动

## Bean是什么

本质上是BeanDefinition的实例
BeanDefinition中包含

- 继承的bean信息
- 依赖的bean信息
- 是否懒加载
- 是否注入其他bean中
- 是否是singleton和prototype
  生产bean有的是用反射，有的是用工厂。
- BeanDefinition中包含了生产bean的工厂信息。

### SpringBean的作用域（5种）

1. Singleton: 单例，默认作用域
2. Prototype: 原型，每次使用注入getBean()方法都会new一个新对象
3. Request: 请求作用域，针对每一次HTTP请求都会产生一个新的Bean，仅适用于webApplicationContext环境
4. 
5. 



## 配置文件

application.yml
application.protilfile
bootstrap.yml

## 加载Bean

- loadBeanDefinitions()
- 加载各个bean放进beanfactory里

1. XmlBeanDefinitionReader 读取配置信息解析。
2. initBeanDefinitionReader(beanDefinitionReader);
3. loadBeanDefinitions(beanDefinitionReader);

- 使用beanDefinitionMap来保存所有的注册bean信息。
- 使用beanDefinitionNames这个ArrayList来保存beanName
- manualSingletonNames这个LinkedHashSet用来保存手动注册的singleton 的bean信息。
  - 手动：registerSingleton(String beanName, Object singletonObject)这个方法注册的bean信息。



# 面试题：

### ioc

- 控制反转，是一种设计思想，是把一些原来需要手动创建的对象交给spring容器管理，
- 可以简化应用开发，同时解耦合，减少复杂的依赖关系。

### aop

### Bean注册

### Bean定义包含什么

BeanDefinition中包含

- 继承的bean信息
- 依赖的bean信息
- 是否懒加载
- 是否注入其他bean中
- 是否是singleton和prototype
  生产bean有的是用反射，有的是用工厂。
- BeanDefinition中包含了生产bean的工厂信息。


### spring容器获取对象的方式

1. xml配置文件注入
2. beanid获取、bean name获取
3. 注解注入`@Service @Repository`
4. 

### spring事务

1. Spring如何实现事务的
   - `@Transactionsal`注解

### schedule使用

### spring中的循环依赖问题及解决方法

注意：`spring的循环依赖其实是可以关闭的，设置allowCircularReference=false`
[循环依赖](https://ximeneschen.blog.csdn.net/article/details/113246104?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-113246104-blog-122805502.pc_relevant_multi_platform_whitelistv3&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-113246104-blog-122805502.pc_relevant_multi_platform_whitelistv3&utm_relevant_index=1)

- 循环依赖：
  是指对象A与对象B之间相互依赖，构成了一个依赖环

- 三种依赖的情况

1. 构造器循环依赖：抛出BeanCurrentlylnCreationException

   1. 主Bean使用构造方法注入从Bean依赖则会报错，从Bean构造函数注入主Bean依赖，主Bean使用属性值注入则不会报错。

   - 原因是主Bean通过构造方法注入依赖的bean时，无法创建所依赖的bean对象，获取该Bean对象的引用。
   - 创建主Bean的调用顺序：
     `1.调用构造函数 2. 放到三级缓存 3. 属性赋值`其中调用构造函数时会触发依赖的创建。

2. 单例模式下的setter循环依赖：三级缓存处理

3. 非单例循环依赖：抛出异常

- **三级缓存：**
  是Spring内部维护的3个Map
- singletonObjects （一级缓存）它是我们最熟悉的朋友，俗称“单例池”“容器”，缓存创建完成单例Bean的地方。
- earlySingletonObjects（二级缓存）映射Bean的早期引用，也就是说在这个Map里的Bean不是完整的，甚至还不能称之为“Bean”，只是一个Instance.
- singletonFactories（三级缓存） 映射创建Bean的原始工厂

- **在没有循环依赖的情况下，三级缓存的目的？**
  是为了延迟代理对象的创建
  ![](https://alcor-1306883605.cos.ap-shanghai.myqcloud.com/my/VqpRvh-20230218204558807.jpg)

- **如何通过三级缓存解决循环依赖？**

1. 首先注入A，A依赖B，此时在123级缓存中寻找B，没有找到，此时将A放入3级：singletonFactories工厂中，然后加载B
2. 加载b发现b依赖A，于是搜索3级缓存，找到A在3级中，然后把A放到2级缓存中，从3级缓存中删除A
3. 然后将A注入到B中
4. B完成属性填充，完成初始化后，放入1级缓存中，同时删除二级缓存中的B，此时B已经是成品。
5. 对象A得到B，将B加载进A中，然后完成初始化，加载进1级缓存中，同时删除二级缓存中的A。

# springboot

## spring cloud

- 断路器作用

- 服务注册组件
  nacos
  远程调用：Feign
  负载均衡：Ribbon
  线程池：Hystrix
  网关：zuul


- 共有多少种设计模式，在spring中用到了几种，分别用到了哪里？

- spring中的设计模式又分为几种模型？


**spring AOP 的实现方式， siglib 能否代理final修饰的类**

- proxy 通过接口实现，可以代理final，需要传入接口类
- cglib 基于继承，不能代理final


Sprngboot常用注解以及组合注解有哪些（讲部分）


- yml和xml的区别
  都可以作为配置文件
  yml可读性更强，可以基于流来处理

xml适合web传输，核心是数据和内容

- **spring事务传播机制**

