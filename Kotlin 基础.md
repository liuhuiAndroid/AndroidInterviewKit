#### Kotlin 的 Any 和 Java 的 Object 区别

`Any` 不允许表示 `null` 值

#### Kotlin 中的 by lazy 和 lateinit 区别

`by lazy` 和 `lateinit` 都是 Kotlin 中用于延迟初始化的机制

**初始化时机：**

- `by lazy`: 这是一个属性代理，它允许你在首次访问属性时执行初始化。具体而言，它在第一次访问该属性时调用 lambda 表达式，并将结果保存起来，以后的访问直接返回保存的结果。
- `lateinit`: 这是一个修饰符，用于延迟初始化属性，但它要求在使用该属性之前显式初始化。这意味着你需要确保在属性被使用之前进行初始化，否则会抛出 `UninitializedPropertyAccessException` 异常。

**可变性**：`by lazy` 更适合于只需要初始化一次且是 `val` 类型的属性，而 `lateinit` 更适合于可变的属性，需要在后续的代码中进行初始化。

#### Kotlin let、with、run、apply、also函数的区别

let	

fun T.let(block: (T) -> R): R = block(this)	

it指代当前对象



with

fun with(receiver: T, block: T.() -> R): R = receiver.block()

this指代当前对象或者省略



run

fun T.run(block: T.() -> R): R = block()

this指代当前对象或者省略



apply

fun T.apply(block: T.() -> Unit): T

this指代当前对象或者省略



also

fun T.also(block: (T) -> Unit): T

it指代当前对象

#### Kotlin 单例如何编写

在 Kotlin 中，你可以使用对象声明（Object Declaration）来创建单例。对象声明在首次访问时被懒初始化，并且在整个应用程序生命周期内只有一个实例。

可以添加一个初始化方法，接收参数

#### Java反射和Kotlin反射有什么区别

#### Java和Kotlin相互调用有什么需要注意的

1. 空安全
2. 伴生对象
