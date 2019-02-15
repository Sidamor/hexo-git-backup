---
title: bios-开放框架
date: 2019-02-08 21:18:24
categories: 
- bios
tags:
- bios
- openframe
---

bios，Business Intelligent Operating System，商业智能操作系统
身为一个操作系统，区别于行业应用，很重要的一点，对于其他应用接入的兼容性。
于是，开放框架，被提了出来。 （最初听到这个概念，很想去问问《中国合伙人2》里的“楚振辉”，他的“红中OS”里是不是也有开放框架）  

言归正传，开放框架是什么，它具体要实现哪些具体功能才能被称为“开放框架”？ 正是我们要研究讨论的。  

<!--more-->

# 先来到google  

* 第一条弹出的是Taro，简单看了一眼，一个前端框架，让你的代码一次编写，APP\小程序\WEB\快应用 多端运行。好吧，以后可能会用到，暂且一记。

* 接下来，看到了apache的微服务框架。 熟悉的RPC模式，注册中心+Provider+Consumer。不错的框架，规范标准，不过好像不是我想要的。

* 开放组体系结构框架（英语：The Open Group Architecture Framework，缩写：TOGAF）。
等等，这好像有点接近我要的东西了。（一顿学习，学到好多，有点困）不过好像有点复杂，现有团队要从零开始做初这个东西，emm.. 我再想想

看到这里，总结一下：
1. 首先，我们不可能做一个类似编译器的东西，制订语言标准，让第三方在这个上面进行应用开发。
2. 微服务框架，很好的现有框架。不过，接入时不能做到很好的可视化。接口协议是apache制定的，不算特别普及，对于第三方应用存在一定的开发难度。
3. TOGAF，很好的东西，非常完善。不过一个初创公司可能更需要的是小而美。 麻雀虽小，五脏俱全，就够了。  

# Mozilla - WOT & Alibaba - TSL（ALink）  

我们的bios可以说是基于物联网的，所以当我看到Mozilla推出物联网开放框架（wot-Web Of Things），没错就是他了。
点进去之后（扶了扶眼镜），这不是阿里云的物模型（TSL-Thing Specification Language）嘛！  

## Mozilla-wot  

* 功能定义：有点多，一部分是名称、描述之类的信息，其实也是3个。 Property、Action、Event。
1. @context member
2. @type member
3. name member
4. description member
5. properties member
6. actions member
7. events member
8. links member
这里把links的例子列出来看一下
```json
EXAMPLE 5: Example links member
"links": [
  {
    "rel": "properties",
    "href": "/things/lamp/properties"
  },
  {
    "rel": "actions",
    "href": "/things/lamp/actions"
  },
  {
    "rel": "events",
    "href": "/things/lamp/events"
  },
  {
    "rel": "alternate",
    "href": "wss://mywebthingserver.com/things/lamp"
  },
  {
    "rel": "alternate",
    "mediaType": "text/html",
    "href": "/things/lamp"
  }
]
```
可以看到，这里通过WebSocket地址的方式，替代了阿里云MQTT里Topic的管理方式。
* 配置：没找到相应配置页面
* 通信包：协议基于json
* 通信方式：RESTAPI、WebSocket，Mozilla提供了定义类型的基类，实现预留方法即可。

## ALink-TSL

* 功能定义：属性、服务、和事件
1. 属性（Property）	一般用于描述设备运行时的状态，如环境监测设备所读取的当前环境温度等。属性支持 GET 和 SET 请求方式。应用系统可发起对属性的读取和设置请求。
2. 服务（Service）	设备可被外部调用的能力或方法，可设置输入参数和输出参数。相比于属性，服务可通过一条指令实现更复杂的业务逻辑，如执行某项特定的任务。
3. 事件（Event）	设备运行时的事件。事件一般包含需要被外部感知和处理的通知信息，可包含多个输出参数。如，某项任务完成的信息，或者设备发生故障或告警时的温度等，事件可以被订阅和推送。

* 配置：阿里云提供了新增、配置页面。 有一定的设备库可以参考使用，还不是很全。
* 通信包：基于json
* 通信方式：阿里云的MQTT，通过对应的topic进行消息通信，ALink也提供了对应的类库

# 回到我们的开放框架

* 功能定义上，ALink更为合理，采用json格式
* 配置页面肯定要有
* 通信包基于json没有问题
* 通信方式，采用WebSocket，在功能定义中加入link字段

## 定义
参照ALink的物模型（TSL），给出属性、服务、事件（Property\Service\Event,以下简称PSE）的定义。每一个注册上来的PSE需要给出它的定义。根据这个定义，调用方组织正确的通信包来进行调用。

### 产品 Product
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

### 属性 Property
“一般用于描述设备运行时的状态，如环境监测设备所读取的当前环境温度等。属性支持 GET 和 SET 请求方式。应用系统可发起对属性的读取和设置请求。”
根据以上功能描述，属性的功能定义需要以下元素：

* 基础信息
* 支持方法（读取/设置）
* 访问方式（访问路径、get/post）

尝试以json方式表示

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


### 服务 Service

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

### 事件 Event
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


## 调用
有了接口定义之后，就是调用了。根据产品的link & PSE的accessUri，可以组成访问的地址。通过这个地址以及对应的访问模式，来进行调用。

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


## 调用上报

作为操作系统，bios需要知道每一条PSE被调用的情况。

服务提供商需要在每一次成功提供服务之后向bios的消息队列提交一条消息，消息体结构如下

```json
{
  "type":"Property/Service/Event",
  "link":"https://$productLink$/api"
}
```