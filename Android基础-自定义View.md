#### 布局流程

从整体看：

- 测量流程：从根 View 递归调⽤每⼀级⼦ View 的 measure() ⽅法，对它们进⾏
  
  测量

- 布局流程：从根 View 递归调⽤每⼀级⼦ View 的 layout() ⽅法，把测量过程得
  
  出的⼦ View 的位置和尺⼨传给⼦ View，⼦ View 保存

为什么要分两个流程？因为可能会重复测量。

从个体看，对于每个 View：

- 运⾏前，开发者在 xml ⽂件⾥写⼊对 View 的布局要求 layout_xxx

- ⽗ View 在⾃⼰的 onMeasure() 中，根据开发者在 xml 中写的对⼦ View 的要
  
  求，和⾃⼰的可⽤空间，得出对⼦ View 的具体尺⼨要求

- ⼦ View 在⾃⼰的 onMeasure() 中，根据⽗ View 的要求和⾃⼰的特性算出⾃⼰
  
  的期望尺⼨

  - 如果是 ViewGroup，还会在这⾥调⽤每个⼦ View 的 measure() 进⾏测量
  
- ⽗ View 在⼦ View 计算出期望尺⼨后，得出⼦ View 的实际尺⼨和位置

- ⼦ View 在⾃⼰的 layout() ⽅法中，将⽗ View 传进来的⾃⼰的实际尺⼨和位置

  保存

  - 如果是 ViewGroup，还会在 onLayout() ⾥调⽤每个⼦ View 的 layout() 把
    
    它们的尺⼨位置传给它们



**⾃定义布局：尺⼨的⾃定义**

1. 继承已有的View，简单改写它们的尺⼨
   1. 重写 onMeasure()
   2. ⽤ getMeasuredWidth() 和 getMeasuredHeight() 获取到测量出的尺⼨
   3. 计算出最终要的尺⼨
   4. ⽤ setMeasuredDimension(width, height) 把结果保存
   5. 例⼦：SquareImageView
2. 对⾃定义 View 完全进行自定义尺⼨计算
   1. 重写 onMeasure()
   2. 计算出⾃⼰的尺⼨
   3. ⽤ resolveSize() 或者 resolveSizeAndState() 修正结果
   4. 使⽤ setMeasuredDimension(width, height) 保存结果
   5. 例⼦：CircleView
3. 自定义 Layout：重写 onMeasure() 和 onLayout()
   1. TagLayout
   2. 查看 google官方 flexbox-layout 源码



**⾃定义** **Layout**

- 重写 onMeasure()

  - 遍历每个⼦ View，测量⼦ View

    - 测量完成后，得出⼦ View 的实际位置和尺⼨，并暂时保存
    - 有些⼦ View 可能需要重新测量

  - 测量出所有⼦ View 的位置和尺⼨后，计算出⾃⼰的尺⼨，并⽤

    setMeasuredDimension(width, height) 保存

- 重写 onLayout()

  - 遍历每个⼦ View，调⽤它们的 layout() ⽅法来将位置和尺⼨传给它们

- 例⼦：TagLayout



1. resolveSizeAndState 方法原理
2. measureChildWithMargins 方法原理



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

#### 自定义ViewGroup流程

需要梳理

#### 自定义 View 如何处理文本过错问题

1. 在 onSizeChanged 方法中调用 TextUtils.ellipsize 
2. onDraw 只负责使用已处理的文本进行绘制
