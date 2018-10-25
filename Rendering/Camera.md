# 摄像机
摄像机是Unity核心组件之一，任意一个Unity应用都重度依赖于它。这也意味着摄像机组件上具有大量的选项，
如果没有合适地配置像[Clear](#Clear)，[Culling](#Culling)，[Skybox](#Skybox)这些选项，你将会得到一个质量极差的视觉效果。

## <h2 id="Clear">Clear（清除）</h2>
在移动端的渲染上（使用Tile-Based），Clear的指令至关重要，Unity注重细节效果，因此在移动端开发上你只需要调整摄像机上的[Clear flags](#Clear-flags)，
不要选择“**Don't Clear**”即可。clear指令下的操作行为取决于你的开发平台以及显卡驱动，也同样依赖于你所选择的Clear flag参数，这些都将极大地影响最终的展示效果。因为Unity要么选择清除先前的内容并设置标签来忽略它，或者重新从内存中将其读取出来。不要在GPU运行时执行这些无用的Clear操作——然而这是在桌面程序、控制台中能经常看到的情况。

### <h3 id="Clear-flags">Clear flags（清除标签）</h3>
在移动端，Unity在创建新场景时都会使用其自带的天空盒（它名称为*Default-Skybox*）,这是十分消耗计算性能的，请避免这么操作。请关闭天空盒渲染，并将**Camera.clearFlags**设置为 SolidColor。然后前往**Lighting Settings**（位置：**Window-Lighting-Setting**）窗口，移除天空盒材质，并将**Ambient Source**设置为*Color*。

### Discard and Restore buffer（丢弃和恢复缓存）
在高通GPU上运行**OpenGLES**，Unity只会丢弃帧缓冲（framebuffer）来避免恢复帧缓冲。在PVR和Mali的GPU中，Unity会通过清除来避免帧缓冲的恢复。

在移动设备中，显卡内存上的移入或移出操作是十分消耗资源的，因为这些设备使用[shared memory（共享内存）](#Share-Memory)的结构，这意味着CPU与GPU共享这相同的物理内存。在像高通，PowerVR抑或是Apple Aseries这样的Tile-Based GPU上，在logical buffer（逻辑缓存？）读取和存储数据会消耗大量的系统时间和电量。将每个Tile的内容从share buffer传递到framebuffer内的一部分，这是大量资源活动的一个重要原因。

### Tile-based Rendering
Tile-based rendering 将视窗划分成许多32×32像素的小块（Tile）。并将它们存储在靠近GPU的更快的内存里（keeps these tiles in faster memory closer to the GPU）。在这些更小的内存与帧缓冲之间的复制操作只会花费较少的时间，因为内存操作要比算法操作慢得多。

内存操作的缓慢也是你必须在新的一帧时通过调用tile-based GPU上的glClear（OpenGLES）指令来避免读取上一个帧缓冲的原因。通过发出一个glClear指令，你将告诉硬件设备：你不在需要先前的缓冲内容，因此它不再需要将帧缓冲中的颜色、深度、模板缓冲内容再复制到小型的tile内存中。

**注意**：少于16个像素的视窗可能再一些特定的芯片组上可能会处理的十分缓慢，这是因为这些芯片获取信息的方式不同所造成的。举个例子，将视窗设置成2×2像素，事实上可能比设置成16×16像素还要慢。这种运行缓慢是设备自身特有的，也不是超出Unity可以控制的部分，所以在你设置的时候不能忽略这个问题。

### RenderTexture Switching（纹理切换）
在你切换渲染目标时，显卡驱动会执行对帧缓冲的读取和存储操作。举个例子来说，如果你在连续的两帧中对一个视图的颜色缓冲和贴图进行渲染，系统会在shared memory 和GPU中重复传递（读取和存储）贴图的内容。

### Framebuffer Compression（帧缓冲压缩）
清除（Clear）指令同样会对包括颜色，深度，模板缓冲的帧缓冲的压缩产生影响。清除所有的缓冲使得帧缓冲能够更为紧密地压缩，减少驱动需要在GPU和内存中传递的数据量为更高的帧率提供了可能。在tile-based 结构下，清除tiles只是一个涉及到将一些二进制数据传输到每一个tile中的小任务。当这些完成时，这将使得每个tile都能够很轻易地从内存中被获取。

**注意**：这些优化运用于tile-based deferred rendering GPU 和 streaming GPU。

## Culling（裁剪）
裁剪发生在每个摄像机上，特别是当复数照相机同时被开启时，会对场景的效果产生极大的影响。裁剪主要分为两种**frustum**和**occlusion**裁剪：
- [Frustum Culling](#Frustum-Culling)在每个Unity自带相机上，均会自动运行。
- [Occlusion Culling](#Occlusion-Culling)将由开发者自己控制

### <h3 id="Frustum-Culling">Frustum Culling（视截体裁剪）</h3>
Frustum Culling（视截体裁剪）保证在摄像机视线范围外的物体不会被渲染，从而节省渲染性能。

![](/Image/Rendering/occlusionfrustumculling.jpg)
*Frustum Culling（视截体裁剪）的例子*

**注意**：Frustum Culling（视截体裁剪）在2017.1及后续版本jobified（搜不到意思，估计是已经默认了不用管了），并且Unity现在也首先采用通过层（layer）裁剪的方式。通过层来裁剪也意味着Unity只会裁剪摄像机使用的层，并且忽略其他层的物体。总而言之，Unity都是基于相机视截体来裁剪物体的。

### <h3 id="Occlusion-Culling">Occlusion Culling（遮挡裁剪）</h3>
当你开启了Occlusion Culling（遮挡裁剪）时，Unity将不再渲染相机看不见的物体。举个例子，如果门是关着的，相机看不见房间内的东西，那么渲染房间就是没有必要的。

![](/Image/Rendering/occlusionfullculling.jpg)
*Occlusion Culling（遮挡裁剪）的例子*

如果你开启了Occlusion Culling（遮挡裁剪），它可以很大程度上帮你提升性能，但是它会占用更多的硬盘空间和RAM，因为Unity Umbra integration在搭建时将烘培遮罩数据，Unity在加载一个场景时需要将它从硬盘加载到RAM。

### Multiple Camera（多个摄像机）
当你在场景里使用多个摄像机时，每个摄像机上的裁剪与渲染会有一个大量的混合。Unity在2017.1版本后通过层裁剪（layer culling）来减少裁剪开销，但是如果摄像机不使用不同的层来构造渲染内容的话，这就没有任何效果了。

![](/Image/Rendering/sceneculling.jpg)
*Unity CPU 性能分析器在时间线视图上展示了主线程的性能。它表明有多个相机，并且你可以看到Unity对每个相机执行了裁剪*
