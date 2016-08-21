获取数据缓存检查
====

在我们的业务中，很多情况下会使用到缓存技术，使用RxJava我们将很好的实现这种方式。
本次关键操作符 *concat*:

在Android mvp官方例子里面有这么一段代码，用于获取数据从本地缓存或网络，

*concat* 只要是第一个条件满足之后，后面的就不会在执行，所有这样我们可以很好的控
制多级缓存获取数据。

```java
            // Query the local storage if available. If not, query the network.
            Observable<List<Task>> localTasks = ...
            Observable<List<Task>> remoteTasks = ...
            return Observable.concat(localTasks, remoteTasks)
                    .filter(new Func1<List<Task>, Boolean>() {
                        @Override
                        public Boolean call(List<Task> tasks) {
                            return !tasks.isEmpty();
                        }
                    }).first();
        }
```