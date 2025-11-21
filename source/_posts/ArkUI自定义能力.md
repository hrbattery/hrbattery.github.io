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

## 自定义组合

@Builder和Component作为最基本的概念之一，只需一笔带过就行。其他的场景也有值得说道的地方：（动画部分会放在另一篇讲）

### 自定义布局

除了使用Stack容器的层叠布局特性来实现自定义布局，也可以自行实现不同类型布局的算法。可以使用的接口有：

- [onMeasureSize](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-custom-component-layout#onmeasuresize10)：组件每次布局时触发，开发者可以在这个回调中增加自定义组件内子组件的大小的计算逻辑，返回自定义组件的尺寸信息，其执行时间先于onPlaceChildren。
- [onPlaceChildren](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-custom-component-layout#onplacechildren10)：组件每次布局时触发，开发者可以在这个回调中增加放置自定义组件内子组件位置的逻辑。

比较简单的一种方法是由Component出发、实现组件的这两个方法，但从使用的语法上，和系统预置的容器组件的写法还是有区别。
### 组合图形

介乎Canvas和预置组件之间的图形组件，本质拼图，好像也没什么好讲的
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
 - 生命周期：包括与
### RenderNode

### BuilderNode

### ComponentContent
## 自定义扩展

### ContentModifier

### AttributeModifier

### AttributeUpdater

### GestureModifier

## 自定义绘制

### Canvas

### DrawModifier

### XComponent

