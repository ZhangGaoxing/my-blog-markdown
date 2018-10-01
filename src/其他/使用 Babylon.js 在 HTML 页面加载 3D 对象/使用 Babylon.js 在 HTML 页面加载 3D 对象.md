*五一 [Windwos Blogs](https://blogs.windows.com) 推了一篇博客， Babylon.js v3.2 发布了。因为一直有想要在自己博客上加载 3D 对象的冲动，这两天正好看到了，就动手研究研究。本人之前也并没有接触过 WebGL ，这方面算是知识盲区，需求完成之后感觉非常炫酷，顺手写篇博客记录下来。不得不说 3D 打印和 VR 慢慢的开始走进平时的生活了，技术的成熟与硬件成本的变低，结合内容跨平台共享与各种简单的 js 框架， WebGL 和 WebVR 很可能就是未来 Web 方向的主流技术。期待美好而炫酷的未来ing*

## Babylon.js 是什么
Babylon.js 是一个 JavaScript 开源框架，可以在浏览器或 Web 应用程序中简单便捷的构建 3D 游戏和 WebGL、WebVR 等 3D 体验。Babylon.js 非常强大，强大到可以去构建商业游戏。毕竟我才花了两天时间去了解它，只用来加载 3D 对象确实是大材小用了，文档和 GitHub 地址在下面。

Babylon GitHub : [https://github.com/BabylonJS/Babylon.js](https://github.com/BabylonJS/Babylon.js)

Babylon Document : [https://doc.babylonjs.com/](https://doc.babylonjs.com/)

## 基本代码
Babylon.js 并不是所有的 3D 对象都支持，支持的类型： **.glTF 、 .obj 、 .stl 、 .babylon** 。

这里以 STL 对象为例，首先需要引入两个 js 文件。一个是 Babylon.js ，另一个是 STL Loader， js 文件在 GitHub 中自行搜索下载引入。
```html
<script src="~/js/babylon.js"></script>
<script src="~/js/babylon.stlFileLoader.min.js"></script>
```

同时还需要一个 HTML5 的 canvas 标签作为 Babylon.js 的渲染容器
```html
<canvas id="renderCanvas" style="width:100%;height:100%;touch-action:none;"></canvas>
```

紧接着注册一个 DOM 事件，我们的渲染代码将在事件里完成，以确保执行渲染之前加载整个 DOM 。
```html
<script>
    window.addEventListener('DOMContentLoaded', function() {
        // TODO
    });
</script>
```

## 实现步骤
1. 获取渲染容器对象

    ```js
    var canvas = document.getElementById('renderCanvas');
    ```

2. 加载渲染引擎
    
    [Engine](https://doc.babylonjs.com/api/classes/babylon.engine) 类负责低级别的 API 接口。第一个参数为渲染容器对象，第二个参数是开启抗锯齿。

    ```js
    var engine = new BABYLON.Engine(canvas, true);
    ```

3. 加载场景

    一个基本场景（[Scene](https://doc.babylonjs.com/api/classes/babylon.scene)）里需要包括相机（[Cameras](https://doc.babylonjs.com/babylon101/cameras)）、光源（[Lights](https://doc.babylonjs.com/babylon101/lights)）、3D 对象。这里相机使用 ArcRotateCamera ，鼠标可以控制旋转和缩放。光源使用 HemisphericLight 半球光，用来模拟现实中的环境光。当然你也可以使用其他相机和光源，文档链接已给出。

    ```js
    // 基本的场景对象
    var scene = new BABYLON.Scene(engine);

    // 半球光对象，朝向天空
    var light = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(0, 1, 0), scene);

    // 弧度旋转相机，参数含义为：α角度、β角度、半径、目标方向、场景对象。
    // 下图非常详细的说明了各个参数的真实场景的含义
    var camera = new BABYLON.ArcRotateCamera('camera1', 0, 0, 10, new BABYLON.Vector3(40, 40, 40), scene);
    // 相机设置在原点位置
    camera.setTarget(BABYLON.Vector3.Zero());
    // 把相机附在渲染对象上
    camera.attachControl(canvas, false);
    
    // 把 STL 对象附加在现有的场景对象上
    // 可以从文件夹中选取对象，也可以给出一个 URL
    BABYLON.SceneLoader.Append("../", "Chariot_Red_f.stl", scene);
    ```

    ![](http://blogres.zhangyue.xin/18-5-7/25161397.jpg)
    <p style="text-align:center;margin-bottom:25px;color:gray"><small>Arc Rotate Camera 示意图</small></p>

    当然，上面的代码可以封装成一个方法。

4. 注册渲染循环
    
    这些代码非常重要，场景是需要循环渲染的。

    ```js
    engine.runRenderLoop(function () {
        scene.render();
    });
    ```
5. 实现容器自动缩放

    ```js
    window.addEventListener('resize', function () {
        engine.resize();
    });
    ```

## 完整代码与效果图

![](http://blogres.zhangyue.xin/18-5-7/18999516.jpg)
<p style="text-align:center;margin-bottom:25px;color:gray"><small>效果图</small></p>

```html
<canvas id="renderCanvas" style="width:100%;height:100%;touch-action:none;"></canvas>

<script src="~/js/babylon.js"></script>
<script src="~/js/babylon.stlFileLoader.min.js"></script>
<script>
    window.addEventListener('DOMContentLoaded', function () {
        var canvas = document.getElementById('renderCanvas');
        var engine = new BABYLON.Engine(canvas, true);

        var scene = new BABYLON.Scene(engine);
        var camera = new BABYLON.ArcRotateCamera('camera1', 0, 0, 10, new BABYLON.Vector3(40, 40, 40), scene);
        camera.setTarget(BABYLON.Vector3.Zero());
        camera.attachControl(canvas, false);
        var light = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(0, 1, 0), scene);

        BABYLON.SceneLoader.Append("", "@Model.PreviewModel", scene);

        engine.runRenderLoop(function () {
            scene.render();
        });

        window.addEventListener('resize', function () {
            engine.resize();
        });
    });
</script>
```