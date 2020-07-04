## 巴比伦本地：迄今为止的旅程

“借助 Babylon.js 的功能来实现跨平台的本机应用程序” —当团队于去年 3 月首次宣布该项目时，这就是 Babylon Native 的目标。十个月后，这项工作现在已经成为[现实](https://github.com/BabylonJS/BabylonNative)，它将在 2 月底与 Babylon.js 4.1 发行版一起提供第一版预览。

![](https://appppppen.github.io/baby/articles/official/BabylonNative/image/1_Xuv0I-AuA-2xbu3wQn7YFw.png)

> 相同的 JavaScript 代码可在 Web 和 Win32 应用程序上呈现完整的 3D 场景（甚至具有物理学！）（[演示视频](https://www.youtube.com/watch?v=JL081J38ens) | [代码](https://playground.babylonjs.com/#17PW63)）

我们已经将该项目介绍给了很多合作伙伴，并且正在积极与其中一些合作伙伴进行跨平台开发。本文分享了到目前为止我们的一些经验，主要是 针对该项目的新手。

### 渲染 3D 跨平台的另一种方法

为了更好地了解 Babylon Native 的位置，让我们开始研究使用 Web 技术渲染 3D 跨平台的不同方法。

- **浏览器中的 3D** -您可以使用 JavaScript 框架（如 Babylon.js）在浏览器中呈现 3D 应用程序。浏览器在 WebGL，WebXR，Web Audio 等标准上运行，这些标准可确保代码的跨平台方面。
- **3D 嵌入浏览器技术** -你可以打包你的网站变成一个渐进的 Web 应用程序，电子应用或嵌入的 WebView。在这种情况下，由于通过浏览器技术启用了它，所以它仍然依赖于跨平台标准，例如 WebGL，WebXR，Web Audio 等。
- **本机应用程序中的 3D** -在 Babylon Native 中，您仍然使用 Web 技术（例如 JavaScript）对应用程序进行编码，但是这次，它在本机 API 上运行。Babylon 本机运行时可在平台本机图形 API（Windows 上的 DirectX，iOS / MacOS 上的 Metal，将来的 Android 和 Vulkan 上）上运行您的应用程序。

### 用例和优势

在与早期合作伙伴的讨论中，由于各种原因，这种方法引起了很多关注。以下是一些用例和优点。

- **跨 Web 和移动应用程序的单一代码库** -能够编写一次并重复使用代码的情况经常出现，因为它减少了开发和维护多个应用程序的成本。

![](https://appppppen.github.io/baby/articles/official/BabylonNative/image/1_vlJLGM2Vhgy5NONYbN12Fg.png)

- **glTF 一致性和一致性** -通过使用 Babylon.js，可以立即支持最新的 PBR 材料扩 ​​ 展和持续一致性。对于客户需要最高保真度的电子商务场景中的产品 3D 表示，这尤其重要。例如，PBR Sheen 材料需要准确地代表沙发的面料（想象一下，可以在在线订购之前先通过 AR 在客厅里对它进行拍照）。
- **移动应用程序的文件大小** -较大的文件大小可能会影响移动应用程序的使用（例如，在 iOS 上，200 MB 之后，在通过无线下载之前，会向用户显示较大应用程序大小的警告消息）。Babylon Native 应该大约几十 MB，最大的部分是 JavaScript 引擎（大约 20 MB）。
- **广泛的开发人员访问权** -使用 Web 技术开发应用程序使更广泛的开发人员和内容创建者可以利用该平台。由于没有编译，开发迭代时间更短，甚至可以动态更新和重新加载应用程序代码。

### 使用 Babylon Native 构建应用程序

Babylon Native 通过在 JavaScript 上下文中运行 Babylon.js 来工作，因此，大多数在 Babylon.js 中编写的代码都将在 Babylon Native 中工作。除了少数例外，您还将受益于本机应用程序中 Babylon.js 框架的所有功能和快速发展。

您可能需要适应的是一些 UI 代码。Babylon.js 具有某些 UI 功能，特别是用于 XR 和其他用途的 3D UI，这些功能也可以在 Babylon Native 中使用。但是，Babylon Native 绝不打算包含 DOM 的实现，因此，根据您选择实现 UI 的方式，您可能必须合并其他技术。

![](https://appppppen.github.io/baby/articles/official/BabylonNative/image/1_kKVuJd1R0aBpVDfaYlpiKw.png)
该团队目前正在研究一种在 Babylon Native 和 React Native 之间集成的解决方案。

![](https://appppppen.github.io/baby/articles/official/BabylonNative/image/1_CRV87kxcIF7A04xk1xVm5g.png)
基于 React 的技术与 Babylon Native 共享许多范例，并且两个库可以很好地协同工作。但是，它们并不捆绑在一起。无论使用 React，React Native，手工原生 UI 还是任何其他技术，您所包括的 UI 技术完全取决于您。

### 动手做

在准备好生产之前，仍有许多工作要做，但是您已经可以对其[进行研究](https://github.com/BabylonJS/BabylonNative)，以更好地了解该项目，并开始考虑它是否可能成为您未来计划的一部分。

我们衷心希望巴比伦原生语言能够更轻松，更快捷地在跨平台原生应用程序中利用功能强大且美观的 3D。无论平台如何，我们都迫不及待想看到您创建的内容！

Thomas Lucchini-Babylon.js 团队

https://twitter.com/thomlucc
