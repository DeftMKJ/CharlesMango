# 1. 环境搭建

### 1. 越狱

由于uncover最新只支持到14.8，因此换成工具Dopamine，也可以电脑下载爱思助手，一键越狱。最好准备一个不怎么用的Appleid用来登录，然后手机上会出现Dopamine，我的设备是6s plus,15.2，直接越狱就行。

不走爱思助手的话，直接按照这个文档操作即可。

{% embed url="https://dkxuanye.cn/?p=9682" %}

还找到一个 [https://ios.cfw.guide/get-started/](https://ios.cfw.guide/get-started/)

### 2. 必装源

### **巨魔大神作者opa334源** **https://opa334.github.io/**

### **刀刀源** **https://xiangfeidexiaohuo.github.io/**



[Cydiakk中文源™](https://apt.cydiakk.com/)

[https://apt.cydiakk.com/](https://apt.cydiakk.com/)



### 2. 必装插件（手机）

咱们不是用uncover越狱的，所以没有Cydia，相对应的就是Sileo，界面更简洁好看些，用法都是一样的

#### 2.1 Apple File Conduit "2"

如果装的是afc2d [https://github.com/Cannathea/afc2d-arm64/releases/tag/v1.2.0](https://github.com/Cannathea/afc2d-arm64/releases/tag/v1.2.0)

建议先把ldid装了，安装完后就可以访问完整的iOS文件系统

#### 2.2 AppSync Unified（关闭iOS App签名检查，装和谐软件必备）

绕过系统验证，随意安装，运行破解的ipa安装包

#### 2.3 Filza File Manager

方便在手机上查看文件目录

#### 2.4 openssh

openssh，openssh-client，openssh-server，openssh-sftp-server

#### 2.5 爱思助手

这个如果越狱用的爱思助手，会自动装一个上去



### 3. 必装软件（电脑）

iFunBox



### ！！！注意事项！！！

1. shell 或者ssh无法工作，无论重启都不做，参考下面

`Shell & SSH doesn't work with Dopamine`

`/private/preboot/UUID修改其他组的权限，默认777，改成755`

`这里可能不同机型不同，我的是6s plus 15.3.1`

\`iPhone 13 Pro, wiith New Term and SSH working well: owner `root:wheel`, permissions `755`.

iPhone XR, wiith New Term and SSH broken: owner `root:wheel`, permissions `777`.

[https://github.com/opa334/Dopamine/issues/110](https://github.com/opa334/Dopamine/issues/110)



2. 修改越狱机root

报错 $ ssh -p 2222 root@127.0.0.1 (root@127.0.0.1) Password for root@MEIKIE-iPhone-Xs-1561: UNIX authentication refused

https://www.youtube.com/watch?v=usIzOZOKb3Y

* 越狱机上下载New Terminal 3
* sudo passwd root
* 历史密码 1，修改新密码 2
* su/whomai 输出是root即可连接

3. 电脑ssh连接失败

IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY! Someone could be eavesdropping on you right now (man-in-the-middle attack)! It is also possible that a host key has just been changed. The fingerprint for the ED25519 key sent by the remote host is SHA256:kzV5E5L2BH1GClhYJU9pjFesZVWMoUUU+zxnZ+WVxAE. Please contact your system administrator. Add correct host key in /Users/mikejing/.ssh/known\_hosts to get rid of this message. Offending ED25519 key in /Users/mikejing/.ssh/known\_hosts:6 Host key for 192.168.50.43 has changed and you have requested strict checking. Host key verification failed.



`删除.ssh/known_hosts 这个文件里面的127.0.0.1或者对应的ip那一行即可。`

####



{% embed url="https://juejin.cn/post/7054186252416843790" %}
