# Runtime

### 1. 介绍

* Objective-C是一门动态性比较强的编程语言，跟C、C++等语言有着很大的不同
* Objective-C的动态性是由Runtime API来支撑的
* Runtime API提供的接口基本都是C语言的，源码由C\C++\汇编语言编写

### 2. isa详解

* 要想学习Runtime，首先要了解它底层的一些常用数据结构，比如isa指针
* 在arm64架构之前，isa就是一个普通的指针，存储着Class、Meta-Class对象的内存地址
* 从arm64架构开始，对isa进行了优化，变成了一个共用体（union）结构，还使用位域来存储更多的信息

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

nonpointer

* 0，代表普通的指针，存储着Class、Meta-Class对象的内存地址
* 1，代表优化过，使用位域存储更多的信息

has\_assoc

* 是否有设置过关联对象，如果没有，释放时会更快

has\_xxx\_dtor

* 是否有C++的析构函数（.cxx\_destruct），如果没有，释放时会更快

shiftcls

* 存储着Class、Meta-Class对象的内存地址信息

magic

* 用于在调试时分辨对象是否未完成初始化

weakly\_referenced

* 是否有被弱引用指向过，如果没有，释放时会更快

deallocating

* 对象是否正在释放

extra\_rc

* 里面存储的值是引用计数器减1

has\_sidetable\_rc

* 引用计数器是否过大无法存储在isa中
* 如果为1，那么引用计数会存储在一个叫SideTable的类的属性中



### 3. Class结构详解

<figure><img src=".gitbook/assets/截屏2023-12-23 22.47.29.png" alt=""><figcaption></figcaption></figure>

`class_rw_t`里面的`methods`、`properties`、`protocols`是二维数组，是可读可写的，包含了类的初始内容、分类的内容

<figure><img src=".gitbook/assets/截屏2023-12-23 22.49.04.png" alt=""><figcaption></figcaption></figure>

`class_ro_t`里面的`baseMethodList`、`baseProtocols`、`ivars`、`baseProperties`是一维数组，是只读的，包含了类的初始内容

<figure><img src=".gitbook/assets/截屏2023-12-23 22.49.34.png" alt=""><figcaption></figcaption></figure>

`method_t`是对方法\函数的封装

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

`IMP`代表函数的具体实现

<figure><img src=".gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

SEL代表方法\函数名，一般叫做选择器，底层结构跟char \*类似

* 可以通过@selector()和sel\_registerName()获得
* 可以通过sel\_getName()和NSStringFromSelector()转成字符串
* 不同类中相同名字的方法，所对应的方法选择器是相同的

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Class内部结构中有个方法缓存（cache\_t），用散列表（哈希表）来缓存曾经调用过的方法，可以提高方法的查找速度

<figure><img src=".gitbook/assets/截屏2023-12-23 22.51.52.png" alt=""><figcaption></figcaption></figure>

### 4. objc\_msgSend流程

消息发送

<figure><img src=".gitbook/assets/截屏2023-12-23 22.53.50.png" alt=""><figcaption></figcaption></figure>

动态方法解析（动态添加方法）

<figure><img src=".gitbook/assets/截屏2023-12-23 22.54.26.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/截屏2023-12-23 22.55.10.png" alt=""><figcaption></figcaption></figure>

消息转发

<figure><img src=".gitbook/assets/截屏2023-12-23 22.56.38.png" alt=""><figcaption></figcaption></figure>

### 5. 应用

查看私有成员变量



遍历类的所有成员变量（修改textfield的占位文字颜色、字典转模型、自动归档解档）



替换方法实现（Hook）



利用关联对象（AssociatedObject）给分类添加属性



### 6. 总结

讲一下 OC 的消息机制

* OC中的方法调用其实都是转成了objc\_msgSend函数的调用，给receiver（方法调用者）发送了一条消息（selector方法名）
* objc\_msgSend底层有3大阶段
  * 消息发送（当前类、父类中查找）、动态方法解析、消息转发











