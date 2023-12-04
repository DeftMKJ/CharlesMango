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
