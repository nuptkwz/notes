@[toc]
# 概述

 - 虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制
 - 类加载机制采用懒加载的方式
# 类加载时机
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200524202750877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)
## 初始化
 - 遇到new、getstatic、putstatic或invokestatic这4条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或者设置一个类的静态字段（被final修饰、已在编译器把结果放入常量池的静态字段除外）的时候，以及调用一个类静态方法的时候
 - 使用Java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化
 - 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化
 - 当虚拟机启动时，用户需要指定一个需要执行的主类（包含main方法的那个类）虚拟机会先初始化这个类
### 不被初始化的例子
 - 通过子类引用父类的静态字段，子类不会被初始化
**例如：**
```java
public class Parent {
    static {
        System.out.println("Parent 初始化了...");
    }
    public static int num = 10;
}
```
```java
public class Child extends Parent {
    static {
        System.out.println("child 初始化...");
    }
}
```
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Child.num);
    }
}
```
**控制台打印输出：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200524210958713.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)
 - 通过数组定义来引用类
```java
public class Main {
    public static void main(String[] args) {
        Child [] child = new Child[10];
    }
}
```
 - 调用类的常量
# 类加载的过程
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020052421184586.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzIyNzk3NDI5,size_16,color_FFFFFF,t_70)
## 加载
主要分为以下三步
 - 通过一个类的全限定名来获取定义此类的二进制流
 - 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
 - 在内存中生成一个代表这个类的Class对象，作为这个类的各种数据的访问入口

加载源包括很多种，如文件（Class文件，Jar文件）、网络、计算生成的一个二进制流（$Proxy），由其他文件生成（jsp）等，数据库
## 验证
 - 验证是连接的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害到虚拟机自身的安全
 - 文件格式验证
 - 元数据验证
 - 字节码验证
 - 符号引用验证

## 准备
准备阶段正式为类变量分配内存并设置变量的初始值。这些变量使用的内存都将在方法区中进行分配（这里的初始值并非我们指定的值，而是其默认值，但是如果被final修饰，那么在这个过程中，常量值会被一同指定）。
## 解析
 - 解析阶段是虚拟机将常量池中的符号引用替换为直接引用的过程
 - 类或者接口的解析
 - 字段解析
 - 类方法解析
 - 接口方法解析
## 初始化
- 初始化是类加载的最后一步
- 初始化是执行&lt;clinit&gt;()方法的过程
# 类加载器
- 编译是将我们的java代码编译成字节码文件，加载是将字节码文件加载成二进制字节流，“通过一个类的全限定名来获取描述此类的二进制字节流”这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类，实现这个代码模块称之为类加载器
- 只有被同一个类加载器加载的类才有可能会相等，相同的字节码被不同的类加载器加载的类不相等
## 类加载器的分类
- 启动类加载器
由C++实现，是虚拟机的一部分，用于加载Javahome下的lib目录下的类
- 扩展类加载器
加载javahome下/lib/ext目录中的类
- 应用程序类加载器
加载用户类路径上的所指定的类库
- 自定义类加载器
