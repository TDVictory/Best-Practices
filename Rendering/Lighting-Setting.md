# 光照设置（Lighting Setting）
**光照设置窗口**（位置：**Winfow-Lighting-Settings**）包含多种可以控制光照表现的选项。

在光照设置窗口下的**Scene**标签内允许你对特定的场景，而非整个项目工程，进行光照设置。这也是我们所熟知的场景光照设置。当Unity中有着多个不同场景光照设置的场景存在时就会产生预期之外的光照效果。这种问题会发生在你在编辑模式下使用多场景编辑，或者是你在编辑、运行模式、输出项目文件时下采用叠加式场景加载的情况下。

当处理多个场景的光照设置时，Unity能够无误地将部分设置结合起来，但是其他部分需要完全重写或者混合。理解哪些光照设置是不需要设置的，哪些是需要重写的，哪些是需要混合的，这对我们处理多场景光照来说至关重要。
- **全局设置（Global Settings）** 是Unity在加载多个场景的时候会重写，混合的设置。
- **场景独立设置（Scene-dependent Settings）** 是Unity在加载多个场景的时候不会受到影响的设置。

## 全局设置（Global Settings）
以下设置均为全局设置：
- Environment中所有的设置
- Realtime Global Illumination
- Mixed Lighting
- Lighting mode
- Directional Mode
- Indirect intensity
- Albedo Boost
- Other Settings中所有的设置
- Auto Generation

当你使用多场景编辑或者是叠加式加载时需要认真考虑这些选项。当存在多个具有冲突全局设置的开放场景时，Unity会将这些设置进行混合与重写。而这也将导致不可预测的光照结果。因此当我们在运行模式下通过叠加式加载场景时，我们需要使用相同的全局设置。

当存在多个开放场景时，全局设定中的*wichever Scene Unity loads first*将成为全局设定的默认设置。这些默认默认的全局设置将成为后续叠加式加载的所有场景的基础设置，以此为基础Unity将数据进行添加和混合。这个状况将持续到你激活另外一个场景（通过破坏性地直接读取另外一个场景或者使用*SceneManager.SetActiveScene()*）。在该情况下，被激活的场景的全局设置将成为新的默认设置。

任何Unity叠加式加载的场景都会将其全局光照数据混合进现存的场景。举个例子，场景A有一个白天的天空盒，场景B有一个夜间的天空盒。如果你先加载了场景A，并且使用叠加式加载来加载了场景B，Unity将对于这两个场景均使用白天的天空盒。

涉及实时全局光照（Realtime Global Illumination）的情况将非常复杂。我们继续以上述例子为例，我们有一个场景A（有着白天的天空盒）和一个场景B（有着夜晚的天空盒），他们都使用了实时全局光照。如果你先加载了场景A，再叠加式加载了场景B，场景B将合并新场景的环境光并表现成白天的光照效果。然而如果关闭场景A的实时全局光照，从而产生的效果是一个白天的环境和一个烘培了夜晚效果的场景B。

的确实时全局光照（Realtime Global Illumination）是一个全局设置。如果你在一个场景内关闭了实时全局光照，而后续添加的带有实时全局光照的场景将会按照原本的光照进行渲染，因为场景内物体只会使用能获取到的烘培结果。同样的，当前激活的场景不存在定向模式（Diretional mode，该模式下会多烘培一张光照贴图，存储更多的光照信息，因此效果看上去会好一些，但是会更吃性能），那么带有定向模式的效果也不会展现。定向模式需要一个额外的贴图，如果当前场景关闭了定向模式，贴图自然也会丢失。

##场景独立设置（Scene-dependent Settings）
你也可以按自己需求设置一些特定的光照设置。下列设置属于场景独立设置，它们在被成功烘培后将不会受到其他场景的影响。
- Progressive Lightmapper内的所有设置
- Lightmap Resolution
- Lightmap Padding
- Lightmap Size
- Compress Lightmap
- Ambient Occlusion
- Final Gather

举个例子，Unity能够稳定地将一个以Lightmap Size为1024烘培的场景和另外一个 个以Lightmap Size为512烘培的场景。这是因为lightmap在烘培后就成为了一张Unity运行时使用的贴图文件，而不再与场景设置相关。如果你想要对场景使用不同的lightmap作为不同方案，请参照[Lightmap Switching Tool](https://github.com/laurenth-unity/lightmap-switching-tool)。

请谨记，如果在Unity编辑器中有着两个及以上场景时，在光照设置（Lighting Setting）视图中只会显示“主场景”——Unity第一个加载的场景，或者是你手动设置的激活场景——的光照设置。

[返回上一级](/Rendering/Lighting-Scenes.md)

[返回主页](/README.md)