#### 如何设计一个头像的自定义`View`，要求使头像展示出来是一个圆形？

1. **创建一个自定义View类**
2. **重写`onDraw`方法：** 在`onDraw`方法中绘制圆形头像。
3. **在布局文件中使用自定义View：** 在XML布局文件中使用你的自定义View。



#### View 绘制流程

1. measure（测量）阶段
   
   1. 在 `measure` 阶段，系统会调用每个 View 的 `onMeasure()` 方法，以确定 View 的大小。
   
   2. `onMeasure()` 方法中会根据 View 的 `LayoutParams` 和父容器的要求来计算宽度和高度。
   
   3. 通过 `setMeasuredDimension()` 来设置 View 的测量宽度和高度。

   可以根据 `MeasureSpec` 来测量自定义 `View` 的大小，并确保其满足父容器的测量要求。
   
   `MeasureSpec` 由两部分组成：测量模式（measure mode）和测量值（measure size）
   
2. layout（布局）阶段
   
   1. 在 `layout` 阶段，系统会调用每个 View 的 `onLayout()` 方法，确定 View 在父容器中的位置。
   
   2. `onLayout()` 方法中会根据 `setMeasuredDimension()` 设置的宽高和父容器的规则来确定 View 的位置。

3. draw（绘制）阶段
   
   1. 在 `draw` 阶段，系统会调用每个 View 的 `onDraw()` 方法，进行实际的绘制。
   
   2. `onDraw()` 方法中使用 `Canvas` 对象来进行绘制操作，包括绘制背景、文本、图形等。

4. 绘制背景和前景
   
   在 `onDraw()` 中，先绘制背景，然后绘制前景，确保前景能够覆盖在背景之上。

5. dispatchDraw（子 View 绘制）
   
   1. 如果是一个容器 View，例如 `ViewGroup`，会在 `draw` 阶段调用 `dispatchDraw()` 来绘制子 View。
   
   2. 子 View 的绘制顺序由它们在容器中的顺序决定。

6. 绘制缓存
   
   如果开启了硬件加速，还会涉及绘制缓存的处理，可以通过 `setLayerType()` 方法来设置。

注意：View的绘制通常发生在`onCreate`之后，在`onResume`之前。

View的绘制是在`onDraw`方法中完成的，而`onDraw`方法的调用通常由系统自动触发，或者通过手动调用`invalidate`来请求重新绘制。



#### 如何在 onCreate 中获取 View 的宽高

1. view.addOnPreDrawListener 可以在绘制之前获取其宽度和高度
2. 使用 `View.post()` 方法
3. 使用 `ViewTreeObserver.OnGlobalLayoutListener`

