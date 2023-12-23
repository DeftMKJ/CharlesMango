---
description: RunLoop
---

# RunLoop

### 1. 介绍

大神：[https://blog.ibireme.com/2015/05/18/runloop/](https://blog.ibireme.com/2015/05/18/runloop/)

一般来讲，一个线程一次只能执行一个任务，执行完成后线程就会退出。如果我们需要一个机制，让线程能随时处理事件但并不退出，通常的代码逻辑是这样的：

```javascript
function loop() {
    initialize();
    do {
        var message = get_next_message();
        process_message(message);
    } while (message != quit);
}
```

这种模型通常被称作 [Event Loop](http://en.wikipedia.org/wiki/Event\_loop)。 Event Loop 在很多系统和框架里都有实现，比如 Node.js 的事件处理，比如 Windows 程序的消息循环，再比如 OSX/iOS 里的 RunLoop。实现这种模型的关键点在于：如何管理事件/消息，如何让线程在没有处理消息时休眠以避免资源占用、在有消息到来时立刻被唤醒。

所以，RunLoop 实际上就是一个对象，这个对象管理了其需要处理的事件和消息，并提供了一个入口函数来执行上面 Event Loop 的逻辑。线程执行了这个函数后，就会一直处于这个函数内部 “接受消息->等待->处理” 的循环中，直到这个循环结束（比如传入 quit 的消息），函数返回。

<figure><img src=".gitbook/assets/截屏2023-12-23 23.03.29.png" alt=""><figcaption></figcaption></figure>

### 2. Runloop对象

* Foundation：NSRunLoop
* Core Foundation：CFRunLoopRef

NSRunLoop和CFRunLoopRef都代表着RunLoop对象

NSRunLoop是基于CFRunLoopRef的一层OC包装

CFRunLoopRef是开源的

[https://opensource.apple.com/tarballs/CF/](https://opensource.apple.com/tarballs/CF/)



### 3. Runloop和线程

* 每条线程都有唯一的一个与之对应的RunLoop对象
* RunLoop保存在一个全局的Dictionary里，线程作为key，RunLoop作为value
* 线程刚创建时并没有RunLoop对象，RunLoop会在第一次获取它时创建
* RunLoop会在线程结束时销毁
* 主线程的RunLoop已经自动获取（创建），子线程默认没有开启RunLoop



### 4. Runloop相关类

Core Foundation中关于RunLoop的5个类

* CFRunLoopRef
* CFRunLoopModeRef
* CFRunLoopSourceRef
* CFRunLoopTimerRef
* CFRunLoopObserverRef

<figure><img src=".gitbook/assets/截屏2023-12-23 23.06.24.png" alt=""><figcaption></figcaption></figure>

* CFRunLoopModeRef代表RunLoop的运行模式
* 一个RunLoop包含若干个Mode，每个Mode又包含若干个Source0/Source1/Timer/Observer
* RunLoop启动时只能选择其中一个Mode，作为currentMode
* 如果需要切换Mode，只能退出当前Loop，再重新选择一个Mode进入
* 不同组的Source0/Source1/Timer/Observer能分隔开来，互不影响

常见的2种Mode

* kCFRunLoopDefaultMode（NSDefaultRunLoopMode）：App的默认Mode，通常主线程是在这个Mode下运行
* UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响



### 5. 运行逻辑

* Source0 触摸事件处理 performSelector:onThread:
* Source1 基于Port的线程间通信 系统事件捕捉
* Timers NSTimer performSelector:withObject:afterDelay:
* Observers 用于监听RunLoop的状态 UI刷新（BeforeWaiting） Autorelease pool（BeforeWaiting）

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### 6. 休眠原理

为了实现消息的发送和接收，mach\_msg() 函数实际上是调用了一个 Mach 陷阱 (trap)，即函数mach\_msg\_trap()，陷阱这个概念在 Mach 中等同于系统调用。当你在用户态调用 mach\_msg\_trap() 时会触发陷阱机制，切换到内核态；内核态中内核实现的 mach\_msg() 函数会完成实际的工作，如下图：

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

这些概念可以参考维基百科: [System\_call](http://en.wikipedia.org/wiki/System\_call)、[Trap\_(computing)](http://en.wikipedia.org/wiki/Trap\_\(computing\))。

RunLoop 的核心就是一个 mach\_msg() (见上面代码的第7步)，RunLoop 调用这个函数去接收消息，如果没有别人发送 port 消息过来，内核会将线程置于等待状态。例如你在模拟器里跑起一个 iOS 的 App，然后在 App 静止时点击暂停，你会看到主线程调用栈是停留在 mach\_msg\_trap() 这个地方。

关于具体的如何利用 mach port 发送信息，可以看看[ NSHipster 这一篇文章](http://nshipster.com/inter-process-communication/)，或者[这里](http://segmentfault.com/a/1190000002400329)的中文翻译 。

关于Mach的历史可以看看这篇很有趣的文章：[Mac OS X 背后的故事（三）Mach 之父 Avie Tevanian](http://www.programmer.com.cn/8121/)。



### 7. 应用

* 控制线程生命周期（线程保活）
* 解决NSTimer在滑动时停止工作的问题
* 监控应用卡顿&#x20;
  1. 创建一个子线程并启动 RunLoop，用于监听主线程的 RunLoop 状态。
  2. 在子线程中添加一个定时器，用于定时检查主线程的 RunLoop 状态。
  3. 使用 CFRunLoopObserver 监听主线程的 RunLoop 状态变化，并记录状态变化的时间。
  4. 当检测到主线程的 RunLoop 状态长时间处于等待或执行状态时，可以认为发生了卡顿。
* 性能优化（异步排版布局绘制，最后的提交参考CAAnimation，监听Observer，注册一个比CoreAnimation低一点优先级的任务同步异步的任务到主线程）

### 8. 总结

* 讲讲 RunLoop，项目中有用到吗？
  * 应用里面提了
* runloop内部实现逻辑？
* runloop和线程的关系？
* timer 与 runloop 的关系？
* 程序中添加每3秒响应一次的NSTimer，当拖动tableview时timer可能无法响应要怎么解决？
* runloop 是怎么响应用户操作的， 具体流程是什么样的？
  *   苹果注册了一个 Source1 (基于 mach port 的) 用来接收系统事件，其回调函数为 \_\_IOHIDEventSystemClientQueueCallback()。

      当一个硬件事件(触摸/锁屏/摇晃等)发生后，首先由 IOKit.framework 生成一个 IOHIDEvent 事件并由 SpringBoard 接收。这个过程的详细情况可以参考[这里](http://iphonedevwiki.net/index.php/IOHIDFamily)。SpringBoard 只接收按键(锁屏/静音等)，触摸，加速，接近传感器等几种 Event，随后用 mach port 转发给需要的App进程。随后苹果注册的那个 Source1 就会触发回调，并调用 \_UIApplicationHandleEventQueue() 进行应用内部的分发。

      \_UIApplicationHandleEventQueue() 会把 IOHIDEvent 处理并包装成 UIEvent 进行处理或分发，其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的。
* 说说runLoop的几种状态
* runloop的mode作用是什么？

多读几遍：[https://blog.ibireme.com/2015/05/18/runloop/](https://blog.ibireme.com/2015/05/18/runloop/)

