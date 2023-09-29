这一节给场景中的模型添加标签，想实现的效果是，通过鼠标点击场景中摩托车的某个部位，则在场景中出现一个标签，并在标签上显示该部位的信息。最终的效果图如下：  

![](1682178795616.png)

  
要实现上面的效果，需要用到 CSS 2D 渲染器，先来了解下 CSS 2D 渲染器

## CSS2DRenderer(CSS 2D 渲染器)

CSS2DRenderer（CSS 2D 渲染器）可以把 HTML 元素作为标签标注到三维场景中，CSS2DRenderer 是 CSS3DRenderer（CSS 3D 渲染器）的简化版本，它唯一支持的变换是位移。通过 CSS2DRenderer 我们可以将三维物体和基于 HTML 的标签相结合，来更好的表达场景中物体的信息。

### 构造函数

CSS2DRenderer()

### 方法

.getSize () 返回一个包含有渲染器宽和高的对象。

.render (scene : Scene, camera : Camera) 使用 camera 渲染 scene。

.setSize (width : Number, height : Number) 将渲染器的尺寸调整为 (width, height).

## 使用 CSS2DRenderer

CSS2DRenderer 是 [threejs](https://so.csdn.net/so/search?q=threejs&spm=1001.2101.3001.7020) 提供的扩展库，在 threejs 文件包目录 examples/jsm/renderers/，文件夹下面可以找到 CSS2DRenderer.js 扩展库。

CSS2DRenderer.js 提供了两个类 CSS2DRenderer（CSS2D 渲染器）和 CSS2DObject（CSS2D 模型对象）

要使用 CSS2DRenderer，我们需要先将其引入

```
// 引入CSS2渲染器CSS2DRenderer和CSS2模型对象CSS2DObject
import { CSS2DRenderer,CSS2DObject } from 'three/examples/jsm/renderers/CSS2DRenderer'
```

## 创建 HTML 标签

在 vue 文件的模板中添加如下代码，都是基础的 HTML 语音，这里不再细说

```
<div id="label">
    <div style="position: relative; width: 139px; height: 167px;color: #ffffff;">
      <img src="../assets/images/info.png" alt="" style="width: 100%; position: absolute;left: 0px;top: 0px;"> 
      <div style="position: absolute; left: 48px;top: 4px;font-size: 12px;">
        <div style="font-size: 14px;font-weight: 400;">
          <span id="txtName">车身</span>
        </div>
        <div style="margin-top: 8px;margin-left: -25px;">
            <span style="color: #fff;font-weight: 300;">材质：</span>
            <span style="font-weight: 400;margin-left: 10px;" id="txtMaterial">金属</span>
        </div>
        <div style="margin-top: 8px;margin-left: -25px;">
            <span style="color: #fff;font-weight: 300;">颜色：</span>
            <span style="font-weight: 400;margin-left: 2px;" id="txtColor">红色</span>
        </div>
      </div>
    </div>
  </div>
```

添加上述代码后，我们创建的标签出现在了浏览器页面的左下方，这是典型的浏览器 div 的排列属性，但是这不是我们想要的，我们想要将这个 [div 标签](https://so.csdn.net/so/search?q=div%E6%A0%87%E7%AD%BE&spm=1001.2101.3001.7020)放到三维场景中去展示。这就涉及到了世界坐标 xyz 与屏幕坐标的转换，这是一个很麻烦的事情。好在 threejs 的扩展库 CSS2DRenderer.js 已经帮我们考虑到了这一点，给我们提供了 CSS2DObject 对象  

![](1682178795723.png)

## CSS2DObject 对象

通过 CSS2DObject 对象，可以把 HTML 元素转化为一个类似 threejs 网格模型的对象，即把 CSS2DObject 当成 threejs 的一个模型一样去设置位置. position 或添加到场景中；

```
const div = document.querySelector('#label')
const tag = new CSS2DObject(div)
```

可以通过.[position 属性](https://so.csdn.net/so/search?q=position%E5%B1%9E%E6%80%A7&spm=1001.2101.3001.7020)设置标签模型对象的 xyz 坐标，调整其在三维世界中的位置。

```
tag.position.set(0,2,0)
```

将 HTML 元素通过 CSS2DObject 转换后，我们就可以把 HTML 元素对应的 CSS2 模型对象添加到其它模型对象或三维场景中。和添加其他对象没什么区别  
**添加到场景**

```
scene.add(tag)
```

**添加到组中**

```
const group = new THREE.Group()
group.add(tag)
```

**添加到 Mesh 中**

```
mesh.add(tag)
```

## 创建 CSS2DRenderer

### 创建 css2dRenderer 对象

```
// 创建一个CSS2渲染器CSS2DRenderer
const css2dRenderer= new CSS2DRenderer()
```

CSS2 渲染器 CSS2DRenderer 和 WebGL 渲染器 WebGLRenderer 相似，它们都有下面几个属性和方法，用法也类似，都有. domElement、.setSize()、.render()

### .setSize 设置大小

```
css2dRenderer.setSize (window.innerWidth,window.innerHeight)
```

### 设置位置

```
css2dRenderer.domElement.style.position = 'absolute'
css2dRenderer.domElement.style.top = '0px'
```

### 将 CSS2Renderer.domElement 添加到 body 中

```
document.body.appendChild(css2dRenderer.domElement)
```

### 渲染 HTML 标签

渲染 HTML 标签使用. render() 方法，和 WebGLRenderer 的 render 方法一样，要写在 rander() 函数中

```
css2dRenderer.render(this.scene,this.camera)
```

理解了上面的知识点后，我们来编写代码

### 创建 initCSS2DRenderer() 初始化函数、调用及渲染

```
initCSS2DRenderer() {
    const div = document.querySelector('#label')
    // HTML元素转换为threejs的css2d模型对象
    const tag = new CSS2DObject(div)
    tag.position.set(0,1.3,0);
    this.scene.add(tag) 

    css2dRenderer = new CSS2DRenderer()
    css2dRenderer.setSize (window.innerWidth,window.innerHeight)
    css2dRenderer.domElement.style.position = 'absolute'
    css2dRenderer.domElement.style.top = '0px'
    // css2dRenderer.domElement.style.zIndex = -1 
    document.body.appendChild(css2dRenderer.domElement)     
  }
```

**在 init() 中调用 initCSS2DRenderer() 函数**

```
// 初始化
  init() {
    // 初始化场景
    this.initScene()   
    // 初始化灯光
    this.initLight()
    // 初始化Mesh
    this.initMesh()
    // 初始化地面
    this.initFloor() 
    // 初始化GUI
    this.initGUI() 
    this.initCSS2DRenderer()
    // 初始化相机
    this.initCamera()
    // 初始化渲染器
    this.initRender() 
    // 初始化轨道控制器
    this.initControls() 
    window.addEventListener('resize',this.onWindowResize.bind(this))
  }
```

**在 render() 中渲染 HTML 标签**

```
render() {     
    this.renderer.render(this.scene,this.camera) 
    css2dRenderer.render(this.scene,this.camera)
    this.controls.update() 
  }
```

完成以后刷新浏览器，可以看到创建的标签已经添加到了三维场景中  

![](1682178795783.png)

  
**出现的问题**  
标签已经添加到场景中了，但是出现了一个问题，我们的鼠标无法控制场景旋转缩放了，这是因为我们创建的 HTML 标签遮挡了画布的鼠标事件，我们只要将 css2dRenderer 样式中的 pointerEvents 设置为 none 就可以了

### 解决 HTML 标签遮挡画布问题

在 initCSS2DRenderer() 函数中添加下面一句代码

```
css2dRenderer.domElement.style.pointerEvents = 'none'
```

现在我们刷新浏览器，转动鼠标就不受影响了

## 实现鼠标选中模型弹出标签效果

要实现鼠标点选模型弹出标签效果，我们需要用到之前将过的光线投射 Raycaster, 对 Raycaster 不了解的小伙伴可以看我前面写的文章 [Threejs 入门之二十一：使用 Raycaster 实现物体与用户的交互](https://blog.csdn.net/w137160164/article/details/130141867)里面有详细的介绍，这里就不细讲了。

### 修改 initCSS2DRenderer() 函数

我们将 initCSS2DRenderer() 函数中获取 div 及创建 tag 对象的代码删除，将其放到 initListener() 函数中

```
initCSS2DRenderer() {
    // const div = document.querySelector('#label')
    // // HTML元素转换为threejs的css2d模型对象
    // const tag = new CSS2DObject(div)
    // tag.position.set(0,1.3,0);
    // this.scene.add(tag) 

    css2dRenderer = new CSS2DRenderer()
    css2dRenderer.setSize (window.innerWidth,window.innerHeight)
    css2dRenderer.domElement.style.position = 'absolute'
    css2dRenderer.domElement.style.top = '0px' 
    css2dRenderer.domElement.style.pointerEvents = 'none'
    document.body.appendChild(css2dRenderer.domElement)     
  }
```

### 创建 initListener() 函数

创建 initListener() 函数，在该函数里面实现鼠标点击事件的监听，并通过 Raycaster 与模型的焦点判断选中的物体

#### 获取 label 标签并创建 CSS2DObject 对象

```
const that = this
const div = document.querySelector('#label')
const txtName = document.querySelector('#txtName')
const txtMaterial = document.querySelector('#txtMaterial')
const txtColor = document.querySelector('#txtColor')
const tag = new CSS2DObject(div)
```

#### 创建鼠标点击的监听事件

```
this.renderer.domElement.addEventListener('click',function(event){
      const px = event.offsetX;
      const py = event.offsetY;    
      const x = (px / window.innerWidth) * 2 - 1;
      const y = -(py / window.innerHeight) * 2 + 1;

      const raycaster = new THREE.Raycaster();
      raycaster.setFromCamera(new THREE.Vector2(x, y), that.camera);
      const intersects = raycaster.intersectObjects(motorModel.children)
      if(intersects.length > 0) {
        intersects[0].object.add(tag)  
        selectObj = intersects[0].object  
      } else {
        if(selectObj) {//把原来选中模型对应的标签和发光描边隐藏
          selectObj.remove(tag) //从场景移除
        }
      }
    })
```

#### 获取选中模型的位置，并将其赋值给 tag 标签

通过`.geometry.attributes.position` 可以获取物体的位置信息，我们在选中物体的同时，获取该物体的位置信息，并赋值给 tag 标签  
在 if(intersects.length> 0) {} 代码段，添加如下代码

```
const pos = selectObj.geometry.attributes.position 
tag.position.set(pos.getX(0),pos.getY(0),pos.getZ(0)) 
tag.position.y = 1;
```

#### 根据点选的模型位置不同，显示不同的信息

我们可以在点选物体是，通过判断. name 属性来修改各个模型的信息，具体代码如下

```
if(selectObj.name === "网格457") {
          txtName.innerHTML='车身'
          txtMaterial.innerHTML = '合金'
          
        }  else if (selectObj.name === "网格457_2") {
          // 座位
          txtName.innerHTML='车座'
          txtMaterial.innerHTML = '真皮'
        } else if (selectObj.name === "网格457_4") {
          // 车把
          txtName.innerHTML='车把'
          txtMaterial.innerHTML = '真皮'
        } else if (selectObj.name === "网格457_3" || selectObj.name === "网格457_5" || selectObj.name === "网格457_6") {
          // 车架
          txtName.innerHTML='车架'
          txtMaterial.innerHTML = '铝合金'
        } else if (selectObj.name === "网格457_7") {
          // 轮胎
          txtName.innerHTML='轮胎'
          txtMaterial.innerHTML = '橡胶'
        } else {

        } 
        txtColor.innerHTML = selectObj.material.color.getHexString()
```

在鼠标点击模型意外的区域是，使用. remove() 方法将 tag 移除

#### initListener() 函数的完整代码如下

```
initListener() {
    const that = this
    const div = document.querySelector('#label')
    const txtName = document.querySelector('#txtName')
    const txtMaterial = document.querySelector('#txtMaterial')
    const txtColor = document.querySelector('#txtColor')
    // div.style.top='-240px'
    const tag = new CSS2DObject(div)
    this.renderer.domElement.addEventListener('click',function(event){
      const px = event.offsetX;
      const py = event.offsetY;
    
      const x = (px / window.innerWidth) * 2 - 1;
      const y = -(py / window.innerHeight) * 2 + 1;

      const raycaster = new THREE.Raycaster();
      raycaster.setFromCamera(new THREE.Vector2(x, y), that.camera);
      const intersects = raycaster.intersectObjects(motorModel.children)
      if(intersects.length > 0) {
        intersects[0].object.add(tag)  
        selectObj = intersects[0].object 
        const pos = selectObj.geometry.attributes.position 
        tag.position.set(pos.getX(0),pos.getY(0),pos.getZ(0)) 
        tag.position.y = 1; 
        
        
        if(selectObj.name === "网格457") {
          txtName.innerHTML='车身'
          txtMaterial.innerHTML = '合金'
          
        }  else if (selectObj.name === "网格457_2") {
          // 座位
          txtName.innerHTML='车座'
          txtMaterial.innerHTML = '真皮'
        } else if (selectObj.name === "网格457_4") {
          // 车把
          txtName.innerHTML='车把'
          txtMaterial.innerHTML = '真皮'
        } else if (selectObj.name === "网格457_3" || selectObj.name === "网格457_5" || selectObj.name === "网格457_6") {
          // 车架
          txtName.innerHTML='车架'
          txtMaterial.innerHTML = '铝合金'
        } else if (selectObj.name === "网格457_7") {
          // 轮胎
          txtName.innerHTML='轮胎'
          txtMaterial.innerHTML = '橡胶'
        } else {

        } 
        txtColor.innerHTML = selectObj.material.color.getHexString()
      } else {
        if(selectObj) { 
          selectObj.remove(tag) //从场景移除
        }
      }
    })
  }
```

