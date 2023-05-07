> https://www.baeldung.com/java-acquire-lock-by-key

## 1. Simple Mutex Lock Example 

```java
public class MyLockClass {

    private static Set<String> usedKeys = ConcurrentHashMap.newKeySet();

    public boolean tryLock(String key) {
        return usedKeys.add(key)
    }

    public void unlock(String key) {
        usedKeys.remove(key);
    }
}
```

```java
String key = "key";

MyLockClass lock = new MyLockClass();

try {
    lock.tryLock(key);
    // 처리해야 할 코드
} catch(Exception e) {

} finally {
    lock.unlock(key);
}
```

<br>

## 2. Simple Mutex Lock Example V2 (not refuse, wait until release the lock)

The application flow will be:
1. thread 1 asks for a lock on a key: it acquires the lock on the key
2. thread 2 asks for a lock on the same key: thread 2 is told to wait
3. thread 1 releases the lock on the key
4. thread 2 auqires the lock on the key and execute its action

```java
public class MyLockClass {

    private static class LockWrapper {
        private final Lock lock = new ReentrantLock();
        private final AtomicInteger numberOfThreadsInQueue = new AtomicInteger(1);

        private LockWrapper addThreadInQueue() {
            numberOfThreadsInQueue.incrementAndGet();
            return this;
        }

        private int removeThreadInQueue() {
            return numberOfThreadsInQueue.decrementAndGet();
        }
    }

    private static ConcurrentHashMap<String, LockWrapper> locks = new ConcurrentHashMap<String, LockWrapper>();

    public void lock(String key) {
        LockWrapper lockWrapper = locks.compute(key, (k, v) -> v == null ? new LockWrapper() : v.addThreadInQueue());
        lockWrapper.lock.lock();
    }

    public void unlock(String key) {
        LockWrapper lockWrapper = locks.get(key);
        lockWrapper.lock.unlock();
        if(lockWrapper.removeThreadFromQueue() == 0) {
            locks.remove(key, lockWrapper);
        }
    }
}
```

```java
String key = "key";
MyLockClass lock = new MyLockClass();
try {
    lock.lock(key);
} finally {
    lock.unlock(key);
}
```

> *" In brief, **a Lock** is an object used for thread synchronization that **allows blocking threads until it can be acquired.** "*

<br>

## 3. Simple Semaphore Example

The application flow will be:
1. thread 1 wants to acquire the lock on the key: it will be allowed to do so
2. thread 2 wants to acquire the lock on the key: it will be allowed to do so
3. thread 3 requests a lock on the same key: it will have to queue until one of the first two threads releases its lock

```java
public class MyLockClass {
    private static final int ALLOWED_THREADS = 2;

    private static ConcurrentHashMap<String, Semaphore> semaphores = new ConcurrentHashMap<String, Semaphore>();

    public void lock(String key) {
        Semaphore semaphore = semaphores.compute(key, (k, v) -> v == null ? new Semaphore(ALLOWED_THREADS) : v);
        semaphore.acquireUninterruptibly(); // Acquires a permit from this semaphore, blocking until one is available.
    }

    public void unlock(String key) {
        Semaphore semaphore = semaphores.get(key);
        semaphore.release();
        if (semaphore.availablePermits() == ALLOWED_THREADS) {
            semaphores.remove(key, semaphore);
        }
    }
}
```

```java
String key = "key";
MyLockClass lock = new MyLockClass();
try {
    lock.lock(key);
} finally {
    lock.unlock(key);
}
```