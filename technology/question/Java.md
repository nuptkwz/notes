# Java
- HashMap的key要注意什么？
- HashMap原理？hash冲突了怎么办？为什么会线程不安全，具体哪里不安全？
- JDK1.8中对hash算法和寻址算法是如何优化的？
- 产线中如果有没有遇到内存爆满、cpu飙升的情况？
- 如何排查GC情况？
- ThreadLocal原理，为什么用弱引用？弱引用和强引用的区别？
- 产线cpu飙升很高，如何排查？
- BIO、NIO、AIO、的区别？NIO是异步的吗？
  BIO：同步阻塞（针对的是磁盘文件的IO读写，FileInputStream,BIO卡在那里，直到你读写完成了才可以）
  NIO: 同步非阻塞（通过NIO的）
  AIO: 基于Proactor模型，是异步非阻塞模型


- Synchronized和Lock的区别？

- 进程间的通信方式？
  管道（pipe）、命名管道（fifo）、消息队列、共享内存

- Socket工作原理
  不同层的协议：
  应用层：http协议
  传输层：tcp协议，socket属于传输层的这么一个编程规范
  网络层：ip协议
  数据链路层：以太网协议

