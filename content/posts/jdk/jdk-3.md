---
title: ThreadLocal
date: 2024-09-13
categories:
  - Technology
tags:
  - JDK
nolastmod: true
cover: /img/pic-9.jpg
---
`Provide thread-local variables, wish to associate state with a thread(e.g., a user ID or Transaction ID).`

每个Thread拥有一个ThreadLocalMap，实际是一个`Entry[] table`，Entry是一个key-value结构，key是一个弱引用。所以get()或set()时，是通过CurrentThread得到ThreadLocalMap，然后以this为key操作的过程。
```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```
先来看下get():
```java
public T get() {
    Thread t = Thread.currentThread();
    // 拿到对应currentThread的map数组
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    // 初始化map数组
    return setInitialValue();
}
```
进入getEntry():
```java
private Entry getEntry(ThreadLocal<?> key) {
    // threadLocalHashCode很神奇的数字
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        // 因为是线性探测，第一下可能没找到，往后接着找
        return getEntryAfterMiss(key, i, e);
}
```
set()相对复杂一点:
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);
    } else {
        // 初始化map并把第一个value放进去，这时候不会冲突
        createMap(t, value);
    }
}
```
详细看下set(key,value):
```java
private void set(ThreadLocal<?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    // 往后找到第一个为null的位置
    for (Entry e = tab[i];
            e != null;
            e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        // 如果key相同，直接替换
        if (k == key) {
            e.value = value;
            return;
        }
        // 如果这个位置key为null，说明它被GC了，需要做一个清理
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    // re-pack and/or re-size the table
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```
每个ThreadLocal对象都有一个threadLocalHashCode，每次获取都是增加`0x61c88647`，通过它获得的散列分布特别好，是个神奇的数字。这样一来哈希冲突的可能性大大降低，然后这里JDK使用的是线性探测法解决冲突。

接着是为什么要使用弱引用，这是为了有效回收没用的ThreadLocal，因为线程池中线程长时间不会销毁，如果是强引用的话线程的ThreadLocal占的内存就一直不会释放。

还有一个问题：当弱引用被GC后，value和Entry还在会造成内存泄漏，ThreadLocal在设计上考虑到了这种情况，这也是为什么在set过程中扫描到key为null需要去做清理。但在实际使用中，在ThreadLocal使用前后都调用remove清理，同时对异常情况也要在finally中清理。这样可以有效避免内存泄漏，以及业务上的逻辑问题。

另一个问题是常常声明为static：我的理解是ThreadLocal本质是一个key，对于不同的Thread过来会有相应的Map对应，使用的时候要注意remove，static的话避免资源浪费。