# 消息

## Looper

通过 ThreadLocal 与当前线程绑定，保证一个线程只有一个 Looper 对象

一个 Looper 拥有一个 MessageQueue

loop() 方法不断从 MessageQueue 获取消息，交给其 target 的 dispatchMessage() 处理消息。

### Looper.prepare()

```java
public static final void prepare() {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(true));
}

public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```

如果 Looper 不为 null，抛出异常，否则，创建一个 Looper，放入 ThreadLocal 中。

Looper.prepare() 不能被调用两次，保证一个线程中只有一个 Looper 实例。

```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mRun = true;
    mThread = Thread.currentThread();
}
```

### loop()

通过调用 Looper.myLooper()，从 sThreadLocal.get() 获取 Looper 实例，如果为 null 抛出异常，因此 loop() 必须在 prepare() 之后执行。

获取 Looper 中的 mQueue 消息队列。

进入死循环，如果没有消息，则阻塞。

调用 msg.target.dispatchMessage(msg) 处理消息，target 就是 handler 对象。

释放 msg。

## Handler

### 构造

```java
public Handler() {
    this(null, false);
}
public Handler(Callback callback, boolean async) {
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }

    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

### 发送消息

通过 Looper.myLooper 获取当前线程保存的 Looper 实例，再获取 MessageQueue

发送消息的各种重载方法，最终调用 setMessageAtTime()

```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
       MessageQueue queue = mQueue;
       if (queue == null) {
           RuntimeException e = new RuntimeException(
                   this + " sendMessageAtTime() called with no mQueue");
           Log.w("Looper", e.getMessage(), e);
           return false;
       }
       return enqueueMessage(queue, msg, uptimeMillis);
   }

private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
       msg.target = this;
       if (mAsynchronous) {
           msg.setAsynchronous(true);
       }
       return queue.enqueueMessage(msg, uptimeMillis);
   }
```

将消息的 target 赋值为 this 自身（handler），最终调用 queue.enqueueMessage()，handler最终发出的消息会保存在消息队列中。

### 处理消息

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```

先通过消息的 callback 处理，再判断 handler 的 callback 是否处理消息，最后调用自身 handleMessage 处理

handler.post(Runnable r)  通过 Message.obtain 获取 message 对象，并将 runnable 绑定到 message 的 callback