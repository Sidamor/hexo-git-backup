---
title: Wechaty Vol.1
date: 2017-01-17 09:57:34
categories: 
- Wechaty
tags:
- Docker
- Wechaty
---

生活中有人提到，想要把关注公众号的人自动拉到一个群里。
刚好前段时间在github上看到一个项目wechaty可以实现这个功能。
花时间来研究一下，顺便学习了Docker等一些工具的用法。
<!--more-->
# 总体设计
功能是需要做到长时间在线，故决定部署在云端Linux服务器上。

# 环境搭建 
## Docker
* [Docker for Linux](https://docs.docker.com/engine/installation/linux/) 网站似乎有点卡，可能要翻墙
之前做了windows下的搭建，看来Docker在安装包部分都做了封装，针对每一个系统。
* Linux内核需要3.1以上
* 以下为ubuntu系统下安装 其他系统参考官方文档
```bash
	apt-get update
	apt-get install docker
```

# 程序编写
## 源程序下载
```bash
	git clone https://github.com/lijiarui/wechaty-getting-started.git
	cd wechaty-getting-started
	docker run -ti --volume="$(pwd)":/bot --rm zixia/wechaty mybot.ts
```
* 实例程序测试通过
* 这里securecrt不能显示出二维码打印，可用XShell5替换做后台操作IDE

## 程序修改
在好友申请自动申请的部分，加入自动发送群邀请的代码
```bash
.on('friend', function(contact, request) {
    if (request) {
        request.accept().then(function() {
            console.log(`Contact: ${contact.name()} send request ${request.hello}`)
            Room.find({ topic: "test" }).then(function(keyroom) {
            if (keyroom) {
                keyroom.add(contact).then(function() {
                    keyroom.say("welcome!", contact)
                })
            }
        		})
        })
    }
})
```

# 运行测试
好友申请自动通过 -> 自动发送群邀请 -> 发送欢迎信息
测试通过