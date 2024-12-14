### 如何在Android下使用FFmpeg

#### Java与C之间的相互调用

1. JNIEnv：Java的本地化环境
2. JavaVM：Java虚拟机，可以获取JNIEnv。一个JavaVM很多线程，一个线程对应一个JNIEnv。
3. 线程：JNIEnv和线程是一一对应的

#### Java调用C/C++方法

1. 在Java层定义native关键字函数

2. 方法一：在C/C++层创建Java_packagename_classname_methodname函数

3. 方法二：RegisterNative手动映射，推荐

   当我们库被加载的时候，会回调JNI_OnLoad函数，在这个函数将native方法注册进去

   当我们库被卸载的时候，会毁掉JNI_OnUnload函数

   ```cpp
   #define JNI_CLASS_PATH "com/example/sec/MainActivity"
   JNIEXPORT jstring JNICALL
   my_test_register(JNIEnv *env, jobject thiz) {
       return env->NewStringUTF("test");
   }
   
   static JNINativeMethod g_methods[] = {
           {
                   "_test", // Java层的方法名称
                   "()Ljava/lang/String;", // Signature
                   (void*) my_test_register
           }
   };
   
   jint JNI_OnLoad(JavaVM *vm, void *reserved) {
       JNIEnv *env = NULL;
       vm->GetEnv((void **) &env, JNI_VERSION_1_6);
     	// 查找关联的Java类
       jclass clazz = env->FindClass(JNI_CLASS_PATH);
     	// 注册方法
       env->RegisterNatives(clazz, g_methods, sizeof(g_methods) / sizeof(g_methods[0]));
       return JNI_VERSION_1_6;
   }
   ```

##### 什么是Signature？

- Signature是Java与C/C++相互调用时，表示函数参数的描述符
- 输入参数放在()内，输出参数放在()外
- 多个参数之间顺序存放，且用;分割
- Java对象参数 “L包路径/类名”
- Java数组 “[”

#### C/C++调Java方法

1. FindClass
2. GetMethodID/GetFieldID
3. NewObject
4. Call<TYPE> Method/[G/S]et<Type>Fileld

```c++
#define JNI_STUDENT_PATH "com/example/sec/Student"
JNIEXPORT jstring JNICALL
my_test_register(JNIEnv *env, jobject thiz) {
    // step 1 找Java层的类
    jclass clazz = env->FindClass(JNI_STUDENT_PATH);
    // step 2 获取Java类中的方法或属性
    jmethodID method_init_id = env->GetMethodID(clazz, "<init>", "()V");
    jmethodID method_set_id = env->GetMethodID(clazz, "setAge", "(I)V");
    jmethodID method_get_id = env->GetMethodID(clazz, "getAge", "()I");
    // step 3 创建Java对象
    jobject obj = env->NewObject(clazz, method_init_id);
    // step 4 执行方法
    env->CallVoidMethod(obj, method_set_id, 18);
    int age = env->CallIntMethod(obj, method_get_id);
    char tmp[] = {0,0,};
    sprintf(tmp, "%d", age);
    std::string hello = "test, age=";
    hello.append(tmp);
    return env->NewStringUTF(hello.c_str());
}
```

### Android下FFmpeg的编译

参考：https://juejin.cn/post/7100938254974877726/

fuck...可能是因为m1芯片，按照很多方案怎么都编译不过，参照上面的文章可以正常编译

```shell
#AS下载的ndk地址：/Users/sec/Documents/github-ndk/android-ndk-r24
#android-ndk-r24-darwin.dmg
#build_ffmpeg_for_android.sh
chmod +x build_ffmpeg_for_android.sh
./build_ffmpeg_for_android.sh
#产出：/Users/sec/tools/outputs/ffmpeg/aarch64
```

```shell
#!/bin/bash
NDK_HOME=/Users/sec/Documents/github-ndk/NDK
# NDK的路径 (在bash.rc、zshrc中设置的路径，其中的NDK_HOME需要配置环境变量，如果你不配，在这改成你自己的也可以。)
NDK_ROOT=/Users/sec/Documents/github-ndk/NDK
# NDK_ROOT=你的NDK路径
TOOLCHAIN_PREFIX=$NDK_HOME/toolchains/llvm/prebuilt/darwin-x86_64
echo "<<<<<< FFMPEG 交叉编译 <<<<<<"
echo "<<<<<< 基于当前系统NDK地址： $NDK_HOME <<<<<<"
# 编译的相关参数
# 目标平台的CPU指令类型 ARMv8
CPU=armv8-a
# 架构类型 ： ARM
ARCH=arm64
# 操作系统
OS=android
# 平台
PLATFORM=aarch64-linux-android
OPTIMIZE_CFLAGS="-march=$CPU"
# 指定输出路径
PREFIX=~/tools/outputs/ffmpeg/aarch64
# SYSROOT
SYSROOT=$TOOLCHAIN_PREFIX/sysroot
# 交叉编译工具链
CROSS_PREFIX=$TOOLCHAIN_PREFIX/bin/llvm-
# Android交叉编译工具链的位置
ANDROID_CROSS_PREFIX=$TOOLCHAIN_PREFIX/bin/${PLATFORM}29
echo ">>>>>> FFMPEG 开始编译 >>>>>>"
./configure \
--prefix=$PREFIX \
--enable-shared \
--enable-gpl \
--enable-neon \
--enable-hwaccels \
--enable-postproc \
--enable-jni \
--enable-small \
--enable-mediacodec \
--enable-decoder=h264_mediacodec \
--enable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffplay \
--disable-avdevice \
--disable-debug \
--disable-static \
--disable-doc \
--disable-symver \
--cross-prefix=$CROSS_PREFIX \
--target-os=$OS \
--arch=$ARCH \
--cpu=$CPU \
--cc=${ANDROID_CROSS_PREFIX}-clang \
--cxx=${ANDROID_CROSS_PREFIX}-clang++ \
--enable-cross-compile \
--sysroot=$SYSROOT \
--extra-cflags="-Os -fPIC $OPTIMIZE_CFLAGS" \
--extra-ldflags="$ADDI_LDFLAGS" \
make clean
# 创建目标路径,如果不存在的话，最终产物存储在Prefix对应路径之下。
mkdir -p $PREFIX
sudo make -j8
sudo make install
echo "<<<<<< 编译完成，产物存储在:$PREFIX <<<<<<"
```

#### Android下如何使用FFmpeg

参考ffmpeg-so-test：https://blog.csdn.net/Kennethdroid/article/details/106956601
