1. 动态库和静态库区别？

2. Android 为什么都是使用动态库，很少使用静态库？
   
   - 代码共享
     
     动态库允许多个应用程序共享同一份库代码。
   
   - 更新和维护
     
     动态库更易于更新和维护。
   
   - 系统和平台要求
     
     Android 系统和平台的设计也鼓励使用动态库。

3. JNI - Java 如何调用 C 方法（重点）
   
   1. 放置头文件
   
   2. 配置CMake或ndk-build：你需要在`CMakeLists.txt`（如果使用CMake）或`Android.mk`（如果使用ndk-build）中指定头文件的路径。
   
   3. 创建JNI桥接代码：在`app/src/main/cpp`目录下创建JNI桥接代码。这些代码使用提供的头文件来调用C++函数，并将这些函数暴露给Java/Kotlin层。

4. JNI - C 如何调用 java 方法
   
   1. 首先，你需要获取你要调用的 Java 方法所在类的引用。使用 `FindClass` 函数可以实现这一点
   2. 接着，使用 `GetMethodID` 或 `GetStaticMethodID` 函数获取方法的 ID。需要提供方法的名称和签名。
   3. 如果你要调用的是实例方法（非静态方法），需要先创建该类的一个实例。使用 `NewObject` 函数可以实现。
   4. 使用 `CallVoidMethod`、`CallObjectMethod`、`CallIntMethod` 等函数，根据 Java 方法的返回类型来调用方法。对于静态方法，使用 `CallStaticVoidMethod`、`CallStaticObjectMethod` 等类似函数。
