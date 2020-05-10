## 简介

WMS 的职责如下：

![](https://upload-images.jianshu.io/upload_images/2828107-9ddff2816fe1805e.png)

窗口管理。**WMS** 是窗口的管理者，它负责窗口的启动、添加和删除。另外窗口的大小和层级也是由 **WMS** 进行管理的。窗口管理的核心成员有 **DisplayContent**、**WindowToken** 和 **WindowState**.

窗口动画。窗口间进行切换时，使用动画可以显得更炫一些，窗口动画由 **WMS** 的动画子系统来负责，动画子系统的管理者为 **WindowAnimator**。

输入系统中转站。通过对窗口的触摸从而产生触摸事件，**InputManagerService(IMS)** 会对触摸事件进行处理，它会寻找一个最合适的窗口来处理触摸反馈信息，**WMS** 是窗口的管理者，它作为输入系统的中转站再合适不过了。

Surface 管理。窗口不具备绘制功能，因此每个窗口都需要有一块 **Surface** 来供自己绘制，为每个窗口分配 **Surface** 是由**WMS** 来完成的。



## 视图创建流程

Activity 创建 Window 并添加 View 的流程大致如下：

Activity的`attach`方法中，在此方法内将为 Activity 创建一个 Window，此时 Window 内还是空白的。

Activity 的`onCreate`方法中，创建 DecorView，并将我们传递过来的 View 添加到 DecorView 的内容栏中，而此时，DecorView 和 Window 两者还是处于分离的状态。

![](https://upload-images.jianshu.io/upload_images/2828107-7a90c03a644a3a1f.png)

Activity 的`onResume`方法中，ViewRootImpl执行 requestlayout()完成view的绘制流程，并通过WindowSession将View和InputChannel添加到WmS中，从而将View添加到Window上并且接收触摸事件。这是一次IPC 过程。

此时就完成了 Activity Window 的创建和 View 的添加过程。

这个IPC过程中WMS的作用：

- 对所有要添加的窗口进行检查，如果窗口不满足一些条件，就不会再执行下面的代码逻辑。
- WindowToken** 相关的处理，比如有的窗口类型需要提供 WindowToken , 没有提供的话就不会执行下面的代码逻辑，有的窗口类型则需要有 WMS 隐式创建 WindowToken.
- WindowState** 的创建和相关处理，将 **WindowToken** 和 **WindowState** 相关联。
- 创建和配置 **DisplayContent**, 完成窗口添加到系统前的准备工作。



![img](https://upload-images.jianshu.io/upload_images/2828107-3f93849f3f3f70c2.png)



## 几个重要关系

**Activity & Window & View**

Activity作为一个生命周期的管理者，和其上的视图View进行了解耦。引入Window的概念，它是个抽象类，对于Activity来说，它的具体实现类是PhoneWindow，在Activity执行attach的时候，会创建一个PhoneWindow对象。PhoneWindow作为装载根视图DecorView的顶级容器，Activity通过setContentView实际上是调用PhoneWindow来创建DecorView，并解析xml布局加载到DecorView的contentView部分。

![](https://upload-images.jianshu.io/upload_images/2828107-6dcd9a3be935afd2.png)



**Activity & Window & View & WM & WMS关系**

WindowManager是一个接口类，继承自接口ViewManager，负责窗口的管理(增、删、改)。它的实现类是WindowManagerImpl，而具体操作实际上又会交给WindowManagerGlobal来处理，它是个单例，进程唯一。WindowManagerGlobal执行addView的方法中会传入DecorView, 还会初始化一个ViewRootImpl。WindowManagerGlobal因为是单例的，它内部会有两个List来分别保存这两个对象，来统一管理。

<img src="https://upload-images.jianshu.io/upload_images/2828107-490351404c329a15.png" width="600">



**ViewRootImpl & WindowManagerService**

WindowManagerGlobal负责对DecorView和对应的ViewRootImpl进行统一管理，而具体功能是由ViewRootImpl来处理。以addView为例，具体window是由WMS统一管理的，所以这里会进行binder IPC。

IWindowSession: 应用程序通过Session与WMS通信，并且每个应用程序进程都会对应一个Session。

IWindow: 作为WMS主动与应用程序通信的client端，因为不同的Window是不同的client，因此它也被作为识别window的key。

<img src="https://upload-images.jianshu.io/upload_images/2828107-ca7ce65d2cab4ed6.png" width="600">





## 窗口管理相关

Activity和Window通过Token关联

窗口属性与类型

窗口组织方式（分组和分层）

参考：https://www.jianshu.com/p/3b5b6f2469d8



## WMS创建流程



## 参考

- https://juejin.im/post/5dcab476f265da4d0a68e3ab
- https://juejin.im/post/5dcab476f265da4d0a68e3ab