#### ⾃定义 View 的触摸反馈

- 重写 onTouchEvent()，在方法内部定制触摸反馈算法
  - 是否消费事件取决于 ACTION_DOWN 事件是否返回 true

#### ⾃定义 ViewGroup 的触摸反馈

- 除了重写 onTouchEvent() ，还需要重写 onInterceptTouchEvent()

  onInterceptTouchEvent() 用来判断是否拦截某个事件

  - onInterceptTouchEvent() 不⽤在第⼀时间返回 true，⽽是在任意⼀个事件⾥，

    需要拦截的时候返回 true 就⾏

  - 在 onInterceptTouchEvent() 中除了判断拦截，还要做好拦截之后的⼯作的准备

    ⼯作（主要和 onTouchEvent() 的代码逻辑⼀致）

#### 触摸反馈的流程

- Activity.dispatchTouchEvent() 用来进行事件的分发，一般不需要重写

  - 递归: ViewGroup(View).dispatchTouchEvent()
    - ViewGroup.onInterceptTouchEvent()
    - child.dispatchTouchEvent()
    - super.dispatchTouchEvent()
    - View.onTouchEvent()
  - Activity.onTouchEvent()

- View.dispatchTouchEvent() 就是调用 View 的 onTouchEvent()

  ```java
  public boolean dispatchTouchEvent(MotionEvent event) {
  		return onTouchEvent(event);
  }
  ```

- ViewGroup dispatchTouchEvent()

  ```java
  public boolean dispatchTouchEvent(MotionEvent event) {
      // 先调用 onInterceptTouchEvent 进行事件拦截判断
      if (onInterceptTouchEvent(event)) {
          // 如果被拦截，自己处理事件
          handled = onTouchEvent(event);
      } else {
          // 如果没有拦截，继续分发事件给子 View
          handled = super.dispatchTouchEvent(event);
      }
      return handled;
  }
  ```

  
