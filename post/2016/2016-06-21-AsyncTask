AsyncTask 源码学习
====

AsyncTask 是系统提供一种用来执行异步任务的一个工具类，也是我们一直接触和使用比较多的一个。
只有我们了解了才能好好的使用它，遇到问题才能马上明白是可能是哪里引起的。


首先有两个基础知识：

* [线程池](http://www.jianshu.com/p/bb1d232aa4df)
* [Future](http://www.cnblogs.com/dolphin0520/p/3949310.html)


# 第一点  AsyncTask如何做到异步线程与主线程通信


代码版本：api-20

查看源码发现，实际上他里面是就 *Thread + Handler*,是不是瞬间感觉这个也不是一个很神秘的东西了。

```java
    //回调主线程
    private Result postResult(Result result) {
        Message message = sHandler.obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
    //执行了两个主线程的方法：一个update和一个Result
    private static class InternalHandler extends Handler {
        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult result = (AsyncTaskResult) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }
        
    @SuppressWarnings({"RawUseOfParameterizedType"})
    private static class AsyncTaskResult<Data> {
        final AsyncTask mTask;
        final Data[] mData;

        AsyncTaskResult(AsyncTask task, Data... data) {
            mTask = task;
            mData = data;
        }
    }
```


# 第二点  AsyncTask的线程是如何调度的

这部分是 *AsyncTask* 内部线程池创建的方式,一共创建了两个线程池

* SERIAL_EXECUTOR 一个线程的线程池
* SERIAL_EXECUTOR 参见线程池参数

```java
    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    private static final int CORE_POOL_SIZE = CPU_COUNT + 1;
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    private static final int KEEP_ALIVE = 1;

    private static final ThreadFactory sThreadFactory = new ThreadFactory() {
        private final AtomicInteger mCount = new AtomicInteger(1);

        public Thread newThread(Runnable r) {
            return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
        }
    };

    private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128);

    //线程池
    public static final Executor THREAD_POOL_EXECUTOR
            = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);

    //执行的线程池
    public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

    private static final int MESSAGE_POST_RESULT = 0x1;
    private static final int MESSAGE_POST_PROGRESS = 0x2;

    private static final InternalHandler sHandler = new InternalHandler();
    //默认执行的线程池
    private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
    
    private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```

我们阅读一下源码会发现在*AsyncTask*创建对象的时候会创建两个对象分别是:*mWorker*和*mFuture*,这就是前面提到的
Future里面的内容，不明白的需要阅读一下那篇博客，里面说的很清楚。

然后我们在看一下提交*Task*地方的代码：

```java
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }


    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute();

        mWorker.mParams = params;
        exec.execute(mFuture);

        return this;
    }
```

这时候我们会发现*AsyncTask*把我们提交的任务全部都穿起来一个个执行了，不明白可以看一下*SerialExecutor*的实现方
法。

每次提交的任务，会有下面情况：

* 有执行的就把这个任务添加在一个堆栈里面,等待前面任务执行完再执行后面的
* 没有任务执行，直接执行当前的任务

所有正常使用*AsyncTask*来执行一些任务的时候需要特别小心了，不然有个任务执行时间很长后面的任务都没办法执行了。
如果不想让这些人物顺序执行可以调用一下这个方法*executeOnExecutor*,可以指定一个线程池执行。

剩下的就是一些辅助的方法了，例如取消、更新进度等

注：

* 线程池有个等待队列数量限制的，如果大于这个数量调用会有异常产生，软件崩溃，当前队列最大值是：
LinkedBlockingQueue<Runnable>(128)，这个各个版本有差异的
* AsyncTask 需要在主线程中创建这个对象，不然*Handler*的消息也在你创建的那个线程

下面是各个系统下面 *AsyncTask* 的线程池参数和执行的差异

|系统版本|SERIAL_EXECUTOR/THREAD_POOL_EXECUTOR|线程池参数|等待队列长度|
|-:|-:|-:|-:|
|2.2|THREAD_POOL_EXECUTOR|5-128-10|10|
|2.3|THREAD_POOL_EXECUTOR|5-128-1|10|
|3.2|THREAD_POOL_EXECUTOR|5-128-1|10|
|4.0|SERIAL_EXECUTOR|5-128-1|10|
|4.1|SERIAL_EXECUTOR|5-128-1|10|
|4.2|SERIAL_EXECUTOR|5-128-1|10|
|4.3|SERIAL_EXECUTOR|5-128-1|10|
|4.4|SERIAL_EXECUTOR|cpu+-cpux2+1-1|128|
|5.0|SERIAL_EXECUTOR|cpu+-cpux2+1-1|128|
|6.0|SERIAL_EXECUTOR|cpu+-cpux2+1-1|128|
|N|SERIAL_EXECUTOR|cpu+-cpux2+1-1|128|

