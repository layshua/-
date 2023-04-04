# 模块化搭建

## 初始化

**1.初始化npm**

~~~
npm init -y
~~~

**2.安装parcel**

[🚀 快速开始 | Parcel 中文网 (parceljs.cn)](https://www.parceljs.cn/getting_started.html)

NPM 方式安装：

~~~js
npm install --save-dev parcel
~~~

接着，通过修改你的`package.json`来添加这些任务脚本

~~~js
{
  "scripts": {
    "dev": "parcel <your entry file>",
    "build": "parcel build <your entry file>"
  }
}
//这里的<your entry file>改为自己创建的html文件名
~~~

然后，你就能运行它了：

```
# 以开发模式运行
yarn dev
# 或
npm run dev

# 以生成模式运行
yarn build
# 或
npm run build
```

**3.安装常用依赖**

~~~js
//1. three依赖
npm install three
//引入
import *as THREE from "three";

//2. 图形用户界面库
npm install dat.gui
//引入
import * as dat from "dat.gui"

//3. web动画库
npm install gsap
//引入 使用时从官网查文档
import gsap from "gsap";
~~~

**4.运行打包**

~~~js
npm run dev
~~~

## 项目设置

**基本布局**

![image-20220922223343115](image-20220922223343115.png)

~~~json
//package.json

{
  "name": "learnthreejs",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "parcel src/index.html",
    "build": "parcel build src/index.html"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "parcel": "^2.7.0",
  },
  "dependencies": {
    "dat.gui": "^0.7.9",
    "gsap": "^3.11.1",
    "three": "^0.144.0"
  }
}


~~~

~~~html
//index.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link rel="stylesheet" href="assets/css/style.css">
</head>
<body>
    <script src="./main/main.js" type="module"></script>
</body>
</html>
~~~

~~~css
//style.css

*{
    margin: 0;
    padding: 0;
}
body{
    background-color: skyblue;
}
~~~

~~~js
//main.js测试场景

import * as THREE from "three";
//轨道控制器（相机）
import {} from 'three/examples/jsm/controls/OrbitControls' 
// console.log(THREE);

// 目标：了解three.js最基本的内容

// 1、创建场景
const scene = new THREE.Scene();

// 2、创建相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);

// 设置相机位置
camera.position.set(0, 0, 10);
scene.add(camera);

// 添加物体
// 创建几何体
const cubeGeometry = new THREE.BoxGeometry(1, 1, 1);
const cubeMaterial = new THREE.MeshBasicMaterial({ color: 0xffff00 });
// 根据几何体和材质创建物体
const cube = new THREE.Mesh(cubeGeometry, cubeMaterial);
// 将几何体添加到场景中
scene.add(cube);

// 初始化渲染器
const renderer = new THREE.WebGLRenderer();
// 设置渲染的尺寸大小
renderer.setSize(window.innerWidth, window.innerHeight);
// console.log(renderer);
// 将webgl渲染的canvas内容添加到body
document.body.appendChild(renderer.domElement);

// 使用渲染器，通过相机将场景渲染进来
renderer.render(scene, camera);
~~~

# 入门

**1. 轨道控制器Controls**

~~~js
//引入
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
//创建轨道控制器
const controls = new OrbitControls(camera, renderer.domElement);

function render() {
    renderer.render(scene, camera);
    requestAnimationFrame(render);
}
render();
~~~

**2. 坐标轴AxesHelper**

~~~js
const axesHelper = new THREE.AxesHelper( 5 );
scene.add( axesHelper );
~~~

**3.模型参数**

~~~js
//位移
cube.position.set(5, 0, 0);
cube.position.x = 5;

//缩放
cube.scale.set(2, 2, 2);

//旋转
cube.rotation.set(Math.PI / 4, 0, 0);  //弧度制
~~~

**4.时间clock**

~~~js
let time = clock.getElapsedTime();
let deltaTime = clock.getDelta();
console.log("时钟运行总时长：", time);
console.log("两次获取时间的间隔时间：", deltaTime);
~~~

**5.屏幕自适应**

~~~js
//监听画面变化，更新渲染画面
window.addEventListener("resize", () => {
    //更新摄像头
    camera.aspect = window.innerWidth / window.innerHeight;
    //更新摄像头的投影矩阵
    camera.updateProjectionMatrix();
    //更新渲染器
    renderer.setSize(window.innerWidth, window.innerHeight);
    //设置渲染器的像素比
    renderer.setPixelRatio(window.devicePixelRatio);
})
~~~

**6.控制屏幕全屏**

~~~js
//控制屏幕全屏
window.addEventListener("dblclick", () => {
    const fullScreenElement = document.fullscreenElement;
    if (!fullScreenElement) {
        //双击屏幕进入全屏，退出全屏
        //让画布对象全屏
        renderer.domElement.requestFullscreen();
    } else {
        document.exitFullscreen();
    }
});
~~~

**7.dat.gui**

~~~js
//创建gui
const gui = new dat.GUI();
//设置对应参数
//位置
gui
    .add(cube.position, "x")
    .min(0)
    .max(5)
    .step(0.01)
    .name("移动x轴坐标")
    .onChange((value) => {
        console.log("值被修改：",value)
    })
    .onFinishChange((value) => {
        console.log("完全停下来",value)
    })
//颜色
const params = {
    color: "#ffff00",
};
gui.addColor(params, "color")
    .onChange((value) => {
        console.log("值被修改：", value);
        cube.material.color.set(value);
    });
//显示与隐藏
gui.add(cube, "visible").name("是否显示");
//控制立方体运动
const params = {
    fn: () => {
        gsap.to(cube.position, { x: 5, duration: 2, yoyo: true, repeat: - 1});
    },
};
gui.add(params, "fn").name("立方体运动");
//文件夹
//可以把上述写过的gui函数改成folder.xx形式加入文件夹
var folder = gui.addFolder("设置立方体");
folder.add(cube.material,"wireframe")	//线框显示

~~~

# 概览

![image-20220922145223713](image-20220922145223713.png)

~~~HTML
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>第一个three.js文件_WebGL三维场景</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      /* 隐藏body窗口区域滚动条 */
    }
  </style>
  <!--引入three.js三维引擎-->
  <script src="http://www.yanhuangxueyuan.com/versions/threejsR92/build/three.js"></script>
  <!-- <script src="./three.js"></script> -->
  <!-- <script src="http://www.yanhuangxueyuan.com/threejs/build/three.js"></script> -->
</head>

<body>
  <script>
    /**
     * 创建场景对象Scene
     */
    var scene = new THREE.Scene();
    /**
     * 创建网格模型
     */
    // var geometry = new THREE.SphereGeometry(60, 40, 40); //创建一个球体几何对象
    var geometry = new THREE.BoxGeometry(100, 100, 100); //创建一个立方体几何对象Geometry
    var material = new THREE.MeshLambertMaterial({
      color: 0x0000ff
    }); //材质对象Material
    var mesh = new THREE.Mesh(geometry, material); //网格模型对象Mesh
    scene.add(mesh); //网格模型添加到场景中
    /**
     * 光源设置
     */
    //点光源
    var point = new THREE.PointLight(0xffffff);
    point.position.set(400, 200, 300); //点光源位置
    scene.add(point); //点光源添加到场景中
    //环境光
    var ambient = new THREE.AmbientLight(0x444444);
    scene.add(ambient);
    // console.log(scene)
    // console.log(scene.children)
    /**
     * 相机设置
     */
    var width = window.innerWidth; //窗口宽度
    var height = window.innerHeight; //窗口高度
    var k = width / height; //窗口宽高比
    var s = 200; //三维场景显示范围控制系数，系数越大，显示的范围越大
    //创建相机对象
    var camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000);
    camera.position.set(200, 300, 200); //设置相机位置
    camera.lookAt(scene.position); //设置相机方向(指向的场景对象)
    /**
     * 创建渲染器对象
     */
    var renderer = new THREE.WebGLRenderer();
    renderer.setSize(width, height);//设置渲染区域尺寸
    renderer.setClearColor(0xb9d3ff, 1); //设置背景颜色
    document.body.appendChild(renderer.domElement); //body元素中插入canvas对象
    //执行渲染操作   指定场景、相机作为参数
    renderer.render(scene, camera);
  </script>
</body>
</html>
~~~

## 1 旋转动画

### 周期性渲染

​	在1.1节中讲解过，每执行一次渲染器对象[WebGLRenderer](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/renderers/WebGLRenderer)的渲染方法`.render()`，浏览器就会渲染出一帧图像并显示在Web页面上，这就是说你按照一定的周期不停地调用渲染方法`.render()`就可以不停地生成新的图像覆盖原来的图像。这也就是说只要一边旋转立方体，一边执行渲染方法`.render()`重新渲染，就可以实现立方体的旋转效果。

​	**为了实现周期性渲染可以通过浏览器全局对象`window`对象的一个方法`setInterval()`,可以通过window对象调用该方法`window.setInterval()`，也可以直接以函数形式调用`setInterval()`。**

​	**`		setInterval()`是一个周期性函数，就像一个定时器，每隔多少毫秒ms执行一次某个函数。**

~~~JS
// 间隔20ms周期性调用函数fun
setInterval("render()",20)
~~~

​	为了实现立方体旋转动画效果，直接使用下面的代码代替1.1节中代码`renderer.render(scene,camera);`即可。

~~~JS
// 渲染函数
function render() {
    renderer.render(scene,camera);//执行渲染操作
    mesh.rotateY(0.01);//每次绕y轴旋转0.01弧度
}
//间隔20ms周期性调用函数fun,20ms也就是刷新频率是50FPS(1s/20ms)，每秒渲染50次
setInterval("render()",20);
~~~

​	上面代码定义了一个渲染函数`render()`，函数中定义了三个语句，通过`setInterval("render()",20);`可以实现m每间隔20毫秒调用一次函数`render()`，每次调用渲染函数的时候，执行`renderer.render(scene,camera);`渲染出一帧图像，执行`mesh.rotateY(0.01);`语句使立方体网格模型绕y轴旋转0.01弧度。

### 渲染频率

​	调用渲染方法`.render()`进行渲染的渲染频率不能太低，比如执行`setInterval("render()",200);`间隔200毫秒调用渲染函数渲染一次，相当于每秒渲染5次，你会感觉到比较卡顿。渲染频率除了不能太低，也不能太高，太高的话计算机的硬件资源跟不上，函数`setInterval()`设定的渲染方式也未必能够正常实现。一般调用渲染方法`.render()`进行渲染的渲染频率控制在每秒30~60次，人的视觉效果都很正常，也可以兼顾渲染性能。

~~~js
//设置调用render函数的周期为200ms，刷新频率相当于5你能明显的感受到卡顿
setInterval("render()",200);
~~~

### `requestAnimationFrame()`

​	前面讲解threejs动画效果，使用了`setInterval()`函数，**实际开发中，为了更好的利用浏览器渲染，可以使用函数`requestAnimationFrame()`代替`setInterval()`函数，**`requestAnimationFrame()`和`setInterval()`一样都是浏览器`window`对象的方法。

`requestAnimationFrame()`参数是将要被调用函数的函数名，**`requestAnimationFrame()`调用一个函数不是立即调用而是向浏览器发起一个执行某函数的请求， 什么时候会执行由浏览器决定，一般默认保持60FPS的频率，**大约每16.7ms调用一次`requestAnimationFrame()`方法指定的函数，60FPS是理想的情况下，如果渲染的场景比较复杂或者说硬件性能有限可能会低于这个频率。可以查看文章[《requestAnimationFrame()》](http://www.yanhuangxueyuan.com/HTML5/time.html)了解更多`requestAnimationFrame()`函数的知识。

```javascript
function render() {
        renderer.render(scene,camera);//执行渲染操作
        mesh.rotateY(0.01);//每次绕y轴旋转0.01弧度
        requestAnimationFrame(render);//请求再次执行渲染函数render
    }
render();
```

### 均匀旋转

​	在实际执行程序的时候，可能`requestAnimationFrame(render)`请求的函数并不一定能按照理想的60FPS频率执行，**两次执行渲染函数的时间间隔也不一定相同，如果执行旋转命令的`rotateY`的时间间隔不同，旋转运动就不均匀，为了解决这个问题需要记录两次执行绘制函数的时间间隔。**

​	使用下面的渲染函数替换原来的渲染函数即可，**`rotateY()`的参数是`0.001*t`，也意味着两次调用渲染函数执行渲染操作的间隔`t`毫秒时间内，立方体旋转了`0.001*t`弧度，很显然立方体的角速度是`0.001`弧度每毫秒**(0.0001 rad/ms = 1 rad/s = 180度/s)。CPU和GPU执行一条指令时间是纳秒ns级，相比毫秒ms低了6个数量级，所以一般不用考虑渲染函数中几个计时语句占用的时间，除非你编写的是要精确到纳秒ns的级别的标准时钟程序。

```javascript
let T0 = new Date();//上次时间
function render() {
        let T1 = new Date();//本次时间
        let t = T1-T0;//时间差
        T0 = T1;//把本次时间赋值给上次时间
        requestAnimationFrame(render);
        renderer.render(scene,camera);//执行渲染操作
        mesh.rotateY(0.001*t);//旋转角速度0.001弧度每毫秒
    }
render();
```

## 2 鼠标控制

​	为了使用鼠标操作三维场景，可以借助three.js众多控件之一**`OrbitControls.js`**，可以在下载的`three.js-master`文件中找到(`three.js-master\examples\js\controls`)。 然后和引入`three.js`文件一样在html文件中引入控件`OrbitControls.js`。本节课的目的不是为了深入讲解`OrbitControls.js`，主要目的一方面向大家展示下threejs的功能，另一方面后面课程学习过程中经常会通过鼠标旋转、缩放模型进行代码调试。

~~~js
<script src="three.js-master/examples/js/controls/OrbitControls.js"></script>
~~~

### 引入控件

​	`OrbitControls.js`控件支持鼠标左中右键操作和键盘方向键操作，具体代码如下，使用下面的代码替换1.1节中`renderer.render(scene,camera);`即可。

~~~js
function render() {
  renderer.render(scene,camera);//执行渲染操作
}
render();
var controls = new THREE.OrbitControls(camera,renderer.domElement);//创建控件对象
controls.addEventListener('change', render);//监听鼠标、键盘事件
~~~

​	**OrbitControls.js控件提供了一个构造函数`THREE.OrbitControls()`，把一个相机对象作为参数的时候，执行代码`new THREE.OrbitControls(camera,renderer.domElement)`，浏览器会自动检测鼠标键盘的变化， 并根据鼠标和键盘的变化更新相机对象的参数，**比如你拖动鼠标左键，浏览器会检测到鼠标事件，把鼠标平移的距离按照一定算法转化为相机的的旋转角度，你可以联系生活中相机拍照,即使景物没有变化，你的相机拍摄角度发生了变化，自然渲染器渲染出的结果就变化了。

​	**通过定义监听事件`controls.addEventListener('change', render)`，如果你连续操作鼠标，相机的参数不停的变化，同时会不停的调用渲染函数`render()`进行渲染，这样threejs就会使用相机新的位置或角度数据进行渲染。**

​	执行构造函数`THREE.OrbitControls()`浏览器会同时干两件事，一是给浏览器定义了一个鼠标、键盘事件，自动检测鼠标键盘的变化，如果变化了就会自动更新相机的数据， 执行该构造函数同时会返回一个对象，可以给该对象添加一个监听事件，只要鼠标或键盘发生了变化，就会触发渲染函数。 关于监听函数`addEventListener`介绍可以关注文章[《HTML5事件》](http://www.yanhuangxueyuan.com/HTML5/event.html)。

### 场景操作

- 缩放：滚动—鼠标中键
- 旋转：拖动—鼠标左键
- 平移：拖动—鼠标右键

### `requestAnimationFrame()`使用情况

​	**如果threejs代码中通过`requestAnimationFrame()`实现渲染器渲染方法`render()`的周期性调用，当通过OrbitControls操作改变相机状态的时候，没必要在通过`controls.addEventListener('change', render)`监听鼠标事件调用渲染函数，因为`requestAnimationFrame()`就会不停的调用渲染函数。**

```javascript
function render() {
  renderer.render(scene,camera);//执行渲染操作
  // mesh.rotateY(0.01);//每次绕y轴旋转0.01弧度
  requestAnimationFrame(render);//请求再次执行渲染函数render
}
render();
var controls = new THREE.OrbitControls(camera,renderer.domElement);//创建控件对象
// 已经通过requestAnimationFrame(render);周期性执行render函数，没必要再通过监听鼠标事件执行render函数
// controls.addEventListener('change', render)
```

​	**注意开发中不要同时使用`requestAnimationFrame()`或`controls.addEventListener('change', render)`调用同一个函数，这样会冲突。**

## 3 插入几何体

​	前面课程绘制了一个立方体效果，下面通过three.js的球体构造函数[SphereGeometry()](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/geometries/SphereGeometry)在三维场景中添加一个球几何体。

### SphereGeometry构造函数

```javascript
SphereGeometry(radius, widthSegments, heightSegments)
```

​	第一个参数`radius`约束的是球的大小，参数`widthSegments`、`heightSegments`约束的是球面的精度，球体你可以理解为正多面体，就像圆一样是正多边形，当分割的边足够多的时候，正多边形就会无限接近于圆，球体同样的的道理， 有兴趣可以研究利用WebGL实现它的算法，对于three.js就是查找文档看使用说明。

| 参数           | 含义                     |
| :------------- | :----------------------- |
| radius         | 球体半径                 |
| widthSegments  | 控制球面精度，水平细分数 |
| heightSegments | 控制球面精度，竖直细分数 |

### 绘制球体网格模型

​	使用`THREE.SphereGeometry(60,40,40);`替换立方体几何体代码`new THREE.BoxGeometry(100, 100, 100);`。

```javascript
var box=new THREE.SphereGeometry(60,40,40);//创建一个球体几何对象
```

### 其他几何体

​	threejs除了立方体、球体还提供了很多的常见几何体的API，这里不再过多讲解，具体可以查看[threejs文档](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/geometries/CylinderGeometry)，你可以在案例源码中测试下面的几何体代码。

```javascript
//长方体 参数：长，宽，高
var geometry = new THREE.BoxGeometry(100, 100, 100);
// 球体 参数：半径60  经纬度细分数40,40
var geometry = new THREE.SphereGeometry(60, 40, 40);
// 圆柱  参数：圆柱面顶部、底部直径50,50   高度100  圆周分段数
var geometry = new THREE.CylinderGeometry( 50, 50, 100, 25 );
// 正八面体
var geometry = new THREE.OctahedronGeometry(50);
// 正十二面体
var geometry = new THREE.DodecahedronGeometry(50);
// 正二十面体
var geometry = new THREE.IcosahedronGeometry(50);
```

### 绘制多个几何体

​	这也比较简单，直接模仿立方体的代码就可以，需要创建一个几何体对象作和一个材质对象，然后把两个参数作为网格模型构造函数`Mesh()`的参数创建一个网格模型，然后再使用场景对象`scene`的方法`.add()`把网格模型`mesh`加入场景中。

​	threejs的几何体默认位于场景世界坐标的原点(0,0,0),所以绘制多个几何体的时候，主要它们的位置设置。**在mesh.add()之前通过mesh.position.set(x,y,z)等函数来设置模型位置：**

~~~js
mesh.position.set(120,0,0);
//or
mesh2.translateY(120); //球体网格模型沿Y轴正方向平移120
~~~

下面代码同时绘制了立方体、球体和圆柱三个几何体对应的网格模型。

```javascript
// 立方体网格模型
var geometry1 = new THREE.BoxGeometry(100, 100, 100);
var material1 = new THREE.MeshLambertMaterial({
  color: 0x0000ff
}); //材质对象Material
var mesh1 = new THREE.Mesh(geometry1, material1); //网格模型对象Mesh
scene.add(mesh1); //网格模型添加到场景中

// 球体网格模型
var geometry2 = new THREE.SphereGeometry(60, 40, 40);
var material2 = new THREE.MeshLambertMaterial({
  color: 0xff00ff
});
var mesh2 = new THREE.Mesh(geometry2, material2); //网格模型对象Mesh
mesh2.translateY(120); //球体网格模型沿Y轴正方向平移120
scene.add(mesh2);

// 圆柱网格模型
var geometry3 = new THREE.CylinderGeometry(50, 50, 100, 25);
var material3 = new THREE.MeshLambertMaterial({
  color: 0xffff00
});
var mesh3 = new THREE.Mesh(geometry3, material3); //网格模型对象Mesh
// mesh3.translateX(120); //球体网格模型沿Y轴正方向平移120
mesh3.position.set(120,0,0);//设置mesh3模型对象的xyz坐标为120,0,0
scene.add(mesh3); //
```

### 添加辅助三维坐标系`AxisHelper`

为了方便调试预览threejs提供了一个辅助三维坐标系`AxisHelper`，可以直接调用`THREE.AxisHelper`创建一个三维坐标系，然后通过`.add()`方法插入到场景中即可。

```javascript
// 辅助坐标系  参数250表示坐标系大小，可以根据场景大小去设置
var axisHelper = new THREE.AxesHelper(250);
scene.add(axisHelper);
```

## 4 材质效果

​	前面案例中几何体对应网格模型材质只是设置了一个颜色，实际渲染的时候往往会设置其他的参数，比如实现玻璃效果要设置材质透明度，一些光亮的表面要添加高光效果。

### 半透明效果

​	更改场景中的球体材质对象构造函数[THREE.MeshLambertMaterial()](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/MeshLambertMaterial)的参数，添加`opacity`和`transparent`属性，`opacity`的值是`0~1`之间，**`transparent`表示是否开启透明度效果**， 默认是`false`表示透明度设置不起作用，值设置为`true`，网格模型就会呈现透明的效果，使用下面的代码替换原来的球体网格模型的材质， 刷新浏览器,通过鼠标旋转操作场景,可以看到半透明的球体和立方体颜色叠加融合的效果。

```javascript
var sphereMaterial=new THREE.MeshLambertMaterial({
    color:0xff0000,
    opacity:0.7,
    transparent:true
});//材质对象
```

材质对象的一些属性可以在构造函数参数中设置，也可以访问材质对象的属性设置。

```javascript
material.opacity = 0.5 ;
material.transparent = true ;
```

### 材质常见属性

| 材质属性    | 简介                                       |
| :---------- | :----------------------------------------- |
| color       | 材质颜色，比如蓝色0x0000ff                 |
| wireframe   | 将几何图形渲染为线框。 默认值为false       |
| opacity     | 透明度设置，0表示完全透明，1表示完全不透明 |
| transparent | 是否开启透明，默认false                    |

### 添加高光效果

​	**直接使用下面的代码替换上面的透明度材质即可**，刷新浏览器可以看到球体表面的高光效果。

```javascript
var sphereMaterial=new THREE.MeshPhongMaterial({
    color:0x0000ff,
    specular:0x4488ee,
    shininess:12
});//材质对象
```

​	**对于three.js而言漫反射、镜面反射分别对应两个构造函数[MeshLambertMaterial()](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/MeshLambertMaterial)、[MeshPhongMaterial()](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/MeshPhongMaterial),**通过three.js引擎你可以很容易实现这些光照模型，不需要自己再使用原生WebGL实现，更多关于光照模型的知识可以参考文章[《WebGL_course光照渲染立方体》](http://www.yanhuangxueyuan.com/WebGL_course/light.html)或计算机图形学的相关书籍。

​	前面案例都是通过构造函数`MeshLambertMaterial()`实现漫反射进行渲染，高光效果要通过构造函数`MeshPhongMaterial()`模拟镜面反射实现，属性`specular`表示球体网格模型的高光颜色，改颜色的RGB值会与光照颜色的RGB分量相乘， `shininess`属性可以理解为光照强度的系数。

### 材质类型

​	threejs提供了很多常用的材质效果，这些效果本质上都是对WebGL着色器的封装，对于开发者来说直接使用就可以,这里不再做过多介绍，大家现有一个印象即可。

| 材质类型                                                     | 功能                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [MeshBasicMaterial](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/MeshBasicMaterial) | 基础网格材质，不受光照影响的材质                             |
| [MeshLambertMaterial](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/MeshLambertMaterial) | Lambert网格材质，与光照有反应，漫反射                        |
| [MeshPhongMaterial](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/MeshPhongMaterial) | 高光Phong材质,与光照有反应                                   |
| [MeshStandardMaterial](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/MeshStandardMaterial) | PBR物理材质，相比较高光Phong材质可以更好的模拟金属、玻璃等效果 |

## 5 光源

为了更好的渲染场景，threejs提供了生活中常见的一些光源的API。

你可以测试前面几节课案例中的光源代码，比如全部删除所有的光源代码，你会发现场景中的物体是黑色的，就像生活中一样没有光，物体是黑色的。

### 常见光源类型

本节课不会对threejs各类光源进行深入介绍，主要是简单展示下，对于初学者有一个印象就可以。

| 光源                                                         | 简介               |
| :----------------------------------------------------------- | :----------------- |
| [AmbientLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/AmbientLight) | 环境光             |
| [PointLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/PointLight) | 点光源             |
| [DirectionalLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/DirectionalLight) | 平行光，比如太阳光 |
| [SpotLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/SpotLight) | 聚光源             |

**环境光创建**

```javascript
//环境光    环境光颜色与网格模型的颜色进行RGB进行乘法运算
var ambient = new THREE.AmbientLight(0x444444);
scene.add(ambient);
```

**点光源创建**

```javascript
//点光源
var point = new THREE.PointLight(0xffffff);
point.position.set(400, 200, 300); //点光源位置
// 通过add方法插入场景中，不插入的话，渲染的时候不会获取光源的信息进行光照计算
scene.add(point); //点光源添加到场景中
```

光源通过`add`方法插入场景中，不插入的话，渲染的时候不会获取光源的信息进行光照计算

### 光源光照强度

通过光源构造函数的参数可以设置光源的颜色，一般设置明暗程度不同的白光RGB三个分量值是一样的。如果把`THREE.AmbientLight(0x444444);`的光照参数`0x444444`改为`0xffffff`,你会发现场景中的立方体渲染效果更明亮。

### 光源位置

```javascript
//点光源
var point = new THREE.PointLight(0xffffff);
point.position.set(400, 200, 300); //点光源位置
scene.add(point); //点光源添加到场景中
```

你可以把点光源的位置设置为`(0,0,0)`，然后不使用其它任何光源，这时候你会发现场景中立方体渲染效果是黑色。其实原因很简单，立方体是有大小占，用一定空间的，如果光源位于立方体里面，而不是外部，自然无法照射到立方体外表面。

```javascript
point.position.set(0, 0, 0);
```

如果只设置一个点光源的情况下，你通过鼠标旋转操作整个三维场景，你会发现立方体点光源无法照射的地方相对其他位置会比较暗，你可以通过下面的代码在新的位置插入一个新的光源对象。点光源设置的位置是(-400, -200, -300)，相当于把立方体夹在两个点光源之间。

```javascript
// 点光源2  位置和point关于原点对称
var point2 = new THREE.PointLight(0xffffff);
point2.position.set(-400, -200, -300); //点光源位置
scene.add(point2); //点光源添加到场景中
```

# 顶点概念、几何体结构

## 1 顶点位置数据解析渲染

重点可以放在学习threejs的API使用细节，threejs引擎是如何封装webgl的。

### JavaScript类型化数组

本节课会用到javascript的类型化数组，你如果不了解，可查看MDN关于[javascript类型化数组](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Typed_arrays)的介绍,也可以查看文章[类型化数组](http://www.yanhuangxueyuan.com/Javascript/typeArray.html)。

### 自定义几何体

​	你可以直接调用`BoxGeometry`直接创建一个立方体几何体，调用`SphereGeometry`创建一个球体几何体。不过为了大家更好的建立顶点概念，通过下面的代码自定义了一个几何体，通过网格模型可以渲染出来两个三角形效果。

![三角形模式](threejs16mesh.jpg)

​	下面代码通过Threejs引擎的[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)和[BufferAttribute](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferAttribute)两个API自定义了一个具有六个顶点数据的几何体。

```javascript
var geometry = new THREE.BufferGeometry(); //创建一个Buffer类型几何体对象
//类型数组创建顶点数据
var vertices = new Float32Array([
  0, 0, 0, //顶点1坐标
  50, 0, 0, //顶点2坐标
  0, 100, 0, //顶点3坐标
  0, 0, 10, //顶点4坐标
  0, 0, 100, //顶点5坐标
  50, 0, 10, //顶点6坐标
]);
// 创建属性缓冲区对象
var attribute = new THREE.BufferAttribute(vertices, 3); //3个为一组，表示一个顶点的xyz坐标
// 设置几何体attributes属性的位置属性
geometry.attributes.position = attribute;
```

通过自定义的几何体创建一个网格模型。对于网格模型而言，几何体所有顶点每三个顶点为一组可以确定一个三角形，几何体是六个顶点，也就是说可以绘制两个三角形，当然了你可以自己再增加三个顶点位置坐标数据，测试下渲染效果。

```javascript
// 三角面(网格)渲染模式
var material = new THREE.MeshBasicMaterial({
  color: 0x0000ff, //三角面颜色
  side: THREE.DoubleSide //两面可见
}); //材质对象
var mesh = new THREE.Mesh(geometry, material); //网格模型对象Mesh
```

### 点模型`Points`

为了更好的理解几何体是由顶点构成的，可以把几何体geometry作为点模型`Points`而不是网格模型`Mesh`的参数，你会发现上面的六个点坐标会渲染为六个方形的点区域，可以用下面代码代替上面的网格模型部分代码测试效果。

对于网格模型`Mesh`而言，几何体geometry三个顶点为一组渲染出来一个三角形，对于点模型`Points`而言，几何体的每个顶点对应位置都会渲染出来一个方形的点区域，该区域可以设置大小。

```javascript
// 点渲染模式
var material = new THREE.PointsMaterial({
  color: 0xff0000,
  size: 10.0 //点对象像素尺寸
}); //材质对象
var points = new THREE.Points(geometry, material); //点模型对象
scene.add(points); //点对象添加到场景中
```

![point](threejs16point.jpg)

### 线模型`Line`

上面两个案例适用点模型和网格模型去渲染几何体的顶点数据，下面代码是把几何体作为线模型`Line`参数，你会发现渲染效果是从第一个点开始到最后一个点，依次连成线。

```javascript
// 线条渲染模式
var material=new THREE.LineBasicMaterial({
    color:0xff0000 //线条颜色
});//材质对象
var line=new THREE.Line(geometry,material);//线条模型对象
scene.add(line);//线条对象添加到场景中
```

![point](threejs16line.jpg)

## 2 顶点颜色数据插值计算

​	通常几何体顶点位置坐标数据和几何体顶点颜色数据都是一一对应的，比如顶点1有一个顶点位置坐标数据，也有一个顶点颜色数据，顶点2同样也有一个顶点位置坐标数据，也有一个顶点颜色数据...

### 每个顶点设置一种颜色

你可以在上节课代码更改为下面代码设置，你可以看到几何体的六个顶点分别渲染为几何体设置的顶点颜色数据。

![point.jpg](threejs17point.jpg)

```javascript
var geometry = new THREE.BufferGeometry(); //声明一个缓冲几何体对象

//类型数组创建顶点位置position数据
var vertices = new Float32Array([
  0, 0, 0, //顶点1坐标
  50, 0, 0, //顶点2坐标
  0, 100, 0, //顶点3坐标

  0, 0, 10, //顶点4坐标
  0, 0, 100, //顶点5坐标
  50, 0, 10, //顶点6坐标
]);
// 创建属性缓冲区对象
var attribue = new THREE.BufferAttribute(vertices, 3); //3个为一组，作为一个顶点的xyz坐标
// 设置几何体attributes属性的位置position属性
geometry.attributes.position = attribue;
//类型数组创建顶点颜色color数据
var colors = new Float32Array([
  1, 0, 0, //顶点1颜色
  0, 1, 0, //顶点2颜色
  0, 0, 1, //顶点3颜色

  1, 1, 0, //顶点4颜色
  0, 1, 1, //顶点5颜色
  1, 0, 1, //顶点6颜色
]);
// 设置几何体attributes属性的颜色color属性
geometry.attributes.color = new THREE.BufferAttribute(colors, 3); //3个为一组,表示一个顶点的颜色数据RGB
//材质对象
var material = new THREE.PointsMaterial({
  // 使用顶点颜色数据渲染模型，不需要再定义color属性
  // color: 0xff0000,
  vertexColors: THREE.VertexColors, //以顶点颜色为准
  size: 10.0 //点对象像素尺寸
});
// 点渲染模式  点模型对象Points
var points = new THREE.Points(geometry, material); //点模型对象
scene.add(points); //点对象添加到场景
```

### 材质属性`.vertexColors`

​	你可以看到上面案例的材质代码和前面稍有不同，原来是通过材质的颜色属性`color`设置模型颜色，而本案例并没有这样设置，而是设置了材质属性`.vertexColors`。

```javascript
var material = new THREE.PointsMaterial({
  // 使用顶点颜色数据渲染模型，不需要再定义color属性
  // color: 0xff0000,
  vertexColors: THREE.VertexColors, //以顶点颜色为准
  size: 10.0 //点对象像素尺寸
});
```

​	关于材质的属性`.vertexColors`可以查看[Material文档](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/Material)介绍，属性`.vertexColors`的默认值是`THREE.NoColors`，这也就是说模型的颜色渲染效果取决于材质属性`.color`，**如果把材质属性`.vertexColors`的值设置为`THREE.VertexColors`,threejs渲染模型的时候就会使用几何体的顶点颜色数据`geometry.attributes.color`。**

### 属性缓冲区对象`BufferAttribute`

​	Threejs提供的接口[BufferAttribute](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferAttribute)**目的是为了创建各种各样顶点数据**，比如顶点颜色数据，顶点位置数据，**然后作为几何体`BufferGeometry`的顶点位置坐标属性`BufferGeometry.attributes.position`、顶点颜色属性`BufferGeometry.attributes.color`的值。**

​	缓冲类型几何体`BufferGeometry`除了顶点位置、顶点颜色属性之外还有其他顶点属性，后面课程都会讲解到。关于`BufferGeometry`更多属性和方法可以查看文档[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)。

### 颜色插值

​	如果你把几何体作为网格模型`Mesh`或者线模型`Line`构造函数的参数，你会发现渲染出渐变的彩色效果。

![line](threejs17line.jpg)

![line](threejs17mesh.jpg)

​	之所以出现渐变是因为Threejs通过底层WebGL进行渲染的时候会对顶点的颜色数据进行插值计算。颜色插值计算简单点说，比如一条直线的端点1设置为红色，端点2设置为蓝色，整条直线就会呈现出从点1到红色点2的蓝色颜色渐变，对于网格模型Mesh而言，就是三角形的三个顶点分别设置一个颜色，三角形内部的区域像素会根据三个顶点的颜色进行插值计算。

插值计算示意图

![插值计算](threejs17插值计算.png)

## 3 顶点法向量数据光照计算

​	前面两节课讲解到了顶点位置坐标数据、顶点颜色数据，这节课讲解第三种顶点数据：顶点法向量。

​	如果你有初高中物理的光学基础，应该会有漫反射、镜面反射的概念。比如太阳光照在一个物体表面，物体表面与光线夹角位置不同的区域明暗程度不同，WebGL中为了计算光线与物体表面入射角，你首先要计算物体表面每个位置的法线方向，在Threejs中表示物体的网格模型Mesh的曲面是由一个一个三角形构成，所以为了表示物体表面各个位置的法线方向，可以给几何体的每个顶点定义一个方向向量。

![立方体](threejs18webgl.png)

![立方体](threejs18332.jpg)

### 不设置顶点法向量数据

下面代码仅仅定义了几何体[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)的顶点位置数据，没有定义顶点法向量数据。没有法向量数据，点光源、平行光等带有方向性的光源不会起作用，三角形平面整个渲染效果相对暗淡，而且两个三角形分界位置没有棱角感。

![img](threejs18无.jpg)

```javascript
var geometry = new THREE.BufferGeometry(); //声明一个空几何体对象
//类型数组创建顶点位置position数据
var vertices = new Float32Array([
  0, 0, 0, //顶点1坐标
  50, 0, 0, //顶点2坐标
  0, 100, 0, //顶点3坐标

  0, 0, 0, //顶点4坐标
  0, 0, 100, //顶点5坐标
  50, 0, 0, //顶点6坐标

]);
// 创建属性缓冲区对象
var attribue = new THREE.BufferAttribute(vertices, 3); //3个为一组
// 设置几何体attributes属性的位置position属性
geometry.attributes.position = attribue
```

### 定义几何体顶点法向量数据

在上面顶点位置数据基础上定义顶点法向量数据，这时候除了环境光以外，点光源也会参与光照计算，三角形整个表面比较明亮，同时两个三角形表面法线不同，即使光线方向相同，明暗自然不同，在分界位置有棱角感。

![img](threejs18有.jpg)

```javascript
var normals = new Float32Array([
  0, 0, 1, //顶点1法向量
  0, 0, 1, //顶点2法向量
  0, 0, 1, //顶点3法向量

  0, 1, 0, //顶点4法向量
  0, 1, 0, //顶点5法向量
  0, 1, 0, //顶点6法向量
]);
// 设置几何体attributes属性的位置normal属性
geometry.attributes.normal = new THREE.BufferAttribute(normals, 3); //3个为一组,表示一个顶点的法向量数据
```

顶点法向量数据和前面两节课讲解到的顶点位置数据、顶点颜色数据一样都是一一对应的。

### API使用总结

```javascript
// 访问几何体顶点位置数据
BufferGeometry.attributes.position
// 访问几何体顶点颜色数据
BufferGeometry.attributes.color
// 访问几何体顶点法向量数据
BufferGeometry.attributes.normal
```

几何体[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)的顶点法向量数据和几何体位置、颜色等顶点数据一样使用[BufferAttribute](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferAttribute)表示。

```javascript
// 设置几何体attributes属性的位置normal属性
geometry.attributes.normal = new THREE.BufferAttribute(normals, 3); //3个为一组,表示一个顶点的法向量
```

## 4 顶点索引复用顶点数据

​	通过几何体[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)的顶点索引属性`BufferGeometry.index`可以设置几何体顶点索引数据，如果你有WebGL基础很容易理解顶点索引的概念，如果没有也没有关系，下面会通过一个简单的例子形象说明。

​	比如绘制一个矩形网格模型,至少需要两个三角形拼接而成，两个三角形，每个三角形有三个顶点，也就是说需要定义6个顶点位置数据。对于矩形网格模型而言，两个三角形有两个顶点位置是重合的。也就是说可以重复的位置可以定义一次，然后通过通过顶点数组的索引值获取这些顶点位置数据。

![index](threejs19index.jpg)

### 不使用顶点索引

下面通过几何体六个顶点定义了两个三角形，几何体的顶点位置数据、顶点法向量数据都是6个。

```javascript
var geometry = new THREE.BufferGeometry(); //声明一个空几何体对象
//类型数组创建顶点位置position数据
var vertices = new Float32Array([
  0, 0, 0, //顶点1坐标
  80, 0, 0, //顶点2坐标
  80, 80, 0, //顶点3坐标

  0, 0, 0, //顶点4坐标   和顶点1位置相同
  80, 80, 0, //顶点5坐标  和顶点3位置相同
  0, 80, 0, //顶点6坐标
]);
// 创建属性缓冲区对象
var attribue = new THREE.BufferAttribute(vertices, 3); //3个为一组
// 设置几何体attributes属性的位置position属性
geometry.attributes.position = attribue
var normals = new Float32Array([
  0, 0, 1, //顶点1法向量
  0, 0, 1, //顶点2法向量
  0, 0, 1, //顶点3法向量

  0, 0, 1, //顶点4法向量
  0, 0, 1, //顶点5法向量
  0, 0, 1, //顶点6法向量
]);
// 设置几何体attributes属性的位置normal属性
geometry.attributes.normal = new THREE.BufferAttribute(normals, 3); //3个为一组,表示一个顶点的xyz坐标
```

### 顶点索引`.index`

下面代码通过几何体[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)的顶点索引`BufferGeometry.index`定义了一个矩形。通过顶点索引组织网格模型三角形的绘制，因为矩形的两个三角形有两个顶点位置重复，所以顶点位置数据、顶点法向量数据都只需要定义4个就可以。

```javascript
var geometry = new THREE.BufferGeometry(); //声明一个空几何体对象
//类型数组创建顶点位置position数据
var vertices = new Float32Array([
  0, 0, 0, //顶点1坐标
  80, 0, 0, //顶点2坐标
  80, 80, 0, //顶点3坐标
  0, 80, 0, //顶点4坐标
]);
// 创建属性缓冲区对象
var attribue = new THREE.BufferAttribute(vertices, 3); //3个为一组
// 设置几何体attributes属性的位置position属性
geometry.attributes.position = attribue
var normals = new Float32Array([
  0, 0, 1, //顶点1法向量
  0, 0, 1, //顶点2法向量
  0, 0, 1, //顶点3法向量
  0, 0, 1, //顶点4法向量
]);
// 设置几何体attributes属性的位置normal属性
geometry.attributes.normal = new THREE.BufferAttribute(normals, 3); //3个为一组,表示一个顶点的xyz坐标
```

通过顶点索引组织顶点数据，顶点索引数组`indexes`通过索引值指向顶点位置`geometry.attributes.position`、顶点法向量`geometry.attributes.normal`中顶面数组。

```javascript
// Uint16Array类型数组创建顶点索引数据
var indexes = new Uint16Array([
  // 0对应第1个顶点位置数据、第1个顶点法向量数据
  // 1对应第2个顶点位置数据、第2个顶点法向量数据
  // 索引值3个为一组，表示一个三角形的3个顶点
  0, 1, 2,
  0, 2, 3,
])
// 索引数据赋值给几何体的index属性
geometry.index = new THREE.BufferAttribute(indexes, 1); //1个为一组
```

创建顶点索引数组的时候，可以根据顶点的数量选择类型数组`Uint8Array`、`Uint16Array`、`Uint32Array`。**对于顶点索引而言选择整型类型数组，对于非索引的顶点数据，需要使用浮点类型数组`Float32Array`等。**

| 类型数组     | 位数 | 字节 | 类型描述           | C语言等价类型 |
| :----------- | :--- | :--- | :----------------- | :------------ |
| Int8Array    | 8    | 1    | 有符号8位整型      | int8_t        |
| Uint8Array   | 8    | 1    | 无符号8位整型      | uint8_t       |
| Int16Array   | 16   | 2    | 有符号16位整型     | int16_t       |
| Uint16Array  | 16   | 2    | 无符号16位整型     | int16_t       |
| Int32Array   | 32   | 4    | 有符号32位整型     | int32_t       |
| Uint32Array  | 32   | 4    | 无符号32位整型     | uint32_t      |
| Float32Array | 32   | 4    | 单精度(32位)浮点数 | float         |
| Float64Array | 64   | 8    | 双精度(64位)浮点数 | double        |

### `BufferGeometry`总结

![image-20220922174750553](image-20220922174750553.png)

## 5 `Geometry`

​	本节课给大家引入一个新的threejs几何体API[Geometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Geometry)。**几何体`Geometry`和缓冲类型几何体`BufferGeometry`表达的含义相同，只是对象的结构不同，Threejs渲染的时候会先把`Geometry`转化为`BufferGeometry`再解析几何体顶点数据进行渲染。**

### [Vector3](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/math/Vector3)定义顶点位置坐标数据

​	`Vector3`是threejs的三维向量对象,可以通过`Vector3`对象表示一个顶点的xyz坐标，顶点的法线向量。

​	几何体`Geometry`的顶点位置属性`geometry.vertices`和缓冲类型几何体`BufferGeometry`顶点位置属性`BufferGeometry.attributes.position`是对应的。

```javascript
var geometry = new THREE.Geometry(); //声明一个几何体对象Geometry
var p1 = new THREE.Vector3(50, 0, 0); //顶点1坐标
var p2 = new THREE.Vector3(0, 70, 0); //顶点2坐标
var p3 = new THREE.Vector3(80, 70, 0); //顶点3坐标
//顶点坐标添加到geometry对象
geometry.vertices.push(p1, p2, p3);
```

### [Color](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/math/Color)定义顶点颜色数据

​	**通过threejs顶点颜色对象`Color`可以定义几何体顶点颜色数据，然后顶点颜色数据构成的数组作为几何体`Geometry`顶点颜色属性`geometry.colors`的值。**

​	几何体`Geometry`的顶点颜色属性`geometry.colors`和缓冲类型几何体`BufferGeometry`顶点颜色属性`BufferGeometry.attributes.color`是对应的。

```javascript
// Color对象表示顶点颜色数据
var color1 = new THREE.Color(0x00ff00); //顶点1颜色——绿色
var color2 = new THREE.Color(0xff0000); //顶点2颜色——红色
var color3 = new THREE.Color(0x0000ff); //顶点3颜色——蓝色
//顶点颜色数据添加到geometry对象
geometry.colors.push(color1, color2, color3);
```

​	**注意设置几何体`Geometry`顶点颜色属性`geometry.colors`，对网格模型`Mesh`是无效的**，对于点模型`Points`、线模型`Line`是有效果。网格模型情况几何体`Geometry`顶点颜色如何设置可以参考下一节课。

### 材质属性`.vertexColors`

**注意使用顶点颜色数据定义模型颜色的时候，要把材质的属性`vertexColors`设置为`THREE.VertexColors`,这样顶点的颜色数据才能取代材质颜色属性`.color`起作用。**

```javascript
//材质对象
var material = new THREE.MeshLambertMaterial({
  // color: 0xffff00,
  vertexColors: THREE.VertexColors, //以顶点颜色为准
  side: THREE.DoubleSide, //两面可见
});
```

## 6 `Face3`对象定义`Geometry`的三角形面

几何体[Geometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Geometry)的三角面属性`geometry.faces`和缓冲类型几何体[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)顶点索引属性`BufferGeometry.index`类似**都是顶点位置数据的索引值**,用来组织网格模型三角形的绘制。

学习本节课最好对照`2.4 顶点索引复用顶点数据`学习。

下面代码自定义了一个由两个三角形构成的几何体，两个三角形有两个顶点坐标位置是重合的。

![img](threejs211.jpg)

```javascript
var geometry = new THREE.Geometry(); //声明一个几何体对象Geometry

var p1 = new THREE.Vector3(0, 0, 0); //顶点1坐标
var p2 = new THREE.Vector3(0, 100, 0); //顶点2坐标
var p3 = new THREE.Vector3(50, 0, 0); //顶点3坐标
var p4 = new THREE.Vector3(0, 0, 100); //顶点4坐标
//顶点坐标添加到geometry对象
geometry.vertices.push(p1, p2, p3,p4);

// Face3构造函数创建一个三角面
var face1 = new THREE.Face3(0, 1, 2); //类似索引的用法
//三角面每个顶点的法向量
var n1 = new THREE.Vector3(0, 0, -1); //三角面Face1顶点1的法向量
var n2 = new THREE.Vector3(0, 0, -1); //三角面2Face2顶点2的法向量
var n3 = new THREE.Vector3(0, 0, -1); //三角面3Face3顶点3的法向量
// 设置三角面Face3三个顶点的法向量
face1.vertexNormals.push(n1,n2,n3);

// 三角面2
var face2 = new THREE.Face3(0, 2, 3);
// 设置三角面法向量
face2.normal=new THREE.Vector3(0, -1, 0);

//三角面face1、face2添加到几何体中
geometry.faces.push(face1,face2);
```

### 设置四个顶点

​	两个三角形有6个顶点，但是两个顶点位置重合的，可以设置4个顶点即可。

```javascript
var p1 = new THREE.Vector3(0, 0, 0); //顶点1坐标
var p2 = new THREE.Vector3(0, 100, 0); //顶点2坐标
var p3 = new THREE.Vector3(50, 0, 0); //顶点3坐标
var p4 = new THREE.Vector3(0, 0, 100); //顶点4坐标
//顶点坐标添加到geometry对象
geometry.vertices.push(p1, p2, p3,p4);
```

### [Face3](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Face3)构建三角形

​	threejs提供了`Face3`对象构建三角形，通过`Face3`构建一个三角形，不要设置顶点位置坐标数据，只需要通过数组索引值从`geometry.vertices`数组中获得顶点位置坐标数据。

`	geometry.vertices`数组索引0, 1, 2对应的顶点位置坐标数据表示三角形1的三个顶点坐标，索引0, 2, 3对应的顶点位置坐标数据表示三角形2的三个顶点坐标。

```javascript
// Face3构造函数创建一个三角面
var face1 = new THREE.Face3(0, 1, 2);
// 三角面2
var face2 = new THREE.Face3(0, 2, 3);
```

### 三角形法线设置

​	前面课程将结果网格模型Mesh的几何体Geometry本质上都是一个一个三角形拼接而成，所以可以通过设置三角形的法线方向向量来表示几何体表面各个位置的法线方向向量。

​	**设置三角形法线方向向量有两种方式：一种是直接定义三角形面的法线方向，另一个是定义三角形三个顶点的法线方向数据来表示三角形面法线方向。**

​	使用三维向量`THREE.Vector3`表示三角形法线方向数值，然后赋值给三角形对象`Face3`的法线属性`Face3.normal`。

```javascript
// 三角面2
var face2 = new THREE.Face3(0, 2, 3);
// 设置三角面法向量
face2.normal=new THREE.Vector3(0, -1, 0);
```

换另一种方式，通过三角形面`Face3`的`Face3.vertexNormals`属性给三角形的三个顶点分别设置一个顶点法线方向数据。

```javascript
// Face3构造函数创建一个三角面
var face1 = new THREE.Face3(0, 1, 2);
//三角面每个顶点的法向量
var n1 = new THREE.Vector3(0, 0, -1); //三角面Face1顶点1的法向量
var n2 = new THREE.Vector3(0, 0, -1); //三角面2Face2顶点2的法向量
var n3 = new THREE.Vector3(0, 0, -1); //三角面3Face3顶点3的法向量
// 设置三角面Face3三个顶点的法向量
face1.vertexNormals.push(n1,n2,n3);
```

### 三角形颜色设置

​	三角形颜色设置和三角形法线方向设置类型，可以直接设置三角形颜色，也可以设置三角形三个顶点的颜色。

```javascript
// 三角形1颜色
face1.color = new THREE.Color(0xffff00);
// 设置三角面face1三个顶点的颜色
face1.color = new THREE.Color(0xff00ff);
```

通过三角形面`Face3`的`.vertexColors`属性设置三角形三个顶点颜色。

三个顶点颜色不同三角形面渲染的时候会进行颜色插值计算，测到一个颜色渐变效果。

```javascript
face1.vertexColors = [
  new THREE.Color(0xffff00),
  new THREE.Color(0xff00ff),
  new THREE.Color(0x00ffff),
]
```

​	**使用顶点颜色数据的时候，注意设置材质的属性`vertexColors`属性值为`THREE.VertexColors`。**

​	**注意设置三角形`Face3`的颜色对threejs网格模型`Mesh`有效，对于点模型`Points`、线模型`Line`是无效果**，如果想设置点、线模型对应的几何体`Geometry`的顶点颜色，可以通过`Geometry`的顶点颜色属性`geometry.colors`实现。

## 7 访问几何体对象的数据

实际开发项目的时候，可能会加载外部模型，有些时候需要获取模型几何体的顶点数据，如果想获取几何体的顶点数据首先要熟悉three.js几何体`BoxGeometry`和`BufferGeometry`的结构。

访问几何体顶点数据其实很简单，刚开始学习不用刻意记忆，直接查看threejs文档，就像访问javascript对象的属性一样。

### 测试`BoxGeometry`

调用`BoxGeometry`创建一个立方体，执行`THREE.BoxGeometry`构造函数会自动生成几何体对象的顶点位置坐标、顶点法向量等数据。

你可以通过执行下面代码，然后查看浏览器控制台打印的数据

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100); //创建一个立方体几何对象Geometry
console.log(geometry);
console.log('几何体顶点位置数据',geometry.vertices);
console.log('三角行面数据',geometry.faces);
```

`BoxGeometry`、`PlaneGeometry`、`SphereGeometry`等几何体类的基类是`Geometry`，所以访问这些几何体的顶点数据,不知道具体属性名称，可以查问threejs文档[Geometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Geometry)

![image-20220922182143727](image-20220922182143727.png)

### 测试`PlaneBufferGeometry`

`PlaneBufferGeometry`表示一个矩形平面几何体，执行下面代码，你可以查看该几何体的相关顶点数据。

```javascript
//创建一个矩形平面几何体
var geometry = new THREE.PlaneBufferGeometry(100, 100);
console.log(geometry);
console.log('几何体顶点位置数据',geometry.attributes.position);
console.log('几何体索引数据',geometry.index);
```

`BoxBufferGeometry`、`PlaneBufferGeometry`、`SphereBufferGeometry`等几何体类的基类是`BufferGeometry`，所以访问这些几何体的顶点数据,不知道具体属性名称，可以查问threejs文档[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)

![image-20220922182115422](image-20220922182115422.png)

### 案例

通过下面代码修改`BoxGeometry`的三角形顶点颜色的数据，可以渲染出来如下效果。

![img](threejs22ex1.jpg)

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100); //创建一个立方体几何对象Geometry
// 遍历几何体的face属性
geometry.faces.forEach(face => {
  // 设置三角面face三个顶点的颜色
  face.vertexColors = [
    new THREE.Color(0xffff00),
    new THREE.Color(0xff00ff),
    new THREE.Color(0x00ffff),
  ]
});
var material = new THREE.MeshBasicMaterial({
  // color: 0x0000ff,
  vertexColors: THREE.FaceColors,
  // wireframe:true,//线框模式渲染
}); //材质对象Material
```



你可以通过下面代码删除立方体部分三角形面，测试删除效果。

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100); //创建一个立方体几何对象Geometry
// pop()：删除数组的最后一个元素   shift：删除数组的第一个元素
geometry.faces.pop();
geometry.faces.pop();
geometry.faces.shift();
geometry.faces.shift();
var material = new THREE.MeshLambertMaterial({
  color: 0x0000ff,
  side: THREE.DoubleSide, //两面可见
}); //材质对象Material
```

### 访问外部模型几何体顶点数据

Threejs加载外部模型的时候，会把几何体解析为缓冲类型几何体`BufferGeometry`，所以访问外部模型几何体顶点数据，可以查看文档[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)。关于外部模型加载的讲解可以查看课程第14章。

## 8 几何体旋转、缩放、平移变换

通过前7节课的学习，我相信你已经对Threejs几何体内部顶点构成有了一定了解。

![image-20220922181947048](image-20220922181947048.png)

几何体[Geometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Geometry)对象有一系列的顶点属性，也封装了一系列的方法，通过`.scale()`、`.translate()`、`.rotateX()`等方法可以对几何体本身进行缩放、平移、旋转等几何变换。通过`.scale()`、`.translate()`、`.rotateX()`这些方法对几何体进行变换，注意本质上都是改变结合体顶点位置坐标数据。你可以执行测斜方法，然后在浏览器控制打印顶点位置坐标数据`console.log(geometry.vertices);`,然后对比几何体变化前后定点位置坐标是否变化。

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100); //创建一个立方体几何对象Geometry
// 几何体xyz三个方向都放大2倍
geometry.scale(2, 2, 2);
// 几何体沿着x轴平移50
geometry.translate(50, 0, 0);
// 几何体绕着x轴旋转45度
geometry.rotateX(Math.PI / 4);
// 居中：偏移的几何体居中
geometry.center();
console.log(geometry.vertices);
```

[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)和几何体[Geometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Geometry) 一样具有`.scale()`、`.rotateZ()`、`.rotateX()`等几何体变换的方法。

### 注意

注意网格模型`Mesh`进行缩放旋转平移变换和几何体`Geometry`可以实现相同的渲染效果，但是网格模型`Mesh`进行这些变换不会影响几何体的顶点位置坐标，网格模型缩放旋转平移变换改变的是模型的本地矩阵、世界矩阵。

你可以执行下面代码测试。

```javascript
// 几何体xyz方向分别缩放0.5,1.5,2倍
geometry.scale(0.5, 1.5, 2);
// 网格模型xyz方向分别缩放0.5,1.5,2倍
mesh.scale.set(0.5, 1.5, 2)
```

# 材质对象

​	本章节主要目的是认识Threejs的各类材质`Material`，所谓材质，简单地说就是字面意思，就像生活中你聊天一样，说这是塑料材质，这是金属材质，这是纤维材质...，深入一点说，就是包含光照算法的着色器GLSL ES代码。如果你想给一个模型设置特定的颜色，如果你想给一个模型设置一定透明度，如果你想实现一个金属效果，你想设置模型纹理贴图，那么Threejs的提供各种材质类就是你的选择。

​	本章节除了讲解Threejs材质知识以外，还会提一些javascript编程概念，比如类、基类、子类、属性、方法等，当然了，对于熟练使用javascript语言的程序员来说，提到这些概念完全是多余的，不过考虑到部分初学者可能也是刚刚接触javascript，就简单提下。**之所以强调这些内容，目的不出为了讲解这些javascript语法，而是强调Threejs所谓材质、几何体、相机等对象都是通过Threejs封装的类或者说构造函数实例化创建**，如果你了解Threejs类与类之间的继承关系，一方面有助于你系统学习Threejs，另一方面查询[Threejs文档](http://example.com/)的时候知道去查看那个API。

## 常用材质介绍

为了方便开发Threejs提供了一系列的材质，所有材质就是对WebGL着色器代码的封装，如果你不了解WebGL，会通过查阅Threejs文档使用相关材质类即可。

![image-20220923153314935](image-20220923153314935.png)

### 点材质`PointsMaterial`

点材质比较简单，只有`PointsMaterial`,通常使用点模型的时候会使用点材质`PointsMaterial`。

点材质`PointsMaterial`的`.size`属性可以每个顶点渲染的方形区域尺寸像素大小。

```javascript
var geometry = new THREE.SphereGeometry(100, 25, 25); //创建一个球体几何对象
// 创建一个点材质对象
var material = new THREE.PointsMaterial({
  color: 0x0000ff, //颜色
  size: 3, //点渲染尺寸
});
//点模型对象  参数：几何体  点材质
var point = new THREE.Points(geometry, material);
scene.add(point); //网格模型添加到场景中
```

### 线材质

线材质有基础线材质`LineBasicMaterial`和虚线材质`LineDashedMaterial`两个，通常使用使用`Line`等线模型才会用到线材质。

基础线材质`LineBasicMaterial`。

```javascript
var geometry = new THREE.SphereGeometry(100, 25, 25);//球体
// 直线基础材质对象
var material = new THREE.LineBasicMaterial({
  color: 0x0000ff
});
var line = new THREE.Line(geometry, material); //线模型对象
scene.add(line); //点模型添加到场景中
```

虚线材质`LineDashedMaterial`。

```javascript
// 虚线材质对象：产生虚线效果
var material = new THREE.LineDashedMaterial({
  color: 0x0000ff,
  dashSize: 10,//显示线段的大小。默认为3。
  gapSize: 5,//间隙的大小。默认为1
});
var line = new THREE.Line(geometry, material); //线模型对象
//  computeLineDistances方法  计算LineDashedMaterial所需的距离数组
line.computeLineDistances();
```

### 网格模型

Threejs提供的网格类材质比较多，网格材质涉及的材质种类和材质属性也比较多，一节课也无法讲解完，本节课先有一个感性的认知。

网格材质顾名思义，网格类模型才会使用的材质对象。

基础网格材质对象`MeshBasicMaterial`,不受带有方向光源影响，没有棱角感。

```javascript
var material = new THREE.MeshBasicMaterial({
  color: 0x0000ff,
})
```

`MeshLambertMaterial`材质可以实现网格Mesh表面与光源的漫反射光照计算，有了光照计算，物体表面分界的位置才会产生棱角感。

```javascript
var material = new THREE.MeshLambertMaterial({
  color: 0x00ff00,
});
```

高光网格材质`MeshPhongMaterial`除了和`MeshLambertMaterial`一样可以实现光源和网格表面的漫反射光照计算，还可以产生高光效果(镜面反射)。

```javascript
var material = new THREE.MeshPhongMaterial({
  color: 0xff0000,
  specular:0x444444,//高光部分的颜色
  shininess:20,//高光部分的亮度，默认30
});
```

### 材质和模型对象对应关系

使用材质的时候，要注意材质和模型的对应关系，通过前面课程案例学习，目前为止你至少应该了解到了网格模型`Mesh`、点模型`Points`、线模型`Line`,随着课程的学习其它的模型对象也会接触到，这里先有个印象就可以。

![image-20220923153327747](image-20220923153327747.png)

## 材质共有属性、私有属性

如果你的javascript语言基础还可以，应该明白类、基类、子类、父类等概念。如果你有这些类的概念，那么在学习Threejs的过程中，如何查找Threejs文档将会比较顺利。

点材质`PointsMaterial`、基础线材质`LineBasicMaterial`、基础网格材质`MeshBasicMaterial`、高光网格材质`MeshPhongMaterial`等材质都是父类[Material](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/Material)的子类。

各种各样的材质子类都有自己的特定属性，比如点材质特有的尺寸属性`.size`、高光网格材质特有的高光颜色属性`.specular`等等这些属性可以成为子类材质的私有属性。

所有子类的材质都会从父类材质[Material](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/Material)继承透明度`opacity`、面`side`等属性，这些来自父类的属性都是子类共有的属性。

### `.side`属性

在Three.js开发过程中你可能会遇到下面的问题，比如three.js矩形平面`planegeometry`的网格模型插入场景看不到，一个球体或立方体网格模型如何背面显示贴图，正面不显示...，对于这些问题可以通过Three.js材质对象`.side`属性来设置。

材质`.side`属性的具体介绍可以查看Threejs文档中所有材质对象的基类`Material`。

`.side`属性的属性值定义面的渲染方式前面后面 或 双面. 属性的默认值是`THREE.FrontSide`，表示前面. 也可以设置为后面`THREE.BackSide` 或 双面`THREE.DoubleSide`.

```javascript
var material = new THREE.MeshBasicMaterial({
  color: 0xdd00ff,
  // 前面FrontSide  背面：BackSide 双面：DoubleSide
  side:THREE.DoubleSide,
});
```

### 材质透明度`.opacity`

通过材质的透明度属性`.opacity`可以设置材质的透明程度，`.opacity`属性值的范围是0.0~1.0，0.0值表示完全透明, 1.0表示完全不透明，`.opacity`默认值1.0。

当设置`.opacity`属性值的时候，需要设置材质属性`transparent`值为`true`，如果材质的transparent属性没设置为true, 材质会保持完全不透明状态。

在构造函数参数中设置`transparent`和`.opacity`的属性值

```javascript
var material = new THREE.MeshPhongMaterial({
  color: 0x220000,
  // transparent设置为true，开启透明，否则opacity不起作用
  transparent: true,
  // 设置材质透明度
  opacity: 0.4,
});
```

通过访问材质对象属性形式设置`transparent`和`.opacity`的属性值

```javascript
  // transparent设置为true，开启透明，否则opacity不起作用
material.transparent = true;
  // 设置材质透明度
material.opacity = 0.4;
```

# 模型对象

## 1 点、线、网格模型介绍

经过前面几章学习相信你对点模型[Points](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/objects/Points)、线模型[Line](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/objects/Line)、网格模型[Mesh](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/objects/Mesh)已经有了大致了解，本节课就对点、线、网格模型模型进行简单总结。

![image-20220923155120518](image-20220923155120518.png)

点模型`Points`、线模型`Line`、网格网格模型`Mesh`都是由几何体Geometry和材质Material构成，这三种模型的区别在于对几何体顶点数据的渲染方式不同，如果有一定WebGL基础，就更容易理解这一点了。

### 点模型`Points`

点模型`Points`就是几何体的每一个顶点数据渲染为一个方形区域，方形区域的大小可以设置。

![webgl](1577279341491.jpg)

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100); //创建一个立方体几何对象Geometry
// 点渲染模式
var material = new THREE.PointsMaterial({
  color: 0xff0000,
  size: 5.0 //点对象像素尺寸
}); //材质对象
var points = new THREE.Points(geometry, material); //点模型对象
```

### 线模型`Line`

两点确定一条直线，线模型`Line`就是使用线条去连接几何体的顶点数据。 ![webgl](1577279383015.jpg)

线模型除了[Line](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/objects/Line)还有[LineLoop](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/objects/LineLoop)和[LineSegments](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/objects/LineSegments),`LineLoop`和`Line`区别是连线的时候会闭合把第一个顶点和最后一个顶点连接起来，`LineSegments`则是顶点不共享，第1、2点确定一条线，第3、4顶点确定一条直线，第2和3点之间不连接。

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100); //创建一个立方体几何对象Geometry
// 线条渲染模式
var material=new THREE.LineBasicMaterial({
    color:0xff0000 //线条颜色
});//材质对象
// 创建线模型对象   构造函数：Line、LineLoop、LineSegments
var line=new THREE.Line(geometry,material);//线条模型对象
```

### 网格模型`Mesh`

三个顶点确定一个三角形，网格模型[Mesh](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/objects/Mesh)默认的情况下，通过三角形面绘制渲染几何体的所有顶点，通过一系列的三角形拼接出来一个曲面。

![webgl](1577279465519.jpg)

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100);
// 三角形面渲染模式  
var material = new THREE.MeshLambertMaterial({
  color: 0x0000ff, //三角面颜色
}); //材质对象
var mesh = new THREE.Mesh(geometry, material); //网格模型对象Mesh
```

如果设置网格模型的`wireframe`属性为`true`，所有三角形会以线条形式绘制出来。开发的时候可以通过设置wireframe属性来查看网格模型的三角形分布特点。

![webgl](1577279497254.jpg)

```javascript
var material = new THREE.MeshLambertMaterial({
  color: 0x0000ff, //三角面颜色
  wireframe:true,//网格模型以线条的模式渲染
});
// 通过访问属性的形式设置
material.wireframe = true;
```

## 2 模型对象旋转平移缩放变换

点模型`Points`、线模型`Line`、网格网格模型`Mesh`等模型对象的基类都是[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)，如果想对这些模型进行旋转、缩放、平移等操作，如何实现，可以查询Threejs文档[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)对相关属性和方法的介绍。

![image-20220923155203225](image-20220923155203225.png)

### 缩放

网格模型`Mesh`的属性`.scale`表示模型对象的缩放比例，默认值是`THREE.Vector3(1.0,1.0,1.0)`,`.scale`的属性值是一个三维向量对象[Vector3](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/math/Vector3)，查看three.js文档你可以知道`Vector3`对象具有属性`.x`、`.y`、`.z`，`Vector3`对象还具有方法`.set()`，`.set`方法有三个表示xyz方向缩放比例的参数。

网格模型xyz方向分别缩放0.5,1.5,2倍

```javascript
mesh.scale.set(0.5, 1.5, 2)
```

x轴方向放大2倍

```javascript
mesh.scale.x = 2.0;
```

### 位置属性`.position`

模型位置`.position`属性和`.scale`属性的属性值一样也是三维向量对象[Vector3](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/math/Vector3)，通过模型位置属性`.position`可以设置模型在场景Scene中的位置。模型位置`.position`的默认值是`THREE.Vector3(0.0,0.0,0.0)`。

设置网格模型y坐标

```javascript
mesh.position.y = 80;
```

设置模型xyz坐标

```javascript
mesh.position.set(80,2,10);
```

### 平移

网格模型沿着x轴正方向平移100，可以多次执行该语句，每次执行都是相对上一次的位置进行平移变换。

```javascript
// 等价于mesh.position = mesh.position + 100;
mesh.translateX(100);//沿着x轴正方向平移距离100
```

沿着Z轴负方向平移距离50。

```javascript
mesh.translateZ(-50);
```

沿着自定义的方向移动。

```javascript
//向量Vector3对象表示方向
var axis = new THREE.Vector3(1, 1, 1);
axis.normalize(); //向量归一化
//沿着axis轴表示方向平移100
mesh.translateOnAxis(axis, 100);
```

执行`.translateX()`、`.translateY()`、`.translateOnAxis()`等方法本质上改变的都是模型的位置属性`.position`。

### 旋转

立方体网格模型绕立方体的x轴旋转π/4，可以多次执行该语句，每次执行都是相对上一次的角度进行旋转变化

```javascript
mesh.rotateX(Math.PI/4);//绕x轴旋转π/4
```

网格模型绕(0,1,0)向量表示的轴旋转π/8

```javascript
var axis = new THREE.Vector3(0,1,0);//向量axis
mesh.rotateOnAxis(axis,Math.PI/8);//绕axis轴旋转π/8
```

执行旋转`.rotateX()`等方法和执行平移`.translateY()`等方法一样都是对模型状态属性的改变，区别在于执行平移方法改变的是模型的位置属性`.position`，执行模型的旋转方法改变的是表示模型角度状态的角度属性`.rotation`或者四元数属性`.quaternion`。

模型的角度属性`.rotation`和四元数属性`.quaternion`都是表示模型的角度状态，只是表示方法不同，`.rotation`属性值是欧拉对象[Euler](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/math/Euler),`.quaternion`属性值是是四元数对象[Quaternion](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/math/Quaternion)

```javascript
// 绕着Y轴旋转90度
mesh.rotateY(Math.PI / 2);
//控制台查看：旋转方法，改变了rotation属性
console.log(mesh.rotation);
```

## 3 对象克隆`.clone()`和复制`.copy()`

Threejs大多数对象都有克隆`.clone()`和复制`.copy()`两个方法,点模型`Points`、线模型`Line`、网格网格模型`Mesh`一样具有这两个方法。

![image-20220923155218972](image-20220923155218972.png)

### 复制方法`.copy()`

`A.copy(B)`表示B属性的值赋值给A对应属性。

```javascript
var p1 = new THREE.Vector3(1.2,2.6,3.2);
var p2 = new THREE.Vector3(0.0,0.0,0.0);
p2.copy(p1)
// p2向量的xyz变为p1的xyz值
console.log(p2);
```

### 克隆方法`.clone()`

`N = M.clone()`表示返回一个和M相同的对象赋值给N。

```javascript
var p1 = new THREE.Vector3(1.2,2.6,3.2);
var p2 = p1.clone();
// p2对象和p1对象xyz属性相同
console.log(p2);
```

### 网格模型复制和克隆

网格模型复制克隆和三维向量基本逻辑是相同，但是注意三维向量`Vector3`的`.x`、`.y`、`.z`属性值是数字，也就是说是基本类型的数据，对于网格模型而言，网格模型对象的几何体属性`mesh.geometry`和材质属性`mesh.material`的属性值都是对象的索引值。

```javascript
var box=new THREE.BoxGeometry(10,10,10);//创建一个立方体几何对象
var material=new THREE.MeshLambertMaterial({color:0x0000ff});//材质对象


var mesh=new THREE.Mesh(box,material);//网格模型对象
var mesh2 = mesh.clone();//克隆网格模型
mesh.translateX(20);//网格模型mesh平移

scene.add(mesh,mesh2);//网格模型添加到场景中
```

缩放几何体box,你可以发现上面代码中的两个网格模型的大小都发生了变化，因为网格模型克隆的时候，mesh对象的几何体对象`mesh.geometry`属性值是box对象的索引值，返回的新对象mesh2几何体属性`mesh.geometry`的值同样是box对象的索引值。

```javascript
box.scale(1.5,1.5,1.5);//几何体缩放
```

### 注意

通过本节课的学习，对Threejs不同对象的克隆`.clone()`和复制`.copy()`方法有一个大致印象即可。

实际开发的时候，注意不同对象的复制或克隆方法可能稍有区别，使用的时候最好通过代码测试，或者直接查看threejs源码某个类对`.clone()`和`.copy()`封装，这样更为直观清楚。

### 几何体复制和克隆

几何体克隆或复制和网格模型在属性值深拷贝、浅拷贝方面有些不同，比如几何体的顶点属性`Geometry.vertices`，`Geometry.vertices`的属性值是一个数组对象，但是复制或克隆的时候，不是获得对象的索引值，而是深拷贝属性的值，可以在threejs源码`Geometry.js`全文检索`copy: function`关键词，找到该类对`copy`方法的封装细节。

# 光源对象

![image-20220923155945780](image-20220923155945780.png)

## 1 常见光源类型

Threejs虚拟光源是对自然界光照的模拟，threejs搭建虚拟场景的时候，为了更好的渲染场景，往往需要设置不同的光源，设置不同的光照强度，就像摄影师给你拍照要设置各种辅助灯光一样。



![img](threejs32light.png)

### 环境光[AmbientLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/AmbientLight)

环境光是没有特定方向的光源，主要是均匀整体改变Threejs物体表面的明暗效果，这一点和具有方向的光源不同，比如点光源可以让物体表面不同区域明暗程度不同。

```javascript
//环境光:环境光颜色RGB成分分别和物体材质颜色RGB成分分别相乘
var ambient = new THREE.AmbientLight(0x444444);
scene.add(ambient);//环境光对象添加到scene场景中
```

你可以把光源颜色从`0x444444`更改为`0x888888`,可以看到threejs场景中的网格模型表面变的更亮。

### 点光源[PointLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/PointLight)

点光源就像生活中的白炽灯，光线沿着发光核心向外发散，同一平面的不同位置与点光源光线入射角是不同的，点光源照射下，同一个平面不同区域是呈现出不同的明暗效果。

和环境光不同，环境光不需要设置光源位置，而点光源需要设置位置属性`.position`，光源位置不同，物体表面被照亮的面不同，远近不同因为衰减明暗程度不同。

你可以把案例源码中点光源位置从`(400, 200, 300)`位置改变到`(-400, -200, -300)`，你会发现网格模型被照亮的位置从前面变到了后面，这很正常，光源只能照亮面对着光源的面，背对着光源的无法照射到，颜色会比较暗。

```javascript
//点光源
var point = new THREE.PointLight(0xffffff);
//设置点光源位置，改变光源的位置
point.position.set(400, 200, 300);
scene.add(point);
```

### 平行光[DirectionalLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/DirectionalLight)

平行光顾名思义光线平行，对于一个平面而言，平面不同区域接收到平行光的入射角一样。

点光源因为是向四周发散，所以设置好位置属性`.position`就可以确定光线和物体表面的夹角，对于平行光而言,主要是确定光线的方向,光线方向设定好了，光线的与物体表面入射角就确定了，仅仅设置光线位置是不起作用的。

在三维空间中为了确定一条直线的方向只需要确定直线上两个点的坐标即可，所以Threejs平行光提供了位置`.position`和目标`.target`两个属性来一起确定平行光方向。目标`.target`的属性值可以是Threejs场景中任何一个三维模型对象，比如一个网格模型`Mesh`，这样Threejs计算平行光照射方向的时候，会通过自身位置属性`.position`和`.target`表示的物体的位置属性`.position`计算出来。

```javascript
// 平行光
var directionalLight = new THREE.DirectionalLight(0xffffff, 1);
// 设置光源的方向：通过光源position属性和目标指向对象的position属性计算
directionalLight.position.set(80, 100, 50);
// 方向光指向对象网格模型mesh2，可以不设置，默认的位置是0,0,0
directionalLight.target = mesh2;
scene.add(directionalLight);
```

平行光如果不设置`.position`和`.target`属性，光线默认从上往下照射，也就是可以认为`(0,1,0)`和`(0,0,0)`两个坐标确定的光线方向。

**注意一点平行光光源的位置属性`.position`并不表示平行光从这个位置向远处照射，`.position`属性只是用来确定平行光的照射方向，平行光你可以理解为太阳光，从无限远处照射过来。**

### 聚光源[SpotLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/SpotLight)

聚光源可以认为是一个沿着特定方会逐渐发散的光源，照射范围在三维空间中构成一个圆锥体。通过属性`.angle`可以设置聚光源发散角度，聚光源照射方向设置和平行光光源一样是通过位置`.position`和目标`.target`两个属性来实现。

```javascript
// 聚光光源
var spotLight = new THREE.SpotLight(0xffffff);
// 设置聚光光源位置
spotLight.position.set(200, 200, 200);
// 聚光灯光源指向网格模型mesh2
spotLight.target = mesh2;
// 设置聚光光源发散角度
spotLight.angle = Math.PI / 6
scene.add(spotLight);//光对象添加到scene场景中
```

### 光源辅助对象

Threejs提供了一些光源辅助对象，就像`AxesHelper`可视化显示三维坐标轴一样显示光源对象,通过这些辅助对象可以方便调试代码，查看位置、方向。

| 辅助对象           | 构造函数名                                                   |
| :----------------- | :----------------------------------------------------------- |
| 聚光源辅助对象     | [SpotLightHelper](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/helpers/SpotLightHelper) |
| 点光源辅助对象     | [PointLightHelper](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/helpers/PointLightHelper) |
| 平行光光源辅助对象 | [DirectionalLightHelper](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/helpers/DirectionalLightHelper) |

## 2 光照阴影实时计算

在具有方向光源的作用下，物体会形成阴影投影效果。

### 平行光投影计算代码

Three.js物体投影模拟计算主要设置三部分，一个是设置产生投影的模型对象，一个是设置接收投影效果的模型，最后一个是光源对象本身的设置，光源如何产生投影。

```javascript
var geometry = new THREE.BoxGeometry(40, 100, 40);
var material = new THREE.MeshLambertMaterial({
  color: 0x0000ff
});
var mesh = new THREE.Mesh(geometry, material);
// mesh.position.set(0,0,0)
scene.add(mesh);

// 设置产生投影的网格模型
mesh.castShadow = true;


//创建一个平面几何体作为投影面
var planeGeometry = new THREE.PlaneGeometry(300, 200);
var planeMaterial = new THREE.MeshLambertMaterial({
  color: 0x999999
});
// 平面网格模型作为投影面
var planeMesh = new THREE.Mesh(planeGeometry, planeMaterial);
scene.add(planeMesh); //网格模型添加到场景中
planeMesh.rotateX(-Math.PI / 2); //旋转网格模型
planeMesh.position.y = -50; //设置网格模型y坐标
// 设置接收阴影的投影面
planeMesh.receiveShadow = true;

// 方向光
var directionalLight = new THREE.DirectionalLight(0xffffff, 1);
// 设置光源位置
directionalLight.position.set(60, 100, 40);
scene.add(directionalLight);
// 设置用于计算阴影的光源对象
directionalLight.castShadow = true;
// 设置计算阴影的区域，最好刚好紧密包围在对象周围
// 计算阴影的区域过大：模糊  过小：看不到或显示不完整
directionalLight.shadow.camera.near = 0.5;
directionalLight.shadow.camera.far = 300;
directionalLight.shadow.camera.left = -50;
directionalLight.shadow.camera.right = 50;
directionalLight.shadow.camera.top = 200;
directionalLight.shadow.camera.bottom = -100;
// 设置mapSize属性可以使阴影更清晰，不那么模糊
// directionalLight.shadow.mapSize.set(1024,1024)
console.log(directionalLight.shadow.camera);

// 不要忘了设置渲染器，允许场景中使用阴影贴图
renderer.shadowMap.enabled = true;
```

### 聚光光源投影计算代码

下面代码是聚光光源的设置，其它部分代码和平行光一样。

```javascript
// 聚光光源
var spotLight = new THREE.SpotLight(0xffffff);
// 设置聚光光源位置
spotLight.position.set(50, 90, 50);
// 设置聚光光源发散角度
spotLight.angle = Math.PI /6
scene.add(spotLight); //光对象添加到scene场景中
// 设置用于计算阴影的光源对象
spotLight.castShadow = true;
// 设置计算阴影的区域，注意包裹对象的周围
spotLight.shadow.camera.near = 1;
spotLight.shadow.camera.far = 300;
spotLight.shadow.camera.fov = 20;
```

### 模型`.castShadow`属性

```
.castShadow`属性值是布尔值，默认false，用来设置一个模型对象是否在光照下产生投影效果。具体查看threejs文档`Object3D
// 设置产生投影的网格模型
mesh.castShadow = true;
```

### `.receiveShadow`属性

```
.receiveShadow`属性值是布尔值，默认false，用来设置一个模型对象是否在光照下接受其它模型的投影效果。具体查看threejs文档`Object3D
// 设置网格模型planeMesh接收其它模型的阴影(planeMesh作为投影面使用)
planeMesh.receiveShadow = true;
```

### 光源`.castShadow`属性

如果属性设置为 true， 光源将投射动态阴影. *警告*: 这需要很多计算资源，需要调整以使阴影看起来正确. 更多细节，查看DirectionalLightShadow. 默认值false.

```javascript
// 设置用于计算阴影的光源对象
directionalLight.castShadow = true;
// spotLight.castShadow = true;
```

### 光源`.shadow`属性

平行光`DirectionalLight`的`.shadow`属性值是平行光阴影对象`DirectionalLightShadow`，聚光源`SpotLight`的`.shadow`属性值是聚光源阴影对象`SpotLightShadow`。关于`DirectionalLightShadow`和`SpotLightShadow`两个类的具体介绍可以参考Three.js文档Lights / Shadows分类，

### 阴影对象基类`LightShadow`

平行光阴影对象`DirectionalLightShadow`和聚光源阴影对象`SpotLightShadow`两个类的基类是`LightShadow`

#### `LightShadow`属性`.camera`

观察光源的相机对象. 从光的角度来看，以相机对象的观察位置和方向来判断，其他物体背后的物体将处于阴影中。

```javascript
// 聚光源设置
spotLight.shadow.camera.near = 1;
spotLight.shadow.camera.far = 300;
spotLight.shadow.camera.fov = 20;
```

#### `LightShadow`属性`.mapSize`

定义阴影纹理贴图宽高尺寸的一个二维向量Vector2.

较高的值会以计算时间为代价提供更好的阴影质量. 宽高分量值必须是2的幂, 直到给定设备的`WebGLRenderer.capabilities.maxTextureSize`, 尽管宽度和高度不必相同 (例如，(512, 1024)是有效的). 默认值为 ( 512, 512 ).

```javascript
directionalLight.shadow.mapSize.set(1024,1024)
```

#### `LightShadow`属性`.map`

该属性的值是WebGL渲染目标对象`WebGLRenderTarget`，使用内置摄像头生成的深度图; 超出像素深度的位置在阴影中。 在渲染期间内部计算。

## 3 基类Light和Object3D

前面课程对光源的介绍主要是从认识光源的角度介绍的，本节课主要从编写代码角度介绍，根据类的继承关系设置光源对象的相关属性。

你查看文档[SpotLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/SpotLight)、[DirectionalLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/DirectionalLight)、环境光[AmbientLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/AmbientLight)等光源对象都有一个共同的基类[Light](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/Light)，光源[Light](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/Light)也有一个基类[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)。也就是说**Threejs环境光、点光源等子类光源可以继承`Light`和`Object3D`两个父类的属性和方法。**

![image-20220923160301249](image-20220923160301249.png)

### 光源位置属性

光源对象继承父类[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)的位置属性`.position`。

以点光源[PointLight](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/PointLight) 为例，`PointLight`的基类是[Light](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/Light)，`Light`的基类是[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)，点光源自然继承对象`Object3D`的位置属性`.position`。

```javascript
var point = new THREE.PointLight(0xffffff);//点光源
//设置点光源位置  
//
//光源对象和模型对象的position属性一样是Vector3对象
point.position.set(400, 200, 300);
```

光源对象的`.add()`方法同样继承自基类[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)

```javascript
//环境光对象添加到scene场景中
scene.add(ambient);
//点光源对象添加到scene场景中
scene.add(point);
```

### 光源颜色属性`.color`和强度属性`.intensity`

你查看光源光源[Light](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/Light)文档，可以看到该类定义了光源颜色属性`.color`和强度系数属性`.intensity`。

光源颜色属性`.color`默认值是白色`0xffffff`,强度属性`.intensity`默认1.0,光照计算的时候会把两个属性值相乘。

比如环境光颜色设置为`0xffffff`,强度系数设置为0.5，把0.5设置为0.8，threejs场景中模型会变得更明亮。调节环境颜色你可以直接设置不同颜色值，比如`0x44444`、`0xddddddd`，也可以使用更为方便的强度系数去调节。对于点光源、聚光源和环境光一样继承基类[Light](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/lights/Light)强度系数属性`.intensity`。

```javascript
//环境光：颜色设置为`0xffffff`,强度系数设置为0.5
var ambient = new THREE.AmbientLight(0xffffff,0.5);
scene.add(ambient);
```

# 层级模型、树结构

​	比如一辆车，在Threejs中你可以使用一个网格模型去描述车上面的一个零件，多个零件就需要多个网格模型表示，这些网格模型之间就会构成父子或兄弟关系，从而形成一个层级结构。在机械、建筑相关的Web3D应用中，通常会用到层级模型的知识，一个层级模型就是一本书的目录一样。

​	本章主要目的是帮助你建立Threejs层级模型的概念，通过Threejs的组对象[Group](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/objects/Group)可以组织各个模型，构成一个层级结构。学习本节课你也可以参考前端中DOM树去理解，Threejs一个一个的模型对象就像HTML元素一样可以组成一个树结构，可以通过特定id或name属性选择某个或某些模型对象。

​	在具体开发过程中，3D美术给你一个包含多个网格模型对象的层级模型，你可能需要操作某个网格模型，这时候3D美术只要通过对模型命名标记模型，那么对于程序员来说，直接调用Threejs的某个方法就可以遍历整个模型，找到某个你想要操作的模型对象。

![image-20220923195552622](D:\Typora\pictures\threejs\image-20220923195552622.png)

## 1 组对象[Group](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/objects/Group)、层级模型

​	本节课的目的是为了大家建立层级模型的概念，所谓层级模型，比如一个机器人，人头、胳膊都是人的一部分，眼睛是头的一部分，手是个胳膊的一部分，手指是手的一部分...这样的话就构成一个一个层级结构或者说树结构。

### Group案例

​	在详细讲解层级模型之前先通过Threejs的类[Group](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/objects/Group)实现一个网格模型简单的案例。

​	下面代码创建了两个网格模型mesh1、mesh2，通过`THREE.Group`类创建一个组对象group,然后通过`add`方法把网格模型mesh1、mesh2作为设置为组对象group的子对象，然后在通过执行`scene.add(group)`把组对象group作为场景对象的scene的子对象。也就是说场景对象是scene是group的父对象，group是mesh1、mesh2的父对象。这样就构成了一个三层的层级结构，当然了你也可以通过`Group`自己创建新模型节点作为层级结构中的一层。

```javascript
//创建两个网格模型mesh1、mesh2
var geometry = new THREE.BoxGeometry(20, 20, 20);
var material = new THREE.MeshLambertMaterial({color: 0x0000ff});
var group = new THREE.Group();
var mesh1 = new THREE.Mesh(geometry, material);
var mesh2 = new THREE.Mesh(geometry, material);
mesh2.translateX(25);
//把mesh1型插入到组group中，mesh1作为group的子对象
group.add(mesh1);
//把mesh2型插入到组group中，mesh2作为group的子对象
group.add(mesh2);
//把group插入到场景中作为场景子对象
scene.add(group);
```

​	网格模型mesh1、mesh2作为设置为父对象group的子对象，如果父对象group进行旋转、缩放、平移变换，子对象同样跟着变换，就像你的头旋转了，眼睛会跟着头旋转。

```javascript
//沿着Y轴平移mesh1和mesh2的父对象，mesh1和mesh2跟着平移
group.translateY(100);
//父对象缩放，子对象跟着缩放
group.scale.set(4,4,4);
//父对象旋转，子对象跟着旋转
group.rotateY(Math.PI/6)
```

### 查看子对象`.children`

​	Threejs场景对象Scene、组对象Group都有一个子对象属性`.children`,通过该属性可以访问父对象的子对象，**子对象属性`.children`的值是数组，所有子对象是数组的值**，你可以在浏览器控制台打印测试上面案例代码。

​	执行`console.log(group.children)`你可以在浏览器控制控制看到group的子对象是案例代码中通过`add`方法添加的两个网格模型对象Mesh。

```javascript
console.log('查看group的子对象',group.children);
```

### 场景对象结构

​	执行`console.log(scene.children)`你在浏览器控制台查看场景对象Scene的子对象，除了可以看到案例代码通过`add`方法添加的组对象group之外，还可以看到通过`add`方法插入到场景中的环境光`AmbientLight`、点光源`PointLight`、辅助坐标对象`AxesHelper`等子对象。

```javascript
console.log('查看Scene的子对象',scene.children);
```

​	场景对象对象scene构成的层级模型本身是一个树结构，场景对象层级模型的第一层，也就是树结构的根节点，一般来说网格模型Mesh、点模型Points、线模型Line是树结构的最外层叶子结点。构建层级模型的中间层一般都是通过Threejs的`Group`类来完成，`Group`类实例化的对象可以称为组对象。

![image-20220923195746434](D:\Typora\pictures\threejs\image-20220923195746434.png)

​	Threejs渲染的时候从根节点场景对象开始解析渲染，如果一个模型要想被渲染出来就要直接或间接插入到场景scene中，一个光源如果要在光照计算中起作用同样需要通过`add`方法插入到场景中。

### `.add()`方法

​	**场景对象`Scene`、组对象`Group`、网格模型对象`Mesh`、光源对象`Light`的`.add()`方法都是继承自它们共同的基类[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)。**

​	**父对象执行`.add()`方法的本质就是把参数中的子对象添加到自身的子对象属性`.children`中。**

`.add()`方法可以单独插入一个对象，也**可以同时插入多个子对象**。

```javascript
group.add(mesh1);
group.add(mesh2);
group.add(mesh1,mesh2);
```

Scene根节点 渲染的问题

### `.remove()`方法

`.add()`方法是给父对象添加一个子对象，**`.remove()`方法是删除父对象中的一个子对象。** 一个对象的全部子对象可以通过该对象的`.children()`属性访问获得，执行该对象的删除方法`.remove()`和添加方法`.add()`一样改变的都是父对象的`.children()`属性。

场景`Scene`或组对象`Group`的`.remove()`方法使用规则可以查看它们的基类[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)。

```javascript
// 删除父对象group的子对象网格模型mesh1
group.remove(mesh1)
// 一次删除场景中多个对象
scene.remove(light,group)
```

## 2 层级模型节点命名、查找、遍历

上节课说过Threejs场景对象Scene和各种子对象构成的层级模型就是一个树结构。如果你有一定的算法基础对树结构肯定会非常了解，如果你了解前端的DOM树结构也非常有助于本节课的学习，如果这些都不了解也没有关系，直接体验本节课的案例源码。

本文通过Three.js的一个类`Group`来介绍Threejs层级模型的概念，如果你对WebGL层级模型已经有一定的概念，直接把重点放在`Group`的了解上，如果没有层级模型的概念，就借着对Three.js API`Group`的介绍了解下该概念。

### 模型命名(`.name`属性)

在层级模型中可以给一些模型对象通过`.name`属性命名进行标记。

```javascript
group.add(Mesh)
// 网格模型命名
Mesh.name = "眼睛"
// mesh父对象对象命名
group.name = "头"
```

### 树结构层级模型

​	实际开发的时候，可能会加载外部的模型，然后从模型对象通过节点的名称`.name`查找某个子对象，为了大家更容易理解，本节课不加载外部模型，直接通过代码创建一个非常简易的机器人模型，然后在机器人基础上进行相关操作。

```javascript
// 头部网格模型和组
var headMesh = sphereMesh(10, 0, 0, 0);
headMesh.name = "脑壳"
var leftEyeMesh = sphereMesh(1, 8, 5, 4);
leftEyeMesh.name = "左眼"
var rightEyeMesh = sphereMesh(1, 8, 5, -4);
rightEyeMesh.name = "右眼"
var headGroup = new THREE.Group();
headGroup.name = "头部"
headGroup.add(headMesh, leftEyeMesh, rightEyeMesh);
// 身体网格模型和组
var neckMesh = cylinderMesh(3, 10, 0, -15, 0);
neckMesh.name = "脖子"
var bodyMesh = cylinderMesh(14, 30, 0, -35, 0);
bodyMesh.name = "腹部"
var leftLegMesh = cylinderMesh(4, 60, 0, -80, -7);
leftLegMesh.name = "左腿"
var rightLegMesh = cylinderMesh(4, 60, 0, -80, 7);
rightLegMesh.name = "右腿"
var legGroup = new THREE.Group();
legGroup.name = "腿"
legGroup.add(leftLegMesh, rightLegMesh);
var bodyGroup = new THREE.Group();
bodyGroup.name = "身体"
bodyGroup.add(neckMesh, bodyMesh, legGroup);
// 人Group
var personGroup = new THREE.Group();
personGroup.name = "人"
personGroup.add(headGroup, bodyGroup)
personGroup.translateY(50)
scene.add(personGroup);

// 球体网格模型创建函数
function sphereMesh(R, x, y, z) {
  var geometry = new THREE.SphereGeometry(R, 25, 25); //球体几何体
  var material = new THREE.MeshPhongMaterial({
    color: 0x0000ff
  }); //材质对象Material
  var mesh = new THREE.Mesh(geometry, material); // 创建网格模型对象
  mesh.position.set(x, y, z);
  return mesh;
}
// 圆柱体网格模型创建函数
function cylinderMesh(R, h, x, y, z) {
  var geometry = new THREE.CylinderGeometry(R, R, h, 25, 25); //球体几何体
  var material = new THREE.MeshPhongMaterial({
    color: 0x0000ff
  }); //材质对象Material
  var mesh = new THREE.Mesh(geometry, material); // 创建网格模型对象
  mesh.position.set(x, y, z);
  return mesh;
}
```

### 递归遍历方法`.traverse()`

​	Threejs层级模型就是一个树结构，可以通过递归遍历的算法去遍历Threejs一个模型对象的所有后代，可以通过下面代码递归遍历上面创建一个机器人模型或者一个外部加载的三维模型。

```javascript
scene.traverse(function(obj) {
  if (obj.type === "Group") {
    console.log(obj.name);
  }
  if (obj.type === "Mesh") {
    console.log('  ' + obj.name);
    obj.material.color.set(0xffff00);
  }
  if (obj.name === "左眼" | obj.name === "右眼") {
    obj.material.color.set(0x000000)
  }
  // 打印id属性
  console.log(obj.id);
  // 打印该对象的父对象
  console.log(obj.parent);
  // 打印该对象的子对象
  console.log(obj.children);
})
```

### 查找某个具体的模型

​	看到Threejs的`.getObjectById()`、`.getObjectByName()`等方法，如果已有前端基础，很容易联想到DOM的一些方法。

​	Threejs和前端DOM一样，可以通过一个方法查找树结构父元素的某个后代对象，对于普通前端而言可以通过name或id等方式查找一个或多个DOM元素，Threejs同样可以通过一些方法查找一个模型树中的某个节点。更多的查找方法和方法的使用细节可以查看基类[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)

```javascript
// 遍历查找scene中复合条件的子对象，并返回id对应的对象
var idNode = scene.getObjectById ( 4 );
console.log(idNode);
// 遍历查找对象的子对象，返回name对应的对象（name是可以重名的，返回第一个）
var nameNode = scene.getObjectByName ( "左腿" );
nameNode.material.color.set(0xff0000);
```

## 3 本地位置坐标、世界位置坐标

### Three.js获得世界坐标`.getWorldPosition()`

​	通过前两节课的学习，想必你已经对Threejs的层级模型有了一定认识，那么本节课就在层级模型概念的基础上，继续给家讲解两个新的概念，即本地坐标系和世界坐标系。

​	如果你对本地坐标系和世界坐标系已经有了一定概念，那么可以直接访问**模型的位置属性`.position`获得模型在本地坐标系或者说模型坐标系下的三维坐标，通过模型的`.getWorldPosition()`方法获得该模型在世界坐标下的三维坐标。**

### `.getWorldPosition()`方法

​	模型对象的方法`.getWorldPosition()`方法和位置属性`.position`一样继承自基类[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)。

```javascript
// 声明一个三维向量用来保存世界坐标
var worldPosition = new THREE.Vector3();
// 执行getWorldPosition方法把模型的世界坐标保存到参数worldPosition中
mesh.getWorldPosition(worldPosition);
```

### 建立世界坐标系概念

​	如果你没有本地坐标系和世界坐标系的概念，可以通过下面的案例源码很快的建立两个坐标系的概念。

​	你首先在案例中测试下面源码，通过位置属性`.position`和`.getWorldPosition()`分别返回模型的本地位置坐标和世界坐标，查看两个坐标x分量有什么不同。你可以看到网格模型mesh通过位置属性`.position`返回的坐标x分量是50，通过`.getWorldPosition()`返回的坐标x分量是100，也就是说**mesh的世界坐标是mesh位置属性`.position`和mesh父对象group位置属性`.position`的累加。**

```javascript
var mesh = new THREE.Mesh(geometry, material);
// mesh的本地坐标设置为(50, 0, 0)
mesh.position.set(50, 0, 0);
var group = new THREE.Group();
// group本地坐标设置和mesh一样设置为(50, 0, 0)
// mesh父对象设置position会影响得到mesh的世界坐标
group.position.set(50, 0, 0);
group.add(mesh);
scene.add(group);

// .position属性获得本地坐标
console.log('本地坐标',mesh.position);

// getWorldPosition()方法获得世界坐标
//该语句默认在threejs渲染的过程中执行,如果渲染之前想获得世界矩阵属性、世界位置属性等属性，需要通过代码更新
scene.updateMatrixWorld(true);
var worldPosition = new THREE.Vector3();
mesh.getWorldPosition(worldPosition);
console.log('世界坐标',worldPosition);
```

### 总结

下面对上面的案例实验进行总结。

​	**所谓本地坐标系或者说模型坐标系，就是模型对象相对模型的父对象而言，模型位置属性`.position`表示的坐标值就是以本地坐标系为参考，表示子对象相对本地坐标系原点(0,0,0)的偏移量。**

​	前面两节课说过Threejs场景Scene是一个树结构，一个模型对象可能有多个父对象节点。世界坐标系默认就是对Threejs整个场景Scene建立一个坐标系，**一个模型相对世界坐标系的坐标值就是该模型对象所有父对象以及模型本身位置属性`.position`的叠加。**

### 本地缩放系数`.scale`

​	通过前面的论述，模型的位置属性`.position`可以称为本地坐标或者说局部坐标，对于属性`.scale`一样，可以称为模型的本地缩放系数或者局部的缩放系数，通过`.getWorldScale()`方法可以获得一个模型的世界缩放系数，就像执行`.getWorldPosition()`方法一样获得世界坐标，关于`.getWorldScale()`方法可以查看基类[Object3D](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Object3D)。

# 几何体对象、曲线、三维模型（未学习）

## 1 常见几何体和曲线API介绍

​	本节课主要内容是对Threejs几何体和曲线相关的API进行一个整体的介绍，更具体的介绍可以查看本章后面几节课程。

### 几何体

​	关于Threejs常见的几何体类下面通过一个脑图进行了简单的分类，并附上了threejs文档连接。

​	几何体本质上就是threejs生成顶点的算法，如果有兴趣你可以打开threejs几何体部分的源码查看threejs具体如何通过程序生成顶点位置、法线方向等顶点数据。

​	所有几何体的基类分为[Geometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Geometry)和[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)两大类，两类几何体直接可以相互转化。

![image-20220923204800225](D:\Typora\pictures\threejs\image-20220923204800225.png)

### 曲线

​	曲线和几何体同样本质上都是用来生成顶点的算法，曲线主要是按照一定的规则生成一系列沿着某条轨迹线分布的顶点。当你把曲线、几何体看成顶点的时候，查考文档很多属性和方法自然很同意理解。

![image-20220923204811169](D:\Typora\pictures\threejs\image-20220923204811169.png)

## 2 直线、椭圆、圆弧、基类Curve

本节课通过介绍直线、圆弧线，以及这些曲线的基类`Curve`。

### 圆弧线[ArcCurve](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/curves/ArcCurve)

圆弧线[ArcCurve](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/curves/ArcCurve)的基类是椭圆弧线[EllipseCurve](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/curves/EllipseCurve),关于圆弧线的使用方法可以查看threejs文档中的椭圆弧线。

```javascript
ArcCurve( aX, aY, aRadius, aStartAngle, aEndAngle, aClockwise )
```

| 参数                   | 含义                          |
| :--------------------- | :---------------------------- |
| aX, aY                 | 圆弧圆心坐标                  |
| aRadius                | 圆弧半径                      |
| aStartAngle, aEndAngle | 起始角度                      |
| aClockwise             | 是否顺时针绘制，默认值为false |

```javascript
//参数：0, 0圆弧坐标原点x，y  100：圆弧半径    0, 2 * Math.PI：圆弧起始角度
var arc = new THREE.ArcCurve(0, 0, 100, 0, 2 * Math.PI);
```

### 曲线[Curve](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/core/Curve)方法`.getPoints()`

`.getPoints()`是基类[Curve](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/core/Curve)的方法，圆弧线[ArcCurve](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/curves/ArcCurve)的基类是椭圆弧线[EllipseCurve](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/curves/EllipseCurve),椭圆弧线的基类是曲线`Curve`，所以圆弧线具有`Curve`的方法`.getPoints()`。

通过方法`.getPoints()`可以从圆弧线按照一定的细分精度返回沿着圆弧线分布的顶点坐标。细分数越高返回的顶点数量越多，自然轮廓越接近于圆形。方法`.getPoints()`的返回值是一个由二维向量[Vector2](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/math/Vector2)或三维向量[Vector3](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/math/Vector3)构成的数组，`Vector2`表示位于同一平面内的点，`Vector3`表示三维空间中一点。

```javascript
var arc = new THREE.ArcCurve(0, 0, 100, 0, 2 * Math.PI);
//getPoints是基类Curve的方法，返回一个vector2对象作为元素组成的数组
var points = arc.getPoints(50);//分段数50，返回51个顶点
```

### 几何体方法`.setFromPoints()`

`.setFromPoints()`是几何体[Geometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Geometry)的方法，通过该方法可以把数组`points`中顶点数据提取出来赋值给几何体的顶点位置属性`geometry.vertices`，数组`points`的元素是二维向量[Vector2](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/math/Vector2)或三维向量[Vector3](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/math/Vector3)。

[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)和`Geometry`一样具有方法`.setFromPoints()`，不过区别是提取顶点数据后赋值给`geometry.attributes.position`属性。

```javascript
// setFromPoints方法从points中提取数据改变几何体的顶点属性vertices
geometry.setFromPoints(points);
console.log(geometry.vertices);
// 如果几何体是BufferGeometry，setFromPoints方法改变的是.attributes.position属性
// console.log(geometry.attributes.position);
```

### 绘制圆弧线案例

![img](D:\Typora\pictures\threejs\threejs411.jpg)

使用threejs的API圆弧线[ArcCurve](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/curves/ArcCurve)绘制一个圆弧轮廓。

```javascript
var geometry = new THREE.Geometry(); //声明一个几何体对象Geometry
//参数：0, 0圆弧坐标原点x，y  100：圆弧半径    0, 2 * Math.PI：圆弧起始角度
var arc = new THREE.ArcCurve(0, 0, 100, 0, 2 * Math.PI);
//getPoints是基类Curve的方法，返回一个vector2对象作为元素组成的数组
var points = arc.getPoints(50);//分段数50，返回51个顶点
// setFromPoints方法从points中提取数据改变几何体的顶点属性vertices
geometry.setFromPoints(points);
//材质对象
var material = new THREE.LineBasicMaterial({
  color: 0x000000
});
//线条模型对象
var line = new THREE.Line(geometry, material);
scene.add(line); //线条对象添加到场景中
```

和上面绘制圆弧线代码实现的功能相同，不过没有借助圆弧线[THREE.ArcCurve](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/curves/ArcCurve)，通过三角函数计算生成圆弧线上的顶点。设置这个案例的目的就是，你可以通过对比两个代码案例，明白Threejs一些曲线API本质上就是通过某种算法得到了沿着特定轨迹的顶点数据。

```javascript
var geometry = new THREE.Geometry(); //声明一个几何体对象Geometry
var R = 100; //圆弧半径
var N = 50; //分段数量
// 批量生成圆弧上的顶点数据
for (var i = 0; i < N; i++) {
  var angle = 2 * Math.PI / N * i;
  var x = R * Math.sin(angle);
  var y = R * Math.cos(angle);
  geometry.vertices.push(new THREE.Vector3(x, y, 0));
}
// 插入最后一个点，line渲染模式下，产生闭合效果
// geometry.vertices.push(geometry.vertices[0])
//材质对象
var material = new THREE.LineBasicMaterial({
  color: 0x000000
});
//线条模型对象
var line = new THREE.Line(geometry, material);
scene.add(line); //线条对象添加到场景中
```

### 绘制直线效果

直接给几何体`Geometry`设置两个顶点数据。

```javascript
var geometry = new THREE.Geometry(); //声明一个几何体对象Geometry
var p1 = new THREE.Vector3(50, 0, 0); //顶点1坐标
var p2 = new THREE.Vector3(0, 70, 0); //顶点2坐标
//顶点坐标添加到geometry对象
geometry.vertices.push(p1, p2);
var material = new THREE.LineBasicMaterial({
  color: 0xffff00,
});//材质对象
//线条模型对象
var line = new THREE.Line(geometry, material);
scene.add(line); //线条对象添加到场景中
```

通过[LineCurve3](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/curves/LineCurve3)绘制一条三维直线。

```javascript
var geometry = new THREE.Geometry(); //声明一个几何体对象Geometry
var p1 = new THREE.Vector3(50, 0, 0); //顶点1坐标
var p2 = new THREE.Vector3(0, 70, 0); //顶点2坐标
// 三维直线LineCurve3
var LineCurve = new THREE.LineCurve3(p1, p2);
// 二维直线LineCurve
var LineCurve = new THREE.LineCurve(new THREE.Vector2(50, 0), new THREE.Vector2(0, 70));
var pointArr = LineCurve.getPoints(10);
geometry.setFromPoints(pointArr);
```

通过[LineCurve](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/curves/LineCurve)绘制一条二维直线。

```javascript
var geometry = new THREE.Geometry(); //声明一个几何体对象Geometry
var p1 = new THREE.Vector2(50, 0); //顶点1坐标
var p2 = new THREE.Vector2(0, 70); //顶点2坐标
// 二维直线LineCurve
var LineCurve = new THREE.LineCurve(p1, p2);
var pointArr = LineCurve.getPoints(10);
geometry.setFromPoints(pointArr);
```

## 3 样条曲线、贝赛尔曲线

​	规则的曲线比如圆、椭圆、抛物线都可以用一个函数去描述，对于不规则的曲线无法使用一个特定的函数去描述，这也就是样条曲线和贝塞尔曲线出现的原因。Threejs提供了这两种曲线的API，不需要自己封装，如果你想深入研究可以学习计算机图形学。

![image-20220923205122814](D:\Typora\pictures\threejs\image-20220923205122814.png)

### 一条光滑样条曲线案例

![img](D:\Typora\pictures\threejs\threejs421.jpg)

​	在三维空间中设置5个顶点，输入三维样条曲线[CatmullRomCurve3](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/curves/CatmullRomCurve3)作为参数，然后返回更多个顶点，通过返回的顶点数据，构建一个几何体，通过`Line`可以绘制出来一条沿着5个顶点的光滑样条曲线。

```javascript
var geometry = new THREE.Geometry(); //声明一个几何体对象Geometry
// 三维样条曲线  Catmull-Rom算法
var curve = new THREE.CatmullRomCurve3([
  new THREE.Vector3(-50, 20, 90),
  new THREE.Vector3(-10, 40, 40),
  new THREE.Vector3(0, 0, 0),
  new THREE.Vector3(60, -60, 0),
  new THREE.Vector3(70, 0, 80)
]);
//getPoints是基类Curve的方法，返回一个vector3对象作为元素组成的数组
var points = curve.getPoints(100); //分段数100，返回101个顶点
// setFromPoints方法从points中提取数据改变几何体的顶点属性vertices
geometry.setFromPoints(points);
//材质对象
var material = new THREE.LineBasicMaterial({
  color: 0x000000
});
//线条模型对象
var line = new THREE.Line(geometry, material);
scene.add(line); //线条对象添加到场景中
```

通过调用threejs样条曲线或贝塞尔曲线的API，你可以输入有限个顶点返回更多顶点，然后绘制一条光滑的轮廓曲线。

### 贝塞尔曲线

贝塞尔曲线和样条曲线不同，多了一个控制点概念。

二次贝赛尔曲线的参数p1、p3是起始点，p2是控制点，控制点不在贝塞尔曲线上。 ![img](D:\Typora\pictures\threejs\threejs422.jpg)

```javascript
var p1 = new THREE.Vector3(-80, 0, 0);
var p2 = new THREE.Vector3(20, 100, 0);
var p3 = new THREE.Vector3(80, 0, 0);
// 三维二次贝赛尔曲线
var curve = new THREE.QuadraticBezierCurve3(p1, p2, p3);
```

二次贝赛尔曲线的参数p1、p4是起始点，p2、p3是控制点，控制点不在贝塞尔曲线上。 ![img](D:\Typora\pictures\threejs\threejs423.jpg)

```javascript
var p1 = new THREE.Vector3(-80, 0, 0);
var p2 = new THREE.Vector3(-40, 100, 0);
var p3 = new THREE.Vector3(40, 100, 0);
var p4 = new THREE.Vector3(80, 0, 0);
// 三维三次贝赛尔曲线
var curve = new THREE.CubicBezierCurve3(p1, p2, p3, p4);
```

## 4 多个线条组合曲线[CurvePath](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/core/CurvePath)

通过组合曲线[CurvePath](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/core/CurvePath)可以把多个圆弧线、样条曲线、直线等多个曲线合并成一个曲线。

![image-20220923205214339](D:\Typora\pictures\threejs\image-20220923205214339.png)

### U型案例

![img](D:\Typora\pictures\threejs\threejs431.jpg)

```javascript
var geometry = new THREE.Geometry(); //声明一个几何体对象Geometry
// 绘制一个U型轮廓
var R = 80;//圆弧半径
var arc = new THREE.ArcCurve(0, 0, R, 0, Math.PI, true);
// 半圆弧的一个端点作为直线的一个端点
var line1 = new THREE.LineCurve(new THREE.Vector2(R, 200, 0), new THREE.Vector2(R, 0, 0));
var line2 = new THREE.LineCurve(new THREE.Vector2(-R, 0, 0), new THREE.Vector2(-R, 200, 0));
// 创建组合曲线对象CurvePath
var CurvePath = new THREE.CurvePath();
// 把多个线条插入到CurvePath中
CurvePath.curves.push(line1, arc, line2);
//分段数200
var points = CurvePath.getPoints(200);
// setFromPoints方法从points中提取数据改变几何体的顶点属性vertices
geometry.setFromPoints(points);
//材质对象
var material = new THREE.LineBasicMaterial({
  color: 0x000000
});
//线条模型对象
var line = new THREE.Line(geometry, material);
scene.add(line); //线条对象添加到场景中
```

## 5 曲线路径管道成型[TubeGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/geometries/TubeGeometry)

[TubeGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/geometries/TubeGeometry)的功能就是通过一条曲线生成一个圆管。它的本质就是以曲线上顶点为基准，生成一系列曲线等径分布的顶点数据， 具体算法如何实现的可以查看three.js引擎源码。

```javascript
构造函数格式：TubeGeometry(path, tubularSegments, radius, radiusSegments, closed)
```

| 参数            | 值                                    |
| :-------------- | :------------------------------------ |
| path            | 扫描路径，基本类是Curve的路径构造函数 |
| tubularSegments | 路径方向细分数，默认64                |
| radius          | 管道半径，默认1                       |
| radiusSegments  | 管道圆弧细分数，默认8                 |
| closed          | Boolean值，管道是否闭合               |

### 样条曲面生成圆管案例

```javascript
//创建管道成型的路径(3D样条曲线)
var path = new THREE.CatmullRomCurve3([
  new THREE.Vector3(-10, -50, -50),
  new THREE.Vector3(10, 0, 0),
  new THREE.Vector3(8, 50, 50),
  new THREE.Vector3(-5, 0, 100)
]);
// path:路径   40：沿着轨迹细分数  2：管道半径   25：管道截面圆细分数
var geometry = new THREE.TubeGeometry(path, 40, 2, 25);
```

你也可以使用下面直线替换上面的样条曲线查看圆管生成效果。

```javascript
// LineCurve3创建直线段路径
var path = new THREE.LineCurve3(new THREE.Vector3(0, 100, 0), new THREE.Vector3(0, 0, 0));
```

### CurvePath多段路径生成管道案例

通过下面代码创建了一段样条曲线和两条直线拼接成的路径，然后通过曲线路径[CurvePath](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/core/CurvePath)把样条曲线和料条曲线合并成为一条路径。

```javascript
// 创建多段线条的顶点数据
var p1 = new THREE.Vector3(-85.35, -35.36)
var p2 = new THREE.Vector3(-50, 0, 0);
var p3 = new THREE.Vector3(0, 50, 0);
var p4 = new THREE.Vector3(50, 0, 0);
var p5 = new THREE.Vector3(85.35, -35.36);
// 创建线条一：直线
let line1 = new THREE.LineCurve3(p1,p2);
// 重建线条2：三维样条曲线
var curve = new THREE.CatmullRomCurve3([p2, p3, p4]);
// 创建线条3：直线
let line2 = new THREE.LineCurve3(p4,p5);
var CurvePath = new THREE.CurvePath();// 创建CurvePath对象
CurvePath.curves.push(line1, curve, line2);// 插入多段线条
//通过多段曲线路径创建生成管道
//通过多段曲线路径创建生成管道，CCurvePath：管道路径
var geometry2 = new THREE.TubeGeometry(CurvePath, 100, 5, 25, false);
```

## 6 旋转造型[LatheGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/geometries/LatheGeometry)

![img](D:\Typora\pictures\threejs\threejs45lathe.png)

生活中有很多的几何体具备旋转特征，比如球体，常见杯子, three.js提供了一个构造函数`LatheGeometry()`， `LatheGeometry`可以利用已有的二维数据生成三维顶点数据，二维数据可以通过二维向量对象`Vector2`定义，也可以通过3D曲线或2D线条轮廓生成。 `LatheGeometry`的二维坐标数据默认绕y轴旋转。

```javascript
格式：LatheGeometry(points, segments, phiStart, phiLength)
```

| 参数      | 值                              |
| :-------- | :------------------------------ |
| points    | Vector2表示的坐标数据组成的数组 |
| segments  | 圆周方向细分数，默认12          |
| phiStart  | 开始角度,默认0                  |
| phiLength | 旋转角度，默认2π                |

```javascript
/**
 * 创建旋转网格模型
 */
var points = [
    new THREE.Vector2(50,60),
    new THREE.Vector2(25,0),
    new THREE.Vector2(50,-60)
];
var geometry = new THREE.LatheGeometry(points,30);
var material=new THREE.MeshPhongMaterial({
    color:0x0000ff,//三角面颜色
    side:THREE.DoubleSide//两面可见
});//材质对象
material.wireframe = true;//线条模式渲染(查看细分数)
var mesh=new THREE.Mesh(geometry,material);//旋转网格模型对象
scene.add(mesh);//旋转网格模型添加到场景中
```

### 样条曲线插值计算

借助[Shape](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/core/Shape)对象的方法`.splineThru()`，把上面的三个顶点进行样条插值计算， 可以得到一个光滑的旋转曲面。

```javascript
var shape = new THREE.Shape();//创建Shape对象
var points = [//定位定点
    new THREE.Vector2(50,60),
    new THREE.Vector2(25,0),
    new THREE.Vector2(50,-60)
];
shape.splineThru(points);//顶点带入样条插值计算函数
var splinePoints = shape.getPoints(20);//插值计算细分数20
var geometry = new THREE.LatheGeometry(splinePoints,30);//旋转造型
```

`shape.getPoints(20)`的作用是利用已有的顶点插值计算出新的顶点，两个顶点之间插值计算出20个顶点，如果细分数是1不是20，相当于不进行插值计算， 插值计算的规则通过[Shape](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/core/Shape)对象的方法`.splineThru()`定义，几何曲线的角度描述，`splineThru`的作用就是创建一个样条曲线，除了样条曲线还可以使用贝赛尔等曲线进行插值计算。

## [7 Shape](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/core/Shape)对象和轮廓填充[ShapeGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/geometries/ShapeGeometry)

![image-20220923205333088](D:\Typora\pictures\threejs\image-20220923205333088.png)

### 填充顶点构成的轮廓

通过下面代码定义了6个顶点坐标，也可以说是5个，最后一个和第一个是重合的，构成一个五边形区域。然后使用这一组二维顶点坐标作为`Shape`的参数构成一个五边形轮廓。把五边形轮廓`Shape`作为`ShapeGeometry`的参数，可以根据轮廓坐标计算出一系列三角形面填充轮廓，形成一个平面几何体。 ![img](D:\Typora\pictures\threejs\threejs460.jpg)

```javascript
var points = [
  new THREE.Vector2(-50, -50),
  new THREE.Vector2(-60, 0),
  new THREE.Vector2(0, 50),
  new THREE.Vector2(60, 0),
  new THREE.Vector2(50, -50),
  new THREE.Vector2(-50, -50),
]
// 通过顶点定义轮廓
var shape = new THREE.Shape(points);
// shape可以理解为一个需要填充轮廓
// 所谓填充：ShapeGeometry算法利用顶点计算出三角面face3数据填充轮廓
var geometry = new THREE.ShapeGeometry(shape, 25);
```

调用`Shape`圆弧方法`.absarc()`绘制一个圆形轮廓，然后通过`ShapeGeometry`可以把该圆形轮廓填充为一个圆形平面几何体。

你可以尝试更改`ShapeGeometry`的参数2，参数2表示细分数，然后网格材质设置为`wireframe: true`查看圆形区域填充三角形的数量变化。

```javascript
// 通过shpae基类path的方法绘制轮廓（本质也是生成顶点）
var shape = new THREE.Shape();
shape.absarc(0,0,100,0,2*Math.PI);//圆弧轮廓
console.log(shape.getPoints(15));//查看shape顶点数据
var geometry = new THREE.ShapeGeometry(shape, 25);
```

下面代码是通过`shpae`绘制了一个矩形区域，更多相关的轮廓绘制方法可以查看[Shape](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/core/Shape)文档。

```javascript
// 通过shpae基类path的方法绘制轮廓（本质也是生成顶点）
var shape = new THREE.Shape();
// 四条直线绘制一个矩形轮廓
shape.moveTo(0,0);//起点
shape.lineTo(0,100);//第2点
shape.lineTo(100,100);//第3点
shape.lineTo(100,0);//第4点
shape.lineTo(0,0);//第5点
```

### shape外轮廓和内轮廓

shape可以用来绘制外轮廓，也可以用来绘制内轮廓，`ShapeGeometry`会使用三角形自动填充shape内轮廓和外轮廓中间的中部。

下面给出了几个通过shape绘制的轮廓图案。

![1.jpg](D:\Typora\pictures\threejs\threejs461.jpg)

```javascript
// 圆弧与直线连接
var shape = new THREE.Shape(); //Shape对象
var R = 50;
// 绘制一个半径为R、圆心坐标(0, 0)的半圆弧
shape.absarc(0, 0, R, 0, Math.PI);
//从圆弧的一个端点(-R, 0)到(-R, -200)绘制一条直线
shape.lineTo(-R, -200);
// 绘制一个半径为R、圆心坐标(0, -200)的半圆弧
shape.absarc(0, -200, R, Math.PI, 2 * Math.PI);
//从圆弧的一个端点(R, -200)到(-R, -200)绘制一条直线
shape.lineTo(R, 0);
var geometry = new THREE.ShapeGeometry(shape, 30);
```

![2](D:\Typora\pictures\threejs\threejs462.jpg)

```javascript
// 一个外轮廓圆弧嵌套三个内圆弧轮廓
var shape = new THREE.Shape(); //Shape对象
//外轮廓
shape.arc(0, 0, 100, 0, 2 * Math.PI);
// 内轮廓1
var path1 = new THREE.Path();
path1.arc(0, 0, 40, 0, 2 * Math.PI);
// 内轮廓2
var path2 = new THREE.Path();
path2.arc(80, 0, 10, 0, 2 * Math.PI);
// 内轮廓3
var path3 = new THREE.Path();
path3.arc(-80, 0, 10, 0, 2 * Math.PI);
//三个内轮廓分别插入到holes属性中
shape.holes.push(path1, path2, path3);
```

![3.jpg](D:\Typora\pictures\threejs\threejs463.jpg)

```javascript
// 矩形嵌套矩形或圆弧
var shape=new THREE.Shape();//Shape对象
//外轮廓
shape.moveTo(0,0);//起点
shape.lineTo(0,100);//第2点
shape.lineTo(100,100);//第3点
shape.lineTo(100,0);//第4点
shape.lineTo(0,0);//第5点

//内轮廓
var path=new THREE.Path();//path对象
// path.arc(50,50,40,0,2*Math.PI);//圆弧
path.moveTo(20,20);//起点
path.lineTo(20,80);//第2点
path.lineTo(80,80);//第3点
path.lineTo(80,20);//第4点
path.lineTo(20,20);//第5点
shape.holes.push(path);//设置内轮廓
```

### 多个轮廓同时填充

![img](D:\Typora\pictures\threejs\threejs4622.jpg)

```javascript
// 轮廓对象1
 var shape=new THREE.Shape();
 shape.arc(-50,0,30,0,2*Math.PI);
 // 轮廓对象2
 var shape2=new THREE.Shape();
 shape2.arc(50,0,30,0,2*Math.PI);
 // 轮廓对象3
 var shape3=new THREE.Shape();
 shape3.arc(0,50,30,0,2*Math.PI);
// 多个shape作为元素组成数组,每一个shpae可以理解为一个要填充的轮廓
var geometry = new THREE.ShapeGeometry([shape,shape2,shape3], 30);
```

### 实例：根据河南边界坐标填充轮廓

![img](D:\Typora\pictures\threejs\threejs464.jpg)

```javascript
// 河南边界轮廓坐标
let arr = [
  [110.3906, 34.585],
  [110.8301, 34.6289],
...
...
...
  [110.6543, 34.1455],
  [110.4785, 34.2334],
  [110.3906, 34.585]
]
var points = [];
// 转化为Vector2构成的顶点数组
arr.forEach(elem => {
  points.push(new THREE.Vector2(elem[0],elem[1]))
});
// 样条曲线生成更多的点
var SplineCurve = new THREE.SplineCurve(points)
var shape = new THREE.Shape(SplineCurve.getPoints(300));
// var shape = new THREE.Shape(points);
var geometry = new THREE.ShapeGeometry(shape);
geometry.center();//几何体居中
geometry.scale(30,30,30);//几何体缩放
var material = new THREE.MeshPhongMaterial({
  color: 0x0000ff,
  side: THREE.DoubleSide //两面可见
}); //材质对象
var mesh = new THREE.Mesh(geometry, material); //网格模型对象
```

## 8 拉伸扫描成型[ExtrudeGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/geometries/ExtrudeGeometry)

![Extrude.png](D:\Typora\pictures\threejs\threejs47Extrude.png)

构造函数[ExtrudeGeometry()](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/geometries/ExtrudeGeometry)和[ShapeGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/geometries/ShapeGeometry)一样是利用[Shape](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/extras/core/Shape)对象生成几何体对象，区别在于`ExtrudeGeometry()`可以利用2D轮廓生成3D模型， 如果你使用任何三维软件都知道可以先绘制一个二维的轮廓图，然后拉伸成型得到三维模型。`ExtrudeGeometry()`第二个参数是拉伸参数，数据类型是对象， 属性`amount`表示拉伸长度，`bevelEnabled`表示拉伸是否产生倒角，其它参数见下表。

构造函数`ExtrudeGeometry()`拉伸参数

| 参数            | 含义                                 |      |
| :-------------- | :----------------------------------- | ---- |
| amount          | 拉伸长度，默认100                    |      |
| bevelEnabled    | 是否使用倒角                         |      |
| bevelSegments   | 倒角细分数，默认3                    |      |
| bevelThickness  | 倒角尺寸(经向)                       |      |
| curveSegments   | 拉伸轮廓细分数                       |      |
| steps           | 拉伸方向细分数                       |      |
| extrudePath     | 扫描路径THREE.CurvePath，默认Z轴方向 |      |
| material        | 前后面材质索引号                     |      |
| extrudeMaterial | 拉伸面、倒角面材质索引号             |      |
| bevelSize       | 倒角尺寸(拉伸方向)                   |      |

```javascript
/**
 * 创建拉伸网格模型
 */
var shape = new THREE.Shape();
/**四条直线绘制一个矩形轮廓*/
shape.moveTo(0,0);//起点
shape.lineTo(0,100);//第2点
shape.lineTo(100,100);//第3点
shape.lineTo(100,0);//第4点
shape.lineTo(0,0);//第5点
var geometry = new THREE.ExtrudeGeometry(//拉伸造型
    shape,//二维轮廓
    //拉伸参数
    {
        amount:120,//拉伸长度
        bevelEnabled:false//无倒角
    }
    );
```

通过使用点模式渲染上面的几何体，可以看出几何体拉伸的本质效果就是空间分布顶点数据的产生。

```javascript
var material=new THREE.PointsMaterial({
    color:0x0000ff,
    size:5.0//点对象像素尺寸
});//材质对象
var mesh=new THREE.Points(geometry,material);//点模型对象
scene.add(mesh);//点模型添加到场景中
```

### 扫描

![scan.png](D:\Typora\pictures\threejs\threejs47scan.png)

拉伸和扫描一样都是三维造型建模方法，three.js提供了一个共同的构造函数来实现扫描和拉伸，对于扫描而言不需要定义`amount`属性设置拉伸距离，设置扫描路径即可， 定义属性`extrudePath`，`extrudePath`的值是路径`THREE.CurvePath`，可以通过样条曲线、贝赛尔曲线构造函数创建不规则曲线扫描轨迹。

```javascript
/**
* 创建扫描网格模型
*/
var shape = new THREE.Shape();
/**四条直线绘制一个矩形轮廓*/
shape.moveTo(0,0);//起点
shape.lineTo(0,10);//第2点
shape.lineTo(10,10);//第3点
shape.lineTo(10,0);//第4点
shape.lineTo(0,0);//第5点
/**创建轮廓的扫描轨迹(3D样条曲线)*/
var curve = new THREE.SplineCurve3([
   new THREE.Vector3( -10, -50, -50 ),
   new THREE.Vector3( 10, 0, 0 ),
   new THREE.Vector3( 8, 50, 50 ),
   new THREE.Vector3( -5, 0, 100)
]);
var geometry = new THREE.ExtrudeGeometry(//拉伸造型
   shape,//二维轮廓
   //拉伸参数
   {
       bevelEnabled:false,//无倒角
       extrudePath:curve,//选择扫描轨迹
       steps:50//扫描方向细分数
   }
);
```

# 纹理贴图

![image-20220923210211824](D:\Typora\pictures\threejs\image-20220923210211824.png)

## 1 创建纹理贴图

​	通过纹理贴图加载器[TextureLoader](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/loaders/TextureLoader)的`load()`方法加载一张图片可以返回一个纹理对象[Texture](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/textures/Texture)，纹理对象`Texture`可以作为模型材质颜色贴图`.map`属性的值。

​	材质的颜色贴图属性`.map`设置后，模型会从纹理贴图上采集像素值，这时候一般来说不需要再设置材质颜色`.color`。`.map`贴图之所以称之为颜色贴图就是因为网格模型会获得颜色贴图的颜色值RGB。

```javascript
// 纹理贴图映射到一个矩形平面上
var geometry = new THREE.PlaneGeometry(204, 102); //矩形平面
// TextureLoader创建一个纹理加载器对象，可以加载图片作为几何体纹理
var textureLoader = new THREE.TextureLoader();
// 执行load方法，加载纹理贴图成功后，返回一个纹理对象Texture
textureLoader.load('Earth.png', function(texture) {
  var material = new THREE.MeshLambertMaterial({
    // color: 0x0000ff,
    // 设置颜色纹理贴图：Texture对象作为材质map属性的属性值
    map: texture,//设置颜色贴图属性值
  }); //材质对象Material
  var mesh = new THREE.Mesh(geometry, material); //网格模型对象Mesh
  scene.add(mesh); //网格模型添加到场景中

  //纹理贴图加载成功后，调用渲染函数执行渲染操作
  // render();
})
```

​	不同的几何体有不同的UV坐标来设置贴图和模型的映射规律，你可以尝试把颜色纹理贴图映射到不同的几何体上查看渲染效果。

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100); //立方体
var geometry = new THREE.SphereGeometry(60, 25, 25); //球体
```

### 纹理对象`Texture`

如果你想进一步了解`.map`的属性值[Texture](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/textures/Texture)可以阅读下面的代码。

通过图片加载器[ImageLoader](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/loaders/ImageLoader)可以加载一张图片，所谓纹理对象Texture简单地说就是，纹理对象Texture的`.image`属性值是一张图片。

```javascript
// 图片加载器
var ImageLoader = new THREE.ImageLoader();
// load方法回调函数，按照路径加载图片，返回一个html的元素img对象
ImageLoader.load('Earth.png', function(img) {
  // image对象作为参数，创建一个纹理对象Texture
  var texture = new THREE.Texture(img);
  // 下次使用纹理时触发更新
  texture.needsUpdate = true;
  var material = new THREE.MeshLambertMaterial({
    map: texture, //设置纹理贴图
  });
  var mesh = new THREE.Mesh(geometry, material); //网格模型对象Mesh
  scene.add(mesh); //网格模型添加到场景中
});
```

### 总结

![image-20220923210241051](D:\Typora\pictures\threejs\image-20220923210241051.png)

## 2 顶点纹理坐标UV(未学习)

​	在课程的第二章对Threejs几何体[Geometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Geometry)和[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)的顶点概念做过比较多的介绍，讲解过顶点位置坐标数据、顶点颜色数据、顶点法线方向向量数据，不过顶点的UV数据没有去讲解，主要是几何体顶点的纹理坐标数据和纹理贴图的映射有关系，所以放在了本章节去讲解。

![image-20220923210318563](D:\Typora\pictures\threejs\image-20220923210318563.png)

### 纹理UV坐标

纹理坐标含义就是一面意思，一张纹理贴图图像的坐标，选择一张图片，比如以图片左下角为坐标原点，右上角为坐标(1.0,1.0)，图片上所有位置纵横坐标都介于0.0~1.0之间。

### 映射

纹理UV坐标和顶点位置坐标是一一对应关系，这也就是为什么一张图片可以映射到一个模型的表面，只要把图片的每个纹理坐标和模型的顶点位置建立一对一的关系，就可以实现图像到模型的映射。

![img](D:\Typora\pictures\threejs\threejs50映射.png)

矩形贴图和球面的映射图 ![img](D:\Typora\pictures\threejs\threejs50球映射.png)

### 两组UV坐标

​	**几何体有两组UV坐标，第一组组用于`.map`、`.normalMap`、`.specularMap`等贴图的映射，第二组用于阴影贴图`.lightMap`的映射**，这里不过过多阐述，本章节除了8.7用到的是第二组UV坐标，其它的章节内部程序用到的都是第一组UV坐标。

### 修改纹理坐标

​	你可以尝试修改上节课代码中几何体的纹理坐标，然后体会纹理坐标的作用。

​	几何体表面所有位置全部对应贴图(0.4,0.4)坐标位置的像素值，这样话网格模型不会显示完整的地图，而是显示采样点纹理坐标`(0.4,0.4)`对应的RGB值。

```javascript
 //矩形平面，细分数默认1，即2个三角形拼接成一个矩形
var geometry = new THREE.PlaneGeometry(204, 102);
...
/**
 * 遍历uv坐标
 */
geometry.faceVertexUvs[0].forEach(elem => {
  elem.forEach(Vector2 => {
    // 所有的UV坐标全部设置为一个值
    Vector2.set(0.4,0.4);
  });
});
```

原来几何体平面默认是两个三角形构成，把细分数设置为4，三角形数量变为16个。

```javascript
// 矩形平面 设置细分数4,4
var geometry = new THREE.PlaneGeometry(204, 102, 4, 4);
...
/**
 * 局部三角面显示完整纹理贴图
 */
var t0 = new THREE.Vector2(0, 1); //图片左下角
var t1 = new THREE.Vector2(0, 0); //图片右下角
var t2 = new THREE.Vector2(1, 0); //图片右上角
var t3 = new THREE.Vector2(1, 1); //图片左上角
var uv1 = [t0, t1, t3]; //选中图片一个三角区域像素——用于映射到一个三角面
var uv2 = [t1, t2, t3]; //选中图片一个三角区域像素——用于映射到一个三角面
// 设置第五、第六个三角形面对应的纹理坐标
geometry.faceVertexUvs[0][4] = uv1
geometry.faceVertexUvs[0][5] = uv2
```

### `Geometry`自定义顶点UV坐标

一般Threejs的球体、圆柱等几何体创建的时候，都会通过特定算法自动生成几何体的UV坐标。

下面代码通过几何体[Geometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Geometry)自定义了一个由两个三角形组成的矩形几何体，并且通过几何体的`.faceVertexUvs[0]`属性设置了每个顶点对应的第一组UV坐标。

```javascript
var geometry = new THREE.Geometry(); //创建一个空几何体对象
/**顶点坐标(纹理映射位置)*/
var p1 = new THREE.Vector3(0,0,0); //顶点1坐标
var p2 = new THREE.Vector3(160,0,0); //顶点2坐标
var p3 = new THREE.Vector3(160,80,0); //顶点3坐标
var p4 = new THREE.Vector3(0,80,0); //顶点4坐标
geometry.vertices.push(p1,p2,p3,p4); //顶点坐标添加到geometry对象
/** 三角面1、三角面2*/
var normal = new THREE.Vector3( 0, 0, 1 ); //三角面法向量
var face0 = new THREE.Face3( 0, 1, 2, normal); //三角面1
var face1 = new THREE.Face3( 0, 2, 3, normal); //三角面2
geometry.faces.push( face0,face1 ); //三角面1、2添加到几何体
/**纹理坐标*/
var t0 = new THREE.Vector2(0,0);//图片左下角
var t1 = new THREE.Vector2(1,0);//图片右下角
var t2 = new THREE.Vector2(1,1);//图片右上角
var t3 = new THREE.Vector2(0,1);//图片左上角
uv1 = [t0,t1,t2];//选中图片一个三角区域像素——映射到三角面1
uv2 = [t0,t2,t3];//选中图片一个三角区域像素——映射到三角面2
geometry.faceVertexUvs[0].push(uv1,uv2);//纹理坐标传递给纹理三角面属性
```

### `BufferGeometry`自定义顶点UV坐标

下面代码通过几何体[BufferGeometry](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/BufferGeometry)自定义了一个由两个三角形组成的矩形几何体，并且通过几何体的`.attributes.uv`属性设置了每个顶点对应的第一组UV坐标。

```javascript
var geometry = new THREE.BufferGeometry(); //声明一个空几何体对象
//类型数组创建顶点位置position数据
var vertices = new Float32Array([
  0, 0, 0, //顶点1坐标
  80, 0, 0, //顶点2坐标
  80, 80, 0, //顶点3坐标
  0, 80, 0, //顶点4坐标
]);
// 创建属性缓冲区对象
var attribue = new THREE.BufferAttribute(vertices, 3); //3个为一组
// 设置几何体attributes属性的位置position属性
geometry.attributes.position = attribue
var normals = new Float32Array([
  0, 0, 1, //顶点1法向量
  0, 0, 1, //顶点2法向量
  0, 0, 1, //顶点3法向量
  0, 0, 1, //顶点4法向量
]);
// 设置几何体attributes属性的位置normal属性
geometry.attributes.normal = new THREE.BufferAttribute(normals, 3); //3个为一组,表示一个顶点的xyz坐标
// Uint16Array类型数组创建顶点索引数据
var indexes = new Uint16Array([
  0, 1, 2, 0, 2, 3,
])
// 索引数据赋值给几何体的index属性
geometry.index = new THREE.BufferAttribute(indexes, 1); //1个为一组
 /**纹理坐标*/
 var uvs = new Float32Array([
   0,0, //图片左下角
   1,0, //图片右下角
   1,1, //图片右上角
   0,1, //图片左上角
 ]);
 // 设置几何体attributes属性的位置normal属性
 geometry.attributes.uv = new THREE.BufferAttribute(uvs, 2); //2个为一组,表示一个顶点的纹理坐标
```

### 加载一个包含UV坐标的模型文件

下面案例代码是通过Threejs加载一个包含UV坐标的外部三维模型文件，加载成功后，给模型设置一张贴图.

```javascript
// 创建一个加载threejs格式JSON文件的加载器
var loader = new THREE.ObjectLoader();
// TextureLoader创建一个纹理加载器对象，可以加载图片作为几何体纹理
var textureLoader = new THREE.TextureLoader();
loader.load('model.json',function (obj) {
  console.log(obj);
  scene.add(obj);//加载返回的对象插入场景中
  // 执行load方法，加载纹理贴图成功后，返回一个纹理对象Texture
  textureLoader.load('Earth.png', function(texture) {
    // 设置球体网格模型材质的map属性
    obj.children[0].material.map = texture;
    // 告诉threejs渲染器系统，材质对象的map属性已更新
    obj.children[0].material.needsUpdate=true;
  })
})
```

## 3 （没学习）数组材质、材质索引`.materialIndex`

这节课为大家讲解数组材质和三角形面[Face3](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Face3)的材质索引属性`.materialIndex`。

### 数组材质

你可以测试把数组材质作为几何体的纹理贴图，**所谓数组材质就是多个材质对象构成一个数组作为模型对象的材质。**

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100); //立方体
// var geometry = new THREE.PlaneGeometry(204, 102, 4, 4); //矩形平面
// var geometry = new THREE.SphereGeometry(60, 25, 25); //球体
// var geometry = new THREE.CylinderGeometry(60, 60, 25,25); //圆柱
//
// 材质对象1
var material_1 = new THREE.MeshPhongMaterial({
  color: 0xffff3f
})
var textureLoader = new THREE.TextureLoader(); // 纹理加载器
var texture = textureLoader.load('Earth.png'); // 加载图片，返回Texture对象
// 材质对象2
var material_2 = new THREE.MeshLambertMaterial({
  map: texture, // 设置纹理贴图
  // wireframe:true,
});
// 设置材质数组
var materialArr = [material_2, material_1, material_1, material_1, material_1, material_1];

// 设置数组材质对象作为网格模型材质参数
var mesh = new THREE.Mesh(geometry, materialArr); //网格模型对象Mesh
scene.add(mesh); //网格模型添加到场景中
```

### 材质索引属性

三角形面[Face3](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/core/Face3)可以设置材质索引属性`.materialIndex`,`Face3.materialIndex`指向数组材质中的材质对象，表达的意思是数组材质中哪一个元素用于渲染该三角形面`Face3`。

通过材质属性`Face3.materialIndex`的介绍，你应该可以明白上面案例代码中数组材质的渲染规律。

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100); //立方体
// 你可以测试BoxGeometry、PlaneGeometry、CylinderGeometry三角形面的材质索引
// 查看face3对象的materialIndex属性
console.log(geometry.faces);
geometry.faces.forEach(elem => {
  console.log(elem.materialIndex);
});
```

### 案例：自定义Face3的材质索引

```javascript
var geometry = new THREE.PlaneGeometry(204, 102, 4, 4); //矩形平面
// 材质对象1
var material1 = new THREE.MeshPhongMaterial({
  color: 0xffff3f,
  // wireframe:true,
})
// 材质对象2
var material2 = new THREE.MeshPhongMaterial({
  color: 0x0000ff,
  // wireframe:true,
}); //材质对象Material
// 数组材质
var materialArr = [material1, material2];
// 设置几何体的材质索引(对于PlaneGeometry而言所有Face3的材质索引默认0)
geometry.faces[4].materialIndex = 1;
geometry.faces[5].materialIndex = 1;
var mesh = new THREE.Mesh(geometry, materialArr); //网格模型对象Mesh
```

### 总结

![image-20220923210343921](D:\Typora\pictures\threejs\image-20220923210343921.png)

## 4 纹理对象`Texture`阵列、偏移、旋转

​	8.1节给大家提到过纹理对象[Texture](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/textures/Texture)，简单的说纹理对象`Texture`就是包含一张图片的对象,纹理对象`Texture`所包含的图片就是`.image`属性，除此外，纹理对象Texture还提供了一些实际开发中经常会用到的属性和方法。

### 阵列

纹理贴图阵列映射。

```javascript
var texture = textureLoader.load('太阳能板.png');
// 设置阵列模式   默认ClampToEdgeWrapping  RepeatWrapping：阵列  镜像阵列：MirroredRepeatWrapping
texture.wrapS = THREE.RepeatWrapping;
texture.wrapT = THREE.RepeatWrapping;
// uv两个方向纹理重复数量
texture.repeat.set(4, 2);
```

### 偏移

不设置阵列纹理贴图，只设置偏移

```javascript
var textureLoader = new THREE.TextureLoader();
var texture = textureLoader.load('太阳能板2.png');// 加载纹理贴图
// 不设置重复  偏移范围-1~1
texture.offset = new THREE.Vector2(0.3, 0.1)
```

阵列纹理贴图的同时，进行偏移设置

```javascript
// 设置阵列模式
texture.wrapS = THREE.RepeatWrapping;
texture.wrapT = THREE.RepeatWrapping;
// uv两个方向纹理重复数量
texture.repeat.set(4, 2);
// 偏移效果
texture.offset = new THREE.Vector2(0.5, 0.5)
```

### 纹理旋转

```javascript
var texture = textureLoader.load('太阳能板.png'); // 加载纹理贴图
// 设置纹理旋转角度
texture.rotation = Math.PI/4;
// 设置纹理的旋转中心，默认(0,0)
texture.center.set(0.5,0.5);
console.log(texture.matrix);
```

### 案例：草地效果

提供一张宽高尺寸比较小的草地贴图，然后通过该贴图设置一片范围比较广的草地效果，这时候阵列贴图是比较好的选择。

```javascript
/**
 * 创建一个地面
 */
var geometry = new THREE.PlaneGeometry(1000, 1000); //矩形平面
// 加载树纹理贴图
var texture = new THREE.TextureLoader().load("grass.jpg");
// 设置阵列
texture.wrapS = THREE.RepeatWrapping;
texture.wrapT = THREE.RepeatWrapping;
// uv两个方向纹理重复数量
texture.repeat.set(10, 10);
var material = new THREE.MeshLambertMaterial({
  map: texture,
});
var mesh = new THREE.Mesh(geometry, material); //网格模型对象Mesh
scene.add(mesh); //网格模型添加到场景中
mesh.rotateX(-Math.PI / 2);
```

### 纹理动画

纹理动画比较简单，必须要在渲染函数中`render()`一直执行`texture.offset.x -= 0.06`动态改变纹理对象`Texture`的偏移属性`.offset`就可以。 ![img](D:\Typora\pictures\threejs\threejs52纹理动画.gif)

```javascript
// 渲染函数
function render() {
  renderer.render(scene, camera); //执行渲染操作
  requestAnimationFrame(render);
  // 使用加减法可以设置不同的运动方向
  // 设置纹理偏移
  texture.offset.x -= 0.06
}
render();
/**
 * 创建一个设置重复纹理的管道
 */
var curve = new THREE.CatmullRomCurve3([
  new THREE.Vector3(-80, -40, 0),
  new THREE.Vector3(-70, 40, 0),
  new THREE.Vector3(70, 40, 0),
  new THREE.Vector3(80, -40, 0)
]);
var tubeGeometry = new THREE.TubeGeometry(curve, 100, 0.6, 50, false);
var textureLoader = new THREE.TextureLoader();
var texture = textureLoader.load('run.jpg');
// 设置阵列模式为 RepeatWrapping
texture.wrapS = THREE.RepeatWrapping
texture.wrapT=THREE.RepeatWrapping
// 设置x方向的偏移(沿着管道路径方向)，y方向默认1
//等价texture.repeat= new THREE.Vector2(20,1)
texture.repeat.x = 20;
var tubeMaterial = new THREE.MeshPhongMaterial({
  map: texture,
  transparent: true,
});
```

### 总结

![image-20220923210404312](D:\Typora\pictures\threejs\image-20220923210404312.png)

## 5 canvas画布、视频作为纹理贴图

通过Three.js两个类[CanvasTexture](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/textures/CanvasTexture)和[VideoTexture](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/textures/VideoTexture)可以分别实现把Canvas画布、视频作为纹理贴图使用。

### Canvas画布作为Three.js纹理贴图(`CanvasTexture`)

Canvas画布可以通过2D API绘制各种各样的几何形状，可以通过Canvas绘制一个轮廓后然后作为Three.js网格模型、精灵模型等模型对象的纹理贴图。

### 一段Canvas代码

下面是一段与WebGL或者说Threejs无关的Canvas代码，你可以复制到.html文件中打开查看一下，通过下面代码绘制的Cnavas画布可以作为Three.js模型对象的纹理贴图。

```html
<script type="text/javascript">
var canvas = document.createElement("canvas");
canvas.width = 512;
canvas.height = 128;
var c = canvas.getContext('2d');
// 矩形区域填充背景
c.fillStyle = "#ff00ff";
c.fillRect(0, 0, 512, 128);
  c.beginPath();
// 文字
c.beginPath();
c.translate(256,64);
c.fillStyle = "#000000"; //文本填充颜色
c.font = "bold 48px 宋体"; //字体样式设置
c.textBaseline = "middle"; //文本与fillText定义的纵坐标
c.textAlign = "center"; //文本居中(以fillText定义的横坐标)
c.fillText("郭隆邦_技术博客", 0, 0);

document.body.appendChild(canvas)
</script>
```

把绘制了几何图案的canvas元素作为构造函数[CanvasTexture](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/textures/CanvasTexture)的参数创建一个canvas纹理贴图。

```javascript
/**
 * 创建一个canvas对象，并绘制一些轮廓
 */
var canvas = document.createElement("canvas");
// 上面canvas代码省略
...
c.fillText("郭隆邦_技术博客", 0, 0);

// canvas画布对象作为CanvasTexture的参数重建一个纹理对象
// canvas画布可以理解为一张图片
var texture = new THREE.CanvasTexture(canvas);
//打印纹理对象的image属性
// console.log(texture.image);
//矩形平面
var geometry = new THREE.PlaneGeometry(128, 32);

var material = new THREE.MeshPhongMaterial({
  map: texture, // 设置纹理贴图
});
// 创建一个矩形平面网模型，Canvas画布作为矩形网格模型的纹理贴图
var mesh = new THREE.Mesh(geometry, material);
```

### Canvas画布加载图片

如果作为纹理贴图使用的Canvas画布加载了图片，注意在图片加载完成的时候更新Threejs相关模型的纹理贴图。如果不更新纹理，你会发现canvas画布上的图片无法现在是Threejs模型的纹理上。

```javascript
var canvas = document.createElement("canvas");
...
var ctx = canvas.getContext('2d');
var Image = new Image();
Image.src = "./贴图.jpg";
Image.onload = function() {
  var bg = ctx.createPattern(Image, "no-repeat");
...
// 注意图片加载完成执行canvas相关方法后，要更新一下纹理
  texture.needsUpdate = true;
}
```

### 视频作为Three.js纹理贴图(`VideoTexture`)

视频本质上就是一帧帧图片流构成，把视频作为Threejs模型的纹理贴图使用，就是从视频中提取一帧一帧的图片作为模型的纹理贴图，然后不停的更新的纹理贴图就可以产生视频播放的效果。

下面是一段视频作为纹理贴图的代码。

```javascript
// 创建video对象
let video = document.createElement('video');
video.src = "1086x716.mp4"; // 设置视频地址
video.autoplay = "autoplay"; //要设置播放
// video对象作为VideoTexture参数创建纹理对象
var texture = new THREE.VideoTexture(video)
var geometry = new THREE.PlaneGeometry(108, 71); //矩形平面
var material = new THREE.MeshPhongMaterial({
  map: texture, // 设置纹理贴图
}); //材质对象Material
var mesh = new THREE.Mesh(geometry, material); //网格模型对象Mesh
scene.add(mesh); //网格模型添加到场景中
```

VideoTexture.js封装了一个update函数，Threejs每次执行渲染方法进行渲染场景中的时候，都会执行VideoTexture封装的update函数，执行update函数中代码`this.needsUpdate = true;`读取视频流最新一帧图片来更新Threejs模型纹理贴图。

## 6 凹凸贴图`bumpMap`和法线贴图`.normalMap`

一个复杂的曲面模型，往往模型顶点数量比较多，模型文件比较大，为了降低模型文件大小，法线贴图`.normalMap`算法自然就产生了，复杂的三维模型3D美术可以通过减面操作把精模简化为简模，然后把精模表面的复杂几何信息映射到法线贴图`.normalMap`上。

![image-20220923210539225](D:\Typora\pictures\threejs\image-20220923210539225.png)

### 法线贴图

下面代码在没有设置法线贴图之前就是一个立方体网格模型`Mesh`,然后把一个携带圆形凹坑信息的法线贴图`3_256.jpg`设置到立方体网格模型的面上，你可以看到面上多个凹陷效果。你可以测试源码案例中法线贴图目录下的其它法线贴图文件，查看渲染效果。

法线贴图`3_256.jpg`

![img](C:\Users\22625\Desktop\LearnThreejs\src\assets\imags\normal.jpg)

立方体设置法线贴图后的效果

![img](D:\Typora\pictures\threejs\threejs540.jpg)

```javascript
// TextureLoader创建一个纹理加载器对象，可以加载图片作为几何体纹理
var textureLoader = new THREE.TextureLoader();
// 加载法线贴图
var textureNormal = textureLoader.load('./normal3_256.jpg');
var material = new THREE.MeshPhongMaterial({
  color: 0xff0000,
  normalMap: textureNormal, //法线贴图
  //设置深浅程度，默认值(1,1)。
  normalScale: new THREE.Vector2(3, 3),
}); //材质对象Material
var mesh = new THREE.Mesh(geometry, material); //网格模型对象Mesh
```

### 法线贴图:地球案例

地球表面法线贴图记录了地面表面的几何信息。

![img](D:\Typora\pictures\threejs\threejs54EarthNormal.png)

你可以对比两个地球的渲染效果，一个设置法线贴图，一个不设置法线贴图。

![img](D:\Typora\pictures\threejs\threejs54地球法线.jpg)

```javascript
var geometry = new THREE.SphereGeometry(100, 25, 25); //球体
// TextureLoader创建一个纹理加载器对象，可以加载图片作为几何体纹理
var textureLoader = new THREE.TextureLoader();
// 加载纹理贴图
var texture = textureLoader.load('./Earth.png');
// 加载法线贴图
var textureNormal = textureLoader.load('./EarthNormal.png');
var material = new THREE.MeshPhongMaterial({
  map: texture, // 普通颜色纹理贴图
  normalMap: textureNormal, //法线贴图
  //设置深浅程度，默认值(1,1)。
  normalScale: new THREE.Vector2(1.2, 1.2),
}); //材质对象Material
```

### 凹凸贴图

凹凸贴图和法线贴图功能相似，只是没有法线贴图表达的几何体表面信息更丰富。凹凸贴图是用图片像素的灰度值表示几何表面的高低深度，如果模型定义了法线贴图，就没有必要在使用凹凸贴图。

你可以对比两面墙一个使用凹凸贴图一个不使用凹凸贴图的视觉效果。

![img](D:\Typora\pictures\threejs\threejs54墙0.jpg) ![img](D:\Typora\pictures\threejs\threejs54墙1.jpg)

threejs54地球法线

```javascript
var textureLoader = new THREE.TextureLoader();
// 加载颜色纹理贴图
var texture = textureLoader.load('./凹凸贴图/diffuse.jpg');
// 加载凹凸贴图
var textureBump = textureLoader.load('./凹凸贴图/bump.jpg');
var material = new THREE.MeshPhongMaterial({
  map: texture,// 普通纹理贴图
  bumpMap:textureBump,//凹凸贴图
  bumpScale:3,//设置凹凸高度，默认值1。
}); //材质对象Material
```

## 7 光照贴图添加阴影(`·lightMap`)

![img](D:\Typora\pictures\threejs\threejs550.jpg)

在三维场景中有时候需要设置模型的阴影，也就是阴影贴图或者说光照贴图`·lightMap`，一般Threejs加载外部模型的光照贴图`·lightMap`，三维模型加载器可以自动设置，不需要程序员通过代码去设置，不过为了让大家更好理解光照贴图`·lightMap`，这里就通过Three.js代码设置场景模型的阴影贴图`·lightMap`。

```javascript
//创建一个平面几何体作为投影面
var planeGeometry = new THREE.PlaneGeometry(300, 200);

planeGeometry.faceVertexUvs[1] = planeGeometry.faceVertexUvs[0];
var textureLoader = new THREE.TextureLoader();
// 加载光照贴图
var textureLight = textureLoader.load('shadow.png');
var planeMaterial = new THREE.MeshLambertMaterial({
  color: 0x999999,
  lightMap:textureLight,// 设置光照贴图
  // lightMapIntensity:0.5,//烘培光照的强度. 默认 1.
});
var planeMesh = new THREE.Mesh(planeGeometry, planeMaterial); //网格模型对象Mesh
...
```

课程5.2节设置模型的阴影是通过实时计算得到的，而光照贴图`·lightMap`是3D美术渲染好提供给程序员。这两种方式相比较通过贴图的方式更为节约资源，提高渲染性功能。

### `Geometry`属性`.faceVertexUvs`

一般几何体有两套UV坐标，对于`Geometry`类型几何体而言

`Geometry.faceVertexUvs[0]`包含的纹理坐标用于颜色贴图map、法线贴图normalMap等,`Geometry.faceVertexUvs[1]`包含的第二套纹理贴图用于光照阴影贴图

一般通过Threejs几何体API创建的几何体默认只有一组纹理坐标`Geometry.faceVertexUvs[0]`，所以为了设置光照阴影贴图，需要给另一组纹理坐标赋值`Geometry.faceVertexUvs[1] = Geometry.faceVertexUvs[0];`

### `BufferGeometry`属性`.uv`和`.uv2`

一般通过Threejs加载外部模型，解析三维模型数据得到的几何体类型是缓冲类型几何体`BufferGeometry`，对于`BufferGeometry`而言两套纹理坐标分别通过`.uv`和`.uv2`属性表示。

```javascript
geometry.attributes.uv2 = geometry.attributes.uv;
```

## 8 高光贴图(`.specularMap`)

高光网格材质[MeshPhongMaterial](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/MeshPhongMaterial)具有高光属性`.specular`,如果一个网格模型`Mesh`都是相同的材质并且表面粗糙度相同,或者说网格模型外表面所有不同区域的镜面反射能力相同，可以直接设置材质的高光属性`.specular`。如果一个网格模型表示一个人，那么人的不同部位高光程度是不同的，不可能直接通过`.specular`属性来描述，在这种情况通过高光贴图`.specularMap`的RGB值来描述不同区域镜面反射的能力，`.specularMap`和颜色贴图`.Map`一样和通过UV坐标映射到模型表面。高光贴图`.specularMap`不同区域像素值不同，表示网格模型不同区域的高光值不同。

下面是一个地球的案例，地球地面和海面的高光值是不同的，海面更为高亮，你可以测试使用高光贴图和不使用高光贴图的渲染效果有什么不同。

```javascript
// 加载纹理贴图
var texture = textureLoader.load('earth_diffuse.png');
// 加载高光贴图
var textureSpecular = textureLoader.load('earth_specular.png');
var material = new THREE.MeshPhongMaterial({
  // specular: 0xff0000,//高光部分的颜色
  shininess: 30,//高光部分的亮度，默认30
  map: texture,// 普通纹理贴图
  specularMap: textureSpecular, //高光贴图
}); //材质对象Material
```

高光贴图属性`.specularMap`和高光属性`.specular`是对应的，也就是说只有高光网格材质对象`MeshPhongMaterial`才具备高光贴图属性`.specularMap`。

## 9 (贴图载入失败)环境贴图(`.envMap`)

Three.js环境贴图`.envMap`字面意思就是三维模型周边环境，比如你渲染一个立方体，立方体放在一个屋子里面，屋子里面的周边环境肯定影响立方体的渲染效果，目的是为了渲染该立方体而不是立方体周围环境，为了更方便所以没必要创建立方体周边环境所有物体的网格模型，可以通过图片来表达立方体周边的环境。

创建一个立方体盒子作为天空盒使用，然后把一个环境中上下左右前后六张视图图片作为立方体盒子的纹理贴图使用。

加环境贴图的6张纹理贴图，可以通过[CubeTextureLoader](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/loaders/CubeTextureLoader)类趋势线。

```javascript
var geometry = new THREE.BoxGeometry(100, 100, 100); //立方体

var loader = new THREE.CubeTextureLoader();
// 所有贴图在同一目录下，可以使用该方法设置共用路径
loader.setPath('环境贴图/');
// 立方体纹理加载器返回立方体纹理对象CubeTexture
var CubeTexture = loader.load(['px.jpg', 'nx.jpg', 'py.jpg', 'ny.jpg', 'pz.jpg', 'nz.jpg']);
//材质对象Material
var material = new THREE.MeshPhongMaterial({
  //网格模型设置颜色，网格模型颜色和环境贴图会进行融合计算
  // color:0xff0000,
  envMap: CubeTexture, //设置环境贴图
  // 环境贴图反射率   控制环境贴图对被渲染三维模型影响程度
  // reflectivity: 0.1,
});
console.log(CubeTexture.image);
var mesh = new THREE.Mesh(geometry, material); //网格模型对象Mesh
scene.add(mesh); //网格模型添加到场景中
```

高光网格材质[MeshPhongMaterial](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/MeshPhongMaterial)和物理PBR材质[MeshStandardMaterial](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/materials/MeshStandardMaterial)通常会使用环境贴图`.envMap`来实现更好的渲染效果。一个材质对应的是普通次时代模型，一个材质对应的是PBR模型。

### 一个用到环境贴图的案例(14.6节)。

```javascript
loader.load('./heart/model.obj', function(obj) {
  // 控制台查看返回结构：包含一个网格模型Mesh的组Group
  console.log(obj);
  scene.add(obj);
  mesh = obj.children[0]; //获得心脏网格模型
  mesh.scale.set(10, 10, 10); //网格模型缩放
  // 创建一个纹理加载器
  var textureLoader = new THREE.TextureLoader();

  // // 加载颜色纹理
  var texture = textureLoader.load('./heart/color.png');
  mesh.material.map = texture;

  // // 加载法线贴图，表面细节更丰富
  var textureNormal = textureLoader.load('./heart/normal.png');
  mesh.material.normalMap = textureNormal
  // 设置深浅程度
  mesh.material.normalScale.set(1.5, 1.5)

  // 设置高光贴图，一个网格模型不同的区域反射光线的能力不同
  var textureSpecular = textureLoader.load('./heart/Specular.png');
  mesh.material.specularMap = textureSpecular;
  mesh.material.specular.set(0xffffff);// 高光反射颜色
  mesh.material.shininess = 100;// 高光高亮程度，默认30

  // 设置环境贴图，反射周围环境，渲染更逼真
  var textureCube = new THREE.CubeTextureLoader()
    .setPath('环境贴图/')
    .load(['px.jpg', 'nx.jpg', 'py.jpg', 'ny.jpg', 'pz.jpg', 'nz.jpg']);
  mesh.material.envMap = textureCube;
})
```

## 10 数据纹理对象`DataTexture`

Three.js数据纹理对象[DataTexture](http://www.yanhuangxueyuan.com/threejs/docs/index.html#api/zh/textures/DataTexture)简单地说就是通过程序创建纹理贴图的每一个像素值。

### 程序生成一张图片的RGB值

![img](D:\Typora\pictures\threejs\threejs580.jpg)

```javascript
var geometry = new THREE.PlaneGeometry(128, 128); //矩形平面
/**
 * 创建纹理对象的像素数据
 */
var width = 32; //纹理宽度
var height = 32; //纹理高度
var size = width * height; //像素大小
var data = new Uint8Array(size * 3); //size*3：像素在缓冲区占用空间
for (let i = 0; i < size * 3; i += 3) {
  // 随机设置RGB分量的值
  data[i] = 255 * Math.random()
  data[i + 1] = 255 * Math.random()
  data[i + 2] = 255 * Math.random()
}
// 创建数据文理对象   RGB格式：THREE.RGBFormat
var texture = new THREE.DataTexture(data, width, height, THREE.RGBFormat);
texture.needsUpdate = true; //纹理更新
//打印纹理对象的image属性
// console.log(texture.image);

var material = new THREE.MeshPhongMaterial({
  map: texture, // 设置纹理贴图
}); //材质对象Material
var mesh = new THREE.Mesh(geometry, material);
```

### 程序生成一张图片的RGBA值

![img](D:\Typora\pictures\threejs\threejs581.jpg)

```javascript
var geometry = new THREE.PlaneGeometry(128, 128); //矩形平面
/**
 * 创建纹理对象的像素数据
 */
var width = 32; //纹理宽度
var height = 32; //纹理高度
var size = width * height; //像素大小
var data = new Uint8Array(size * 4); //size*4：像素在缓冲区占用空间
for (let i = 0; i < size * 4; i += 4) {
  // 随机设置RGB分量的值
  data[i] = 255 * Math.random()
  data[i + 1] = 255 * Math.random()
  data[i + 2] = 255 * Math.random()
  // 设置透明度分量A
  data[i + 3] = 255 * 0.5
}
// 创建数据文理对象   RGBA格式：THREE.RGBAFormat
var texture = new THREE.DataTexture(data, width, height, THREE.RGBAFormat);
texture.needsUpdate = true; //纹理更新
//打印纹理对象的image属性
console.log(texture.image);

var material = new THREE.MeshPhongMaterial({
  map: texture, // 设置纹理贴图
  transparent:true,//允许透明设置
});
var mesh = new THREE.Mesh(geometry, material);
```

### 图片格式

像素值包含RGB三个分量的图片格式有`.jpg`、`.BMP`等格式，通过WebGL原生API加载解析这些类型格式的图片需要设置`gl.RGB`，对于Threejs而言对WebGL进行封装了，`gl.RGB`对应的设置是`THREE.RGBFormat`

像素值包含RGBA四个分量的图片格式有`.PNG`等格式，通过WebGL原生API加载解析这些类型格式的图片需要设置`gl.RGBA`，对于Threejs而言对WebGL进行封装了，`gl.RGBA`对应的设置是`THREE.RGBAFormat`



# GLTF


## 包含内容

相比较obj、stl等格式而言，.gltf格式可以包含更多的模型信息。

.gltf格式文件几乎可以包含所有的三维模型相关信息的数据，比如网格模型、PBR材质、纹理贴图、骨骼、变形、动画、光源、相机...

### GLTF格式信息

如果你有一定的前端基础，那么你对JSON一定不陌生，GLTF文件就是通过JSON的键值对方式来表示模型信息，比如`meshes`表示网格模型信息，`materials`表示材质信息...

```JavaScript
{
  "asset": {
    "version": "2.0",
  },
...
// 模型材质信息
  "materials": [
    {
      "pbrMetallicRoughness": {//PBR材质
        "baseColorFactor": [1,1,0,1],
        "metallicFactor": 0.5,//金属度
        "roughnessFactor": 1//粗糙度
      }
    }
  ],
  // 网格模型数据
  "meshes": ...
  // 纹理贴图
  "images": [
        {
            // uri指向外部图像文件
            "uri": "贴图名称.png"//图像数据也可以直接存储在.gltf文件中
        }
   ],
     "buffers": [
    // 一个buffer对应一个二进制数据块，可能是顶点位置 、顶点索引等数据
    {
      "byteLength": 840,
     //这里面的顶点数据，也快成单独以.bin文件的形式存在   
      "uri": "data:application/octet-stream;base64,AAAAPwAAAD8AAAA/AAAAPwAAAD8AAAC/.......
    }
  ],
}
```

### `.bin`文件

有些glTF文件会关联一个或多个.bin文件，.bin文件以二进制形式存储了模型的顶点数据等信息。
.bin文件中的信息其实就是对应gltf文件中的buffers属性，buffers.bin中的模型数据，可以存储在.gltf文件中,也可以单独一个二进制.bin文件。

```JavaScript
"buffers": [
    {
        "byteLength": 102040,
        "uri": "文件名.bin"
    }
]
```

### 二进制.glb

gltf格式文件不一定就是以扩展名.gltf结尾，**.glb就是gltf格式的二进制文件**。比如你可以把.gltf模型和贴图信息全部合成得到一个.glb文件中，.glb文件相对.gltf文件体积更小，网络传输自然更快。


### gltf格式模型在线预览

你可以通过gltf-viewer平台预览GLTF格式模型，当然你也可以通过three.js editor预览gltf格式模型。

1. **gltf-viewer**：https://gltf-viewer.donmccurdy.com/

2. **three.js editor**:https://threejs.org/editor/

3. **vscode**预览gltf模型：vscode搜索gltf，可以看到glTF Tools的工具


### 导出gltf

可以把**three.js editor**作为工具，加载其他模型导出的obj、fbx等格式，然后转化为gltf格式。

3damx gltf相关插件：https://github.com/BabylonJS/Exporters/releases

blender：最新版本可以直接导出gltf，旧的版本可以通过gltf插件实现。

## 加载GLTF格式文件

# Three.js解析加载.gltf文件

three.js扩展库中提供了gltf模型的加载器`GLTFLoader.js`，查看目录`./threejs/examples/js/loaders/`可以找到文件`GLTFLoader.js`。


three.js不同版本对应的gltf加载`GLTFLoader.js`可能会有差异，具体以你使用的three.js版本对应文档为准。


###  引入`GLTFLoader.js`3

```js
// 课程案例源码中：引入gltf模型加载库GLTFLoader.js
import {GLTFLoader} from '../../../../three.js-r123/examples/jsm/loaders/GLTFLoader.js';
// npm安装工程化开发的时候引入方式
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader.js';
```

### 加载.gltf文件

```JavaScript
// 创建GLTF加载器对象
var loader = new THREE.GLTFLoader();
loader.load( 'gltf模型.gltf', function ( gltf ) {
    console.log('控制台查看gltf对象结构', gltf);
    //gltf.scene可以包含网格模型Mesh、光源Light等信息，至于gltf.scene是否包含光源，要看.gltf文件中是否有光源信息
    console.log('gltf对象场景属性', gltf.scene);
  // 返回的场景对象gltf.scene插入到threejs场景中
  scene.add( gltf.scene );
})
```


### 注意！！！

注意加载外部模型的时候，不能简单的套用代码，这样可能不一定能正常渲染出来，比如光源是否添加、相机渲染空间和模型尺寸是否匹配等问题,模型是否居中等问题。

# CSS2渲染器`CSS2DRenderer.js`

通过three.js扩展库`CSS2DRenderer.js`可以实现把**HTML元素**作为标签，标注三维场景中的三维模型。


### 扩展库CSS2DRenderer.js

threejs扩展库CSS2DRenderer.js提供了两个构造函数CSS2渲染器`THREE.CSS2DRenderer`、CSS2模型对象`THREE.CSS2DObject`。

```js
// 引入threejs扩展库CSS2DRenderer.js
import { CSS2DRenderer, CSS2DObject } from 'three.js-r123/examples/jsm/renderers/CSS2DRenderer.js'
```

### CSS2渲染器`THREE.CSS2DRenderer`

CSS2渲染器`THREE.CSS2DRenderer`和常用的WebGL渲染器`WebGLRenderer`一样都是渲染器，只是渲染技术不同，WebGL渲染器`WebGLRenderer`解析渲染threejs模型对象的时候会调用底层的WebGL API，CSS2渲染器`THREE.CSS2DRenderer`功能是渲染与threejs场景中网格模型绑定的HTML元素。

CSS2渲染器`.domElement`、`.setSize()`、`.render()`等方法和属性功能和WebGL渲染器相同。webgl渲染器的部分属性和方法CSS3渲染是不具备的，比如设置背景颜色的方法`.setClearColor()`。

```JavaScript
// 创建一个CSS2渲染器CSS2DRenderer
var labelRenderer = new THREE.CSS2DRenderer();
labelRenderer.setSize(window.innerWidth, window.innerHeight);
labelRenderer.domElement.style.position = 'absolute';
// 避免renderer.domElement影响HTMl标签定位，设置top为0px
labelRenderer.domElement.style.top = '0px';
labelRenderer.domElement.style.left = '0px';
//设置.pointerEvents=none，以免模型标签HTML元素遮挡鼠标选择场景模型
labelRenderer.domElement.style.pointerEvents = 'none';
document.body.appendChild(labelRenderer.domElement);
...
//渲染场景中的HTMl元素包装成的CSS2模型对象
labelRenderer.render(scene, camera);
```

### CSS2模型对象`CSS2DObject`

CSS2模型对象`CSS2DObject`作用是把HTML元素设计的UI包装为一个类似threejs网格模型`Mesh`的模型对象，可以设置`.position`属性，可以通过`.add()方法`插入到场景中


```JavaScript
/**
 * HTML元素编写一个UI效果作为模型标签
 */
var div = document.createElement('div');
div.innerHTML = '立方体';
div.style.padding = '10px';
div.style.color = '#fff';
div.style.backgroundColor = 'rgba(25,25,25,0.5)';
div.style.borderRadius = '5px'
// div.style.position = 'absolute';//不需要设置绝对定位

//HTML元素标签作为参数创建一个CSS2模型对象CSS2DObject
//你可以把CSS2DObject模型对象类比为网格模型Mesh，一样具有position属性
//CSS2DObject模型对象不具有角度和缩放.scale属性
var label = new THREE.CSS2DObject(div);
//设置模型对象CSS2DObject在场景位置
//标签标注boxMesh模型所以复制boxMesh的位置
label.position.copy(boxMesh.position);
//适当偏移标签
label.position.y += 30
scene.add(label);//CSS2模型标签插入到场景中
```

### 1. `CSS2DRenderer.js`

通过three.js扩展库`CSS2DRenderer.js`可以实现把**HTML元素**作为标签，标注三维场景中的三维模型。


### 2. `.getWorldPosition()`获取世界坐标

如果你想标注一个模型对象，比如一个粮仓，你首先应该知道粮仓模型在三维坐标系中的位置，也就是世界坐标。

通过Three.js`.getWorldPosition()`获取可以获取一个模型在三维坐标系中世界坐标。

```js
var pos = new THREE.Vector3();
obj.getWorldPosition(pos);//获取obj世界坐标
```

### 3. 模型局部坐标系坐标原点

每个模型都有一个`.position`属性，这个属性可以控制模型相对**局部坐标系坐标原点**的位置，一个模型所有父对象和自己的`.position`属性累积起来就是一个模型**相对世界坐标系坐标原点**的位置。

**注意**：通过`.getWorldPosition()`方法可以获取每个粮仓在世界坐标中的坐标，但是你要考虑到粮仓模型是有尺寸的，不是一个点，这个时候，最好通过三维软件调整粮仓模型的局部坐标系坐标原点位置，选择一个**特殊的位置**设置局部坐标系坐标原点，比如坐标原点设置在粮仓模型的底部中心位置，这样坐标的好处是为了精准调整标签位置。

# 第一人称

[three.js/misc_controls_pointerlock.html at master · mrdoob/three.js (github.com)](https://github.com/mrdoob/three.js/blob/master/examples/misc_controls_pointerlock.html)

# 鼠标漫游

[Three.js - FLyControls 飞行控件_「已注销」的博客-CSDN博客](https://blog.csdn.net/ithanmang/article/details/82352555)

[three.js/misc_controls_fly.html at master · mrdoob/three.js (github.com)](https://github.com/mrdoob/three.js/blob/master/examples/misc_controls_fly.html)

~~~html
<!DOCTYPE html>
<html lang="en">
	<head>
		<title>three.js webgl - fly controls - earth</title>
		<meta charset="utf-8">
		<meta name="viewport" content="width=device-width, user-scalable=no, minimum-scale=1.0, maximum-scale=1.0">
		<link type="text/css" rel="stylesheet" href="main.css">
		<style>
			body {
				background:#000;
				color: #eee;
			}

			a {
				color: #0080ff;
			}

			b {
				color: orange
			}
		</style>
	</head>

	<body>

		<div id="info"><a href="https://threejs.org" target="_blank" rel="noopener">three.js</a> - earth [fly controls]<br/>
		<b>WASD</b> move, <b>R|F</b> up | down, <b>Q|E</b> roll, <b>up|down</b> pitch, <b>left|right</b> yaw
		</div>

		<!-- Import maps polyfill -->
		<!-- Remove this when import maps will be widely supported -->
		<script async src="https://unpkg.com/es-module-shims@1.3.6/dist/es-module-shims.js"></script>

		<script type="importmap">
			{
				"imports": {
					"three": "../build/three.module.js",
					"three/addons/": "./jsm/"
				}
			}
		</script>

		<script type="module">

			import * as THREE from 'three';

			import Stats from 'three/addons/libs/stats.module.js';

			import { FlyControls } from 'three/addons/controls/FlyControls.js';
			import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
			import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
			import { FilmPass } from 'three/addons/postprocessing/FilmPass.js';

			const radius = 6371;
			const tilt = 0.41;
			const rotationSpeed = 0.02;

			const cloudsScale = 1.005;
			const moonScale = 0.23;

			const MARGIN = 0;
			let SCREEN_HEIGHT = window.innerHeight - MARGIN * 2;
			let SCREEN_WIDTH = window.innerWidth;

			let camera, controls, scene, renderer, stats;
			let geometry, meshPlanet, meshClouds, meshMoon;
			let dirLight;

			let composer;

			const textureLoader = new THREE.TextureLoader();

			let d, dPlanet, dMoon;
			const dMoonVec = new THREE.Vector3();

			const clock = new THREE.Clock();

			init();
			animate();

			function init() {

				camera = new THREE.PerspectiveCamera( 25, SCREEN_WIDTH / SCREEN_HEIGHT, 50, 1e7 );
				camera.position.z = radius * 5;

				scene = new THREE.Scene();
				scene.fog = new THREE.FogExp2( 0x000000, 0.00000025 );

				dirLight = new THREE.DirectionalLight( 0xffffff );
				dirLight.position.set( - 1, 0, 1 ).normalize();
				scene.add( dirLight );

				const materialNormalMap = new THREE.MeshPhongMaterial( {

					specular: 0x333333,
					shininess: 15,
					map: textureLoader.load( 'textures/planets/earth_atmos_2048.jpg' ),
					specularMap: textureLoader.load( 'textures/planets/earth_specular_2048.jpg' ),
					normalMap: textureLoader.load( 'textures/planets/earth_normal_2048.jpg' ),

					// y scale is negated to compensate for normal map handedness.
					normalScale: new THREE.Vector2( 0.85, - 0.85 )

				} );

				// planet

				geometry = new THREE.SphereGeometry( radius, 100, 50 );

				meshPlanet = new THREE.Mesh( geometry, materialNormalMap );
				meshPlanet.rotation.y = 0;
				meshPlanet.rotation.z = tilt;
				scene.add( meshPlanet );

				// clouds

				const materialClouds = new THREE.MeshLambertMaterial( {

					map: textureLoader.load( 'textures/planets/earth_clouds_1024.png' ),
					transparent: true

				} );

				meshClouds = new THREE.Mesh( geometry, materialClouds );
				meshClouds.scale.set( cloudsScale, cloudsScale, cloudsScale );
				meshClouds.rotation.z = tilt;
				scene.add( meshClouds );

				// moon

				const materialMoon = new THREE.MeshPhongMaterial( {

					map: textureLoader.load( 'textures/planets/moon_1024.jpg' )

				} );

				meshMoon = new THREE.Mesh( geometry, materialMoon );
				meshMoon.position.set( radius * 5, 0, 0 );
				meshMoon.scale.set( moonScale, moonScale, moonScale );
				scene.add( meshMoon );

				// stars

				const r = radius, starsGeometry = [ new THREE.BufferGeometry(), new THREE.BufferGeometry() ];

				const vertices1 = [];
				const vertices2 = [];

				const vertex = new THREE.Vector3();

				for ( let i = 0; i < 250; i ++ ) {

					vertex.x = Math.random() * 2 - 1;
					vertex.y = Math.random() * 2 - 1;
					vertex.z = Math.random() * 2 - 1;
					vertex.multiplyScalar( r );

					vertices1.push( vertex.x, vertex.y, vertex.z );

				}

				for ( let i = 0; i < 1500; i ++ ) {

					vertex.x = Math.random() * 2 - 1;
					vertex.y = Math.random() * 2 - 1;
					vertex.z = Math.random() * 2 - 1;
					vertex.multiplyScalar( r );

					vertices2.push( vertex.x, vertex.y, vertex.z );

				}

				starsGeometry[ 0 ].setAttribute( 'position', new THREE.Float32BufferAttribute( vertices1, 3 ) );
				starsGeometry[ 1 ].setAttribute( 'position', new THREE.Float32BufferAttribute( vertices2, 3 ) );

				const starsMaterials = [
					new THREE.PointsMaterial( { color: 0x555555, size: 2, sizeAttenuation: false } ),
					new THREE.PointsMaterial( { color: 0x555555, size: 1, sizeAttenuation: false } ),
					new THREE.PointsMaterial( { color: 0x333333, size: 2, sizeAttenuation: false } ),
					new THREE.PointsMaterial( { color: 0x3a3a3a, size: 1, sizeAttenuation: false } ),
					new THREE.PointsMaterial( { color: 0x1a1a1a, size: 2, sizeAttenuation: false } ),
					new THREE.PointsMaterial( { color: 0x1a1a1a, size: 1, sizeAttenuation: false } )
				];

				for ( let i = 10; i < 30; i ++ ) {

					const stars = new THREE.Points( starsGeometry[ i % 2 ], starsMaterials[ i % 6 ] );

					stars.rotation.x = Math.random() * 6;
					stars.rotation.y = Math.random() * 6;
					stars.rotation.z = Math.random() * 6;
					stars.scale.setScalar( i * 10 );

					stars.matrixAutoUpdate = false;
					stars.updateMatrix();

					scene.add( stars );

				}

				renderer = new THREE.WebGLRenderer( { antialias: true } );
				renderer.setPixelRatio( window.devicePixelRatio );
				renderer.setSize( SCREEN_WIDTH, SCREEN_HEIGHT );
				document.body.appendChild( renderer.domElement );

				//

				controls = new FlyControls( camera, renderer.domElement );

				controls.movementSpeed = 1000;
				controls.domElement = renderer.domElement;
				controls.rollSpeed = Math.PI / 24;
				controls.autoForward = false;
				controls.dragToLook = false;

				//

				stats = new Stats();
				document.body.appendChild( stats.dom );

				window.addEventListener( 'resize', onWindowResize );

				// postprocessing

				const renderModel = new RenderPass( scene, camera );
				const effectFilm = new FilmPass( 0.35, 0.75, 2048, false );

				composer = new EffectComposer( renderer );

				composer.addPass( renderModel );
				composer.addPass( effectFilm );

			}

			function onWindowResize() {

				SCREEN_HEIGHT = window.innerHeight;
				SCREEN_WIDTH = window.innerWidth;

				camera.aspect = SCREEN_WIDTH / SCREEN_HEIGHT;
				camera.updateProjectionMatrix();

				renderer.setSize( SCREEN_WIDTH, SCREEN_HEIGHT );
				composer.setSize( SCREEN_WIDTH, SCREEN_HEIGHT );

			}

			function animate() {

				requestAnimationFrame( animate );

				render();
				stats.update();

			}

			function render() {

				// rotate the planet and clouds

				const delta = clock.getDelta();

				meshPlanet.rotation.y += rotationSpeed * delta;
				meshClouds.rotation.y += 1.25 * rotationSpeed * delta;

				// slow down as we approach the surface

				dPlanet = camera.position.length();

				dMoonVec.subVectors( camera.position, meshMoon.position );
				dMoon = dMoonVec.length();

				if ( dMoon < dPlanet ) {

					d = ( dMoon - radius * moonScale * 1.01 );

				} else {

					d = ( dPlanet - radius * 1.01 );

				}

				controls.movementSpeed = 0.33 * d;
				controls.update( delta );

				composer.render( delta );

			}

		</script>
	</body>
</html>


```js
asd

```

