Explain the differences between Parcelable and Serializable in Android. When would you choose one over the other for object serialization and why?

解释 Android 中 Parcelable 和 Serializable 之间的区别。 您何时会选择其中之一来进行对象序列化，为什么？

Parcelable and Serializable are two mechanisms(/ ˈmekəˌnɪzəm /) in Android for object serialization, but they differ in terms of performance and use cases.
Parcelable 和 Serialized 是 Android 中用于对象序列化的两种机制，但它们在性能和用例方面有所不同。

Serializable is a more generic Java interface used for object serialization. It is easy to implement but can be less efficient (/ ɪˈfɪʃ(ə)nt /) due to the reflection-based approach it uses. It serializes objects into a stream of bytes, making it suitable for scenarios(/ səˈnærioʊz /) where objects need to be stored or transmitted.
Serialized 是一个更通用的 Java 接口，用于对象序列化。 它很容易实现，但由于它使用基于反射的方法，效率可能较低。 它将对象序列化为字节流，适合需要存储或传输对象的场景。

On the other hand, Parcelable is an Android-specific interface optimized for performance. It requires explicit implementation but is faster than Serializable because it operates at the object level and doesn't rely on reflection. Parcelable is recommended for Android-specific scenarios like passing data between components within the same app, especially in performance-critical situations such as ListView or RecyclerView adapters.
另一方面，Parcelable 是针对性能进行优化的 Android 专用接口。 它需要显式实现，但比 Serialized 更快，因为它在对象级别运行并且不依赖反射。 Parcelable 建议用于特定于 Android 的场景，例如在同一应用程序内的组件之间传递数据，特别是在性能关键的情况下，例如 ListView 或 RecyclerView 适配器。

In summary, if the goal is to serialize objects for general Java purposes, Serializable is appropriate. However, for Android-specific use cases, especially when performance is crucial(/ ˈkruːʃ(ə)l /), using Parcelable is often the preferred choice.
总之，如果目标是为了一般 Java 目的而序列化对象，则 Serialized 是合适的。 然而，对于 Android 特定的用例，尤其是当性能至关重要时，使用 Parcelable 通常是首选。



序列化：将实例的状态转换为可以存储或传输的形式的过程

Parcelable

- 对象自行实现出入口方法，避免对类结构的反射
- 二进制流存储在连续内存中，占用空间更小
- 牺牲易用性（可以通过Parcellize弥补），换取极致的性能

Serializable

- 用反射获取类的结构和属性信息，过程中会产生中间信息
- 有缓存结构，在解析相同类型的情况下，能复用缓存
- 性能在可接受的范围内，易用性比较好



Parcelable 为什么速度优于 Serializable？

- 在大多数场景下，Parcelable 确实存在性能优势，Serializable 的性能缺陷来自，需要通过反射构建 ObjectStreamClass 类型的描述信息

- 构建 ObjectStreamClass 类型的描述信息的过程，有缓存机制，能够大量复用缓存的使用场景下，Serializable 反而会存在性能优势

- Parcelable 原本在易用性上存在短板，但 Kotlin 的 Parcelize 很好的弥补了 Parcelable 的易用性