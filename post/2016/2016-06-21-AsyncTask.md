AsyncTask Դ��ѧϰ
====

AsyncTask ��ϵͳ�ṩһ������ִ���첽�����һ�������࣬Ҳ������һֱ�Ӵ���ʹ�ñȽ϶��һ����
ֻ�������˽��˲��ܺúõ�ʹ��������������������������ǿ�������������ġ�


��������������֪ʶ��

* [�̳߳�](http://www.jianshu.com/p/bb1d232aa4df)
* [Future](http://www.cnblogs.com/dolphin0520/p/3949310.html)


# ��һ��  AsyncTask��������첽�߳������߳�ͨ��


����汾��api-20

�鿴Դ�뷢�֣�ʵ�����������Ǿ� *Thread + Handler*,�ǲ���˲��о����Ҳ����һ�������صĶ����ˡ�

```java
    //�ص����߳�
    private Result postResult(Result result) {
        Message message = sHandler.obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
    //ִ�����������̵߳ķ�����һ��update��һ��Result
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


# �ڶ���  AsyncTask���߳�����ε��ȵ�

�ⲿ���� *AsyncTask* �ڲ��̳߳ش����ķ�ʽ,һ�������������̳߳�

* SERIAL_EXECUTOR һ���̵߳��̳߳�
* SERIAL_EXECUTOR �μ��̳߳ز���

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

    //�̳߳�
    public static final Executor THREAD_POOL_EXECUTOR
            = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);

    //ִ�е��̳߳�
    public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

    private static final int MESSAGE_POST_RESULT = 0x1;
    private static final int MESSAGE_POST_PROGRESS = 0x2;

    private static final InternalHandler sHandler = new InternalHandler();
    //Ĭ��ִ�е��̳߳�
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

�����Ķ�һ��Դ��ᷢ����*AsyncTask*���������ʱ��ᴴ����������ֱ���:*mWorker*��*mFuture*,�����ǰ���ᵽ��
Future��������ݣ������׵���Ҫ�Ķ�һ����ƪ���ͣ�����˵�ĺ������

Ȼ�������ڿ�һ���ύ*Task*�ط��Ĵ��룺

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

��ʱ�����ǻᷢ��*AsyncTask*�������ύ������ȫ����������һ����ִ���ˣ������׿��Կ�һ��*SerialExecutor*��ʵ�ַ�
����

ÿ���ύ�����񣬻������������

* ��ִ�еľͰ�������������һ����ջ����,�ȴ�ǰ������ִ������ִ�к����
* û������ִ�У�ֱ��ִ�е�ǰ������

��������ʹ��*AsyncTask*��ִ��һЩ�����ʱ����Ҫ�ر�С���ˣ���Ȼ�и�����ִ��ʱ��ܳ����������û�취ִ���ˡ�
�����������Щ����˳��ִ�п��Ե���һ���������*executeOnExecutor*,����ָ��һ���̳߳�ִ�С�

ʣ�µľ���һЩ�����ķ����ˣ�����ȡ�������½��ȵ�

ע��

* �̳߳��и��ȴ������������Ƶģ������������������û����쳣�����������������ǰ�������ֵ�ǣ�
LinkedBlockingQueue<Runnable>(128)����������汾�в����
* AsyncTask ��Ҫ�����߳��д���������󣬲�Ȼ*Handler*����ϢҲ���㴴�����Ǹ��߳�

�����Ǹ���ϵͳ���� *AsyncTask* ���̳߳ز�����ִ�еĲ���

|ϵͳ�汾|SERIAL_EXECUTOR/THREAD_POOL_EXECUTOR|�̳߳ز���|�ȴ����г���|
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

