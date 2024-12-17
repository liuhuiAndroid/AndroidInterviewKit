1. Flow StateFlow和ShareFlow有什么区别？

   https://juejin.cn/post/7397019920284074022

   Flow 是冷流，因此它需要消费端主动调用 collect 才能触发数据的流动， 而一旦消费端停止 collect 或者生产端结束，Flow 会自动关闭。除此之外， Flow 也没有数据缓存的机制，无法主动获取当前发送的数据。

   与 Flow 不同的是，SharedFlow 和 StateFlow 是热流，不依赖 collect 来触发数据的流动。同时 SharedFlow 和 StateFlow 都支持数据共享的，内部也有缓存，可以获取发送的数据。

   StateFlow 中文翻译是状态流，主要用于共享一个状态的数据流时。在功能上，StateFlow 可以说就是一个 LiveData。不同于 StateFlow 和 LiveData，默认配置下 SharedFlow 既不会重放数据，也不会丢失数据。

2. suspend 关键字作用

   标记函数是挂起函数

3. Activity 使用协程展示网络数据流程

4. ActivityScope 和 ViewModelScope 区别

   viewModelScope 是一个内置 CoroutineScope，会自动遵循 ViewModel 的生命周期。

5. 页面全局消息监听实现方案？广播？

6. 包体积优化

7. 布局优化，绘制优化，解决IO问题嵌套问题

8. 自定义 View 实现 FlexboxLayout 瀑布流的效果，如何测量布局绘制。

9. OkHttp 线程池是什么实现的，拦截器流程。

10. Glide 如何实现生命周期监听，最新实现方式是什么。

   生产用于监听的 Fragment

11. ConstraintLayout Barrier

12. 实现生产者消费者模型，sync 实现方式， ArrayBlockingQueue 实现方式，rxjava，协程

13. synchronized 为什么是重量级锁，除了 synchronized 还有哪些锁，比如读锁，写锁

    - synchronized 锁粒度大，在锁被占用时，依赖操作系统 pthread_mutex_lock 实现，导致线程在用户态和内核态之间切换，消耗更多资源。

14. ANR 如何解决， ANRWatchDog、火焰图；使用 Systrace 或 Perfetto 工具

    - 避免主线程阻塞，确保耗时任务在工作线程中完成，主线程仅用于处理 UI 相关的逻辑。
    - 排查线程同步问题检查是否存在锁竞争，避免长时间占用主线程。
    - 优化布局和绘制性能，使用 ViewStub 或延迟加载减少布局复杂度。

15. LeakCanary 生成新的桌面图标实现原理，说出一个常见的内存泄漏。

    定义了一个 DisplayLeakActivity，设置了 <action android:name="android.intent.action.MAIN"/> 和 <category android:name="android.intent.category.LAUNCHER"/>；2.0 改成 activity-alias 实现。

    一个 Applicaiton 下可以有多个 activity 使用上述的 intent-filter。

