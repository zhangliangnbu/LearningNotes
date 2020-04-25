#Android系统机制
---
###APP启动过程

1. Launcher线程捕获onclick的点击事件，调用Launcher.startActivitySafely，进一步调用Launcher.startActivity，最后调用父类Activity的startActivity。
2. Activity和ActivityManagerService交互，引入Instrumentation，将启动请求交给Instrumentation，调用Instrumentation.execStartActivity



# Android开机过程

* BootLoder引导,然后加载Linux内核.
* 0号进程init启动.加载init.rc配置文件,配置文件有个命令启动了zygote进程
* zygote开始fork出SystemServer进程
* SystemServer加载各种JNI库,然后init1,init2方法,init2方法中开启了新线程ServerThread.
* 在SystemServer中会创建一个socket客户端，后续AMS（ActivityManagerService）会通过此客户端和zygote通信
* ServerThread的run方法中开启了AMS,还孵化新进程ServiceManager,加载注册了一溜的服务,最后一句话进入loop 死循环
* run方法的SystemReady调用resumeTopActivityLocked打开锁屏界面





###Android内核解读-应用的安装过程

[http://blog.csdn.net/singwhatiwanna/article/details/19578947](http://blog.csdn.net/singwhatiwanna/article/details/19578947)
apk的安装过程分为两步：

1. 将apk文件复制到程序目录下(/data/app/)
2. 为应用创建数据目录(/data/data/package name/)、提取dex文件到指定目录(/data/delvik-cache/)、修改系统包管理信息。

4. (Result)
	返回的数据会作为参数传递到此方法中，可以利用返回的一些数据来进行一些UI操作。





