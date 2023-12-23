# Block

### 1. 本质

* block本质上也是一个OC对象，它内部也有个isa指针
* block是封装了函数调用以及函数调用环境的OC对象

<figure><img src=".gitbook/assets/截屏2023-12-23 22.14.50.png" alt=""><figcaption></figcaption></figure>



### 2. 变量捕获

为了保证block内部能够正常访问外部的变量，block有个变量捕获机制

<figure><img src=".gitbook/assets/截屏2023-12-23 22.16.32.png" alt=""><figcaption></figcaption></figure>

auto变量

<figure><img src=".gitbook/assets/截屏2023-12-23 22.17.35.png" alt=""><figcaption></figcaption></figure>

### 3. Block类型

<figure><img src=".gitbook/assets/截屏2023-12-23 22.18.27.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/截屏2023-12-23 22.19.49.png" alt=""><figcaption></figcaption></figure>

在ARC环境下，编译器会根据情况自动将栈上的block复制到堆上，比如以下情况

* block作为函数返回值时
* 将block赋值给\_\_strong指针时
* block作为Cocoa API中方法名含有usingBlock的方法参数时
* block作为GCD API的方法参数时

MRC下block属性的建议写法

```objectivec
@property (copy, nonatomic) void (^block)(void);
```

ARC下block属性的建议写法

```objectivec
@property (strong, nonatomic) void (^block)(void);
@property (copy, nonatomic) void (^block)(void);

```

### 4. 对象类型的auto变量

当block内部访问了对象类型的auto变量时

* 如果block是在栈上，将不会对auto变量产生强引用

如果block被拷贝到堆上

* 会调用block内部的copy函数
* copy函数内部会调用\_Block\_object\_assign函数
* \_Block\_object\_assign函数会根据auto变量的修饰符（\_\_strong、\_\_weak、\_\_unsafe\_unretained）做出相应的操作，形成强引用（retain）或者弱引用

如果block从堆上移除

* 会调用block内部的dispose函数
* dispose函数内部会调用\_Block\_object\_dispose函数
* \_Block\_object\_dispose函数会自动释放引用的auto变量（release）

<figure><img src=".gitbook/assets/截屏2023-12-23 22.25.01.png" alt=""><figcaption></figcaption></figure>

### 5. \_\_block

\_\_block可以用于解决block内部无法修改auto变量值的问题

\_\_block不能修饰全局变量、静态变量（static）

编译器会将\_\_block变量包装成一个对象

<figure><img src=".gitbook/assets/截屏2023-12-23 22.31.13.png" alt=""><figcaption></figcaption></figure>

\_\_block内存管理

当block在栈上时，并不会对\_\_block变量产生强引用

当block被copy到堆时

* 会调用block内部的copy函数
* copy函数内部会调用\_Block\_object\_assign函数
* \_Block\_object\_assign函数会对\_\_block变量形成强引用（retain）

<figure><img src=".gitbook/assets/截屏2023-12-23 22.32.37.png" alt=""><figcaption></figcaption></figure>

当block从堆中移除时

* 会调用block内部的dispose函数
* dispose函数内部会调用\_Block\_object\_dispose函数
* \_Block\_object\_dispose函数会自动释放引用的\_\_block变量（release）

<figure><img src=".gitbook/assets/截屏2023-12-23 22.33.32.png" alt=""><figcaption></figcaption></figure>

\_\_block的\_\_forwarding指针

<figure><img src=".gitbook/assets/截屏2023-12-23 22.34.17.png" alt=""><figcaption></figcaption></figure>

在 Objective-C 的 block 实现中，当一个 block 被复制到堆上时，它会有一个名为 forwarding 的指针。这个指针用于确保在访问 block 的变量时，始终使用正确的 block 实例。因为 block 可能在堆、栈或全局区域中，所以需要 forwarding 指针来统一处理访问方式。

在 block 的内部实现中，forwarding 指针初始指向 block 自身。这是因为在 block 创建时，它可能处于栈上，此时并不需要复制到堆上。当 block 被复制到堆上时，forwarding 指针会被更新为指向堆上的 block 实例，以确保后续访问都是使用堆上的实例。

总结一下，forwarding 指针之所以初始指向 block 自身，是因为在 block 创建时可能处于栈上，不需要复制到堆上。当 block 被复制到堆上时，forwarding 指针会被更新，确保后续访问都是使用堆上的实例。这样可以确保在访问 block 的变量时，始终使用正确的 block 实例。

### 6. 总结

block的原理是怎样的？本质是什么？

* 封装了函数调用以及调用环境的OC对象

\_\_block的作用是什么？有什么使用注意点？

在 Objective-C 中，block 会捕获其作用域内的变量，但默认情况下，这些变量在 block 内部是不可修改的，只能被访问。如果你想在 block 内部修改一个变量的值，那么就需要使用 \_\_block 修饰符。

使用 \_\_block 修饰符的注意点包括：

1. \_\_block 变量的生命周期：\_\_block 变量的生命周期会被延长，即使离开了它的作用域，只要还被 block 引用，它就不会被销毁。
2. 堆和栈的转移：\_\_block 变量最初是存储在栈上的，但当 block 被复制到堆上时，\_\_block 变量也会被复制到堆上。这个过程被称为 "Block 的 copy 操作"。
3. 内存管理：如果你在 ARC（Automatic Reference Counting）环境下使用 \_\_block 修饰的对象，需要注意避免循环引用。因为 \_\_block 修饰的对象在被 block 捕获后，block 会持有这个对象的强引用，如果这个对象又持有 block，就会形成循环引用，导致内存泄漏。
4. 多线程安全：\_\_block 变量在多线程环境下可能会有线程安全问题，因为多个线程可能会同时访问和修改这个变量，所以在使用时需要注意同步控制。

block的属性修饰词为什么是copy？使用block有哪些使用注意？

* block一旦没有进行copy操作，就不会在堆上
* 使用注意：循环引用问题

block在修改NSMutableArray，需不需要添加\_\_block？

当你在修改`NSMutableArray`时，通常不需要添加`__block`。`__block`关键字用于指示一个对象可以在块中被修改，它主要用于解决块内部对变量的修改不能影响到块外部的问题。

`NSMutableArray`本身就是可变的，这意味着你可以直接修改它的内容，而不需要使用`__block`关键字。但是，如果你想在块中修改一个指向`NSMutableArray`的指针，那么你需要使用`__block`关键字。

