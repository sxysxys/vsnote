## 锁

### 1. Synchronized和Lock区别

>  Synchroized demo

``` java
public class Ticket2 {
    private int number = 80;
    
    public synchronized void sale() {
        if (number > 0) {
            System.out.println(Thread.currentThread().getName() + "买了一张票，还剩" + --number + "张");
        }
    }
}
```

>  Lock demo

``` java
public class Ticket {
    private int number = 80;

    private Lock lock = new ReentrantLock();

    /**
     *
     */
    public void sale() {
        lock.lock();
        try {
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "买了一张票，还剩" + --number + "张");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

> Synchronized和Lock的区别

1. synchronized是一个java内置的语法，Lock是一个类
2. synchronized是自动的，Lock要手动释放锁
3. synchronized是可重入锁，并且不可中断，非公平；Lock是可重入锁，可以判断锁的状态，非公平（能够自己设置）；
4. synchronized只要被一个线程获得了锁，就算阻塞了，别的线程也会傻傻等下去；Lock不会，它有判断机制。
5. synchronized适合锁少量的代码块；Lock适合锁大量的代码块。



## 线程间通信

### wait()和notify()

> wait

- 作用是等待，并且在等待的时候将线程持有的此对象的锁释放，让其他的线程也能持有这个对象。

- 只有别的对象在此对象的方法中执行notify或者notifyAll的时候唤醒这个线程，并且在

> notify()



### 生产者消费者问题

> synchronize demo

``` java

```









