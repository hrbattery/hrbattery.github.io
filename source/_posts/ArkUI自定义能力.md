---
title: arkui - 自定义能力
date: 2025-11-21
tags: ArkUI
---

任何一个UI框架提供的最基本的预制菜组件都总是不够用的。ArkUI的自定义能力能帮助用户对UI渲染的不同程度的定制需求。根据开放自定义程度的从低到高、使用难度的由易到难，官方文档分为四个层次：

- 自定义组合：使用@Builder和Component来复用已有组件、封装新的组件。
- 自定义扩展：基于Modifier，通过与UI分离的方式，对已有UI组件的属性、手势、内容等进行扩展修改。
- 自定义节点：具备底层实体节点的部分基础能力的节点对象，通过自定义占位节点与系统组件混合显示。具备单个节点的测算布局、设置基础属性、设置事件监听、自定义绘制渲染内容的自定义能力。
- 自定义渲染：通过XComponent暴露出NativeWindow，使用NDK的接口，直接接触到EGL/OpenGLES层，将显示数据直接写入NativeWindow中，实现渲染内容的自定义。

<!-- more -->

## 自定义组合

@Builder和Component作为最基本的概念之一，只需一笔带过就行。其他的场景也有值得说道的地方：（动画部分会放在另一篇讲）

### 自定义布局

除了使用Stack容器的层叠布局特性来实现自定义布局，也可以自行实现不同类型布局的算法。可以使用的接口有：

- [onMeasureSize](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-custom-component-layout#onmeasuresize10)：组件每次布局时触发，开发者可以在这个回调中增加自定义组件内子组件的大小的计算逻辑，返回自定义组件的尺寸信息，其执行时间先于onPlaceChildren。
- [onPlaceChildren](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-custom-component-layout#onplacechildren10)：组件每次布局时触发，开发者可以在这个回调中增加放置自定义组件内子组件位置的逻辑。

比较简单的一种方法是由Component出发、实现组件的这两个方法，但从使用的语法上，和系统预置的容器组件的写法还是有区别。
### 组合图形

介乎Canvas和预置组件之间的图形组件，本质拼图，好像也没什么好讲的

## 自定义扩展

自定义扩展依然是基于系统已有的组件做小修小补，相当于一个可以复用的“变化”，将同一套“变化”用在不同的组件上。
### ContentModifier

### AttributeModifier

### AttributeUpdater

### GestureModifier

## 自定义节点

这一级的自定义能力，在于不需要依赖其他的预置组件。各种系统组件在渲染的组件树中，都是以渲染树的树节点存在。使用自定义节点，可以使用更底层的自定义能力。基本的使用流程是这样的：
- 在组件树中声明一个自定义占位节点；
- 为自定义占位节点传入一个有实际绘制逻辑的自定义节点。
### 自定义占位节点

有两种自定义占位节点可以使用：

- [NodeContainer](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-basic-components-nodecontainer)作为容器节点存在，具备通用属性，是UI节点；
- [ContentSlot](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-rendering-control-contentslot)只是一个语法节点，无通用属性，不参与布局和渲染。

NodeContainer是一个容器节点，布局参考左上角对齐的Stack组件，不会按照父容器的布局规则进行布局；ContentSlot只是一个语法节点，不参与布局，添加的子节点会按照父容器的布局规则进行布局。

可以认为，NodeContainer适合用在复杂的纯ArkTS布局中。整个流程是：

- 创建一个NodeController对象
- 重写makeNode方法，返回一个FrameNode
- 将控制器绑定到NodeContainer上

相应的，ContentSlot在Native侧提供了接口，可以实现原生的命令式组件的绘制。整个流程是：

- 在ArkTS侧，创建一个NodeContent对象，并将其传递给ContentSlot组件
- 在Native侧，通过NDK UI API来获取与ArkTS侧的NodeContent对应的句柄（handle）
- 在Native侧创建所需的UI组件节点，并通过接口将其添加到NodeContent句柄的管理下

关于使用NDK UI API来绘制的部分，我们留在另一篇文章讲。
### FrameNode

官方文档给出了一个非常有意义的场景：将已有的第三方框架的UI结构转换为ArkUI的声明式描述。

FrameNode表示组件树中的实体节点，与自定义占位容器组件NodeContainer相配合，实现在占位容器内构建一棵自定义的节点树。该节点树支持动态操作，如节点的增加、修改和删除。

考查FrameNode提供的方法，基本可以归类为以下几类：

 - 节点树操作：包括子节点的增删查改和父节点的查找。
 - 布局操作：包括对节点自身的尺寸和位置的测量与设置。
 - 绘制操作：包括可见性、透明度的控制、绘制行为和时机、动画控制等。
 - 事件与属性：包括对自定义与通用事件和属性的获取与设置。
### RenderNode

对于不具备自己的渲染环境的三方框架，尽管已实现前端解析、布局及事件处理等功能，但仍需依赖系统的基础渲染和动画能力。FrameNode上的通用属性与通用事件对这类框架而言是冗余的，会导致多次不必要的操作，涵盖布局、事件处理等逻辑。

相比之下，RenderNode是更加轻量的渲染节点，仅提供了渲染相关的功能。它提供了设置基础渲染属性的能力、节点的动态添加、删除和自定义绘制的能力以及动画支持。

实际上，FrameNode就是通过持有RenderNode来实现渲染能力的。

另外，通过在draw回调中接受DrawContext的入参，可以在Native侧进行自定义的绘制操作，以此来实现高性能的自定义绘制。
### BuilderNode

BuilderNode的用法有二：

- 一是通过全局@Builder函数，提供将系统组件“转化”为FrameNode的途径。这样的套了FrameNode的壳的系统组件，既可以通过FrameNode自定义组件树的方法来调整显示效果、实现混合显示，又可以通过纹理导出功能、在XComponent中实现同层渲染。总之，目的是实现系统组件和自定义组件的混合显示。对于需与自定义节点对接的第三方框架，BuilderNode提供了嵌入系统组件的方法。
- 二是提供了组件预创建的能力，对于初始化耗时较长的声明式组件，比如Web、XComponent等，可以先提前创建好壳BuilderNode、再在需要时挂载显示。BuilderNode的预加载并不会减少组件的创建时间。Web组件创建的时候需要在内核中加载资源，预创建不能减少Web组件的创建的时间，但是可以让内核进行预加载，减少正式使用时候内核的加载耗时。

需要注意的是，BuilderNode仅可以作为叶节点使用。
### ComponentContent

众所周知，homoos的文档并不保证在出现一个概念之前对这个概念有最基本的解释（

ComponentContent表示组件内容的实体封装，其对象支持在非UI组件中创建与传递，便于开发者对弹窗类组件进行解耦封装。其底层使用了BuilderNode，相当于在系统构建弹窗功能的能力时，系统框架二次使用了自定义节点的功能。

FrameNode有addComponentContent方法，可以通过加载ComponentContent来设置该自定义节点的内容。

## 自定义绘制

### Canvas

Canvas画布主要是用来进行2D图形绘制的，提供的API基本和Html5提供的canvas的api几乎完全一致，除了必须在Canvas组件的onReady回调执行后才能执行绘制。我建议直接看[MDN的Canvas API教程](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API)。
### DrawModifier


### XComponent

Xcomponent可以用于EGL/OpenGLES和媒体数据写入，通过使用XComponent持有的NativeWindow来渲染画面。

XComponent有surface和texture两种渲染模式，前者直接将定制的绘制内容单独展示到屏幕上，后者则将定制的绘制内容和Xcomponent组件的内容合成后展示到屏幕上。

XComponent持有一个**Surface**。这个Surface是一个不对外暴露、但具体存在的绘制区域。通过传递给XComponent组件的 XComponentController 的`getXComponentSurfaceId()`方法，可以获取这个Surface的uid——我们得以了解其确实是存在的、而不是一个纯抽象的概念。通过Surface来获取一个**OHNativeWindow指针**。使用这个指针来进行图形绘制。

作为一个组件，除了绘制图形，还需要考虑到生命周期和交互的问题。而作为一个和Native侧有密切关系的组件，生命周期和交互的控制权应该在哪一侧，这是关键的问题。以此作为差别，官方提供了两种使用方式：

- 在ArkUI侧控制生命周期和交互，使用XComponentController管理Surface生命周期；
- 在Native侧控制生命周期和交互，使用OH_ArkUI_SurfaceHolder或NativeXComponent管理Surface生命周期。

不难思考得出：

- 前者适合交互逻辑不复杂、交互不需要频繁影响Surface绘制的场景，比如视频播放、相机预览；
- 后者适合交互逻辑多样复杂、交互会直接频繁影响Surface绘制，比如游戏场景、自绘制跨平台框架绘制。

实际上，之前的React Native也是使用XComponent来当作节点、再挂载通过C-API由React Component转换而来的ArkUI子节点。但由于React Native是原生绘制、而不是类似Flutter的Skia自绘制方案，所以后续就换成使用ContentSlot来做RN组件的根节点了。腾讯的Kuikly跨平台框架也是使用和RN类似的方案。

关于OH_ArkUI_SurfaceHolder和NativeXComponent方案的区别：

- 前者使用ArkTS侧的XComponent组件或者自定义组件节点：
	- 在onAttach阶段时将该组件对应的FrameNode节点传递给Native侧，来获取一个NodeHandle，然后调用`OH_ArkUI_SurfaceHolder_Create`接口创建SurfaceHolder实例。
	- 生命周期和事件回调分别在NodeHandle和SurfaceHolder上。与Surface相关的回调调用SurfaceHolder提供的`OH_ArkUI_SurfaceHolder_AddSurfaceCallback`接口来注册，其他的通用UI事件回调通过调用`registerNodeEvent`在NodeHandle上注册。
	- 基于SurfaceHolder实例注册生命周期和事件回调，来获取NativeWindow实例。随后绘制的步骤与XComponentController方案相同。
- 后者并不使用ArkTS侧的XComponent组件：
	- 在Native侧创建一个NodeHandle、将其属性定义为XComponent所需的属性、创建出一个Native侧定义的XComponent；
	- 在ArkTS侧，使用一个ContentSlot，其在Native侧提供NodeContent接口，将NativeXComponent节点提供给该接口，来实现完整的对接。
	- 生命周期和事件回调都在NativeXComponent实例上。生命周期的回调通过`OH_NativeXComponent_RegisterCallback`来注册，而其他事件的回调，要通过非常多不同的方法来设置不同类型事件的回调。
	- 基于NativeXComponent实例的生命周期回调来获取NativeWindow实例。随后绘制的步骤与XComponentController方案相同。

个人觉得最大的区别有二：一是简化了创建XComponent的流程，在这部分需要考虑的细节更少；二是可以将不同分类的回调区分开，各部分职责更清楚，否则可能在NativeXComponent和对应的NodeHandle上注册了相同的事件的监听和回调，实现就不优雅了。比如`OH_NativeXComponent_RegisterOnTouchInterceptCallback`方法，甚至在文档中都建议用NodeHandle监听`NODE_ON_TOUCH_INTERCEPT`事件来代替。但很遗憾，前者只能在API19及更新的系统上使用，因此还有一段时间才能减少兼容性上的担忧、以防旧设备无法使用。

在获取了NativeWindow对象之后，就可以使用EGL或者Vulkan来实现具体的绘制了。具体流程如下：

- 应用RequestBuffer获取空闲帧
- 应用生产帧数据
- 应用调用FlushBuffer提交到BufferQueue
- 系统渲染侧通过AcquireBuffer获取帧
- 渲染到屏幕
- 系统渲染侧通过调用ReleaseBuffer释放

#### 生命周期和事件回调

|             | XComponentController(ArkTS) | OH_ArkUI_SurfaceHolder(Native) |     |
| ----------- | --------------------------- | ------------------------------ | --- |
| Surface创建   | onSurfaceCreated            | onSurfaceCreated               |     |
| Surface大小改变 | onSurfaceChanged            | OnSurfaceChanged               |     |
| Surface销毁   | onSurfaceDestroyed          | OnSurfaceDestroyed             |     |


