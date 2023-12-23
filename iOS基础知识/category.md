# Category

### 1. 底层结构

<figure><img src=".gitbook/assets/截屏2023-12-23 21.07.51.png" alt=""><figcaption></figcaption></figure>

### 2. 加载处理过程

1. 一个Category对应上面一个结构体，main函数之前，通过Runtime加载某个类的所有Category数据
2. 把所有Category的方法、属性、协议数据，合并到一个大数组中\
   后面参与编译的Category数据，会在数组的前面
3. 将合并后的分类数据（方法、属性、协议），插入到类原来数据的前面

这里引入一下启动流程：

{% embed url="https://mp.weixin.qq.com/s?__biz=MzI1MzYzMjE0MQ==&amp;mid=2247486932&amp;idx=1&amp;sn=eb4d294e00375d506b93a00b535c6b05&amp;scene=21#wechat_redirect" %}

1. 点击图标，创建进程
2. mmap 主二进制，找到 dyld 的路径
3. mmap dyld，把入口地址设为`_dyld_start`
4. 重启手机/更新/下载 App 的第一次启动，会创建启动闭包
5. 把没有加载的动态库 mmap 进来，动态库的数量会影响这个阶段
6. 对每个二进制做 bind 和 rebase，主要耗时在 Page In，影响 Page In 数量的是 objc 的元数据
7. `初始化 objc 的 runtime，由于闭包已经初始化了大部分，这里只会注册 sel 和装载 category`
8. \+load 和静态初始化被调用，除了方法本身耗时，这里还会引起大量 Page In
9. 初始化 `UIApplication`，启动 Main Runloop
10. 执行 `will/didFinishLaunch`，这里主要是业务代码耗时
11. Layout，`viewDidLoad` 和`Layoutsubviews` 会在这里调用，`Autolayout` 太多会影响这部分时间
12. Display，`drawRect` 会调用
13. Prepare，图片解码发生在这一步
14. Commit，首帧渲染数据打包发给 RenderServer，启动结束

因此上面提到的Runtime加载过程，是在第七步完成的



### 3. “添加成员变量”

默认情况下，因为分类底层结构的限制，不能添加成员变量到分类中。但可以通过关联对象来间接实现

```objectivec
添加关联对象
void objc_setAssociatedObject(id object, const void * key,
                                id value, objc_AssociationPolicy policy)

获得关联对象
id objc_getAssociatedObject(id object, const void * key)

移除所有的关联对象
void objc_removeAssociatedObjects(id object)

static void *MyKey = &MyKey;
objc_setAssociatedObject(obj, MyKey, value, OBJC_ASSOCIATION_RETAIN_NONATOMIC)
objc_getAssociatedObject(obj, MyKey)

static char MyKey;
objc_setAssociatedObject(obj, &MyKey, value, OBJC_ASSOCIATION_RETAIN_NONATOMIC)
objc_getAssociatedObject(obj, &MyKey)

使用属性名作为key
objc_setAssociatedObject(obj, @"property", value, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
objc_getAssociatedObject(obj, @"property");

使用get方法的@selecor作为key
objc_setAssociatedObject(obj, @selector(getter), value, OBJC_ASSOCIATION_RETAIN_NONATOMIC)
objc_getAssociatedObject(obj, @selector(getter))
```

| objc\_AssociationPolicy              | 对应的修饰符            |
| ------------------------------------ | ----------------- |
| OBJC\_ASSOCIATION\_ASSIGN            | assign            |
| OBJC\_ASSOCIATION\_RETAIN\_NONATOMIC | strong, nonatomic |
| OBJC\_ASSOCIATION\_COPY\_NONATOMIC   | copy, nonatomic   |
| OBJC\_ASSOCIATION\_RETAIN            | strong, atomic    |
| OBJC\_ASSOCIATION\_COPY              | copy, atomic      |

#### 3.1 添加原理

实现关联对象技术的核心对象有

* AssociationsManager
* AssociationsHashMap
* ObjectAssociationMap
* ObjcAssociation

<figure><img src=".gitbook/assets/截屏2023-12-23 22.09.29.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/截屏2023-12-23 22.10.22.png" alt=""><figcaption></figcaption></figure>

### 4. 总结

Category的使用场景是什么？

* 给已有的类扩展方法或者添加属性（属性指生成get和setter，成员变量得用关联的方式添加）

Category的实现原理？

* Category编译之后的底层结构是struct category\_t，里面存储着分类的对象方法、类方法、属性、协议信息
* 在程序运行的时候第七步，runtime会将Category的数据，合并到类信息中（类对象、元类对象中）

Category和Class Extension的区别是什么？

* Class Extension在编译的时候，它的数据就已经包含在类信息中
* Category是在运行时，才会将数据合并到类信息中

Category中有load方法吗？load方法是什么时候调用的？load 方法能继承吗？

* 有load方法
* load方法在runtime加载类、分类的时候调用
* load方法可以继承，但是一般情况下不会主动去调用load方法，都是让系统自动调用

Category能否添加成员变量？如果可以，如何给Category添加成员变量？

* 不能直接给Category添加成员变量，但是可以间接实现Category有成员变量的效果
