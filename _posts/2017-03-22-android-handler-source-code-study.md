---
layout: post
title:  Handler源码解读——handler使用时的注意事项 
date:   2017-03-22 21:10:25 +0800
categories: Android
tag: [Handler]
---

* content
{:toc}



### Handler源码解读——handler使用时的注意事项        
工作中经常会遇到从子线程发送消息给主线程，让主线程更新UI的操作，常见的有handler.sendMessage(Message)，和handler.post(runnable)和handler.postDelayed(runnable, milliseconds);一直在使用这些方法，却不知道他们的原理，今天就来解释一下他们的原理。
        
先来一个常见的handler.sendMessage(message)的例子吧。代码如下：

定义一个handler的子类，并实现它的handleMessage()方法：这里的msg.what的值是你自己定义的，就不多解释了。

```
private Handler handler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        switch (msg.what) {
            case 1000:
                int arg = msg.arg1;
                // TODO: 17/3/22
                break;
            case 1001:

                break;
            default:
                break;
        }
    }
};
```
接下来是在子线程中发送消息的部分：
   

```
//handler.sendMessage(Msg)
new Thread(new Runnable() {
    @Override
    public void run() {
        Message msg = Message.obtain();
        msg.what = 1000;
        msg.arg1 = 10;
        handler.sendMessage(msg);
    }
}).start();

```
可以看到，我在一个新开的子线程中使用刚才定义的handler发送了一个Message对象，这个Message对象可以携带数据，这里简单起见，我只携带了一个int类型的值10.

至于为什么在子线程中使用handler发送的Message,可以在UI线程中执行了呢？这是因为在Android的主线程中有一个Loop循环器，它一直在轮询这个Loop循环器所属的MessageQueue消息队列中的消息，如果MessageQueue里面有Message，就调用这个Message.target.dispatchMessage(Message)方法，这里其实是调用了handler对象的dispatchMessage()方法，这个方法里面最终（满足一定条件）会调用handler的handleMessage(Message)方法，也就是在主线程中调用了我们刚才定义的handler的handleMessage(Message)方法。

好吧，说了这么多，先来个精简版的简图，接下来再用源码解释原理：

![这里写图片描述](http://img.blog.csdn.net/20170322205815139?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


ps: 上图所涉及的源码，可在Android Studio中可直接查看

#### 1、ActivityThread类的main方法中，最后调用了Loop.loop()方法，开启了主线程。

```
    public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

#### 2、Loop.loop()方法中：

源代码注解：在这个线程中一直调用MessageQuene队列，确保loop被quit之前一直调用。

```
    /**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            final long end;
            try {
                msg.target.dispatchMessage(msg);
                end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (slowDispatchThresholdMs > 0) {
                final long time = end - start;
                if (time > slowDispatchThresholdMs) {
                    Slog.w(TAG, "Dispatch took " + time + "ms on "
                            + Thread.currentThread().getName() + ", h=" +
                            msg.target + " cb=" + msg.callback + " msg=" + msg.what);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```
①获取当前线程的loop对象。

②获取loop中的MessageQueue。

③使用for( ; ; )的开启一个死循环，开始轮询MessageQueue中的消息。

④如果消息不为空，则调用Message对象的target的dispatchMessage(msg)方法

#### 3、msg.target.dispatchMessage(msg)
    Message类中：
```
    /*package*/ Handler target;
```
①target对象为Handler类型，表示发送这个消息的handler对象

②可以通过obtain的一系列重载方法给message设置target，通过无参数的obtain方法获取到的message的handler为null;

③target对象有set和get方法

```
public void setTarget(Handler target) {
    this.target = target;
}

/**
 * Retrieve the a {@link android.os.Handler Handler} implementation that
 * will receive this message. The object must implement
 * {@link android.os.Handler#handleMessage(android.os.Message)
 * Handler.handleMessage()}. Each Handler has its own name-space for
 * message codes, so you do not need to
 * worry about yours conflicting with other handlers.
 */
public Handler getTarget() {
    return target;
}

```
④至于我们这个handler发送的message对象中的target是如何与handler勾搭上的，留待后边说明。

#### 4、Handler.dispatchMessage(message)方法：

```
/**
 * Handle system messages here.
 */
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
这段代码表明，dispatchMessage方法中可能会调用handleMessage(msg)方法，不过需要满足几个条件。好吧，这个问题留待后边解决。我们现在已经简单的看完了handler在子线程发送消息，最终在主线程进行处理的流程。接下来我们需要解决过程中遇到的几个困惑。

困惑一：第3步时，handler发送的message对象中的target是如何与这个handler勾搭上的。

困惑二：第4步时，在dispatchMessage方法中，执行handleMessage(msg)方法的条件如何解释。

解答一：handler发送的message的target对象是如何与这个handler勾搭上的。
①我们得从handler发送消息说起，先来看我们在子线程中发送消息的代码：来找找handler对象是何时设置给message.target的。

```
        new Thread(new Runnable() {
            @Override
            public void run() {
                Message msg = Message.obtain();
                msg.what = 1000;
                msg.arg1 = 10;
                handler.sendMessage(msg);
            }
        }).start();
```
②点进Message.obtain()方法：

```
    /**
     * Return a new Message instance from the global pool. Allows us to
     * avoid allocating new objects in many cases.
     */
    public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }
```
里面的sPool是Message类型，是为了Message的复用，这里没有设置target的数据，再看最下边的new Message();

```
/** Constructor (but the preferred way to get a Message is to call {@link #obtain() Message.obtain()}).
*/
public Message() {
}
```
好吧，这里是个空实现，啥也没有。

③既然在Message里面没有找到，我们就在handler.sendMessage(msg)里面找找，点进去：

```
public final boolean sendMessage(Message msg)
{
	return sendMessageDelayed(msg, 0);
}
```
没有，继续找：

```
public final boolean sendMessageDelayed(Message msg, long delayMillis)
{
	if (delayMillis < 0) {
		delayMillis = 0;
	 }
	return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```
继续：

```
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
```
这里拿到了当前的消息队列，不过依然没有我们要找的，继续：

```
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```
好了，找到了。enqueueMessage()方法的第一步就给msg.target设置了值，就是当前的handler对象，也就是发送消息的那个handler。这个方法的最后一句将这个msg添加进当前的MessageQueue，这样，进入队列的message就携带上Handler类型的target对象了。

到现在为止，我们已经说明，在Loop.loop()方法里的那句msg.target.dispatchMessage(msg)，调用的就是当前hander对象的dispatchMessage(msg)方法了。

解答二：在dispatchMessage方法中，执行handleMessage(msg)方法的条件是怎么回事
①先上dispatchMessage(msg)源码：

```
    /**
     * Handle system messages here.
     */
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

具体流程如图所示：
![这里写图片描述](http://img.blog.csdn.net/20170322210623154?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

疑惑又来了，msg.callback是什么鬼？handleCallback(msg)方法是干嘛的？mCallback是什么鬼？它后边的mCallback.handleMessage(msg)方法又是干嘛的？

②msg.callback和handleCallback(msg)方法；

进入Message类中，发现callback成员变量为Runnable类型，Runnable接口自不必多说，它里面就一个无参无返回值的run方法；

在Message中是搜索callback，发现可以给它赋值的位置只有静态方法obtain()的两个重载方法：

```
	public static Message obtain(Message orig) {
        Message m = obtain();
        m.what = orig.what;
        m.arg1 = orig.arg1;
        m.arg2 = orig.arg2;
        m.obj = orig.obj;
        m.replyTo = orig.replyTo;
        m.sendingUid = orig.sendingUid;
        if (orig.data != null) {
            m.data = new Bundle(orig.data);
        }
        m.target = orig.target;
        m.callback = orig.callback;

        return m;
    }
```

```
public static Message obtain(Handler h, Runnable callback) {
    Message m = obtain();
    m.target = h;
    m.callback = callback;

    return m;
}

```

再看一眼刚才的handleCallback(msg)方法：它就是调用message.callback.run方法。

```
private static void handleCallback(Message message) {
    message.callback.run();
}

```

也就是说，只有当我们直接通过Message.obtain(Message)和Message.obtain(Handler,Runnable)方法来获取Message对象的时候（或者设置message.callback的值不为空），才有可能让msg.callback不为空，才会调用callback.run方法，此时就绝对不会调用我们的handleMessage(msg)方法。

③接下来看mCallback和mCallback.handleMessage(msg)是何方神圣；
mCallback是Handler里面定义的一个Callback接口类型，

```
/**
 * Callback interface you can use when instantiating a Handler to avoid
 * having to implement your own subclass of Handler.
 *
 * @param msg A {@link android.os.Message Message} object
 * @return True if no further handling is desired
 */
public interface Callback {
	public boolean handleMessage(Message msg);
}
```

上方注释的意思是：当你实例化一个Handler的时候，可以通过这个可以接口避免自己写一个Handler的实现类。

这句话是什么意思呢？且往下看。

在Handler类里面找给mCallback赋值的地方，找到三处，都是Handler的带参构造方法：

```
    public Handler(Callback callback) {
        this(callback, false);
    }
```

```
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

```
public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}

```

看到这儿，Handler里面这个接口的注释应该很好理解了。如果你使用上边这两个Handler的构造方法来实例化一个Handler对象，那么你需要传入一个Callback接口类型的参数，在callback的handleMessage(msg)方法里面，你就可以直接写主线程处理Message的逻辑了；此时我们就不需要再写Handler的实现类，重写handler的handleMessage(msg)方法了；另外，这个Callback接口里的handleMessage(msg)方法有返回值，如果你返回为true，就不会执行handler.handleMessage(msg)方法，如果返回为false，依旧会执行handler.handleMessage(msg)方法。这就给我们使用handler提供了多种选择。

这里插一句嘴，上方的两个Handler的构造方法中，都有一个boolean类型的async参数，同学们可能不知道怎么调用，没关系，看看Handler()这个空参构造是怎么写的：

```
public Handler() {
	this(null, false);
}
```
这个this(null,false)调用的就是Handler(Callback callback,boolean async)这个方法，也就是说，默认的我们传入的async这个参数为false就可以。至于这个参数何时为true，何时为false，这个问题留待后续研究，这次不做讲解。

好，原理和流程解释完了，来对关键流程做下总结。

![这里写图片描述](http://img.blog.csdn.net/20170322210906095?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

handler.dispatchMessage(msg)里面的处理，前面已经解释过了，这里不再重复。

#### 使用总结：

①在使用handler发送消息的时候，要记住，自己实现的handleMessage(msg)方法不一定会执行。

②如果我们发送的Message的callback参数(Runnable类型)不为空，那么就不会执行handler的handleMessage(msg)方法，只会执行message.callback的run方法。这种情况主要发生在我们使用handler.post(runnable)和handler.postDelayed(runnable, milliseconds)方法发送消息的时候。另外，在Message对象的callback的参数不为空的情况下，如果我们在创建Handler对象的时候既传入了Callback接口参数，也重写了里面的handleMessage(msg)方法，那么此时依旧只会执行message.callback的run方法。

③在我们发送的Message的callback参数(Runnable类型)为空的情况下（默认为空），在使用Handler的带参构造方法初始化Handler的时候，如果传入了Callback类型的参数，并且那么这个接口里面的方法肯定会被调用；如果同时我们也重写了该handler对象的handleMessage(msg)方法，那么handler对象的handleMessage(msg)方法不一定执行，取决于Callback接口中handleMessage(msg)方法的返回值，如果为true，那么handler对象的handleMessage(msg)方法不会执行，为false时，handler对象的handleMessage(msg)方法会在后边执行。
        
关于handler的总结，希望对你有用。