---
title: ThreadPoolExecutor
date: 2024-09-11
categories:
  - Technology
tags:
  - JDK
nolastmod: true
cover: /img/pic-8.jpg
draft: true
---
### runState&workerCount
`private final AtomicInteger ctl`: the main pool control state, 高3位用来表示线程池的runState，底29位用来表示线程数workCount。这种思想让我联想到了SDS中header的sdshdr5结构，flags低3位保存type，高5位在`type=sdshdr5`时表示len。
### execute()
```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        // try to start a new thread with the given command as its first task
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        // if a task can be successfully queued, then we still need to double-check whether we should have added a thread
        int recheck = ctl.get();
        // if neccessary roll back the enqueuing if stopped
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // or start a new thread if there are none
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // if we cannot queue task, then we try to add a new thread. If it fails, we know we are shut down or saturated and so reject the task.
    else if (!addWorker(command, false))
        reject(command);
}
```
可以看出`execute()`方法会根据线程池现有的状态来决定`addWorker()`or`reject()`

这涉及到线程池的几个关键参数，corePoolSize，maximumPoolSize，workQueue，剩下还有threadFactory，rejectedExecutionHandler，keepAliveTime。
### addWorker()
```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
                firstTask == null &&
                ! workQueue.isEmpty()))
            return false;

        // 自旋workerCount+1
        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        // creates with given first task and thread from ThreadFactory
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    // HashSet<Worker>
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```
`addWorker`主要做了以下几件事：
1. CAS workerCount+1
2. w = new Worker(firstTask);// 创建线程，在构造方法中由threadFactory创建
3. workers.add(w);// 获取mainLock添加worker到HashSet
4. t.start();// 开启线程

### Worker
Class Worker mainly maintains interrupt state for threads running tasks.extends AQS(实现了一个不可重入锁). This protects against interrupts that are intended to wake up a worker thread waiting for a task from instead interrupting a task being run. 
```java
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable
{
    final Thread thread;
    Runnable firstTask;
    volatile long completedTasks;
    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this);
    }
    public void run() {
        runWorker(this);
    }
}
```

由于`implements Runnable`，当上面第四步`t.start()`开启线程后，去执行`runWorker()`

### runWorker()
```java
// Repeatedly gets tasks from queue and executes them
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                    (Thread.interrupted() &&
                    runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```
主要是getTask()和task.run()，Repeatedly gets tasks from queue and executes them.

Before running any task, the lock is acquired to prevent other pool interrupts while the task is executing.

另外提供`beforeExecute()`和`afterExecute()`两个钩子方法。

### 用到的锁
两个地方用到了锁，对works(HashSet<Worker>)的操作都需要获取mainLock(ReentrantLock),因为HashSet是线程不安全的，至于为什么不用并发安全的Set，是为了避免中断风暴的风险，shutdown()会调用interruptIdleWorkers()，不用mainLock会导致对正在中断中的中断，发起中断。具体可以看<https://www.cnblogs.com/thisiswhy/p/15493027.html>.\
还有是在线程处理任务时，需要获取锁，这是通过继承AQS实现的不可重入锁，为了防止被中断。