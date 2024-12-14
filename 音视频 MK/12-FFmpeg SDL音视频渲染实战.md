### FFmpeg SDL音视频渲染实战

##### SDL介绍：https://www.libsdl.org/

- SDL：Simple DirectMedia Layer
- 由C语言实现的跨平台的媒体开源库
- 多用于开发游戏、模拟器、媒体播放器等多媒体应用领域

```shell
# 下载：[SDL2-2.0.22.tar.gz](https://www.libsdl.org/release/SDL2-2.0.22.tar.gz)
tar -zxvf SDL2-2.0.22.tar.gz
cd SDL2-2.0.22
# 生成 Makefile
./configure --prefix=/usr/local/sdl
# 安装
sudo make -j 4 && make install
```

##### 使用SDL基本步骤

- 添加头文件 #include <SDL.h>
- 初始化SDL
- 退出SDL

SDL渲染窗口

- SDL_Init() / SDL_Quit()
- SDL_CreateWindow() / SDL_DestoryWindow()
- SDL_CreateRender() 

```c
// vi first_sdl.c
// clang -g -o first_sdl first_sdl.c `pkg-config --cflags --libs sdl2`
// ./first_sdl
#include <SDL.h>
#include <stdio.h>
int main(int argc, char* argv[]){
  	SDL_Window *window = NULL;
		SDL_Init(SDL_INIT_VIDEO);
  	// 创建窗口，但是不会显示
  	window = SDL_CreateWindow("SDL2 Window", 200, 200, 640, 480, SDL_WINDOW_SHOWN | SDL_WINDOW_BORDER);
  	if(!window){
      	prinf("Failed to Create window.");
      	goto __EXIT;
    }
  	SDL_DestoryWindow(window);
__EXIT:
  	SDL_Quit();
		return 0;
}
```

##### SDL渲染窗口

- SDL_CreateRenderer / SDL_DestoryRenderer
- SDL_RenderClear
- SDL_RenderPresent

```c
// vi first_sdl.c
// clang -g -o first_sdl first_sdl.c `pkg-config --cflags --libs sdl2`
// ./first_sdl
#include <SDL.h>
#include <stdio.h>
int main(int argc, char* argv[]){
  	SDL_Window *window = NULL;
  	SDL_Renderer *render = NULL;
		SDL_Init(SDL_INIT_VIDEO);
  	window = SDL_CreateWindow("SDL2 Window", 200, 200, 640, 480, SDL_WINDOW_SHOWN | SDL_WINDOW_BORDER);
  	if(!window){
      	prinf("Failed to Create window.");
      	goto __EXIT;
    }
  	render = SDL_CreateRenderer(window, -1, 0);
  	if(!render){
      	SDL_Log("Failed to Create Render.");
      	goto __DWINDOW;
    }
  	// 涂色
  	SDL_SetRenderDrawColor(render, 255, 255, 0, 255);
  	// 清屏
  	SDL_RenderClear(render);
  	// 展示数据
  	SDL_RenderPresent(render);
  	SDL_Delay(8000);
  	SDL_DestoryRenderer(render);
__DWINDOW:
  	SDL_DestoryWindow(window);
__EXIT:
  	SDL_Quit();
		return 0;
}
```

##### SDL事件处理基本原理

- SDL将所有事件都存放在一个队列中
- 所有对事件的操作，其实就是对队列的操作

SDL事件种类：

- SDL_WindowEvent：窗口事件
- SDL_KeyboardEvent：键盘事件
- SDL_MouseMotionEvent：鼠标事件

SDL事件处理

- SDL_PollEvent()：轮训，不及时
- SDL_WaitEvent()：阻塞式等待
- SDL_WaitEventTimeout()：阻塞式等待一小段时间

```c
// vi event_sdl.c
// clang -g -o event_sdl event_sdl.c `pkg-config --cflags --libs sdl2`
// ./event_sdl
#include <SDL.h>
#include <stdio.h>
int main(int argc, char* argv[]){
  	int quit = 1;
  	SDL_Event event;
  	SDL_Window *window = NULL;
  	SDL_Renderer *render = NULL;
		SDL_Init(SDL_INIT_VIDEO);
  	window = SDL_CreateWindow("SDL2 Window", 200, 200, 640, 480, SDL_WINDOW_SHOWN | SDL_WINDOW_BORDER);
  	if(!window){
      	prinf("Failed to Create window.");
      	goto __EXIT;
    }
  	render = SDL_CreateRenderer(window, -1, 0);
  	if(!render){
      	SDL_Log("Failed to create render.");
      	goto __DWINDOW;
    }
  	SDL_SetRenderDrawColor(render, 255, 255, 0, 255);
  	SDL_RenderClear(render);
  	SDL_RenderPresent(render);
  	do{
      	SDL_WaitEvent(&event);
      	switch(event.type){
          case SDL_QUIT:
            quit = 0;
            break;
          default:
            SDL_Log("event type is %d", event.type);
        }
    }while(quit);
  	SDL_DestoryRenderer(render);
__DWINDOW:
  	SDL_DestoryWindow(window);
__EXIT:
  	SDL_Quit();
		return 0;
}
```

##### 纹理渲染

SDL 渲染基本原理：内存图像通过渲染器变成纹理，交给显卡在窗口展示

SDL纹理相关API：

- SDL_CreateTexture()：创建纹理

  参数：format：YUV，RGB

  参数：access：访问Texture类型，Target，Stream

- SDL_DestoryTexture()：销毁纹理

SDL渲染相关API：

- SDL_SetRenderTarget()：往哪去渲染
- SDL_RenderClear()：清屏，刷屏
- SDL_RenderCopy()：将纹理拷贝到显卡
- SDL_RenderPresent()：渲染

```c
// vi texture_sdl.c
// clang -g -o texture_sdl texture_sdl.c `pkg-config --cflags --libs sdl2`
// ./texture_sdl
#include <SDL.h>
#include <stdio.h>
int main(int argc, char* argv[]){
  	int quit = 1;
  	SDL_Event event;
  	SDL_Textuer *texture = NULL;
  	// 方块
    SDL_Rect rect;
    rect.w = 30;
    rect.h = 30;
  	SDL_Window *window = NULL;
  	SDL_Renderer *render = NULL;
		SDL_Init(SDL_INIT_VIDEO);
  	window = SDL_CreateWindow("SDL2 Window", 200, 200, 640, 480, SDL_WINDOW_SHOWN | SDL_WINDOW_BORDER);
  	if(!window){
      	prinf("Failed to Create window.");
      	goto __EXIT;
    }
  	render = SDL_CreateRenderer(window, -1, 0);
  	if(!render){
      	SDL_Log("Failed to create render.");
      	goto __DWINDOW;
    }
  	// 创建纹理
  	texture = SDL_CreateTexture(render, SDL_PIXELFORMAT_RGBA8888, SDL_TEXTUREACCESS_TARGET, 640, 480);
  	if(!texture){
      	SDL_Log("Failed to Create Texture.");
      	goto __RENDER;
    }
  	do{
      	// 在窗口内实现一个跳来跳去的方块
      	SDL_PollEvent(&event);
      	switch(event.type){
          case SDL_QUIT:
            quit = 0;
            break;
          default:
            SDL_Log("event type is %d", event.type);
        }
      	rect.x = rand() % 600;
      	rect.y = rand() % 450;
      	// 对纹理填充颜色
      	SDL_SetRenderTarget(render, texture);
        SDL_SetRenderDrawColor(render, 0, 0, 0, 0);
        SDL_RenderClear(render);
      	// 对方块填充颜色
        SDL_RenderDrawRect(render, &rect);
        SDL_SetRenderDrawColor(render, 255, 0, 0, 0);
      	SDL_RenderFillRect(render, &rect);
      	// 将纹理交给显卡
      	SDL_SetRenderTarget(render, NULL);
      	SDL_RenderCopy(render, texture, NULL, NULL);
      	// 将纹理渲染到窗口
      	SDL_RenderPresent(render);
    }while(quit);
  	// 销毁纹理
  	SDL_DestoryTexture(texture);
__RENDER:
  	SDL_DestoryRenderer(render);
__DWINDOW:
  	SDL_DestoryWindow(window);
__EXIT:
  	SDL_Quit();
		return 0;
}
```

##### 实战：YUV视频播放器

- 创建线程

  - SDL_CreateThread()

    参数 fn：线程执行函数
    
    参数 name：线程名
    
    参数 data：执行函数参数

- SDL更新纹理

  - SDL_UpdateTexture()
  - SDL_UpdateYUVTexture()

```c
// vi yuv_player.c
// clang -g -o yuv_player yuv_player.c `pkg-config --cflags --libs sdl2`
// ./yuv_player
```

##### SDL播放音频

播放音频基本流程：打开音频设备，设置音频参数，向声卡喂数据，播放音频，关闭设备

播放音频的基本原则：

- 声卡向你要数据而不是你主动推给声卡
- 数据的多少由音频参数决定的

SDL 音频API

- SDL_OpenAudio / SDL_CloseAudio
- SDL_PauseAudio
- SDL_MixAudio

##### 实战：实现PCM播放器

```c
// vi pcm_player.c
// clang -g -o pcm_player pcm_player.c `pkg-config --cflags --libs sdl2`
// ./pcm_player
#include <SDL.h>
#include <stdio.h>

#define BLOCK_SIZE 4096000
static size_t buffer_len = 0;
static Uint8 *audio_buf = NULL;
static Uint8 *audio_pos = NULL;

void read_audio_data(void *udata, Uint8 *stream, int len){
    if(buffer_len == 0){
      return;
    }
  	// 清空SDL缓冲区
    SDL_memset(stream, 0, len);
    len = (len < buffer_len) ? len : buffer_len;
  	// 拷贝数据
    SDL_MixAudio(stream, audio_pos, len, SDL_MIX_MAXVOLUME);
    audio_pos += len;
    bufer_len -= len;
}

int main(int argc, char* argv[]){
    int ret = -1;
    char *path = "1.pcm";
    FILE* audio_fd = NULL;
  	// SDL初始化
    if(SDL_Init(SDL_INIT_AUDIO | SDL_INIT_TIMER)){
        SDL_Log("Failed to initial.");
        return ret;
    }
  	// 打开pcm文件
    audio_fd = fopen(path, "r");
    if(!audio_fd){
        SDL_Log("Failed to open pcm file.");
        goto __FAIL;
    }
  	// 分配空间
    audio_buf = (Uint8*)malloc(BLOCK_SIZE);
    if(!audio_buf){
        SDL_Log("Failed to alloc memory.");
        goto __FAIL;
    }
    SDL_AudioSpec spec;
    spec.freq = 44100;
  	// 通道数
    spec.channels = 2;
  	// 采样大小
    spec.format = AUDIO_S16SYS;
    spec.silence = 0;
  	// 回调函数
    spec.callback = read_audio_data;
    spec.userdata = NULL;
  	// 打开音频设备
    if(SDL_OpenAudio(&spec, NULL)){
        SDL_Log("Failed to open audio device.");
        goto __FAIL;
    }
  	// 音频设备开始播放
    SDL_PauseAudio(0);
  	// 从pcm中读取数据放入buf
    do{
      	// 从文件中读取BLOCK_SIZE大小的数据到audio_buf
        bufer_len = fread(audio_buf, 1, BLOCK_SIZE, audio_fd);
        audio_pos = audio_buf;
        while(audio_pos < (audio_buf + bufer_len)){
          	SDL_Delay(1);
        }
    }while(bufer_len != 0)
    SDL_CloseAudio();
    ret = 0;
__FAIL:
    if(audio_buf){
       free(audio_buf);
    }
    if(audio_fd){
       fclose(audio_fd);
    }
  	// 退出SDL
    SDL_Quit();
    return ret;
}
```
