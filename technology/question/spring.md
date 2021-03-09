# spring
- 有没有实现过SpringBoot内置的接口？
- spring的作用域有哪些？
- spring中代理模式原理，cglib代理的原理？
- spring中的IOC机制？
  spring IOC框架，控制反转，依赖注入，可以通过xml文件来进行一个配置，后来进化到基于注解来进行自动依赖注入。
  spring IOC启动的时候会启动这个容器，它会根据这个xml的配置或者你的注解去实例化你的一些bean对象，然后根据xml配置或者注解，
  去对bean对象之间的引用关系，去进行依赖注入，某个bean依赖于另一个bean。容器去管理bean的实例化，它去管理bean与bean之间的关系
  的注入。
  它底层的核心技术就是根据反射来实现的，通过反射直接根据你的类去自己构建对应的对象出来。
  spring IOC最大的贡献就是让类与类之间彻底的解耦。
- spring中的AOP机制？


