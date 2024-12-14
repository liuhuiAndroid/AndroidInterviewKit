#### FFmpeg常用命令

- 基本信息查询命令

  ```shell
  # 显示版本
  ffmpeg -version
  # 显示可用的demuxers
  ffmpeg -demuxers
  # 显示可用的muxers
  ffmpeg -muxers
  # 显示可用的设备
  ffmpeg -devices
  # 显示所有的编解码器
  ffmpeg -codecs | more
  ffmpeg -codecs | grep 264
  # 显示可用的解码器
  ffmpeg -decoders
  # 显示可用的编码器
  ffmpeg -encoders
  # 显示比特流filter
  ffmpeg -bsfs
  # 显示可用的格式
  ffmpeg -formats
  ffmpeg -formats | grep mp3
  # 显示可用的协议
  ffmpeg -protocols
  # 显示可用的过滤器
  ffmpeg -filters
  # 显示可用的像素格式
  ffmpeg -pix_fmts
  # 显示可用的采样格式
  ffmpeg -sample_fmts
  # 显示channel名称
  ffmpeg -layouts
  # 显示识别的颜色名称
  ffmpeg -colors
  ```

- 录制命令

  ```shell
  # 录屏
  # -f:指定使用 avfoundation 采集数据
  # -i:指定从哪儿采集数据，它是一个文件索引号，1代表屏幕的索引值
  # -r:指定帧率
  ffmpeg -f avfoundation -i 1 -r 30 out.yuv
  # 播放录屏 测试失败 
  ffplay -s 2880x1800 -pix_fmt uyvy422 out.yuv
  # 查看 avfoundation 支持视频设备和音频设备
  ffmpeg -f avfoundation -list_devices true -i ""
  # 录音，通过命令方式采集音频
  # :0代表音频设备，视频设备在:之前，音频设备在:之后
  ffmpeg -f avfoundation -i :0 out.wav
  # 播放音频
  ffplay out.wav
  # 录视频
  ffmpeg -framerate 30 -f avfoundation -i 0 out.mp4
  # 播放视频
  ffplay -s 640x480 -pix_fmt uyvy422 out.mp4
  ```

- 分解/复用命令 demuxer/muxer，媒体格式互转或抽取数据

  ```shell
  # ffplay 韵动中国.mp4
  # 多媒体格式转换
  #-i:输入文件
  #-vcodec copy:视频编码处理方式
  #-acodec copy:音频编码处理方式
  ffmpeg -i china.mp4 -vcodec copy -acodec copy china.flv
  # 抽取视频流
  # -vn:不包括视频
  # -an:不包括音频
  ffmpeg -i china.mp4 -vcodec copy -an out.h264
  ffplay out.h264
  ffmpeg -i china.mp4 -vn -acodec copy out.aac
  ```

- 处理原始数据命令，处理解码后的数据，音频pcm，视频yuv

  ```shell
  # 提取yuv数据
  # -an:不包括音频
  # -c:v rawvideo 使用原始视频编码
  # -pix_fmt:指定像素格式
  ffmpeg -i xxx.mp4 -an -c:v rawvideo -pix_fmts yuv420p out.yuv
  ffplay -s 2880x1800 -pix_fmt yuv420p out.yuv
  
  # 提取pcm数据
  # -vn:不包括视频
  # -ar:音频采样率 audio rate
  # -ac:单声道还是双声道还是立体声
  # -f:音频存储格式，s代表有符号16位le小头，是一种存储方式
  ffmpeg -i china.mp4 -vn -ar 44100 -ac 2 -f s16le out.pcm
  ffplay -ar 44100 -ac 2 -f s16le out.pcm
  ```

- 滤镜命令

  首先拿到解码后的数据帧，然后经过滤镜程序，再重新编码

  ```shell
  # 通过视频滤镜裁剪视频
  # -vf:视频滤镜
  # in_w:视频宽度，-200减少200
  # in_h:视频高度，-200减少200
  # -c:v 设置视频编码器
  # -c:a 设置音频编码器
  # copy格式:crop=out_w:out_h:x:y
  ffmpeg -i xxx.mov -vf crop=in_w-200:in_h-200 -c:v libx264 -c:a copy out.mp4
  ffplay out.mp4
  ```

- 裁剪与合并命令

  ```shell
  # 视频裁剪
  # -ss指定开始裁剪的时间
  # -t指定持续多长时间
  ffmpeg -i china.mp4 -ss 00:10:00 -t 10 1.ts
  ffplay 1.ts
  
  # 音视频合并
  # inputs.txt 内容为 'file filename'格式
  # file '1.ts'
  # file '2.ts'
  ffmpeg -f concat -i inputs.txt out.flv
  ```

- 图片/视频互转命令

  ```shell
  # 视频转图片
  # -r 1 每秒转1张图片
  # -f 输出格式image2
  ffmpeg -i china.mp4 -r 1 -f image2 image-%3d.jpeg
  open image-001.jpeg
  
  # 图片转视频
  ffmpeg -i image-%3d.jpeg out.mp4
  ffplay out.mp4
  ```

- 直播相关命令

  ```shell
  # 直播推流
  # -re 减慢帧率速度
  # -c copy 不对音视频做修改
  ffmpeg -re -i out.mp4 -c copy -f flv rtmp://server/live/streamName
  
  # 直播拉流
  ffmpeg -i rtmp://server/live/streamName -c copy dump.flv
  ffmpeg -i http://xxx.m3u8 -c copy dump.m3u8
  ffplay dump.m3u8
  ```
