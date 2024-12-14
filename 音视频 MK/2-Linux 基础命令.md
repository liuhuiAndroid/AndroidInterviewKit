基本命令

```shell
ls
ls -a
ls -alt
cd 
cd - # 回去
pwd
mkdir 
mkdir -p 1/2/3
cp 
rm 
rm -rf
sudo 
pkg-config
echo "123" > test.txt
```

安装工具

```shell
# ubuntu
apt
# mac
brew
brew search vim
brew install vim
brew remove vim
# centos
yum
```

```shell
# mac 下查看环境变量
env 
# PATH：可执行的二进制文件，添加进去后可以作为命令调用
env | grep PATH
vi ~/.bash_profile
which ls
# PKG_CONFIG_PATH：用于放置.pc库
vi ~/.bash_profile
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/ffmpeg/lib/pkgconfig/
source ~/.bash_profile
pkg-config --libs --cflags libavutil
# LD_LIBRARY_PATH：用于放置.so库
# PKG_CONFIG_PATH 与 LD_LIBRARY_PATH 区别
```



测试环境变量有没有生效

```cpp
// vi test.c
// gcc -g -o test test.c `pkg-config --libs --cflags libavutil`
#include <stdio.h>
#include <libavutil/avutil.h>

int main(int argc, char* argv[]){
  	return 0;
}
```

