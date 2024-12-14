#### 学习 WorkManager - 满足您的后台调度需求

你的应用程序不会总处于前台，但你仍需确保那些重要的后台任务将会执行。例如：下载更新、与服务器同步等等。Google 为我们提供了这些现有的 API 来处理后台操作，然而当这些 API 使用不当时，可能会造成这样的结果。那么为了帮助用户省电，Android 在最近几个版本中引入了一系列的电量优化功能，作为一个开发者，您需要了解电量优化的限制，并且妥善处理和使用这些 API，以确保你的后台任务在不同的 API 等级下都可顺利执行。为了避免后台任务无法执行的情况，您可以需要写冗长的代码来支持在不同设备上的用户群。那么为了解决这样的痛点，我们向您推出 WorkManager 库。

WorkManager 为后台任务提供了一套统一的解决方案。它可以根据 Android 电量优化以及设备的 API 等级，选择合适的方式执行。WorkManager 属于 Android Jetpack 的一部分，它向后兼容至 API 14，并且对无论集成 Google Play 服务与否的设备都予以支持。在这一基础上，WorkManager 还为您提供了更多的便利功能。

使用 WorkManager 管理任务主要有以下两大特点：第一允许延迟，第二保证执行。可延迟的工作是指那些即使不立即执行，也仍然有必要运行的任务。例如，向服务器发送分析数据就是可延迟的任务，但是发送即时消息就不满足这样的条件。保证执行的意思是，即时在应用程序被关闭，或者设备重启的情况下，任务也会照常执行，图片备份就是一个十分典型的 WorkManager 用例。这个任务可以推迟，但同时又必须保证执行。同时，你也可以把 WorkManager 和其他的 API 配合使用。例如当服务器端有新的数据需要在客户端更新时，你可以发送一个 firebase cloud messaging 消息，然后把实际但同步任务交给 WorkManager 来完成。值得注意的是，我们谈到的后台操作，不是也不包括后台线程。WorkManager 不能用来代替 Kotlin coroutines、线程池以及 RxJava 等 API，但可以和他们配合使用。WorkManager 也不适用于需要在特定时间触发的任务，你可以通过使用 AlarmManager 来触发特定时间的任务。如若用户希望立即执行一项任务，您也可以开启一个前台服务。

现在，让我们来了解一下 WorkManager 的使用方法。假设我想用 WorkManager 上传一张图片。首先，需要在 Worker 中定义这项任务，具体执行的任务内容写在 doWork 方法中。然后使用 WorkRequest 类创建一个任务请求。WorkRequest 类的主要作用是配置任务的运行时间和方式，所以在创建 WorkRequest 时，你可以添加约束条件，指定输入，并且选择单次或者周期性的方式来执行任务。在这里上传图片是单次任务，所以我们选择 OneTimeWorkRequest，然后添加只有在网络时上传的约束条件。最后，使用 WorkManager 中的 Data 类，把图片的 URI 作为输入值传递到这个任务请求中。WorkRequest 创建好之后，你需要通过使用 WorkManager.enqueue 把任务加入队列中，这样，我们就完成了一个使用 WorkManager 调度管理任务的简单实例。即使用户关闭应用或者设备重启，只要在满足网络的条件下，系统都会确保在将来的某个时间点上传图片。接下来，我们想在任务结束时更新 UI 界面，你可以通过使用 getWorkInfoByIdLiveData 方法来获取一个 WorkInfo 的 LiveData，WorkInfo 包含任务状态的所有信息，如当前状态以及任务的输出。LiveData 对 WorkInfo  的包裹，使任务信息成为了一个可观察数据。

在 doWork 方法中，我们可以返回一个输出值，然后在 UI 里加入观察 LiveData 的代码，在任务结束之后，UI 界面就会显示输出。那么当任务被 enqueue 之后，WorkManager 又是如何管理调度的呢？在前面提到过，WorkManager 一直向后兼容到 API 14，而且不需要 Google Play 服务也可以运行。为了实现这一目的，WorkManager 在运行时会选择 JobScheduler，或者 AlarmManager 和 BroadcastReceiver 的组合方式来执行任务。为了保证任务执行，所有被加入队列的任务信息，都会保存在 WorkManager 的一个数据库中。所以，即使任务被中断，也可以恢复执行。在默认情况下，Worker 会使用 Executor 在非 UI 线程执行任务，如果您想通过其它方式管理线程，可以选择 RxWorker 或者 CoroutineWorker，如果需要更多的控制权限，您也可以继承 ListenableWorker 来创建自定义的 Worker 类进行线程管理。

WorkManager 另一个特别出彩的地方在于对任务链的管理，您可以利用 WorkManager 把几个相互依赖的任务，组合到一条任务链中。例如我现在有几张图片，我想先给它们添加一个滤镜，然后进行压缩，最后批量上传。假设您已经创建好所有的任务请求了，也加入了合适的约束条件。我们将这些任务按照执行顺序加入队列，加滤镜的任务可以并行执行，所以我们把这些任务请求作为一个 list 同时加入到队列中，WorkManager 负责任务链之间输入和输出的传递。在这个例子中，加入滤镜的 WorkRequest 输出就是上传图片的 WorkRequest 的输入。除此之外，WorkManager 还为您提供了更多的功能。比如说，您可以用它指定需要阶段性重复的任务，或者创建唯一的任务，标记你的任务，以便查询或取消所有统一标记的任务，以及对已经创建的任务设置重试策略。

最后终结一下：WorkManager 可以帮助您：在指明运行条件的情况下，管理和调度单次或者需要定期循环的后台任务，创建任务链，指定输出和输入，支持按顺序执行或者需要并行的多个任务，观察任务状态，从而更新 UI 界面，以及自定义 WorkManager 的线程管理策略。除此以外，WorkManager 支持向后兼容，遵循系统电量优化的最佳实践，并且确保任务执行，所以我们推荐，WorkManager 成为您处理 Android 的后台工作的首选方案。

[Android Jetpack WorkManager](<https://www.bilibili.com/video/av56276889>)

[跟上脚步，进入后台执行新时代](https://mp.weixin.qq.com/s/lvUJEL7PAZFAzNjrscGEuw)

[[译\] 从Service到WorkManager](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5b04d064f265da0b80711759%23heading-3)

[《即学即用Android Jetpack - WorkManger》](https://www.jianshu.com/p/68e720b8a939)

#### WorkManager 介绍

WorkManager 可以自动维护后台任务的执行时机，执行顺序，执行状态。

WorkManager 任务构建可以解决任务顺序问题

```
WorkContinuation left, right;
left = workManager.beginWith(A).then(B);
right = workManager.beginWith(C).then(D);
WorkContinuation.combine(Arrays.asList(left,right)).then(E).Enqueue();
```

WorkManager 任务状态通知

WorkManager 任务控制

```java
cancelWorkById(UUID)      // 通过 ID 取消单个任务
cancelAllWorkByTag(String)// 通过 Tag 取消所有任务
cancelUniqueWork(String)  // 通过名字取消唯一任务
```

WorkManager 核心类

- Work

  任务的执行者，是一个抽象类，需要继承它实现要执行的任务

- WorkRequest

  指定让哪个 Work 执行任务，指定执行的环境，执行的顺序等。要使用它的子类 OneTimeWorkRequest 或 PeriodicWorkRequest

- WorkManager

  管理任务请求和任务队列，发起的 WorkRequest 会进入它的任务队列

- WorkStatus

  包含有任务的状态和任务的信息，以 LiveData 的形式提供给观察者

WorkManager 简单使用

```java
//1.编写一个UploadFileWorker.class 继承自 Worker,创建任务
//2.创建输入参数
Data inputData = new Data.Builder()
     .putString("key", "value")
     .putBoolean("key1", false)
     .putStringArray("key2", new String[]{})
     .build();

// 编写约束条件
Constraints constraints = new Constraints();
//设备存储空间充足的时候 才能执行 ,>15%
constraints.setRequiresStorageNotLow(true);
//必须在执行的网络条件下才能好执行,不计流量 ,wifi
constraints.setRequiredNetworkType(NetworkType.UNMETERED);
//设备的充电量充足的才能执行 >15%
constraints.setRequiresBatteryNotLow(true);
//只有设备在充电的情况下 才能允许执行
constraints.setRequiresCharging(true);
//只有设备在空闲的情况下才能被执行 比如息屏，cpu利用率不高
constraints.setRequiresDeviceIdle(true);
//workmanager利用contentObserver监控传递进来的这个uri对应的内容是否发生变化,当且仅当它发生变化了，我们的任务才会被触发执行，以下三个api是关联的
constraints.setContentUriTriggers(null);
//设置从content变化到被执行中间的延迟时间，如果在这期间。content发生了变化，延迟时间会被重新计算
constraints.setTriggerContentUpdateDelay(0);
//设置从content变化到被执行中间的最大延迟时间
constraints.setTriggerMaxContentDelay(0);

//3.创建workrequest
OneTimeWorkRequest request = new OneTimeWorkRequest
      .Builder(UploadFileWorker.class)
      .setInputData(inputData)
      //其它许许多多条件约束
      .setConstraints(constraints)
      //设置一个拦截器，在任务执行之前可以做一次拦截，去修改入参的数据然后返回新的数据交由worker使用
      .setInputMerger(null)
      //当一个任务被调度失败后，所要采取的重试策略，可以通过BackoffPolicy来执行具体的策略，每隔10秒重试一次
      .setBackoffCriteria(BackoffPolicy.EXPONENTIAL, 10, TimeUnit.SECONDS)
      //任务被调度执行的延迟时间
      .setInitialDelay(10, TimeUnit.SECONDS)
      //设置该任务执行失败，尝试执行的最大次数
      .setInitialRunAttemptCount(2)
      //设置这个任务开始执行的时间
      .setPeriodStartTime(0, TimeUnit.SECONDS)
      //指定该任务被调度的时间
      .setScheduleRequestedAt(0, TimeUnit.SECONDS)
      //当一个任务执行状态编程finish时，又没有后续的观察者来消费这个结果，难么workamnager会在内存中保留一段时间的该任务的结果。超过这个时间，这个结果就会被存储到数据库中下次想要查询该任务的结果时，会触发workmanager的数据库查询操作，可以通过uuid来查询任务的状态
      .keepResultsForAtLeast(10, TimeUnit.SECONDS)
      .build();

// 4.加入队列
List<OneTimeWorkRequest> workRequests = new ArrayList<>();
workRequests.add(request);
WorkContinuation continuation = WorkManager.getInstance().beginWith(workRequests).enqueue();

// 5.观察执行状态 及结果      
continuation.getWorkInfosLiveData().observe(this, new Observer<List<WorkInfo>>() {
    @Override
    public void onChanged(List<WorkInfo> workInfos) {
        //监听任务执行的结果
        for (WorkInfo workInfo : workInfos) {
            WorkInfo.State state = workInfo.getState();
        } 
    }
 });
```

WorkManager 使用场景 TODO

#### WorkManager 原理分析

WorkManager 架构图 TODO 

-  JobScheduler

  [参考：安卓电量优化之JobScheduler使用介绍](https://www.cnblogs.com/leipDao/p/8268918.html)

  JobScheduler 是安卓5.0版本推出的API，允许开发者在符合某些条件时创建执行在后台的任务。与AlarmManager 不同的是这个执行时间是不确定的。JobScheduler API 允许同时执行多个任务。

  JobSchedule 的宗旨就是把一些不是特别紧急的任务放到更合适的时机批量处理。这样做有两个好处：避免频繁的唤醒硬件模块，造成不必要的电量消耗以及避免在不合适的时间(例如低电量情况下、弱网络或者移动网络情况下的)执行过多的任务消耗电量。

  ```java
  public class TestJobService extends JobService {
  
      // 任务开始
      @Override
      public boolean onStartJob(JobParameters params) {
          // 返回值是 false,系统假设这个方法返回时任务已经执行完毕
          // 返回值是 true,那么系统假定这个任务正要被执行，当任务执行完毕时开发者需要自己调用jobFinished 来通知系统。
          return false;
      }
  
      // 取消任务
      @Override
      public boolean onStopJob(JobParameters params) {
          return false;
      }
  }
  ```

  需要到 AndroidManifest.xml 中添加一个service节点让你的应用拥有绑定和使用这个 JobService 的权限

  ```xml
  <service
      android:name=".TestJobService"
      android:permission="android.permission.BIND_JOB_SERVICE" >
  </service>
  ```

  ```java
  // 创建JobScheduler对象
  JobScheduler tm =(JobScheduler) getSystemService(Context.JOB_SCHEDULER_SERVICE);
  
  ComponentName mServiceComponent = new ComponentName(this, TestJobService.class);
  // 第一个参数是你要运行的任务的标识符，第二个是这个 Service 组件的类名
  JobInfo.Builder builder = new JobInfo.Builder(kJobId++, mServiceComponent);
  ```

- AlarmManager

  系统提供的一个定时任务管理器。

  Timer 类也是安卓开发中常用到的一个定时器类，但是Timer只能在手机运行的时候使用，也就是说在手机休眠（黑屏）的情况下是没有作用的，但是 AlarmManager 是系统的一个类，它能在手机休眠的情况下也能够运行。

  

