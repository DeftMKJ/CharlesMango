# KVC

### 1. 介绍

KVC的全称是Key-Value Coding，俗称“键值编码”，可以通过一个key来访问某个属性

常见的API有

* `- (void)setValue:(id)value forKeyPath:(NSString *)keyPath;`
* `- (void)setValue:(id)value forKey:(NSString *)key;`
* `- (id)valueForKeyPath:(NSString *)keyPath;`
* `- (id)valueForKey:(NSString *)key;`&#x20;



### `2. 原理`

<figure><img src=".gitbook/assets/截屏2023-12-23 21.04.39.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/截屏2023-12-23 21.05.50.png" alt=""><figcaption></figcaption></figure>



### 3. 总结

通过KVC修改属性会触发KVO么？

* 会调用`set方法，很明显会触发`
