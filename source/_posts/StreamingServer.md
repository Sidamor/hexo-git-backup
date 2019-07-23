---
title: 流媒体服务器技术调研
date: 2019-07-12 11:02:27
categories: 
- StreamingServer
tags:
- rtsp
- rtmp
- flv
- hls
---

2B的物联网，好像怎么都绕不开流媒体。 从十年前，基于各个厂商的sdk开发ocx控件，在IE下查看实时、历史视频。

到了后来，可以在各种内核的浏览器中嵌入VLC的播放器，直接播放rtsp视频流。可惜好景不长，各个浏览器突然就停止了对于VLC控件的支持（基于NPAPI）。

之后，又到了各个厂家提供可以“嵌入”浏览器的插件（实质还是播放rtsp视频流）的现在。

跨操作系统（LINUX/WIN/IOS/ANDROID）、跨浏览器（SAFRI/CHROME/IE/FIREFOX），播放端使用通用的<video>标签，或者flash就可以播放，这是我想要做的一个解决方案。

<!--more-->

# flash

[locomote@github](https://github.com/AxisCommunications/locomote-video-player) 可以实现 
But，出于安全考虑，从 Flash 10 开始，需要一个 Socket Policy Server 来“授权”，才能允许 Flash 读取其他服务器上的内容，而且你读的视频流在哪个IP地址，Socket Policy Server 也必须在那个服务器上（默认843端口），而摄像头显然没有条件让你在上面跑另外一个服务。

## 实测: 

```json
	{message: "Socket reported a security error: Error #2048.", code: 731}
```

locomote-video-player上的issue回答:
If you can't add a socket policy server on the RTSP server, you will not be able to use Locomote to play RTSP from that server.

显然，指望厂商在IPC或者NVR上加这个policy server不现实，这个方案需要自己搭建一个流媒体服务器，对于厂家的rtsp进行转发，同时，在流媒体服务器上增加一个policy服务

## 解决方案（731）

增加policy服务之后：

```json
	{message: "RTSPClient: Unable to determine control URL.", code: 824}
```

locomote-video-player上的 issue#117 回答:
missing the protocol "rtsp://" into the "Content-base" header

## 解决方案（824）：

1. RTSP服务器返回头，Content-base 加上rtsp://

需要在RTSP服务器端做修改，不同厂商、不同型号，较难做到。

2. 修改locomote-video-player源码重新编译，解析RTSP返回头时，支持不带rtsp://的Content-base

locomote源码中,http/url.as里的function isAbsolute校验,返回true
根据[locomote@github](https://github.com/AxisCommunications/locomote-video-player)中Building Locomote章节描述，重新编译Player.swf

重新编译后，发现EasyDarwin返回的content-base居然是null，

注意：
* 先运行 npm install gulp，安装gulp，再通过gulp进行编译(gulpfile.js)
* 需要安装[ant-apache](https://ant.apache.org/bindownload.cgi) 镜像选择国内的，安装完配置到PATH中
* Flex SDK不支持64位的jvm.dll(mxmlc Error loading: jre\bin\server\jvm.dll)，装好后改\node_modules\flex-sdk\lib\flex_sdk\bin\jvm.config的java.home参数
* windows下Flex报错乱码，无法定位错误原因,
* nvm切换到7.9.0版本，Ubuntu编译顺利通过

Ubuntu下编译步骤：
* `git clone https://github.com/AxisCommunications/locomote-video-player.git`
* 安装nvm管理nodejs版本
`curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash`
* `npm install -g gulp`
* `npm install`
* `apt install ant`
* `gulp`

参考链接：
[几种浏览器播放RTSP视频流的方案](https://bnlt.org/2019/%E5%87%A0%E7%A7%8D%E6%B5%8F%E8%A7%88%E5%99%A8%E6%92%AD%E6%94%BERTSP%E8%A7%86%E9%A2%91%E6%B5%81%E7%9A%84%E6%96%B9%E6%A1%88/)
[locomote@github](https://github.com/AxisCommunications/locomote-video-player)
[Setting up a socket policy file server](https://www.adobe.com/devnet/flashplayer/articles/socket_policy_files.html)

# 流媒体技术调研阶段总结：
1. 各大厂商IPC提供的流媒体格式普遍都只有RTSP
2. 海康中心服务器提供的RTSP为私有协议，不是标准的RTSP，仅支持海康提供的SDK/控件
3. 浏览器对于RTSP的支持度为0
3.1 VLC控件，基于NPAPI，各大浏览器已经取消了对于NPAPI的支持
3.2 FLASH, 9之后，对于socket通信，增加了policyfile认证要求，需要再IPC端增加policy server。IPC的操作系统不开放，在实施过程中难度过大。
4. 浏览器现支持RTMP\HLS

总结：需要搭建流媒体服务器，对于RTSP流进行转换，提供RTMP/HLS格式。

# 开源的解决方案

看了一圈开源的解决方案，解决了一些转码的问题，界面都不太友好。

一些集群的方案也都是收费的。

还是考虑自己搭建一个流媒体服务器，转码+管理平台。

管理平台不难做，转码部分考虑用ffmpeg。毕竟从头开始看RTSP还是需要一定时间成本的。

# ffmpeg + nginx

* 服务器安装ffmpeg环境，java代码可以命令行调用ffmpeg，对于指定rtsp进行转码，推送到nginx指定地址。指定rtsp及指定nginx地址由平台进行配置。

* 当前端发起一个预览请求，后端起一个ffmpeg实例，进行转码。同时，前端取指定rtmp地址获取转码后的视频流。

* java执行ffmpeg命令使用开源项目[FFCH4J](https://github.com/eguid/FFCH4J)

* rtmp播放要使用videojs 5.x的版本，或者videojs7配合videojs-flash.min.js可以播放

* RTMP延迟2-3s `ffmpeg -i rtsp://admin:a1234567@10.30.30.124:554/h264/ch33/sub/av_stream -vcodec libx264 -acodec aac -f flv rtmp://192.168.10.87:1935/live/home`

```html
<!DOCTYPE html>
<html>
    <head>
        <title>播放器</title>
        <!-- 导入的videojs是7.0版本以上的，集成VHS协议库，可播放HLS流媒体视频 -->
        <link href="https://unpkg.com/video.js/dist/video-js.css" rel="stylesheet">
        <script src="https://unpkg.com/video.js/dist/video.js"></script>
        <!-- 引入的videojs-flash.js插件主要是为播放rtmp视频流-->
        <script src="https://cdn.jsdelivr.net/npm/videojs-flash@2/dist/videojs-flash.min.js"></script>
    </head>
    <body>
        <video id='myvideo' width=960 height=540 class="video-js vjs-default-skin" controls>
            <!-- RTMP直播源地址-->
            <source src="rtmp://127.0.0.1:1935/live/home">    
        </video>
        <script> 
            var player = videojs('myvideo', {}, function(){console.log('videojs播放器初始化成功')})
            player.play();
        </script>
    </body>
</html>
```


* HLS，文件切片，延迟>30S    
`ffmpeg -i rtsp://admin1:abc123456@10.30.30.100:554/cam/realmonitor?channel=1&subtype=0 -vcodec libx264 -codec:a libfaac -map 0 -f hls -hls_time 10 playlist.m3u8`



# 根据RTSP协议标准进行转码

熟悉RTSP&RTMP&HLS协议标准，基于socket获取RTSP码流，转换成RTMP/HLS，推送到指定地址。


