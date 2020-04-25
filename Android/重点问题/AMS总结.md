简介

AMS是Android中**最核心的服务**，主要负责系统中四大组件的**启动、切换、调度及应用进程的管理和调度**等工作，其职责与操作系统中的进程管理和调度模块相类似。



## App和Activity的启动

以am命令启动一个Activity为例，分析应用进程的创建、Activity的启动，以及它们和AMS之间的交互等知识



<img src="https://upload-images.jianshu.io/upload_images/11462765-b891c3c6cd18ecc0.jpg" width="720">



**启动流程**：

1. 点击桌面App图标，Launcher进程采用Binder IPC向system_server进程发起startActivity请求；
2. system_server进程接收到请求后，向zygote进程发送创建进程的请求；
3. Zygote进程fork出新的子进程，即App进程；
4. App进程，通过Binder IPC向sytem_server进程发起attachApplication请求；
5. system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送scheduleLaunchActivity请求；
6. App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息；
7. 主线程在收到Message后，通过反射机制创建目标Activity，并回调Activity.onCreate()等方法。  到此，App便正式启动，开始进入Activity生命周期，执行完onCreate/onStart/onResume方法，UI渲染结束后便可以看到App的主界面。



详细流程

Activity启动分为两种，根Activity和普通Activity。根Activity启动我将它分为7步。

第一步：Launcher 通过binder机制请求AMS启动Activity

这里面有个“监视”思想值得我们学习，我们组件化开发的时候可以给我们各个组件加一个监视类，来监视组件的交互。Instrumentation主要监视应用程序和系统的交互，这里的系统包含Framework层，也就是说应用程序和系统的交互都需要经过这个类。这一步涉及到两个进程Launcher进程和system_server进程，通过Binder机制通信。Binder机制是C/S模型的，此处 Launcher是C端，AMS是S端。

第二步：配置Activity的信息。AMS接收到请求后通过ActivityStarter解析Intent和Flag，在通过ActivityStack为Activity配置栈信息。并判断是否需要创建应用进程。

第三步：配置应用进程信息并通过Socket请求Zygote创建子进程。通过系统进程管理类Process配置应用进程信息，这一步涉及到连个进程system_server进程和 Zygote进程，通过Socket通信。

第四步：创建应用进程。（启动进程）ZygoteServer接收到AMS的Socket请求，通过ZygoteConnection fork应用进程。fork进程后会返回一个pid，通过判断pid进入到应用进程。Zygote进程在初始化的的时候就创建了Socket并Loop等待AMS的请求。

第五步：应用进程初始化（进程准备工作，进程Loop） 初始化Binder线程池（通过ZygoteInit），初始化运行时环境（通过RunntimeInit）。通过ActivityThread创建主线程，创建H类，开始Loop循环。并通过Binder向AMS发送绑定Application的请求。

第六步：绑定Application AMS接收到ActivityThread发送的请求后，把Application和进程进程绑定（这就是每个进程只有一个Application的原因）。最后通过Binder机制请求启动Activity。

第七步：启动Activity ApplicationThread接收到请求后调用ActivityThread#performLaunchActivity来间接调用Activity的onCreate方法。这里有个注意点ActivityThread调用Activity也通过了Instrumention。








Broadcast和Service的启动



ContentProvider



Crash





## 参考

- https://www.jianshu.com/p/480750598ce9
- https://juejin.im/post/5d88db38f265da03f565244d
  

