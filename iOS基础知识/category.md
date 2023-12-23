# Category

### 1. 底层结构

<figure><img src=".gitbook/assets/截屏2023-12-23 21.07.51.png" alt=""><figcaption></figcaption></figure>

### 2. 加载处理过程

1. 一个Category对应上面一个结构体，main函数之前，通过Runtime加载某个类的所有Category数据
2. 把所有Category的方法、属性、协议数据，合并到一个大数组中\
   后面参与编译的Category数据，会在数组的前面
3. 将合并后的分类数据（方法、属性、协议），插入到类原来数据的前面

