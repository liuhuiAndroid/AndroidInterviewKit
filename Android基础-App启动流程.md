当用户点击应用图标启动应用时，系统首先会检查应用是否已经在后台运行。如果是，则系统会唤醒应用的进程，否则，将启动一个新的进程。随后，系统会初始化应用的Application类，这是整个应用生命周期的入口。在这个阶段，可以进行一些全局的初始化操作。接下来，系统会启动应用的主Activity。Activity是应用中负责用户界面和用户交互的主要组件。在Activity的生命周期方法中（如onCreate()、onStart()、onResume()），可以进行界面的准备工作和数据加载。一旦主Activity准备就绪，系统就会将应用的界面显示在屏幕上，用户便可以开始与应用进行交互。整个过程包括了应用的进程管理、Application的初始化以及主Activity的启动和界面展示。

#### 当我们点击应用图标时，系统都做了什么？

冷启动：App进程 -> Application -> FirstActivity；包括 App 进程创建、Application 实例创建、Activity 实例创建、Activity 的显示

热启动：将已启动过的Activity重新启动，或切到前台；需理解 Activity 任务栈

温启动：Application -> FirstActivity

要点：

1. 冷、热、温启动的基本流程
2. Zygote 和 App 进程启动
3. AMS 的作用
4. Activity 任务栈和生命周期
5. 如何启动未注册的 Activity
6. 启动速度优化的一些思路

#### 冷启动流程：

1. Launcher 请求 startActivity 告知 AMS
2. AMS 告知 Zygote 进程，请求启动 App 进程
3. Zygote 进程通过 fork 的方式，启动 App 进程，并注册句柄给 AMS
4. AMS 启动 Activity，管理生命周期，调度四大组件

系统的启动：

Kernel 启动，运行第一个用户级进程就是 Init 进程

- 用户空间的第一个进程，进程号是1

- 贯穿操作系统的生命周期

- 作为守护进程，防止子进程成为僵尸进程
  
  Init 进程可以解析 init.rc 配置文件，根据配置文件内容启动系统服务，包含 Zygote 进程

Zygote 进程的作用

- 孵化 APP 进程
- 为进程提供系统资源和虚拟机
- 启动 system server 进程，也就是 AMS 所在的进程

什么是 Context？

- 给系统组件提供访问系统服务、系统资源的能力

AMS 和冷启动

ClientTransaction

- 统一管理生命周期跨进程消息

- 降低 AMS 和 App 之间的代码耦合

- 减少通信次数 

进程内启动 Activity

#### 如何启动未注册的 Activity？



ATM 为 Activity 分配 Task
