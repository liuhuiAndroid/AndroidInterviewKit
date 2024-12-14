##### Filter 可以做什么？

- 音视频的倍速播放
- 视频添加/删除Logo
- 视频画中画

##### Filter 的类型

- 简单滤镜
- 复杂滤镜
- 滤镜的级联

##### Filter的基本原理

- 将压缩后的每一帧数据进行解码
- 对解码后的音视频数据进行运算
- 最后再将处理好的数据进行编码

##### 命令行使用 Filter

```shell
ffmpeg -filters
ffmpeg -filters | grep drawbox
ffmpeg -filters | grep overlay
ffmpeg -filters | more
# 查看Filter的详细信息
ffmpeg -h filter=drawbox
ffmpeg -h filter=showvolume
# -vf 音频滤镜
ffplay -i /Users/sec/Movies/Videos/china.mp4 -vf "drawbox=x=30:y=10:w=64:h=64:c=red"
ffplay -i /Users/sec/Movies/Videos/china.mp4 -vf "drawbox=30:10:64:64:red"
ffplay -i /Users/sec/Movies/Videos/china.mp4 -vf "movie=/Users/sec/Movies/Videos/logo.png,scale=64:64[wm];[in][wm]overlay=30:10"
# 2倍速播放
ffmpeg -i /Users/sec/Movies/Videos/china.mp4 -vf "setpts=0.5*PTS[v]" /Users/sec/Movies/Videos/china-speed2.mp4
```

##### Filter语法

1. 在 avfilter 参数之间用 : 来分割
2. 多个 avfilter 之间串联用 , 来分割
3. 多个 avfilter 之间没有关联用 ; 来分割

##### API的方式使用 Filter

1. 引入avfilter库头文件和库文件
2. 设置filter描述符
3. 初始化filter
4. 进行滤镜处理
5. 释放filter资源

##### 两个特殊的filter

1. buffer filter：用于数据源缓冲
2. buffersink filter：用于输出数据缓冲

##### 初始化 filter 的基本过程

1. 创建 graph， avfilter_graph_alloc();
2. 创建 buffer filter 和 buffersink filter
3. 分析 filter 描述符，并构建 AVFilterGraph
4. 使构建好的 AVFilterGraph 生效

##### 使用 filter 的步骤

1. 获得解码后的原始数据 PCM/YUV
2. 将数据添加到 buffer filter 中
3. 从 buffer sink 中取出处理好的数据
4. 当所有数据处理完后，释放资源

```c
// vi ffmpeg_filter.c
// clang -g -o ffmpeg_filter ffmpeg_filter.c `pkg-config --libs libavutil libavformat libavfilter libavcodec`
// ./ffmpeg_filter
#include <stdio.h>

#include "libavutil/avutil.h"
#include "libavutil/opt.h"
#include "libavformat/avformat.h"
#include "libavcodec/avcodec.h"
#include "libavfilter/avfilter.h"
#include "libavfilter/buffersrc.h"
#include "libavfilter/buffersink.h"

/**
 * @brief open media file 打开多媒体文件
 * @param filename xxx
 * @param [out] fmt_ctx xxx
 * @param [out] dec_ctx xxx
 * @return 0: success, <0: failure
 */
static int open_input_file(const char *filename,
                           AVFormatContext **fmt_ctx,
                           AVCodecContext **dec_ctx,
                           int *v_stream_index){
    int ret = -1;
    AVCodec *dec = NULL;
    if(( ret = avformat_open_input(fmt_ctx, filename, NULL, NULL)) < 0){
        av_log(NULL, AV_LOG_ERROR, "Failed to open file %s\n", filename);
        return ret;
    }
    if((ret = avformat_find_stream_info((*fmt_ctx), NULL)) < 0){
        av_log(NULL, AV_LOG_ERROR, "Failed to find stream information!\n");
        return ret;
    }
  	// 找到视频流
    if ((ret = av_find_best_stream(*fmt_ctx, AVMEDIA_TYPE_VIDEO, -1, -1, &dec, 0)) < 0){
        av_log(NULL, AV_LOG_ERROR, "Can't find video stream!\n");
        return ret;
    }
    *v_stream_index = ret;
    *dec_ctx = avcodec_alloc_context3(dec);
    if(!(*dec_ctx)){
      	// 内存不足
        return AVERROR(ENOMEM);
    }
    avcodec_parameters_to_context(*dec_ctx, (*fmt_ctx)->streams[*v_stream_index]->codecpar);
    if((ret = avcodec_open2(*dec_ctx, dec, NULL)) < 0){
        av_log(NULL, AV_LOG_ERROR, "Failed to open decoder!\n");
        return ret;
    }
    return 0;
}

static int init_filters(const char *filter_desc,
                        AVFormatContext *fmt_ctx,
                        AVCodecContext *dec_ctx,
                        int v_stream_index,
                        AVFilterGraph **graph,
                        AVFilterContext **buf_ctx,
                        AVFilterContext **bufsink_ctx){
    int ret = -1;
    char args[512] = {};
    AVRational time_base = fmt_ctx->streams[v_stream_index]->time_base;
    AVFilterInOut *inputs = avfilter_inout_alloc();
    AVFilterInOut *outputs = avfilter_inout_alloc();
    if(!inputs || !outputs){
        av_log(NULL, AV_LOG_ERROR, "No Memory when alloc inputs or outputs!\n");
        return AVERROR(ENOMEM);
    }
    
    *graph = avfilter_graph_alloc();
    if(!(*graph)){
        av_log(NULL, AV_LOG_ERROR, "No Memory when create graph!\n");
        return AVERROR(ENOMEM);
    }
    
    const AVFilter *bufsrc = avfilter_get_by_name("buffer");
    if(!bufsrc){
        av_log(NULL, AV_LOG_ERROR, "Failed to get buffer filter!\n");
        return -1;
    }
    
    const AVFilter *bufsink = avfilter_get_by_name("buffersink");
    if(!bufsink){
        av_log(NULL, AV_LOG_ERROR, "Failed to get buffersink filter!\n");
        return -1;
    }
    
    // 构造输入 buffer filter
    // "[in]drawbox=xxxx[out]"
  	// snprintf 输出字符串
    snprintf(args, 512,
             "video_size=%dx%d:pix_fmt=%d:time_base=%d/%d:pixel_aspect=%d/%d",
             dec_ctx->width, dec_ctx->height,
             dec_ctx->pix_fmt,
             time_base.num, time_base.den,
             dec_ctx->sample_aspect_ratio.num, dec_ctx->sample_aspect_ratio.den);
    
    if((ret = avfilter_graph_create_filter(buf_ctx, bufsrc, "in", args, NULL, *graph))<0){
        av_log(NULL, AV_LOG_ERROR, "Failed to create buffer filter context!\n");
        goto __ERROR;
    }
    
    // 构造输出 buffer sink filter
    enum AVPixelFormat pix_fmts[] = {AV_PIX_FMT_YUV420P, AV_PIX_FMT_GRAY8, AV_PIX_FMT_NONE};
    if((ret =avfilter_graph_create_filter(bufsink_ctx, bufsink, "out", NULL, NULL, *graph)) < 0 ){
        av_log(NULL, AV_LOG_ERROR, "Failed to create buffer sink filter context!\n");
        goto __ERROR;
    }
    av_opt_set_int_list(*bufsink_ctx, "pix_fmts", pix_fmts, AV_PIX_FMT_NONE, AV_OPT_SEARCH_CHILDREN);

    //create in/out
    inputs->name = av_strdup("out");
    inputs->filter_ctx = *bufsink_ctx;
    inputs->pad_idx = 0;
    inputs->next = NULL;
    
    outputs->name = av_strdup("in");
    outputs->filter_ctx = *buf_ctx;
    outputs->pad_idx = 0;
    outputs->next = NULL;
    
    //create filter and add graph for filter desciption
    if((ret = avfilter_graph_parse_ptr(*graph, filter_desc, &inputs, &outputs, NULL)) < 0){
        av_log(NULL, AV_LOG_ERROR, "Failed to parse filter description!\n");
        goto __ERROR;
    }
    
    if((ret = avfilter_graph_config(*graph, NULL)) < 0){
        av_log(NULL, AV_LOG_ERROR, "Failed to config graph!\n");
    }

__ERROR:
  	// 释放资源
    avfilter_inout_free(&inputs);
    avfilter_inout_free(&outputs);
    return ret;
}

static int do_frame(AVFrame *filt_frame, FILE *out){
    
    fwrite(filt_frame->data[0], 1, filt_frame->width*filt_frame->height, out);
    fflush(out);
    
    return 0;
}

//do filter
static int filter_video(AVFrame *frame,
                        AVFrame *filt_frame,
                        AVFilterContext *buf_ctx,
                        AVFilterContext *bufsink_ctx,
                        FILE *out){
    
    int ret;
    
    if ((ret = av_buffersrc_add_frame(buf_ctx, frame)) < 0 ){
        av_log(NULL, AV_LOG_ERROR, "Failed to feed to filter graph!\n");
        return ret;
    }
    
    while(1){
        ret = av_buffersink_get_frame(bufsink_ctx, filt_frame);
        if(ret == AVERROR(EAGAIN) || ret == AVERROR_EOF){
            break;
        }
        
        if(ret < 0) {
            return ret;
        }
        
        do_frame(filt_frame, out);
        av_frame_unref(filt_frame);
    }
    
    av_frame_unref(frame);
    
    return ret;
}

//解码视频帧并对视频帧进行滤镜处理
static int decode_frame_and_filter(AVFrame *frame,
                                   AVFrame *filt_frame,
                                   AVCodecContext *dec_ctx,
                                   AVFilterContext *buf_ctx,
                                   AVFilterContext *bufsink_ctx,
                                   FILE *out){
    
    int ret = avcodec_receive_frame(dec_ctx, frame);
    if(ret < 0){
        if(ret != AVERROR_EOF && ret != AVERROR(EAGAIN)){
            av_log(NULL, AV_LOG_ERROR, "Error while receiving a frame from decoder!\n");
        }

        return ret;
    }
    
    return filter_video(frame,
                        filt_frame,
                        buf_ctx,
                        bufsink_ctx,
                        out);
}


int main(int argc, const char * argv[]) {
    int ret;
    FILE *out = NULL;
    
    AVFormatContext *fmt_ctx;
    AVCodecContext *dec_ctx;
    
    AVFilterGraph *graph = NULL;
    AVFilterContext *buf_ctx = NULL;
    AVFilterContext *bufsink_ctx = NULL;
    
    int v_stream_index = -1;

    AVPacket packet;
    AVFrame *frame = NULL;
    AVFrame *filt_frame = NULL;
    
  	// 描述符
    const char *filter_desc="drawbox=30:10:64:64:red";
    const char* filename = "/Users/sec/Movies/Videos/china.mp4";
    const char* outfile = "/Users/sec/Movies/Videos/china.yuv";
    
  	// 设置日志级别
    av_log_set_level(AV_LOG_DEBUG);
    
    frame = av_frame_alloc();
    filt_frame = av_frame_alloc();
    if(!frame || !filt_frame){
        av_log(NULL, AV_LOG_ERROR, "No Memory to alloc frame\n");
        exit(-1);
    }
    
    out = fopen(outfile, "wb");
    if(!out){
        av_log(NULL, AV_LOG_ERROR, "Failed to open yuv file!\n");
        exit(-1);
    }
    
  	// 打开媒体文件
    if((ret = open_input_file(filename,
                       &fmt_ctx,
                       &dec_ctx,
                       &v_stream_index)) < 0) {
        av_log(NULL, AV_LOG_ERROR, "Failed to open media file\n");
        goto __ERROR;
    }else{
      	// 初始化 filter
        if((ret = init_filters(filter_desc,
                        fmt_ctx,
                        dec_ctx,
                        v_stream_index,
                        &graph,
                        &buf_ctx,
                        &bufsink_ctx)) < 0){
            av_log(NULL, AV_LOG_ERROR, "Failed to initialize filter!\n");
            goto __ERROR;
        }
    }
    
    //read avpacket from media file
    while(1){
        // 读取数据
        if((ret = av_read_frame(fmt_ctx, &packet)) < 0){
            goto __ERROR;
        }
        // 检测是否是同一个流
        if(packet.stream_index == v_stream_index){
            ret = avcodec_send_packet(dec_ctx, &packet);
            if(ret < 0){
                av_log(NULL, AV_LOG_ERROR, "Failed to send avpakcet to decoder!\n");
                goto __ERROR;
            }
            // 滤镜处理
            if((ret = decode_frame_and_filter(frame,
                                              filt_frame,
                                              dec_ctx,
                                              buf_ctx,
                                              bufsink_ctx,
                                              out)) < 0){
                
                if(ret == AVERROR(EAGAIN) || ret == AVERROR_EOF){
                    continue;
                }
                
                av_log(NULL, AV_LOG_ERROR, "Failed to decode or filter!\n");
                goto __ERROR;
            }
        }
    }
    
__ERROR:
    if(graph){
        avfilter_graph_free(&graph);
    }
    
    if(dec_ctx){
        avcodec_free_context(&dec_ctx);
    }
    
    if(fmt_ctx){
        avformat_close_input(&fmt_ctx);
    }
    
    if(frame){
        av_frame_free(&frame);
    }
    
    if(filt_frame){
        av_frame_free(&filt_frame);
    }
    
    return ret;
}
```

##### 如何实现一个 Filter

1. 找一个线程的filter为模版
2. 替换代码中的所有filter name关键字
3. 修改 libavfilter 中的 Makefile，增加新的 filter
4. 修改 all filters.c，增加新 filter，并重新编译 ffmpeg

##### 实现filter的几个重要结构

1. AVFilter
2. AVFilterPad
3. AVFilterLink
4. AVFilterFormat
