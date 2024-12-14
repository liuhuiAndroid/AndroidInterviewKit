#### FFmpeg代码结构

| 名称          | 介绍                                 |
| :------------ | :----------------------------------- |
| libavcodec    | 提供了一系列编码器的实现。           |
| libavformat   | 实现在流协议，容器格式及其IO访问。   |
| libavutil     | 包括了hash器，解码器和各种工具函数。 |
| libavfilter   | 提供了各种音视频过滤器。             |
| libavdevice   | 提供了访问捕获设备和回放设备的接口。 |
| libswresample | 实现了混音和重采样。                 |
| libswscale    | 实现了色彩转换和缩放功能。           |

#### FFmpeg日志系统

- include <libavutil/log.h>
- av_log_set_level(AV_LOG_DEBUG)
- av_log(NULL,AV_LOG_INFO,"...%s\n",op)

```shell
vi ffmpeg_log.c

#include<stdio.h>
#include<libavutil/log.h>
int main(int argc, char* argv[]){
		av_log_set_level(AV_LOG_DEBUG);
		av_log(NULL, AV_LOG_INFO, "Hello World. %s\n", "sec");
		return 0;
}

# clang -g -o ffmpeg_log ffmpeg_log.c -lavutil
# 上面这个有问题
clang -g -o ffmpeg_log ffmpeg_log.c `pkg-config --libs --cflags libavutil`
gcc -g -o ffmpeg_log ffmpeg_log.c `pkg-config --libs --cflags libavutil`
./ffmpeg_log
```

常用日志级别

- AV_LOG_ERROR
- AV_LOG_WARNING
- AV_LOG_INFO
- AV_LOG_DEBUG

#### 文件的删除与重命名

- 删除：avpriv_io_delete()
- 重命名：avpriv_io_move()

```shell
# echo "123" > test.txt
vi ffmpeg_file.c

#include<libavutil/log.h>
#include<libavformat/avformat.h>
int main(int argc, char* argv[]){
		int ret;
		ret = avpriv_io_move("./test.txt", "./test111.txt")
		if(ret < 0){
				av_log(NULL, AV_LOG_ERROR, "Failed to rename\n");
				return -1;
		}
		av_log(NULL, AV_LOG_INFO, "Success to rename\n");
		
		// delete file
		ret = avpriv_io_delete("./test111.txt");
		if(ret < 0){
				av_log(NULL, AV_LOG_ERROR, "Failed to delete file test111.txt \n");
				return -1;
		}
		av_log(NULL, AV_LOG_INFO, "Success to delete file test111.txt \n");
		return 0;
}

pkg-config --libs libavformat
# pkg-config --libs libavformat 可以帮我们在系统里找libavformat库的路径
clang -g -o ffmpeg_delete ffmpeg_file.c `pkg-config --libs libavformat`
./ffmpeg_delete
```

#### FFmpeg操作目录及list的实现

操作目录重要函数

- avio_open_dir()：打开目录
- avio_read_dir()：读取目录中每一项文件信息
- avio_close_dir()：关闭资源

操作目录重要结构体

- AVIODirContext：操作目录的上下文
- AVIODirEntry：目录项。用于存放文件名，文件大小等信息

```shell
# 实现简单的ls命令
vi ffmpeg_list.c

#include<libavutil/log.h>
#include<libavformat/avformat.h>
int main(int argc, char* argv[]){
		av_log_set_level(AV_LOG_INFO);
		int ret;
		AVIODirContext *ctx = NULL;
		AVIODirEntry *entry = NULL; 
		ret = avio_open_dir(&ctx, "./", NULL);
		if(ret < 0){
				av_log(NULL, AV_LOG_ERROR, "Canot open dir: %s\n", av_err2str(ret));
				goto __fail;
		}
		while(1){
				ret = avio_read_dir(ctx, &entry);
				if(ret < 0){
            av_log(NULL, AV_LOG_ERROR, "Canot read dir: %s\n", av_err2str(ret));
            goto __fail;
        }
        // 判断是否读到末尾了
        if(!entry){
        		break;
        }
        av_log(NULL, AV_LOG_INFO, "%12"PRId64" %s\n", entry->size, entry->name);
        avio_free_directory_entry(&entry);
		}
__fail:
		avio_close_dir(&ctx);
		return 0;
}

clang -g -o ffmpeg_list ffmpeg_list.c `pkg-config --libs libavformat libavutil`
./ffmpeg_list
```

#### FFmpeg处理流数据的基本概念

- 多媒体文件其实是个容器
- 在容器里有很多流（Stream/Track）
- 每种流是由不同的编码器编码的
- 从流中读出的数据称为包
- 在一个包中包含着一个或多个帧

几个重要的结构体

- AVFormatContext：格式上下文
- AVStream：流或轨
- AVPacket：包

FFmpeg操作流数据的基本步骤：解复用 -> 获取流 -> 读数据包 -> 释放资源

#### FFmpeg打印音视频Meta信息

- av_register_all()：注册FFmpeg中所有的编解码器和协议
- avformat_open_input() / avformat_close_input()：打开多媒体文件
- av_dump_format()：将多媒体文件中的Meta信息打印出来

```shell
vi ffmpeg_media_info.c

#include <libavutil/log.h>
#include <libavformat/avformat.h>
int main(int argc, char* argv[]){
		int ret;
		AVFormatContext *fmt_ctx = NULL;
		av_log_set_level(AV_LOG_INFO);
		av_register_all();
		ret = avformat_open_input(&fmt_cxt, "./test.mp4", NULL, NULL);
		if(ret < 0){
				av_log(NULL, AV_LOG_ERROR, "Can't open input: %s\n", av_err2str(ret));
				return -1;
		}
		// 第四个参数0:输入流 1:输出流
		av_dump_format(fmt_ctx, 0, "./test.mp4", 0);
		avformat_close_input(&fmt_cxt);
		return 0;
}

clang -g -o ffmpeg_media_info ffmpeg_media_info.c `pkg-config --libs libavformat libavutil`
./ffmpeg_media_info
```

#### FFmpeg抽取音频数据

- av_init_packet()：初始化数据包结构体
- av_find_best_stream()：在多媒体文件中找到最好的一路流
- av_read_frame() / av_packet_unref()：读取数据包 / 释放

```c
// vi ffmpeg_extra_audio.c
// clang -g -o ffmpeg_extra_audio ffmpeg_extra_audio.c `pkg-config --libs libavformat libavutil`
// ./ffmpeg_extra_audio ./killer.mp4 ./killer.aac
// ffplay killer.aac
#include <stdio.h>
#include <libavutil/log.h>
#include <libavformat/avformat.h>

// 需要adts header才能播放
#define ADTS_HEADER_LEN  7;
void adts_header(char *szAdtsHeader, int dataLen){
    int audio_object_type = 2;
    int sampling_frequency_index = 7;
    int channel_config = 2;
    int adtsLen = dataLen + 7;
    szAdtsHeader[0] = 0xff;         //syncword:0xfff                          高8bits
    szAdtsHeader[1] = 0xf0;         //syncword:0xfff                          低4bits
    szAdtsHeader[1] |= (0 << 3);    //MPEG Version:0 for MPEG-4,1 for MPEG-2  1bit
    szAdtsHeader[1] |= (0 << 1);    //Layer:0                                 2bits 
    szAdtsHeader[1] |= 1;           //protection absent:1                     1bit
    szAdtsHeader[2] = (audio_object_type - 1)<<6;            //profile:audio_object_type - 1                      2bits
    szAdtsHeader[2] |= (sampling_frequency_index & 0x0f)<<2; //sampling frequency index:sampling_frequency_index  4bits 
    szAdtsHeader[2] |= (0 << 1);                             //private bit:0                                      1bit
    szAdtsHeader[2] |= (channel_config & 0x04)>>2;           //channel configuration:channel_config               高1bit
    szAdtsHeader[3] = (channel_config & 0x03)<<6;     //channel configuration:channel_config      低2bits
    szAdtsHeader[3] |= (0 << 5);                      //original：0                               1bit
    szAdtsHeader[3] |= (0 << 4);                      //home：0                                   1bit
    szAdtsHeader[3] |= (0 << 3);                      //copyright id bit：0                       1bit  
    szAdtsHeader[3] |= (0 << 2);                      //copyright id start：0                     1bit
    szAdtsHeader[3] |= ((adtsLen & 0x1800) >> 11);           //frame length：value   高2bits
    szAdtsHeader[4] = (uint8_t)((adtsLen & 0x7f8) >> 3);     //frame length:value    中间8bits
    szAdtsHeader[5] = (uint8_t)((adtsLen & 0x7) << 5);       //frame length:value    低3bits
    szAdtsHeader[5] |= 0x1f;                                 //buffer fullness:0x7ff 高5bits
    szAdtsHeader[6] = 0xfc;
}

int main(int argc, char* argv[]){
		int ret;
  	int len;
  	int audio_index;
  	char* src = NULL;
  	char* dst = NULL;
  	AVPacket pkt;
		AVFormatContext *fmt_cxt = NULL;
		av_log_set_level(AV_LOG_INFO);
		av_register_all();
  	// 1. read two params from console.
  	if(argc < 3){
      	av_log(NULL, AV_LOG_ERROR, "The count of params should be more tham three.\n");
      	return -1;
    }
  	// 输入参数
  	src = argv[1];
  	// 输出参数
  	dst = argv[2];
  	if(!src || !dst){
      	av_log(NULL, AV_LOG_ERROR, "src or dst is null.\n");
      	return -1;
    }
  	// 打开输入文件
		ret = avformat_open_input(&fmt_cxt, src, NULL, NULL);
		if(ret < 0){
				av_log(NULL, AV_LOG_ERROR, "Can't open input: %s\n", av_err2str(ret));
				return -1;
		}
  	// 打开输出文件
  	File* dst_fd = fopen(dst, "wb");
  	if(!dst_fd){
      	av_log(NULL, AV_LOG_ERROR, "Can't open out file\n");
				avformat_close_input(&fmt_cxt);
				return -1;
    }
		av_dump_format(fmt_cxt, 0, src, 0);
  	// 2. get stream 
  	// 第二个参数：想抽取的流的类型
  	ret = av_find_best_stream(fmt_ctx, AVMEDIA_TYPE_AUDIO, -1, -1, NULL, 0);
  	if(ret < 0){
				av_log(NULL, AV_LOG_ERROR, "Can't find best stream.\n");
				avformat_close_input(&fmt_cxt);
      	fclose(dst_fd);
				return -1;
		}
  	audio_index = ret;
  	av_init_packet(&pkt);
  	// 读流中所有的数据包
  	while(av_read_frame(fmt_ctx, &pkt) >= 0){
      	if(pkt.stream_index == audio_index){
          	char adts_header_buf[7];
          	adts_header(adts_header_buf, pkt.size);
          	fwrite(adts_header_buf, 1, 7, dst_fd);
  					// 3. write audio data to aac file. 
          	// 输出到目标文件
          	len = fwrite(pkt.data, 1, pkt.size, dst_fd);
          	if(len != pkt.size){
              	av_log(NULL, AV_LOG_WARNING, "warning, length of data is not equal size of pkt.\n");
            }
        }
      	av_packet_unref(&pkt);
    }
		avformat_close_input(&fmt_cxt);
  	if(dst_fd){
      	fclose(dst_fd);
    }
		return 0;
}
```

#### FFmpeg抽取视频数据

- Start code：特征码，区分帧与帧间隔
- SPS/PPS：解码视频参数
- codec -> extradata：获取SPS/PPS

```shell
vi extra_video.c
clang -g -o extra_video extra_video.c `pkg-config --libs libavformat libavutil`
./extra_video ./killer.mp4 killer.h264
ffplay killer.h264
```

#### FFmpeg多媒体格式转封装：将mp4转成flv

- avformat_alloc_output_context2() / avformat_free_context()：分配上下文
- avformat_new_stream()
- avcodec_parameters_copy()
- avformat_write_header()：写头
- av_write_frame() / av_interleaved_write_frame()：写数据
- av_write_trailer()：写尾部

```shell
vi remux.c
clang -g -o remux remux.c `pkg-config --libs libavformat`
./remux ./killer.mp4 killer.flv
```

#### FFmpeg视频裁剪

- av_seek_frame()

```shell
vi cutvideo.c
clang -g -o cutvideo cutvideo.c `pkg-config --libs libavformat libavutil libavcodec`
./cutvideo 10 20 ./killer.mp4 killer2.mp4
ffplay killer2.mp4
```

#### FFmpeg实现小咖秀

- 将两个媒体文件中分别抽取音频与视频轨
- 将音频与视频轨合并成一个新文件
- 对音频与视频轨进行裁剪
