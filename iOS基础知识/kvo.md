# KVO

### 1. 介绍

KVO的全称是Key-Value Observing，俗称“键值监听”，可以用于监听某个对象属性值的改变

<figure><img src=".gitbook/assets/截屏2023-12-23 20.51.01.png" alt=""><figcaption></figcaption></figure>

### 2. 原理

未使用KVO之前

<figure><img src=".gitbook/assets/截屏2023-12-23 20.51.40.png" alt=""><figcaption></figcaption></figure>

使用了KVO后

<figure><img src=".gitbook/assets/截屏2023-12-23 20.52.57.png" alt=""><figcaption></figcaption></figure>

`_NSSetIntValueAndNofify原理`

<figure><img src=".gitbook/assets/截屏2023-12-23 20.54.58.png" alt=""><figcaption></figcaption></figure>

### 3. 总结

iOS用什么方式实现对一个对象的KVO？(KVO的本质是什么？)

* 利用RuntimeAPI动态生成一个子类，并且让instance对象的isa指向这个全新的子类
* 当修改instance对象的属性时，会调用Fundation的\_NSSetXXXValueAndNotify函数
  * willChangeValueForKey
  * super 原来的set方法
  * didChangeValueForKey （内部会调用监听对象的observerValueForKeyPath方法）

如何手动触发KVO？

* 手动调用willChangeValueForKey和didChangeValueForKey



直接修改成员变量会触发KVO么？

* 不会触发！

