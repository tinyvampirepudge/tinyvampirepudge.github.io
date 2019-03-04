---
layout: post
title:  Handler切换线程原理解析
date:   2018-04-21 11:23:29 +0800
categories: Handler
tag: [Handler原理]
---

* content
{:toc}



### Handler切换线程原理解析
写在前面：本文的目的是想将Handler、Looper和Thread之间绑定的原理讲明白，如果没讲明白，也希望能给关于Handler的学习留个印象。

Android中的多线程间交互离不开Handler，开发中最常见的操作是在子线程中执行耗时操作，在主线程中更新UI，这其中就涉及到了Handler的线程切换操作。

提到Handler消息机制，就不得不提它的几个组成元素：Handler、Looper、MessageQueue、Message。

Handler的主要作用是发送和处理消息；MessageQueue称作消息队列，采用单链表的数据结构存储Message；Looper是一个循环，会在一个无限循环中不断从MessageQueue中获取Message，如果有Message，就交给对应的Handler去处理；Message是传递的消息，它的target参数持有是发送它的Handler对象。

接下来从开发中常见的场景逐步说明上述几个类是如何关联起来的。

先从一个在子线程中创建Handler对象开始。

```javascript
	@OnClick(R.id.btn_test1)
    public void onBtnTest1Clicked() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                Handler handler = new Handler();
            }
        }).start();
```

上面这几行代码开启了一个子线程并执行，在子线程中创建了一个Handler，不出意外的话这行代码会报错，错误如下：
![子线程中创建Handler时最长见的错误](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/dc36d86884a42097bbaaeb2df608d28c.png)

这个错误的含义是说在线程调用Looper.prepare( )之前不能创建Handler。
&ensp;&ensp;&ensp;&ensp;这是为什么呢？我们追下Handler的源码，代码如下，

```stylus
	public Handler() {
        this(null, false);
    }
```
```stylus
	public Handler(Callback callback, boolean async) {
        ...

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
&ensp;&ensp;&ensp;&ensp;从上述代码可以看出，Handler初始化时调用Looper.myLooper()方法获取一个Looper对象给自己，如果mLooper为null就报刚才出现的错误，因此可以得出结论，Handler在创建之前必须跟一个Looper绑定。另外，Handler中的mQueue就是关联的Looper中的MessageQueue对象。
&ensp;&ensp;&ensp;&ensp;接下来我们看下Looper的实现，先看下Looper类的注释，注释如下，

```stylus
/**
  * Class used to run a message loop for a thread.  Threads by default do
  * not have a message loop associated with them; to create one, call
  * {@link #prepare} in the thread that is to run the loop, and then
  * {@link #loop} to have it process messages until the loop is stopped.
  *
  * <p>Most interaction with a message loop is through the
  * {@link Handler} class.
  *
  * <p>This is a typical example of the implementation of a Looper thread,
  * using the separation of {@link #prepare} and {@link #loop} to create an
  * initial Handler to communicate with the Looper.
  *
  * <pre>
  *  class LooperThread extends Thread {
  *      public Handler mHandler;
  *
  *      public void run() {
  *          Looper.prepare();
  *
  *          mHandler = new Handler() {
  *              public void handleMessage(Message msg) {
  *                  // process incoming messages here
  *              }
  *          };
  *
  *          Looper.loop();
  *      }
  *  }</pre>
  */
```

&ensp;&ensp;&ensp;&ensp;大意是说，Looper通常用来为线程创建一个消息循环器。线程默认是没有关联消息循环器的，在线程中调用Looper.prepare()方法可以将线程与Looper关联起来，然后调用Looper.loop()方法可以让线程处理消息，直到Looper停止。与消息循环的大多数交互都是通过Handler实现。

&ensp;&ensp;&ensp;&ensp;注释中还提供了Looper和Handler的典型实现，先调用prepare()，然后创建一个与Looper交互的Handler对象，最后调用loop()方法。
&ensp;&ensp;&ensp;&ensp;按照提示，我对刚才报错的代码进行修改，错误就会消失，代码如下。

```stylus
	@OnClick(R.id.btn_test1)
    public void onBtnTest1Clicked() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                Looper.prepare();
                Handler handler = new Handler();
                Looper.loop();
            }
        }).start();
    }enter code here
```

### Looper.prepare( )
&ensp;&ensp;&ensp;&ensp;接下来我们看下Looper.prepare()和Looper.loop()方法的源码，看下他们都做了什么。先看Looper.prepare()。

```stylus
	public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```

&ensp;&ensp;&ensp;&ensp;prepare()方法会调用prepare(true)，prepare(boolean quitAllowed)方法也很简单，先判断当前线程的Looper是否为空，如果不为空就会抛出"Only one Looper may be created per thread"异常，说明一个线程只能关联一个Looper。接下来会新建一个Looper对象，并设置给当前Thread对象。

&ensp;&ensp;&ensp;&ensp;这里的解释有些笼统，我逐步说明下。先看下sThreadLocal这个成员变量，它是一个ThreadLocal<Looper>类型的对象，它的主要作用是将Thread和Looper关联起来。具体如何关联的，主要是ThreadLocal的set()和get()方法，我们还是从 sThreadLocal.set(new Looper(quitAllowed))这行看起，先看new Looper(quitAllowed)构造方法，

```stylus
	private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```

&ensp;&ensp;&ensp;&ensp;这里会给新创建的Looper对象中的mQueue赋值为新的MessageQueue对象，给mThread赋值为当前Thread对象，这样这个Looper就持有了当前Thread对象。接下来看ThreadLocal中的set()方法；

```stylus
	public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

&ensp;&ensp;&ensp;&ensp;这里先获取当前Thread对象，然后调用getMap方法，获取Thread中ThreadLocal.ThreadLocalMap类型的成员变量threadLocals；接着对Thread中的threadLocals进行非空判断：如果Thread的threadLocals变量为空，就调用createMap( )方法，

```stylus
	void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

&ensp;&ensp;&ensp;&ensp;新建一个ThreadLocalMap对象并赋值给当前Thread的threadLocals变量；如果Thread的threadLocals变量不为空，就调用ThreadLocal.ThreadLocalMap#set(ThreadLocal key, Object value)方法，这个方法会将以Thread对象为key，以Looper对象为value设置给threadLocals变量，这样就把Thread和Looper的对应关系存储进ThreadLocal.ThreadLocalMap对象了，也就是Thread.threadLocals变量中。具体代码的细节读者可自行查看，这里暂不深究。

&ensp;&ensp;&ensp;&ensp;接下来再看下ThreadLocal#get( )方法实现，
		
```
	public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

&ensp;&ensp;&ensp;&ensp;跟set()方法一样，还是获取当前Thread对象，接着根据获取当前线程中的threadLocals，如果threadLocals不为空，并且threadLocals中存储的Looper对象不为空，就会返回threadLocals中存储的Looper对象，也就是与Thread相关联的Looper对象；接下来还会调用setInitialValue()方法，将setInitialValue()方法的返回值作为get()方法的返回值。

```stylus
	private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
```

&ensp;&ensp;&ensp;&ensp;我们看下setInitialValue()的实现，这里给Looper的初始值为null，然后返回将null设置给Thread的threadLocals对象，最后将null返回。

&ensp;&ensp;&ensp;&ensp;回顾一下ThreadLocal#get()方法，如果当前Thread中的threadLocals变量中关联了Looper对象，就返回；如果没有关联就返回null。

&ensp;&ensp;&ensp;&ensp;看完了ThreadLocal的get()和set()方法，我们回顾下，Looper持有ThreadLocal对象，ThreadLocal中的get()和set()方法会依据当前Thread中持有的ThreadLocal.ThreadLocalMap类型的对象threadLocals，将Thread和Looper关联起来。这样，在Looper.prepare()方法中我们就可以将Looper和当前Thread关联起来。
	
### Looper.loop( )

&ensp;&ensp;&ensp;&ensp;接下来我们看下loop()方法的作用，

```stylus
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

        ...

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            ...
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
            ...

            msg.recycleUnchecked();
        }
    }
```

&ensp;&ensp;&ensp;&ensp;loop()方法的源码如上所示，先获取当前线程关联的Looper，如果Looper为空，就抛出"No Looper; Looper.prepare() wasn't called on this thread."异常。

&ensp;&ensp;&ensp;&ensp;接着获取Looper持有的MessageQueue对象queue，然后开启一个无限循环，不停的获取queue中的Message对象msg，如果msg为null，就跳出无限循环，结束对queue的轮询；如果获取到的msg不为空，就会调用msg.target.dispatchMessage(msg)方法，实质上调用的是Handler#dispatchMessage(msg)方法，将消息交给Message发送时绑定的Handler对象去处理了。需要注意的是这里的Handler#dispatchMessage(msg)方法是在创建Handler时的Looper中执行的，这样就成功的将代码切换到指定的逻辑中去执行了。

### Handler#sendMessage(msg)

&ensp;&ensp;&ensp;&ensp;前面讲了在创建Handler的过程中，Handler是如何与Thread、Looper绑定的，以及Looper是如何接收并处理消息的，这里讲下Handler是如何发送消息的。这里以Handler#sendMessage(msg)为例进行说明：
		

```stylus
	public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }
	
	public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
	
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

&ensp;&ensp;&ensp;&ensp;Handler中消息发送一般是通过post和send系列方法来实现，而post系列方法最终也是通过send方法来实现的，上面通过sendMessage(msg)方法展示了send方法的典型调用过程。如上所示，最终都会调用enqueueMessage方法，将Message插入到Handler的queue中，也就是Looper的queue中去了。这样跟Handler关联的Looper就收到了消息，在Looper.loop()方法的循环中就会将这个Message拿出来，然后调用发送消息的Handler处理了。我们可以在A线程正常创建handlerA，然后在B线程中利用handlerA发送消息，此时消息就发送给与handlerA关联的Looper了，而这个Looper唯一关联的线程就是A，这样我们的消息就会在A线程中执行了。

### Handler#dispatchMessage(msg)

&ensp;&ensp;&ensp;&ensp;接下来我们看下Handler是如何处理Message的。源码如下：

```stylus
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

&ensp;&ensp;&ensp;&ensp;如上所示，Handler处理消息的过程如下：

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;第一步，检查Message的callback对象是否为null，如果不为null，就调用handleCallback(msg)方法，Message的callback对象就是一个Runnable对象，实际上就是Handler#post系列方法中传递的Runnable参数。

```stylus
	private static void handleCallback(Message message) {
        message.callback.run();
    }
```

&ensp;&ensp;&ensp;&ensp;如果Message的callback对象为null，就向下执行。

&ensp;&ensp;&ensp;&ensp;第二步，如果msg.callback为null，会检查Handler的mCallback参数是否为null，不为null就调用mCallback.handleMessage(msg)方法来处理消息，如果mCallback.handleMessage(msg)的返回值为true，就结束执行，否则就继续向下执行。

&ensp;&ensp;&ensp;&ensp;Callback是个接口，定义如下：

```stylus
	/**
     * Callback interface you can use when instantiating a Handler to avoid
     * having to implement your own subclass of Handler.
     */
    public interface Callback {
        /**
         * @param msg A {@link android.os.Message Message} object
         * @return True if no further handling is desired
         */
        public boolean handleMessage(Message msg);
    }
```

&ensp;&ensp;&ensp;&ensp;查看源码可以发现，mCallback是在Handler(Callback callback, boolean async)这个构造方法里面进行赋值的，只有Handler(Callback callback)这个构造方法是传递了Callback对象过来的，所以我们可以采用Handler handler = new Handler(callback)这种方式来创建handle对象.这种做法的意义是什么呢？Callback源码里面的注释已经做了说明：可以用来创建一个Handler的实例但并不需要派生Handler的子类。在日常开发中，创建Handler最常见的方式就是派生一个Handler的子类并重写其handleMessage方法来处理具体的消息，而Callback给我们提供了另外一种使用Handler的方式，当我们不想派生子类时，就可以通过Callback来实现。

&ensp;&ensp;&ensp;&ensp;第三步，继续向下执行，调用handleMessage(msg)方法。
		
&ensp;&ensp;&ensp;&ensp;这里dispatchMessage(msg)的调用可以用一张图表示：

![Handler#dispatchMessage(msg)调用示意图](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/9a1b41fc4ab229ef764de0751f643dbe.png)
	
### Handler、Looper、MessageQueue、Thread的对应关系

&ensp;&ensp;&ensp;&ensp;关于Handler、Looper、MessageQueue、Thread的关系，下面这张图应该可以说明问题。
![Handler、Looper、MessageQueue、Thread的关系](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Handler%E3%80%81Looper%E3%80%81Thread%E3%80%81MessageQueue%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.png)
&ensp;&ensp;&ensp;&ensp;另外，再附图一张，解释下Handler、Looper、Thread中相互持有的引用：
![相互持有的引用](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/Handler%E3%80%81Looper%E3%80%81Thread%E5%85%B3%E8%81%94%E6%96%87%E5%AD%97%E8%AF%B4%E6%98%8E.png)
&ensp;&ensp;&ensp;&ensp;这里关于Handler的分析就告一段落，需要说明的是，本文中没有对ThreadLocal中具体关联Looper和Thread的代码逻辑深追到底，这里留待读者自己阅读理解。

参考：Android开发艺术探索。

