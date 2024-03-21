1. 动态库和静态库区别？

2. Android 为什么都是使用动态库，很少使用静态库？

   1. 代码共享

      动态库允许多个应用程序共享同一份库代码。

   2. 更新和维护

      动态库更易于更新和维护。

   3. 系统和平台要求

      Android 系统和平台的设计也鼓励使用动态库。

3. JNI C 如何调用 java 方法

   1. 首先，你需要获取你要调用的 Java 方法所在类的引用。使用 `FindClass` 函数可以实现这一点
   2. 接着，使用 `GetMethodID` 或 `GetStaticMethodID` 函数获取方法的 ID。需要提供方法的名称和签名。
   3. 如果你要调用的是实例方法（非静态方法），需要先创建该类的一个实例。使用 `NewObject` 函数可以实现。
   4. 使用 `CallVoidMethod`、`CallObjectMethod`、`CallIntMethod` 等函数，根据 Java 方法的返回类型来调用方法。对于静态方法，使用 `CallStaticVoidMethod`、`CallStaticObjectMethod` 等类似函数。
