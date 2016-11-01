---
title: java-design-patterns-singleton
date: 2016-11-01 14:22:14
tags:
	- java
	- design-pattern
	- singleton
categories: java
toc: true
---

### "茴"字的四种写法

`java-design-patterns`中提供了5种单例实现形式，抛开`Enum`实现不谈，逐步来探讨四种实现，主要是`InitializingOnDemandHolderIdiom`方式。

#### 最简单

常规饱汉式

```java
/**
 * Singleton class. Eagerly initialized static instance guarantees thread safety.
 */
public final class IvoryTower {

  /**
   * Static to class instance of the class.
   */
  private static final IvoryTower INSTANCE = new IvoryTower();

  /**
   * Private constructor so nobody can instantiate the class.
   */
  private IvoryTower() {}

  /**
   * To be called by user to obtain instance of the class.
   *
   * @return instance of the singleton.
   */
  public static IvoryTower getInstance() {
    return INSTANCE;
  }
}
```

#### 内存优化

饿汉式

```java
/**
 * Thread-safe Singleton class. The instance is lazily initialized and thus needs synchronization
 * mechanism.
 *
 * Note: if created by reflection then a singleton will not be created but multiple options in the
 * same classloader
 */
public final class ThreadSafeLazyLoadedIvoryTower {

  private static ThreadSafeLazyLoadedIvoryTower instance;

  private ThreadSafeLazyLoadedIvoryTower() {}

  /**
   * The instance gets created only when it is called for first time. Lazy-loading
   */
  public static synchronized ThreadSafeLazyLoadedIvoryTower getInstance() {

    if (instance == null) {
      instance = new ThreadSafeLazyLoadedIvoryTower();
    }

    return instance;
  }
}
```

#### 双重检查的饿汉式优化

上面的饿汉式为了线程安全，将整个方法设置同步，为了提高性能人们引入了双重检查机制。

这里`volatile`关键字的引入很关键。

> http://ifeve.com/doublecheckedlocking/
>
> http://www.infoq.com/cn/articles/double-checked-locking-with-delay-initialization

```java
/**
 * Double check locking
 * <p/>
 * http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html
 * <p/>
 * Broken under Java 1.4.
 *
 * @author mortezaadi@gmail.com
 */
public final class ThreadSafeDoubleCheckLocking {

  private static volatile ThreadSafeDoubleCheckLocking instance;

  /**
   * private constructor to prevent client from instantiating.
   */
  private ThreadSafeDoubleCheckLocking() {
    // to prevent instantiating by Reflection call
    if (instance != null) {
      throw new IllegalStateException("Already initialized.");
    }
  }

  /**
   * Public accessor.
   *
   * @return an instance of the class.
   */
  public static ThreadSafeDoubleCheckLocking getInstance() {
    // local variable increases performance by 25 percent
    // Joshua Bloch "Effective Java, Second Edition", p. 283-284
    
    ThreadSafeDoubleCheckLocking result = instance;
    // Check if singleton instance is initialized. If it is initialized then we can return the instance.
    if (result == null) {
      // It is not initialized but we cannot be sure because some other thread might have initialized it
      // in the meanwhile. So to make sure we need to lock on an object to get mutual exclusion.
      synchronized (ThreadSafeDoubleCheckLocking.class) {
        // Again assign the instance to local variable to check if it was initialized by some other thread
        // while current thread was blocked to enter the locked zone. If it was initialized then we can 
        // return the previously created instance just like the previous null check.
        result = instance;
        if (result == null) {
          // The instance is still not initialized so we can safely (no other thread can enter this zone)
          // create an instance and make it our singleton instance.
          instance = result = new ThreadSafeDoubleCheckLocking();
        }
      }
    }
    return result;
  }
}
```

#### 完全体

在单例实现方式的纠结中，最终出现了一种被广泛接受的形式。下面形式在内部类熟悉的修饰符上存在争议，然而考虑到外部类访问内部类`private`的特殊处理，可以采用`default`。

> http://ifeve.com/initialization-on-demand-holder-idiom/

```java
/**
 * The Initialize-on-demand-holder idiom is a secure way of creating a lazy initialized singleton
 * object in Java.
 * <p>
 * The technique is as lazy as possible and works in all known versions of Java. It takes advantage
 * of language guarantees about class initialization, and will therefore work correctly in all
 * Java-compliant compilers and virtual machines.
 * <p>
 * The inner class is referenced no earlier (and therefore loaded no earlier by the class loader) than
 * the moment that getInstance() is called. Thus, this solution is thread-safe without requiring special
 * language constructs (i.e. volatile or synchronized).
 *
 */
public final class InitializingOnDemandHolderIdiom {

  /**
   * Private constructor.
   */
  private InitializingOnDemandHolderIdiom() {}

  /**
   * @return Singleton instance
   */
  public static InitializingOnDemandHolderIdiom getInstance() {
    return HelperHolder.INSTANCE;
  }

  /**
   * Provides the lazy-loaded Singleton instance.
   */
  private static class HelperHolder {
    private static final InitializingOnDemandHolderIdiom INSTANCE =
        new InitializingOnDemandHolderIdiom();
  }
}
```





