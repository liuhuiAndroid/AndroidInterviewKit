### Jetpack

为了帮助开发者更高效、更容易地构建优秀的应用，Google 推出了 Android Jetpack。它包含了开发库、工具、以及最佳实践指南。

#### DataBinding



#### ViewModel

ViewModel 原理分析

利用 onRetainNonConfigurationInstance() 方法在旋转屏幕时保存 ViewModelStore 实例，并在重新创建时重新赋值。

#### LiveData 和 Lifecycle

Lifecycle 原理分析

1. Lifecycle 利用 ReportFragment 来实现监听生命周期，ReportFragment 重写了生命周期回调的方法，并在生命周期回调里调用了内部 dispatch 的方法来分发生命周期事件
2. ComponentActivity#onCreate 方法注入了 ReportFragment，通过 Fragment 来实现生命周期监听
3. LifecycleRegistry#handleLifecycleEvent 方法接收事件
4. Lifecycle 的生命周期事件与状态的定义

#### Room



#### Paging



#### WorkManager

学习 WorkManager - 满足您的后台调度需求



#### CameraX



#### Hilt

- 依赖注入有什么用？

  内部的值不由自己来提供，交给外部。

  - 自动加载
  - 自动加载的关键：数据共享
  - 不共享的数据需要依赖注入吗？也需要；可能需要在多处使用且逻辑一样，少些重复代码

- Hilt 有什么用？

  - 更方便地进行依赖注入

- 要用 Hilt 吗？

  - Dagger2
    - Dagger 需要手动注入
    - 需要自行定义 Scope
    - Dagger 复杂且完备；Hilt 对 Dagger 场景化，专门应用于 Android 开发
    - Dagger 有一定的上手成本
    - Dagger 依赖关系无法追踪；Hilt 可以追踪
    - Dagger 编译前生成代码，增加初次编译时间
  - Koin
    - 比 Dagger 方便
    - 运行时动态进行

- ButterKnife 是依赖注入吗？不是依赖注入，是视图绑定

#### ViewPager2

- ViewPager2 与 ViewPager 比较

  ViewPager2 的核心实现就是 RecyclerView + LinearLayoutManager

  ViewPager 的弊端：不能关闭预加载

  ViewPager2 的优势：默认是开启预加载，关闭离屏加载
