## 简介

Handler消息机制比较简洁：

- 一个线程通过`ThreadLocal`维护一个`Looper`实例，Looper维护一个MessageQueue；
- 每个Handler持有当前线程维护的`Looper`实例；
- Handler发送消息，消息被压进消息队列，并被`Looper.loop`维护、发送;
- 最后每个消息持有Handler引用，调用`Handler#dispatchMessage(Message msg)`发送消息，并最终`Handler#handleMessage(Message msg)`处理。

简单示例：

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        new Thread(new Runnable() {
            @Override
            public void run() {
              	// 创建当前线程对应的Looper实例
                Looper.prepare();
              	// 创建Handler，Handler与当前线程的Looper联系起来
                Handler handler = new Handler() {
                    @Override
                    public void handleMessage(Message msg) {
                        super.handleMessage(msg);
                        Log.d(Thread.currentThread().getName(), "---" + msg.what);
                    }
                };
              	// 发送消息，消息持有Handler引用
                handler.sendEmptyMessage(1);
              	// 循环维护消息
                Looper.loop();
            }
        }).start();
    }
```





## 主线程是如何处理消息的？

Looper.loop()在主线程里运行，这样主线程才不会退出，程序才处于活跃状态。

事件是通过其他线程调用主线程的Handler发送消息到主线程，让其处理的。

![img](https://pic4.zhimg.com/80/7fb8728164975ac86a2b0b886de2b872_720w.jpg)







