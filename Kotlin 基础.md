#### Kotlin 的 Any 和 Java 的 Object 区别



#### Kotlin 中的 by lazy 和 lateinit 区别

`by lazy` 和 `lateinit` 都是 Kotlin 中用于延迟初始化的机制

`by lazy` 更适合于只需要初始化一次且是 `val` 类型的属性，而 `lateinit` 更适合于可变的属性，需要在后续的代码中进行初始化。



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

