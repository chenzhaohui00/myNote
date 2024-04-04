## Handler线程间通信的原理

1. 每个Handler都和一个线程的Looper相对应，每个Looper有自己的MessageQueue
2. 只要发送方通过接收方的Handler发送一个消息，就可以把消息入队到接收方线程的Looper的MessageQueue中。
3. 接收方的Looper一旦开始loop，就会不断地从自己的MessageQueue中拿Message。
4. 每个线程都有自己的Looper，这个是存在一个Looper类的静态变量Looper.sThreadLocal中，这个变量的类型就是ThreadLocal，这是一个java类，他是一个数据集合，可以简单认为是一个map，每个ThreadLocal的只能存一种数据，key固定是每个线程，value就是存的值。这样每个线程去get直接就会get到自己的值
5. ThreadLocal实际上的数据结构是object[]，每两个对象是一个slot，每个slot中第一个对象是key，也就是线程，第二个是value，也就是存的值。然后他定义了一系列根据自己的数据结构去增删改查的方法。



## Looper的构造和prepare

### 构造

```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```

可以看到，Looper在new的时候只是创建了MessageQueue和获取了当前Thread对象，并没有把Looper对象存到sThreadLocal中

### ThreadLocal相关

```java
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

```java
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}

static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
```

可以看到这里对sThreadLocal的存取操作。



## Handler

### Handler的构造，和Looper相关的部分

Handler在new的时候可以传一个Looper，如果不传，就会去Looper.myLooper()获取自己的Looper，而如果获取的时候发现还没存过，就会报错说必须先调用Looper.prepare().

另外这里的mCallback也是后面执行的时候用到的。

```java
//没有传looper的构造
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

//传了looper的构造
public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

然后看到，handler直接存了Looper的MessageQueue。

### 发消息

Handler发消息通常有两种，一种是sendMessage，一种是post一个Runnable，分别看下这两个方法：

```java
public final boolean sendMessage(@NonNull Message msg) {
    return sendMessageDelayed(msg, 0);
}

public final boolean post(@NonNull Runnable r) {
   return  sendMessageDelayed(getPostMessage(r), 0);
}

//post调的通过runnable获取msg的方法，其实就是把要执行的方法存为message的callback
private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}
 
//上面两个方法都是调的这个
public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis) {
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}

public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
  MessageQueue queue = mQueue;
  if (queue == null) {
      RuntimeException e = new RuntimeException(
              this + " sendMessageAtTime() called with no mQueue");
      Log.w("Looper", e.getMessage(), e);
      return false;
  }
  return enqueueMessage(queue, msg, uptimeMillis);
}

//最后调了这个
private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
      long uptimeMillis) {
  msg.target = this;
  msg.workSourceUid = ThreadLocalWorkSource.getUid();

  if (mAsynchronous) {
      msg.setAsynchronous(true);
  }
  //最后是调了queue的msg入队方法
  return queue.enqueueMessage(msg, uptimeMillis);
}
```

可以看到，最后这两个方法都是调用了MessageQueue的入队方法，把要入队的消息和执行时间丢进去。



## MessageQueue的入队方法

messageQueue内存储了Message的链表，然后因为message的链表插入都是按照时间增序排列的，所以在入队的时候只要找到第一个时间在要插入的时间以后的消息即可，细节代码如下：

```java
//存message队列的地方，其实完全依赖于message自己的单链表的结构存储的
Message mMessages;

//入队操作
boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }

    synchronized (this) {
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }

        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            //message的链表插入都是按照时间增序排列的，所以只要找到第一个时间在要插入的时间以后的消息即可
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```



## Message

Message中有很多常用的参数比如what、when、arg1、obj等等，就不列出来了，主要列出几个其他的重要的参数：

```java
//缓冲池，避免短时间大量的对象创建和回收，obtain方法和recycle方法使用
private static Message sPool;

//用Handler的post(Runnable runnable)方法传进来的runnable，后面执行的时候用的
/*package*/ Runnable callback;

//执行这个msg回调的Handler对象
/*package*/ Handler target;

//单链表用到的next
/*package*/ Message next;
```



## Looper的Loop

```java
public static void loop() {
  	//获取当前线程的looper对象
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    if (me.mInLoop) {
        Slog.w(TAG, "Loop again would have the queued messages be executed"
                + " before this one completed.");
    }

    me.mInLoop = true;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    // Allow overriding a threshold with a system prop. e.g.
    // adb shell 'setprop log.looper.1000.main.slow 1 && stop && start'
    final int thresholdOverride =
            SystemProperties.getInt("log.looper."
                    + Process.myUid() + "."
                    + Thread.currentThread().getName()
                    + ".slow", 0);

    me.mSlowDeliveryDetected = false;

  	//循环调用loopOnce
    for (;;) {
        if (!loopOnce(me, ident, thresholdOverride)) {
            return;
        }
    }
}

private static boolean loopOnce(final Looper me, final long ident, final int thresholdOverride) {
    //从MessageQueue中拿一条
    Message msg = me.mQueue.next(); // might block
    if (msg == null) {
        // No message indicates that the message queue is quitting.
        return false;
    }
  
    /**
    	省略其他代码
    **/
  
    final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
    final long dispatchEnd;
    Object token = null;
    if (observer != null) {
        token = observer.messageDispatchStarting();
    }
    long origWorkSource = ThreadLocalWorkSource.setUid(msg.workSourceUid);
    try {
        //msg的回调处理
        msg.target.dispatchMessage(msg);
        if (observer != null) {
            observer.messageDispatched(token, msg);
        }
        dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
    } catch (Exception exception) {
        if (observer != null) {
            observer.dispatchingThrewException(token, msg, exception);
        }
        throw exception;
    } finally {
        ThreadLocalWorkSource.restore(origWorkSource);
        if (traceTag != 0) {
            Trace.traceEnd(traceTag);
        }
    }
    /**
    	省略其他代码
    **/
  
  	//消息回收
    msg.recycleUnchecked();

    return true;
}
```

可以看到loop方法就是循环调用loopOnce()，而后者最主要就是干了两件事：

1. me.mQueue.next()：从MessageQueue中拿一条
2. 分发处理：msg.target.dispatchMessage(msg)，msg.target就是Handler对象

接下来看一下这两部分。



## MessageQueue的next()

```java
Message next() {
    // Return here if the message loop has already quit and been disposed.
    // This can happen if the application tries to restart a looper after quit
    // which is not supported.
    final long ptr = mPtr;
    if (ptr == 0) {
        //没有消息，直接返回null
        return null;
    }

    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }

        //epool机制，调用这个方法主线程会释放 CPU 资源进入休眠状态，native层监听到有消息后再叫醒自己
        nativePollOnce(ptr, nextPollTimeoutMillis);

        synchronized (this) {
            // Try to retrieve the next message.  Return if found.
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                //先忽略这部分
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    //发现第一条msg的执行时间还没到，就设置下次要wake up的时间
                    // Next message is not ready.  Set a timeout to wake up when it is ready.
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // Got a message.
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        //移动指向的message链表表头的指针
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    msg.markInUse();
                    //返回这一条
                    return msg;
                }
            } else {
                // No more messages.
                nextPollTimeoutMillis = -1;
            }

            /**
              省略其他代码
            **/
          
        }
    }
}
```

可以看到，这里有一些native的操作，但是核心就是如果没有消息直接返回，有消息就看消息的执行时间到了没有，如果到了就返回，没有到就会block住，并且告诉native层，有消息了叫醒我。



## Handler.dispatchMessage

```java
public void dispatchMessage(@NonNull Message msg) {
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

可以看到这个执行顺序：

1. 先看msg有没有post过来的runnable，如果有就执行它
2. 再看Handler有没有new的时候传过来的callback，如果有就执行它
3. 最后执行继承Handler时重写的handlerMessage方法的代码。