## 项目开发中遇到的一些问题及重点功能

### 灯光

```typescript
//设置作用物体
light.includedOnlyMeshes;
//设置排除物体
light.excludedMeshes;
```

### 相机

```typescript
//相机运动触发的事件
camera.onViewMatrixChangedObservable;

//获取摄像机能看到的mesh
scene.activeCamera.getActiveMeshes();
scene.getActiveMeshes();

//判断mesh是否在摄像机范围内
scene.isActiveMesh(scene.meshes[2]);
scene.activeCamera.isActiveMesh(scene.meshes[2]);
```

```typescript
//漫游相机定制跳跃转向碰撞等功能

//有两种方法可以根据世界矢量进行碰撞位移
//1.
camera._collideWithWorld(camera.getFrontPosition(10).subtract(camera.position));
//2.
camera.cameraDirection.addInPlace(camera._transformedDirection);

//如跳跃，则相对世界向上
scene.activeCamera.cameraDirection.addInPlace(new BABYLON.Vector3(0, 4, 0));
//重力回到地面
camera._needMoveForGravity = true;
//定制位移方向
var speedXX = camera._computeLocalCameraSpeed() * (左右惯性因子 * panel);
var speed = camera._computeLocalCameraSpeed() * (前后惯性因子 * panel);
camera._localDirection.copyFromFloats(speedX, 0, speedZ);
camera.getViewMatrix().invertToRef(camera._cameraTransformMatrix);
BABYLON.Vector3.TransformNormalToRef(
  camera._localDirection,
  camera._cameraTransformMatrix,
  camera._transformedDirection
);
camera.cameraDirection.addInPlace(camera._transformedDirection);

//定制旋转方向
camera.rotation.x = beta惯性因子 * panel;
camera.rotation.y = alpha惯性因子 * panel;
```

### Mesh

创建一个 SubMesh

```typescript
new SubMesh（materialIndex，verticesStart，verticesCount，indexStart，indexCount，mesh，renderingMesh，createBoundingBox）
//materialIndex    数    要使用的材料的索引（此索引用于在多材质的subMaterials集合内找到正确的材质）
//verticesStart    数
//verticesCount    数    使用的顶点数
//indexStart    任何要使用的第一个indice的索引
//indexCount       数    指数计数
//Mesh  @param mesh   网格    摘要
//renderingMesh    网格    如果已定义，则用于代替网格参数（可选的）
```

合并网格

```typescript
var boxes_merged = BABYLON.Mesh.MergeMeshes(scene.getMeshesByTags("box"));
boxes_merged.position.x = 0;
```

### 材质

```typescript
material.maxSimultaneousLights; //最多可接受灯光个数，默认4个
material.useLogarithmicDepth;
material.needDepthPrePass; //深度判断
material.transparencyMode = 2;
```

### dynamicTexture

```typescript
var dynamicTexture = new BABYLON.DynamicTexture(
  "DynamicTexture",
  { width: 50, height: 50 },
  scene,
  true
);
dynamicTexture.hasAlpha = true;
dynamicTexture.drawText(
  text,
  10,
  50,
  "bold 100px Arial",
  color,
  "transparent",
  true,
  true
);

var plane = new BABYLON.Mesh.CreatePlane("TextPlane", size, scene, true);
plane.material = new BABYLON.StandardMaterial("TextPlaneMaterial", scene);
plane.material.backFaceCulling = false;
plane.material.specularColor = new BABYLON.Color3(0, 0, 0);
plane.material.diffuseTexture = dynamicTexture;
```

### 镜面反射材质

```typescript
Material.diffuseColor = new BABYLON.Color3(0.4, 0.4, 0.4);
Material.reflectionTexture = new BABYLON.MirrorTexture(
  "mirror",
  1024,
  scene,
  true
); //Create a mirror texture
Material.reflectionTexture.mirrorPlane = new BABYLON.Plane(0, -1.0, 0, -10.0);
Material.reflectionTexture.renderList = [ground, skybox];
Material.reflectionTexture.level = 0.6; //Select the level (0.0 > 1.0) of the reflection
```

### 折射

平面

```typescript
var refractionTexture = new BABYLON.RefractionTexture("th", 1024, scene);

refractionTexture.renderList.push(yellowSphere);
refractionTexture.renderList.push(greenSphere);
refractionTexture.renderList.push(ground);
refractionTexture.refractionPlane = new BABYLON.Plane(0, 0, -1, 0);
refractionTexture.depth = 2.0;
```

非平面

```typescript
var probe = new BABYLON.ReflectionProbe("main", 512, scene);

probe.renderList.push(yellowSphere);
probe.renderList.push(greenSphere);
probe.renderList.push(blueSphere);
probe.renderList.push(mirror);

mainMaterial.refractionTexture = probe.cubeTexture;
```

得要有 refraction
materialSphere3.indexOfRefraction = 1; //折射率越大， 镜片越薄，看起来越折射

### 探针

```typescript
var box = BABYLON.Mesh.CreateBox("box", 5000, scene);
//直接贴反射贴图
//material.reflectionTexture = new BABYLON.CubeTexture('img/bg/4', scene);
//box.material = new BABYLON.StandardMaterial("boxMaterial", scene);
//box.material.reflectionTexture = new BABYLON.CubeTexture('img/bg/4', scene);
//box.material.reflectionTexture.coordinatesMode = BABYLON.Texture.SKYBOX_MODE;
//box.material.backFaceCulling = false;
//box.rotation.z = Math.PI / 2;

var probe = new BABYLON.ReflectionProbe("main", 512, scene);
probe.renderList.push(yellowSphere);
probe.renderList.push(greenSphere);
probe.renderList.push(blueSphere);
probe.renderList.push(mirror);
mainMaterial.diffuseColor = new BABYLON.Color3(1, 0.5, 0.5);
mainMaterial.reflectionTexture = probe.cubeTexture;
mainMaterial.reflectionFresnelParameters = new BABYLON.FresnelParameters();
mainMaterial.reflectionFresnelParameters.bias = 0.02;
```

### 动画

- createAndStartAnimation 不会打断其他动画，但是 beginAnimation 会打断所有 animation 包括 createAndStartAnimation
- beginAnimation 会打断 createAndStartAnimation,createAndStartAnimation 不会打断 beginAnimation
- 创建一个动画以将属性的当前值插入给定目标

```typescript
new InterpolateValueAction(
  triggerOptions,
  target,
  propertyPath,
  value,
  duration,
  condition,
  stopOtherAnimations,
  onInterpolationDone
);
// triggerOptions  触发器选项
// target          目标
// propertyPath    目标的属性
// value           目标的值
// duration        持续时间
// condition       条件
// stopOtherAnimations   停止其他动画  boolean (可选的)
```

- 此操作是一个容器。 您可以使用它在同一触发器上同时执行多个操作。 children 属性必须是一个操作数组

```typescript
new CombineAction(triggerOptions, children, condition) ：
// triggerOptions：触发器选项
// children ：  The childrens actions
// condition ： 执行动作的条件（可选）
```

- 创建新的设置值操作

```typescript
new SetValueAction(triggerOptions, target, propertyPath, value, condition);
//triggerOptions  触发器选项
//target         行动目标
//propertyPath   行动目标
//value          动作值
```

```typescript
new DoNothingAction(triggerOptions, condition); //不作为
new SetStateAction(triggerOptions, target, value, condition);
```

### 粒子

```typescript
2、子粒子，放烟花
particleSystem.subEmitters = [particleSystem,..]

3、members
particleSystem.disposeOnStop=true;
```

### 获取轴心

```typescript
//如果之前有freeze矩阵，则需要先重新计算
mesh.computeWorldMatrix(true);
mesh.getAbsolutePivotPoint();
```

### 设置本身轴偏差

```typescript
mesh.setPivotPoint(vec3);
```

### 阴影

```typescript
var shadowGenerator=new BABYLON.ShadowGenerator(阴影贴图的大小：1024,light);
//产生阴影的物体列表
shadowGenerator.getShadowMap().renderList.push(box);

//软化阴影
//泊松抽样
shadowGenerator.usePoissonSampling = true;
//指数阴影贴图
shadowGenerator.useExponentialShadowMap = true;
shadowGenerator.depthScale=50
//模糊指数阴影图
shadowGenerator.useBlurExponentialShadowMap = true;
//模糊质量
shadowGenerator.useKernelBlur = true;
shadowGenerator.blurScale = 1;
shadowGenerator.blurKernel = 60;
//接收阴影
plane.receiveShadows=true;
//阴影计算默认1秒60次，静态阴影改为1次就可以了;
shadowGenerator.getShadowMap().refreshRate =BABYLON.RenderTargetTexture.REFRESHRATE_RENDER_ONCE //（0）We need to compute it just once
//让灯不要重新计算阴影位置
light.autoUpdateExtends = false;
//取消泊松亮斑
shadowGenerator.bias=0.001;
//webGL2.0
usePercentageCloserFiltering=true;
```

### 世界坐标系 与 本地坐标系

1. lookAt(vector)是以 z 轴指向 vector 这个点的位置，使用这个值后，会清零 rotation,并将本地轴的信息存放在 rotationQuaternion 中。
2. 本地轴默认为世界坐标系(左手坐标系).

```typescript
* translate(vector,distance,BABYLON.Space.LOCAL)
* rotate(vector,distance,BABYLON.Space.LOCAL)

```

5. 位移(position,translate)都是沿着世界坐标系或者父元素的本地坐标系进行位移，若要沿着自己的本地轴进行位移，则

```typescript
* locallyTranslate(vector)
```

6. 如果设置了 rotationQuaternion,那么本地轴存放在 rotationQuaternion 中（如果 3DSMAX 中导出轴不是世界坐标系，那么本地轴也存放在这里

### 优化

```typescript
scene.freezeMaterials();
```

2. 默认情况下，Babylon.js 使用索引的网格，其中顶点可以由面部重用。当顶点重用率低时，如果顶点结构相当简单（就像一个位置和一个正常的），那么你可能需要展开顶点并停止使用索引

```typescript
mesh.convertToUnIndexedMesh();
```

5. 当 mesh 的顶点数量太多，检测碰撞时候最好开启子网格，子网格进行检测

```typescript
mesh.subdivide(x);
mesh.createOrUpdateSubmeshesOctree(capacity, maxDepth);
mesh.useOctreeForCollisions;
mesh.useOctreeForPicking;
mesh.useOctreeForRenderingSelection;
//var octree=scene.createOrUpdateSelectionOctree();
//for(var i=0;i<scene.meshes.length;i++){
// octree.dynamicContent.push(scene.meshes[i])
//}
```

8.简化网格，生成不同 faces 的 LOD

```typescript
simplify(settings, parallelProcessing, simplificationType, successCallback) → void
//scene.meshes[0].simplify([{ quality: 0.1, distance: 10000 }],
// true, BABYLON.SimplificationType.QUADRATIC, function() {
// alert("LOD finisehd, let's have a beer!");
// });
```

### toDataURL

```
//先进行设置
var engine = new BABYLON.Engine(canvas, antialiasing, {
preserveDrawingBuffer: true,
}, false);
canvas.toDateURL();
```
