#### ViewPager2 与 ViewPager 比较

ViewPager2 的核心实现就是 RecyclerView + LinearLayoutManager

ViewPager 的弊端：

1. 不能关闭预加载

ViewPager2 的优势：

1. 默认是开启预加载，关闭离屏加载

#### ViewPager2 简单使用

#### TabLayout 结合 ViewPager2

```java
	//限制页面预加载
	viewPager2.setOffscreenPageLimit(ViewPager2.OFFSCREEN_PAGE_LIMIT_DEFAULT);
    //viewPager2默认只有一种类型的Adapter。FragmentStateAdapter
    //并且在页面切换的时候 不会调用子Fragment的setUserVisibleHint ，取而代之的是onPause(),onResume()、
    viewPager2.setAdapter(new FragmentStateAdapter(getChildFragmentManager(), this.getLifecycle()) {
        @NonNull
        @Override
        public Fragment createFragment(int position) {
            // FragmentStateAdapter内部自己会管理已实例化的fragment对象。
            return getTabFragment(position);
        }

        @Override
        public int getItemCount() {
            return tabs.size();
        }
    });
    //viewPager2 就不能和再用TabLayout.setUpWithViewPager()了
    //取而代之的是TabLayoutMediator。我们可以在onConfigureTab()方法的回调里面 做tab标签的配置
    //其中autoRefresh的意思是:如果viewPager2 中child的数量发生了变化，也即我们调用了adapter#notifyItemChanged()前后getItemCount不同。
    //要不要重新刷野tabLayout的tab标签。视情况而定,像咱们sofaFragment的tab数量一旦固定了是不会变的，传true/false  都问题不大
    mediator = new TabLayoutMediator(tabLayout, viewPager2, true, new TabLayoutMediator.TabConfigurationStrategy() {
        @Override
        public void onConfigureTab(@NonNull TabLayout.Tab tab, int position) {
            tab.setCustomView(makeTabView(position));
        }
    });
    mediator.attach();

    viewPager2.registerOnPageChangeCallback(mPageChangeCallback);
    //切换到默认选择项,那当然要等待初始化完成之后才有效
    viewPager2.post(() -> viewPager2.setCurrentItem(tabConfig.select, false));
```

#### ViewPager2 源码分析



#### 参考

[views-widgets-samples](https://github.com/android/views-widgets-samples)

[深入了解ViewPager2](https://zhuanlan.zhihu.com/p/97511079?from_voters_page=true)

