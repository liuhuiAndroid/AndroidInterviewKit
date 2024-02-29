1. Activity 的 ViewTree 结构

2. 页面刷新机制

3. Activity、Dialog、PopupWindow 的区别

4. 常见的导致页面卡顿的原因





Activity 的 attach 方法中会初始化 PhoneWindow 和 WindowManagerImpl

onCreate 生命周期内 setContentView 方法中会创建 DecorView，把我们自己实现的布局构建 View 树并挂载

onResume 生命周期内通过 addView 的方式创建 ViewRootImpl，并用它来管理 View 的生命周期工作，最后再通过 requestLayout 来显示 Activity 中的内容
