## 如何优化你的场景

1. HowToOptimizeYourScene 如何优化你的场景
2. UseTransformNodeinsteadofAbstractMeshoremptymeshes 使用变换节点代替抽象网格或者空网格
3. Changingpermeshcullingstrategy 改变每个网格的剔除策略
4. ReducingShadersOverhead 降低着色器开销
5. ReducingWorldMatricesComputation 减少世界矩阵计算
6. Freezingtheactivemeshes 冻结活动的网格
7. Reducingdrawcalls 减少绘制调用
8. Reducingcallstogl.clear()减少 gl.clear()的调用
9. Usingdepthpre-pass 使用深度预加载
10. Usingunindexedmeshes 使用未索引的网格
11. TurningAdaptToDeviceRatioOff 关闭适应设备比例（ratio 是比例，rate 是速率）
12. Blockingthedirtymechanism 停止不好的机制
13. UsingAnimationRatio 使用动画比率
14. HandlingWebGLcontextlost 处理 WebGL 上下文丢失
15. Scenewithlargenumberofmeshes 处理包含大量网格的场景
16. Instrumentation 仪表
17. EngineInstrumentation 引擎仪表
18. SceneInstrumentation 场景仪表

### 使用变换节点代替抽象网格或者空网格

如果你需要节点容器或者变换节点，不要使用网格，而是使用变换节点来替代。只在有关联的内容需要绘制时使用网格。

网格需要经过一个验证流程，在这个流程中相机将检查这些网格是否在视截椎体内部。这是一个性能消耗较大的流程，所以在可能时通过使用变换节点减少这种计算的数量是一个好的实践。

（使用后 1366\*720 分辨率 fps 似从 20 提升到 22~24）

### 改变每个网格的剔除策略

从 Babylon.jsv3.3 开始，你可以设定一个策略来剔除一个特定的网格，通过设置

`mesh.cullingStrategy`

你可以将它设置为：

`BABYLON.AbstractMesh.CULLINGSTRATEGY_STANDARD`

这是默认值，它将结合使用边界球剔除和边界盒剔除，然后是截锥体剔除。

`BABYLON.AbstractMesh.CULLINGSTRATEGY_BOUNDINGSPHERE_ONLY`

这个策略将使用边界球剔除，然后是截锥体剔除。这种策略比基础的版本更加快速，但是可能引起更多的网格被发送到 GPU。如果你的 CPU 资源有限，这种方法会很有用。（把更多工作分给显卡做，但是我的 CPU 和 GPU 都不富裕啊）

### 降低着色器开销

Babylon.js 使用一种先进的并且自动的着色器引擎。这个系统将根据材质的选项保持着色器的更新。如果你使用的是静态材质（比如一个不变的材质）那么你可以通过以下的代码让 Babylon.js 知道这一点：

`material.freeze();`

一旦被冻结，这个着色器将保持不变，即使你改变这个材质的属性（纹理是否属于着色器？更换其他材质会不会影响？网格的 isvisble 会不会影响？）。你必须解冻它才能更新内部的着色器：

`material.unfreeze();` （建议在完成静态材质的所有设定之后使用，不影响更换其他材质，提升到 26~27）

### 减少世界矩阵计算

每个网格都有一个世界矩阵用来确定它的位置、姿态、缩放。这个矩阵在每一帧中被计算。你可以通过冻结这个矩阵来改进性能。这样任何传递（通过父元素继承？）的位置、姿态、缩放改变都将被忽略：

`mesh.freezeWorldMatrix();`

你可以使用如下代码解冻一个网格：

`mesh.unfreezeWorldMatrix();`（适合用在父元素固定不变的情况下，或者提前预知哪一帧会发生改变，然后只在这一帧解冻！！！！天空盒如果设置了 infiniteDistance 不应冻结世界矩阵，因为天空盒需一直移动，冻结地面之后静止帧数提升至 29~30）

### 冻结活动的网格

如果你的 CPU 有限，你可以考虑保持“活动网格列表”不变，这样可以节省 CPU 用来判断网格是否活动的时间：

`scene.freezeActiveMeshes();`

你可以使用如下代码解冻活动网格列表

`scene.unfreezeActiveMeshes();`

`mesh.alwaysSelectAsActiveMesh=true.`
注意你可以在冻结活动网格列表前，强制一个网格进入活动网格列表，通过

`mesh.alwaysSelectAsActiveMesh=true.`
（比如 isVisible==false 就是非活动网格？）

### 减少绘制调用

请尽量快的用 `instances` 代替通过单独绘制调用绘制

`mesh.clone("newName")`
如果实例对象必须共用同一个材质对象是一个问题，你可以考虑通过语句 mesh.clone("newName")用克隆方法，克隆方法产生的 mesh 共用同一个多边形对象
（instances 可以以一个网格为基础，复制出许多形状和材质相同的实例，它们的材质和多边形对象都是共用的）

### 减少 gl.clear()的调用

默认情况下，Babylon.js 会在渲染场景之前自动清空颜色、深度和模板缓存。它同样会在切换到新相机之后以及渲染一个新的渲染组之前清空深度和模板缓存。对于填充率较低的系统，这些清空操作可以被快速的汇总并且对性能有巨大的影响。

如果你的场景按这种方式设定：视点一直百分之百的被不透明多边形填满（比如你一直处在一个天空盒内部），你可以用以下方法禁用默认的场景清理：

`scene.autoClear=false;`// 颜色缓存

`scene.autoClearDepthAndStencil=false;`//显然是深度和模板缓存

如果你确定在某个渲染组中的多边形将一直位于其他组的多边形之前，你可以禁用这个组的缓存清理，通过以下方法：

`scene.setRenderingAutoClearDepthStencil(renderingGroupIdx,autoClear,depth,stencil);`

autoClear:
为真则启动自动清理。否则，覆盖深度和模板

depth:
默认为真启动深度缓存清理

stencil:
默认为真启动模板缓存清理

尽管大胆测试这些设置。如果你在你的应用中看到任何污渍你将知道这个设置是否适合。

（在场景层面执行两个方法禁用了三种缓存，暂时未发现渲染异常，静止 fps 增加到 37~38，但是在场景中漫游一段时间后 fps 又降低了，但是如果要在场景中使用多重半透明材质，还需要更谨慎的测试）

### 使用深度预传递

在处理复杂的场景时，使用深度预传递是很有用的。这项技术将只在深度缓存中（意思是一个不显示的网格，但能影响其他网格的显示？）渲染指定的网格，来影响早期的深度测试剔除。在有网格应用了高级着色器的场景中这会很有用。要启用一个网格的深度预传递，只需要调用：

`mesh.material.needDepthPrePass=true.`

### 使用非索引的网格

默认情况下 Babylon.js 使用索引的网格，这时顶点可以通过面来引用。当顶点引用较少或者顶点结构相当简单时（比如只有一个位置和一个法线），那么你可能想要展开你的顶点并且停止使用索引：

`mesh.convertToUnIndexedMesh();`

例如这种设定对于立方体非常好用，这时发送 32 个位置以代替 24 个位置和 32 个索引是更有效率的。

（其实是换成了 OpenGL 更基础的一种顶点绘制方式，给没有精确点击的网格设置这种绘制方式？也许能加快场景加载速度？但是 fps 反而降低了，改回索引方式后 fps 恢复，估计可能是非索引方式在检测 pick 时消耗更大）

### 关闭适应设备比例

默认情况下，Babylon.js 会适应设备的比例来产生尽可能高的图像质量，即使是在高分辨率设备上。
后果是在低端设备上这将造成很大的消耗。你可以通过引擎的构造方法的第四个参数关闭它：

```typescript
var engine = new BABYLON.Engine(canvas, antialiasing, null, false);
```

同样是在这个构造函数中，你也可能想通过第二个参数关闭抗锯齿支持。

### 停止不好的机制

默认情况下，场景将在你改变任何潜在的可能影响材质的属性时更新所有的材质（透明度、纹理更新等等）。要做到这一点场景需要遍历所有的材质然后将它们标志为脏的。在你有许多材质时这可能成为一个潜在的瓶颈。

要阻止这种自动的更新，你可以执行：

`scene.blockMaterialDirtyMechanism=true;`

不要忘了将它恢复为 false，在你需要批量更改材质时。

（使用后帧数似乎略有提高，可能和前面的材质冻结存在部分功能重合）

### 使用动画比率

Babylon.js 根据当前帧率计算速度。

在低端设备上动画或者相机移动可能和高端设备不同。要补偿这一点你可以使用：

`scene.getAnimationRatio();`

在低帧率的情况下返回值更高

### 处理 WebGL 上下文丢失

从 3.1 版本开始 Babylon.js 可以处理 WebGL 上下文丢失事件。这个事件由浏览器在 CPU 需要从你的代码上移开时触发。例如这种情况会在再混合场景中使用 WebVR 时发生（具有多个 CPU）。在这种情况下，Babylon.js 不得不重新建立所有的底层资源（包括纹理、着色器、程序、缓存等等）。这个流程是完全透明的并且由 Babylon.js 的底层进行。

作为一个开发者你不需要了解这个机制。但是，要支持这个机制，Babylon.js 可能需要一些额外的内存来追踪资源的建立。如果你不需要支持 WebGL 上下文丢失事件，你可以通过将引擎的 doNotHandleContextLost 选项设为 true 来关闭这个追踪。

`engine.onContextLostObservableandengine.`

如果你的已经被建立的资源需要被重建（比如顶点缓存或者索引缓存），你可以使用

`engine.onContextLostObservable` 和 `engine.onContextRestoredObservable` 两个标志来追踪上下文丢失和上下文重建事件
（也就是说显卡要去做别的用，或者换成另一块显卡？）

### 具有大量网格的场景

如果你的场景中有大量的网格，并且需要降低向这个场景中添加或者移除网格的时间消耗，场景工作函数的一些选项可能有帮助

将 useGeometryIdsMap 选项设为 true 将加速场景中几何体的添加和移除

将 useMaterialMeshMap 选项设为 true 将加速材质的回收，通过减少寻找绑定的网格所需的时间做到这一点。

将 useClonedMeshMap 选项设为 true 将加速网格的回收，通过减少相关的被克隆网格查找时间来实现这一点。

对于开启的每一个选项，Babylon.js 都会需要额外的内存。（上面三种的原理和数据库通过建索引加快查询速度的原理是一样的）

同样，如果你要在一行中回收大量的网格，你可以节省一些不必要的计算，通过在回收这些网格前将场景的

blockfreeActiveMeshesAndRenderingGroups 属性（成块释放活动网格和渲染组？）设为 true，然后在回收完毕后将它恢复为 false，就像这样：

在这里一次性回收所有的网格

`scene.blockfreeActiveMeshesAndRenderingGroups=false;`

### 指示器

在你想要优化一个场景时，指示器是一种关键的工具。它将帮助你指出哪里是性能瓶颈，于是你可以优化需要被优化的地方。

#### 引擎指示器

引擎指示器类允许你获取如下计数器：

GPU 帧时间计数器：GPU 渲染一帧所花费的时间（以纳秒表示）。需要把

`instrumentation.captureGPUFrameTime 属性设为 true。`

着色器计算时间计数器：CPU 计算所有着色器的时间（以毫秒表示）。必须把

`instrumentation.captureShaderCompilationTime 属性设为 true。`

这是一个如何使用引擎指示器的例子：https://www.babylonjs-playground.com/#HH8T00#1-

```typescript
var createScene = function () {
  varscene = newBABYLON.Scene(engine);
  varcamera = newBABYLON.ArcRotateCamera(
    "Camera",
    0,
    0,
    10,
    BABYLON.Vector3.Zero(),
    scene
  );
  varmaterial = newBABYLON.StandardMaterial("kosh", scene);
  varsphere1 = BABYLON.Mesh.CreateSphere("Sphere1", 32, 5, scene);
  varlight = newBABYLON.PointLight(
    "Omni0",
    newBABYLON.Vector3(-17.6, 18.8, -49.9),
    scene
  );

  camera.setPosition(newBABYLON.Vector3(-15, 3, 0));
  camera.attachControl(canvas, true);

  //Sphere1material
  material.refractionTexture = newBABYLON.CubeTexture(
    "/textures/TropicalSunnyDay",
    scene
  );
  material.reflectionTexture = newBABYLON.CubeTexture(
    "/textures/TropicalSunnyDay",
    scene
  );
  material.diffuseColor = newBABYLON.Color3(0, 0, 0);
  material.invertRefractionY = false;
  material.indexOfRefraction = 0.98;
  material.specularPower = 128;
  sphere1.material = material;

  material.refractionFresnelParameters = new BABYLON.FresnelParameters();
  material.refractionFresnelParameters.power = 2;
  material.reflectionFresnelParameters = new BABYLON.FresnelParameters();
  material.reflectionFresnelParameters.power = 2;
  material.reflectionFresnelParameters.leftColor = BABYLON.Color3.Black();
  material.reflectionFresnelParameters.rightColor = BABYLON.Color3.White();

  //Skybox
  var skybox = BABYLON.Mesh.CreateBox("skyBox", 100.0, scene);
  var skyboxMaterial = newBABYLON.StandardMaterial("skyBox", scene);
  skyboxMaterial.backFaceCulling = false;
  skyboxMaterial.reflectionTexture = newBABYLON.CubeTexture(
    "/textures/TropicalSunnyDay",
    scene
  );
  skyboxMaterial.reflectionTexture.coordinatesMode =
    BABYLON.Texture.SKYBOX_MODE;
  skyboxMaterial.diffuseColor = newBABYLON.Color3(0, 0, 0);
  skyboxMaterial.specularColor = newBABYLON.Color3(0, 0, 0);
  skyboxMaterial.disableLighting = true;
  skybox.material = skyboxMaterial;

  varcolorGrading = newBABYLON.ColorGradingTexture(
    "/textures/LateSunset.3dl",
    scene
  );
  skyboxMaterial.cameraColorGradingTexture = colorGrading;
  material.cameraColorGradingTexture = colorGrading;
  skyboxMaterial.cameraColorGradingEnabled = true;
  material.cameraColorGradingEnabled = true;

  //Instrumentation初始化指示器
  varinstrumentation = newBABYLON.EngineInstrumentation(engine);
  instrumentation.captureGPUFrameTime = true;
  instrumentation.captureShaderCompilationTime = true;

  //GUI
  varadvancedTexture = BABYLON.GUI.AdvancedDynamicTexture.CreateFullscreenUI(
    "UI"
  );
  varstackPanel = newBABYLON.GUI.StackPanel();
  stackPanel.verticalAlignment = BABYLON.GUI.Control.VERTICAL_ALIGNMENT_TOP;
  stackPanel.isVertical = true;
  advancedTexture.addControl(stackPanel);

  vartext1 = newBABYLON.GUI.TextBlock();
  text1.text = "";
  text1.color = "white";
  text1.fontSize = 16;
  text1.height = "30px";
  stackPanel.addControl(text1);

  vartext2 = newBABYLON.GUI.TextBlock();
  text2.text = "";
  text2.color = "white";
  text2.fontSize = 16;
  text2.height = "30px";
  stackPanel.addControl(text2);

  vartext3 = newBABYLON.GUI.TextBlock();
  text3.text = "";
  text3.color = "white";
  text3.fontSize = 16;
  text3.height = "30px";
  stackPanel.addControl(text3);

  vartext4 = newBABYLON.GUI.TextBlock();
  text4.text = "";
  text4.color = "white";
  text4.fontSize = 16;
  text4.height = "30px";
  stackPanel.addControl(text4);

  vartext5 = newBABYLON.GUI.TextBlock();
  text5.text = "";
  text5.color = "white";
  text5.fontSize = 16;
  text5.height = "30px";
  stackPanel.addControl(text5);

  vari = 0;
  scene.registerBeforeRender(function () {
    colorGrading.level = Math.sin(i++ 120 ) * 0.5 + 0.5;

//读取指示器
    text1.text =
      "currentframetime(GPU):" +
      (instrumentation.gpuFrameTimeCounter.current * 0.000001).toFixed(2) +
      "ms";
    text2.text =
      "averageframetime(GPU):" +
      (instrumentation.gpuFrameTimeCounter.average * 0.000001).toFixed(2) +
      "ms";
    text3.text =
      "totalshadercompilationtime:" +
      instrumentation.shaderCompilationTimeCounter.total.toFixed(2) +
      "ms";
    text4.text =
      "averageshadercompilationtime:" +
      instrumentation.shaderCompilationTimeCounter.average.toFixed(2) +
      "ms";
    text5.text =
      "compilershaderscount:" +
      instrumentation.shaderCompilationTimeCounter.count;
  });

  returnscene;
};
```

请注意每个计数器都是一个性能计数器对象，这个对象可以提供多种属性，比如平均值、合计值、最小值、最大值、计数等等。

GPU 计数器要起作用需要一个特殊的扩展（EXT_DISJOINT_TIMER_QUERY）。这个扩展因为幽灵与熔毁被所有的主要浏览器禁用了。但这时仍然可能通过在火狐浏览器上设置 gfx.webrender.debug.gpu-time-queries 标志来启用它。这一功能应该会很快在浏览器中被重新启用。

#### 场景计数器

场景计数器类允许你获取如下的计数器（每个场景都有）：

活动网格评价时间计数器：评价活动网格（根据活动相机的截锥体）所花费的时间（用毫秒表示）。必须设置

`instrumentation.captureActiveMeshesEvaluationTime=true.`

渲染目标渲染时间计数器：渲染所有渲染目标纹理所花费的时间（毫秒）必须设置

`instrumentation.captureRenderTargetsRenderTime=true.`

绘制调用计数器：每一帧里调用绘制的次数（其实是调用 engine.draw 的次数）。一个忠告是保持这个数字越小越好。

纹理冲突计数器：纹理不得不被移除来释放纹理槽位的次数。一般的，最新的硬件都有 16 个纹理槽位。Babylon.js 会尝试使用所有的槽位，因为绑定一个纹理的过程是昂贵的。保持这个数字尽量低是一个好主意。

帧时间计数器：计算一整个帧（包括动画、物理效果、渲染目标、特殊效果等等）所花费的时间（毫秒）。必须设置

`instrumentation.captureFrameTime=true.`

渲染时间计数器：渲染一帧花费的时间

`instrumentation.captureInterFrameTime=true.`

跨帧时间：两帧之间的时间

`instrumentation.captureParticlesRenderTime=true.`

粒子渲染时间：渲染粒子所花费的时间（包括粒子的动画）

`instrumentation.captureSpritesRenderTime=true.`

精灵渲染时间：渲染精灵所花费的时间

`instrumentation.capturePhysicsTime=true.`

物理效果时间：进行物理模拟所花费的时间

`instrumentation.captureCameraRenderTime=true.`

相机渲染时间：渲染一个相机所花费的时间

在每一帧的开始所有这些计数器都会被重置为 0.因此在 onAfterRender 的回调里获取它们是更容易的。
