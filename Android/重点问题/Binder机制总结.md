



## 简介

Binder到底是什么？中文即 粘合剂，意思为粘合了两个不同的进程。

网上有很多对Binder的定义，但都说不清楚：Binder是跨进程通信方式、它实现了IBinder接口，是连接 ServiceManager的桥梁blabla，估计大家都看晕了，没法很好的理解。

我认为：对于Binder的定义，在不同场景下其定义不同。即：

- 机制角度： Binder是一种在Android中实现跨进程通信的方式；
- 模型角度：Binder是一种虚拟的物理设备驱动；
- 源码实现角度：Binder是一个类，实现了IBinder接口。



## 知识储备

在讲解Binder前，我们先了解一些Linux的基础知识

**进程空间划分**
一个进程空间分为 用户空间 & 内核空间（Kernel），即把进程内 用户 & 内核 隔离开来。

进程内，用户空间 & 内核空间 进行交互 需通过 系统调用，主要通过函数：

- copy_from_user（）：将用户空间的数据拷贝到内核空间
- copy_to_user（）：将内核空间的数据拷贝到用户空间

进程间，用户空间的数据不可共享，所以用户空间 = 不可共享空间；进程间，内核空间的数据可共享，所以内核空间 = 可共享空间。所有进程共用1个内核空间。


**进程隔离 & 跨进程通信（ IPC ）& 内存映射**

进程隔离。为了保证 安全性 & 独立性，一个进程 不能直接操作或者访问另一个进程，即Android的进程是相互独立、隔离的。

传统跨进程通信（ IPC ）。即进程间需进行数据交互、通信。跨进程通信的基本原理：

![示意图](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS0xMjkzNTY4NGU4ZWMxMDdjLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)

> a. 而Binder的作用则是：连接 两个进程，实现了mmap()系统调用，主要负责 创建数据接收的缓存空间 & 管理数据接收缓存
> b. 注：传统的跨进程通信需拷贝数据2次，但Binder机制只需1次，主要是使用到了内存映射，具体下面会详细说明

内存映射方式实现IPC。高效，数据拷贝只有一次。

> 具体请看文章：[操作系统：图文详解 内存映射](https://www.jianshu.com/p/719fc4758813)

![](https://upload-images.jianshu.io/upload_images/944365-df2a3cb545cb59ea.png)





## Binder 跨进程通信机制 模型

3.1 模型原理图
Binder 跨进程通信机制模型基于 Client - Server 模式，底层原理为内存映射。其中，Binder作为一种虚拟设备驱动，是链接Service进程、Client进程和ServiceManager进程的桥梁，其作用有：

- 通过内存映射传递进程间数据；
- 通过Binder线程池进行自身管理。

Binder跨进程通信的模型和过程说明如下：

![img](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1kM2M3OGIxOTNjM2U4YTM4LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)


说明1：Client进程、Server进程 & Service Manager 进程之间的交互 都必须通过Binder驱动（使用 open 和 ioctl文件操作函数），而非直接交互。原因：Client进程、Server进程 & Service Manager进程属于进程空间的用户空间，不可进行进程间交互。Binder驱动 属于 进程空间的 内核空间，可进行进程间 & 进程内交互。所以，原理图可表示为以下：

虚线表示并非直接交互

![示意图](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS1iNDcwMDhhMDkyNjViOWM2LnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)

说明2： Binder驱动 & Service Manager进程 属于 Android基础架构（即系统已经实现好了）；而Client 进程 和 Server 进程 属于Android应用层（需要开发者自己实现）。所以，在进行跨进程通信时，开发者只需自定义Client & Server 进程 并 显式使用上述3个步骤，最终借助 Android的基本架构功能就可完成进程间通信。





## Binder机制 在Android中的具体实现原理

Binder机制在 Android中的实现主要依靠 Binder类，其实现了IBinder 接口。下面会详细说明

- **IBinder** : IBinder 是一个接口，代表了一种跨进程通信的能力。只要实现了这个借口，这个对象就能跨进程传输。
- **IInterface** : IInterface 代表的就是 Server 进程对象具备什么样的能力（能提供哪些方法，其实对应的就是 AIDL 文件中定义的接口）
- **Binder** : Java 层的 Binder 类，代表的其实就是 Binder 本地对象。BinderProxy 类是 Binder 类的一个内部类，它代表远程进程的 Binder 对象的本地代理；这两个类都继承自 IBinder, 因而都具有跨进程传输的能力；实际上，在跨越进程的时候，Binder 驱动会自动完成这两个对象的转换。
- **Stub** : AIDL 的时候，编译工具会给我们生成一个名为 Stub 的静态内部类；这个类继承了 Binder, 说明它是一个 Binder 本地对象，它实现了 IInterface 接口，表明它具有 Server 承诺给 Client 的能力；Stub 是一个抽象类，具体的 IInterface 的相关实现需要开发者自己实现。

实例说明：Client进程 需要调用 Server进程的加法函数（将整数a和b相加），略过client和Service的创建。

```java
// 有点问题，报错Binder invocation to an incorrect interface，不知道怎么解决！    
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(com.liang.androidskilldemo.R.layout.activity_binder_demo)

        // binder 实现原理
        //
        // service
        val binder = Stub()
        val plus = object : IPlus {
            override fun add(a: Int, b: Int): Int {
                return a + b
            }

            override fun asBinder(): IBinder? {
                return null
            }
        }
        binder.attachInterface(plus,I_NAME)

        // client
        // create data
        val data = android.os.Parcel.obtain()
        val reply = android.os.Parcel.obtain()
        try {
            data.writeInt(1)
            data.writeInt(2)
            data.writeInterfaceToken(I_NAME)

            // 实际代码应该使用proxy, 但最终会调用Binder#transact方法
            binder.transact(Stub_add, data, reply, 0)
            reply.readException()
            val result = reply.readInt()
            Log.d("result -> ", result.toString())
        } finally {
            reply.recycle()
            data.recycle()
        }

    }
    

public interface IPlus extends IInterface {
		public int add(int a,int b);
}

public class Stub extends Binder {

    @Override
    protected boolean onTransact(int code, @NonNull Parcel data, @Nullable Parcel reply, int flags) throws RemoteException {
        switch (code) {
            case Stub_add: {
                data.enforceInterface(I_NAME);
                int  arg0  = data.readInt();
                int  arg1  = data.readInt();
                int  result = ((IPlus)this.queryLocalInterface(I_NAME)).add(arg0,  arg1);
                reply.writeInt(result);
                return true;
            }
        }
        return super.onTransact(code, data, reply, flags);
    }

    public static final int Stub_add = 0;

    public static final String I_NAME = "add_two_int";
}

```



一个完整的示例可以见：https://github.com/BaronZ88/HelloBinder







## 参考

- [Android跨进程通信：图文详解 Binder机制 原理](https://blog.csdn.net/carson_ho/article/details/73560642)
- [操作系统：图文详解 内存映射](https://www.jianshu.com/p/719fc4758813)
- [写给 Android 应用工程师的 Binder 原理剖析](https://zhuanlan.zhihu.com/p/35519585)