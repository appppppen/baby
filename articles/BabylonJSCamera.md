Babylonjs基础入门 相机
## 开始我的巴比伦之旅
在Babylon.js中，最常用的两个相机有可能是：
1.用于第一人称运行的通用相机
2.用于查看物体的弧形旋转相机
虽然随着WebVR的出现，这可能会改变。
对于创建的相机对象，必须要将相机附加到canvas画布上面，才可以被用户使用。
```
camera.attachControl(canvas, true);
```
第二个参数是可选的，默认值为false。如果值为false，Babylon.js会阻止canvas的默认事件。如果设置为true，canvas将会触发默认事件。
ps：
3. 游戏手柄可以用于操作控制器（作为游戏引擎这个功能是必须的）。
4. 触摸控制需要任意一种PEP（Pointer Events Polyfill-指针事件的Polyfill）或者hand.js（微软出品的一个Polyfill）。在这里科普一下，Polyfill相当于一个补丁，用于在低版本浏览器使用高版本的原生js接口的兼容性插件。
### 通用相机 Universal Camera
首次出现于Babylon.js的2.3版本，支持由键盘，鼠标，触摸屏或者游戏手柄控制器操控，可以自动判断输入设备。它扩展并取代了仍然可以使用的自由相机（Free Camera），触摸相机（Touch Camera）和游戏手柄相机（Gamepad Camera）。
现在Universal Camera为Babylon.js的默认相机，如果你想在场景中使用类似于FPS操控的空间，这种相机是你的最佳选择。Babylon.js的很多demo也是基于此相机实现。这样使用Xbox手柄你仍然可以浏览操控大部分案例。
Universal Camera的默认操作方式是：

键盘 - 左右箭头左右移动相机，上下箭头前后移动相机。
鼠标 - 将以相机为中心点旋转相机。
触摸屏幕 - 向左向右滑动可以左右移动相机，向上向下滑动可以向前向后移动相机。
游戏操作手柄 - 请查看相关链接
注意：操作之前你需要点击渲染区域使Babylon.js获得焦点。

如何使用：
```
// 实例化相机，并传入相关值: 名称，位置，场景对象
    var camera = new BABYLON.UniversalCamera("UniversalCamera", new BABYLON.Vector3(0, 0, -10), scene);

// 使用setTarget方法设置相机交点，并传入一个场景的位置点
    camera.setTarget(BABYLON.Vector3.Zero());

// 设置相机将事件绑定到画布上面
    camera.attachControl(canvas, true);
```
### 弧形旋转相机 Arc Rotate Camera
此相机始终会朝向给定目标的位置点，并且可以围绕当前目标旋转，以目标作为旋转中心。它可以通过光标和鼠标控制，也可以用触摸事件控制。
可以把弧形旋转相机想象成一个绕其目标位置轨道运行的轨道，或者更具想象力地将其作为绕地球轨道运行的间谍卫星。它相对于目标（地球）的位置可以通过三个参数设置，α-alpha（弧度）表示经度旋转，β-beta（弧度）表示纬度旋转， radius半径表示距目标位置的距离。
![](2019022621510211.jpg)
由于技术原因，将beta的值设置为0或者Math.PI可能会导致问题，在这种情况下，beta会偏移0.1弧度（约0.6度）。
alpha和beta均以顺时针方向增加。
摄像机的位置也可以通过矢量设置，该矢量将覆盖alpha，beta和radius的当前值。这比计算出所需的角度容易得多。
无论是使用键盘，鼠标还是触摸滑动左右方向都会该表alpha和向上方向改变beta。
构造弧形旋转相机
```
// 创建相机的配置项：name, alpha, beta, radius, target position, scene
var camera = new BABYLON.ArcRotateCamera("Camera", 0, 0, 10, new BABYLON.Vector3(0, 0, 0), scene);

// 通过设置相机位置来覆盖 alpha, beta, radius
camera.setPosition(new BABYLON.Vector3(0, 0, 20));

// 将相机绑定到画布上面
camera.attachControl(canvas, true);
```
使用键盘CTRL键+鼠标左键（默认操作方式）也可以实现弧形旋转相机的平移。如果只允许鼠标右键平移的话，我们可以通过设置attachControl的第三个值为false实现。

```
camera.attachControl(canvas, true, false);
```
如果需要，你还可以直接将相机平移完全禁掉：

```
scene.activeCamera.panningSensibility = 0;
```
### 跟随相机 FollowCamera
跟随相机可以通过按照我们的设置。设置一个模型网格作为它的焦点，从所处的位置到移动来观察焦点模型的。当模型移动时，相机也会跟随移动。
在创建跟随相机时，我们需要设置它的三个参数：

1.相机与目标模型的距离 - camera.radius
2.相机高度与目标模型的高度差 - camera.heightOffset
3.在x y平面上围绕目标的角度。
相机移动到目标位置是通过它的加速度（camera.cameraAcceleration），并可以限制最大速度（camera.maxCameraSpeed）。
创建跟随相机

```
// 创建相机配置项: name, position, scene
var camera = new BABYLON.FollowCamera("FollowCam", new BABYLON.Vector3(0, 10, -10), scene);

// 在全局坐标系下相机距离场景的距离
camera.radius = 30;

//在局部坐标下相机的高度高于目标的高度的值
camera.heightOffset = 10;

// 相机在x y平面内围绕目标的局部原点旋转值
camera.rotationOffset = 0;

// 摄像机从当前位置移动到目标位置的加速度
camera.cameraAcceleration = 0.005

// 最大加速值
camera.maxCameraSpeed = 10

// 相机绑定到画布上面
camera.attachControl(canvas, true);

// 注意：在目标创建后要设置相机的目标，注意Babylon.js在2.5版本以后变更设置。
// 在此处设置相机的目标网格
camera.target = targetMesh;   // 2.4版本以及更低版本
camera.lockedTarget = targetMesh; //2.5以后的版本

```




### 立体相机 AnaglyphCameras
立体相机扩展了通用相机和弧形旋转相机，可用于红蓝3d眼镜查看立体效果。它是通过后处理实现的效果。

####  创建立体通用相机 AnaglyphUniversalCamera
```
// 配置项：名称，相机位置，左右眼偏移量，场景对象
var camera = new BABYLON.AnaglyphUniversalCamera("af_cam", new BABYLON.Vector3(0, 1, -15), 0.033, scene);

```


#### 创建立体弧形旋转相机 AnaglyphArcRotateCamera
```
// 配置项：名称，初始经度，初始纬度，与焦点距离，焦点位置，眼距，场景对象
var camera = new BABYLON.AnaglyphArcRotateCamera("aar_cam", -Math.PI/2, Math.PI/4, 20, new BABYLON.Vector3.Zero(), 0.033, scene);

```
eyeSpace配置项是设置左眼视图和右眼视图之间的偏移量。带上3d眼镜后，你可能需要尝试使用这个值。
欲了解更多请自行搜索相关内容。

### 陀螺仪相机 DeviceOrientationCamera
这个相机会对于设备陀螺仪做出反应的相机，例如现在的手机都会有相关的功能。
创建陀螺仪相机：

```
//配置项：相机名称，相机位置，场景对象
var camera = new BABYLON.DeviceOrientationCamera("DevOr_camera", new BABYLON.Vector3(0, 0, 0), scene);

// 设置相机朝向一个特定的位置
camera.setTarget(new BABYLON.Vector3(0, 0, -10));

// 设置相机对移动和旋转的灵敏度
camera.angularSensibility = 10;
camera.moveSensibility = 10;

// 将相机事件绑定到canvas上面
camera.attachControl(canvas, true);
```


### 虚拟操纵杆相机 VirtualJoysticksCamera

虚拟操纵杆相机会在界面上生成虚拟的操作图形，用于控制相机或其他场景的物体。
需要
第三方的文件hand.js
完整案例
这是一个完整的示例，可以加载Espilit示例并将默认摄像头切换都成虚拟操纵杆相机：

```
document.addEventListener("DOMContentLoaded", startGame, false);
    function startGame() {
        if (BABYLON.Engine.isSupported()) {
            var canvas = document.getElementById("renderCanvas");
            var engine = new BABYLON.Engine(canvas, true);

            BABYLON.SceneLoader.Load("Espilit/", "Espilit.babylon", engine, function (newScene) {

                var VJC = new BABYLON.VirtualJoysticksCamera("VJC", newScene.activeCamera.position, newScene);
                VJC.rotation = newScene.activeCamera.rotation;
                VJC.checkCollisions = newScene.activeCamera.checkCollisions;
                VJC.applyGravity = newScene.activeCamera.applyGravity;

                // Wait for textures and shaders to be ready
                newScene.executeWhenReady(function () {
                    newScene.activeCamera = VJC;
                    // Attach camera to canvas inputs
                    newScene.activeCamera.attachControl(canvas);
                    // Once the scene is loaded, just register a render loop to render it
                    engine.runRenderLoop(function () {
                        newScene.render();
                    })
                })
            }, function (progress) {
                // To do: give progress feedback to user.
            })
        }
    }
```

如果切换成别的相机，请不要忘记先调用dispose()方法。实际上，虚拟操纵杆相机在3d WebGL画布上创建了一个2d画布，用于绘制带有青色和黄色的操纵杆。如果你忘记调用dispose()函数，3d画布将保留，并将继续触发相关事件。

### VR 陀螺仪相机

对于拥有VR头显设备的开发者。
### 创建VR陀螺仪自由相机 VRDeviceOrientationFreeCamera
```
// 配置项：相机名称，相机位置，场景对象，补偿失真，vrCameraMetrics
var camera = new BABYLON.VRDeviceOrientationFreeCamera ("Camera", new BABYLON.Vector3 (-6.7, 1.2, -1.3), scene);
```
###  创建VR陀螺仪弧形旋转相机 VRDeviceOrientationArcRotateCamera
```
//配置项: 相机名称，经度，纬度，距离，目标位置，场景对象, 补偿失真, vrCameraMetrics
var camera = new BABYLON.VRDeviceOrientationArcRotateCamera ("Camera", Math.PI/2, Math.PI/4, 25, new BABYLON.Vector3 (0, 0, 0), scene);
```



### 创建VR陀螺仪游戏手柄相机 VRDeviceOrientationGamepadCamera
```
//配置项: 相机名称, 相机位置, 场景对象, 补偿失真, vrCameraMetrics
var camera = new BABYLON.VRDeviceOrientationGamepadCamera("Camera", new BABYLON.Vector3 (-10, 5, 14));
```

### WebVR自由相机 WebVRFreeCamera

```
// 配置项: 相机名称, 相机位置, 场景对象, webVROptions
var camera = new BABYLON.WebVRFreeCamera("WVR", new BABYLON.Vector3(0, 1, -15), scene);

```
这个相机需要用一篇文章讲解，在后面再仔细介绍吧。
### 飞行相机 FlyCamera
```
// 配置项：相机名称，相机位置，场景对象
var camera = new BABYLON.FlyCamera("FlyCamera", new BABYLON.Vector3(0, 5, -10), scene);

// 类似飞机的旋转，具有更快的滚转修正和倾斜转弯。
// 默认值为100。数字越大，校正速度越慢。
camera.rollCorrect = 10; //滚动修正
// 默认值为false
camera.bankedTurn = true; //倾斜旋转
//默认值九十度，以弧度赋值，配置相机滚动多远
camera.bankedTurnLimit = Math.PI / 2;
// 设置偏航（转弯）如何影响滚动（转弯）。
// 小于1将减少滚动，并且超过1将增加滚动。
camera.bankedTurnMultiplier = 1;

// 相机事件绑定到画布
camera.attachControl(canvas, true);
```