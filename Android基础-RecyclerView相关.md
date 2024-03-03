#### RecyclerView 的优势

- 默认支持 Linear，Grid，Staggered Grid 三种布局
- 友好的 ItemAnimator 动画 API
- 强制实现 ViewHolder
- 解耦的架构设计
- 相比 ListView 更好的性能

#### RecyclerView 的重要组件

- LayoutManager 负责布局
- ItemAnimator 负责动画
- Adapter 负责数据
- ViewHolder

#### 简单 Demo

#### ViewHolder 究竟是什么？

- ViewHolder 解决的是什么问题？
  - 解决重复 findViewById，提升效率
- ViewHolder 和 ListView 的 ItemView 是什么关系？一对一？一对多？
  - ItemView 和 ViewHolder 一一对应
- ViewHolder 和 ListView 的 ItemView 的复用有什么关系？
  - 和 ItemView 的复用没有任何关系

#### RecyclerView 缓存机制

通过 LayoutManager Recycler 管理缓存

- Scrap 管理屏幕内部的 ItemView，可以直接复用，根据 position 查找

- Cache 管理屏幕外部的 ItemView，可以直接复用，根据 position 查找

  前两层差不多，都可以直接复用，不需要重新绑定，根据 position 查找

- ViewCacheExtension 用户自定义的 Cache 策略，返回 ItemView

  例子：广告卡片，短期内不会发生变化，可以在 ViewCacheExtension 保存4个广告缓存

- RecyclerViewPool 存放被废弃的 View，数据都是 dirty，通过 ViewType 查找，不需要 onCreateView，但是需要重新 onBindView

全部找不到，Recycler 负责 Create View

列表中广告的显示次数统计： onViewAttachedWindow() 统计

#### RecyclerView 性能优化策略

- 不能在 onBindViewHolder 里面设置点击监听，会反复调用

  在 onCreateViewHolder 里设置点击监听

  View - ViewHolder - View.OnClickListener 三者一一对应

- LinearLayoutManager.setInitialPrefetchItemCount()

  用户滑动到横向滑动的 item RecyclerView 的时候，由于需要创建更复杂的 RecyclerView 以及多个子 view，可能会导致页面卡顿

  由于 RenderThread 的存在，RecyclerView 会进行 prefetch

  LinearLayoutManager.setInitialPrefetchItemCount() 横向列表初次显示时可见的 item个数

  - 只有 LinearLayoutManager 有这个 API
  - 只有嵌套在内部的 RecyclerView 才会生效

- RecyclerView.setHasFixedSize()

  true：layoutChildren()，false：需要 requestLayout()

  如果 Adapter 的数据变化不会导致 RecyclerView 的大小变化：RecyclerView.setHasFixedSize(true)

- 多个 RecyclerView 共用 RecyclerViewPool 

- DiffUtil：计算两个列表的差异，针对差异来更新

  局部更新方法 notifyItemXXX() 不适用于所有情况

  notifyDataSetChange() 会导致整个布局绘制，重新绑定所有 ViewHolder，而且会失去可能的动画效果

  DiffUtil 适用于整个页面需要刷新，但是有部分数据可能相同的情况

  Myers Diff Algorithm 算法 - - 动态规划

  在列表很大的时候异步计算 diff：可以使用 RxJava 或者 AsyncListDiffer / ListAdapter

#### 为什么 ItemDecoration 可以绘制分割线？

ItemDecoration：装饰当前屏幕内显示的内容，实现了解耦，职责很抽象

一条分割线为什么要这么复杂？

左边有间距的分割线怎么画：设置分割线边界

ItemDecoration 还可以做什么？高亮、分组等

可以有多个 ItemDecoration，效果可以叠加

#### 推荐教程

https://github.com/h6ah4i/android-advancedrecyclerview

https://advancedrecyclerview.h6ah4i.com