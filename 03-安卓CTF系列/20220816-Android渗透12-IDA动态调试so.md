# Android渗透12：IDA动态调试so

## 0x00 前言

上一篇分享了使用 **Android studio** 和 **Jeb** 对 Apk 文件直接进行动态调试，本文将分享使用 **IDA pro 调试 so**。

调试的 apk 文件还是使用 CTF案例4 的文件，已经上传到知识星球，可自行下载

本文涉及技术：

+ IDA pro 工具使用
+ 调试android 应用
+ 动态调试技术

> 注意：本案例所需要的 apk 文件，已经上传到知识星球，需要的朋友可以到文末关注后下载



## 0x01 准备

1、下载案例 Apk 文件

使用 Android Killer 工具修改配置文件，加上 **android:debuggable="true"**  这个配置，这样 apk 就可以被调试了

![1660657645319](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660657645319.png)

2、找到 `IDA_Pro_v7.5_Portable\dbgsrv` 目录，里面文件对应不同平台的 server 文件：

![1660654872494](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660654872494.png)

3、查看手机的 cpu 架构：

![1660654971600](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660654971600.png)

4、调试的手机是 android 的 arm64-v8a，可以选择 **android_server64** ,把这个文件发送到 手机的 `/data/local/tmp` 目录，然后赋予执行权限，最后执行

![1660655188000](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660655188000.png)

默认的端口是 23946 ，这里故意改为 22222

5、端口转发

```
adb forward tcp:11111 tcp:22222    
```

这里表示将本地 11111 端口（我本机是 windows），转发到远程手机 22222 端口：

![1660655381784](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660655381784.png)

准备工作做好之后，将讲解两种方式调试，一种就是普通启动 apk 的调试，一种是以 Debug 模式启动 apk 的调试。如果调试的代码逻辑，在启动之后，则可以使用普通启动apk的调试模式，如果调试的代码逻辑在启动的时候就执行了，这时需要使用 Debug 模式启动 Apk 进行调试



## 0x02 普通模式调试

1、adb 启动对应的 Activity，这里使用 **普通模式启动 apk**

```
adb shell am start -n  com.example.hellojni/com.example.application.IsThisTheRealOne 
```

2、打开 IDA pro 64 位版本，加载 arm64-v8a 中 libhello-jni.so 文件

3、IDA 调试配置

选择 Remote ARM Linux/Android debugger 调试

![1660655686992](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660655686992.png)

选择 Debugger options 勾选以下三个选项

![1660655740790](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660655740790.png)

选择 Process options 进行配置：

![1660655788941](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660655788941.png)

选择需要 attach 的进程

![1660655874159](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660655874159.png)

4、出现  Debugger warning

这表示在手机中找到和本地加载相同的 so 文件，选择 same：

![1660655952358](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660655952358.png)

5、在模块中搜索，需要调试的模块

![1660656073230](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660656073230.png)

然后定位要打断点的函数：

![1660656083271](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660656083271.png)

6、打断点

这里可以直接使用快捷键 F5 ，然后在 c/c++ 的伪代码里打断点：

![1660656146760](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660656146760.png)

7、开始调试

点击 **Quick debug view**，打开 **Locals** 窗口，F9 运行，F8 单步步过，进行调试

![1660656290240](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660656290240.png)

8、 成功拿到 flag 值



## 0x03 Debug模式调试

1、打开 **Android device monitor** 工具

这个工具在 Android studio  的 `Android\Sdk\tools` 目录下

2、以 **Debug 模式**启动 apk的 Activity

```
adb shell am start -D -n  com.example.hellojni/com.example.application.IsThisTheRealOne
```

发现在 **Android device monitor** 中，将要调试的程序前面，多了一个**红色的虫子**

![1660656701031](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660656701031.png)

3、打开 IDA ，加载 so（同上）

4、IDA 调试配置，勾选三项等，找到要调试的进程（同上）

5、执行 jdb 命令

```
jdb -connect com.sun.jdi.SocketAttach:hostname=127.0.0.1,port=8608
```

这里的端口，可以在 Monitor 中查看到

执行完成之后，**红色的虫子变为绿色的虫子**，接着就可以开始调试了

![1660657074005](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660657074005.png)

6、开始调试，按 F9 ,知道出现

![1660657179166](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660657179166.png)

这个时候就加载到需要调试的 so 了

7、在模块中查找需要调试的模块，以及模块中的具体函数（同上）

![1660657272560](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660657272560.png)

8、继续调试

打开 Locals 查看变量

![1660657359264](20220816-Android%E6%B8%97%E9%80%8F12-IDA%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95so.assets/1660657359264.png)

成功拿到 flag 值



## 0x04 结语

在 Android 逆向中，动态调试是非常重要的一个技术，IDA pro 时调试 so 代码的利器。感兴趣的朋友，可以下载案例的 apk 文件，实战起来。







