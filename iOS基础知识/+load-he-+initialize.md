# +load和+initialize

### 1. +load

`+load`方法会在runtime加载类、分类时调用

每个类、分类的+load，在程序运行过程中只调用一次

1. 先调用类的+load

按照编译先后顺序调用（先编译，先调用）

调用子类的+load之前会先调用父类的+load

2. 再调用分类的+load

按照编译先后顺序调用（先编译，先调用）

> \+load方法是根据方法地址直接调用，并不是经过objc\_msgSend函数调用



### 2. initialize

`+initialize`方法会在`类`第一次接收到消息时调用

1. 先调用父类的+initialize，再调用子类的+initialize
2. (先初始化父类，再初始化子类，每个类只会初始化1次)



### 3. 总结

load、initialize方法的区别什么？它们在category中的调用的顺序？以及出现继承时他们之间的调用过程？

1. 调用方式

load是根据函数地址直接调用

initialize是通过objc\_msgSend调用

2. 调用时刻

load是runtime加载类，扩展的时候调用（只会调用一次）

initialize是类第一次接受消息的时候调用，每一个类只会initialize一次（父类可能被调用多次）

3. 顺序

父类优先子类，load方法类优先扩展，且按顺序调用

