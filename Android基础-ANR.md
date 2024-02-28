什么是ANR？

Application Not responding是Android系统中的一种错误状态，指的是当应用程序在主线程上执行太长时间而无法响应用户输入时发生的情况。
Android系统会定期检测所有正在运行的应用程序的主线程状态。如果发现某个应用程序的主线程在一段时间内无法响应用户输入，就会触发ANR。

ANR，全称Application Not Responding（应用程序无响应），是指在Android应用程序中，当主线程在一定时间内未能响应用户输入或无法执行其他关键任务时，系统会触发ANR错误，导致应用无法正常运行。这可能是因为主线程被长时间阻塞，例如在执行耗时操作、网络请求或者数据库查询时未使用异步方式，从而导致用户体验下降。为了避免ANR，开发者需要将耗时任务移到子线程或使用异步机制，确保主线程保持响应性，提升应用的稳定性和用户体验。



ANR是如何被触发的？

ANR是一套监控Android应用响应是否及时的机制，可以把发生ANR比作是引爆炸弹，那么整个流程包含三部分组成：

1. 设置定时炸弹
   
   通常都是在system_server进程内启动倒计时，在规定时间内，如果App进程没有处理完所要完成的任务并解除炸弹，则会发生ANR。

2. 拆除定时炸弹
   
   App进程，在规定的时间完成任务，并及时向system_server进程报告完成，请求解除定时炸弹，则不会发生ANR。

3. 定时炸弹爆炸
   
   system_server进程，保存当前状态、抓取快照(traces)，并且发生ANR。



ANR的触发场景有哪些？

1. Service Timeout:如前台服务在20s内未执行完成；

2. BroadcastQueue Timeout：如前台广播在10s内未执行完成;

3. ContentProvider Timeout：内容提供者,在publish过超时10s;

4. InputDispatching Timeout: 输入事件分发超时5s，包括按键和触摸事件。



ANR是由谁来计时并且弹出提示的？
