requestLayout 触发重新绘制，结束绘制后 post 到屏幕上显示



VSync信号

- 当整个屏幕刷新完毕，即一个垂直刷新周期完成，就会发出 VSync 信号

- 以 60Hz 刷新率为前提，VSync 信号的周期，应该是 16.66 ms

- SurfaceFlinger 会把硬件的 VSync 信号转换成软件的 VSync 信号



重入标记，一次 VSync 标记只会 requestLayout 一次
