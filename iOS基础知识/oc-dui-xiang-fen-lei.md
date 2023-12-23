# OC对象分类

### 1. instance对象

<figure><img src=".gitbook/assets/截屏2023-12-23 20.20.38.png" alt=""><figcaption></figcaption></figure>



### 2. class对象

<figure><img src=".gitbook/assets/截屏2023-12-23 20.17.37.png" alt=""><figcaption></figcaption></figure>

### 3. meta-class对象

<figure><img src=".gitbook/assets/截屏2023-12-23 20.18.07.png" alt=""><figcaption></figcaption></figure>

### 4. isa指针

<figure><img src=".gitbook/assets/截屏2023-12-23 20.21.53.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/截屏2023-12-23 20.23.33.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/截屏2023-12-23 20.24.02.png" alt=""><figcaption></figcaption></figure>



### 5. objc\_class结构

`class,meta-class`对象的本质结构都是`struct objc_class`

<figure><img src=".gitbook/assets/截屏2023-12-23 20.37.10.png" alt=""><figcaption></figcaption></figure>

主结构体：

* isa指针
* super class指针
* cache方法缓存
* bits用于获取具体的类信息（通过位运算获取到rw结构体）

rw结构体

* ro结构体 （编译期间就决定了类的内存布局）
* methods方法列表 （categroy扩展添加）
* properties属性列表 （同上）
* protocols协议列表 （同上）

ro结构体

* instanceSize instance对象占用的内存空间
* name 类名
* baseMethodList 方法列表
* baseProtocols协议列表
* ivars成员变量列表





### 5. 总结

<figure><img src=".gitbook/assets/截屏2023-12-23 20.26.25.png" alt=""><figcaption></figcaption></figure>

对象的isa指针指向哪里？

* instance对象的的isa指针指向class对象
* class对象的isa指向meta-class对象
* meta-class对象的isa指向基类的meta-class对象

OC的类信息存放在哪里？

* 对象方法、属性、成员变量和协议信息存放在class对象中
* 类方法、存放在meta-class对象中
* 成员变量的具体指，存放在instance对象
