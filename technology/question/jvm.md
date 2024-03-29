# JVM
- Java中如何进行类加载的？
- Java中的双亲委派机制，这样实现有什么好处？

- Jvm有哪几块区域？Java8之后对内存分代做了什么改进？
  基于tomcat的部署，tomcat是基于java来开发的，我们启动的tomcat不是一个系统，tomcat本身就是一个jvm进程，我们的写的系统只不过是一些
  代码，放在tomcat目录里，tomcat会加载我们的代码到jvm里去

  java8以后的内存分代的改进：
  1.java8之前：永久代里放了一些常量池+类信息
  2.java8之后：常量池->堆里面，类信息 -> metaspace（区域）

  Jvm的内存分代模型，年轻代和老年代
  默认情况下eden和两个s区域的内存比值为：8:1:1

  如果eden区域满了，此时必然触发垃圾回收，young gc,ygc,回收没人引用的垃圾对象。
  两种对象不能回收：
  1.方法正在执行，方法中有局部变量正在引用某个对象，这时候这个对象不能被回收
  2.某个类还活着，静态变量引用了这个类，也不能被回收
  被局部变量或者类变量引用的对象不能被回收

- Jvm年轻代垃圾回收算法？对象什么时候转移到老年代？
  stop the world：停止你的jvm里的工作线程的运行，然后扫描所有的对象，判断哪些可以回收，哪些不可以回收
  年轻代和老年代加起来叫做堆内存
  年轻代的垃圾回收算法有：
  a.复制算法：一个eden和两个s区域配合着使用进行垃圾回收(每次eden满了之后，会将存活的对象放到S1区域内，下次又将eden和S1中存活的对象
    放到S2区域内)
  对象什么时候转移到老年代？
  3种场景下年轻代对象会转移到老年代
  1.有的对象熬过了很多次回收，15次垃圾回收，此时会认为这个对象是长期存活的对象(例如spring中每个bean对象就一个，长期存活，一直被使用)
  2.ygc的时候发现存活的对象很多，没办法放到S1、S2区域内，直接放到了老年代
  3.大对象直接放到老年代（放到S1、S2中占空间）

- Jvm老年代的回收算法有哪些？常用的垃圾回收器有哪些？
  老年代的对象越来越多，老年代的内存空间也会满的。（老年代的对象不能使用年轻代的垃圾回收策略，因为老年代的对象大多数都是被长期引用的
  对象，比如spring容器中被管理的bean等）
  1.对老年代而言，垃圾对象没那么多的。最开始使用的是标记-清除算法，就是找出那些垃圾对象，然后直接把垃圾对象在老年代里清理掉。
    标记-清除会导致内存碎片的问题，这样大对象过来没办法被分配
  2.由此引入了标记-整理算法，把老年代里的存活对象标记出来，移动到一起，将存活对象压缩到一片内存空间里去，剩余的空间都是垃圾对象整个给
    清理掉，剩余的都是连续可用的内存空间，解决了内存碎片的问题
  常用的垃圾回收器有哪些？
  1.parnew + cms的组合：jdk8以及之前使用的，
    a.parnew:年轻代多线程进行垃圾回收
    b.cms:老年代使用，分为：初始标记、并发标记、并发清理等,老年代的回收速度是比较慢的，比年轻代要慢10倍以上，cms的垃圾回收算法，刚开始
      标记-清理，标记出来垃圾对象，清理掉一些垃圾对象；整理，把一些存活的对象压缩到一起，避免内存碎片的产生。cms垃圾回收器的思路就是尽
      可能的让垃圾回收和工作线程并发的运行，并发的来执行。
  2.g1直接分代回收：老年代和年轻代都可以回收，新版本慢慢的主推g1垃圾回收器了，以后会淘汰掉parnew+cms的组合。jdk8~jdk9用的比较多一些，
    parnew+cms的组合比较多一些

- 生产环境中的Tomcat是如何设置JVM参数的？是如何检查JVM运行情况的？
  1.

- Jvm实战
  主要是全程专注于JVM生产实践，主要解决JVM生产环境的参数优化，JVM GC问题和JVM OOM问题的处理

- 内存划分
  1.方法区（Metaspace）：存放类的方法区，在JDK1.8之后将方法区叫做元空间（Metaspace），这里主要存放我们自己写的各种类相关信息。
  2.程序计数器：记录当前执行的字节码指令的位置的，也就是目前执行到哪一条字节码指令了。每个线程都有自己的程序计数器，
        专门记录当前这个线程目前执行到哪条字节码指令了。
  3.Java虚拟机栈：用来保存每个方法内的局部变量等数据的。作用：调用执行任何方法时，都会给方法创建栈帧然后
         入栈，方法执行完毕后就会出栈。
  4.Java堆内存：我们在Java堆内存里创建的对象，都是占用内存资源的，而且内存资源有限。





