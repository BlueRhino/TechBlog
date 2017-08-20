# CopyOnWriteArrayList学习总结
> 以下内容约600字（不含代码），预计阅读需要5分钟
## 简介
CopyOnWriteArrayList类位于java.util.concurrent包，JDK1.5引入，作者为[Doug Lea][1]。CopyOnWriteArrayList是一个线程安全的List，使用“写时复制”的思想在进行所有写入操作（增加、删除等）是都会进行内部存储元素数组的复制。
## 代码分析
### 写时过程
由于采用写时复制的策略，其主要作用过程在于对存储数据进行修改时。以add函数为例。一下为JDK8中实现源码：
	public boolean add(E e) {
	    final ReentrantLock lock = this.lock;
	    lock.lock();
	    try {
	        Object[] elements = getArray();
	        int len = elements.length;
	        Object[] newElements = Arrays.copyOf(elements, len + 1);
	        newElements[len] = e;
	        setArray(newElements);
	        return true;
	    } finally {
	        lock.unlock();
	    }
	}
 
1. 进入函数首先获取ReentrantLock重入锁[^1]；
2. 获取当前对象中实际存储数据的Objecte数组；
3. 复制当前数组并且将数组长度扩展1；
4. 将新数据插入当前数组的最后一位；
5. 将指向原数组的变量置为指向新数组，释放锁，完成工作。
### 对比ArrayList
增加元素代码：
	public boolean add(E e) {
	        ensureCapacityInternal(size + 1);  // Increments modCount!!
	        elementData[size++] = e;
	        return true;
	    }
通过查看代码可知，ArrayList在写入时是没有加锁的，所以多线程情况下使用并不安全，需要程序员手动进行并发控制。引用源码中说明为：
> Note that this implementation is not synchronized. If multiple threads access an ArrayList instance concurrently, and at least one of the threads modifies the list structurally, it must be synchronized externally. (A structural modification is any operation that adds or deletes one or more elements, or explicitly resizes the backing array; merely setting the value of an element is not a structural modification.) This is typically accomplished by synchronizing on some object that naturally encapsulates the list. If no such object exists, the list should be "wrapped" using the Collections.synchronizedList method. This is best done at creation time, to prevent accidental unsynchronized access to the list:
> List list = Collections.synchronizedList(new ArrayList(...));
验证ArrayList线程不安全可使用以下代码
	public class Main {
	
	    static class AddThread implements Runnable {
	        private List<String> list;
	        private CountDownLatch countDownLatch;
	
	        public AddThread(List<String> list, CountDownLatch countDownLatch) {
	            this.list = list;
	            this.countDownLatch = countDownLatch;
	        }
	
	        @Override
	        public void run() {
	            try {
	                countDownLatch.await();
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	            System.out.println(Thread.currentThread().getName());
	            for(int i=0;i<500;i++){
	                list.add("1");
	            }
	        }
	    }
	
	    public static void main(String[] args) {
	        CountDownLatch countDownLatch = new CountDownLatch(1);
	        List<String> list = new ArrayList<>();
	        for (int i = 0; i < 200; i++) {
	            new Thread(new AddThread(list, countDownLatch)).start();
	        }
	        countDownLatch.countDown();
	    }
	}
运行过程中大概率会出现Java.lang.ArrayIndexOutOfBoundsException错误
## 优点
使用CopyOnWriteArrayList可以不用进行手动同步控制，并且由于使用了读写分离的措施，在进行写入时可以同时进行数据的读取，并且数据读取过程中不需要加锁。
## 缺点
根据其实现代码，CopyOnWriteArrayList的缺点同样明显。首先、其每次写入都需要进行数组的复制，所以对于写入操作非常消耗系统资源，尤其是内存资源，这种情况在处理存储较多数据的List时尤其明显。对于gc[^2]也造成了较大压力。其次，由于其在读取时不需要获取锁，造成如写入数据同时进行读取可能造成读取到的数据为旧数组中的数据，可能造成数据的不一致。
## 使用场景
由此可见，CopyOnWriteArrayList比较适合用在大量读取小量写入的场景，并且要求系统对于数据的不一致宽容度较高。
针对写时复制的策略，在必须要进行写入时尽量将多次写入合并为一次写入，减少数组复制次数。

[^1]:	后续补充对于重入锁的介绍

[^2]:	后续补充对于gc知识的总结

[1]:	https://baike.baidu.com/item/Doug%20Lea/6319404?fr=aladdin