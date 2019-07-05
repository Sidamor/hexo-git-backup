---
title: bios蓝图-vol2
date: 2019-07-05 10:21:10
categories: 
- bios
tags:
- bios
- design
---

接上篇，既然是操作系统，肯定不能像之前传统的项目落地那样。

可视化、服务接入、应用授权、服务管理、控制面板、桌面系统等等，一个都不能少。

<!--more-->

# 使用场景

* bios可以搭建在私有云、公有云、混合云上
* bios操作系统的使用者，分为：开发者、实施人员、管理人员、最终用户

## 开发者

bios作为一个开放的操作系统，允许所有开发者发布自己的应用/服务到市场上，实施人员可以根据指引部署自己需要的应用/服务。

使用场景：

* 开发服务（仅提供接口）、应用（包含界面），发布到应用市场
* 获取日志
* 查看服务运行状态

## 实施人员

作为2b的操作系统，每个项目落地都需要有实施人员去实施
经过简单的培训，实施人员就可以在bios上接入所有设备；部署需要的应用/服务

使用场景：

* 使用后端组态、专家系统接入设备（点集）
* 将点集与设备（资产）进行绑定
* 部署需要的应用/服务（自动化部署->可视化部署）
* 服务间的依赖性确认，API调用授权确认
* 使用前端组态搭建管理人员、用户需要的桌面应用

## 管理人员

项目实施完毕后，交付甲方使用，很多用户配置、部门配置、权限分配的事情。会提供一个超级管理员，交由甲方来做这些事情。

使用场景：

* 配置部门
* 添加人员（部门信息、手机号、个人信息）
* 分配功能权限（应用，以及应用中的子功能）
* 分配设备权限（对于点的读取、控制权限）

## 最终用户

每个用户拿到自己账户后可以开始使用bios操作系统下的功能了

使用场景：

* 停车、能耗、物业费等应用
* 入口包括B/S页面、APP、小程序等


# 操作系统的安装

bios操作系统本身的安装是一个课题，公有云可以由我们进行安装；私有云和混合云，是和服务器厂商合作，进行系统预装，或者是提供安装包（安装过程可能过于复杂）

* openstack（私有云）
* openshift
* k8s（服务集群）
* istio（集群管理）

以上这些，安装过程可能过于复杂。 可以考虑和服务器厂商合作，出厂前进行系统预装。


# 功能 & 服务

* 实施人员：控制面板、服务管理
* 管理人员&用户：桌面应用

![BI-OS.png](BI-OS.png)

详细描述：

![table.png](table.png)


# 部署方案

严格意义上来说，这里所谓私有云，是混合云。

我们需要监控操作系统的运行状态，会在公有云上部署我们自己的运营平台。

而这里所谓的混合云，是指将一些服务放到公有云上来。
适用于一些体量较小的项目，资源占用不大，没必要在私有云上部署一些大体量的服务。

* 数据服务（MySql, mongodb, Redis)
* 消息服务（Kafka, Redis, mqtt)
* BI