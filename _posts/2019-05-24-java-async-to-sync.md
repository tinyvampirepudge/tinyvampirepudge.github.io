---
layout: post
title:  Java中实现异步转同步的几种方式
date:   2019-05-25 01:29:51 +0800
categories: Java
tag: [异步转同步]
---

* content
{:toc}



### Java中实现异步转同步的几种方式

Android常见的异步转同步的方式是通过Callback + Handler的方式来完成，常见的例子是在子线程请求网络，成功后调用Callback，然后通过Handler发送消息给主线程，让子线程更新UI。当然了，实际开发还有好多方式可以实现这种操作。

这里展示Java中的几种异步转同步的方式。

注意：这里只讲实现，不讲原理，具体原理请自行Google。

#### 1、CountDownLatch

CountDownLatch是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程的操作执行完后再执行。

这里有三个子任务Task1、Task2和Task3，我们想在Task2和Task3都执行完了以后，才执行Task1。实现如下：

1、先自定义一个Runnable，run方法中会打印当前线程，在线程执行完毕后会调用CountDownLatch#countDown()方法。

```
    /**
     * 自定义Runnable
     */
    private static class CustomRunnable implements Runnable {

        private CountDownLatch countDownLatch;
        private String name;
        private int delayTime;

        public CustomRunnable(CountDownLatch countDownLatch, String name, int delayTime) {
            this.countDownLatch = countDownLatch;
            this.name = name;
            this.delayTime = delayTime;
        }

        @Override
        public void run() {
            LogUtils.e(TAG, "开始执行" + name);
            ThreadUtils.logCurrThreadName(TAG + " " + name);
            try {
                TimeUnit.SECONDS.sleep(delayTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            LogUtils.e(TAG, name + "执行完毕");
            if (countDownLatch != null) {
                countDownLatch.countDown();
            }
        }
    }
```

2、
①初始化CountDownLatch，这里传入了2，这是因为Task1等待的任务有2个，Task2和Task3.
②创建自定义Runnable Task1，线程内部调用CountDownLatch#await()方法开始等待。
③创建线程池，依次提交任务Task1、Task2、Task3。

```
    public void onBtnJavaCountDownLatchClicked() {
        /**
         * Task2和Task3都执行完了以后，才执行Task1
         */
        countDownLatch = new CountDownLatch(2);

        Runnable Task1 = () -> {
            LogUtils.e(TAG, "开始执行Task1");
            ThreadUtils.logCurrThreadName(TAG + " Task1");
            try {
                // 注意这里是await方法，不是wait方法，不要问我为什么，难受。
                countDownLatch.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            LogUtils.e(TAG, "Task1执行完毕");
        };

        // 创建线程池
        ExecutorService executors = Executors.newFixedThreadPool(3);

        // 执行任务。
        executors.submit(Task1);
        executors.submit(new CustomRunnable(countDownLatch, "Task2", 3));
        executors.submit(new CustomRunnable(countDownLatch, "Task3", 5));

        // 任务完成后关闭线程池
        executors.shutdown();

    }
```

输出结果：

```
05-24 11:14:32.632 2018-2052/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 开始执行Task1
05-24 11:14:32.633 2018-2052/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity Task1: sub Thread,name --> pool-2-thread-1
05-24 11:14:32.634 2018-2053/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 开始执行Task2
05-24 11:14:32.634 2018-2053/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity Task2: sub Thread,name --> pool-2-thread-2
05-24 11:14:32.635 2018-2054/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 开始执行Task3
05-24 11:14:32.635 2018-2054/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity Task3: sub Thread,name --> pool-2-thread-3
05-24 11:14:35.635 2018-2053/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: Task2执行完毕
05-24 11:14:37.636 2018-2054/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: Task3执行完毕
05-24 11:14:37.636 2018-2052/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: Task1执行完毕
```

我们先提交了Task1任务，它先执行，当执行到CountDownLatch#await()方法时开始等待，等待Task2和Task3都执行完毕后，Task1才继续执行，上述log印证了这点。

大家可以想象，如果这里没有CountDownLatch，那么Task1、Task2和Task3这三个任务会各自独立执行，互不影响。

添加了CountDownLatch后，我们做到了让任务Task2和Task3都执行完以后，再继续执行Task1的任务，代码是不是很简单。

#### 2、CyclicBarrier
CyclicBarrier是一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。

①新建CyclicBarrier，它有两个参数，第一个参数表示如果有10个线程到达屏障位置时，就会执行第二个Runnable参数的run方法。

```
        CyclicBarrier cyclicBarrier = new CyclicBarrier(10, () -> {
            LogUtils.e(TAG, "所有任务都执行完毕了");
            ThreadUtils.logCurrThreadName(TAG + " barrierAction");
        });
```

②创建10个任务，每个任务执行完毕后都会调用CyclicBarrier#await()方法。
③将这10个任务都放入线程池中去执行。

```
        List<Runnable> runnables = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            int finalI = i;
            Runnable runnable = () -> {
                LogUtils.e(TAG, "当前是第" + finalI + "个任务，开始执行");
                ThreadUtils.logCurrThreadName(TAG + " 子任务");
                try {
                    TimeUnit.SECONDS.sleep(finalI);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                LogUtils.e(TAG, "当前是第" + finalI + "个任务，执行完毕");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            };
            runnables.add(runnable);
        }

        ExecutorService executorService = Executors.newCachedThreadPool();
        for (Runnable r : runnables) {
            executorService.submit(r);
        }
        executorService.shutdown();
```

输出结果：

```
05-24 11:55:04.272 2018-2190/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第2个任务，开始执行
05-24 11:55:04.272 2018-2190/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 子任务: sub Thread,name --> pool-3-thread-3
05-24 11:55:04.272 2018-2191/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第3个任务，开始执行
05-24 11:55:04.272 2018-2191/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 子任务: sub Thread,name --> pool-3-thread-4
05-24 11:55:04.273 2018-2188/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第0个任务，开始执行
05-24 11:55:04.273 2018-2188/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 子任务: sub Thread,name --> pool-3-thread-1
05-24 11:55:04.273 2018-2188/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第0个任务，执行完毕
05-24 11:55:04.273 2018-2189/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第1个任务，开始执行
05-24 11:55:04.273 2018-2189/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 子任务: sub Thread,name --> pool-3-thread-2
05-24 11:55:04.274 2018-2192/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第4个任务，开始执行
05-24 11:55:04.274 2018-2192/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 子任务: sub Thread,name --> pool-3-thread-5
05-24 11:55:04.275 2018-2193/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第5个任务，开始执行
05-24 11:55:04.275 2018-2193/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 子任务: sub Thread,name --> pool-3-thread-6
05-24 11:55:04.275 2018-2194/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第6个任务，开始执行
05-24 11:55:04.275 2018-2194/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 子任务: sub Thread,name --> pool-3-thread-7
05-24 11:55:04.277 2018-2197/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第9个任务，开始执行
05-24 11:55:04.278 2018-2197/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 子任务: sub Thread,name --> pool-3-thread-10
05-24 11:55:04.278 2018-2195/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第7个任务，开始执行
05-24 11:55:04.278 2018-2195/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 子任务: sub Thread,name --> pool-3-thread-8
05-24 11:55:04.279 2018-2196/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第8个任务，开始执行
05-24 11:55:04.279 2018-2196/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 子任务: sub Thread,name --> pool-3-thread-9
05-24 11:55:05.274 2018-2189/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第1个任务，执行完毕
05-24 11:55:06.272 2018-2190/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第2个任务，执行完毕
05-24 11:55:07.273 2018-2191/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第3个任务，执行完毕
05-24 11:55:08.276 2018-2192/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第4个任务，执行完毕
05-24 11:55:09.277 2018-2193/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第5个任务，执行完毕
05-24 11:55:10.280 2018-2194/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第6个任务，执行完毕
05-24 11:55:11.282 2018-2195/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第7个任务，执行完毕
05-24 11:55:12.282 2018-2196/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第8个任务，执行完毕
05-24 11:55:13.278 2018-2197/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 当前是第9个任务，执行完毕
05-24 11:55:13.278 2018-2197/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 所有任务都执行完毕了
05-24 11:55:13.278 2018-2197/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity barrierAction: sub Thread,name --> pool-3-thread-10
```

从输出结果可以看出，当在10个线程中调用了CyclicBarrier#await()方法后，我们的CyclicBarrier对象初始化时传入的Runnable对象的run方法会被调用。

#### 3、FutureTask
这里提供三种实现方式：

##### ①Callable + Future + ExecutorService

第一步，先自定义一个Callable对象，在call方法中进行计算并将结果返回。

```
    class CustomCallable implements Callable<Integer> {

        @Override
        public Integer call() throws Exception {
            LogUtils.e(TAG, "在子线程进行计算");
            ThreadUtils.logCurrThreadName(TAG + " Task start");
            int sum = 0;
            for (int i = 0; i < 20; i++) {
                Thread.sleep(100);
                sum += i;
                ThreadUtils.logCurrThreadName(TAG + " Task sum:" + sum);
            }
            ThreadUtils.logCurrThreadName(TAG + " Task end");
            return sum;
        }
    }
```

第二步，创建线程池，通过submit方法提交刚才的自定义Callable，并通过Future接受返回结果。

第三步，通过Future#get()方法完成异步转同步操作。

```
    public void onBtnJavaCallableFutureClicked() {
        // Callable + Future + ExecutorService
        ThreadUtils.logCurrThreadName(TAG + "  main start");
        ExecutorService executorService = Executors.newCachedThreadPool();
        CustomCallable task = new CustomCallable();
        Future<Integer> result = executorService.submit(task);
        executorService.shutdown();

        ThreadUtils.logCurrThreadName(TAG + "  main 111");

        try {
            LogUtils.e(TAG, "task执行结果:" + result.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

        for (int i = 0; i < 10; i++) {
            ThreadUtils.logCurrThreadName(TAG + "  main 222:" + i);
        }

        ThreadUtils.logCurrThreadName(TAG + "  main 主线程任务执行完毕");
    }
```

输出结果：

```
05-24 12:06:36.609 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main start: main Thread,name --> main
05-24 12:06:36.613 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 111: main Thread,name --> main
05-24 12:06:36.614 2314-2344/com.tiny.demo.firstlinecode E/FutureTaskActivity: 在子线程进行计算
05-24 12:06:36.614 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task start: sub Thread,name --> pool-2-thread-1
05-24 12:06:36.714 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:0: sub Thread,name --> pool-2-thread-1
05-24 12:06:36.814 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:1: sub Thread,name --> pool-2-thread-1
05-24 12:06:36.915 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:3: sub Thread,name --> pool-2-thread-1
05-24 12:06:37.016 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:6: sub Thread,name --> pool-2-thread-1
05-24 12:06:37.116 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:10: sub Thread,name --> pool-2-thread-1
05-24 12:06:37.217 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:15: sub Thread,name --> pool-2-thread-1
05-24 12:06:37.318 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:21: sub Thread,name --> pool-2-thread-1
05-24 12:06:37.418 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:28: sub Thread,name --> pool-2-thread-1
05-24 12:06:37.519 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:36: sub Thread,name --> pool-2-thread-1
05-24 12:06:37.620 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:45: sub Thread,name --> pool-2-thread-1
05-24 12:06:37.620 2314-2344/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task end: sub Thread,name --> pool-2-thread-1
05-24 12:06:37.622 2314-2314/com.tiny.demo.firstlinecode E/FutureTaskActivity: task执行结果:45
05-24 12:06:37.622 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:0: main Thread,name --> main
05-24 12:06:37.623 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:1: main Thread,name --> main
05-24 12:06:37.623 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:2: main Thread,name --> main
05-24 12:06:37.623 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:3: main Thread,name --> main
05-24 12:06:37.623 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:4: main Thread,name --> main
05-24 12:06:37.623 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:5: main Thread,name --> main
05-24 12:06:37.623 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:6: main Thread,name --> main
05-24 12:06:37.623 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:7: main Thread,name --> main
05-24 12:06:37.623 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:8: main Thread,name --> main
05-24 12:06:37.623 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:9: main Thread,name --> main
05-24 12:06:37.623 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 主线程任务执行完毕: main Thread,name --> main
```

可以看到，在调用了Future#get方法后，主线程一直在等待子线程的执行结果，当子线程的计算执行完毕后，就继续执行主线程的代码。

##### ②Callable + FutureTask + ExecutorService

与上面的demo类似，只不过我们把Future换成FutureTask。

```
    public void onBtnJavaCallableFutureTaskClicked() {
        // Callable + FutureTask + ExecutorService
        ThreadUtils.logCurrThreadName(TAG + "  main start");
        ExecutorService executorService = Executors.newCachedThreadPool();
        CustomCallable task = new CustomCallable();
        FutureTask<Integer> futureTask = new FutureTask<>(task);
        executorService.submit(futureTask);
        executorService.shutdown();

        ThreadUtils.logCurrThreadName(TAG + "  main 111");

        try {
            LogUtils.e(TAG, "task执行结果:" + futureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

        for (int i = 0; i < 10; i++) {
            ThreadUtils.logCurrThreadName(TAG + "  main 222:" + i);
        }

        ThreadUtils.logCurrThreadName(TAG + "  main 主线程任务执行完毕");
    }
```

输出结果：

```
05-24 12:10:08.028 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main start: main Thread,name --> main
05-24 12:10:08.029 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 111: main Thread,name --> main
05-24 12:10:08.030 2314-2362/com.tiny.demo.firstlinecode E/FutureTaskActivity: 在子线程进行计算
05-24 12:10:08.030 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task start: sub Thread,name --> pool-3-thread-1
05-24 12:10:08.131 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:0: sub Thread,name --> pool-3-thread-1
05-24 12:10:08.232 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:1: sub Thread,name --> pool-3-thread-1
05-24 12:10:08.333 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:3: sub Thread,name --> pool-3-thread-1
05-24 12:10:08.434 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:6: sub Thread,name --> pool-3-thread-1
05-24 12:10:08.534 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:10: sub Thread,name --> pool-3-thread-1
05-24 12:10:08.635 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:15: sub Thread,name --> pool-3-thread-1
05-24 12:10:08.735 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:21: sub Thread,name --> pool-3-thread-1
05-24 12:10:08.836 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:28: sub Thread,name --> pool-3-thread-1
05-24 12:10:08.937 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:36: sub Thread,name --> pool-3-thread-1
05-24 12:10:09.038 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:45: sub Thread,name --> pool-3-thread-1
05-24 12:10:09.038 2314-2362/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task end: sub Thread,name --> pool-3-thread-1
05-24 12:10:09.038 2314-2314/com.tiny.demo.firstlinecode E/FutureTaskActivity: task执行结果:45
05-24 12:10:09.038 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:0: main Thread,name --> main
05-24 12:10:09.038 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:1: main Thread,name --> main
05-24 12:10:09.038 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:2: main Thread,name --> main
05-24 12:10:09.039 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:3: main Thread,name --> main
05-24 12:10:09.039 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:4: main Thread,name --> main
05-24 12:10:09.039 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:5: main Thread,name --> main
05-24 12:10:09.039 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:6: main Thread,name --> main
05-24 12:10:09.039 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:7: main Thread,name --> main
05-24 12:10:09.039 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:8: main Thread,name --> main
05-24 12:10:09.039 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:9: main Thread,name --> main
05-24 12:10:09.039 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 主线程任务执行完毕: main Thread,name --> main
```

可以看到，输出结果跟上个例子一致。

##### ③Callable + FutureTask + Thread

这里我们将上面例子中的线程池换成Thread，其他不变。

```
    public void onViewClicked() {
        // Callable + FutureTask + Thread
        ThreadUtils.logCurrThreadName(TAG + "  main start");
        CustomCallable task = new CustomCallable();
        FutureTask<Integer> futureTask = new FutureTask<>(task);
        Thread thread = new Thread(futureTask);
        thread.start();

        ThreadUtils.logCurrThreadName(TAG + "  main 111");

        try {
            LogUtils.e(TAG, "task执行结果:" + futureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

        for (int i = 0; i < 10; i++) {
            ThreadUtils.logCurrThreadName(TAG + "  main 222:" + i);
        }

        ThreadUtils.logCurrThreadName(TAG + "  main 主线程任务执行完毕");
    }
```

输出结果：

```
05-24 12:12:08.493 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main start: main Thread,name --> main
05-24 12:12:08.495 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 111: main Thread,name --> main
05-24 12:12:08.496 2314-2377/com.tiny.demo.firstlinecode E/FutureTaskActivity: 在子线程进行计算
05-24 12:12:08.496 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task start: sub Thread,name --> Thread-215
05-24 12:12:08.596 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:0: sub Thread,name --> Thread-215
05-24 12:12:08.698 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:1: sub Thread,name --> Thread-215
05-24 12:12:08.798 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:3: sub Thread,name --> Thread-215
05-24 12:12:08.899 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:6: sub Thread,name --> Thread-215
05-24 12:12:09.000 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:10: sub Thread,name --> Thread-215
05-24 12:12:09.100 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:15: sub Thread,name --> Thread-215
05-24 12:12:09.202 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:21: sub Thread,name --> Thread-215
05-24 12:12:09.303 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:28: sub Thread,name --> Thread-215
05-24 12:12:09.404 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:36: sub Thread,name --> Thread-215
05-24 12:12:09.506 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task sum:45: sub Thread,name --> Thread-215
05-24 12:12:09.507 2314-2377/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity Task end: sub Thread,name --> Thread-215
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/FutureTaskActivity: task执行结果:45
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:0: main Thread,name --> main
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:1: main Thread,name --> main
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:2: main Thread,name --> main
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:3: main Thread,name --> main
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:4: main Thread,name --> main
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:5: main Thread,name --> main
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:6: main Thread,name --> main
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:7: main Thread,name --> main
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:8: main Thread,name --> main
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 222:9: main Thread,name --> main
05-24 12:12:09.507 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: FutureTaskActivity  main 主线程任务执行完毕: main Thread,name --> main
```

输出效果与上面一致。

#### 4、rxjava

RxJava在处理异步任务方面特别强悍，这里通过两个例子说明。

##### map——让异步任务顺序执行

首先，我们通过一个例子让几个异步任务依次顺序执行，下一个任务的启动依赖于上一个任务的结果。

这里我们举一个注册完成后直接登录的例子。

大体流程如下：

```
注册请求 --> 注册响应 --> 登录请求 --> 登录响应
```

###### ①先创建四个Bean，用于模拟数据转换。分别是注册请求Bean，注册请求响应Bean，登录请求Bean，登录请求相应Bean。

下面这个是注册请求Bean的部分代码，相应的getter/setter方法和toString方法已省略。

```
public class RegisterReqBean {
    private String name;
    private String phone;
    private String pwd;

    public RegisterReqBean(String name, String phone, String pwd) {
        this.name = name;
        this.phone = phone;
        this.pwd = pwd;
    }
    ...
}
```

注册请求相应Bean：

```
public class RegisterRespBean {
    private String name;
    private String pwd;
    private String msg;
    ...
}
```

登录请求Bean：

```
public class LoginReqBean {
    private String name;
    private String pwd;
    private String msg;
    ...
}
```

登录请求响应Bean：

```
public class LoginRespBean {
    private String name;
    private String pwd;
    private String msg;
    ...
}
```

###### ②创建对应的Observable对象，准备好数据转换的素材。

```
        RegisterReqBean registerReqBean = new RegisterReqBean("猫了个咪", "13333333333", "123456");
        Observable registerRequest = Observable.create(new ObservableOnSubscribe<RegisterReqBean>() {
            @Override
            public void subscribe(ObservableEmitter<RegisterReqBean> emitter) throws Exception {
                LogUtils.e(TAG, "注册请求 registerReqBean:" + registerReqBean);
                ThreadUtils.logCurrThreadName(TAG + " 注册请求成功");
                emitter.onNext(registerReqBean);
                LogUtils.e(TAG, "发送注册请求");
                emitter.onComplete();
                LogUtils.e(TAG, "发送注册请求完成");
            }
        });

        // 模拟注册响应。将RegisterReqBean转化为RegisterRespBean。
        Function mockRegisterResp = new Function<RegisterReqBean, RegisterRespBean>() {
            @Override
            public RegisterRespBean apply(RegisterReqBean registerReqBean) throws Exception {
                RegisterRespBean registerRespBean = new RegisterRespBean(registerReqBean.getName(),
                        registerReqBean.getPwd(), "恭喜您注册成功");
                LogUtils.e(TAG, "注册响应 registerRespBean:" + registerRespBean);
                TimeUnit.SECONDS.sleep(5);// 模拟网络延迟
                ThreadUtils.logCurrThreadName(TAG + " 注册响应成功");
                return registerRespBean;
            }
        };

        // 模拟登录请求。将RegisterRespBean转化为LoginReqBean。
        Function mockLoginReq = new Function<RegisterRespBean, LoginReqBean>() {
            @Override
            public LoginReqBean apply(RegisterRespBean registerRespBean) throws Exception {
                LoginReqBean loginReqBean = new LoginReqBean(registerRespBean.getName() + " 我要登陆", registerRespBean.getPwd());
                LogUtils.e(TAG, "登录请求 loginReqBean:" + loginReqBean);
                ThreadUtils.logCurrThreadName(TAG + " 登录请求成功");
                return loginReqBean;
            }
        };

        // 模拟登录响应。将LoginReqBean转化为LoginRespBean。
        Function mockLoginResp = new Function<LoginReqBean, LoginRespBean>() {
            @Override
            public LoginRespBean apply(LoginReqBean loginReqBean) throws Exception {
                LoginRespBean loginRespBean = new LoginRespBean(loginReqBean.getName(),
                        loginReqBean.getPwd(), "恭喜您登录成功");
                LogUtils.e(TAG, "登录响应 loginRespBean:" + loginRespBean);
                TimeUnit.SECONDS.sleep(5);// 模拟网络延迟
                ThreadUtils.logCurrThreadName(TAG + " 登录响应成功");
                return loginRespBean;
            }

        };
        
        // 模拟接受登录响应结果。
        Observer<LoginRespBean> resultObserver = new Observer<LoginRespBean>() {
            @Override
            public void onSubscribe(Disposable d) {
                LogUtils.e(TAG, "onSubscribe");
            }

            @Override
            public void onNext(LoginRespBean loginRespBean) {
                LogUtils.e(TAG, "onNext 登录成功，可以更新UI了。 loginRespBean:" + loginRespBean);
                ThreadUtils.logCurrThreadName(TAG + " ");
            }

            @Override
            public void onError(Throwable e) {
                LogUtils.e(TAG, "onError e:" + e.getMessage());
            }

            @Override
            public void onComplete() {
                LogUtils.e(TAG, "onComplete");
            }
        };
```


###### ③使用map操作符进行转化

```
        registerRequest.subscribeOn(Schedulers.io())
                .observeOn(Schedulers.newThread())
                .map(mockRegisterResp)
                .observeOn(Schedulers.io())
                .map(mockLoginReq)
                .observeOn(Schedulers.newThread())
                .map(mockLoginResp)
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(resultObserver);
```

最后的输出结果：

```
05-24 13:22:17.823 2314-2314/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: onSubscribe
05-24 13:22:17.826 2314-2619/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 注册请求 registerReqBean:RegisterReqBean{name='猫了个咪', phone='13333333333', pwd='123456'}
05-24 13:22:17.826 2314-2619/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 注册请求成功: sub Thread,name --> RxCachedThreadScheduler-1
05-24 13:22:17.827 2314-2619/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 发送注册请求
05-24 13:22:17.827 2314-2619/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 发送注册请求完成
05-24 13:22:17.830 2314-2620/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 注册响应 registerRespBean:RegisterRespBean{name='猫了个咪', pwd='123456', msg='恭喜您注册成功'}
05-24 13:22:22.833 2314-2620/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 注册响应成功: sub Thread,name --> RxNewThreadScheduler-1
05-24 13:22:22.835 2314-2622/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 登录请求 loginReqBean:LoginReqBean{name='猫了个咪 我要登陆', pwd='123456'}
05-24 13:22:22.835 2314-2622/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 登录请求成功: sub Thread,name --> RxCachedThreadScheduler-2
05-24 13:22:22.836 2314-2623/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: 登录响应 loginRespBean:LoginRespBean{name='猫了个咪 我要登陆', pwd='123456', msg='恭喜您登录成功'}
05-24 13:22:27.838 2314-2623/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity 登录响应成功: sub Thread,name --> RxNewThreadScheduler-2
05-24 13:22:27.838 2314-2314/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: onNext 登录成功，可以更新UI了。 loginRespBean:LoginRespBean{name='猫了个咪 我要登陆', pwd='123456', msg='恭喜您登录成功'}
05-24 13:22:27.838 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity : main Thread,name --> main
05-24 13:22:27.838 2314-2314/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: onComplete
```

##### merge——多个异步任务执行完毕后在执行目标任务

等待多个任务完成后再执行目标任务，不保证顺序。

```
    public void onBtnJavaRxjavaMergeClicked() {
        /**
         * 等待多个任务完成后再执行，不保证顺序
         */
        Observable o1 = Observable.create((ObservableOnSubscribe<String>) e -> {
            ThreadUtils.logCurrThreadName(TAG + " Observable1");
            TimeUnit.SECONDS.sleep(5);
            e.onNext("第一个Observable");
            e.onComplete();
        }).subscribeOn(Schedulers.newThread());

        Observable o2 = Observable.create((ObservableOnSubscribe<String>) e -> {
            ThreadUtils.logCurrThreadName(TAG + " Observable2");
            TimeUnit.SECONDS.sleep(3);
            e.onNext("第二个Observable");
            e.onComplete();
        }).subscribeOn(Schedulers.newThread());

        Observable o3 = Observable.create((ObservableOnSubscribe<String>) e -> {
            ThreadUtils.logCurrThreadName(TAG + " Observable3");
            TimeUnit.SECONDS.sleep(1);
            e.onNext("第三个Observable");
            e.onComplete();
        }).subscribeOn(Schedulers.newThread());

        /**
         * 三个任务都发送了onComplete事件后，订阅者的onComplete方法才会调用。
         */
        Observable.merge(o1, o2, o3)
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        LogUtils.e(TAG, "onSubscribe");
                    }

                    @Override
                    public void onNext(Object o) {
                        LogUtils.e(TAG, "onNext: " + o);
                        ThreadUtils.logCurrThreadName(TAG + " onNext");
                    }

                    @Override
                    public void onError(Throwable e) {
                        LogUtils.e(TAG, "onError");
                    }

                    @Override
                    public void onComplete() {
                        LogUtils.e(TAG, "onComplete");
                    }
                });
    }
```

输出结果：

```
05-24 13:26:59.176 2314-2314/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: onSubscribe
05-24 13:26:59.179 2314-2644/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity Observable1: sub Thread,name --> RxNewThreadScheduler-3
05-24 13:26:59.181 2314-2645/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity Observable2: sub Thread,name --> RxNewThreadScheduler-4
05-24 13:26:59.181 2314-2646/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity Observable3: sub Thread,name --> RxNewThreadScheduler-5
05-24 13:27:00.183 2314-2314/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: onNext: 第三个Observable
05-24 13:27:00.183 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity onNext: main Thread,name --> main
05-24 13:27:02.183 2314-2314/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: onNext: 第二个Observable
05-24 13:27:02.184 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity onNext: main Thread,name --> main
05-24 13:27:04.180 2314-2314/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: onNext: 第一个Observable
05-24 13:27:04.181 2314-2314/com.tiny.demo.firstlinecode E/tiny_module: AsyncToSyncActivity onNext: main Thread,name --> main
05-24 13:27:04.181 2314-2314/com.tiny.demo.firstlinecode E/AsyncToSyncActivity: onComplete
```

当三个Observable都发送了onComplete事件后，Observer的onComplete方法才会调用。

#### 参考

[CountDownLatch](http://www.importnew.com/15731.html)

[FutureTask](http://www.importnew.com/27305.html)

[深入浅出java CyclicBarrier](https://www.jianshu.com/p/424374d71b67)

