#### Android KTX

Android KTX 是包含在 Android Jetpack 及其他 Android 库中的一组 Kotlin 扩展程序。

这套扩展插件能帮助开发者更为简洁、通顺和优雅地使用 Kotlin 语言开发 Android 程序。

通过 Android KTX 编写的代码更为简短而且读写起来也更为自然。

#### Core KTX

Core KTX 模块为属于 Android 框架的通用库提供扩展程序。

[Core KTX 文档](https://developer.android.google.cn/kotlin/ktx#core)

```groovy
    dependencies {
        implementation "androidx.core:core-ktx:1.2.0"
    }
```

###### 比较常见的用法：对比下使用 KTX 后的变化

1. 将字符串转换为 URI

   ```kotlin
   val myUriString = "https://www.baidu.com"
   // Kotlin
   val uri1 = Uri.parse(myUriString)
   // Kotlin with Android KTX
   val uri2 = myUriString.toUri()
   ```

2. 编辑偏好设置

   ```kotlin
   // Kotlin
   sharedPreferences
       .edit()  // create an Editor
       .putBoolean(key, value)
       .apply() // write to disk asynchronously
   // Kotlin with Android KTX
   sharedPreferences.edit {
       putBoolean(key, value)
   }
   ```

3. 在 onPreDraw 回调中执行其他任务

   ```kotlin
   // Kotlin
   view.viewTreeObserver.addOnPreDrawListener(
       object : ViewTreeObserver.OnPreDrawListener {
           override fun onPreDraw(): Boolean {
               view.viewTreeObserver.removeOnPreDrawListener(this)
               actionToBeTriggered()
               return true
           }
       }
   )
   // Kotlin with Android KTX
   view.doOnPreDraw { actionToBeTriggered() }
   ```

###### 主目录：

- [androidx.core.animation](https://developer.android.google.cn/reference/kotlin/androidx/core/animation/package-summary)

  简化了动画监听事件的写法

- [androidx.core.content](https://developer.android.google.cn/reference/kotlin/androidx/core/content/package-summary)

  包含了 contentvalues、context 的扩展函数和 sp 的简化

- [androidx.core.content.res](https://developer.android.google.cn/reference/kotlin/androidx/core/content/res/package-summary)

- [androidx.core.database](https://developer.android.google.cn/reference/kotlin/androidx/core/database/package-summary)

  包含了数据库游标的扩展函数和事务处理的简化

- [androidx.core.database.sqlite](https://developer.android.google.cn/reference/kotlin/androidx/core/database/sqlite/package-summary)

  

- [androidx.core.graphics](https://developer.android.google.cn/reference/kotlin/androidx/core/graphics/package-summary)

  这里面包含的内容很多，主要用于自定义 View，如：Path、Point、Canvas等等

- [androidx.core.graphics.drawable](https://developer.android.google.cn/reference/kotlin/androidx/core/graphics/drawable/package-summary)

  

- [androidx.core.location](https://developer.android.google.cn/reference/kotlin/androidx/core/location/package-summary)

  对 Location 对象的解构，可写成 ( lat , lon ) = Location()

- [androidx.core.net](https://developer.android.google.cn/reference/kotlin/androidx/core/net/package-summary)

  包含了 uri、file 和 String 相互转换

- [androidx.core.os](https://developer.android.google.cn/reference/kotlin/androidx/core/os/package-summary)

  有 bundle 和 Handler 的简化写法

- [androidx.core.text](https://developer.android.google.cn/reference/kotlin/androidx/core/text/package-summary)

  包含了 String、SpannableString 等等

- [androidx.core.transition](https://developer.android.google.cn/reference/kotlin/androidx/core/transition/package-summary)

  简化了对 transition 的监听操作

- [androidx.core.util](https://developer.android.google.cn/reference/kotlin/androidx/core/util/package-summary)

  这个里面包含了一系列工具类，如：LruCache、Pair、Size 和 Range 等等

- [androidx.core.view](https://developer.android.google.cn/reference/kotlin/androidx/core/view/package-summary)

  包含了三块：View、ViewGroup 和 Menu 三个类的多个方法

- [androidx.core.widget](https://developer.android.google.cn/reference/kotlin/androidx/core/widget/package-summary)

  这个组件中貌似专门为 EditText 的 addTextChangedListener 准备的

#### Collection KTX

Collection 扩展程序包含在 Android 的节省内存的集合库中使用的效用函数，包括 ArrayMap、LongParseArray、LruCache 等等。

```groovy
    dependencies {
        implementation "androidx.collection:collection-ktx:1.1.0"
    }
```

#### Fragment KTX

Fragment KTX 模块提供了一系列扩展程序来简化 Fragment API

```groovy
    dependencies {
        implementation "androidx.fragment:fragment-ktx:1.2.4"
    }
```

#### Activity KTX

```groovy
    dependencies {
        implementation "androidx.activity:activity-ktx:1.1.0"
    }
```
