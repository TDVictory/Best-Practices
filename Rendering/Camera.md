# Camera（摄像机）
摄像机是Unity核心组件之一，任意一个Unity应用都重度依赖于它。这也意味着摄像机组件上具有大量的选项，
如果没有合适地配置像[Clear](#clear)，[Culling](#culling)，[Skybox](#skybox)这些选项，你将会得到一个质量极差的视觉效果。

## <h2 id="clear">Clear（清除）</h2>
在移动端的渲染上（使用Tile-Based），Clear的指令至关重要，Unity注重细节效果，因此在移动端开发上你只需要调整摄像机上的Clear flags，
不要选择“**Don't Clear**”即可。clear指令下的操作行为取决于你的开发平台以及显卡驱动，也同样依赖于你所选择的Clear flag参数，这些都将极大地影响最终的展示效果。因为Unity要么选择清除先前的内容并设置标签来忽略它，或者重新从内存中将其读取出来。不要在GPU运行时执行这些无用的Clear操作——然而这是在桌面程序、控制台中能经常看到的情况。

### Clear flags （清除标签）
在移动端，Unity在创建新场景时都会使用其自带的天空盒（它名称为*Default-Skybox*）,这是十分消耗计算性能的，请避免这么操作。请关闭天空盒渲染，并将**Camera.clearFlags**设置为 SolidColor。然后前往**Lighting Settings**（位置：**Window-Lighting-Setting**）窗口，移除天空盒材质，并将**Ambient Source**设置为*Color*。

### Discard and Restore buffer（丢弃和恢复缓存）
在高通GPU上运行**OpenGLES**，Unity只会丢弃帧缓冲（framebuffer）来避免恢复帧缓冲。在PVR和Mali的GPU中，Unity会通过清除来避免帧缓冲的恢复。

在移动设备中，显卡内存上的移入或移出操作是十分消耗资源的，因为这些设备使用shared memory（共享内存）的结构，这意味着CPU与GPU共享这相同的物理内存。在像高通，PowerVR抑或是Apple Aseries这样的Tile-Based GPU上，在logical buffer（逻辑缓存？）读取和存储数据会消耗大量的系统时间和电量。将每个Tile的内容从share buffer传递到framebuffer内的一部分，这是大量资源活动的一个重要原因。

### Tile-based Rendering
Tile-based rendering 将视窗划分成许多32×32像素的小块（Tile）。并将它们存储在靠近GPU的更快的内存里（keeps these tiles in faster memory closer to the GPU）。在这些更小的内存与帧缓冲之间的复制操作只会花费较少的时间，因为内存操作要比算法操作慢得多。

内存操作的缓慢也是你必须在新的一帧时通过调用tile-based GPU上的glClear（OpenGLES）指令来避免读取上一个帧缓冲的原因。通过发出一个glClear指令，你将告诉硬件设备：你不在需要先前的缓冲内容，因此它不再需要将帧缓冲中的颜色、深度、模板缓冲内容再复制到小型的tile内存中。

**注意**：少于16个像素的视窗可能再一些特定的芯片组上可能会处理的十分缓慢，这是因为这些芯片获取信息的方式不同所造成的。举个例子，将视窗设置成2×2像素，事实上可能比设置成16×16像素还要慢。这种运行缓慢是设备自身特有的，也不是超出Unity可以控制的部分，所以在你设置的时候不能忽略这个问题。

### RenderTexture Switching（纹理切换）
在你切换渲染目标时，显卡驱动会执行对帧缓冲的读取和存储操作。举个例子来说，如果你在连续的两帧中对一个视图的颜色缓冲和贴图进行渲染，系统会在shared memory 和GPU中重复传递（读取和存储）贴图的内容。

### Framebuffer Compression（帧缓冲压缩）
清除（Clear）指令同样会对包括颜色，深度，模板缓冲的帧缓冲的压缩产生影响。清除所有的缓冲使得帧缓冲能够更为紧密地压缩，减少驱动需要在GPU和内存中传递的数据量为更高的帧率提供了可能。在tile-based 结构下，清除tiles只是一个涉及到将一些二进制数据传输到每一个tile中的小任务。当这些完成时，这将使得每个tile都能够很轻易地从内存中被获取。

**注意**：这些优化运用于tile-based deferred rendering GPU 和 streaming GPU。

## <h2 id="culling">Culling（裁剪）</h2>
裁剪发生在每个摄像机上，特别是当复数照相机同时被开启时，会对场景的效果产生极大的影响。裁剪主要分为两种**frustum**和**occlusion**裁剪：
- Frustum Culling在每个Unity自带相机上，均会自动运行。
- Occlusion Culling将由开发者自己控制

### Frustum Culling（视截体裁剪）
Frustum Culling（视截体裁剪）保证在摄像机视线范围外的物体不会被渲染，从而节省渲染性能。

![](/Image/Rendering/occlusionfrustumculling.png)

*Frustum Culling（视截体裁剪）的例子*

**注意**：Frustum Culling（视截体裁剪）在2017.1及后续版本jobified（搜不到意思，估计是已经默认了不用管了），并且Unity现在也首先采用通过层（layer）裁剪的方式。通过层来裁剪也意味着Unity只会裁剪摄像机使用的层，并且忽略其他层的物体。总而言之，Unity都是基于相机视截体来裁剪物体的。

### Occlusion Culling（遮挡裁剪）
当你开启了Occlusion Culling（遮挡裁剪）时，Unity将不再渲染相机看不见的物体。举个例子，如果门是关着的，相机看不见房间内的东西，那么渲染房间就是没有必要的。

![](/Image/Rendering/occlusionfullculling.png)

*Occlusion Culling（遮挡裁剪）的例子*

如果你开启了Occlusion Culling（遮挡裁剪），它可以很大程度上帮你提升性能，但是它会占用更多的硬盘空间和RAM，因为Unity Umbra integration在搭建时将烘培遮罩数据，Unity在加载一个场景时需要将它从硬盘加载到RAM。

### Multiple Camera（多个摄像机）
当你在场景里使用多个摄像机时，每个摄像机上的裁剪与渲染会有一个大量的混合。Unity在2017.1版本后通过层裁剪（layer culling）来减少裁剪开销，但是如果摄像机不使用不同的层来构造渲染内容的话，这就没有任何效果了。

![](/Image/Rendering/sceneculling.jpg)

*Unity CPU 性能分析器在时间线视图上展示了主线程的性能。它表明有多个相机，并且你可以看到Unity对每个相机执行了裁剪*

### Per-Layer culling distances（不同层的裁剪距离）
你可以通过在脚本在相机上认为设置每层的裁剪距离。设置该裁剪距离对于当相机在一个给定的距离上看到并裁剪对场景没有贡献的小物体有很大的帮助。

## Skinned Motion Vectors（蒙皮运动矢量）
你可以开启相机上的Skinned Motion Vectors（蒙皮运动矢量）。如果你不开启它，Unity将不能使用运动矢量，也不会有性能上的影响。具体请参看官方文档[Camera.depthTextureMode](https://docs.unity3d.com/Manual/SL-CameraDepthTexture.html?_ga=2.121155969.849018300.1540284890-1902167384.1531452477)来了解更多。像Temporal Anti-aliasing (TAA)这样的后处理特效就需要激活运动矢量。

## Fillrate（填充率）
像素填充率的下降是过绘制（Overdraw）和片元着色器复杂所造成的~~难道不是填充率上升么？~~（Decreased pixel fillrate is a result of overdraw and fragment shader complexity）。Unity经常多线程（multiple passes）执行Shader（绘制漫反射，镜面反射等）。使用多线程会导致过绘制，也即会有多个着色器多次处理（读/写）同一个像素点。（对于硬件来说，能做到的像素填充率越高越牛逼，但是我们在优化的时候应该尽可能减少像素填充率）

### Overdraw（过绘制）
Unity的Frame Debugger（帧调试工具）对于我们了解Unity是如何绘制场景来说是十分有帮助的。请注意当你把大部分屏幕都用物体堆满时，Unity依然在绘制所有物体后面的东西（即便看不见）。该情景最常见的例子就是就是在一个激活的3D场景前唤出一个菜单（例如设置玩家的仓库），但是其实只是把该3D场景放在菜单后面而已。你同样应该注意那些Unity多次绘制的物体，因为Unity接下来将通过多线程绘制它。

### Overdraw view（过绘制窗口）
过绘制窗口允许你可以看到Unity在最上层绘制的物体交互，你可以通过**Scene View Control Bar**在Scene View内浏览过绘制界面。

![](/Image/Rendering/overdrawview.png)

*在Scene View Control Bar中观看过绘制*

![](/Image/Rendering/normal-shaded.png)

*在标准着色下的场景*

![](/Image/Rendering/overdraw.png)

*在过绘制视图下的相同场景*

过绘制视图在你想要将屏幕视图调整到你想要的位置时是十分有效的。Unity将物体渲染成透明轮廓。当透明物体堆叠后，就变得容易观察到那些物体重叠绘制的地方。白色部分是优化最差的部分，因为这部分的像素会被多次绘制，而黑色表示这里没有发生过绘制。

### Transparency（透明）
透明的物体也会被加到过绘制内，在最佳方案下，屏幕上的每个像素在每帧只应该被修改一次。

### Alpha Blending（透明度混合）
你应该避免堆叠具有Alpha Blending的物体（例如密集的粒子效果和全屏的后处理效果）来保证较低的像素填充率。

### Draw Order（绘制顺序）
在Unity中，不透明队列中的物体渲染顺序是从前到后的，使用一个边界盒（AABB的中心坐标系，应该是指视线内）（原文：using a bounding box (AABB center coordinates)）和深度测试来最小化过绘制（不透明的物体先渲染最前面的，然后开始深度测试，未通过则不会渲染，从而避免过绘制）。而透明队列的物体则是相反的（从后至前），保证透明队列的物体服从过绘制并不会执行深度测试。（因为透明物体是一定会过绘制的，从最后开始渲染每层都会叠加渲染，和深度无关）。Unity也把透明物体放在它们边界盒的中间位置（Unity also sorts Transparent GameObjects based on the center position of their bounding boxes.）

### Z-testing（亦可称深度测试）
Z-tesing要比绘制一个像素更快（我觉得应该是更早）。Unity通过边界盒执行裁剪和不透明物体的排序。因此Unity可能首先会绘制背景物体，例如天空盒和地面，因为边界盒十分大且包含许多在被其他物体过渲染后不可见的像素。如果你发现这类现象发生，人为地把这些物体放在队列的最后。

## Draw Call Batching（批量处理渲染要求）
PC硬件会推送大量的draw call，但是每个draw call的开销依然大到需要我们有理由去削减。在移动端，draw call的优化也是至关重要。

你可以通过下述简单方法来最大化批量处理：
- 在一个场景内使用尽可能少的纹理（Texture）。越少的纹理所需求的独立材质也就越少，这样使得对它们的批量处理也越为简单。顺带一提，无论何时请尽可能地使用纹理贴图集(Texture Atlas)。
- 将所有不移动的网格都在Inspector视图中标为**Static**。Unity在构造时将所有标志为Static的网格整合成一个大型的网格。（批量渲染主要用于节省提交给CPU的时间，cpu从提交多次到提交一次，对gpu来说也不用多次切换渲染状态。美术会根据场景的规模，将相邻的一片物件的贴图合并到一张或几张1024或512的大图上，这样这些物件可以使用同一个material，就可以静态批在一起，大幅节省Draw Call）你也可以在运行时通过使用StaticBatchingUtility来自己生成static batches。
- 每次烘培光照贴图（lightmap）时都采取最大的atlas（图集？）尺寸。越少的光照贴图所需求的材质状态变化也就越少。举个例子来说，一个三星S6/S8可以没啥压力地推送一个4096k大小的光照贴图，但是得注意内存占用。**注意**：你不需要把所有的物体都放在光照贴图内（当你把物体勾选了lightmap-static后就会放进光照贴图内）。虽然上述说的建议都是十分正确的——你需要把所有不移动的物体标注为**Static**——但是你需要忽略一些小物体（碎石，茶杯，书本之类），因为把它们加进光照图会强制Unity创建新的光照贴图（如果一张光照贴图空间不足的话）。小物体在使用光照探针（Light Probes）照亮时也有着非常不错的效果。
- 请小心不要实例化（Instance）材质。当你使用***Renderer.material***会自动生成一个材质的实例化（即我们每一次引用就会生成一个新的material到内存中。但是在引用后并不会改变我们项目工程中材质球的原始属性设置。），然后就会把该物体从批量处理中摘出。
无论何时都请使用***Renderer.sharedMaterial***（当我们改变Renderer.sharedMaterial的时候，所有使用这个材质球物体都会被改变，并且改变后的设置将会被保存在项目工程中。）。
- 注意那些多线程的着色器，无论合适都需要给你的着色器添加***noforwardadd***（似乎只有使用Surface的着色器使用，关闭Forward 渲染附加通道。 这使得shader支持单方向完全光照，以及所有其他由每个顶点/SH计算的光照。同时使得shader更小。）这样可以避免Unity提供多方向的光照破坏批量处理。
- 在优化时使用性能检测器（Profiler），内置的性能检测log，或者是统计框时，留意静、动态批量处理的数量对整个draw call数量的影响。

如果想要了解更多的知识，可以在Unity官方文档中阅读有关[draw call batching](https://docs.unity3d.com/Manual/DrawCallBatching.html?_ga=2.183556399.849018300.1540284890-1902167384.1531452477)的相关内容。

### Instancing（实例化）
实例化操作会强制Unity占用固定量的内存，在桌面端的GPU来说效果很好，但是在移动设备上运行缓慢。实例化操作只有在50-100网格时才有足够的帮助，当然也取决于现使用的硬件设备。

## Geometry（几何学）

