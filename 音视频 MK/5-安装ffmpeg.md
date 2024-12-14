### FFmpeg都能做啥？

- FFmpeg是一个非常优秀的多媒体框架
- FFmpeg可以运行在Linux、Mac、Windows等平台上
- 能够解码、编码、转码，复用，解复用，过滤音视频数据

### brew 安装 ffmpeg安装

方便但是不可以改源码，不够灵活。前期推荐

```shell
brew search ffmpeg
brew update
brew install ffmpeg
ffmpeg -version
```

### 源码安装 ffmpeg安装

```shell
ssh root@106.14.118.72
# 下载ffmpeg：http://ffmpeg.org/download.html
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
# 查看配置选项
# more：空格翻页 PageUp：向上翻页
./configure --help | more
# 查看解码器
./configure --list-decoders
./configure --list-filters
./configure --help | grep static 
./configure --help | grep share
# 编译安装ffmpeg
./configure --prefix=/usr/local/ffmpeg --enable-debug=3
# 编译安装ffmpeg：关闭静态库，打开动态库
./configure --prefix=/usr/local/ffmpeg --enable-debug=3 --disable-static --enable-shared
make -j 4 && make install
cd /usr/local/ffmpeg && ls
# 配置环境变量
vi ~/.bash_profile
# 添加
export PATH="/usr/local/ffmpeg/bin:$PATH"
source ~/.bash_profile
```

##### YUM安装 ffmpeg

```shell
# 升级系统
sudo yum install epel-release -y
sudo yum update -y
# 安装 Nux Dextop Yum 源
sudo rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
sudo rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
# 安装
sudo yum install ffmpeg ffmpeg-devel -y
ffmpeg -version
```

