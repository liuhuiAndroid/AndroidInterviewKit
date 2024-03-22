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



#### 使用 `.mk` 文件的 ndk-build 和使用 `CMakeLists.txt` 的 CMake 有什么区别

1. **构建系统设计**:
   - **ndk-build**：基于 GNU Make，使用 Android.mk（通常简称为 `.mk` 文件）脚本来定义构建规则。ndk-build 系统是 Android 特有的，紧密集成于 Android 的原生开发工具链。
   - **CMake**：更现代且更广泛使用的跨平台构建系统。在 Android Studio 中的原生支持始于 2.2 版本。CMake 使用 CMakeLists.txt 文件来定义构建规则。
2. **跨平台能力**:
   - **ndk-build**：主要用于 Android 平台。
   - **CMake**：由于其跨平台性，广泛用于多个操作系统和项目。在 Android 环境中使用 CMake 可以更容易地将项目移植到其他平台。
3. **语法和易用性**:
   - **ndk-build**：使用 Makefile 语法，这对于不熟悉 Makefile 的开发者来说可能比较难以理解和使用。
   - **CMake**：具有更简洁和更容易理解的语法，尤其是对于项目依赖性和库链接的管理。
4. **社区和文档**:
   - **ndk-build**：虽然有着广泛的社区，但随着 Google 对 CMake 的支持，社区和文档逐渐转向 CMake。
   - **CMake**：拥有大量的社区支持和丰富的文档资源。
5. **灵活性和功能**:
   - **ndk-build**：对于专注于 Android 平台的项目来说足够使用。
   - **CMake**：由于其跨平台和现代化特点，提供了更多的灵活性和功能。
