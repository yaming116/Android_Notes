Gradle Task 顺序
===

记录一下学习**Gradle Task**执行顺序几个关键字
 
 * 依赖: dependsOn ,依赖是一个集合，所有多个依赖是没有问题的。
 
 ```gradle
 task A << {println 'Hello from A'}
 task B << {println 'Hello from B'}
 
 A.dependsOn B
 ```
 或
 ```gradle
 task A << {println 'Hello from A'}
 task B {
     dependsOn A
     doLast {
         println 'Hello from B'  
     }
 }
 ```
 
 * mustRunAfter 必须在什么之后,gradle 2.4 可能删除
 
 ```gradle
  task A << {println 'Hello from A'}
  task B << {println 'Hello from B'}
  task C << {println 'Hello from C'}
  
  A.dependsOn B
  A.dependsOn C
  //如果希望 C在 B之后执行
  C.mashRunAfter B 
 ```
 
 * finalizeBy 当前task执行之后要执行的任务，用于清理工作或合并,gradle 2.4 可能删除
 
  ```gradle
   task A << {println 'Hello from A'}
   task B << {println 'Hello from B'}
   task C << {println 'Hello from C'}
   
   A.dependsOn B
   A.finalizeBy C 
  ```
 
 * 检查当命令环境
 
 ```gradle
 
gradle.taskGraph.whenReady {taskGraph ->
    println taskGraph.getAllTasks()
    if (!taskGraph.hasTask(assembleRelease)) {
        throw new IllegalArgumentException("can't found android.buildTypes.release")
    }
}
 ```