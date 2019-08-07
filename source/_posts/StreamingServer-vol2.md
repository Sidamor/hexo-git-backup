---
title: 流媒体服务器实践
date: 2019-08-02 16:12:21
categories: 
- StreamingServer
tags:
- rtsp
- rtmp
- flv
- hls
---

* 各大浏览器取消了对于VLC控件的支持（基于NPAPI的一切控件方案 X ）
* 从 Flash 10 开始，需要一个 Socket Policy Server 来“授权”，才能允许 Flash 读取其他服务器上的内容，而且你读的视频流在哪个IP地址，Socket Policy Server 也必须在那个服务器上（默认843端口），而摄像头显然没有条件让你在上面跑另外一个服务。 so,基于Flash解码的方案 X

思考：基于js对于rtsp协议进行解码可行吗？ 限于对js和rtsp了解都不深入，这个方案先放一放，回头讨论（想要学flv.js，通过js对rtsp协议进行解码）。

总结：搭建流媒体服务器，进行转码、推流

<!--more-->

# 技术架构

思路：

1. nginx搭建流媒体服务器  compiled with [nginx-http-flv-module](https://github.com/winshining/nginx-http-flv-module/blob/master/README.CN.md)
2. 使用ffmpeg对于摄像头的标准rtsp流进行解码，推送到nginx上
3. 浏览器从nginx上获取rtmp视频流，或者http-flv视频流

![技术架构图](architect.png)


# nginx搭建流媒体服务器

采用了[nginx-http-flv-module](https://github.com/winshining/nginx-http-flv-module/blob/master/README.CN.md)，相比[nginx-rtmp-module](https://obsproject.com/forum/resources/how-to-set-up-your-own-private-rtmp-server-using-nginx.50/)，不仅增加了对于http-flv的支持，还有一些配置上的优化。

|       功能       | nginx-http-flv-module | nginx-rtmp-module |                  备注                  |
| :--------------: | :-------------------: | :---------------: | :------------------------------------: |
| HTTP-FLV (播放)  |           √           |         x         |        支持HTTPS-FLV和chunked回复      |
|     GOP缓存      |           √           |         x         |                                        |
|     虚拟主机     |           √           |         x         |                                        |
| 省略`listen`配置 |           √           |       见备注      |        配置中必须有一个`listen`        |
|    纯音频支持    |           √           |       见备注      | `wait_video`或`wait_key`开启后无法工作 |
| 定时打印访问记录 |           √           |         x         |                                        |
|  JSON风格的stat  |           √           |         x         |                                        |



## centos7上的安装

* 安装

    yum install https://extras.getpagespeed.com/release-el$(rpm -E %{rhel})-latest.rpm
    yum install nginx-module-flv

修改/etc/nginx/nginx.conf配置，其中有可能要个性化配置的几个点
1. https支持，端口修改
2. http中，直播的location我选择了/live
3. rtmp中，application的名字选择
```conf
#worker_processes  1; #运行在Windows上时，设置为1，因为Windows不支持Unix domain socket
worker_processes  auto; #1.3.8和1.2.5以及之后的版本

#worker_cpu_affinity  0001 0010 0100 1000; #只能用于FreeBSD和Linux
worker_cpu_affinity  auto; #1.9.10以及之后的版本

error_log logs/error.log error;

#如果此模块被编译为动态模块并且要使用与RTMP相关的功
#能时，必须指定下面的配置项并且它必须位于events配置
#项之前，否则NGINX启动时不会加载此模块或者加载失败

load_module modules/ngx_http_flv_live_module.so;

events {
    worker_connections  4096;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    keepalive_timeout  65;

    server {
        listen       80;

        location / {
            root   /var/www;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        location /live {
            flv_live on; #打开HTTP播放FLV直播流功能
            chunked_transfer_encoding on; #支持'Transfer-Encoding: chunked'方式回复

            add_header 'Access-Control-Allow-Origin' '*'; #添加额外的HTTP头
            add_header 'Access-Control-Allow-Credentials' 'true'; #添加额外的HTTP头
        }

        location /hls {
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }

            root /tmp;
            add_header 'Cache-Control' 'no-cache';
        }

        location /dash {
            root /tmp;
            add_header 'Cache-Control' 'no-cache';
        }

        location /stat {
            #push和pull状态的配置

            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }

        location /stat.xsl {
            root /var/www/rtmp; #指定stat.xsl的位置
        }

        #如果需要JSON风格的stat, 不用指定stat.xsl
        #但是需要指定一个新的配置项rtmp_stat_format

        #location /stat {
        #    rtmp_stat all;
        #    rtmp_stat_format json;
        #}

        location /control {
            rtmp_control all; #rtmp控制模块的配置
        }
    }
}

rtmp_auto_push on;
rtmp_auto_push_reconnect 1s;
rtmp_socket_dir /tmp;

rtmp {
    out_queue           4096;
    out_cork            8;
    max_streams         128;
    timeout             15s;
    drop_idle_publisher 15s;

    log_interval 5s; #log模块在access.log中记录日志的间隔时间，对调试非常有用
    log_size     1m; #log模块用来记录日志的缓冲区大小

    server {
        listen 1935;
        server_name www.test.*; #用于虚拟主机名后缀通配

        application myapp {
            live on;
            gop_cache on; #打开GOP缓存，减少首屏等待时间
        }

        application hls {
            live on;
            hls on;
            hls_path /tmp/hls;
        }

        application dash {
            live on;
            dash on;
            dash_path /tmp/dash;
        }
    }

    server {
        listen 1935;
        server_name *.test.com; #用于虚拟主机名前缀通配

        application myapp {
            live on;
            gop_cache on; #打开GOP缓存，减少首屏等待时间
        }
    }

    server {
        listen 1935;
        server_name www.test.com; #用于虚拟主机名完全匹配

        application myapp {
            live on;
            gop_cache on; #打开GOP缓存，减少首屏等待时间
        }
    }
}
```

## 其他unix系统

在centos7上也尝试了手动编译nginx，增加了nginx-http-flv-module的依赖

1. 系统安装 gcc pcre pcre-dev openssl-dev这几个依赖项
2. 下载[nginx源码包](http://nginx.org/en/download.html)
3. 下载[nginx-http-flv-module源码包](https://github.com/winshining/nginx-http-flv-module)
4. 解压到同一目录下，进入nginx源码包目录
5. 编译（默认两个包解压到同一个目录下）
```bash
./configure --add-module=../nginx-http-flv-module
make
make install
```
过程中可能会缺少1中所述的依赖包，根据操作系统情况进行安装就行了

* nginx安装完毕，启动

```bash
nginx
```

* 从外部检查nginx的端口1935，80是否开放。原因可能有
1. nginx.conf配置问题
2. 操作系统防火墙没有打开



# ffmpeg进行转码、推流

```bash
ffmpeg -i "rtsp://admin1:abc123456@10.30.30.100:554/cam/realmonitor?channel=1&subtype=0" -vcodec libx264 -acodec aac -f flv rtmp://192.168.10.36:1935/myapp/home
```

1. rtsp地址外加引号，因为&可能影响命令的识别
2. 推流地址中，url第一个目录为application配置的地址，第二个为流地址，可以自定义，每一个不同就行了

PS
1. 如果取流、推流地址都是确定的，可以将ffmpeg在nginx中配置死
2. 我们的需求是可以配置流媒体服务器，所以取流、推流的地址都需要改变
3. 用java调用命令行的方式(exec)，执行ffmpeg转码，从而灵活配置参数。github上有可用的开源项目[FFCH4J](https://github.com/eguid/FFCH4J)，感觉作用不大，可以自己写。

PPS
现在的直播延迟有点大(10s+)，ffmpeg的转码效率应该可以进行优化，待研究。


## ffmpeg最新版安装 

[centos下]https://trac.ffmpeg.org/wiki/CompilationGuide/Centos




## 参数调优

1. 码流类型subtype可以选择子码流，速度大大加快
2. 开启x264的 -preset fast/faster/verfast/superfast/ultrafast参数
3. 使用-tune zerolatency 参数
4. 码率控制，官方推荐-b和-bufsize搭配使用  -b:v 2000k -bufsize 2000k


当前最优参数（3-5s延迟）
```bash
ffmpeg -i "rtsp://admin:abc123456@hua@10.30.30.109:554/cam/realmonitor?channel=1&subtype=1" -tune zerolatency -vcodec libx264 -preset ultrafast -acodec aac -f flv rtmp://192.168.10.36:1935/myapp/home
```



# 播放端

* rtmp
```url
rtmp://192.168.10.36:1935/myapp/home
```

1. myapp是nginx.conf中，rtmp->application
2. home是stream的id，可以自定义，不重复就行

* http-flv
```url
http://192.168.10.36/live?app=myapp&stream=home

```

1. live是nginx.conf中，http->server->location，里面配置了flv_live on
2. 参数中，app对应rtmp的application
3. 参数中，stream对应ffmpeg推流的流id


## VLC

实测通过， 10s+延迟

## 浏览器 video.js

未测

## 浏览器 flv.js

未测