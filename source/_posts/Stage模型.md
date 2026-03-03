---
title: arkui - Stage模型
date: 2026-3-3
tags: ArkUI
---


![[Pasted image 20251008165000.png]]

# 应用组件

- AbilityStage
	- AbilityStage实例是一个组件管理器，所以每个module（entry、feature）都对应一个AbilityStage单例。
	- 当Hap中的代码首次被加载到进程中的时候，系统会先创建AbilityStage实例。这是一个单例实例。其销毁的时机是在对应module的最后一个Ability实例销毁时。
	- 可以通过一些钩子，在某些事件发生时触发回调：
		- onAcceptWant()：在UIAbility通过[指定实例模式（specified）](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/uiability-launch-type#specified%E5%90%AF%E5%8A%A8%E6%A8%A1%E5%BC%8F)启动时候触发的事件回调。
		- onConfigurationUpdate()：在系统环境变量发生变更时触发的事件回调。
		- onMemoryLevel()：在系统调整应用可用内存空间时的事件回调，
		- onNewProcessRequest()：UIAbility启动时触发的事件回调，可以用于指定其启动是否在独立进程中创建。
		- onPrepareTermination()：当应用被用户关闭时调用。
- UIAbility
	- UIAbility组件是一种包含UI的应用组件，主要用于和用户交互。
	- UIAbility组件是系统调度的基本单元，为应用提供绘制界面的窗口。一个应用可以包含一个或多个UIAbility组件。例如，在支付应用中，可以将入口功能和收付款功能分别配置为独立的UIAbility。
	- 每一个UIAbility组件实例都会在最近任务列表中显示一个对应的任务。
	- 当用户在执行应用启动、应用前后台切换、应用退出等操作时，系统会触发相关应用组件的生命周期回调。作为一种包含UI的应用组件，UIAbility的生命周期不可避免地与[WindowStage](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-window-stage)的生命周期存在关联关系。![[Pasted image 20251008172353.png]]
	- 系统提供了singletion、multition和specified三种启动UIAbility的方式。
		- 单实例模式下，每次调用startAbility方法时，会复用系统中已经存在的UIAbility，会调用onNewWant()回调、而不会进入onCreate()和OnWindowStageCreate()回调。
		- 多实例模式下，每次调用startAbility方法时，会在应用进程中创建一个新UIAbility实例。
		- 指定实例模式下，会根据AbilityStage中的onAcceptWant()方法，判定复用已有的UIAbility实例（进入onNewWant()回调）、或者创建新的UIAbility实例（进入OnCreate()回调）。
	- UIAbility会持有一个UIAbilityContext实例，其存储了UIAbility的上下文信息，以及操作UIAbility实例的方法。
		- 在UIAbility中可以通过this.context获取UIAbility实例的上下文信息。
		- 当业务完成后，开发者如果想要终止当前[UIAbility](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-app-ability-uiability)实例，可以通过调用[terminateSelf()](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-inner-application-uiabilitycontext#terminateself)方法实现。
- ExtensionAbility
	- ExtensionAbility组件是一种面向特定场景的应用组件。它由应用的AbilityStage管理，但能力和生命周期是非常特定的，受操作系统管理。
	- 所有类型的[ExtensionAbility](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-app-ability-extensionability)组件均不能被应用直接启动，而是由相应的系统管理服务拉起，以确保其生命周期受系统管控，使用时拉起，使用完销毁。ExtensionAbility组件的调用方无需关心目标ExtensionAbility组件的生命周期。注意，使用方并非提供方。
- WindowStage
	- WindowStage是用于管理应用进程内窗口管理器的一个实例，可以加载不同的ArkUI页面。
	- 每个UIAbility实例都会与一个WindowStage类实例一对一绑定。
- Context
	- Application、AbilityStage、UIAbility、各种ExtensionAbility，都会持有自己的Context，提供对应的持有其的应用组件所可以调用的各种资源和能力。
- Want
	- 其实就是安卓的Intent（
	- 可以在同应用或跨应用的应用组件间传递信息。
	- 其中，一种常见的使用场景是作为[startAbility()](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-inner-application-uiabilitycontext#startability)方法的参数。例如，当UIAbilityA需要启动UIAbilityB并向UIAbilityB传递一些数据时，可以使用Want作为一个载体，将数据传递给UIAbilityB。
	- 显式Want已经不推荐用于拉起其他应用，而是使用应用链接的方式、用openLink方法来打开其他应用。

# 应用进程模型和线程模型

现在我们有代表应用的Application，有代表模块的Module、有管理模块所有组件的AbilityStage、还有其他的各种组件，那么对应回操作系统最基本的进程和线程模型上，是什么对应关系呢？
## 进程模型

在应用运行态，可能存在的进程类型有：

- 主进程：默认情况下，同一个应用的所有UIAbility都运行在同一个进程中，称其为主进程；
- ExtensionAbility进程：同一应用的所有同一类型ExtensionAbility均运行在一个独立进程中。特别地，对于继承自[UIExtensionAbility](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-app-ability-uiextensionability)的ExtensionAbility，可以为每个实例配置独立进程。
- Web渲染进程：特别的，如果有UIAbility调用了Web组件，这个Web组件的渲染是在一个独立的渲染进程的。
![[Pasted image 20251008185601.png]]

在2in1和Tablet设备上，针对UIAbility，还支持如下特殊进程类型：

- 模块独立进程：可以通过设置isolationMode字段，让同一个Hap下的所有UIAbility运行在统一的、独立的进程中。
- 动态指定进程：可以通过设置isolationProcess字段，在主控进程（MasterProcess）中控制运行在哪个进程上。
- 静态指定进程：可以通过配置abilities/extensionAbilities标签的process字段，将其分配在不同进程中。
- 子进程：以调用[childProcessManager](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-app-ability-childprocessmanager)中的接口创建子进程。

## 线程模型

一个进程里可以有多个线程，可以共享进程的内存资源、全局变量等资源。

- 主线程
	- 执行UI绘制
	- 管理主线程的ArkTS引擎示例、使多个UIAbility组件能够运行在其之上。
	- 管理其他线程的ArkTS引擎实例，例如使用TaskPool（任务池）创建任务或取消任务、启动和终止Worker线程。
	- 分发交互事件。
	- 处理应用代码的回调，包括事件处理和生命周期管理。
	- 接受TaskPool以及Worker线程发送的消息。
- TaskPool线程
	- 用于执行耗时操作，支持设置调度优先级、负载均衡等功能，推荐使用。
- Worker线程
	- 用于执行耗时操作，支持线程间通信。
	- 建议用在运行时间超过3分钟的任务和有关联的一系列同步任务中。

在同一线程内可以用EventHub这一基于订阅者模式的事件通信机制来传递数据和状态，而跨线程的数据通信需要使用Emitter来进行线程间通信，跨进程的数据通信可以使用CES、IPC/rpc来进行进程间通信。