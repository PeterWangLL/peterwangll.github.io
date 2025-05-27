---
title: AbstractQueuedSynchronizer
date: 2024-09-11
categories:
  - Technology
tags:
  - JDK
nolastmod: true
cover: /img/pic-7.jpg
---
`Provides a framework for implementing blocking locks and related synchronizers (semaphores, events, etc) that rely on first-in-first-out (FIFO) wait queues.`

该抽象队列同步器通过依赖FIFO的队列同步器来实现阻塞式的锁机制。
```java
static final class Node{
    volatile int waitStatus;
    volatile Node prev;
    volatile Node next;
    volatile Thread thread;
    Node nextWaiter;
}
```
实际上维护了一个共享变量`state`和一个双向链表，链表节点除了prev, next, 还有`waitStatus`:

`Status field, taking on only the values:SIGNAL, The successor of this node is (or will soon be) blocked (via park), so the current node must unpark its successor when it releases or cancels.`

这里有两个点：SIGNAL和successor(继任者)

`Thread`和`nextWaiter`后面再说。

独占模式下:`acquire`和`release`两个方法:
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```
`tryAcquire`交给子类实现，从命名中就可以看出是尝试去获取锁，从ReentrantLock的实现中：
```java
final Thread current = Thread.currentThread();
int c = getState();
if (c == 0) {
    if (compareAndSetState(0, acquires)) {
        setExclusiveOwnerThread(current);
        return true;
    }
}
```
可以看出尝试获取锁的逻辑就是通过改变state的值来实现的，同时将`exclusiveOwnerThread`设置为当前线程。这里的`setExclusiveOwnerThread`是AQS的父类方法。

回到`acquire`方法，当获取锁失败，则进行入队操作:
```java
// Creates and enqueues node for current thread and given mode.
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```
`enq`之前，先尝试将节点添加到末尾，如果失败（并发），则一直重试直到成功。

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
node就是刚刚入队的节点，当prev是head节点时才去尝试获取锁，如果失败`shouldParkAfterFailedAcquire`去设置前一个节点waitStatus为SIGNAL，然后`parkAndCheckInterrupt`就是去挂起当前线程。

有意思的是，`shouldParkAfterFailedAcquire`当`prev.waitStatus>0`时，会先从后往前删去这些大于0的节点，等下一轮循环再去设置SIGNAL值。

---

`release`比较简单，因为是独占式的，尝试释放锁，unpark下一个节点。
```java
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```
这里如果next为空则从后往前遍历找到下一个唤醒的线程，由于并发问题，`addWaiter()`入队操作和`cancelAcquire()`取消排队操作都会造成next链的不一致，而prev链是强一致的，所以这时从后往前找是最安全的。

waitStatus有5个状态，这里只讲2个:
1. SIGNAL(-1): to avoid races, acquire methods must first indicate they need a signal, 从`unparkSuccessor`中可以看到当`waitStatus<0`时去unpark
2. CANCELLED(1): due to timeout or interrupt,so `shouldParkAfterFailedAcquire`要从后向前删除`waitStatus>0`的节点