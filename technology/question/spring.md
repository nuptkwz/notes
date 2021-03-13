# spring
- 有没有实现过SpringBoot内置的接口？

- spring的作用域有哪些？

- spring中的IOC机制？
  spring IOC框架，控制反转，依赖注入，可以通过xml文件来进行一个配置，后来进化到基于注解来进行自动依赖注入。
  spring IOC启动的时候会启动这个容器，它会根据这个xml的配置或者你的注解去实例化你的一些bean对象，然后根据xml配置或者注解，
  去对bean对象之间的引用关系，去进行依赖注入，某个bean依赖于另一个bean。容器去管理bean的实例化，它去管理bean与bean之间的关系
  的注入。
  它底层的核心技术就是根据反射来实现的，通过反射直接根据你的类去自己构建对应的对象出来。
  spring IOC最大的贡献就是让类与类之间彻底的解耦。

- spring中的AOP机制？
  spring AOP 面向切面编程，spring在运行的时候采用的动态代理技术，AOP的核心技术就是动态代理。它会给你那些类生成动态代理类。

- spring中代理模式原理，cglib代理的原理，它和jdk的动态代理的区别是什么？
  动态代理就是动态的创建一个代理类出来，创建这个代理类的实例对象，在这个里面引用你真正自己写的类，所有的方法调用都是先走代理类的对象，
  它负责做一些代码上的增强，再去调用你写的那个类。

  spring里使用AOP，比如你对一批类和他们的方法做了一个切面，定义好了要在这些类的方法里面增强的代码，spring必然要对那些类生成动态代理，
  在动态代理类里面去执行你定义的一些增强代码。
  a.如果你的类实现了某个接口，spring aop会使用jdk动态代理，生成一个跟你实现同样接口的代理类，构造一个实例对象出来，jdk动态代理，其实
    是在你的类有接口的时候，就会使用
  b.很多时候，我们可能某个类是没有实现接口的，spring aop会改用cglib来生成动态代理，它是生成你的类的一个子类，它可以动态生成字节码，覆盖
    你的一些方法，在方法里面加入增强代码

- spring中的bean是线程安全的吗？
  spring容器中的bean可以分为5个范围(作用域)：
  a.singleton: 默认，每个容器中只有一个bean实例
  b.prototype: 为每一个bean请求提供一个实例
  c.request: 为每一个网络请求创建一个实例，在请求完成之后，bean会失效，并被垃圾回收器回收
  d.session: 与request范围类似，确保每一个session中有一个bean的实例，在session过期后,bean会随之失效
  e.global-session:

  spring中的bean不是线程安全的，对于spring bean来说,singleton是线程不安全的。比如tomcat是一个多线程的模型访问的，
  多线程可能对同一个bean实例变量进行并发的访问，所以说是线程不安全的。
  对于java web系统，一般来说很少在spring bean里放一些实例变量，一般来说他们都是多个组件相互调用，最终去访问数据库的。

- spring中的事务实现原理是什么？聊聊你对事务传播机制的理解？
  a.事务的实现原理：
    如果你加了一个@Transactional注解，此时spring会使用aop的思想，对你这个方法在执行之前，先去开启事务，执行完毕之后，
    根据你方法是否报错，来决定回滚还是提交事务。
  b.事务的传播机制：
    spring的传播机制解决的是两个或者多个方法它们都加了@Transactional注解，在串起来调用的时候，这个事务怎么传播的问题。
    事务传播机制分为以下几种：
    1.PROPAGATION_REQUIRED: 如果当前没有事务就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。
    2.PROPAGATION_SUPPORTS: 支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行（就把它
      当做普通没有事务的方法执行）。
    3.PROPAGATION_MANDATORY: 支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛异常。
    4.PROPAGATION_REQUIRES_NEW: 创建新事务，无论当前存不存在事务，都创建新事物（它自己的事务不会受别人事务的影响，它
      也不会影响别人的事务）。
    5.PROPAGATION_NOT_SUPPORTED: 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
    6.PROPAGATION_NEVER: 以非事务的方式执行，如果当前存在事务，则抛出异常。
    7.PROPAGATION_NESTED: 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED执行（嵌套事务，如果外层的
      事务回滚，会导致内层的事务也回滚；当时如果内层的事务回滚只会回滚自己的事务）。

- Spring Boot的核心架构？
   1.实例化bean：如果要使用一个bean的话
   2.设置对象属性（依赖注入）：它需要去看看你的这个bean依赖了谁，把你依赖的bean也创建出来，给你进行一个注入，比如通过构造函数，setter方法
   3.处理Aware接口：如果这个Bean已经实现了ApplicationContextAware接口，spring容器就会调用我们的bean的setApplicationContext
     (ApplicationContext)方法，传入spring上下文，把spring容器传递给这个bean
   4.BeanPostProcessor：如果想在bean实例构建好了之后，此时在这个时间点，我们想对bean进行一些自定义的处理，那么可以让bean实现了
     BeanPostProcessor接口，那将会调用postProcessBeforeInitialization(Object obj,String s)方法。
   5.InitializingBean与init-method：
     如果Bean在Spring配置文件中配置了init-method属性，则会自动调用其他配置的初始化方法。
   6.如果这个Bean已经实现了BeanPostProcessor接口，那将会调用postProcessAfterInitialization(Object obj,String s)方法。
   7.DisposableBean:
     当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy()方法
   8.destroy-method:
     最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

     创建+初始化一个bean -> spring容器管理的bean长期存活  ->  销毁bean （两个回调函数）

- spring中用到哪些设计模式？
    工厂、单例、代理
  1.工厂模式：spring ioc的核心设计模式的思想体现，它自己就是一个大的工厂，把所有的bean实例都给放在了spring容器里
    （大工厂），如果你要使用这个bean，就找spring容器就可以了。
  2.单例模式：spring 默认来说，对每一个bean走的都是一个单例模式，确保说你的一个类在系统运行期间只有一个实例对象。
  3.代理模式：对一些类的方法切入一些增强的代码，会创建一些动态代理的对象，让你对那些目标对象的访问，先经过动态代理的对象，
    动态代理对象会做一些增强的代码，再去调用你的目标对象

- 能说下Spring MVC核心架构吗？
  1.tomcat的工作线程将请求转发给spring mvc框架的DispatcherServlet
  2.DispatcherServlet去查找@Controller注解的controller,一般controller上会加上@RequestMapping的注解，标注说哪些controller
    是用来处理哪些请求的，此时根据请求的uri去定位到哪个controller来进行处理。
  3.根据@RequestMapping去查找，使用这个controller内的哪个方法来进行请求的处理，对于每一个方法一般也会加上@RequestMapping注解
  4.前后端分离会直接调用我们的controller层某个方法进行请求处理
  5.controller层方法会有一个返回值，以前一般还会走jsp、模板等，把前端页面放在后端的工程里面，返回一个页面模板的名字，spring mvc框架
    使用模板技术，对html页面做了一个渲染；返回json串。前后端分离之后，前端发送一个请求过来，后端只需要返回一个json数据
  6.把渲染之后的html页面返回给浏览器去进行显示；前端负责把html页面渲染给浏览器就可以了。

