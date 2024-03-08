网络请求使用 LiveData 订阅后，在页面销毁后可以自动取消订阅

`ViewModel` 的生命周期会比与其关联的 `Fragment` 或 `Activity` 的生命周期长。`ViewModel` 的生命周期由它所关联的 `Fragment` 或 `Activity` 的生命周期范围决定。当 `Fragment` 或 `Activity` 被销毁时，`ViewModel` 会继续存在，直到关联的 `ViewModelStore` 被清理，这通常发生在应用关闭或进程终止的情况下。



#### ViewModel的作用和优势

ViewModel 的作用在于解决 Android 应用中 Activity 和 Fragment 的生命周期问题。它允许数据在屏幕旋转等配置更改时存活，并确保数据在不同组件之间共享而不丢失。主要优势包括：

- 生命周期感知：ViewModel能够感知与UI相关的生命周期变化，确保数据存活时间比短暂的UI组件更长。
- 数据共享：通过ViewModel，可以在不同的UI组件之间共享和管理数据，避免重复加载或丢失数据。
- 状态保存：ViewModel在配置变更时保持其状态，例如屏幕旋转，避免重新加载数据和执行耗时操作。



#### LiveData 和 ViewModel 的工作原理

LiveData是一种可观察的数据持有者，ViewModel用于存储和管理与用户界面相关的数据。深入理解包括：

- **LiveData的粘性事件：** 了解`postValue`和`setValue`的区别，以及如何避免LiveData的粘性事件在特定场景中引发的问题。
- **ViewModel的存活周期：** 使用`ViewModel`正确处理配置变化，保证数据在屏幕旋转等情况下不丢失。
