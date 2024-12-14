#### FFmpeg H264 视频解码

1. 添加头文件：其中包括常用的编码解码API

    ```c
    #include <libavcodec/avcodec.h>
    ```

2. 常用数据结构

   1. AVCodec 编码器结构体
   2. AVCodecContext 编码器上下文
   3. AVFrame 解码后的帧

3. 结构体内存的分配与释放

   1. av_frame_alloc() / av_frame_free()
   2. avcodec_alloc_context3()
   3. avcodec_free_context()

4. 解码步骤

   1. 查找解码器：avcodec_find_decoder()
   2. 打开解码器：avcodec_open2()
   3. 解码：avcodec_decode_video2()

#### FFmpeg H264 视频编码

H264编码流程

1. 查找编码器：avcodec_find_encoder_by_name()
2. 设置编码参数，并打开编码器：avcodec_open2()
3. 编码：avcodec_encode_video2()

```
vi encode_video.c
```

#### FFmpeg AAC编解码

- 编码流程与视频相同
- 编码函数：avcodec_encodec_audio2()

```
vi encode_audio.c
clang -g -o encode_audio encode_audio.c `pkg-config --libs libavcodec`
./encode_audio 1.aac
```

#### 实战：FFmpeg 视频转图片

```shell
vi decode_video.c
clang -g -o decode_video decode_video.c `pkg-config --libs libavformat libavcodec libavutil libswscale`
./decode_video killer.mp4 ./
open a.bmp
# vi 查找代码在哪 快捷键
```

#### 生成带色彩的BMP图片
