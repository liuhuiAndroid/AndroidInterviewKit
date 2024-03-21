介绍下MVVM架构模式

核心概念：MVVM是一种软件架构模式，主要用于分离应用程序的用户界面UI逻辑和业务逻辑。

三个主要组件：Model模型、View视图、ViewModel视图模型。

Model模型：负责处理应用程序的数据和业务逻辑。

View视图：表示应用程序的用户界面，负责展示数据并与用户进行交互。视图通常包括用户界面元素、布局和样式。

ViewModel：位于视图和模型之间，负责处理视图和模型之间的通信。通常不直接引用视图，而是通过数据绑定来更新视图。
优势：降低耦合、提高代码可测试性和可维护性。

MVVM的核心思想是实现数据驱动的界面，通过将业务逻辑从视图中分离出来，使得代码更加模块化、可维护性更强。在Android开发中，Jetpack组件中的ViewModel和Data Binding库通常与MVVM一起使用，以实现更优雅的界面开发。

Data Binding、Jetpack ViewModel、Jetpack LiveData、Jetpack Compose、Kotlin Flow等官方工具库和官方实践，都是为了解决Android MVC时期存在的种种问题，这些工具，大多符合MVVM的思想。



#### MVVM 需要使用 DataBinding 吗？



#### MVI

单向数据流



#### 问题：介绍一下目前开发的项目结构