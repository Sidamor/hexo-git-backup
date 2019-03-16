---
title: bios-开放框架-II
date: 2019-02-18 09:51:12
categories: 
- bios
tags:
- bios
- openframe
---

上一篇调研的记录有点凌乱，另开一篇进行整理。

<!--more-->

# 简要总结
在整个开放框架中，存在三个角色：开放框架、服务提供商、消费者。

明确一下三个角色分别要做什么。

## 开放框架要做什么

* 服务配置页面，服务提供商将自己的服务公布在开放框架
* 用户注册页面，消费者需要在bios注册一个自己的账户
* 服务调用情况上报，维护一个消息队列，接收服务调用情况的上报

PS.以后可能还需要完善的

* 封装服务提供商搭建服务的API类库
* 封装消费者调用服务的API类库
* 封装服务调用情况上报的类库
* 服务是否可用的检测

## 服务提供商要做什么

* 服务搭建
* 在bios注册自己的服务
* 对于用户调用进行鉴权
* 上报服务调用情况

## 消费者要做什么

* 在bios注册，获取调用服务权限
* 在bios获取服务列表
* 调用服务

![brief](brief.jpg)

# 详细说明

针对之前介绍到的功能：服务注册、用户注册、获取服务列表、调用服务、上报服务调用，作一个详细说明。

## 服务注册
服务提供商根据物模型规范填写服务详情（Json格式），提交到开放平台。
这一流程考虑在开放平台形成注册页面，填写相应字段，由平台生成json格式服务定义。

<center>产品注册页面</center>
![产品注册页面](product.png)

```json
{
    "link":"产品服务调用地址",
    "profile":{
        "identifier":"产品id",
        "name":"产品名称",
        "desc":"产品描述"
    }
}
```

<center>属性注册页面</center>
![属性注册页面](property.png)

```json
{
    "productId":"产品ID",
    "accessUri":"访问地址",
    "accessType":"访问方式 get、post",
    "accessMode":"属性读写类型，只读(r)，读写(rw)",

    "identifier":"属性唯一标识符(产品下唯一)",
    "name":"属性名称",    
    "required":"是否是标准功能的必选属性",
    "dataType":{
        "type":"属性类型: int(原生)，float(原生)，double(原生), text(原生)，date(String类型UTC毫秒)，bool(0或1的int类型)，enum(int类型), struct(结构体类型，可包含前面6种类型)，array(数组类型，支持int/double/float/text)",
        "specs":{
            "min":"参数最小值(int, float, double类型特有)",
            "max":"参数最大值(int, float, double类型特有)",
            "unit":"属性单位",
            "unitName":"单位名称",
            "size":"数组大小，默认最大128(数组特有)",
            "item":{
                "type":"数组元素的类型"
            }
        }
    }
}
```

<center>服务注册页面</center>
![服务注册页面](service.png)

```json
{
    "productId":"产品ID",
    "accessUri":"访问地址",
    "accessType":"访问方式 get、post",

    "identifier":"服务唯一标识符(产品下唯一，产品下唯一，其中set/get是根据属性的accessMode默认生成的服务)",
    "name":"服务名称",
    "desc":"服务描述",
    "required":"是否是标准功能的必选服务",
    "inputData":[
        {
            "identifier":"入参唯一标识符",
            "name":"入参名称",
            "dataType":{
                "type":"属性类型: int(原生)，float(原生)，double(原生), text(原生)，date(String类型UTC毫秒)，bool(0或1的int类型)，enum(int类型), struct(结构体类型，可包含前面6种类型)，array(数组类型，支持int/double/float/text)",
                "specs":{
                    "min":"参数最小值(int, float, double类型特有)",
                    "max":"参数最大值(int, float, double类型特有)",
                    "unit":"属性单位",
                    "unitName":"单位名称",
                    "size":"数组大小，默认最大128(数组特有)",
                    "item":{
                        "type":"数组元素的类型"
                    }
                }
            }
        }
    ],
    "outputData":[
        {
            "identifier":"出参唯一标识符",
            "name":"出参名称",
            "dataType":{
                "type":"属性类型: int(原生)，float(原生)，double(原生), text(原生)，date(String类型UTC毫秒)，bool(0或1的int类型)，enum(int类型), struct(结构体类型，可包含前面6种类型)，array(数组类型，支持int/double/float/text)",
                "specs":{
                    "min":"参数最小值(int, float, double类型特有)",
                    "max":"参数最大值(int, float, double类型特有)",
                    "unit":"属性单位",
                    "unitName":"单位名称",
                    "size":"数组大小，默认最大128(数组特有)",
                    "item":{
                        "type":"数组元素的类型(数组特有)"
                    }
                }
            }
        }
    ],
    "method":"服务对应的方法名称(根据identifier生成)"
}
```

<center>事件注册页面</center>
![事件注册页面](event.png)

```json
{
    "productId":"产品ID",
    "accessUri":"访问地址",
    "accessType":"访问方式 get、post",

    "identifier":"事件唯一标识符(产品下唯一，其中post是默认生成的属性上报事件)",
    "name":"事件名称",
    "desc":"事件描述",
    "type":"事件类型(info,alert,error)",
    "required":"是否是标准功能的必选事件",
    "outputData":[
        {
            "identifier":"参数唯一标识符",
            "name":"参数名称",
            "dataType":{
                "type":"属性类型: int(原生)，float(原生)，double(原生), text(原生)，date(String类型UTC毫秒)，bool(0或1的int类型)，enum(int类型), struct(结构体类型，可包含前面6种类型)，array(数组类型，支持int/double/float/text)",
                "specs":{
                    "min":"参数最小值(int, float, double类型特有)",
                    "max":"参数最大值(int, float, double类型特有)",
                    "unit":"属性单位",
                    "unitName":"单位名称",
                    "size":"数组大小，默认最大128(数组特有)",
                    "item":{
                        "type":"数组元素的类型"
                    }
                }
            }
        }
    ],
    "method":"事件对应的方法名称(根据identifier生成)"
}
```


## 用户注册
用户注册，获取UserKey，UserSecret，用于调用服务时的鉴权


## 获取服务列表
用户从开放平台获取已注册服务列表，这里应该对于服务提供商，以及下属服务有一个层级划分，便于用户查看。


## 调用服务  
用户根据服务列表的请求地址，发起服务调用请求（附带鉴权信息）；
服务根据开放平台提供的鉴权接口进行权限认证；

### 属性 Property
访问对应uri，确认get\post方式

#### 读取属性值
##### 所有属性值
请求数据格式
```
REQUEST
GET http://$productLink$/api/properties
Accept: application/json
```
返回数据格式
```json
RESPONSE
200 OK
{
  "temperature": 21,
  "humidity": 50,
  "led": true
}
```

##### 单个属性值
请求数据格式
```json
REQUEST
GET http://$productLink$/api/properties/temperature
Accept: application/json
```
返回数据格式
```json
RESPONSE
200 OK
{
  "temperature": 21
}
```
#### 设置属性值
请求数据格式
```json
REQUEST
PUT http://$productLink$/api/properties/led
{
  "led": true
}
Accept: application/json
```
响应数据格式
```json
200 OK
{
  "led": true
}
```

### 服务 Service
请求数据格式
```json
Request a Service
POST https://$productLink$/api/service/lamp
Accept: application/json
{
  "fade": {
    "input": {
      "level": 50,
      "duration": 2000
    }
  }
}
```
响应数据格式
```json
Response
200 OK
{
  "fade": {
    "input": {
      "level": 100,
      "duration": 2000
    },
    "timeRequested": "2017-01-24T11:02:45+00:00",
    "timeCompleted": "2017-01-24T11:02:46+00:00",
    "status": "completed"
  }
}
```


### 事件 Event
以http异步发送事件通知形式实现

#### 订阅事件
订阅事件，注册事件异步通知地址，当事件发生时，服务向回调地址发送http请求，通知事件情况。

请求数据格式
```json
Request a Service
POST https://$productLink$/api/event/lamp/overheated
Accept: application/json
{
  "type" :"subscribe",
  "callbackUri": "https://ip:port/eventRcv"
}
```
响应数据格式
```json
200 OK
{
  "type" :"subscribe",
  "eventSrc":"https://$productLink$/api/event/lamp/overheated",
  "callbackUri": "https://ip:port/eventRcv"
}
```


异步通知数据格式
```json
Event Response
{
  "eventSrc": "https://$productLink$/api/event/lamp/overheated",
  "eventContent": [
    {
      "overheated": {
        "data": 102,
        "timestamp": "2017-01-25T15:01:35+00:00"
      }
    }
  ]
}
```

#### 取消订阅

请求数据格式
```json
Request a Service
POST https://$productLink$/api/event/lamp/overheated
Accept: application/json
{
  "type" :"unSubscribe",
  "callbackUri": "https://ip:port/eventRcv"
}
```
响应数据格式
```json
200 OK
{
  "type" :"unSubscribe",
  "eventSrc":"https://$productLink$/api/event/lamp/overheated",
  "callbackUri": "https://ip:port/eventRcv"
}
```



## 上报服务调用
在提供服务后，上报服务调用信息（用户、服务详情、调用结果）


```json
{
  "type":"Property/Service/Event",
  "link":"https://$productLink$/api",
  "timestamp": "2017-01-25T15:01:35+00:00"
}
```