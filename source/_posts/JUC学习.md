---
title: JUC并发学习笔记
date: 2021-04-08 20:19:08
tags:
  - java
categories:
  - 笔记
post_meta:
  item_text: false
  created_at: true
  updated_at: true
  categories: true
  tags: true  
---

JUC



>  采用源码+文档方式学习

什么是JUC

>  java.util工具包

![image-20210322170330290](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210322170332.png)



>  并发:多线程操作操作一个资源

* cpu一核,模拟多条线程

  >  并行:多个人一起走
* CPU多核,多个线程同时运行

### Wait和sleep

1、来自不同的类
wait ->Object

 sleep -> Thread
2、关于锁的释放
wait会释放锁，sleep睡觉了，抱着锁睡觉，不会释放

3、使用的范国是不同的

* `wait必须在同步代码块中`

* sleep可以再任何地方睡

  

4、是否需要捕获异常
Wait不需要捕获异常

//现在也需要了

sleep必须要捕获异常?

### 锁

#### 传统 Synchronized锁

> 线程就是一个单独的资源类,没有任何附属的操作
> 
> 资源类OOP
> 
> 多线程操作同一个资源类,把资源类丢入线程

```java
// 资源类 OOP
class Ticket {
        // 属性、方法
        private int number = 30;
        // 卖票的方式
        // synchronized 本质: 队列，锁
        public synchronized void sale(){
        if (number>0){
            System.out.println(Thread.currentThread().getName()+"卖出了"+(number- -)+"票,剩余："+number);
        	}
  }
```

```java
import java.time.OffsetDateTime;Lock 接口
/**
* 真正的多线程开发，公司中的开发，降低耦合性
* 线程就是一个单独的资源类，没有任何附属的操作！
* 1、 属性、方法
*/
public class SaleTicketDemo01 {
public static void main(String[] args) {
// 并发：多线程操作同一个资源类, 把资源类丢入线程
Ticket ticket = new Ticket();
// @FunctionalInterface 函数式接口，jdk1.8 lambda表达式 (参数)->{ 代码 }
        new Thread(()->{
            for (int i = 1; i < 40 ; i++) {
            	ticket.sale();
            }
        },"A").start();
    //同上创建名称为B,C的线程并start
    //--------------省略的另外两个线程
        }
}

}
```

#### Lock锁

![image-20210330212342160](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210330212342.png)

`实现类`

![image-20210330212403217](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210330212403.png)

>  默认为非公平锁

![image-20210330212423864](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210330212424.png)

```java
// Lock三部曲
// 1、 new ReentrantLock();
// 2、 lock.lock(); // 加锁
// 3、 finally=> lock.unlock(); // 解锁
class Ticket2 {
// 属性、方法
            private int number = 30;
            Lock lock = new ReentrantLock();
            public void sale(){
                    lock.lock(); // 加锁
                    try {
                    // 业务代码
                    if (number>0){
                    	System.out.println(Thread.currentThread().getName()+"卖出了"+(number--)+"票,剩余："+number);
                    	}
                    } 
                catch (Exception e) {
                   		 e.printStackTrace();
                    } finally {
                  		  lock.unlock(); // 解锁
              }
}
```

* 操作代码

```java
//公平锁：十分不公平：可以插队 （默认）
import java.util.concurrent.locks.ReentrantLock;
public class SaleTicketDemo02 {
public static void main(String[] args) {
// 并发：多线程操作同一个资源类, 把资源类丢入线程
        Ticket2 ticket = new Ticket2();
        // @FunctionalInterface 函数式接口，jdk1.8 lambda表达式 (参数)->{ 代码 }
        new Thread(()->{for (int i = 1; i < 40 ; i++)
      		  ticket.sale();},"A").start();
        new Thread(()->{for (int i = 1; i < 40 ; i++)
       		 ticket.sale();},"B").start();
        new Thread(()->{for (int i = 1; i < 40 ; i++)
       		 ticket.sale();},"C").start();
        }
```



> > Synchronized 和 Lock 区别
> > 1、Synchronized 内置的Java关键字， Lock 是一个Java类
> > 2、Synchronized 无法判断获取锁的状态，Lock 可以判断是否获取到了锁
> > 3、Synchronized 会自动释放锁，lock 必须要手动释放锁！如果不释放锁，死锁
> > 4、Synchronized 线程 1（获得锁，阻塞）、线程2（等待，傻傻的等）；Lock锁就不一定会等待下
> > 去；可以设置等待超时?
> > 5、Synchronized 可重入锁，不可以中断的，非公平；Lock ，可重入锁，可以 判断锁，非公平（可以
> > 自己设置）；
> > 6、Synchronized 适合锁少量的代码同步问题，Lock 适合锁大量的同步代码！
> > 锁是什么，如何判断锁的是谁！
> > 4、生产者和消费者问题
> > 面试的：单例模式、排序算法、生产者和消费者、死锁

### 生产者和消费者问题

`判断要使用if,不能使用while`:见多线程笔记

#### lock实现

> 并发资源类

```java
// 判断等待，业务，通知
class Data2{ // 数字 资源类
private int number = 0;
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();
//condition.await(); // 等待
//condition.signalAll(); // 唤醒全部
//+1
public void increment() throws InterruptedException {
        lock.lock();
        try {
        // 业务代码
        while (number!=0){ //0
        // 等待
        condition.await();
}
	number++;
	System.out.println(Thread.currentThread().getName()+"=>"+number);
// 通知其他线程，我+1完毕了
	condition.signalAll();
} 
    catch (Exception e) {
	e.printStackTrace();
	} finally {
l	ock.unlock();
	}
}
//-1
public synchronized void decrement() throws InterruptedException {
        lock.lock();
        try {
       		 while (number==0){ // 1
        // 等待
       		 condition.await();
        }
        number--;
        System.out.println(Thread.currentThread().getName()+"=>"+number);
        // 通知其他线程，我-1完毕了
        condition.signalAll();
        } catch (Exception e) {
       		 e.printStackTrace();
        } finally {
      		  lock.unlock();
        }
        }
}
```

> 并发访问

```java
package com.kuang.pc;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
public class B {
public static void main(String[] args) {Data2 data = new Data2();
        new Thread(()->{
                for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
        }
        },"A").start();
            new Thread(()->{
            for (int i = 0; i < 10; i++) {
            try {
           			 data.decrement();
            } catch (InterruptedException e) {
           		 e.printStackTrace();
           	 }
            }
            },"B").start();
            new Thread(()->{
            for (int i = 0; i < 10; i++) {
            try {
           			 data.increment();
            } catch (InterruptedException e) {
            		e.printStackTrace();
            }
            }
            },"C").start();
            new Thread(()->{
            for (int i = 0; i < 10; i++) {
            try {
            data.decrement();
            } catch (InterruptedException e) {
            e.printStackTrace();
            }
            }
            },"D").start();
}
}
```

任何一个新的技术，绝对不是仅仅只是覆盖了原来的技术，优势和补充！

> Condition 精准的通知和唤醒线程
> 
> `condition1.await();`
> 
> `condition2.signal();`

```java
public void printA(){
lock.lock();
try {
// 业务，判断-> 执行-> 通知
while (number!=1){
// 等待
condition1.await();
}
System.out.println(Thread.currentThread().getName()+"=>AAAAAAA");
// 唤醒，唤醒指定的人，B
number = 2;
condition2.signal();
} catch (Exception e) {
e.printStackTrace();
} finally {
lock.unlock();
}
}
```

### 8锁问题

> 如何判断锁的是谁！永远的知道什么锁，锁到底锁的是谁！

### Callable

1、可以有返回值 

2、可以抛出异常 

3、方法不同，run()/ call()

```java
public Integer call() {
System.out.println("call()"); // 会打印几个call
// 耗时的操作
return 1024;
}
```



```java
public class CallableTest {
public static void main(String[] args) throws ExecutionException,
InterruptedException {
// new Thread(new Runnable()).start();
// new Thread(new FutureTask<V>()).start();
// new Thread(new FutureTask<V>( Callable )).start();
new Thread().start(); // 怎么启动Callable
MyThread thread = new MyThread();
FutureTask futureTask = new FutureTask(thread); // 适配类
new Thread(futureTask,"A").start();
new Thread(futureTask,"B").start(); // 结果会被缓存，效率高
Integer o = (Integer) futureTask.get(); //这个get 方法可能会产生阻塞！把他放到
最后
// 或者使用异步通信来处理！
System.out.println(o);
}
}
class MyThread implements Callable<Integer> {
@Override
public Integer call() {
System.out.println("call()"); // 会打印几个call
// 耗时的操作
return 1024;
}}
```

> 1、有缓存 2、结果可能需要等待，会阻塞！

### 常用的辅助类(必会)

#### CountDownLatch

```java
public class CountDownLatchDemo {
public static void main(String[] args) throws InterruptedException {
// 总数是6，必须要执行任务的时候，再使用！
CountDownLatch countDownLatch = new CountDownLatch(6);
for (int i = 1; i <=6 ; i++) {
new Thread(()->{
System.out.println(Thread.currentThread().getName()+" Go out");
countDownLatch.countDown(); // 数量-1
},String.valueOf(i)).start();
}
countDownLatch.await(); // 等待计数器归零，然后再向下执行
System.out.println("Close Door");
}
}
```

`countDownLatch.countDown();` // 数量-1

`countDownLatch.await();`

 // 等待计数器归零，然后再向下执行 每次有线程调用 countDown() 数量-1，假设计数器变为0，countDownLatch.await() 就会被唤醒，继续 执行！

#### CyclicBarrier

> > 加法计数器

#### Semaphore

>   信号量
> 
>  semaphore.acquire() 获得，假设如果已经满了，等待，等待被释放为止
>  semaphore.release(); 释放，会将当前的信号量释放 + 1，然后唤醒等待的线程！ 作用： 多个共享资源互斥的使用！并发限流，控制最大的线程数！


```java
public class SemaphoreDemo {
public static void main(String[] args) {
// 线程数量：停车位! 限流！
Semaphore semaphore = new Semaphore(3);
for (int i = 1; i <=6 ; i++) {
new Thread(()->{
// acquire() 得到
try {
semaphore.acquire();
System.out.println(Thread.currentThread().getName()+"抢到车
位");
TimeUnit.SECONDS.sleep(2);
System.out.println(Thread.currentThread().getName()+"离开车
位");
} catch (InterruptedException e) {
e.printStackTrace();
} finally {
semaphore.release(); // release() 释放
}
},String.valueOf(i)).start();
}
}
}
```

### 读写锁

```java
/**
* 独占锁（写锁） 一次只能被一个线程占有
* 共享锁（读锁） 多个线程可以同时占有
* ReadWriteLock
* 读-读 可以共存！
* 读-写 不能共存！
* 写-写 不能共存！
*/
public class ReadWriteLockDemo {
public static void main(String[] args) {
MyCache myCache = new MyCache();
// 写入
for (int i = 1; i <= 5 ; i++) {
final int temp = i;
new Thread(()->{myCache.put(temp+"",temp+"");
},String.valueOf(i)).start();
}
// 读取
for (int i = 1; i <= 5 ; i++) {
final int temp = i;
new Thread(()->{
myCache.get(temp+"");
},String.valueOf(i)).start();
}
}
}
// 加锁的
class MyCacheLock{
private volatile Map<String,Object> map = new HashMap<>();
// 读写锁： 更加细粒度的控制
private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
private Lock lock = new ReentrantLock();
// 存，写入的时候，只希望同时只有一个线程写
public void put(String key,Object value){
readWriteLock.writeLock().lock();
try {
System.out.println(Thread.currentThread().getName()+"写入"+key);
map.put(key,value);
System.out.println(Thread.currentThread().getName()+"写入OK");
} catch (Exception e) {
e.printStackTrace();
} finally {
readWriteLock.writeLock().unlock();
}
}
// 取，读，所有人都可以读！
public void get(String key){
readWriteLock.readLock().lock();
try {
System.out.println(Thread.currentThread().getName()+"读取"+key);
Object o = map.get(key);
System.out.println(Thread.currentThread().getName()+"读取OK");
} catch (Exception e) {
e.printStackTrace();
} finally {
readWriteLock.readLock().unlock();
}
}
}
/**
* 自定义缓存
*/
class MyCache{
private volatile Map<String,Object> map = new HashMap<>();10、阻塞队列
阻塞队列：
// 存，写
public void put(String key,Object value){
System.out.println(Thread.currentThread().getName()+"写入"+key);
map.put(key,value);
System.out.println(Thread.currentThread().getName()+"写入OK");
}
// 取，读
public void get(String key){
System.out.println(Thread.currentThread().getName()+"读取"+key);
Object o = map.get(key);
System.out.println(Thread.currentThread().getName()+"读取OK");
}
}
```

### 阻塞队列

什么情况下我们会使用 阻塞队列：多线程并发处理，线程池！ 学会使用队列 添加、移除 

![image-20210330214037634](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210330214037.png)

### 线程池(重点)

`线程池：三大方法、7大参数、4种拒绝策略`

> > ![image-20210323151912115](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210323151912.png)
> >


> > ![image-20210323151944753](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210323151944.png)
> >
> > 
> >
> >
> > ![输入图片描述](JUC_md_files/01_20210325150248.png?v=1&type=image&token=V1:8HEaAjBtXmTIvprHJ7YSw1om8WE1nF9vrzOwwCSIJw8)
> > ![输入图片描述](JUC_md_files/02_20210325150257.png?v=1&type=image&token=V1:bQD8d945Z777r5xwEbmNVNoJMKc3tqpOOogHR_5od-o)
> > ![输入图片描述](JUC_md_files/03_20210325150330.png?v=1&type=image&token=V1:gpwseHVrgAL5RKm_0c2-yMuJRc7vJccqH1R98cDNaJI)
> > ![输入图片描述](JUC_md_files/04_20210325150340.png?v=1&type=image&token=V1:5RHrJAF1fsp4VFh1zIjp-PdUB-pnFfZT3ERcqZyW67o)
> > ![输入图片描述](JUC_md_files/05_20210325150348.png?v=1&type=image&token=V1:_D7GTafOz3trfRN0aCyOy8aIRv7BxYY_LGwzyCRbBvg)
> > ![输入图片描述](JUC_md_files/06_20210325150356.png?v=1&type=image&token=V1:fSWbriyf80g3GqY-bUR_Q0Y4HwtfV47WVTE7p3mOeZE)
> > ![输入图片描述](JUC_md_files/07_20210325150407.png?v=1&type=image&token=V1:fsWIsw_fCC9UkhX28Fyu8PmYv5VbujnnyJ0oiWCw8t8)\
> > ![输入图片描述](JUC_md_files/08_20210325150417.png?v=1&type=image&token=V1:kGqW0DrYgT4PF3gTUbg-Etl2WipuUsC4JyEdKrzKpcA)
> > ![输入图片描述](JUC_md_files/09_20210325150425.png?v=1&type=image&token=V1:T_9z9K9Wj3sZr8xQ_Fg4818LkU6NDn46NWbXDRK0e6E)
> > ![输入图片描述](JUC_md_files/10_20210325150435.png?v=1&type=image&token=V1:sYs64Wqv1V6-Hw2pZHVXftibLBH_yytdeCr_8eoHuQE)
> > ![输入图片描述](JUC_md_files/10_20210325150444.png?v=1&type=image&token=V1:elZuMttx19IJGRFHxB_PhFoAe5Iv-LQbgnHlWkuFrW8)
> > ![输入图片描述](JUC_md_files/11_20210325150506.png?v=1&type=image&token=V1:lqJVF59Kpo5GeUeAzeowYSQoUe7DyB3zSN3oTUTzsY0)
> > ![输入图片描述](JUC_md_files/12_20210325150515.png?v=1&type=image&token=V1:YLvkwlFgsseRDarnJc4ZODiQx6bfCpb2nRv_iRcNBKg)![输入图片描述](JUC_md_files/13_20210325150524.png?v=1&type=image&token=V1:mC0S6cOo7mRCKvY5fd0Npgc-ZJLCHA59psB2L_wTZqw)![输入图片描述](JUC_md_files/14_20210325150533.png?v=1&type=image&token=V1:-1DXkutrjaCNDNOix5P-jmVi6fAq5Au-n1VUDqZG5Os)![输入图片描述](JUC_md_files/15_20210325150542.png?v=1&type=image&token=V1:E_ZzLxZoW0PoD9y4DVHajyuBVArL9jnC6FTN5ohMo6U)![输入图片描述](JUC_md_files/16_20210325150550.png?v=1&type=image&token=V1:wUq3fM9Dfr_AS0C0hUx5yEea7eZQPxuRI2wuKj8S5F4)![输入图片描述](JUC_md_files/17_20210325150600.png?v=1&type=image&token=V1:dySM0LHdejxhlv_UmEpBeGjfOlJkfyDCq66YmftZDQw)![输入图片描述](JUC_md_files/18_20210325150608.png?v=1&type=image&token=V1:fjetujmUR3gP1peHXiX86gdo6yyAg4yJDhpKGwwkmxM)![输入图片描述](JUC_md_files/19_20210325150616.png?v=1&type=image&token=V1:Y9cQig7yytkhQ_HCIHOCCM_vG2FPj1RUrOewccb3x64)![输入图片描述](JUC_md_files/19_20210325150622.png?v=1&type=image&token=V1:HtYkcgipjqRojL2wYf0eA0M-MJz76etv2VS3it-dHq4)![输入图片描述](JUC_md_files/20_20210325150628.png?v=1&type=image&token=V1:YrgACay29Q1Gam8Z0wSg9eohoIZA2OrDZdK85QU2hOA)![输入图片描述](JUC_md_files/21_20210325150635.png?v=1&type=image&token=V1:z0fZy0MQeHiVJ7yuCPlIkt35QPsKUXGphXEwwIjnNmY)![输入图片描述](JUC_md_files/22_20210325150641.png?v=1&type=image&token=V1:sANnjjRAGW1D6jVWxukwxyJpntA18_qQUGEqZ01Qe2U)![输入图片描述](JUC_md_files/23_20210325150647.png?v=1&type=image&token=V1:SUi-uwu7yPzW_YxlgfoBmAuCpWlE3MFMDxWbA9g8dtw)
### 12:四大函数式接口

lamdba表达式,函数式接口,链式编程,Stream流

![image-20210323154333067](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210323154333.png)

* Function接口:入参分别为参数类型和返回值类型

* 断定型接口,一个输入参数,返回值只能是布尔值

* 消费型接口,生产者接口

* 消费型接口:只有输入,没有返回值

* 生产者接口,没有输入只有返回值

* > > 函数式接口都可以用lambda接口简化

* > > 泛型,枚举,反射,传统程序员常用

### Stream流式计算

`操作:存储+计算`

* 存储:集合,mysql

* 计算都应该交给流来操作

![image-20210323160253580](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210323160253.png)

![image-20210323160507757](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210323160508.png)10

* 打印传入的参数

![image-20210323160621019](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210323160621.png)

![image-20210323160733487](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210323160733.png)

* 每个方法返回this就能链式

### 14:ForkJoin

> > 分支合并
> >
> > * jdk1.7之后出来的
> > * 并行执行任务,提高效率
> > * 跳过

### 15:异步回调



### 16:JMM

### 17:Volatile可见性及其非原子性验证                                                                                                                                                                                                                                                                                                                                                                                                  

* 保证可见性

* 不保证原子性

* ACID中就有原子性:不可分割

* 幻读,脏读

  `atomic包`

  #### 使用原子类解决原子性问题



### 指令重排

> 源代码->编译器优化的重排,指令并行也可能会重排->内存系统也会重排-->执行
> 
> * 处理器进行指令重排的时候,会考虑数据的依赖性
> * 

volatile避免指令重排

* 内存屏障.cpu指令,作用:
* 1:保证特定的操作的执行顺序
* 可以保证某些变量的内存可见性

### DCL懒汉式就使用了volatile

* 饿汉式单例--构造器私有

![image-20210324210646067](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210324210652.png)

* 可能浪费内存

懒汉式

* 先构造器私有
* 不为空则再为创建
* ![image-20210324211031110](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210324211031.png)
* 多线程情况下失败
* 如果==null,先上锁
* 双重检测锁懒汉式
* lazayman=new lazyman()不是原子性操作
* * 分配内存空间
  * 执行构造方法,初始化对象
  * 把对象指向空间
  * 如果走成132
  * 其他线程发现空间里有对象,但是实际上未初始化,返回了未初始化的对象
  * ![image-20210324211618757](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210324211618.png)
* ![image-20210324211142656](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210324211142.png)
* 利用反射破坏单例模式
* ![image-20210324211902466](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210324211902.png)
* 防止
* 给构造器加个锁,如果已经有对象,
* ![image-20210324211944040](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210324211944.png)

### 枚举:

> jdk1.5新增
> 
> 本身也是一个class类

### CAS

> compareAndSwap:比较并交换

>  如果期望的值达到了,则更新,否则不更新

![image-20210324213529144](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210324213529.png)

java无法操作内存

java可以调用C++native

C++可以操作内存

java的后门,可以通过这个操作内存

![image-20210324213831438](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210324213831.png)

offset:内存偏移

![image-20210324214022006](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210324214022.png)

* 内存操作,效率很高
* 自旋锁,不停旋转直到成功为止
* ![image-20210324214144352](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20210324214145.png)
* CAS:ABA问题(狸猫换太子)
* * 比较当前工作内存中的值和主内存的值,如果之歌值是期望的,那么执行操作,如果不是就一直循环
  * 好处:自带原子性
  * 缺点 :
  * 循环会耗时
  * 一次性只能保证一个共享变量的原子性
* UnSafe类
* 

### ABA问题

CAS时捣乱线程改过又改回去了

* 乐观锁,悲观锁

### 原子引用

TimeUnit.secondes.sleep()来延时

像SQl插入后版本号提升1

![image-20210324220212708](C:/Users/Lenovo/AppData/Roaming/Typora/typora-user-images/image-20210324220212708.png)

原子类型改为原子引用,加上了时间戳

乐观锁有CAS和版本号两种解决方式

![image-20210324220422410](C:/Users/Lenovo/AppData/Roaming/Typora/typora-user-images/image-20210324220422410.png)

### 此处需要敲代码,还有单例模式这些

### 21:对各种锁的理解

#### 公平锁,非公平锁

* 公平锁暖插队,公平锁可以插队
* 非公平锁,可以插队(默认都是非公平)
* 3s的不等待很多秒的
* Lock =new RestrantLock()
* 参数传入True则为公平锁

#### 可重入锁/递归锁

Re en trant:Reentrant
001

![image-20210324220914187](C:/Users/Lenovo/AppData/Roaming/Typora/typora-user-images/image-20210324220914187.png)

