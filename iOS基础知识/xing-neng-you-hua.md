# 性能优化

### 1. CPU和GPU

屏幕成像过程中，CPU和GPU起着至关重要的作用

CPU：对象的创建和销毁，属性的调整，布局计算，文本计算和排版，图片的格式转换，编解码、图像的绘制

GPU：纹理的渲染

<figure><img src=".gitbook/assets/截屏2023-12-24 17.41.44.png" alt=""><figcaption></figcaption></figure>

iOS采用的是双缓冲机制，有前帧缓存和后帧缓存



### 2. 卡顿优化

#### 2.1 产生原因

<figure><img src=".gitbook/assets/截屏2023-12-24 17.43.02.png" alt=""><figcaption></figcaption></figure>

尽量减少CPU和GPU在主线程的消耗

按照60FPS来说，每16ms就会有一次VSync信号



#### 2.2 优化CPU

尽量用轻量级的对象，比如用不到事件处理的地方，可以考虑使用CALayer取代UIView

不要频繁地调用UIView的相关属性，比如frame、bounds、transform等属性，尽量减少不必要的修改

尽量提前计算好布局，在有需要时一次性调整对应的属性，不要多次修改属性

Autolayout会比直接设置frame消耗更多的CPU资源

图片的size最好刚好跟UIImageView的size保持一致

控制一下线程的最大并发数量

尽量把耗时的操作放到子线程

* 文本处理（尺寸计算、绘制）
* 图片处理（解码、绘制）

#### 2.3 优化GPU

尽量避免短时间内大量图片的显示，尽可能将多张图片合成一张进行显示

GPU能处理的最大纹理尺寸是4096x4096，一旦超过这个尺寸，就会占用CPU资源进行处理，所以纹理尽量不要超过这个尺寸

尽量减少视图数量和层次

减少透明的视图（alpha<1），不透明的就设置opaque为YES

尽量避免出现离屏渲染



#### 2.4 离屏渲染

在OpenGL中，GPU有2种渲染方式

* On-Screen Rendering：当前屏幕渲染，在当前用于显示的屏幕缓冲区进行渲染操作
* Off-Screen Rendering：离屏渲染，在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作

离屏渲染消耗性能的原因

* 需要创建新的缓冲区
* 离屏渲染的整个过程，需要多次切换上下文环境，先是从当前屏幕（On-Screen）切换到离屏（Off-Screen）；等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上，又需要将上下文环境从离屏切换到当前屏幕

哪些操作会触发离屏渲染？

* 光栅化 `layer.shouldRasterize = YES`
* 遮罩 `layer.mask`
* 圆角 同时设置layer.masksToBounds = YES、layer.cornerRadius大于0
* 阴影  `layer.shadowXXX`

#### 2.5 卡顿检测

* 创建一个子线程并启动 RunLoop，用于监听主线程的 RunLoop 状态。
* 在子线程中添加一个定时器，用于定时检查主线程的 RunLoop 状态。
* 使用 CFRunLoopObserver 监听主线程的 RunLoop 状态变化，并记录状态变化的时间。
* 一旦发现进入睡眠前的 kCFRunLoopBeforeSources 状态，或者唤醒后的状态 kCFRunLoopAfterWaiting，在设置的时间阈值内一直没有变化，即可判定为卡顿。

[https://alexcode2.gitbook.io/ios-development-guidelines/ios-za-tan/ru-he-li-yong-runloop-yuan-li-qu-jian-kong-ka-dun](https://alexcode2.gitbook.io/ios-development-guidelines/ios-za-tan/ru-he-li-yong-runloop-yuan-li-qu-jian-kong-ka-dun)



### 3. 总结

* 你在项目中是怎么优化内存的？



* 优化你是从哪几方面着手？



* 列表卡顿的原因可能有哪些？你平时是怎么优化的？



* 遇到tableView卡顿嘛？会造成卡顿的原因大致有哪些？

