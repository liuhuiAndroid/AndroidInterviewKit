dispacthTouchEvent

用来进行事件的分发

onInterceptTouchEvent

用来判断是否拦截某个事件



伪代码：

```
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
