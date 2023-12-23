# OC对象本质



## 1. Objective-C的

我们平时编写的OC代码，底层实现本质都是C\C++代码，所有OC的面向对象都是基于C\C++的数据结构实现的

```objectivec
// OC 代码
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSObject *object = [[NSObject alloc] init];
    }
    return 0;
}
```

```sh
# 转换
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m -o main.cpp
```

```cpp
// C++ 代码
struct NSObject_IMPL {
    Class isa; // isa是指向结构体的指针，64bit下占8个字节
};
typedef struct objc_class *Class;
```



## 2. 一个对象在内存中的布局

<figure><img src=".gitbook/assets/截屏2023-11-27 22.02.45.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/截屏2023-12-23 19.40.16.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/截屏2023-12-23 19.40.37.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/截屏2023-12-23 19.40.58.png" alt=""><figcaption></figcaption></figure>

Person对象实际需要12个字节，系统分配了16个字节

Student对象实际需要16个字节，系统分配了16个字节

<figure><img src=".gitbook/assets/截屏2023-12-23 19.46.42.png" alt=""><figcaption></figcaption></figure>

### 3. API

创建一个实例对象，至少需要多少内存？

* `#import <objc/runtime.h>`
* `class_getInstanceSize([NSObject class]);也是对齐过的，比如结构体最大占用量的倍数，isa是8个字节，age是4个字节，因此为了结构体的对齐，就是两个8,返回16`

创建一个实例对象，实际上分配了多少内存？

* `#import <malloc/malloc.h>`
* `malloc_size((__bridge const void *)obj);`



### `4. 总结`

一个NSObject对象占用多少内存？

* 系统分配了16个字节给NSObject对象（通过malloc\_size函数获得）
* 但NSObject对象内部只使用了8个字节的空间（64bit环境下，可以通过class\_getInstanceSize函数获得）
