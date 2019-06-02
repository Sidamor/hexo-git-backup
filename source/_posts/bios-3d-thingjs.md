---
title: 智慧园区 3D可视化 实现（thingjs）
date: 2019-06-02 14:35:58
categories: 
- bios
tags:
- bios
- 3d
- thingjs
---

基于thingjs开发智慧园区3d可视化，三个月过去，已经做了两版了（写实版、科技版）。做一个小结。

# 技术选型
threejs ? 还是 thingjs or glendale or hightopo or bimserver

## threejs
当然，最理想的是开源的threejs。然后理想很丰满，现实很骨感。初入智慧园区这个行业，团队在3d方面也没有任何经验。

threejs基本上可以理解为，将webgl的实现代码封装、简化了，并且做了一定程度上的性能优化。
举个例子，实现同样的功能，webgl的代码量是threejs的5倍：
threejs
```
var camera = new THREE.PerspectiveCamera(45, 4 / 3, 1, 1000);
camera.position.set(0, 0, 5);
camera.lookAt(new THREE.Vector3(0, 0, 0));
scene.add(camera);

var material = new THREE.MeshBasicMaterial({
        color: 0xffffff // white
});
// plane
var planeGeo = new THREE.PlaneGeometry(1.5, 1.5);
var plane = new THREE.Mesh(planeGeo, material);
plane.position.x = 1;
scene.add(plane);

// triangle
var triGeo = new THREE.Geometry();
triGeo.vertices = [new THREE.Vector3(0, -0.8, 0),
        new THREE.Vector3(-2, -0.8, 0), new THREE.Vector3(-1, 0.8, 0)];
triGeo.faces.push(new THREE.Face3(0, 2, 1));
var triangle = new THREE.Mesh(triGeo, material);
scene.add(triangle);

renderer.render(scene, camera);
```
webgl
```
var gl;
function initGL(canvas) {
    try {
        gl = canvas.getContext("experimental-webgl");
        gl.viewportWidth = canvas.width;
        gl.viewportHeight = canvas.height;
    } catch (e) {
    }
    if (!gl) {
        alert("Could not initialise WebGL, sorry :-(");
    }
}

function getShader(gl, id) {
    var shaderScript = document.getElementById(id);
    if (!shaderScript) {
        return null;
    }

    var str = "";
    var k = shaderScript.firstChild;
    while (k) {
        if (k.nodeType == 3) {
            str += k.textContent;
        }
        k = k.nextSibling;
    }

    var shader;
    if (shaderScript.type == "x-shader/x-fragment") {
        shader = gl.createShader(gl.FRAGMENT_SHADER);
    } else if (shaderScript.type == "x-shader/x-vertex") {
        shader = gl.createShader(gl.VERTEX_SHADER);
    } else {
        return null;
    }

    gl.shaderSource(shader, str);
    gl.compileShader(shader);

    if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
        alert(gl.getShaderInfoLog(shader));
        return null;
    }

    return shader;
}

var shaderProgram;

function initShaders() {
    var fragmentShader = getShader(gl, "shader-fs");
    var vertexShader = getShader(gl, "shader-vs");

    shaderProgram = gl.createProgram();
    gl.attachShader(shaderProgram, vertexShader);
    gl.attachShader(shaderProgram, fragmentShader);
    gl.linkProgram(shaderProgram);

    if (!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)) {
        alert("Could not initialise shaders");
    }

    gl.useProgram(shaderProgram);

    shaderProgram.vertexPositionAttribute = gl.getAttribLocation(shaderProgram, "aVertexPosition");
    gl.enableVertexAttribArray(shaderProgram.vertexPositionAttribute);

    shaderProgram.pMatrixUniform = gl.getUniformLocation(shaderProgram, "uPMatrix");
    shaderProgram.mvMatrixUniform = gl.getUniformLocation(shaderProgram, "uMVMatrix");
}

var mvMatrix = mat4.create();
var pMatrix = mat4.create();

function setMatrixUniforms() {
    gl.uniformMatrix4fv(shaderProgram.pMatrixUniform, false, pMatrix);
    gl.uniformMatrix4fv(shaderProgram.mvMatrixUniform, false, mvMatrix);
}

var triangleVertexPositionBuffer;
var squareVertexPositionBuffer;

function initBuffers() {
    triangleVertexPositionBuffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, triangleVertexPositionBuffer);
    var vertices = [
         0.0,  1.0,  0.0,
        -1.0, -1.0,  0.0,
         1.0, -1.0,  0.0
    ];
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);
    triangleVertexPositionBuffer.itemSize = 3;
    triangleVertexPositionBuffer.numItems = 3;

    squareVertexPositionBuffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, squareVertexPositionBuffer);
    vertices = [
         1.0,  1.0,  0.0,
        -1.0,  1.0,  0.0,
         1.0, -1.0,  0.0,
        -1.0, -1.0,  0.0
    ];
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);
    squareVertexPositionBuffer.itemSize = 3;
    squareVertexPositionBuffer.numItems = 4;
}

function drawScene() {
    gl.viewport(0, 0, gl.viewportWidth, gl.viewportHeight);
    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

    mat4.perspective(45, gl.viewportWidth / gl.viewportHeight, 0.1, 100.0, pMatrix);

    mat4.identity(mvMatrix);

    mat4.translate(mvMatrix, [-1.5, 0.0, -7.0]);
    gl.bindBuffer(gl.ARRAY_BUFFER, triangleVertexPositionBuffer);
    gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, triangleVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);
    setMatrixUniforms();
    gl.drawArrays(gl.TRIANGLES, 0, triangleVertexPositionBuffer.numItems);

    mat4.translate(mvMatrix, [3.0, 0.0, 0.0]);
    gl.bindBuffer(gl.ARRAY_BUFFER, squareVertexPositionBuffer);
    gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, squareVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);
    setMatrixUniforms();
    gl.drawArrays(gl.TRIANGLE_STRIP, 0, squareVertexPositionBuffer.numItems);
}

function webGLStart() {
    var canvas = document.getElementById("lesson01-canvas");
    initGL(canvas);
    initShaders();
    initBuffers();

    gl.clearColor(0.0, 0.0, 0.0, 1.0);
    gl.enable(gl.DEPTH_TEST);

    drawScene();
}

 var gl;
function initGL(canvas) {
    try {
        gl = canvas.getContext("experimental-webgl");
        gl.viewportWidth = canvas.width;
        gl.viewportHeight = canvas.height;
    } catch (e) {
    }
    if (!gl) {
        alert("Could not initialise WebGL, sorry :-(");
    }
}

function getShader(gl, id) {
    var shaderScript = document.getElementById(id);
    if (!shaderScript) {
        return null;
    }

    var str = "";
    var k = shaderScript.firstChild;
    while (k) {
        if (k.nodeType == 3) {
            str += k.textContent;
        }
        k = k.nextSibling;
    }

    var shader;
    if (shaderScript.type == "x-shader/x-fragment") {
        shader = gl.createShader(gl.FRAGMENT_SHADER);
    } else if (shaderScript.type == "x-shader/x-vertex") {
        shader = gl.createShader(gl.VERTEX_SHADER);
    } else {
        return null;
    }

    gl.shaderSource(shader, str);
    gl.compileShader(shader);

    if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
        alert(gl.getShaderInfoLog(shader));
        return null;
    }

    return shader;
}

var shaderProgram;

function initShaders() {
    var fragmentShader = getShader(gl, "shader-fs");
    var vertexShader = getShader(gl, "shader-vs");

    shaderProgram = gl.createProgram();
    gl.attachShader(shaderProgram, vertexShader);
    gl.attachShader(shaderProgram, fragmentShader);
    gl.linkProgram(shaderProgram);

    if (!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)) {
        alert("Could not initialise shaders");
    }

    gl.useProgram(shaderProgram);

    shaderProgram.vertexPositionAttribute = gl.getAttribLocation(shaderProgram, "aVertexPosition");
    gl.enableVertexAttribArray(shaderProgram.vertexPositionAttribute);

    shaderProgram.pMatrixUniform = gl.getUniformLocation(shaderProgram, "uPMatrix");
    shaderProgram.mvMatrixUniform = gl.getUniformLocation(shaderProgram, "uMVMatrix");
}

var mvMatrix = mat4.create();
var pMatrix = mat4.create();

function setMatrixUniforms() {
    gl.uniformMatrix4fv(shaderProgram.pMatrixUniform, false, pMatrix);
    gl.uniformMatrix4fv(shaderProgram.mvMatrixUniform, false, mvMatrix);
}

var triangleVertexPositionBuffer;
var squareVertexPositionBuffer;

function initBuffers() {
    triangleVertexPositionBuffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, triangleVertexPositionBuffer);
    var vertices = [
         0.0,  1.0,  0.0,
        -1.0, -1.0,  0.0,
         1.0, -1.0,  0.0
    ];
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);
    triangleVertexPositionBuffer.itemSize = 3;
    triangleVertexPositionBuffer.numItems = 3;

    squareVertexPositionBuffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, squareVertexPositionBuffer);
    vertices = [
         1.0,  1.0,  0.0,
        -1.0,  1.0,  0.0,
         1.0, -1.0,  0.0,
        -1.0, -1.0,  0.0
    ];
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);
    squareVertexPositionBuffer.itemSize = 3;
    squareVertexPositionBuffer.numItems = 4;
}

function drawScene() {
    gl.viewport(0, 0, gl.viewportWidth, gl.viewportHeight);
    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

    mat4.perspective(45, gl.viewportWidth / gl.viewportHeight, 0.1, 100.0, pMatrix);

    mat4.identity(mvMatrix);

    mat4.translate(mvMatrix, [-1.5, 0.0, -7.0]);
    gl.bindBuffer(gl.ARRAY_BUFFER, triangleVertexPositionBuffer);
    gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, triangleVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);
    setMatrixUniforms();
    gl.drawArrays(gl.TRIANGLES, 0, triangleVertexPositionBuffer.numItems);

    mat4.translate(mvMatrix, [3.0, 0.0, 0.0]);
    gl.bindBuffer(gl.ARRAY_BUFFER, squareVertexPositionBuffer);
    gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, squareVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);
    setMatrixUniforms();
    gl.drawArrays(gl.TRIANGLE_STRIP, 0, squareVertexPositionBuffer.numItems);
}

function webGLStart() {
    var canvas = document.getElementById("lesson01-canvas");
    initGL(canvas);
    initShaders();
    initBuffers();

    gl.clearColor(0.0, 0.0, 0.0, 1.0);
    gl.enable(gl.DEPTH_TEST);

    drawScene();
}
```
此外，threejs并没有在某个方面应用的拓展（逻辑上的、业务上的）。对于初入智慧园区这个行业的我来说，相当于要从头开始封装这个行业3d可视化方面的逻辑、业务等。且不说工作量，很多地方都无从下手，甚至连做成什么样的都很模糊。


## 适用性
最后比较一下各种库在智慧园区场景下的适用性

* glendale
用在工业上较多

* hightopo
效果酷炫，各个行业都有应用

* thingjs
主要是园区、厂区，在园区逻辑上有自己的一套逻辑，较成熟

以上四个3d库进行一个排序：
1. 对于web的支持
thingjs = hightopo > bimserver > glendale

2. 部署难易程度
thingjs = hightopo > glendale > bimserver

## 费用

### sdk使用费
* thingjs
支持obj格式模型（可通过3dmax将不同格式模型转为obj）
GIS模块（城市搭建模块）、图表模块、全景图模块、楼层展开
公有云，2888/项目*年
离线部署，58888/项目*终生

* glendale
葛兰岱尔轻量化BIM引擎须独立部署于一台windows系统的服务器，服务器上安装对应建模软件可以支持多种格式模型文件
支持多种格式模型，需要在windows上部署轻量化BIM引擎，使用复杂。
终生授权模式，30万。


* hightopo
2d、3d都有很好的支持，项目使用费也在20-30万之间。有很好的外包支持，一个项目下来大概60万左右的报价（含项目使用费）


### 建模
做3d离不开建模，自己建or外包。
以单个园区为例，外包费用：
1个大楼模型+6个单层模型（B3、B2、B1、1F、2-4F、5F-17F） 费用： 单层 楼层 2000，大楼模型 5000，总价2000 * 6 + 5000 = 17000，打九折17000 * 0.9 = 15300 

**传统建模外包效果不理想**

thingjs提供了campusbuilder（模模搭），完美贴合thingjs。


# 实现
多方面比较，选择thingjs

## 逻辑

### 模型加载
* 建模和编码剥离
* 通过campusbuilder搭建的园区，包括所有的模型，一次性加载。

```
var app = new THING.App({
    // 自定义面板
    "el": "div3d",
    //背景设置
    "skyBox": sceneSkyBox,
    "background": sceneBackGround,
    complete: function() {
        app.create({
            type: "Campus",
            url: sceneUrl,
            visible: false,
            complete: function (ev) {
                init(); //初始化
                ev.object.visible = true;
                callFuncInMain('initNotify', '');
            }
        });
    }
});
```

### 物体查找
query函数
* app.query 在全局范围内查找对象
* 此外，Campus\Building\Floor都可以使用.query查找此父阶段范围内的对象
* query函数返回的是一个数组（不管几个），在获取单个对象的时候需要加下标[0]


### 层级切换
thingjs成熟的园区逻辑包含以下几层：
* Campus
* Building
* Floor
* Thing

以上四层基本上是从大到小，层层包含。也存在一些交叉的情况，比如Thing除了Floor之外可以直接隶属于Campus或者Building。

* 为了提高页面性能，所有Thing对象默认都是隐藏的。
* 当页面生命周期滚到了相应的页面、相应的功能点、相应的层级，才会显示相应Thing对象。离开的时候会销毁。
* 所有的页面资源也是这样。

## Campus 园区层
* Campus除了包含Building之外，会有一些Thing直接隶属于Campus。 
* 一般的，我们首次进入模型页面，默认进入campus层。
* 默认的，园区地面Ground直接隶属于Campus。   
* Ground.plan 园区的地面
* 进入园区层函数
```
app.level.change(app.query('.Campus')[0]);
```

### 常用事件
* 进入Campus
在这里做一些Campus视角的调整，根据不同功能模块，设置camera到响应的视角
```
app.on(THING.EventType.EnterLevel, '.Campus', function (ev) {
    if ('elevator' == prePage) {
        // 电梯侧视
        app.camera.flyTo(globals.camera_focus.elevator);
    }else if ('door' == prePage || 'ups' == prePage) {
        // 模型靠上
        app.camera.flyTo(globals.camera_focus.door);
    }else if ('buildOperate' == prePage || 'power' == prePage) {
        // 模型靠左,斜俯视
        app.camera.flyTo(globals.camera_focus.rental);
    }else if ('dehumidifier' == prePage || 'camera' == prePage) {
        // 模型靠左
        app.camera.flyTo(globals.camera_focus.camera);
    }else {
        // 模型靠中
        app.camera.flyTo(globals.camera_focus.campus);
    }
}, 'customLevelFly');
```
* 离开Campus
当前事件除了视角跳转，没有做别的事情（显示、隐藏对象，加载资源等）。另外，这个事件的后置事件不明确（可能是Building、可能是Floor、可能是Thing，相应触发写在后置事件中）



## Building 建筑层
* 建筑层在单楼栋的园区项目中用到较少
* 目前只在停车场场景下，展开楼层中使用
* 进入建筑层函数
```
app.level.change(app.query('.Building')[0]);
```

### 常用事件
* 进入Building
停车场景下，进入建筑层。
进入之后的事件如下，楼层展开、显示车位、显示车辆。
```
app.on(THING.EventType.EnterLevel, '.Building', function (ev) {
    if ('park' == prePage) {
        ev.object.expandFloors({
            'time': 500,
            'length': 20
        });
        app.camera.flyTo(globals.camera_focus.park);
        app.query(/park/).forEach(park => {
            park.visible = true
        });
        app.query(/car/).forEach(car => {
            car.visible = true
        });
    } else {
        app.camera.flyTo({
            object: ev.object,
            time: 500
        });
    }
}, 'customLevelFly');
```
* 离开Building
这个事件的后置事件不明确（可能是Building、可能是Floor、可能是Thing，相应触发写在后置事件中）
进入Building的操作：
1. 楼层展开，离开park功能模块时才需要unexpand，不在离开Building做处理（因为进入Floor也是离开Building）
2. 显示车位、显示车辆，离开park功能模块时才需要隐藏，所以不在离开Building做处理。

## Floor 楼层层
* 进入楼层函数 
```
app.level.change(app.query('#' + floorNO)[0]);
```

### 常用事件
* 进入楼层
```
app.on(THING.EventType.EnterLevel, '.Floor', function (ev) {
    // 显示当前floor下things
    let things = ev.object.query('.Thing');
    if (null != things) {
        for (let j = 0; j < things.length; j++) {
            things[j].visible = true;
        }
    }
    // 不同模块指定视角
    if ('park' == prePage) {
        app.camera.flyTo(eval('globals.camera_focus.park_' + ev.object.id));
    } else {
        app.camera.flyTo({
            object: ev.object,
            time: 500
        });
    }
}, 'customLevelFly');
```

* 离开楼层
```
app.on(THING.EventType.LeaveLevel, '.Floor', function (ev) {                
    let obj = ev.object;
    // 隐藏当前floor下things
    obj.query('.Thing').forEach(thing => {
        thing.visible = false;
    });
});
```

## Thing 物体层
* 进入物体函数
```
app.level.change(app.query('#' + uid)[0]);
```

### 常用事件
* 进入物体：camera飞到物体正面，拉开一段距离，看向物体
```
app.on(THING.EventType.EnterLevel, '.Thing', function (ev) {
    let pos = ev.object.selfToWorld([0, 2.0, 3.0]);
    let targ = ev.object.position;
    targ[1] += 0.95;
    app.camera.flyTo({
        time: 500,
        position: pos,
        target: targ,
    });
}, 'customLevelFly');   
```


# 附录

## 视角存储
* 不同项目，不同功能模块，视角都不一样。为了保证代码的可复用性，每一个项目都要录入对应的视角到数据库。
* 所有该项目视角存入全局变量 
```
globals.camera_focus
```

如下，camera位置、camera镜头方向，camera飞行时间
```
"campus": {
    "position": [99, 37, -122], 
    "target": [101, 33, -15], 
    "time": 500
}, 
```

## 暂停默认事件
所有自定义过的事件，都需要暂停其默认事件
```
app.pauseEvent(THING.EventType.EnterLevel, '.Campus', THING.EventTag.LevelFly);
app.pauseEvent(THING.EventType.EnterLevel, '.Building', THING.EventTag.LevelFly);
app.pauseEvent(THING.EventType.EnterLevel, '.Floor', THING.EventTag.LevelFly);
app.pauseEvent(THING.EventType.LeaveLevel, '.Floor', THING.EventTag.LevelFly);
app.pauseEvent(THING.EventType.EnterLevel, '.Thing', THING.EventTag.LevelFly);
app.pauseEvent(THING.EventType.EnterLevel, '.Building', THING.EventTag.LevelSetBackground);
app.pauseEvent(THING.EventType.EnterLevel, '.Floor', THING.EventTag.LevelSetBackground);
```

## 暂停鼠标事件相应
* 默认的，鼠标双击对象进入层级，鼠标右键空白处退出层级
* 根据需求，用户不能直接操作模型
* 暂停这两个事件响应
```
app.pauseEvent(THING.EventType.DBLClick, '*', THING.EventTag.LevelEnterOperation);
app.pauseEvent(THING.EventType.Click, '*', THING.EventTag.LevelBackOperation);
```

## iframe内嵌
* iframe内嵌后，3d页面捕获不到键盘事件。但是可以捕获到鼠标事件。
* iframe内嵌后，捕获?之后的传参，可以用于免登、当前园区、当前模块、3d风格等。
* 3d与主框架的交互
```
    // 调用 MainFrame 页面方法
    function callFuncInMain(funcName, data) {
        var message = {
            'source': 'thingjs',
            'funcName': funcName, // 所要调用父页面里的函数名
            'param': data
        }
        // 向父窗体(用户主页面)发送消息
        // 第一个参数是具体的信息内容，
        // 第二个参数是接收消息的窗口的源（origin），即"协议 + 域名 + 端口"。也可以设为*，表示不限制域名，向所有窗口发送
        window.parent.postMessage(message, '*');
    }

    window.funcTable = {// 模块入口
        // 首页，ups， 能耗，  楼宇运营，     告警，  给排水，消防，    电梯，     照明，   门禁， 停车，监控， 除湿机
        homepage, ups, power, buildOperate, alarm, drain, fireAlarm, elevator, light, door, park, camera, dehumidifier,
        // 接口调用
        // 通用跳转(uid) 楼层跳转(楼层)                                转到门禁(uid) 标注蓝点(uid) 标注红点(uid)  绘制多边形(uidListArray)
        gotoThing, gotoFloor, gotoBuilding, gotoCampus, paintOutline, gotoDoor, paintBluePoint, paintRedPoint, paintPolygonRegion
    }
    // 监听 MainFrame 页面传回的数据 并调用 ThingJS 页面方法
    window.addEventListener('message', function (e) {
        var data = e.data;
        console.log(data)
        var funcName = data.funcName;
        var param = data.param;

        if ('callFuncInMain' == funcName) {
            return;
        }
        // 调用 ThingJS 页面方法
        window.funcTable[funcName](param);
    });
```

## 科技版重绘
* 重绘的过程中，会有一段时间的模型留白。
* 加载场景过程中隐藏模型，加载完成后再显示。
```
var app = new THING.App({
    // 自定义面板
    "el": "div3d",
    //背景设置
    "skyBox": sceneSkyBox,
    "background": sceneBackGround,
    complete: function() {
        app.create({
            type: "Campus",
            url: sceneUrl,
            visible: false,
            complete: function (ev) {
                init(); //初始化
                ev.object.visible = true;
                callFuncInMain('initNotify', '');
            }
        });
    }
});
```

## h5
* thingjs有一些h5元素不支持显示，以h5的形式存入[div3d]
* 通过操作DOM的方式，如cloneNode、appendChild等操作加入thingjs

```
app.create({
    type: "UI",
    el: createDomElement('mark_redPoint'),
    parent: app.query('#' + uid)[0]
}).visible = true;

function createDomElement(markId) {
    let tmpEle = document.getElementById(markId).cloneNode(true);
    globals.tempDom.appendChild(tmpEle); 
    let tName = document.createAttribute("name");
    tName.nodeValue = markId;
    tmpEle.setAttributeNode(tName);
    tmpEle.style.display = 'block';
    tmpEle.style.zIndex = 10;
    return tmpEle;
}	

document.getElementById('div3d').innerHTML =
"<div class=\"bluePoint\" id=\"mark_bluePoint\" >\
    <img width='100%' height='100%' src='/uploads/wechat/c29uZ3k=/file/iSysCore/bluepoint.png' />\
</div>\
<div class=\"redPoint\" id=\"mark_redPoint\" >\
    <img width='100%' height='100%' src='/uploads/wechat/c29uZ3k=/file/iSysCore/redpoint.png' />\
</div>"
```

## closePage 页面资源释放
* 进入新的功能模块时，需释放之前模块的页面资源
1. 隐藏所有Things
2. h5资源（DOM）
3. websocket资源
4. 不同模块的个性化资源