### FFmpeg播放器核心功能开发

##### 实现最简单的播放器：解码和渲染结合的播放器

- 该播放器只实现视频播放
- 将FFmpeg与SDL结合到一起
- 通过FFmpeg解码视频数据
- 通过SDL进行渲染
- 进阶：可以同时播放音频与视频
- 进阶：使用队列存放音频包

```shell
clang -g -o player2 player2.c `pkg-config --cflags --libs sdl2 libavformat libavutil libswscale libavcodec`
./player2 test.mov
```

##### 多线程与锁

为啥要用多线程，多线程的好处

线程的互斥与同步

锁与信号量

锁的种类：

- 读写锁
- 自旋锁

SDL线程的创建

- SDL_CreateThread
- SDL_WaitThread

SDL锁

- SDL_CreateMutex / SDL_DestoryMutex
- SDL_LockMutex / SDL_UnlockMutex

SDL条件变量

- SDL_CreateCond / SDL_DestroyCond
- SDL_CondWait / SDL_CondSignal

##### 播放器线程模型

输入文件，创建线程来解复用，再创建子线程，视频解码线程

解复用将解出的视频数据放入视频流队列，音频流数据放入音频流队列

#### 音视频同步

时间戳 

- PTS：Presentation timestamp
- DTS：Decoding timestamp
- I / B / P 帧
  - 实际帧顺序： I B B P
  - 存放帧顺序：I P B B

从哪儿获得PTS

- AVPacket 中的 PTS
- AVFrame 中的 PTS
- av_frame_get_best_effort_timestamp()

时间基

- tbr：帧率
- tbn：time base of stream
- tbc：time base of codec

计算当前帧的PTS：

- PTS = PTS * av_q2d(video_stream -> time_base)
- av_q2d(AVRotional a){ return a.num / (double)a.den;}
- video_clock：预测的下一帧视频的PTS
- frame_delay：1/tbr
- audio_clock：音频当前播放的时间戳

音视频同步方式：

- 视频同步到音频
- 音频同步到视频
- 音频和视频都同步到系统时钟

视频播放的基本思路：

一般的做法，展示第一帧视频帧后，获得要显示的下一个视频帧的PTS，然后设置一个定时器，当定时器超时后，刷新新的视频帧，如此反复操作。

##### 音视频同步

