#### CDN 实现商业娱乐直播

直播产品的种类

- 泛娱乐化直播

  花椒、映客等娱乐直播，还有斗鱼、虎牙等游戏直播

- 实时互动直播

  音视频会议、教育直播等，像Zoom、声网

##### Nginx RTMP 环境搭建

- 准备流媒体服务器（Linux或Mac）

  - 下载nginx源码 

    http://nginx.org/en/download.html

  - 下载nginx-rtmp-module 

    https://github.com/arut/nginx-rtmp-module.git
    
  - 下载openssl

    https://www.openssl.org/

    https://github.com/openssl/openssl

- 编译并安装Nginx

  ```shell
  tar -zxvf nginx-1.17.7.tar.gz
  ./configure --prefix=/usr/local/nginx --add-module=../nginx-rtmp-module --with-openssl=../openssl
  vi Makefile
  make -j 4 && make install
  ```

- 配置RTMP服务并启动Nginx服务

  ```shell
  vim conf/nginx.conf
  rtmp{
  	server{
  		listen 1935;
  		chunk_size 4000;
  		application live{
  			# 直播
  			live on;
  			# 允许直接播放
  			allow play all;
  		}
  	}
  }
  sudo ./sbin/nginx -c ./conf/nginx.conf
  ps -ef | grep nginx
  netstat -an | grep 1935
  ```

- 测试

  ```shell
  ffplay rtmp://localhost/live/room
  ffmpeg -re -i xxx.flv -c copy -f flv rtmp://localhost/live/room
  ```

#### SRS

https://github.com/ossrs/srs

Simple Rtmp Server，它是单进程实现的。在同一台服务器上可以启动多个进程同时提供服务。

它的定位是运营级的互联网直播服务器集群，它提供了非常丰富的接入方案，支持RTMP、HLS、HTTP-FLV等。

SRS部署：

```shell
git clone https://github.com/ossrs/srs.git
cd srs/trunk
./configure --prefix=/usr/local/srs
make -j 4
make install
cd /usr/local/srs
./objs/srs -c conf/srs.conf
ps -ef | grep srs
netstat -ntpl | grep 1935
ffplay rtmp://localhost/live/room
ffmpeg -re -i xxx.flv -c copy -f flv rtmp://localhost/live/room
```

Vhost作用：rtmp://vhost/Live/stream

- 支持多客户
- 支持多配置
- 域名调度

```shell
listen 1935;
vhost show.cctv.cn{
}
vhost show.mgtv.cn{
}
```

SRS 集群部署：

```shell
1:
cd /usr/local/srs
ps -ef | grep srs
vi ./conf/origin.nocluster.edge.conf
./objs/srs -c ./conf/origin.nocluster.edge.conf

2:
cd /usr/local/srs
ps -ef | grep srs
vi ./conf/origin.conf
./objs/srs -c ./conf/origin.conf

3:
ffplay rtmp://learningrtc1.cn/live/room
4:
ffmpeg -re -i xxx.flv -c copy -f flv rtmp://learningrtc2/live/room
```

##### 什么是CND网络

CDN：Content Delivery Network 即内容分发网络。

最初的目的是解决静态页面的加速问题

通过就近接入方法解决访问网络资源的问题

阿里云视频直播步骤：

1. 注册阿里账户
2. 开通视频直播业务
3. 购买流量资源包
4. 设置推拉流域名
5. 配置CNAME
6. 生成推拉流地址
