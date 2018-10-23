# 光照场景（Lighting Scenes）
优化场景光照并不是一项精细活——你对光照的处理往往依赖于你自身的艺术方向。为了创造一个满意的环境光往往需要许多步骤，而这也十分依赖该项目工程以及其独特的框架，包括且不限于该项目使用单独的场景、多场景编辑抑或是叠加式加载（additive loading）。因为这些原因，这篇文档将提供许多不同的处理方法，而非一个被推荐的工程流程。在下列章节中特别地包含了该主旨下的优良建议。
- [自动生成和光照生成(Auto Generate and Generate Lighting)](/Rendering/Auto-Generate-and-Generate-Lighting.md
)
- 光照设置(Lighting Setting)
- 多场景编辑(Multi-Scene editing)

当你开始一个新的项目时，遵守下列设置来正确运用多场景，并达到一个顺利的光照流程：
- 仅在单个场景的时候开启Auto Generate。
- 不要将开启了自动生成光照的场景和没有开启自动生成光照的场景混合。
- 无论是使用多场景编辑还是additive loading，确保所有的场景都有着相同的光照设置。

[返回上一级](/Rendering/Rendering.md)

[返回主页](/README.md)
