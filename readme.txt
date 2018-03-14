JVM 参数

1)TRACE
-verbose:gc
-XX:+printGC
可以打印GC的简要信息

-XX:+PringGCDetails
打印GC的详细信息
-XX:+PrintGCTimeStamps
打印GC发生的时间戳

-Xloggc:
-Xloggc:log/gc.log
重定向GC的log位置

-XX:+PrintHeapAtGC 
每一次GC后，都在前后打印堆信息

-XX:+TraceClassLoading
监控类的加载

-XX:+PrintClassHistogram  （好像有点像JMAP）
按下Ctrl+Break后，打印类的信息
分别显示： 序号，实例数量，总大小， 类型


2）堆的分配
-Xmx 最大
-Xms 最小
Runtime. maxMemory()/freeMemory()/totalMemory()
具体参数

-Xmn 设置新生代的大小
-XX:NewRatio新生代和老年代的比值
-4， 新生代：老年代=1:4，及年轻代占堆1/5
-XX:SurvivorRatio 设置Survivor区和Eden的比
8 表示，连个Survivor:eden=2:8, 即一个Survivor占年轻代的1/10

对于大对象， 如果超过新生代的上限，则不会触发GC，直接进老年代

-XX:+HeapDumpOnOutOfMemoryError
-OOM时导出堆到文件

-XX:+HeapDumpPath
-导出OOM的路径

-XX:OnOutOfMemoryError
-在OOM时，执行一个脚本
可以打印堆栈信息，发送邮件

最佳实践
新生代占堆3/8 (默认值是这个吗)
幸存代占新生代1/10 (默认值是这个吗)
在OOM时，记得Dump出堆，确保可以排查现场问题


永久区分配参数 (一般几十兆，几百兆)
-XX:PermSize 
-XX:MaxPermSize
使用CGLIB等库的时候，可能会产生大量的类，这些类有可能撑爆永久区导致OOM
堆空间没有用完也可能抛出OOM,有可能是永久区导致的



3)栈大小分配
-Xss
通常只有几百K（每个线程启动的分配数，如果想多跑线程，就要减小栈空间，递归调用？？？）
决定了函数调用的深度
每个线程都有独立的栈空间
局部变量，参数都在栈上



GC
FULL GC



GC的对象有堆和永久区

算法
1) 引用计数法
问题：
a.引用和去引用都伴随加减法，影响性能
b.很难处理循环引用
2) 标记-清除
a)标记:通过根节点可达则不可删除，反之，可删除
b)清除标记为垃圾的节点
3)标记-压缩 适用于存活对象较多的场合，如老年代
在标记-清除基础上做了优化，在对所有可达的对象做了一次标记之后。并不是简单的清理未标记的对象，
而是将所有存活对象压缩到内存的一端，之后清理边界外所有的空间
优势？？对比标记-清除
4）复制算法 （但是不适合存活对象较多的场合，如老年代） - survivor
有两块内存空间，并且每次只用一块
复制算法+标记清理思想

根据不同代的特点。选取合适的收集算法
-少量对象存活，适合复制算法
-大量对象存活，适合标记清理或者标记压缩

复制算法- 新生代
标记清除，标记压缩 - 老年代

可触及性
可触及的
- 从根节点可以触及到这个对象
可复活的
- 一旦所有引用被释放，就是可复活状态
- 因为在finalize中可能复活对象
不可触及的
- 在finalize()后，可能会进入不可触及状态
-不可触及的对象不能复活
-可以回收

finalize 只会被调用一次
最佳实践
避免使用finalize，
优先级低，不知道什么时候调用，因为何时发生GC不知道


根
栈中的对象（线程的栈）
方法区中静态成员或者常量引用的对象（全局对象）
JNI方法栈中的对象

Stop-The-World
多半由GC引起
-DUMP线程
-死锁检查
-堆DUMP

危害：
-长时间服务停止，没有响应
-遇到HA系统，可能引起准备切换，严重危害生产环境
































