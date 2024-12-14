### No.1 两小时用 Compose 写个微信界面

### No.2 用 Compose 写出一个简单界面

声明式 UI：不手动更新 UI

独立于平台（独立于最新版Android），Android Studio 预览功能

Compose 会跨到 iOS 吗？短期内希望渺茫。可支持桌面版、Web 版。多平台（multi-platform）

Coil

FrameLayout -> Box()

ScrollView -> Modifier.verticalScroll()

RelativeLayout -> Box()

ConstraintLayout -> 依然可以用

RecyclerView -> LazyColumn()

ScrollView -> Modifier.verticalScroll()

ViewPager -> Pager()

--------------

Modifier.clip(CircleShape).size(128.dp)

Modifier.background(Color.Green).padding(8.dp).background(Color.Red).padding(8.dp)

match_parent = Modifier.fillMaxWidth / Height / Size()

通用的设置用Modifier，专项的设置用函数参数。

点击监听器用Modifier

---------

compile -> runtime -> ui -> animation -> foundation -> material

包依赖三条原则

1. 依赖 material(3) 就够了；可能跳过 material 依赖 foundation
2. 如果你需要 ui-tooling，需要单独依赖：提供预览功能
3. 如果你需要 material-icons-extended，需要单独依赖

### No.3 状态订阅和自动更新

##### 自定义 Composable

什么是 Composable，带 @Composable 注解的函数

自定义 Composable 的首字母大写

自定义 Composable 内部只能调用一个别的 Composable

自定义 Composable = 自定义View + XML布局

##### MutableState 和 mutableStateOf()

```kotlin
val name = mutableStateOf("Hello World")
setContext {
  Text(name.value)
}
lifecycleScope.launch {
  delay(3000)
  name.value = "I do need a car."
}

// by 关键字：左边的对象由右边代理；自动订阅
// import getValue & setValue 扩展函数
var name by mutableStateOf("Hello World")
setContext {
  Text(name)
}
```

##### 重组作用域和 remember()

Recompose Scope

```kotlin
// remember：缓存作用，防止 Recompose 时重复初始化，建议全部包上；变量不在 Compose 函数内不需要 remember
// 用在可能进入 Recompose 的对象上，无法判断
val name by remember { mutableStateOf("Hello World") }
// 带参数版本的 remember
// remember(key...)，key 一样用缓存
```

##### 「无状态」、状态提升和单向数据流

Stateless

状态：控件的属性，比如 TextView 的 text

State Hoisting：状态提升，状态放入函数参数；尽量不往上提

Single Source of Truth

```kotlin
val name by remember { mutableStateOf("Hello World") }
TextField(name, { newValue ->
  name = newValue
})
```

Unidirectional Data Flow - 单向数据流

##### 更新 List 竟然不会触发刷新？

mutableStateOf()

mutableStateListOf()

mutableStateMapOf()

##### 重组的性能风险和智能优化、@Stable

Recompose 的执行过程

Recompose Scope 重组范围

Compose 自动更新 -> 更新范围过大，超过需求 -> 智能优化跳过没必要的更新

可靠的类：使用 Structual Equality 结构性相等； Kotlin ==； Java equals

不可靠的类直接进入 Recompose

给不可靠的类加 @Stable 注解，告诉编译器插件此类型人工保证可靠

@Stable 注解的类不使用 data class，使用原始的 equals 方法保证可靠

@Stable 的稳定：

1. 现在相等就永远相等：不要重写 equals 函数；不能使用 data class
2. 当 var 修饰的公开属性改变的时候，通知到用这个属性的 Composition：var 使用 by mutableStateOf 代理
3. 公开属性也必须是可靠类型或者稳定类型

```Kotlin
// 实战中这样写
class User(name: String){
  var name by mutableStateOf(name)
}
```

##### derivedStateOf() —和 remember() 有什么区别？

derivedStateOf：convert one or multiple state objects into another state

https://developer.android.com/jetpack/compose/side-effects?hl=zh-cn

1. 监听状态变化从而自动刷新有两种写法
   1. 带参数的 remember()
   2. 不带参数的 remember() + derivedStateOf()
2. 对于状态对象来说，比如 mutableStateListOf() 带参数的 remember() 不可使用，无法监听内部状态的改变，只能使用 derivedStateOf()
3. 对于函数参数里的字符串，由于监听链条会被掐断，不能用 derivedStateOf()，而只能用带参数的 remember() 

带参数的 remember() 作用：Recompose 的时候判断参数是否同一个对象

derivedStateOf() 作用：适用于监听状态对象

```Kotlin
var name by remember { mutableStateOf("AAA") }
val processedName by remember { derivedStateOf { name.uppercase() } }
val processedName = remember(name) { name.uppercase() }
Text(processedName)

var names by remember { mutableStateListOf("AAA", "BBB") }
// 放入自定义 Composable 内需要传入 State 状态对象而不能是 String，不实用
val processedNames by remember { derivedStateOf { names.map { it.uppercase() } } }
// 无效，remember 记录的 names 对象不变，自己跟自己比较
// 原因：String 的改变只有一种方式，重新赋值；List 的改变除了赋值，主要是内部元素的改变
val processedNames = remember(names) { names.map { it.uppercase() } }
Column{
  for(processedName in processedNames){
    Text(processedName)
  }
}
```

##### CompositionLocal - 是状态但又不全是

CompositionLocal：具有穿透函数功能的局部变量，不需要显示传递的函数参数；无形传参

```kotlin
CompositionLocalProvider(LocalName provides "seckill") {
	User()
}

@Composable
fun User() {
  Text(LocalName.current)
}

// 缩小刷新范围，经常改变时使用
val LocalName = compositionLocalOf { error("no default value.") }
// 消除记录，但是能全量刷新，计算量大，不经常改变时使用
val LocalName = staticCompositionLocalOf { error("no default value.") }

CompositionLocalProvider(LocalBackground provides Color.Yellow) {
	TextWithBackground()
}

@Composable
fun TextWithBackground(){
  Surface(color = LocalBackground.current){
    Text("有背景的文字")
  }
}
LocalBackground.current
LocalContext.current
```

### No.4 动画

##### 状态转移型动画 animateXxxAsState()

```kotlin
// var size by remember { mutableStateOf(48.dp) }
var big by mutableStateOf(false)
setContent{
  var size by animateDpAsState(if(big) 96.dp else 48.dp)
	Box(
    Modifier.size(size)
      .background(Color.Green)
      .clickable{
        big = !big
      }
	)
}
```

##### 流程定制型动画 Animatable

```kotlin
var big by mutableStateOf(false)
setContent{
  val size = remember(big) { if(big) 96.dp else 48.dp }
  val anim = remember { Animatable(size, Dp.VectorConverter) }
  LaunchedEffect(big){
    anim.snapTo(if(big) 192.dp else 0.dp)
    anim.animateTo(size)
  }
	Box(
    Modifier.size(anim.value)
      .background(Color.Green)
      .clickable{
        big = !big
      }
	)
}
```

##### AnimationSpec 之 TweenSpec

补间动画

Easing：缓动

FastOutSlowInEasing 快出慢进，先加速再减速；从A状态变成B状态

LinearOutSlowInEasing 	适合元素入场动画；全程减速；

FastOutLinearInEasing 	 适合元素出场动画；全程加速；

LinearEasing 全程匀速运动

```kotlin
var big by mutableStateOf(false)
setContent{
  val size = remember(big) { if(big) 96.dp else 48.dp }
  val offset = remember(big) { if(big) (-48).dp else 148.dp }
  val anim = remember { Animatable(size, Dp.VectorConverter) }
  val offsetAnim = remember { Animatable(offset, Dp.VectorConverter) }
  LaunchedEffect(big){
    anim.snapTo(if(big) 192.dp else 0.dp)
    anim.animateTo(size, spring(Spring.DampingRatioMediumBouncy))
    anim.animateTo(size, TweenSpec(easing = LinearEasing))
    offsetAnim.animateTo(offset, TweenSpec(easing = FastOutLinearInEasing))
    offsetAnim.animateTo(offset, tween())
  }
	Box(
    Modifier.size(anim.value)
    	.offset(offsetAnim.value, offsetAnim.value)
      .background(Color.Green)
      .clickable{
        big = !big
      }
	)
}
```

##### AnimationSpec 之 SnapSpec

```kotlin
// 和 anim.snapTo 作用一样，唯一区别可以延迟
offsetAnim.animateTo(offset, SnapSpec(3000))
offsetAnim.animateTo(offset, snap(3000))
```

##### AnimationSpec 之 KeyframesSpec

分段式 TweenSpec

```kotlin
anim.animateTo(size, KeyframesSpec(KeyframesSpec.KeyframesSpecConfig<Dp>().apply{
}))
// =
anim.animateTo(size, keyframes {
	durationMillis = 450
  delayMillis = 500
  // 150ms的时候需要运行到144dp的位置
  144dp at 150 with FastOutLinearInEasing
  200dp at 300 with FastOutSlowInEasing
})
```

##### AnimationSpec 之 SpringSpec

无法设置时长

```kotlin
// dampingRatio 阻尼比
// stiffness 刚度
// visiblityThreshold 阈值
anim.animateTo(size, spring())
anim.animateTo(size, spring(Spring.DampingRatioMediumBouncy))
anim.animateTo(size, spring(0.1f, Spring.StiffnessHigh), 200.dp)
```

##### AnimationSpec 之 RepeatableSpec

```kotlin
// iterations
// animation
// repeatMode
// initialStartOffset 时间偏移，延迟或快进
anim.animateTo(size, repeatable(3, tween(), RepeatMode.Reverse, StartOffset(500, StartOffsetType.Delay)))
```

##### AnimationSpec 之 InfiniteRepeatableSpec

```kotlin
// 无限循环，和 RepeatableSpec 本质上没有区别
anim.animateTo(size, infiniteRepeatable(tween(), RepeatMode.Reverse, StartOffset(500, StartOffsetType.Delay)))
```

##### AnimationSpec 之其他 Spec

FloatTweenSpec 针对 Float 类型

FloatSpringSpec 针对 Float 类型

```kotlin
var big by mutableStateOf(false)
setContent{
  val float = remember(big) { if(big) (-48).dp else 148.dp }
  val floatAnim = remember { Animatable(float) }
  LaunchedEffect(big){
    floatAnim.animateTo(200f, tween())
    floatAnim.animateTo(200f, spring())
  }
}
```

VectorizedAnimationSpec 系列是之前的 AnimationSpec 底层使用

DecayAnimationSpec

##### 消散型动画 animateDecay()

惯性衰减，初始速度需要精确指定

```kotlin
// initialVelocity 初始速度，像素
// animationSpec
// block
val decay = remember { ntialDecay<Dp>() }
anim.animateDecay(100.dp, decay)

// frictionMultiplier 摩擦力系数
// absVelocityThreshold 速度阈值绝对值，速度绝对值小于阈值动画直接结束
// 指数型衰减，不会针对像素密度修正，面向Dp；使用范围广
// 由于没有用到 density（用户可以在系统调节） 所以没有带remember版本；但还是需要自己包一层remember避免对象重复创建
exponentialDecay<Dp>()
// 用来实现惯性滑动
splineBasedDecay<Int>(LocalDensity.current)
// 建议直接用带 remember 版本，不同屏幕像素手机滑动距离不一样;根据像素密度修正，Dp依然修正；所以只能使用像素不能使用Dp，面向像素；
rememberSplineBasedDecay()
```

##### block 参数：监听每一帧

animateTo 和 animateDecay 都有 block 参数

```kotlin
val decay = remember { ntialDecay<Dp>() }
anim.animateDecay(100.dp, decay) {
  // 每一帧刷新都会执行，相当于对动画的监听
  // 可以让 A 动画更新实时值传递给 B 动画
  println("hhh")
}
```

##### 打断施法：动画的边界限制、结束和取消

Animatable

```kotlin
// animateDecay animateTo snapTo 可以相互打断；新动画打断旧动画
// anim.stop() 主动打断，不能和要被掐断的动画同一个协程
// anim.updateBounds() 设置边界；动画边界触达；正常结束不会抛异常
var big by mutableStateOf(false)
setContent{
  BoxWithConstraints{
    LaunchedEffect(Unit){
      delay(1000)
      try{
        val animResult = anim.animateDecay(2000.dp, decay)
        if(animResult.endReason == AnimationEndReason.BoundReached){
          // 反弹
          animResult.animateDecay(-animResult.endState.velocity, decay)
        }
      } catch (e: CancellationException){
        println("被打断了")
      }
    }
    LaunchedEffect(Unit){
      delay(1500)
      anim.animateDecay(-1000.dp, decay)
    }
    anim.updateBounds(0.dp, upperBound = maxWidth - 100.dp)
    // 可以分来设置边届，动画分开停止
    animY.updateBounds(upperBound = maxHeight - 100.dp)
    
    // 手动计算回弹值交给 padding
    val paddingX = remember(anim.value){
      val usedValue = anim.value
      while (usedValue >= (maxWidth -100.dp) * 2) {
        usedValue -= (maxWidth -100.dp) * 2
      }
      if(usedValue < maxWidth - 100.dp){
        usedValue
      } else {
        (maxWidth -100.dp) * 2 - usedValue
      }
    }
  }
}
```

##### Transition：多属性的状态切换

转场动画，状态切换的动画

```kotlin
val big by remember { mutableStateOf(false) }
// val bigTransition = updateTransition(big, "big")
val bigTransition = updateTransition(
  MutableTransitionState(big).apply{ targetState = true }, "big")
// Transition 会统一管理，label在预览页面可以看
val size by bigTransitiom.animateDp({ 
	when {
    false isTransitioningTo true -> spring() 
    else tween()
  }
}, label = "size"){ if(big) 96.dp else 48.dp }
val corner by bigTransitiom.animateDp(label = "corner"){ if(big) 0.dp else 18.dp }
Box(
	Modifier
  	.size(size)
  	.clip(RoundedCornerShape(corner))
  	.background(Color.Green)
  	.clickable{
      big = !big
    }
)
```

##### Transition 延伸：AnimatedVisibility()

扩展 Transition

```kotlin
var shown by remember { mutableStateOf(true) }
// AnimatedVisibility(shown, enter = fadeIn(initialAlpha = 0.5f)) {
//   TransitionSquare()
// }
AnimatedVisibility(shown, enter = slideIn{ IntOffset(-it.width, -it.height) }) {
  TransitionSquare()
}
AnimatedVisibility(shown, enter = expandIn(
  tween(5000),
	expandFrom = Alignment.TopStart,
  initialSize = { IntSize(it.width / 2, it.height / 2) }
  clip = false
)) {
  TransitionSquare()
}
AnimatedVisibility(shown, enter = scaleIn(transformOrigin = TransformOrigin())) {
  TransitionSquare()
}
// 0.5f 废弃
AnimatedVisibility(shown, enter = fadeIn(initialAlpha = 0.3f) + fadeIn(initialAlpha = 0.5f)) {
  TransitionSquare()
}
// + 把两个动画进行合并
AnimatedVisibility(shown, enter = scaleIn() + expandIn()]) {
  TransitionSquare()
}
Button(onClick = { shown = !shown }){
  Text("切换")
}

val big by remember { mutableStateOf(false) }
val bigTransition = updateTransition(big, "big")
bigTransition.AnimatedVisibility({ it }){
  
}
```

##### Transition 延伸：Crossfade()

让两个交替出现的组件渐变

```kotlin
var shown by remember { mutableStateOf(true) }
Crossfade(shown, animationSpec = tween()){
  if (it) {
    A()
  } else{
    Box(Modifier.size(24.dp).background(Color.Red))
  }
}
```

##### Transition 延伸：AnimatedContent()

面向多个组件出场和入场定制

```kotlin
var shown by remember { mutableStateOf(true) }
AnimatedContent(shown, transitionSpec = {
  // ContentTransform(fadeIn(), fadeOut())
  // fadeIn() with fadeOut()
  fadeIn(tween(3000)) with fadeOut(tween(3000, 3000))
}) {
  when (it) {
    1 -> Box()
    2 -> Box()
    3 -> Box()
    else -> Box()
  }
}
```

### No.5 Modifier 全解析

##### modifier：Modifier = Modifier 的含义

Modifier.Companion 伴生对象

modifier：Modifier = Modifier 默认值，开发者可以选填

Modifier 建议作为第一个有默认值的参数

##### then()、CombinedModifier 和 Modifier.Element

then Modifier 融合

CombinedModifier 融合器，链式装载多个 Modifier

##### Modifier.composed() 和 ComposedModifier

使用 Modifier.composed 函数来使用 ComposedModifier

```
// 好处，用在 Modifier 有状态的场景
// 把同一个 Modifier 作用在多处
// 作用：创建带状态的 Modifier，用在自定义 Modifier
fun Modifier.paddingJumpModifier() = composed{
	val padding by remember { mutableStateOf(8.dp) }
	Modifier.padding(padding)
			.clickable{ padding = 0.dp }
}
```

##### LayoutModifier 和 Modifier.layout()

使用 Modifier.layout 函数来使用 LayoutModifier，修改组件的属性和位置；测量和布局

```
Text("Hello", Modifier.layout{ measurable, constraints ->
	val paddingPx = 10.dp.roundToPx()
	val placeable = measurable.measure(constraints.copy(
		maxWidth = constraints.maxWidth - paddingPx * 2
		maxHeight = constraints.maxHeight - paddingPx * 2
	))
	layout(placeable.width + paddingPx * 2, placeable.height + paddingPx * 2){
		placeable.placeRelative(paddingPx, paddingPx)
	}
})
```

##### LayoutModifier 的工作原理和对布局的精细影响

Modifier 对顺序敏感

Modifier.size(40.dp).size(80.dp).background(Color.Blue)	40dp

requiredSize 不听左边的话

Modifier.size(40.dp).requiredSize(80.dp).background(Color.Blue)	80dp

##### DrawModifier 的工作原理和对绘制的精细影响

管理绘制

```kotlin
Box(Modifier.size(40.dp).drawWithContent{
  drawContent()
})
```

##### PointerInputModifier 的功能介绍和原理简析

用于定制触摸算法

```kotlin
Modifier.combindClickable{
  
}

// 触摸反馈，触摸范围右边最近的Modifier，左边覆盖右边
Modifier.pointInput(Unit){
  detectTapGestures(onTap = {}, onDoubleTap = {}, onLongPress = {}, onPress = {})
  forEachGesture{
    awaitPointEventScope {
      val down = awaitFirstDown()
    }
  }
}
```

##### ParentDataModifier 的作用

给子组件设置属性，用于父组件辅助测量

```kotlin
Modifier.layoutId("big")

@Composable
fun CustomLayout(modifier: Modifier = Modifier, content: @Composable () -> Unit){
  Layout(content, modifier) { measurable, constraints ->
    measurable.forEach {
      when (it.layoutId) {
        "big" -> it.measure(constraints.xxx)
        "small" -> it.measure(constraints.xxx)
        else -> it.measure(constraints)
      }
    }
    layout(100, 100){
      ...
    }
  }
}
```

##### ParentDataModifier 的写法

自定义 Composable 用到子组件特殊信息才需要用到 ParentDataModifier

```kotlin
@Composable
fun CustomLayout(modifier: Modifier = Modifier, content: @Composable CustomLayout2Scope.() -> Unit){
  Layout(content, modifier) { measurable, constraints ->
    measurable.forEach {
      val data = it.parentData as? Layout2Data
      val big = data?.big
      val weight = data?.weight
    }
    layout(100, 100){
      ...
    }
  }
}

class Layout2Data(var weight: Float = 0f, var big: Boolean = false)

fun Modifier.weightData(weight: Float) = then(object : ParentDataModifier {
  // parentData 参数为了适配多条数据
  override fun Density.modifyParentData(parentData: Any?): Any? {
    return if (parentData == null) Layout2Data(weight)
    else (parentData as Layout2Data).apply { this.weight = weight }
  }
})

fun Modifier.bigData(big: Boolean) = then(object : ParentDataModifier {
  // parentData 参数为了适配多条数据
  override fun Density.modifyParentData(parentData: Any?): Any? {
    return (parentData as Layout2Data) ?: Layout2Data().also { it.big = big }
  }
})

@LayoutScopeMarker // 限制不在内部无法写
@Immutable // 减少不必要的重组
interface CustomLayout2Scope  {
  
}

CusteomLayout{
  // 忽略了2f
  Text("1", Modifier.weightData(1f).bigData(true))
  Text("2")
}
```

##### ParentDataModifier 的原理

##### SemanticsModifier 的作用、写法和原理

提供 semantics tree 语义树；无障碍功能

```
Box(Modifier.width(40.dp).height(40.dp).semantics{
	contentDescription = "XXX"
})
```

##### addBefore-() 和 addAfter-() 的区别以及最新版代码的调整

##### OnRemeasuredModifier 的作用、写法和原理

OnRemeasuredModifier 相当于 onMeasure()，重新测量会触发

```kotlin
Text("Hello", Modifier.onSizeChanged{  }
```

##### OnPlacedModifier 的作用、写法和原理

类似 OnRemeasuredModifier，每次摆放的时候回调；类似 onMeasure 和 onLayout

```
Modifier.onPlaced{ layoutCoordinates ->
	val pos = layoutCoordinates.positionInParent()
}.layout { measurable, constraints ->
	val placeable = measurable.measure()
	layout()
}
```

##### LookaheadOnPlacedModifier 的作用、写法和原理

```
LookaheadLayout(content = , measurePilicy = )
```

##### OnGloballyPositionedModifier 的作用、写法和原理

全局定位；OnGloballyPositionedModifier 管的是左边

```
Modifier.onGloballyPositioned{ }.size(10.dp).onGloballyPositioned{ }
```

##### ModifierLocal (-Provider - -Consumer)

```kotlin
Modifier.modifierLocalProvider {
  modifierLocalOf { "空" }
}{
  "Hello"
}.modifierLocalConsumer {
  sharedKey.current
}
```

### No.6 附带效应（副作用）和 Kotlin 协程

##### 副作用（附带效应）和 SideEffect()

##### DisposableEffect()

可以丢弃的 SideEffect

```kotlin
// 参数改变之后才会重新执行
DisposableEffect(Unit){
	println("进去界面啦。。。")
	onDispose{
		println("离开界面啦。。。")
	}
}
```

##### 协程：LaunchedEffect()

特殊的 DisposableEffect，和 DisposableEffect 底层实现一致

```kotlin
LaunchedEffect(Unit){
  delay(3000)
  ...
}
```

##### rememberUpdatedState()

```kotlin
var rememberedWelcome by remember { mutableStateOf(welcome) }
rememberedWelcome = welcome
// 替代上面代码
val rememberedWelcome by rememberUpdatedState(welcome) 
```

##### 协程：rememberCoroutineScope()

```kotlin
// 在需要 Composable 组件外部调用的时候派上用场
val scope = rememberCoroutineScope()
val coroutine = remember { scope.launch{} }
```

##### 从 produceState() 说起：协程（和其他）状态向 Compose 状态的转换

```kotlin
val geoManager: GeoManager = TODO()
val positionData: LiveData<Point> = TODO()

var position by remember { mutableStateOf(Point(0, 0)) }
val state = produceState(initialValue = ) {
  val callback = object: ??? { newPosition ->
    position = newPosition
  }
  geoManager.register(callback)
  onDispose{
    geoManager.unregister(callback)
  }
}
Text(state)
```

```kotlin
val geoManager: GeoManager = TODO()
val positionData: LiveData<Point> = TODO()
val positionState: StateFlow<Point> = TODO()

setContent{
  CustomTheme{
    var position by remember { mutableStateOf(Point(0, 0)) }
    DisposableEffect(Unit) {
      val callback = object: ??? { newPosition ->
        position = newPosition
      }
      geoManager.register(callback)
      onDispose{
        geoManager.unregister(callback)
      }
    }
    DisposableEffect(Unit) {
      val observer = Observer<Point> { newPosition ->
        position = newPosition
      }
      positionData.observe(this@Mainactivity, observer)
      onDispose{
        geoManager.unregister(callback)
      }
    }
    // 需要添加 androidx.compose.runtime:runtime-livedata:xxx 依赖
    // 订阅回调，帮你取消订阅
    positionData.observeAsState()
    
    LaunchedEffect(Unit){
      positionState.collect { newPosition ->
        position = newPosition
      }
    }
  }
}
```

```kotlin
var position by remember { mutableStateOf(Point(0, 0)) }
LaunchedEffect(Unit){
  positionState.collect { newPosition ->
  	position = newPosition
  }
}
// 等价于
val produceState = produceState(Point(0, 0)) {
  positionState.collect { newPosition ->
  	position = newPosition
  }
}
```

##### snapshotFlow()：把 Compose 的 State 转换成协程 Flow

```kotlin
var name by remember { mutableStateOf("Hello") }
val flow = snapshotFlow{ name }
LaunchedEffect(Unit){
  flow.collect { name ->
  	println(name)
  }
}
```

### No.7 Compose里的【自定义View】布局、绘制、触摸

##### 自定义绘制

```kotlin
Text("Hello", Modifier.drawBehind{
	drawRect(Color.Yellow)
})
Box(Modifier.size(80.dp).drawBehind{
  
})
// 自由决定绘制顺序
Text("Hello", Modifier.drawWithContent{
	drawRect(Color.Yellow)
  // drawContent 上面xxx 被覆盖
  drawContent()
  // drawContent 下面xxx 盖住原先内容
  drawLine(Color.Red, Offset(0f, size.height/2), 
           Offset(size.width, size.height/2), 2.dp.toPx())
})

val image = ImageBitmap.imageResource(R.drawable.avatar)
val paint by remember { mutableStateOf(Paint()) }
// 两种旋转 1. graphicsLayer 使用 RenderNode 旋转坐标系
Canvas(Modifier.size(80.dp).graphicsLayer{ 
  rotationX = 45f
  rotationY = 45f
}){
  rotate(30f){
    drawImage(image, dstSize = IntSize(size.width.roundToInt(), size.height.roundToInt()))
  }
  // rotateRad() 弧度
  drawInCanvas{
    it.drawImageRect(image, dstSize = IntSize(size.width.roundToInt(), size.height.roundToInt()), paint = paint)
  }
}

// 旋转动画
val rotationAnimatable = remember{
  Animatable(0f)
}
LaunchedEffect(Unit){
  rotationAnimatable.animateTo(360f, infiniteRepeatable(tween(2000)))
}
// 多维旋转使用 Camera
val camera by remember { mutableStateOf(Camera()) }// .apply{
//   value.rotateX(45f)
//   value.rotateY(45f)
// }
val paint by remember { mutableStateOf(Paint()) }
Canvas(Modifier.size(80.dp)){
  // 下沉到Compose的Canvas
  drawInCanvas{
    // 使用更下层的接口
    it.translate(size.width/2, size.height/2)
    it.rotate(-45f)
    camera.save()
    camera.ratateX(rotationAnimatable.value)
    // 下沉到原生Canvas
    camera.applyToCanvas(it.nativeCanvas)
    camera.restore()
    it.rotate(45f)
    it.translate(-size.width/2, -size.height/2)
    it.drawImage(image, dstSize = IntSize(size.width.roundToInt(), size.height.roundToInt()), paint = paint)
  }
}
```

##### 自定义布局和 Layout()

```kotlin
// 对自己进行测量
Modifier.layout{ measurable, constraints ->
	
}

@Preview
@Composable
fun CustomLayoutPreview{
  CustomLayout{
    Box(Modifier.size(80.dp)).background(Color.Red)
    Box(Modifier.size(80.dp)).background(Color.Yellow)
    Box(Modifier.size(80.dp)).background(Color.Blue)
  }
}

// 自定义布局，简单版自定义Column
@Composable
fun CustomLayout(modifier: Modifier = Modifier, content: @Composable () -> Unit) {
  // Layout 等价于 ViewGroup
  Layout(content) {measurables, constraints ->
    var width = 0
    var height = 0
    val placeables = measurables.map{ measurable ->
      measurable.measure(constraints).also{ placeable ->
        width = max(width, placeable.width)
				height += max(height, placeable.height)
      }
    }
		layout(width, height){
      var totalHeight = 0
      placeables.forEach {
        it.placeRelative(0, totalHeight)
        totalHeight += it.height
      }
    }
  }
}
```

##### 自定义布局：SubcomposeLayout()

LazyColumn：动态控制显示数量和细节等，避免性能消耗。Scaffold；

缺点：对界面性能有负面影响，能不加入Subcompose就不用。也不用因噎废食

```kotlin
BoxWithConstraints() {
  // 可以拿到测量过程父组件对子组件的尺寸限制再决定如何组合；Box拿不到
  constraints
  if(minWidth >= 360.dp){
    Text("Hello", Modifier.align(Alignment.Center))
  } else {
    Text("World", Modifier.align(Alignment.Center))
  }
}

// Subway，sub次一级
SubcomposeLayout{ constraints ->
  // 组合
	val measureable = subcompose(1){
    Text("Hello")
  }[0]
  // 测量
  val placeable = measureable.measure(constraints)
  // 布局 
  layout(placeable.width, placeable.height){
    placeable.placeRelative(0, 0)
  }
}
```

##### 自定义布局：LookaheadLayout()

设计用来做过度动画

```kotlin
// 前瞻布局过程
LookaheadLayout({
  Column{
    Text("Hello")
    Text("World")
  }
}){ measurables, constraints ->
  // 不能多次测量
  val placeables = measurables.map { it.measure(constraints) }
  val width = placeables.maxOf { it.width }
  val height = placeables.maxOf { it.height }
  layout(width, height){
    placeables.forEach { it.placeRelative(0, 0) }
  }
}

Layout({
  Text("Hello", Modifier.layout{ measurable, constraints ->
    val placeables = measurables.map { it.measure(constraints) }
    layout(placeables.width, placeables.height){
      placeables.placeRelative(0, 0)
    }
  })
}){ measurables, constraints ->
  // 不能多次测量
  val placeables = measurables.map { it.measure(constraints) }
  val width = placeables.maxOf { it.width }
  val height = placeables.maxOf { it.height }
  layout(width, height){
    placeables.forEach { it.placeRelative(0, 0) }
  }
}

@Composable
private fun CustomLookaheadLayout(){
  var textHeight by remember { mutableStateOf(100.dp) }
  var textHeightAnim by animateDpAsState(textHeight)
  Column{
    LookaheadLayout({
       Text("Hello", Modifier.height(textHeightAnim).clickable{
         textHeight = if (textHeight == 100.dp) 200.dp else 100.dp
       })
    }){ measurables, constraints ->
      val placeables = measurables.map { it.measure(constraints) }
      val width = placeables.maxOf { it.width }
      val height = placeables.maxOf { it.height }
      layout(width, height){
        placeables.forEach { it.placeRelative(0, 0) }
      }
    }
    Text("World")
  }
}
```

```kotlin
@Composable
private fun CustomLookaheadLayout(){
  var isTextHeight200dp by remember { mutableStateOf(false) }
  var textHeightPx by remember { mutableStateOf(0) }
  var textHeightAnim by animateDpAsState(textHeightPx)
  Column{
    SimpleLookaheadLayout{
       Text("Hello", 
            Modifier
            .intermediateLayout{ measurable, constraints, lookaheadSize -> 
              textHeightPx = lookaheadSize.height
              val placeable = measurable.measure(
                Constraints.fixed(lookaheadSize.width, textHeightAnim)
              )
              layout(width, height){
                placeables.forEach { it.placeRelative(0, 0) }
              }
            }
            .then(if (isTextHeight200dp) Modifier.height(200dp) else Modifier)
            .clickable{
         			isTextHeight200dp = !isTextHeight200dp
       			})
    }
    Text("World")
  }
}

@Composable
private fun SimpleLookaheadLayout(content: @Composable LookaheadLayoutScope.() -> Unit){
  LookaheadLayout(content){ measurables, constraints ->
    val placeables = measurables.map { it.measure(constraints) }
    val width = placeables.maxOf { it.width }
    val height = placeables.maxOf { it.height }
    layout(width, height){
      placeables.forEach { it.placeRelative(0, 0) }
    }
  }
}
```

##### 自定义触摸和一维滑动监测

```kotlin
Modifier.clickable{}
Modifier.combinedClickable{}
Modifier.pointerInput(Unit){
  // 太底层，用得少
  awaitEachGesture{
    val event = awaitPointerEvent()
  }
  detectTapGestures {
    
  }
}

val interactionSource = remember { MutableInteractionSource() }
val offsetX by remember { mutableStateOf(0f) }
Text("Hello world.", Modifier
     .offset{ IntOffset(offsetX.roundToInt, 0) }
     // 监测一维滑动
     .draggable(rememberDraggableState{ delta ->
        println("$delta px.")
        offsetX += it                                 
     }, Orientation.Horizontal))
val isDragged = interactionSource.collectIsDraggedAsState()
Text(if (isDragged) "拖动中" else "静止")


// 增加滑动功能
Modifier.verticalScroll()
Modifier.HorizontalScrollable()
// 滑动监测
Modifier.scrollable(rememberScrollableState{
  println("scroll $it")
}, Orientation.Horizontal, overscrollEffect = ScrollableDefaults.overscrollEffect())

SwipToDismiss()
```

##### 嵌套滑动和 nestedScroll()

```kotlin
Modifier.nestedScroll()

Scaffold{
  LargeTopAppBar(title="", scrollBehavior = TopAppBarDefaults.exitUntilCollapsedScrollBehavior())
}

var offsetY by remember{ mutableStateOf(0f) }
val dispatcher = remember{ NestedScrollDispatcher() }
Column(
	Modifier.offset{ IntOffset(0, offsetY.roundToInt()) }
  .draggable(rememberDraggableState{
    dispatcher.dispatchPreScroll(Offset(0f, it), NestedScrollSource.Drag)
    offsetY += it
    dispatcher.dispatchPostScroll()
  }, Orientation.Vertical)
  .nestedScroll()
)
```

##### 自定义触摸：二维滑动监测

```kotlin
Modifier.pointInput(Unit){
  detectDragGestures{ change, dragAmout ->
    
  }
}

val draggableState = rememberDraggableState{}
Modifier.draggable(draggableState, Orientation.Horizontal)
LaunchedEffect(Unit){
  draggableState.drag{
    dragBy(100f)
  }
}
```

##### 自定义触摸：多指手势

```kotlin
Modifier.pointInput(Unit){
  detectTransformGestures{ centroid, pan, zoom, rotation ->
    
  }
}
```

##### 自定义触摸：最底层的 100% 定义触摸算法

```kotlin
Text("", Modifier.pointerInput(Unit){
  awaitPointerEventScope{
    var event = awaitPointEvent()
    print("${event.type}")
    if(event.type == PointerEventType.Release){
      
    }
  }
})
```

### 和传统的 View 系统混用

SurfaceView、TextureView 在 Compose中没有对等实现

```kotlin
// View 里面使用 Compose
val linearLayout = LinearLayout(this)
val composeView = ComposeView(this).apply{
  setContent{ CustomText() }
}
linearLayout.addView(composeView, ViewGroup.LayoutParams(
  	ViewGroup.LayoutParams.MATCH_PARENT,
  	ViewGroup.LayoutParams.WARP_CONTENT,
))
// xml 添加 androidx.compose.ui.platform.ComposeView
// <androidx.compose.ui.platform.ComposeView
//   android:layout_width="match_parent"
//   android:layout_height="wrap_content"/>
findViewById<ComposeView>(R.id.composeView).setContent{ CustomText() }

// Compose 使用 View
setContent{
  val context = LocalContext.current
  val name by remember{ mutableStateOf("hello world") }
  Theme{
    AndroidView(factory = {
      TextView(context).apply{
        text = "hello world"
      }
    }){
      //  重组过程中会 update
      it.text = name
    }
  }
}
```
