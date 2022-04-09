## Unity 逆向学习记录 - 0

<br>

### 0. 一些废话

虽然一直叫嚣着 “我要学游戏开发” 什么的，然而拜拖延症所赐也就是叫嚣罢了。万万没想到生活所迫这玩意重新搞起来了 doge

<br>

### 1. 食材

今天的目标是分析 assets 相关文件。之前都是直奔 Assembly-CSharp.dll 去的，现在想来这也大抵没错。但教程中提到的 WebGL 编译使用了 il2cpp，所以需要先从还原 Assembly-CSharp.dll （或者说至少拿到一个函数声明表？）开始。

食材为从 Unity Learn 上随便抓的一个游戏 [2D Platformer](https://assetstore.unity.com/packages/templates/platformer-microgame-151055) 。如果时光能倒流，我选择以分析 Hello World 开始。 QAQ

教程中该游戏大概长这样：

![](https://connect-prd-cdn.unity.com/20190313/learn/images/ef3bf79d-def0-41d8-bd08-a842cc92c0e2_PlatformerTrails.png)

目标平台是用的是教程默认的 WebGL，所生成目录结构如下：

```shel
192:WebGL Builds Kyan0s$ tree
.
├── Build
│   ├── WebGL Builds.data.gz
│   ├── WebGL Builds.framework.js.gz
│   ├── WebGL Builds.loader.js
│   └── WebGL Builds.wasm.gz
├── GUID.txt
├── ProjectVersion.txt
├── TemplateData
│   ├── favicon.ico
│   ├── fullscreen-button.png
│   ├── progress-bar-empty-dark.png
│   ├── progress-bar-empty-light.png
│   ├── progress-bar-full-dark.png
│   ├── progress-bar-full-light.png
│   ├── style.css
│   ├── unity-logo-dark.png
│   ├── unity-logo-light.png
│   └── webgl-logo.png
├── dependencies.txt
└── index.html
```

根据粗浅搜索，剥离（我个人理解）的二进制文件为 `.wasm`，而函数名等需要从作为资源文件的 `.data` 中摘出。结合 `.wasm` 与从资源文件中提取出的 `global-metadata.dat` ，Il2CppDumper 能够还原函数声明等，并据此辅助对 `.wasm` 的逆向。

AssetStudio 能够完成提取 `global-metadata.dat` 这一工作。但一开始我只知道要用这个软件却不知道应具体使用其何种功能，于是先折腾了下其在 MacOS 上的平替：由 Python 实现的 `UnityPy`。

<br>

### 2. 围观 UnityPy

虽然目前来看 UnityPy 相比 AssetStudio 还是 Bug 多了一点，但清楚的文档和使用示例还是蛮适合纯零基础小白入坑。

首先围观了下资源文件中都包含哪些类型的资源：

```python
{
    'Animator', 
    'RectTransform', 
    ...,
    'MonoBehaviour', 
    'DelayedCallManager', 
    'Material', 
    'MonoScript', 
    'AudioSource', 
    'Grid', 
    'Transform', 
    'AnimatorController', 
    'CircleCollider2D', 
    'Mesh', 
    'MonoManager', 
    'SpriteRenderer'
}
```

在各式解包文章中经常被提到的似乎是 `MonoScript` 和 `MonoBehaviour`，后续也着重分析了这两种类型。通过魔改 UnityPy 提供的 [extractor.py](https://github.com/K0lb3/UnityPy/blob/master/UnityPy/tools/extractor.py) 脚本，记录 `MonoBehaviour` 及 `MonoScript` 的相关信息：

```json
{
    "m_AssemblyName": "Unity.TextMeshPro.dll",
    "m_ClassName": "TMP_FontAsset",
    "m_ExecutionOrder": 0,
    "m_Namespace": "TMPro",
    "platform": "WebGL",
    "name": "TMP_FontAsset"
}
```

然而并不是所有的 `MonoBehaviour` 都能 dump 出相关信息。下述 `.bin` 文件即为分析失败的对象；结合游戏截图和文件名推测，分析失败的对象似乎反而更为重要：

```shell
192:MonoBehaviour Kyan0s$ ls | cat
Default_Style_Sheet.json
EmojiOne.json
GameSkin.json
LiberationSans_SDF.json
MidgroundFiller.bin
ShortBuilding.bin
TMP_Settings.json
TallBuilding.bin
TileFloatingLeftEdge.bin
TileFloatingRightEdge.bin
TileFloatingTileMiddle.bin
TileGround.bin
TileGroundDark.bin
TileGroundTop.bin
cloud.bin
fence.bin
hillside.bin
house.bin
midground.bin
mountains.bin
plant.bin
tree.bin
```
<br>

### 3. 分析的正途

上文分析失败的 `MonoBehaviour` 被 AssetStudio 好好地分析出来了，虽然乍一看可读性不如  UnityPy：

![](https://raw.githubusercontent.com/Kyan0s/Kyan0s.github.io/main/assets/img/assets-cloud.png)

但这不妨碍通过 `File -> Extract file` 提取得到 `global-metadata.dat`。然后倒入 `.wasm`，加以搅拌后喂给 Il2CppDumper，于是期望发生的一切都发生了（？）。新版本的 Il2CppDumper 会将函数声明等保存为 json 文件，然后提供了在 IDA / Ghidra 等工具中使用该 json 文件的脚本。~~真的好好奇这些脚本如何工作，拥有相似特征的函数万一很多怎么办？总之之后先上手试试。~~

<br>
