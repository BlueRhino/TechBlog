# java.util.concurrent.locks.Lock详解
## 简介
java.util.concurrent.locks.Lock接口（以下简称Lock）作者[Doug Lea][1]，对比使用synchronized关键字进行并发控制，Lock接口的实现类可以完成更灵活的加锁操作。
本文以下会详细介绍Lock接口相关功能，并对比Lock接口及synchronized关键字在功能上（非性能对比）的异同。
## Lock接口相关功能介绍

## Lock接口及synchronized关键字对比
synchronized[^1]作为Java语言的并发控制关键字，在多线程编程中十分常用，主要有三种使用方式：
1. 修饰对象方法；
2. 修饰静态方法；
3. 修饰同步方法快。
使用synchronized关键字可以完成对访问共享资源的代码块的显示加锁和隐式释放锁。在代码离开synchronized关键字修饰的区域后，synchronized自动释放锁定。
Lock接口将加锁及解锁操作控制权都交给程序员，这增加了锁的灵活性却也增加了出现逻辑问题的可能性。
## 总结

[^1]:	synchronized深度研究

[1]:	https://baike.baidu.com/item/Doug%20Lea/6319404?fr=aladdin