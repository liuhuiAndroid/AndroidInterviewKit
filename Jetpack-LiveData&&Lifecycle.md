#### 学习 LiveData 和 Lifecycle

为了帮助开发者更高效、更容易地构建优秀的应用，Google 推出了 Android Jetpack。它包含了开发库、工具、以及最佳实践指南。

Lifecycle 库：它可以有效的避免内存泄漏，和解决 Android 生命周期的常见难题

Lifecycle 库中的 LiveData 类：LiveData 是一种具有生命周期感知能力的可观察数据持有类，LiveData 可以使屏幕上显示的内容与数据随时保持同步。

LiveData 对象通常保存在 ViewModel 类中，假设你正在为某个 User 对象创建 Activity 和 ViewModel，我们将使用一个 UserLiveData 对象来保存这个 User 对象，接下来，在 Activity 的 onCreate 方法中，我们可以从 ViewModel 中获取 LiveData，并在 LiveData 上调用 observe 方法，方法中第一个参数为 Context，第二个参数是一个“Observer 观察者”，属于方法回调，回调之后界面会被更新。如果需要更新 LiveData，可以调用 setValue 或 postValue 方法，两者的区别在于，setValue 只可以在主线程上运行，postValue 只可以在后台线程上运行。调用 setValue 或 postValue 时，LiveData 将通知 Observer，并更新界面。

现支持 LiveData、ViewModel 与数据绑定的搭配使用，通常情况下会在XML布局内绑定 ViewModel，将 ViewModel 和数据绑定布局关联后，添加 binding.setLifecycleOwner(this) 后，即可让XML内绑定的 LiveData 和 Observer 建立观察连接，接下来，在XML中添加对 ViewModel 的引用，LiveData 包含在 ViewModel 之中。如果使用数据绑定，那么就不需要像我们之前介绍的那样，在 LiveData 上调用 observe 方法了，可以在XML中 TextView 上直接引用 LiveData。LiveData 与其他可观察对象的不同之处在于，它可以感知元件的生命周期。因为我们在调用 observe 方法时用了 UI 界面参数，所以 LiveData 了解界面的状态。

LiveData生命周期感知的优势有以下几点：

如果 Activity 不在屏幕上，LiveData 不会触发没必要的界面更新，如果 Activity 已销毁，LiveData 将自动清空与 Observer 的连接。这样，屏幕外或者已销毁的 Activity 或 Fragment，不会被意外地调用。

生命周期感知的实现，得益于 Android Framework使用了以下 Lifecycles 中的类：

Lifecycle：表示 Android 生命周期及状态的对象

LifecycleOwner：用于连接有生命周期的对象，例如 AppCompatActivity 和 ActivityFragment

LifecycleObserver：用于观察 LifecycleOwner 的接口

LiveData 是一个 LifecycleObserver，它可以直接感知 Activity 或 Fragment 的生命周期。

复杂用例：

Room + LiveData：Room 数据框架可以很好地支持 LiveData，Room 返回的 LiveData 对象，可以在数据库数据变化时自动收到通知，还可以在后台线程中加载数据，这样我们可以在数据库更新的时候，轻松地更新界面。

LiveData 的数据转换：

map() 可以转换 LiveData 的输出：可以将输出 LiveData A 的方法传递到 LiveData B

switchMap() 可以更改被 LiveData 观察的对象，switchMap 和 map 很相似，区别在于，switchMap 给出的是 LiveData，而 map 方法给出的是具体值

MediatorLiveData 可以用于自定义转换，它可以添加或者移除原 LiveData 对象，然后多个原 LiveData，可以进行被组合，作为单一 LiveData 向下传递。

[第一步：LiveData 官方文档](<https://developer.android.com/topic/libraries/architecture/livedata>)

[Android 架构组件基本示例](https://github.com/googlesamples/android-architecture-components/tree/master/BasicSample)

[视频 Android Jetpack: LiveData 和 Lifecycle](<https://www.bilibili.com/video/av33633628>)

[《即学即用Android Jetpack - ViewModel & LiveData》](https://www.jianshu.com/p/81a284969f03)

[MVVM项目实战之路-搭建一个登录界面](<https://cloud.tencent.com/developer/article/1153469>)

-------------

[【AAC 系列二】深入理解架构组件的基石：Lifecycle](https://juejin.im/post/5cd81634e51d453af7192b87)

[Android-Lifecycle超能解析](https://www.jianshu.com/p/2c9bcbf092bc)

[【AAC 系列三】深入理解架构组件：LiveData](https://juejin.im/post/5ce54c2be51d45106343179d)

#### 具体使用

- MutableLiveData

  LiveData 的一个最简单实现，它可以接收数据更新并通知观察者

- Transformations#map()

  将数据从一个 LiveData 传递到另一个 LiveData

- MediatorLiveData

  将多个 LiveData 源数据集合起来

- Transformations#switchMap

  用来添加一个新数据源并相应地删除前一个数据源

#### Lifecycle 重要角色

- ##### LifeCycleOwner

  生命周期拥有者，即Activity与Fragment

  ```java
  public class ComponentActivity implements LifecycleOwner{}
  public class Fragment implements LifecycleOwner{}
  ```

  ```java
  public interface LifecycleOwner {
      // 只有一个方法，让拥有者获取 Lifecycle
      Lifecycle getLifecycle();
  }
  ```

- **LifeCycleObserver**

  生命周期观察者，可以是任何类

#### Lifecycle 原理分析

1. 简单使用

   ```kotlin
       // LifeCycleOwner 和 LifeCycleObserver 建立联系
       lifecycle.addObserver(object : LifecycleObserver {
           @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
           fun onResume(){
               Log.d("TAG", "LifecycleObserver onResume called");
           }
       })
   ```

   ```java
       // 推荐使用 Java8 不使用注解的方式
       // implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"
       // If you use Java 8 language, always prefer it over annotations.
   	lifecycle.addObserver(object : DefaultLifecycleObserver {
       	// 感知生命周期
           override fun onResume(owner: LifecycleOwner) {
   			Log.d("TAG", "LifecycleObserver onResume called");
           }
       })
   ```

2. Lifecycle 利用 ReportFragment 来实现监听生命周期，ReportFragment 重写了生命周期回调的方法，并在生命周期回调里调用了内部 dispatch 的方法来分发生命周期事件

   ```java
   public class ReportFragment extends Fragment {
   
       @Override
       public void onStart() {
           super.onStart();
           dispatchStart(mProcessListener);
           dispatch(Lifecycle.Event.ON_START);
       }
   
       @Override
       public void onResume() {
           super.onResume();
           dispatchResume(mProcessListener);
           dispatch(Lifecycle.Event.ON_RESUME);
       }
   
       private void dispatch(@NonNull Lifecycle.Event event) {
           if (Build.VERSION.SDK_INT < 29) {
               dispatch(getActivity(), event);
           }
       }
       
       static void dispatch(@NonNull Activity activity, @NonNull Lifecycle.Event event) {
           if (activity instanceof LifecycleRegistryOwner) {
               ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
               return;
           }
   
           if (activity instanceof LifecycleOwner) {
               Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
               if (lifecycle instanceof LifecycleRegistry) {
                   ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
               }
           }
       }
   }
   ```

3. ComponentActivity#onCreate 方法注入了 ReportFragment，通过 Fragment 来实现生命周期监听

   ```java
   public class ComponentActivity implements LifecycleOwner{
       
       // 负责管理 Observer
       private final LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);
       
       @Override
       protected void onCreate(@Nullable Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           // 注入 ReportFragment
           ReportFragment.injectIfNeededIn(this);
       }
       
       @Override
       protected void onSaveInstanceState(@NonNull Bundle outState) {
           Lifecycle lifecycle = getLifecycle();
           if (lifecycle instanceof LifecycleRegistry) {
               ((LifecycleRegistry) lifecycle).setCurrentState(Lifecycle.State.CREATED);
           }
           super.onSaveInstanceState(outState);
       }
       
       @Override
       public Lifecycle getLifecycle() {
           return mLifecycleRegistry;
       }
   }
   ```

4. LifecycleRegistry#handleLifecycleEvent 方法接收事件

   ```java
   // LifecycleRegistry 是 Lifecycle 的实现类，负责管理 Observer
   public class LifecycleRegistry extends Lifecycle {
       // 
       protected void onSaveInstanceState(@NonNull Bundle outState) {
           Lifecycle lifecycle = getLifecycle();
           // 把 Lifecycle 状态标记为 Lifecycle.State.CREATED
           // 其余的操作都交给 ReportFragment 处理
           if (lifecycle instanceof LifecycleRegistry) {
               ((LifecycleRegistry) lifecycle).setCurrentState(Lifecycle.State.CREATED);
           }
           super.onSaveInstanceState(outState);
           mSavedStateRegistryController.performSave(outState);
       }
       
       // 
       public void addObserver(@NonNull LifecycleObserver observer) {
           State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
           ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
           // 把 observer 维护到 ObserverWithState 然后装到 mObserverMap 里
           ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);
           // ... 
       }
       
   	// Sets the current state and notifies the observers.
       // 处理状态，遍历 mObserverMap 来通知 observers
       public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
           State next = getStateAfter(event);
           moveToState(next);
       }
       
   	private void moveToState(State next) {
           sync();
       }
       
       // 对比了当前 mState 以及上一个 mState，判断是应该前移还是后退
       private void sync() {
           while (!isSynced()) {
               if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
                   backwardPass(lifecycleOwner);
               }
               if (!mNewEventOccurred && newest != null
                       && mState.compareTo(newest.getValue().mState) > 0) {
                   forwardPass(lifecycleOwner);
               }
           }
       }
       
       private void backwardPass(LifecycleOwner lifecycleOwner) {
           Iterator<Entry<LifecycleObserver, ObserverWithState>> descendingIterator =
                   mObserverMap.descendingIterator();
           while (descendingIterator.hasNext() && !mNewEventOccurred) {
               Entry<LifecycleObserver, ObserverWithState> entry = descendingIterator.next();
               ObserverWithState observer = entry.getValue();
               while ((observer.mState.compareTo(mState) > 0 && !mNewEventOccurred
                       && mObserverMap.contains(entry.getKey()))) {
                   Event event = downEvent(observer.mState);
                   pushParentState(getStateAfter(event));
                   // 重点，分发 Event 
                   observer.dispatchEvent(lifecycleOwner, event);
                   popParentState();
               }
           }
       }
       
       static class ObserverWithState {
           LifecycleEventObserver mLifecycleObserver;
           
           void dispatchEvent(LifecycleOwner owner, Event event) {
               // 最终调到这里
               mLifecycleObserver.onStateChanged(owner, event);
           }
       }
   }
   
   // 可以发现 mLifecycleObserver 其实是 ReflectiveGenericLifecycleObserver
   static class ObserverWithState {
       LifecycleEventObserver mLifecycleObserver;
   
       ObserverWithState(LifecycleObserver observer, State initialState) {
           mLifecycleObserver = Lifecycling.lifecycleEventObserver(observer);
       }
   }
   
   @NonNul
   static LifecycleEventObserver lifecycleEventObserver(Object object) {
       return new ReflectiveGenericLifecycleObserver(object);
   }
   
   class ReflectiveGenericLifecycleObserver implements LifecycleEventObserver {
       private final CallbackInfo mInfo;
       
       ReflectiveGenericLifecycleObserver(Object wrapped) {
           mWrapped = wrapped;
           // 通过反射获取 method 信息
           mInfo = ClassesInfoCache.sInstance.getInfo(mWrapped.getClass());
       }
       
       @Override
       public void onStateChanged(LifecycleOwner source, Event event) {
           // 通过反射调用 method
           mInfo.invokeCallbacks(source, event, mWrapped);
       }
   }
   ```

   我们在 Observer 用注解修饰的方法，会被通过反射的方式获取，并保存下来，然后在生命周期发生改变的时候再找到对应 Event 的方法，通过反射来调用方法。

5. Lifecycle 的生命周期事件与状态的定义

   ```java
   public abstract class Lifecycle {
       // 生命周期事件
       public enum Event {
           ON_CREATE,
           ON_START,
           ON_RESUME,
           ON_PAUSE,
           ON_STOP,
           ON_DESTROY,
           ON_ANY
       }
       // 当前组件的生命周期状态
       public enum State {
           INITIALIZED,
           CREATED,
           STARTED,
           RESUMED;
           DESTROYED,
       }
   }
   ```

#### Lifecycle 实战应用

- 自动移除 Handler 的消息：LifecycleHandler
- 给 ViewHolder 添加 Lifecycle 的能力

#### LiveData 原理分析

1. 简单使用

   ```kotlin
       val liveString = MutableLiveData<String>()
       liveString.observe(this, Observer { Log.d("TAG", "called : s = $it") })
       liveString.postValue("LiveData")
   ```

2. LiveData.observe()

   ```java
       @MainThread
       public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
           assertMainThread("observe");
           if (owner.getLifecycle().getCurrentState() == DESTROYED) {
               // ignore 忽略 DESTROYED 状态
               return;
           }
           // 把 Observer 用 LifecycleBoundObserver 包装起来
           LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
           // 缓存
           ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
           // 如果已经 observe 过，并且两次的 owner 不同则报错
           if (existing != null && !existing.isAttachedTo(owner)) {
               throw new IllegalArgumentException("Cannot add the same observer"
                       + " with different lifecycles");
           }
           if (existing != null) {
               return;
           }
           // 绑定 owner
           owner.getLifecycle().addObserver(wrapper);
       }
   ```

3. LifecycleBoundObserver

   ```java
   class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {
       @NonNull
       final LifecycleOwner mOwner;
   
       LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<? super T> observer) {
           super(observer);
           mOwner = owner;
       }
   
       @Override
       boolean shouldBeActive() {
           // 判断 owner 当前的状态是否是至少 STARTED，认为是 active 状态
           return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
       }
   
       // 处理生命周期改变
       @Override
       public void onStateChanged(@NonNull LifecycleOwner source,
                                  @NonNull Lifecycle.Event event) {
           // 如果是 DESTROYED 就自动解除
           if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
               removeObserver(mObserver);
               return;
           }
           activeStateChanged(shouldBeActive());
       }
   
       @Override
       boolean isAttachedTo(LifecycleOwner owner) {
           return mOwner == owner;
       }
   
       @Override
       void detachObserver() {
           mOwner.getLifecycle().removeObserver(this);
       }
   }
   ```

4. ObserverWrapper

   ```java
       private abstract class ObserverWrapper {
           final Observer<? super T> mObserver;
           boolean mActive;
           int mLastVersion = START_VERSION;
   
           ObserverWrapper(Observer<? super T> observer) {
               mObserver = observer;
           }
   
           // 是否是 active 状态
           abstract boolean shouldBeActive();
   
           boolean isAttachedTo(LifecycleOwner owner) {
               return false;
           }
   
           void detachObserver() {
           }
   
           void activeStateChanged(boolean newActive) {
               if (newActive == mActive) {
                   return;
               }
               // immediately set active state, so we'd never dispatch anything to inactive
               // owner
               mActive = newActive;
               boolean wasInactive = LiveData.this.mActiveCount == 0;
               LiveData.this.mActiveCount += mActive ? 1 : -1;
               if (wasInactive && mActive) {
                   onActive();
               }
               if (LiveData.this.mActiveCount == 0 && !mActive) {
                   onInactive();
               }
               // 如果是 active 状态下，则发送数据更新通知
               if (mActive) {
                   dispatchingValue(this);
               }
           }
       }
   ```

5. 当我们调用 observe() 注册后，由于绑定了 owner，所以在 active 的情况下，LiveData 如果有数据，则 Observer 会立马接受到该数据修改的通知。

   流程：observe --> onStateChanged --> activeStateChanged

    --> dispatchingValue --> considerNotify --> onChanged

   

6. dispatchingValue 分析

   ```java
       // 分发事件逻辑的处理方法
   	void dispatchingValue(@Nullable ObserverWrapper initiator) {
           // 如果正在分发则直接返回
           if (mDispatchingValue) {
               // 标记分发失效
               mDispatchInvalidated = true;
               return;
           }
           // 标记分发开始
           mDispatchingValue = true;
           do {
               mDispatchInvalidated = false;
               // 生命周期改变调用的方法，initiator 不为 null
               if (initiator != null) {
                   considerNotify(initiator);
                   initiator = null;
               } else {
                   // postValue/setValue 方法调用，传递的 initiator 为 null
                   for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                           mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                       considerNotify(iterator.next().getValue());
                       if (mDispatchInvalidated) {
                           break;
                       }
                   }
               }
           } while (mDispatchInvalidated);
           // 标记分发结束
           mDispatchingValue = false;
       }
   ```

7. considerNotify 分析

   ```java
       // 确保了只将最新的数据分发给 active 状态下的 Observer
   	private void considerNotify(ObserverWrapper observer) {
           // 检查状态，确保不会分发给 inactive 的 observer
           if (!observer.mActive) {
               return;
           }
           if (!observer.shouldBeActive()) {
               observer.activeStateChanged(false);
               return;
           }
           // setValue 会增加 version，初始 version 为 -1
           if (observer.mLastVersion >= mVersion) {
               return;
           }
           observer.mLastVersion = mVersion;
           // 引入了版本管理来管理数据以确保发送的数据总是最新的
           observer.mObserver.onChanged((T) mData);
       }
   ```

8. ObserverWrapper 不为 null 的情况

   ```java
       class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {
           
           @Override
           public void onStateChanged(@NonNull LifecycleOwner source,
                   @NonNull Lifecycle.Event event) {
               activeStateChanged(shouldBeActive());
           }
   
       }
   
       void activeStateChanged(boolean newActive) {
           if (mActive) {
           	// 调用了 dispatchingValue 方法，并传入了 this，进入分发通知逻辑
               dispatchingValue(this);
           }
       }
   ```

9. ObserverWrapper 为 null 的情况：postValue /  setValue

   ```java
       protected void postValue(T value) {
           boolean postTask;
           synchronized (mDataLock) {
               postTask = mPendingData == NOT_SET;
               mPendingData = value;
           }
           if (!postTask) {
               return;
           }
           // 把操作 post 到主线程
           ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
       }
   
       private final Runnable mPostValueRunnable = new Runnable() {
           @SuppressWarnings("unchecked")
           @Override
           public void run() {
               Object newValue;
               synchronized (mDataLock) {
                   newValue = mPendingData;
                   mPendingData = NOT_SET;
               }
               // 最后调用的还是 setValue 方法
               setValue((T) newValue);
           }
       };
   
   	// setValue 必须是在主线程调用
       @MainThread
       protected void setValue(T value) {
           assertMainThread("setValue");
           mVersion++;
           mData = value;
           // 调用了 dispatchingValue 方法，并传入了 null，进入分发通知逻辑
           dispatchingValue(null);
       }
   ```