## Unity 逆向学习 - 0

<br>

### 0. 一些废话

虽然一直叫嚣着 “我要学游戏开发” 什么的，然而拜拖延症所赐也就是叫嚣罢了。然而万万没想到有朝一日我会 ~~因为生活所迫而~~ 真的去学 unity ~~的皮毛~~。好在时间还算勉强充裕，因此决定老老实实来按照自己定的技术路线来搞一下。

<br>

### 1. 食材

今天的目标是分析 assets 相关文件。之前都是直奔 Assembly-CSharp.dll 去的；但今天我看到一篇 [CTF 解析](https://www.cnblogs.com/algonote/p/15589898.html) ，这才意识到首先应该厘清 “什么对象绑定了什么行为”。

虽然现在看来我应该选一个 Hello World 来做作为第一个分析的对象，白天的我还是一往无前地随便从 Unity Learn 上随便抓了一个 [2D Platformer](https://assetstore.unity.com/packages/templates/platformer-microgame-151055) 资源文件就开始尝试打包。以及萌新感慨：原来打包真的会卡死啊。

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

分析目标是 `WebGL Builds.data`。因为 AssetStudio 是 Windows 限定而我又懒得拖着台式机干活，本文所使用的分析工具为由 Python 实现的 `UnityPy`。

<br>

### 2. 围观资源类型

首先看了下资源文件中都包含哪些类型的 “资源”：

```python
{
    'Animator', 
    'RectTransform', 
    'Canvas', 
    'AudioListener', 
    'GraphicsSettings', 
    'GameObject', 
    'Tilemap', 
    'BuildSettings', 
    'ComputeShader', 
    'AudioManager', 
    'QualitySettings', 
    'InputManager', 
    'Physics2DSettings',
    'MeshFilter', 
    'LightingSettings', 
    'SortingGroup', 
    'TextAsset', 
    'StreamingManager', 
    'BoxCollider2D', 
    'Rigidbody2D', 
    'PlayerSettings', 
    'TagManager', 
    'ScriptMapper', 
    'CapsuleCollider2D', 
    'Sprite', 
    'NavMeshProjectSettings', 
    'UnityConnectSettings', 
    'Camera', 
    'NavMeshSettings', 
    'RuntimeInitializeOnLoadManager', 
    'PhysicsManager', 
    'VFXManager', 
    'RenderSettings', 
    'TilemapCollider2D', 
    'Font', 
    'ResourceManager', 
    'PreloadData', 
    'TilemapRenderer', 
    'AudioClip', 
    'TimeManager', 
    'AnimationClip', 
    'PolygonCollider2D', 
    'LightmapSettings', 
    'CanvasRenderer', 
    'Texture2D', 
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

根据之前粗浅的学习 + xjb 猜，觉得 `MonoScript, MonoBehaviour, ScriptMapper, DelayedCallManager, MonoManager` 这些应该比较重要；在各式解包文章中经常被提到的似乎是前两个，于是今天也只先围观了前两种。

<br>

### 3. 围观 MonoBehaviour

魔改 UnityPy 提供的 [extractor.py](https://github.com/K0lb3/UnityPy/blob/master/UnityPy/tools/extractor.py) 脚本，记录 MonoBehaviour 的所在二进制文件名等信息：

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

然而并不是所有的 MonoBehaviour 都能 dump 出相关信息（不知道如果使用 AssetStudio 能否规避掉这一问题。）对于未能 dump 信息的 MonoBehaviour，其 raw_data 被保存为 `<MonoBehaviour.name>.bin` 文件。结合游戏截图和文件名推测，`.bin` 文件所代表的似乎都是非库函数（所以更为重要？）的 MonoBehaviour：

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

**2022-04-09 更新**

用 AssetStudio 看了下，所谓开幕雷击：

![](https://raw.githubusercontent.com/Kyan0s/Kyan0s.github.io/main/assets/img/assets-cloud.png)

<br>

### 3. 围观 MonoScript

同样 dump。有趣的是这里未发现 dump 失败的问题。

```json
{
    "m_AssemblyName": "UnityEngine.TestRunner.dll",
    "m_ClassName": "AssemblyLoadProxy",
    "m_ExecutionOrder": 0,
    "m_Namespace": "UnityEngine.TestTools.Utils",
    "platform": "WebGL",
    "name": "AssemblyLoadProxy"
}
```

<br>

### 4. 思考 + TODO

+ ~~确认下 AssetStudio 是否更加好用。如若果真如此，搞清楚为什么对方更好用应该需要对 assets 文件格式的深入学习吧 ~~
+ MonoBehaviour 和 MonoScript 区别在哪里？目前的理解是 MonoBehaviour 需要绑定某一对象，不过这个不一定对。~~等等我为啥不打开源文件去 check~~
+ 学习如何找到 assets 中函数名与 `wasm` 中函数体的映射。粗浅瞅了一眼感觉会花一些功夫。

<br>


