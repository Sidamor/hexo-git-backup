---
title: 智慧园区 3D可视化UED设计
date: 2019-02-15 13:43:10
categories: 
- bios
tags:
- bios
- 3d
- ued
---

商业操作系统的智慧园区部分，前端设计。传统的2d或者2.5d&图表的展现形式，被客户戏称为20年前的产品。
为了改变客户的这一印象，我们决定更进一步，向真正的3d可视化努力。

<!--more-->


# 需求模块划分
展现形式改变了，但是原有的智慧园区的产品功能一样都不能少。

* 概览
* 告警
* 能耗
* 楼宇运营
* 人员管理
* 停车场管理
* 照明 
* 电梯
* 消防
* 监控
* 门禁
* 给排水
* 除湿机
* UPS

# 3D可视化功能设计

![design](design.jpg)

如上图所示，
顶部为LOGO、时间、天气，以及选中详情展示的简介。
左侧为每个功能模块的概览，中间为大楼3D可视化展示部分，右侧为详情展示。

当选中左侧相应功能模块时，3D展示切换到对应视角，展示对应效果；右侧详情展示对应详情信息。

## 能耗概览

![energy](energy.png)

![energy2](energy2.png)

选中该模块时，3D展示切换到能耗视图，展示园区能耗情况。
详情页沿用原先能耗管理模块页面。

## 停车概览

![parking](parking.png)

选中该模块时，3D展示切换到地下+地面停车场视图，展示园区停车情况。
详情页沿用原先停车管理模块页面。

## 电梯概览

![elevator](elevator.png)

选中该模块时，3D展示切换到电梯视图，展示电梯状态、当前楼层以及运行方向。
详情页沿用原先电梯管理模块页面。

## 告警概览

选中该模块时，3D展示切换到告警视图，高亮&闪烁告警位置。
详情页沿用原先告警管理模块页面。


## 楼宇运营

![rent1](rent1.png)

![rent2](rent2.png)

选中该模块时，用户可以在中间做3D可视化交互
* 选中对应楼层查看租赁情况等
* 选中对应租户显示对应楼层、区域

详情页沿用原先电梯管理模块页面，另外新增楼层3D图，并高亮租赁区域

## 人员管理

![people](people.png)

选中该模块时，3D展示切换到人员管理视图，dashbord显示最近进出人员头像。
详情页沿用原先人员管理模块页面。