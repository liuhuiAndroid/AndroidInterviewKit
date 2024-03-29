#### 什么是内存泄露？

传统定义的内存泄露：申请的内存忘记释放了。
Android 的内存泄露：短⽣命周期的对象被⻓⽣命周期的对象持有，从⽽导致短⽣命周期的对象不能被释放。这可能导致应用程序占用过多内存，最终影响性能和稳定性。
常见原因：保持引用、上下文引用、非静态内部类、未关闭资源
解决方法：注意对象引用的生命周期、使用弱引用、正确处理上下文引用、避免非静态内部类



#### 内存泄露的问题

内存泄漏并不会⻢上把程序搞挂掉。但是随着应⽤的使⽤，不能回收的垃圾对象会越来越多，就导致了可⽤的内存越来越少，到最后应⽤就有可能在任何位置抛出OutOfMemoryError ，这种情况下，每次 OOM 错误堆栈都不同，就很难定位问题。



#### 垃圾回收机制

如何判断一个对象是否是垃圾对象

- 引用计数法
- 可达性分析：GC Root 向下搜索



#### 监控 Activity 和 Fragment 原理

通过注册 Application 和 Fragment 上的⽣命周期回调来完成在 Activity 和 Fragment 销毁的时候开始观察。



### 四⼤引⽤

- 强引⽤——不会被垃圾回收
- 弱引⽤——可以通过 get() 获得引⽤对象，会被垃圾回收
- 软引⽤——可以通过 get() 获得引⽤对象，内存不⾜会被垃圾回收
- 虚引⽤——不能通过 get() 获得引⽤对象，会被垃圾回收

弱引⽤在引⽤对象被垃圾回收之前，会将引⽤放⼊它关联的队列中。所以可以通过队列中是否有对应的引⽤来判断对象是否被垃圾回收了。



### watch() ⽅法

原理：就是通过弱引⽤的⽅式来判断队列中是否有弱引⽤来判断对象是否被垃圾回收了

通过 dumpHprofData 来获取 hprof ⽂件

分析会在独⽴进程中进⾏， 1.x 内部使⽤ haha ， 2.x 内部使⽤ shark 。

在 1.x 版本中，通过 haha 第三⽅库进⾏堆⽂件分析。

在 2.x 版本中，通过 shark 第三⽅库进⾏堆⽂件分析，内存占⽤⼤幅度减少，分析速度⼤幅度提⾼



LeakCanary 是一个用于检测Android应用中内存泄漏的开源库。它的原理是通过监视应用程序的堆内存，检测对象的引用关系，从而发现潜在的内存泄漏问题。

1. 对象引用追踪： LeakCanary会监视应用程序的堆内存，定期检查对象的引用关系。它会查找无法被释放的对象引用。
2. 弱引用和弱引用队列： LeakCanary 使用弱引用和弱引用队列（WeakReference和WeakReferenceQueue）来监视对象的生命周期。如果一个对象只被弱引用持有，并且没有强引用指向它，那么它会被认为是可以被垃圾回收的。
3. Heap Dump： 当 LeakCanary 检测到潜在的内存泄漏时，它会触发应用程序生成堆转储（heap dump）。堆转储是应用程序内存状态的快照，可以在后续进行分析。
4. 分析工具： LeakCanary 使用分析工具来分析堆转储，识别泄漏的对象和引用链。这些分析工具能够帮助确定哪个对象是泄漏的、泄漏的原因是什么以及在哪个部分的代码中发生的。
5. 通知开发者： 一旦 LeakCanary 检测到潜在的内存泄漏，它会向开发者发出通知，通常是通过通知栏、日志输出或者其他途径。通知包含了泄漏对象的详细信息，帮助开发者迅速定位和修复问题。



#### LeakCanary 如何使用弱引用和弱引用队列来监视对象的生命周期

如果一个对象被垃圾回收器回收，而弱引用队列中保留了对该对象的引用，那么这意味着该对象在某些地方仍然被持有。LeakCanary 正是通过这种方式来发现潜在的内存泄漏。

1. **正常情况：** 如果一个对象在没有内存泄漏的情况下被垃圾回收，它的弱引用应该会被清理掉，不会出现在弱引用队列中。
2. **内存泄漏情况：** 如果一个对象发生了内存泄漏，即使被垃圾回收器回收，该对象的引用可能仍然存在于某个地方，导致它的弱引用在弱引用队列中被注册。LeakCanary 正是通过监测这些弱引用队列中的对象，来发现潜在的内存泄漏。

通过这种方式，LeakCanary能够帮助开发者检测出在应用中被意外保留的对象，帮助追踪和解决内存泄漏问题。



#### LeakCanary 可不可以使用软引用来监视对象的生命周期

使用软引用来监视对象的生命周期在 LeakCanary 的场景中并不适用，因为它会在内存不足时失去其监视的对象，从而无法准确检测内存泄漏。



#### LeakCanary 2.0 不需要主动初始化的原理

ContentProvider#onCreate 会在 Application#onCreate 之前先执⾏ ，在这个 onCreate 中就可以进⾏初始化了。 LeakCanary 2.0 利⽤了这个原理，所以不需要我们⼿动进⾏初始化
