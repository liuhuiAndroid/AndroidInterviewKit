1. Activity 的 ViewTree 结构

2. 页面刷新机制

3. Activity、Dialog、PopupWindow 的区别

4. 常见的导致页面卡顿的原因

Activity 的 attach 方法中会初始化 PhoneWindow 和 WindowManagerImpl

onCreate 生命周期内 setContentView 方法中会创建 DecorView，把我们自己实现的布局构建 View 树并挂载

onResume 生命周期内通过 addView 的方式创建 ViewRootImpl，并用它来管理 View 的生命周期工作，最后再通过 requestLayout 来显示 Activity 中的内容



#### Activity、Window 和  View 关系

1. `Activity`通过其`Window`来承载UI，`Window`是`Activity`的根容器。

2. `Activity`的界面元素（`View`）被添加到`Window`中，通过`setContentView`方法设置布局。

3. `View`是`Window`中显示的实际内容，它们构成了用户界面的各个元素。

在Android应用中，这三者共同协作，`Activity`负责管理应用的生命周期和用户交互，`Window`负责显示和管理`Activity`的界面，而`View`则是构成界面的基本单元。
