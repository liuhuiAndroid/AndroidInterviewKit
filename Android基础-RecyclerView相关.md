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
  
- AsyncListDiffer — RecyclerView最好的伙伴

#### 为什么 ItemDecoration 可以绘制分割线？

ItemDecoration：装饰当前屏幕内显示的内容，实现了解耦，职责很抽象

一条分割线为什么要这么复杂？

左边有间距的分割线怎么画：设置分割线边界

ItemDecoration 还可以做什么？高亮、分组等

可以有多个 ItemDecoration，效果可以叠加

#### 推荐教程

https://github.com/h6ah4i/android-advancedrecyclerview

https://advancedrecyclerview.h6ah4i.com

#### RecyclerView最好的伙伴：AsyncListDiffer

用于优化 RecyclerView 刷新效率，AsyncListDiffer 比 DiffUtil 更新

#### 自定义 LayoutManager

重写 smoothScrollToPosition 方法配合 SmoothScroller 可以控制滚动位置和滚动速度

#### SortedList 使用

如果列表需要排序的话，可以使用这个集合来代替，比较方便和高效。

```kotlin
private val dataList =
SortedList<String>(String::class.java, object : SortedListAdapterCallback<String>(this) {
    // 比较两个Item的内容是否一致，如不一致则会调用adapter的notifyItemChanged()
    override fun areItemsTheSame(item1: String?, item2: String?): Boolean = false

    // 用于排序，大于0升序，小于0降序，等于0不变
    override fun compare(o1: String?, o2: String?): Int = 0

    // 两个Item是不是同一个，可以用他们的id，或者其他的字段进行比较是否一样
    override fun areContentsTheSame(oldItem: String?, newItem: String?): Boolean = true
})
```

批量更新和删除

```kotlin
// 调用beginBatchedUpdates()之后，所有的对SortedList操作都会等到endBatchedUpdates()之后一起生效。
dataList.beginBatchedUpdates();  // 开始批量更新
dataList.addAll(items);          // 更新一批数据
dataList.remove(item);           // 删除一批数据
dataList.endBatchedUpdates();    // 结束更新
```

#### 预取 Prefetch

 [RecyclerView 数据预取](https://juejin.im/entry/58a3f4f62f301e0069908d8f)

```kotlin
LayoutManager.setItemPrefetchEnabled()
LayoutManager.setInitialPrefetchItemCount()
```

#### 缓存

- Scrap 保存被移除掉但可能马上要使用的缓存，优先考虑

  ```java
  // 存储的是刚从屏幕上分离出来的，但是又即将添加到屏幕上去的 ViewHolder
  final ArrayList<ViewHolder> mAttachedScrap = new ArrayList<>();
  // 存储的是数据被更新的 ViewHolder
  ArrayList<ViewHolder> mChangedScrap = null;
  ```

- Cache

  缓存默认大小为2，可以通过 **setViewCacheSize** 改变容量大小

  存储预取的 ViewHolder，同时在回收 ViewHolder 时，也会可能存储一部分的 ViewHolder

  ```java
  final ArrayList<ViewHolder> mCachedViews = new ArrayList<ViewHolder>();
  ```

- ViewCacheExtension 

  自定义缓存，通常用不到

  ```java
  private ViewCacheExtension mViewCacheExtension;
  ```

- RecyclerViewPool 

  根据 ViewType 缓存无效的 ViewHoler，数组大小为5

  mCachedViews 存不下的会被保存到 mRecyclerPool 中，mRecyclerPool 保存满之后只能无情抛弃掉，它也有一个默认的容量大小

  ```java
  RecycledViewPool mRecyclerPool;
  ```

#### 优化

- 关闭默认动画提升效率：

  ```java
   /**
    * 打开默认局部刷新动画
    */
   public void openDefaultAnimator() {
       this.getItemAnimator().setAddDuration(120);
       this.getItemAnimator().setChangeDuration(250);
       this.getItemAnimator().setMoveDuration(250);
       this.getItemAnimator().setRemoveDuration(120);
       ((SimpleItemAnimator) this.getItemAnimator()).setSupportsChangeAnimations(true);
   }
  
   /**
    * 关闭默认局部刷新动画
    */
   public void closeDefaultAnimator() {
       this.getItemAnimator().setAddDuration(0);
       this.getItemAnimator().setChangeDuration(0);
       this.getItemAnimator().setMoveDuration(0);
       this.getItemAnimator().setRemoveDuration(0);
       ((SimpleItemAnimator) this.getItemAnimator()).setSupportsChangeAnimations(false);
   }
  ```

- 使用 ConstraintLayout 替换复杂的布局

- 使用 DiffUtil 进行差异化比较

- Item 高度是固定，使用 RecyclerView.setHasFixedSize(true) 避免重复测量

- 多个 RecycledView 的 Adapter 一样，可以共用 RecyclerViewPool 

- 加大缓存，用空间换时间来提高滚动的流畅性

  ```kotlin
  recyclerView.setItemViewCacheSize(20)
  // 开启
  recyclerView.setDrawingCacheEnabled(true)
  recyclerView.setDrawingCacheQuality(View.DRAWING_CACHE_QUALITY_HIGH)
  ```

- 共用监听器

- 数据缓存，提高二次加载速度

- onCreateViewHolder 中 setOnClickListener

- 不到万不得已不要全局刷新，使用局部刷新

  - 如果使用了全局刷新，Adapter 重写 getItemId 方法，并且设置 setHasStableIds(true)

#### 其他

- RecyclerView 滑动时图片加载的优化

  在滑动时停止加载图片，在滑动停止时开始加载图片，这里用了Glide.pause 和 Glide.resume

#### 参考

- [RecyclerView一些你可能需要知道的优化技术](https://www.jianshu.com/p/1d2213f303fc)
- [RecyclerView 性能优化](https://juejin.im/post/5baedbf05188255c596714ab)
- [RecyclerView Scrolling Performance](https://stackoverflow.com/questions/27188536/recyclerview-scrolling-performance)