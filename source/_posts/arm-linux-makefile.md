---
title: RXTX库文件交叉编译
date: 2019-03-16 14:30:00
categories: 
- arm
tags:
- arm-linux
- rxtx
- cross-compile
---

本来arm-linux下的串口通信，理所应当该是C来编写。考虑到公司成员的技术栈，继续沿用之前在linux和window系统下使用的rxtx库。

<!--more-->

# 背景

rxtx官方的思路也是通过jni调用c的串口操作库实现。 官网上提到可以支持android的arm系统，但是没有给出对应的库文件。

所以下载官网上rxtx的源码，交叉编译生成库文件，将jar包和库文件放到arm系统下运行，不报错，但是找不到串口。具体是什么问题还在调查中，暂时先记录下交叉编译过程中遇到的一些问题。


# 问题（坑）

## arm-linux
1. 下载源文件后 ./configure
2. 进入Makefile，把所有gcc改成arm-linux-gcc
3. 发现很多头文件没有包含，加include目录

明明在Makefile文件中已经没有gcc了，make过程中后台打印还是有gcc

解决办法：

```bash
./configure --host=arm-linux
```

不止解决了gcc的问题，同时include的问题也解决了

--host=HOST
指定软件运行的系统平台.如果没有指定,将会运行'config.guess'来检测。

PS. 运行无效的问题考虑configure的参数设置，暂无结论。


## UTS_RELEASE

解决了交叉编译平台类型的问题，make的时候还是会报类似如下错误

```bash
错误1：/tmp/rxtx-2.1-7r2/./src/I2CImp.c:135: error: ‘UTS_RELEASE’ undeclared (first use in this function)
```

这是由于version.h中缺少’UTS_RELEASE’信息，需要手工添加。先获取当前系统的版本信息：

```bash
uname -r
```

由于交叉编译环境和最终运行环境是不同的，这里我理解应该是运行环境的系统版本信息。
打开configure.h 或者 include/linux目录下的version.h 对应版本号定义
```bash
#define UTS_RELEASE "3.10.24+"
```

# 正确流程
1. 下载源文件，解压
```bash
unzip ***.zip
```
2. 进入目录运行configure指令
```bash
cd ***
./configure --host=arm-linux
```
3. 定义版本号
4. make
