我们的任务是创建世界上功能最强大，最美观，最简单的 Web 渲染引擎之一。我们的热情是使它对所有人完全开放和免费。

今天，我们很高兴宣布[Babylon.js 4.1](http://www.babylonjs.com/)正式发布！在深入探讨之前，我们还要衷心感谢 250 多位杰出的贡献者社区为建立此框架所做的努力。

<video id="video" controls="" preload="none" poster="https://appppppen.github.io/baby/articles/official/WelcomeToBabylon4.1/video/2020-07-04-09-57-34-817.png">
      <source id="mp4" src="https://appppppen.github.io/baby/articles/official/WelcomeToBabylon4.1/video/WelcometoBabylon4.1.mp4" type="video/mp4">
      <p>Your user agent does not support the HTML5 Video element.</p>
</video>

Babylon.js 4.1 的体积缩小了 3 倍，速度提高了 12％，包括无数的性能优化，延续了我们高性能引擎的血统。借助新的 Node Material Editor，Babylon Native，层叠阴影，导航网格，更新的 WebXR 和 glTF 支持以及在更多方面的真正跨平台开发体验，Babylon.js 4.1 为您的 Web 开发工具箱提供了更多功能。

### 节点材质编辑器

![](https://appppppen.github.io/baby/articles/official/WelcomeToBabylon4.1/image/1_UuzPUCWHRLJuTSqe1m62Ag.png)
引入功能强大且简单的“节点材质编辑器”。这个基于用户友好节点的新系统为每个人释放了 GPU 的强大功能。传统上，对着色器（GPU 程序）的编写对于任何不了解底层代码的人来说都不是那么容易或难以实现。巴比伦的新节点材料系统在不牺牲功能的情况下消除了复杂性。使用此新系统，绝对任何人都可以通过简单地将节点连接在一起来创建漂亮的着色器网络。
![](https://appppppen.github.io/baby/articles/official/WelcomeToBabylon4.1/image/1_ZdmVAMqoDzDoaAfYFfcpTA.gif)
为了了解 Node Material 的实际应用，我们为您提供了一些演示。在[水下演示](https://aka.ms/Uwater)及相应的[神秘演示教程视频](https://www.youtube.com/playlist?list=PLsaE__vWcRam5eDcUlGHvylTaATXCAQnC)展示节点材质如何使得编写复杂的顶点着色器更容易。在[幻想武器](https://aka.ms/DWeapon)演示展示了一些了不起的灯光效果着色器。如果您迫不及待想亲自尝试使用[节点材质编辑器](https://nme.babylonjs.com/)。

### 巴比伦本地预览

![](https://appppppen.github.io/baby/articles/official/WelcomeToBabylon4.1/image/1_jUbBGeMQ_DvS7mrCE5dZbA.png)
软件开发的圣杯是编写一次代码，并使其绝对在任何地方都可以工作：在任何设备，每个平台上。这就是巴比伦土生土长的灵感。巴比伦平台的这一令人兴奋的新功能使任何人都可以使用其 Babylon.js 代码并用它构建本机应用程序，从而释放本机技术的力量。您可以在[此处](https://aka.ms/Bnative)了解更多信息。

### 屏幕空间的思考

![](https://appppppen.github.io/baby/articles/official/WelcomeToBabylon4.1/image/1_rVZYOvsGB0Z5lp4K6PsEtg.png)
实时屏幕空间反射在这里！通过专用的[Julien Moreau Mathis](https://github.com/julien-moreau)的惊人努力，您现在可以为您所有的巴比伦体验增加全新的现实主义，深度和魅力。简单易用且美观，此功能确实是“必须尝试！” 在此处查看[现场演示](https://aka.ms/SReflect)。

### 级联阴影

![](https://appppppen.github.io/baby/articles/official/WelcomeToBabylon4.1/image/1_8H83p1W2uZW6jlNVG5AS_g.png)
Babylon.js 4.1 为引擎带来了最受社区欢迎的功能之一：级联阴影贴图！这项令人兴奋的新功能有助于分配阴影的分辨率，使阴影看起来清晰，平滑和美丽。最棒的是，它是由我们自己的社区成员之一 Popov72 创建的。在此处查看演示。

### 用于 2D 体验的 Thin Engine

![](https://appppppen.github.io/baby/articles/official/WelcomeToBabylon4.1/image/10_ojBgYfBsSKIwxSBp.jpg)
现在，Babylon.js 核心引擎的功能已经精简了，我们称之为“[瘦引擎](https://aka.ms/Tengine)”。
巴比伦的场景图以及所有其他工具和功能都依赖于作为技术中心枢纽的引擎。对于任何想大规模使用 Babylon.js 进行 2D 体验的人来说，此中央引擎的大小至关重要。该瘦引擎在一个很小的封装尺寸换来的原始动力删除功能。我们将核心引擎简化为裸机，我们创建了一个版本，专门用于以尽可能小的封装大小（未压缩的〜100KB）加速[2D 体验](https://doc.babylonjs.com/features/controls)。

### 导航网格物体和人群代理

![](https://appppppen.github.io/baby/articles/official/WelcomeToBabylon4.1/image/1_lRDbSfHSVQPm21K-b0NUjw.png)
借助新颖有趣且简单的 Navigation Mesh 系统，利用出色的开源 Recast 导航库的强大功能，为您的游戏或交互式体验创建令人信服的“ AI”比以往更加容易。只需为人群特工提供一个导航网格，该特工的移动将被限制在网格中。正如在此“ [水下演示](https://aka.ms/Uwater)”中的鱼所看到的，您会发现它对于 AI 和路径查找或替换物理来进行碰撞检测非常有用（仅允许玩家去到可能的地方，而不是使用碰撞检测）。更多信息在[这里](https://doc.babylonjs.com/extensions/navigationmesh)。

### 更新了 WebXR 支持

![](https://appppppen.github.io/baby/articles/official/WelcomeToBabylon4.1/image/1_ziY-gPXy4KHf0Rm2IY6dyw.png)
毫无疑问，网络上的 AR / VR 体验的前途一片光明。Babylon 4.1 通过带来以下优势，进一步提高了引擎的一流 WebXR 支持：

- 易于使用的体验助手
- 专为高级用户设计的会话管理器
- 一台相机统治一切
- 全面支持任何接受 WebXR 会话的设备
- 全面的输入源支持
- 实验性 AR 功能的 API
- 传送，场景交互，物理等

您可以在我们对[WebXR](https://doc.babylonjs.com/how_to/introduction_to_webxr)的介绍中找到更多详细信息。

### 更多

当然，这仅仅是冰山一角。此发行版中包含了太多内容，以至于几乎没有什么可提及的……几乎……

- 使用[MultiView](https://doc.babylonjs.com/how_to/multi_canvases)从 2 个不同的画布渲染同一场景
- 使用[Offscreen Canvas](https://doc.babylonjs.com/how_to/using_offscreen_canvas)在第二个工作线程中渲染 UI 元素
- 通过[实例缓冲区](https://doc.babylonjs.com/how_to/how_to_use_instances#custom-buffers)渲染数千个具有差异的对象
- 借助功能强大的新型[2D 控件](https://doc.babylonjs.com/features/controls)加速常见的[Web 控件](https://doc.babylonjs.com/features/controls)
- 通过实验性[KTX2 + BasisU](http://github.khronos.org/KTX-Specification/)支持减少了 3D 文件的大小
- 对即将到来的 glTF 扩展的实验支持：KHR_texture_basisu，KHR_mesh_quantization，KHR_materials_clearcoat，KHR_materials_sheen，KHR_materials_specular

有关新功能，更新，性能改进和错误修复的完整列表，请转到[此处](https://doc.babylonjs.com/whats-new)。

Babylon.js 4.1 标志着朝着创建世界上功能最强大，最美观，最简单和最开放的渲染引擎之一迈出的又一大步。我们迫不及待想看到您的作品。
