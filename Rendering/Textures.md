# 纹理贴图（Textures）
纹理贴图Unity工程内是一个至关重要的部件，你需要时刻关注贴图的尺寸和压缩情况。在移动端和控制台，由于运行内存与磁盘空间受限，保持一个小的贴图尺寸显得格外重要。
选择一个正确的压缩格式对于将贴图压缩到一个合适的大小来节省内存带宽有着极大的帮助。

## Asset Auditing（资产审计）
通过资产审计自动化，你可以避免突发或是意外的资产设置变动。在Github上的[AssetAuditor](https://github.com/MarkUnity/AssetAuditor)包含了许多种的审计过程。资产审计不仅能帮你还原贴图的性能，你还能同样运用于Unity内大量的资产类型。你可以通过[Understanding Optimisation in Unity](https://docs.unity3d.com/Manual/BestPracticeUnderstandingPerformanceInUnity.html?_ga=2.190121515.1660928137.1540775762-1902167384.1531452477)中的[Asset Auditing](https://docs.unity3d.com/Manual/BestPracticeUnderstandingPerformanceInUnity4.html?_ga=2.190121515.1660928137.1540775762-1902167384.1531452477)来进行学习。

## Texture Compressiong（贴图压缩）
当你合理地运用贴图压缩后，它会对你的表现有着极大的帮助。新式新的移动设备上，你应该选择使用ASTC贴图压缩格式。如果你使用的目标设备不支持ASTC，在Android上使用ETC2，IOS上使用PVRTC

### ASTC
Unity 4.3以后为ARM添加的ASTC压缩提供支持。这在构建时非常有用，因为它允许Unity比ETC2​​或PVRTC更快地压缩ASTC。在iOS上，ASTC可用于A8芯片及更高版本; 在Android上，ASTC可用于大多数现代芯片组。

Mali GPU（Mali T-760 MP8及以上版本）需要在ETC2上进行ASTC压缩。

有关更多信息，请参见第4.2.3节“ASTC纹理压缩”中的官方ARM文档。

如果硬件不支持ASTC（例如，在Adreno GPU上），则必须选择后备，例如ETC2。有关ASTC的其他信息，请参阅NVidia文章为游戏资产使用ASTC纹理压缩。

### PVRTC
在Apple添加ASTC之前，PVRTC是iOS上的主要纹理压缩格式。如果您在Android上使用PVRTC，则应尽可能将其替换为ETC2。

**注意**： iOS和ETC格式（Android 4.x设备）上的PVRTC纹理格式需要方形纹理。压缩非方形纹理时，可能会出现两种行为：

如果没有Sprite使用纹理并且压缩的内存占用量小于未压缩时的内存占用量，Unity会根据非二次幂（NPOT）纹理比例因子调整纹理大小。

否则，Unity不会调整纹理大小，并将其标记为未压缩。

## GPU Upload（GPU上传）
Unity会在读取完毕贴图后直接上传给GPU，而并不会等到贴图被摄像机所看到后再执行操作。

当加载线程完成场景或资产的加载后，Unity需要将它们唤醒。加载过程在Unity内发生在何处，又是如何发生都取决于Unity自身的版本和用来初始化加载的指令。

### Load Behavior（加载行为）
如果从AssetBundles，Resources或Scenes加载资产，Unity将从预加载线程（磁盘I / O）转到图形线程（GPU上载）。如果您使用Unity 5.5或更高版本，并且启用了图形作业，Unity会从预加载作业直接转到GPU。

### Awake Behavior（加载行为）
Unity会在主线程中唤醒所有场景物体后直接唤醒资产（Assets）。如果你使用**AssetBundle.LoadAsset**, **Resources.Load** or **SceneManager.LoadScene** 来加载资产和场景，Unity会阻断主线程并唤醒所有的资产，如果你使用这些指令的无阻断版本（异步加载， 例如AssetBundle.LoadAssetAsync），Unity会以time-slicing（时间切分）来唤醒这些资产。

### Memory Behavior（记忆行为）
当一次性读取数个贴图时，如果读取速率不够迅速，抑或是主线程停滞，你可以调整贴图缓存。但是这种更改默认值的操作，可能会导致较高的存储压力。

**注意**：如果GPU内存过载了，GPU就会把近期最少使用的贴图卸载，并当该贴图再次进入视线中时强制CPU来重载它。

[返回上一级](/Rendering/Rendering.md)

[返回主页](/README.md)
