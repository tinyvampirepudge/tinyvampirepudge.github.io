---
layout: post
title:  基本的线程机制—Java编程思想
date:   2019-01-21 16:20:34 +0800
categories: Java
tag: [Thread, Think In Java]
---

* content
{:toc}



### 基本的线程机制—Java编程思想
并发编程使我们可以将程序分为多个分离的、独立运行的任务。通过使用多线程机制，这些独立人物（也被称为子任务）中的每一个都将由执行线程来驱动。

一个线程就是在进程中的一个单一的顺序控制流，因此，单个进程可以拥有多个并发执行的任务。

在使用线程时，CPU将轮流给每个任务分配其占用时间。

线程的一大好处是可以使你从这个层次抽身出来，即diamante不必知道它是运行在具有一个还是多个CPU的机器上。

#### 21.2.1 定义任务
线程可以驱动任务，这里我们实现了Runnable接口的方式来定义任务。
要实现线程行为，你必须显式的将一个任务附着到线程上。

##### Thread.yield()
这个方法是对`线程调度器`的一种`建议`，它在声明：“我已经执行完声明周期中最重要的部分了，此刻正是切换给其他任务执行一段时间的大好时机”。

线程调度器：Java线程机制的一部分，可以将CPU从一个线程转移到另一个线程。

#### 21.2.2 Thread类
将Runnable对象转变为工作任务的传统方式是把它提交给一个Thread构造器。

调用Thread对象的start()方法为该线程执行必须的初始化操作，然后调用Runnable的run()方法，以便在这个新线程中启动该任务。

任何线程都可以启动另一个线程。

线程调度机制是非确定性的。

在使用普通对象时，这对于垃圾回收来说是一场对它的引用，但是在使用Thread时，情况就不同了。每个Thread都“注册”了自己，因此确实有一个对它的引用，而且在它的任务退出其run()并死亡之前，垃圾回收器无法清除它。

一个线程会创建一个单独的执行线程，在对start()的调用完成之后，它仍旧会继续存在。

#### 21.2.3 使用Executor
Java SE5的java.util.concurrent包中的执行器（Executor）将为你管理Thread对象，这样可以简化并发编程。

Executor允许你管理异步任务的执行，而无须显式地管理线程的生命周期。Executor在Java SE5/6中是启动任务的优选方法。

对ExecutorService#shotdown()方法的调用可以防止新任务被提交给这个Executor，当前任务将继续执行在shotdown()被调用之前提交的所有任务。这个程序将在Executor中的所有任务完成之后尽快退出。

##### FixedThreadPool
FixedThreadPool使用了有限的线程集来执行所提交的任务。有了FixedThreadPool，你就可以一次性预先执行代价高昂的县城分配，因此也就可以限制线程的数量了。这可以节省时间，因为你不用为每个任务都固定地付出创建线程的开销。

注意，在任何线程池中，现有线程在可能的情况下，都会被自动复用。

##### CachedThreadPool
CachedThreadPool在程序执行过程中通常会创建与所需数量相同的线程，然后在它回收旧线程时停止创建新线程，因此它是合理的Executor的首选。只有当这种方式会引发问题时，你才需要切换到FixedThreadPool。

##### SingleThreadExecutor
SingleThreadExecutor就像是线程数量为1的FixedThreadPool。这对于你希望在另一个线程中连续运行的任何事物（长期存活的任务）来说，都是很有用的。

如果想SingleThreadExecutor提交了多个任务，那么这些任务将排队，每个任务都会在下一个任务开始之前运行结束，所有任务都将使用相同的线程。

SingleThreadExecutor会序列化所有提交给它的任务，并会维护它自己（隐藏）的悬挂任务队列。

SingleThreadExecutor就可以确保任何时刻在任何线程中都只有唯一的任务在运行。

#### 21.2.4 从任务中产生返回值——Callable
Runnable是执行工作的独立任务，但是它不反悔任何职。如果你希望任务在完成时能够返回一个值，那么可以实现Callable接口而不是Runnable接口。在Java SE5中引入的Callable事一个具有参数类型的泛型，它的类型参数表示的是从方法call()(而不是run())中返回的值，并且必须使用ExecutorService.submit()方法调用它。

submit()方法会产生Future对象，它用Callable返回结果的特定类型进行了参数化。你可以用isDone()方法来查询Future是否已经完成。当任务完成时，它具有一个结果，你可以调用get()方法来获取该结果。你也可以不用isDone()进行检查就直接调用get()，在这种情况下，get()将阻塞，直至结果准备就绪。你还可以在试图调用get()来获取结果之前，先调用具有超时的get()，或者调用isDone()来查看任务是否完成。

#### 21.2.5 休眠——Thread.sleep()
影响任务行为的一种简单方法时调用sleep()，这将使任务中止执行给定的时间。

对sleep()的调用可以跑出InterruptedException异常，并且它可以在run()中被捕获。因为异常不能跨线程传播回main()，所以你必须在本地处理所有在任务内部产生的异常。

#### 21.2.6 优先级
线程的优先级将该线程的重要性传递给了调度器。
尽管CPU处理现有线程集的顺序是不确定的，但是调度器将倾向于最高的线程先执行。然而，这并不是意味着优先权较低的线程将得不到执行（也就是说，优先权不会导致死锁）。优先级较低的线程仅仅是执行的频率较低。

绝大多数时间里，线程都应该以默认的优先级运行。视图操纵线程优先级通常是一种错误。

你可以时候用getPriority()来读取现有线程的优先级，并且在任何时刻都可以通过setPriority()来修改它。

注意，优先级是在run()的开头部分设定的，在构造器中设置它们不会有任何好处，因为Executor在此刻还没有开始执行任务。

JDK有10个优先级，但它与多数操作系统都不能映射的很好。唯一可移植的方法时当调整优先级的时候，只使用MAX_PRIORITY、NORM_PRIORITY和MIN_PRIORITY三种级别。

##### volatile
变量使用volatile修饰，以努力确保不进行任何编译器优化。

##### Thread.toString()
Thread.toString()方法会打印线程的名称、线程的优先级以及线程所属的“线程组”。你可以通过构造器来自己设置这个名称。

##### Thread.currentThread()
你可以在一个任务的内部，通过调用Thread.currentThread()来获得对该任务的Thread对象的引用。


#### 21.2.7 让步——Thread.yield()
如果知道run()方法中的工作做的差不多了，可以让别的线程使用CPU了，这时就可以使用yield()方法给出一个暗示（不过这只是一个暗示，没有任何机制保证它将会被采纳）。

当调用yield()时，你也是在建议具有`相同优先级`的其他线程可以运行。

大体上，对于任何重要的控制或在调整应用时，都不能依赖于yield()。实际上，yield()经常被误用。

#### 21.2.8 后台线程——daemon
所谓后台（daemon）线程，是指在程序运行的时候在后台提供一种通信服务的线程，并且这种线程并不属于程序中不可或缺的部分。

当所有的非后台线程结束时，程序也就终止了，同时会杀死进程中的所有后台线程。反过来说，只要有任何非后台线程还在运行，程序就不会终止。

必须在线程启动之前调用setDaemon()方法，才能把它设置为后台线程。

每个静态的ExecutorService创建方法都被重载为接受一个ThreadFactory对象，而这个对对象将被用来创建新的线程。
通过编写定制的ThreadFactory可以定制由Executor创建的线程的属性（后台、优先级、名称）。

可以通过调用isDaemon()方法来确定线程是否是一个后台线程。如果是一个后台线程，那么它创建的任何线程都将被自动设置成后台线程。

##### 后台线程在不执行finally语句的情况下，就会终止其run()方法。

```
public class DaemonsDontRunFinally {

    public static void main(String[] args) {
        Thread t = new Thread(new ADaemon());
        t.setDaemon(true);
        t.start();
    }

}

class ADaemon implements Runnable {

    @Override
    public void run() {
        try {
            System.out.println("Starting ADaemon");
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            System.out.println("Exiting via InterruptedException");
        } finally {
            System.out.println("This should always run?");
        }
    }

}
```

当你运行这个程序时，你将看到finally子句就不会执行，但是如果你注释掉对setDaemon()的调用，就会看到finally子句将会执行。

这种行为是正确的，即便你基于前面对finally给出的承诺，并不希望出现这种行为，但情况仍将如此。当最后一个费后台线程终止时，后台现场会“突然”终止。因此一旦main()退出，JVM就会立即关闭所有的后台进程，而不会有任何你希望出现的确认形式。因此你不能以优雅的方式来关闭后台线程，所以它们几乎不是一种好的思想。

非后台的Executor通常是一种更好的方式，因为Executor控制的所有任务可以同时被关闭。在这种情况下，关闭将以有序的方式执行。

#### 21.2.9 编码的变体
1、继承Thread，重写run方法，在构造方法中调用start()方法启动线程。

通过调用适当的Thread构造器为Thread对象赋予具体的名称，这个名称可以通过使用getName()从toString()中获得。

2、自管理的Runnable。实现Runnable接口，实现run方法，同时初始化一个Thread，在构造方法中调用Thread#start()方法。

注意，start()是在构造器中调用的。你应该意识到，在构造器中启动线程可能会有问题，因为另一个任务可能会在构造器结束之前开始执行，这意味着该任务能够访问处于不稳定状态的对象。这就是优选Executor而不是显式地创建Thread对象的另一个原因。

3、通过内部类来讲线程代码隐藏在类中。
①定义一个成员内部类，继承Thread类，重写run方法，并在内部类的构造方法中执行start()方法。
②匿名内部类继承Thread类，实现同上。
③定义一个成员内部类，实现Runnable接口，重写run方法，并在内部类的构造方法中执行start()方法。
④匿名内部类集成Runnable接口，实现同上。

4、在独立的方法中执行定义任务和启动线程的方法。这里使用的是Thread的匿名内部类。

#### 21.2.10 术语
在Java中，你可以选择如何实现并发编程。

要执行的任务和驱动它的线程之间有一个差异，这个差异在Java类库中尤为明显，因为你对Thread类没有任何控制权（这种隔离在使用执行器时更加明显，因为执行器将替你处理线程的创建和管理）。

你创建任务，并通过某种方式将一个线程附着到任务上，以使得这个线程可以驱动任务。

在Java中，Thread类自身不执行任何操作，它只是驱动赋予它的任务，但是线程研究中总是不变的使用“线程执行这项或那项动作”这样的语言。

Java的线程机制基于来自C的地基的p线程方式，这是一种你必须深入研究，并且需要完全理解其所有事物的所有细节的方式。

#### 21.2.11 加入一个线程——join,interrupt
一个线程可以在其他线程之上调用join()方法，其效果是其他线程等待一段时间，直到这个线程结束后，其他线程才会继续执行。

线程被挂起时，t.isAlive()返回为假。

也可以在调用join()时带上一个超市参数（单位客户以是毫秒，或者毫秒和纳秒），这样如果目标线程在这段时间到期时还没有结束的话，join()方法总能返回。

对join()方法的调用可以被中断，做法是在调用线程上调用interrupt()方法，这时需要用到try-catch子句。

可以用isInterrupted()方法返回线程是否被中断。当线程调用interrupt()方法时，将给该线程设定一个标志，表明该线程已经被中断。然而，异常被捕获时将清理这个标志，所以在catch子句中，在异常被捕获的时候这个标志总是为假。

#### 21.2.13 线程组
线程组持有一个线程集合。

#### 21.2.14 捕获线程异常
由于线程的本质特性，使得你不能捕获从线程中逃逸的异常。一旦有异常逃出任务的run()方法，它就会向外传播到控制台，除非你采取特殊的步骤捕获这种错误的异常。

在run方法中抛出异常，然后我们将执行线程的代码放在try-catch语句块中，你会发现try-catch没有生效。

为了解决这个问题，我们要修改Executor产生线程的方式。Thread.UncaughtExceptionHandler是Java SE5中的新接口，它允许你在每个Thread上都附着一个异常处理器。Thread.UncaughtExceptionHandler.uncaughtException()会在线程因未捕获的异常而临近死亡时被调用。

我们创建一个ThreadFactory，它将在每个新创建的Thread对象上附着一个Thread.UncaughtExceptionHandler。我们将这个工厂传递给Executor即可。

如果你需要在代码中处处使用相同的异常处理器，那么可以在Thread类中设置一个静态与，并将这个处理器设置为默认的未捕获异常处理器。

```
Thread.setDefaultUncaughtExceptionHandler(new MyUncaughtExceptionHandler());
```

这个处理器只有在不存在线程专有的未捕获异常处理器的情况下才会被调用。

系统检查专有版本，如果没有发现，则检查线程组是否有其转悠的uncaughtException()方法，如果也没有，再调用defaultUncaughtExceptionHandler。

