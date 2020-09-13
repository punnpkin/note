# 1. ReentrantLock

## 使用

```java
class X{
    //定义锁对象
    private final ReentrantLock lock=new ReentrantLock();
    //定义需要保证线程安全的方法
    public void m(){
        //加锁
        lock.lock();
        try{
            //...method body
        }
        //使用finally块来保证释放锁
        finally{
            lock.unlock();
        }
    }
}
```

## **ReentrantLock 与 synchronized 区别**

**synchronized** 是 Java 语言层面提供的语法，所以不需要考虑异常；

ReentrantLock 是 Java 代码实现的锁，所以必须先要获取锁，然后再正确释放锁；

**synchronized** 在获取锁时必须一直等待没有额外的尝试机制；

ReentrantLock 可以尝试获取锁（这一点等下分析源码时会看到），ReentrantLock 支持获取锁时的公平和非公平选择。

## 锁流程

### **lock接口**

```java
public interface Lock {
    void lock();

    void lockInterruptibly() throws InterruptedException;

    boolean tryLock();

    boolean tryLock(long var1, TimeUnit var3) throws InterruptedException;

    void unlock();

    Condition newCondition();
}
```

### **使用**

```java
Lock lock = new ReentrantLock();
lock.lock();
```

### **入口**

入口是一个lock()方法：

```java
public void lock() {
    this.sync.acquire(1);
}
```

### 公平锁实现

#### FairSync

```java

public final void acquire(int arg) {
    if (!this.tryAcquire(arg) && 
        this.acquireQueued(this.addWaiter(Node.EXCLUSIVE), arg)) {
        selfInterrupt();
    }
}

```

在这里，会先 tryAcquire 去尝试获取锁，如果获取成功，那就返回 true ，如果失败就通过 addWaiter 方法，将当前线程封装成 Node 插入到等待队列中。

如果tryAcquire() 方法获取锁成功，那就直接执行线程的任务就可以了，执行完毕释放锁。

如果获取锁失败，就会调用 addWaiter 方法，将当前线程插入到等待队列中，插入的逻辑大概是这样的：

* 将当前线程封装成 Node 节点
* 当前链表中 tail 节点是否为空，如果不为空，则 CAS 操作将当前线程的 node 添加到 AQS 队列
* 如果为空，或者 CAS 操作失败，则调用 enq 方法，再次**自旋插入**

#### tryAcquire

```java
static final class FairSync extends ReentrantLock.Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    FairSync() {
    }

    @ReservedStackAccess
    // 重写了 Sync 的 tryAcquire 方法
    protected final boolean tryAcquire(int acquires) {
        // 获取当前执行的线程
        Thread current = Thread.currentThread();
        // 获取 state 的值
        int c = this.getState();
        // 在无锁状态下
        if (c == 0) {
        // 没有前驱节点且替换 state 的值成功时
            if (!this.hasQueuedPredecessors() && this.compareAndSetState(0, acquires)) {
                // 保存当前获得锁的线程,下次再来时,就不需要尝试竞争锁,直接重入即可
                // 设置独占所有者线程
                this.setExclusiveOwnerThread(current);
                return true;
            }
        } else if (current == this.getExclusiveOwnerThread()) {
            // 如果是同一个线程来获得锁,直接增加重入次数即可
            int nextc = c + acquires;
            // nextc 小于 0 ,抛异常
            if (nextc < 0) {
                throw new Error("Maximum lock count exceeded");
            }

            this.setState(nextc);
            // 获取锁成功
            return true;
        }
		
        // 获取锁失败
        return false;
    }
}
```

#### addWaiter

```java
private Node addWaiter(Node mode) {
    // 生成该线程所对应的 Node 节点
    Node node = new Node(mode);

    Node oldTail;
    do {
        while(true) {
            oldTail = this.tail;
            if (oldTail != null) {
                // 当前节点的pre指向tail
                node.setPrevRelaxed(oldTail);
                break;
            }
			// 初始化同步队列
            this.initializeSyncQueue();
        }
    } while(!this.compareAndSetTail(oldTail, node));
	
    // 双向链表
    oldTail.next = node;
    return node;
}
```

---

通过 addWaiter 将当前线程加入到队列中之后，会走 `acquireQueued(addWaiter(Node.EXCLUSIVE), arg)` 方法

acquireQueued 方法实现的主要逻辑是:

- 获取当前节点的**前驱节点 p**
- 如果节点 p 为 head 节点，说明当前节点为第二个节点，那么它就可以尝试获取锁，调用 tryAcquire 方法尝试进行获取
- 调用 tryAcquire 方法获取锁成功之后，就将 head 指向自己，原来的节点 p 就需要从队列中删除
- 如果获取锁失败，则调用 shouldParkAfterFailedAcquire 或者 parkAndCheckInterrupt 方法来决定后面操作

#### cancelAcquire 

最后，通过 cancelAcquire 方法取消获得锁 看具体的代码实现:

```java
final boolean acquireQueued(Node node, int arg) {
    boolean interrupted = false;

    try {
        while(true) {
            // 获取前置节点
            Node p = node.predecessor();
            // 节点 p 为 head 节点，说明当前节点为第二个节点，调用 tryAcquire 方法尝试获取
            
            if (p == this.head && this.tryAcquire(arg)) {
                // 成功后，删除原来的p
                this.setHead(node);
                p.next = null;
                return interrupted;
            }
			// 失败后 park 挂起
            if (shouldParkAfterFailedAcquire(p, node)) {
                interrupted |= this.parkAndCheckInterrupt();
            }
        }
    } catch (Throwable var5) {
        // 取消锁
        this.cancelAcquire(node);
        if (interrupted) {
            selfInterrupt();
        }

        throw var5;
    }
}
```

#### shouldParkAfterFailedAcquire 

线程获取锁失败之后，会通过调用 shouldParkAfterFailedAcquire 方法，来决定这个线程要不要挂起

shouldParkAfterFailedAcquire 方法实现的主要逻辑:

- 首先判断 pred 的状态是否为 SIGNAL ，如果是，则直接挂起即可
- 如果 pred 的状态大于 0 ，说明该节点被取消了，那么直接从队列中移除即可
- 如果 pred 的状态不是 SIGNAL 也不大于 0 ，进行 CAS 操作修改节点状态为 SIGNAL ，返回 false ，也就是不需要挂起

```java
private static boolean shouldParkAfterFailedAcquire(AbstractQueuedSynchronizer.Node pred, AbstractQueuedSynchronizer.Node node) {
    
    int ws = pred.waitStatus;
    // 状态获取
    if (ws == -1) {
        return true;
    } else {
        如果 pred 的状态大于 0 ，说明该节点被取消了
        if (ws > 0) {
            do {
                node.prev = pred = pred.prev;
            } while(pred.waitStatus > 0);

            pred.next = node;
        } else {
            // cas修改节点状态
            pred.compareAndSetWaitStatus(ws, -1);
        }

        return false;
    }
}
```

---

其中， sync 是 ReentrantLock 的静态内部类，它继承 AQS 来实现**重入锁**的逻辑， Sync 有两个**具体实现类**: `NonfairSync` 和 `FairSync`

#### 非公平锁

NonfairSync

`NonfairSync`是syn的实现类：

```java
static final class NonfairSync extends ReentrantLock.Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    NonfairSync() {
    }
	
    // 重写了 AQS 的 tryAcquire 方法
    
    protected final boolean tryAcquire(int acquires) {
        return this.nonfairTryAcquire(acquires);
    }
}

// FairSync 只是多了一个判断就是，是否有前驱节点
final boolean nonfairTryAcquire(int acquires) {
    // 获取当前线程
    Thread current = Thread.currentThread();
    // 获取状态
    int c = this.getState();
    if (c == 0) {
        // 如果无锁，执行cas
        if (this.compareAndSetState(0, acquires)) {
            // 保存带锁线程，设置可重入
            this.setExclusiveOwnerThread(current);
            return true;
        }
    } // 如果是同一个线程，设置可重入
    else if (current == this.getExclusiveOwnerThread()) {
        // 增加重入次数
        int nextc = c + acquires;
        if (nextc < 0) {
            throw new Error("Maximum lock count exceeded");
        }

        this.setState(nextc);
        return true;
    }
	// 获取失败返回
    return false;
}
```

## 释放锁

```java
public void unlock() {
    sync.release(1);
}
```

```java
public final boolean release(int arg) {
    if (this.tryRelease(arg)) {
        // 释放成功，获取头节点
        AbstractQueuedSynchronizer.Node h = this.head;
        if (h != null && h.waitStatus != 0) {
            // 头节点不为空，状态！=0，唤醒后续节点
            this.unparkSuccessor(h);
        }

        return true;
    } else {
        return false;
    }
}
```

```java
protected final boolean tryRelease(int releases) {
    int c = this.getState() - releases;
    // 判断当前线程是否为获取到锁的线程,如果不是则抛出异常
    // 只有获取到锁的线程才释放锁
    if (Thread.currentThread() != this.getExclusiveOwnerThread()) {
        throw new IllegalMonitorStateException();
    } else {
        boolean free = false;
        // c=0 释放完毕
        if (c == 0) {
            free = true;
            // 线程置为null
            this.setExclusiveOwnerThread((Thread)null);
        }
		// 更新重入次数
        this.setState(c);
        return free;
    }
}
```

```java
// 唤醒
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0) {
        // 如果节点状态小于 0 ,则进行 CAS 操作设置为 0
        node.compareAndSetWaitStatus(ws, 0);
    }
	
    // 获取当前节点的下一个节点 s
    Node s = node.next;
    // 如果 s 为空,则从尾部节点开始,或者s.waitStatus 大于 0 ,说明节点被取消
    // 从尾节点开始,寻找到距离 head 节点最近的一个 waitStatus <= 0 的节点
    if (s == null || s.waitStatus > 0) {
        s = null;

        for(Node p = this.tail; p != node && p != null; p = p.prev) {
            if (p.waitStatus <= 0) {
                s = p;
            }
        }
    }

    if (s != null) {
        // next 节点不为空,直接唤醒即可
        LockSupport.unpark(s.thread);
    }
}
```



