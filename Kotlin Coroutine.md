#### Kotlin 协程实战训练

###### 一、Kotlin 协程是什么 Kotlin Cooperative Routine

- 协程是一种在程序中处理并发任务的方案；也是这种方案的一个组件
- 协程和线程属于一个层级的概念
  - 协程中不存在线程，也不存在并行——并行（Concurrency）不是并发（Parallelism）
- Kotlin 的协程不需要关心这些
  - 因为 Kotlin for Java 的协程并不属于广义的协程
- Kotlin 协程是一套由 Kotlin 官方提供的**线程框架**
- Kotlin 协程可以用看起来同步的代码写出实质上异步的操作，即非阻塞式挂起，不卡线程就是非阻塞式
- 关键：线程的自动切回

###### 二、协程的代码怎么写

- 用 launch() 来开启一段协程，一般需要指定 Dispatchers.Main

- 把要在后台工作的函数，写成 suspend函数。需要在内部调用其他 suspend 函数来真正切线程

- 按照一条线写下来，线程会自动切换

  ```kotlin
  GlobalScope.launch {
  	println("Coroutines Camp ${Thread.currentThread().name}")
  }
  
  GlobalScope.launch(Dispatchers.Main) {
  	withContext(Dispatchers.IO) {
          delay(1000)
          println("Coroutines Camp io ${Thread.currentThread().name}")
      }
      println("Coroutines Camp ${Thread.currentThread().name}")
  }
  ```

- 协程的额外天然优势：性能！很方便将耗时操作放在后台

###### 三、suspend

- 并不是用来切线程的
- 语法上是标记和提醒的作用：标记函数是挂起函数，是耗时的，需要在协程中调用，需要协程上下文
- 编译过程也起到一定的作用
- 挂起函数：以 suspend 修饰的函数
- 挂起函数只能在其他挂起函数或协程中调用
- 挂起函数调用时包含了协程挂起的语义
- 挂起函数返回时则包含了协程恢复的语义
- 将回调转写成挂起函数
  - 使用 suspendCoroutine 获取挂起函数的 Continuation
  - 回调成功的分支使用 Continuation.resume(value)
  - 回调失败则使用 Continuation.resumeWithException(e)

###### 四、Kotlin 的协程和线程

- Kotlin 的协程和线程分别是什么？
- Kotlin 的协程和线程哪个容易使用？——Kotlin 的协程
  - 协程需要和 Executor 比较，而不是线程
- Kotlin 的协程相比线程的优势和劣势？——好用但是不好上手
- Kotlin 的协程和 Handler 相比呢？——不是一个纬度
  - Handler 是一个只负责Android内切线程的特殊场景化的 Executor
  - 协程在 Android 端底层就用到了Handler，比直接使用 Handler 方便

###### 五、协程和 RxJava

- RxJava 可以把切线程变得很灵活，有各种操作符让我们对事件流处理的随心所欲
- 协程比 RxJava 更方便，更简单
- 协程性能并没有比 RxJava 强
- 选协程！RxJava 很多场景会慢慢被替代

###### 六、Retrofit 对协程的支持

- Retrofit 可以在接口方法加上 suspend 关键字变成挂起函数

  ```kotlin
  @GET("users/{user}/repos")
  suspend fun listRepos(@Path("user") user: String): List<Repo>
  
  GlobalScope.launch(Dispatchers.Main) {
      val one = async { api.listRepos("rengwuxian") }
      val two = async { api.listRepos("rengwuxian") }
      val same = one.await()[0].name == two.await()[0].name
      textView.text = "$same same"
  }
  ```

###### 七、CoroutineScope 作用域

- GlobalScope 顶级作用域 异常不向外部传播

- 协程泄露：本质上是线程泄露

  ```kotlin
  // 1.防止协程泄露
  private val scope = MainScope()
  scope.lanch(){
  }
  scope.cancel()
  
  // 2.子Scope
  scope.lanch(){
  	coroutineScope{
          launch{
              a()
          }
          launch{
              b()
          }
  	}
  	// 在a()，b()都执行结束才执行c()
  	c()
  }
  
  // lifecycle-runtime-ktx 添加了对 CoroutineScope 的支持
  lifecycleScope.launch(Dispatchers.Main) {
      delay(1000)
      println("Coroutines Camp ${Thread.currentThread().name}")
  }
  ```

###### 八、协程和 ViewModel、LiveData、Room等 Jetpack 组件结合使用

- LiveData

  ```kotlin
  class RengViewModel : ViewModel() {
    val repos = liveData {
      emit(loadUsers())
    }
  
    private suspend fun loadUsers(): List<Repo> {
      val retrofit = Retrofit.Builder()
        .baseUrl("https://api.github.com/")
        .addConverterFactory(GsonConverterFactory.create())
        .addCallAdapterFactory(RxJava3CallAdapterFactory.create())
        .build()
      val api = retrofit.create(Api::class.java)
      return api.listReposKt("rengwuxian")
    }
  }
  ```

###### 九、协程本质探秘

- 协程是怎么切线程的？
  - withContext(Dispatchers.IO){ } 本质上也是开了一个协程
- 协程为什么可以挂起却不卡主线程？
- 协程的 delay() 和 Thread.sleep()
  - delay() 性能更好？—— delay() 是挂起函数



#### 问题

1. Kotlin 协程挂起是怎么实现的

   Kotlin 协程的挂起是通过一种轻量级的线程切换机制实现的。



###### 官方协程框架的运用

- kotlinx.coroutines

  - 官方协程框架，基于标准库实现的特性封装
  - <https://github.com/Kotlin/kotlinx.coroutines>

- 协程框架的引入

  ```
  // 标准库
  implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
  // 协程基础库
  implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.5'
  // 协程Android库，提供Android UI调度器
  implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.5'
  ```



###### 回调转协程的完整写法

- 不支持取消的写法

  ```
  suspend fun <T> Call<T>.awaitNonCancellable(): T = suspendCoroutine{
      ...
  }
  ```

- 支持取消的写法

  ```
  suspend fun <T> Call<T>.await(): T = suspendCancellableCoroutine{
      continuation ->
      continuation.invokeOnCancellation{
          cancel()
      }
      ...
  }
  ```

- Retrofit 回调转协程，官方也有同样实现

  ```
  suspend fun <T> Call<T>.await(): T = suspendCancellableCoroutine{
      continuation ->
      continuation.invokeOnCancellation{
          cancel()
      }
      enqueue(object:Callback<T>{
          override fun onFailure(call: Call<T>,t: Throwable){
              continuation.resumeWithException(t)
          }
          override fun onResponse(call: Call<T>,response: Response<T>){
              response.takeIf{ it.isSuccessful }?.body()?.also{ continuation.resume(it) }?			continuation.resumeWithException(HttpException(response))
          }
      })
  }
  ```

- Handler

  ```
  suspend fun <T> Handler.run(block: ()-> T) = suspendCoroutine<T>{
      continuation ->
      post{
          try{
              continuation.resume(block())
          }catch(e: Exception){
              continuation.resumeWithException(e)
          }
      }
  }
  
  suspend fun <T> Handler.runDelay(delay: Long, block: ()-> T) = suspendCancellableCoroutine<T>{
      continuation ->
      val message = Message.obtain(this){
          try{
              continuation.resume(block())
          }catch(e: Exception){
              continuation.resumeWithException(e)
          }
      }.also{
          it.obj = continuation
      }
      continuation.invokeOnCancellation{
      	removeCallbacksAndMessage(continuation)
      }
      sendMessageDelayed(message,delay)
  }
  
  suspend fun main(){
      Looper.prepareMainLooper()
      GlobalScope.launch{
          val handler = Handler(Looper.getMainLooper())
          val result = handler.run{ "Hello" }
          val delayedResult = handler.runDelay(1000){ "World" }
  	    Looper.getMainLooper().quit()
      }
  }
  ```

###### 
