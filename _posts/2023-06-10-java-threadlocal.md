---
title: Java ThreadLocal
date: 2023-06-10
categories: [Programing]
tags: [java, concurrency]
---

Java `ThreadLocal` 能够让你创建只能由同一线程读取和写入变量，因此，即使两个线程执行相同的代码，并且改代码引用了相同的 ThreadLocal ,这两个线程也看不到彼此的 ThreadLocal 变量。

## 创建一个 ThreadLocal

``` java
private ThreadLocal threadLocal = new ThreadLocal();
```

## 设置 ThreadLocal 的值

``` java
threadLocal.set("A thread local value");
```

## 获得 ThreadLocal 的值

```java
String threadLocalValue = (String) threadLocal.get()
```

## 移除 ThreadLocal 的值

```java
threadLocal.remove();
```

## ThreadLocal 的泛型用法

```java
private ThreadLocal<String> threadLocal = new ThreadLocal<String>();

threadLocal.set("Hello ThreadLocal");
String threadLocalValue = threadLocal.get();
```

## 初始化 ThreadLocal 的值

有两种方式可以指定 ThreadLocal 的初始值

1. 创建一个重写 `initialValue()` 方法的 ThreadLocal 子类；
2. 使用 `Supplier` 接口实现创建 ThreadLocal；

### 重写 initialValue()

```java
private ThreadLocal threadLocal = new ThreadLocal<String>(){

    @Override
    protected String initialValue(){
        return String.valueOf(System.currentTimeMillis());
    }
};
```

注意如果 `initialValue()` 方法返回相同的对象，那么所有线程仍然会看到相同的对象。

### Supplier 实现

向 ThreadLocal 的静态方法 `withInitial(Supplier)` 传递一个 `Supplier` 实现。

```java
ThreadLocal threadLocal = ThreadLocal.withInitial(
    () -> String.valueOf(System.currentTimeMillis()));
```

## 延迟初始 ThreadLocal 的值

在某些场景中不能用标准的方法初始化值，例如当设置值时，依赖的某些配置不可用。

例如 `SimpleDateFormat` 不是线程安全的, 所以可以为每个线程创建一个 SimpleDateFormat。

```java

private ThreadLocal<SimpleDateFormat> simpleDateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>();

public String format(Date date) {
    SimpleDateFormat simpleDateFormat = getThreadLocalSimpleDateFormat();
    return simpleDateFormat.format(date);
}

private SimpleDateFormat getThreadLocalSimpleDateFormat() {
    SimpleLocalDateFormat simpleDateFormat = simpleDateFormatThreadLocal.get();
    if (null == simpleDateFormat) {
        simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        simpleDateFormatThreadLocal.set(simpleDateFormat);
    }
}
```

## InheritableThreadLocal

`InheritableThreadLocal` 类是 `ThreadLocal` 的子类，与之相比，而是将访问权限授予一个线程和这个线程创建的所有子线程。

```java

public class InheritableThreadLocalExample {
  public static void main(String[] args) {
    ThreadLocal<String> threadLocal = new ThreadLocal<>();
    InheritableThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();

    Thread thread1 =
        new Thread(
            () -> {
              System.out.println("Thread 1");
              threadLocal.set("Thread 1 -- threadLocal");
              inheritableThreadLocal.set("Thread 1 -- inheritableThreadLocal");
              System.out.println(threadLocal.get());
              System.out.println(inheritableThreadLocal.get());

              Thread childThread =
                  new Thread(
                      () -> {
                        System.out.println("Child Thread");
                        System.out.println(threadLocal.get());
                        System.out.println(inheritableThreadLocal.get());
                      });
              childThread.start();
            });
    thread1.start();
    Thread thread2 =
        new Thread(
            () -> {
              try {
                Thread.sleep(3000);
              } catch (InterruptedException e) {
                e.printStackTrace();
              }
              System.out.println("Thread 2");
              System.out.println(threadLocal.get());
              System.out.println(inheritableThreadLocal.get());
            });
    thread2.start();
  }
}
```

输出

```
Thread 1
Thread 1 -- threadLocal
Thread 1 -- inheritableThreadLocal
Child Thread
null
Thread 1 -- inheritableThreadLocal
Thread 2
null
null
```

## ThreadLocal 场景

1. 线程上下文信息传递，当多个方法之间需要传递某些上下文信息，但是又不想通过方法参数传递，可以用 ThreadLocal 来存储这些信息。
2. 事务管理，在需要实现简单的事务管理时，ThreadLocal可以用于存储当前线程的事务状态。比如，在一个数据库访问框架中，可以使用ThreadLocal来存储数据库连接对象，确保在同一线程中使用同一个连接，从而实现线程级别的事务隔离。
3. 线程安全的对象存储：某些对象可能不是线程安全的，但在特定线程中需要使用。通过将这些对象存储在ThreadLocal中，每个线程都可以拥有自己的对象副本，避免了线程安全问题。
4. 性能优化：有些对象的创建和初始化开销很大，可以将这些对象存储在ThreadLocal中，每个线程只需要创建一次并复用，避免了频繁的创建和销毁过程。

需要注意的是，ThreadLocal使用不当容易导致内存泄漏，因为它的生命周期和线程的生命周期相关。在使用完ThreadLocal后，及时清理其存储的对象，避免长时间占用内存。
