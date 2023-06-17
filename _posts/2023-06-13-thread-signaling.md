---
title: Thread Signaling
date: 2023-06-13
categories: [Programing]
tags: [java, concurrency]
---

Java `Thread Signaling` 能让一个线程给其他线程发送信号，并让这些线程等待这个信号。

这个功能是通过 `wait()`、`notify()`、`notifyAll()` 方法实现的，这些方法是属于 `Object` 类。

## wait(), notify(), notifyAll()

Java 有个内部机制，可以让线程等代其他线程的信号时处于非活动状态。

一个线程在任何对象上调用call()后，处于非活动状态直到另外一个线程在那个对象上调用notify() 或 notifyAll() 。为了调用 wait(), notify(), notifyAll(), 调用的线程首先必须获得那个对象的锁。换句话说，调用线程必须从在该对象上同步的同步块内部调用 wait(), nofify()。

```java

public class MonitorObject{}

public class WaitNotify{
    MonitorObject obj = new MonitorObject();

    public void doWait() {
        synchronized(obj) {
            try {
                obj.wait();
            } catch (InterruptedException e){}
        }
    }

    public void doNotify() {
        synchronized(obj) {
            obj.notify();
        }
    }
}
```

当第一个线程调用 doWait(), 它会首先进入一个同步块然后调用对象上的wait(), 调用之后，调用线程会释放这个对象的锁并阻塞，直到另外一个线程调用该对象上的notify() 或者 notifyAll()。

当第二个线程调用 doNotify(), 它会进入对象同步的同步块，然后调用对象上的notify(), 这会唤醒因在同一对象上调用wait()而阻塞的线程，但是, 在这个调用 notify() 线程释放锁之前，这些唤醒的线程依然不能退出wait() 方法。

![Thread signaliing processing](/assets/img/posts/2023/06/thread-signaling.png)

notify() 会唤醒单个线程，如果有多个线程阻塞，不确定唤醒哪个。
notifyAll() 会唤醒所有阻塞的线程。

## 丢失信号

当调用 notify() 或 notifyAll() 方法时，如果没有线程在等待状态，这些方法的调用将被忽略，也就是所谓的信号丢失问题。

如果过在调用 notify() 之前没有等待线程调用 wait() 方法，那么该通知信号将会丢失，这会导致等待线程永远不会被唤醒。

为了避免这个情况，可以使用一个标志变量来表示是否发生了需要通知的事件。当事件发生时，如果没有等待线程，可以将标志设置为true，并在等待线程进入等待状态时进行检查。这样，即使通知信号被错过，等待线程也可以在进入等待状态后立即检查标志并执行相应的操作。

```java
public class MyWaitNotify2{

  MonitorObject myMonitorObject = new MonitorObject();
  boolean wasSignalled = false;

  public void doWait(){
    synchronized(myMonitorObject){
      if(!wasSignalled){
        try{
          myMonitorObject.wait();
         } catch(InterruptedException e){...}
      }
      //clear signal and continue running.
      wasSignalled = false;
    }
  }

  public void doNotify(){
    synchronized(myMonitorObject){
      wasSignalled = true;
      myMonitorObject.notify();
    }
  }
}
```

## 虚假唤醒

即使没有调用 notify() 或 notifyAll(), 等待状态的线程也有可能被唤醒。

为了处理虚假唤醒，通常建议将等待状态的线程放在一个循环中，并使用条件判断来检查是否满足继续执行的条件。这样，在虚假唤醒时，线程会再次进入循环并重新检查条件，从而避免不必要的执行。

```java
public class MyWaitNotify3{

  MonitorObject myMonitorObject = new MonitorObject();
  boolean wasSignalled = false;

  public void doWait(){
    synchronized(myMonitorObject){
      while(!wasSignalled){
        try{
          myMonitorObject.wait();
         } catch(InterruptedException e){...}
      }
      //clear signal and continue running.
      wasSignalled = false;
    }
  }

  public void doNotify(){
    synchronized(myMonitorObject){
      wasSignalled = true;
      myMonitorObject.notify();
    }
  }
}
```

## 场景

1. 生产者-消费者模型：在生产者-消费者模型中，生产者线程负责生成数据，而消费者线程负责消费数据。通过线程信号，生产者线程可以通知消费者线程有新数据可用，从而触发消费者线程的执行。这可以避免消费者线程轮询检查数据是否可用，提高效率。

2. 任务分配与处理：当有多个线程可用于执行任务时，可以使用线程信号来通知空闲线程有任务可执行。这种场景可以用于线程池、并行计算等情况，以提高任务的并发执行能力。

3. 条件满足时执行：当某个条件满足时，线程可以等待该条件，并在条件满足时被其他线程通知。这种场景可以用于解决线程间的依赖关系，确保线程在满足特定条件后再执行。

4. 线程协调与同步：线程信号还可以用于协调和同步多个线程的执行顺序。通过合适的信号机制，线程可以在特定的时刻等待其他线程的完成或达到某个状态，以保证线程间的顺序执行或互斥访问共享资源。
