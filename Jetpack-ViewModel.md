#### 学习 ViewModel

为了帮助开发者更高效、更容易地构建优秀的应用，Google 推出了 Android Jetpack。它包含了开发库、工具、以及最佳实践指南。

其中的 Lifecycle 库可以有效避免内存泄漏和解决常见的 Android 生命周期难题。

Lifecycle 库中的 ViewModel 类：辅助程序类，用来保存应用UI数据，负责为界面准备数据。它会在配置变更后继续存在。

推荐这样的架构设计：将应用所有的UI数据保存在 ViewModel 中，而不是 Activity 中，这样就能确保数据不会受到配置变更的影响。

ViewModel 可以有效地划分责任，符合单一责任原则。具体的，它可以用来保存 Activity 的所有UI 数据，然后 Activity 仅负责了解如何在屏幕上显示该数据和接收用户互动，但是它不会处理这些互动。如果应用加载和存储数据，建议创建一个 Respository 的存储区类，另外需要确保 ViewModel 不会因为承担过多责任而变得臃肿，要避免这种情况，可以创建 Presenter 类，或者实现一种更成熟的架构。

要创建一个 ViewModel，首先需要扩展 ViewModel 类，然后将 Activity 中之前与 UI 相关的实例变量摆放在这个 ViewModel 中，接着，在 Activity 的 onCreate 中，从 ViewModelProvider 的框架实用类再获取 ViewModel。请注意：ViewModelProvider 将获取一个 Activity 实例，这种机制让你可以旋转屏幕，获取一个新 Activity 实例，不过，请确保它始终与同一个 ViewModel 关联。对于 ViewModel 实例，你可以使用 getter 函数，从 Activity 直接获取 UI 数据。ViewModel 的默认构造函数是没有任何参数的，如果你想要修改，可以使用 ViewModelFactory 创建一个自定义构造函数。

ViewModel 类也可以很好地与 LiveData 和 Data Binding 互相搭配使用，使用 ViewModel 和 LiveData，你可以创建反应式界面，也就是说当底层数据被更新时，UI 也会相应的自动更新。假设 ViewModel 包含 LiveData，可以像平常一样利用 DataBinding 来绑定数据。

[第一步：ViewModel官方文档](<https://developer.android.com/topic/libraries/architecture/viewmodel>)

最佳实践：

1. 任何时候都不应该将 Context 传入 ViewModel，也就是说 Fragment、Activity 和 View 都不能被传入。因为ViewModel 可以比相关联的 Activity 和 Fragment 的生命周期更长。
2. 如果需要比 ViewModel 的生命周期更长的 Application 类，你可以使用 AndroidViewModel 子类。通过这个子类，你就可以直接使用 Application 的引用了。
3. ViewModel 不应该取代 onSaveInstanceState 的使用，它们两是相辅相成的。当进程被关闭时，ViewModel 将被销毁，但是 onSaveInstanceState 将不会受到影响。ViewModel 可以用来存储大量数据，而 onSaveInstanceState 只可以用来存储有限的数据。我们尽可能把多一点的 UI 数据往 ViewModel 内存储，以便在配置变更时不需要重新加载或生成数据。另一方面，如果进程被 framework 关闭，我们应该用 onSaveInstanceState 来存储足以还原UI状态的最少量数据。

[视频 Android Jetpack - ViewModel](<https://www.bilibili.com/video/av29949898>)

[《即学即用Android Jetpack - ViewModel & LiveData》](https://www.jianshu.com/p/81a284969f03)

------------------------------------------------------------------------

[【AAC 系列四】深入理解架构组件：ViewModel](https://juejin.im/post/5d0111c1e51d45108126d226)

[ViewModel 和 ViewModelProvider.Factory](https://blog.csdn.net/qq_43377749/article/details/100856599)

#### ViewModel 简单使用

```kotlin
	// deprecated
	ViewModelProviders.of(this).get(ShowHouseViewModel.class)
	// API 变更
	ViewModelProvider(this)[ShowHouseViewModel::class.java]
```

#### 使用 ViewModelProvider.NewInstanceFactory() 的好处

默认的 ViewModelProvider.Factory 默认是用 ViewModel 的无参构造实例化 ViewModel。如果想在构造方法中添加参数，你必须编写自己的 ViewModelProvider 来创建 ViewModel 实例。

ViewModelProviders.Factory 负责实例化 ViewModel 对象。

如果你的 ViewModel 没有依赖项，这时你就不需要去自己创建 ViewModelProvider.Factory。可以使用系统自带的方法帮助你创建 ViewModel。

```kotlin
	private val showHouseViewModel: ShowHouseViewModel by viewModels {
        CustomViewModelProvider.providerShowHouseViewModel()
    }

    object CustomViewModelProvider {

        fun providerShowHouseViewModel(): ShowHouseViewModelFactory {
            return ShowHouseViewModelFactory()
        }
    }

    class ShowHouseViewModelFactory: ViewModelProvider.NewInstanceFactory() {

        override fun <T : ViewModel?> create(modelClass: Class<T>): T {
            return ShowHouseViewModel() as T
        }
    }
```

#### ViewModel 原理分析

1. ViewModelProvider(this)[ShowHouseViewModel::class.java]

   ```java
       public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
           // 第一个参数是：ViewModelStore
           this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
                   ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
                   : NewInstanceFactory.getInstance());
       }
   ```

   其中：getDefaultViewModelProviderFactory 在 ComponentActivity 返回的是 SavedStateViewModelFactory，它持有 AndroidViewModelFactory

   NewInstanceFactory 和 AndroidViewModelFactory 都是通过反射来构建 ViewModel 的工厂类，且都是个单例。

2. ViewModel 的 class 是传给了 ViewModelProvider.get() 方法

   ```java
       public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
           String canonicalName = modelClass.getCanonicalName();
           if (canonicalName == null) {
               throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
           }
           // 每个 ViewModel 类都有一个唯一的 key
           return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
       }
   
       public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
           // 通过 key 尝试先从 ViewModelStore 中获取 ViewModel 的实例
           // ViewModelStore 负责存储 ViewModel
           ViewModel viewModel = mViewModelStore.get(key);
   
           if (modelClass.isInstance(viewModel)) {
               if (mFactory instanceof OnRequeryFactory) {
                   ((OnRequeryFactory) mFactory).onRequery(viewModel);
               }
               return (T) viewModel;
           } else {
               //noinspection StatementWithEmptyBody
               if (viewModel != null) {
                   // TODO: log a warning.
               }
           }
           // 通过 Factory 去创建 ViewModel 实例
           if (mFactory instanceof KeyedFactory) {
               viewModel = ((KeyedFactory) (mFactory)).create(key, modelClass);
           } else {
               viewModel = (mFactory).create(modelClass);
           }
           // 把新的 ViewModel 实例存入到 ViewModelStore
           mViewModelStore.put(key, viewModel);
           return (T) viewModel;
       }
   ```

3. owner.getViewModelStore()

   ```java
   public interface ViewModelStoreOwner {
       @NonNull
       ViewModelStore getViewModelStore();
   }
   ```

4. 查看 ViewModelStoreOwner 实现

   ```java
   public class ComponentActivity{
   
       	// ViewModelStore 相关的数据都会通过这个类进行保存，以免被销毁
           static final class NonConfigurationInstances {
               Object custom;
               ViewModelStore viewModelStore;
           }
       
           // Lazily recreated from NonConfigurationInstances by getViewModelStore()
           private ViewModelStore mViewModelStore;
       
      		// 在配置改变时保存需要的实例
           public final Object onRetainNonConfigurationInstance() {
               Object custom = onRetainCustomNonConfigurationInstance();
   
               ViewModelStore viewModelStore = mViewModelStore;
               if (viewModelStore == null) {
                   // No one called getViewModelStore(), so see if there was an existing
                   // ViewModelStore from our last NonConfigurationInstance
                   NonConfigurationInstances nc =
                           (NonConfigurationInstances) getLastNonConfigurationInstance();
                   if (nc != null) {
                       viewModelStore = nc.viewModelStore;
                   }
               }
   
               if (viewModelStore == null && custom == null) {
                   return null;
               }
   
               NonConfigurationInstances nci = new NonConfigurationInstances();
               nci.custom = custom;
               nci.viewModelStore = viewModelStore;
               return nci;
           }
       
           public ViewModelStore getViewModelStore() {
               if (getApplication() == null) {
                   throw new IllegalStateException("Your activity is not yet attached to the "
                           + "Application instance. You can't request ViewModel before onCreate call.");
               }
               if (mViewModelStore == null) {
                   NonConfigurationInstances nc =
                           (NonConfigurationInstances) getLastNonConfigurationInstance();
                   if (nc != null) {
                       // Restore the ViewModelStore from NonConfigurationInstances
                       mViewModelStore = nc.viewModelStore;
                   }
                   if (mViewModelStore == null) {
                       mViewModelStore = new ViewModelStore();
                   }
               }
               return mViewModelStore;
       	}
       
   }
   ```

   总结一下就是利用 onRetainNonConfigurationInstance() 方法在旋转屏幕时保存 ViewModelStore 实例，并在重新创建时重新赋值。