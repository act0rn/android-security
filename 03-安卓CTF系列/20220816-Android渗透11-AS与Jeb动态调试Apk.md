# Android渗透11：AS与Jeb动态调试Apk

## 0x00 前言

本文分享 **Android studio** 和 **Jeb** 的**动态调试 Apk**，后一篇再介绍 IDA pro 动态调试 so 代码

上篇分享了4种方式来获取 CTF案例4 的 flag 值，其中动态调试方式，没有具体展开，接下来，以上篇 CTF案例4 的 apk 作为调试的对象，开始分享使用 AS 和 Jeb 动态调试 Apk，并且获得 flag 值

本文涉及技术

+ Android studio工具
+ Jeb 工具
+ 动态调试技术
+ 安卓程序修改和重新打包

> 注意：本案例所需要的 apk 文件，已经上传到知识星球，需要的朋友可以到文末关注后下载



## 0x01 准备

首先下载好案例所需的 apk 文件

使用 Android Killer 工具修改配置文件，加上 **android:debuggable="true"**  这个配置，这样 apk 就可以被调试了![1660650608704](20220816-Android%E6%B8%97%E9%80%8F11-AS%E4%B8%8EJeb%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95Apk.assets/1660650608704.png)

保存后，重新打包，然后安装到手机



## 0x02 Android Studio 动态调试 Apk

> 准备：使用 Android Studio 调试，需要安装一个 smalidea 插件，本文使用的是 0.06 版本，android studio 是 2021 版本，手机是 pixel 2xl 

![1660652226746](20220816-Android%E6%B8%97%E9%80%8F11-AS%E4%B8%8EJeb%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95Apk.assets/1660652226746.png)

1、使用 adb 工具，以调试模式打开相应的 Activity

```
adb shell am start -D  -n  com.example.hellojni/com.example.application.IsThisTheRealOne
```

2、查看启动的进程 id

```
adb shell ps | grep hellojni
```

![1660651034193](20220816-Android%E6%B8%97%E9%80%8F11-AS%E4%B8%8EJeb%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95Apk.assets/1660651034193.png)

3、本地调试端口转发到远程相应的进程

```
adb forward tcp:8700 jdwp:24756     // 8700为本地调试端口，24756 是手机中的进程 id
```

![1660651174994](20220816-Android%E6%B8%97%E9%80%8F11-AS%E4%B8%8EJeb%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95Apk.assets/1660651174994.png)

4、打开 Android studio

![1660651202154](20220816-Android%E6%B8%97%E9%80%8F11-AS%E4%B8%8EJeb%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95Apk.assets/1660651202154.png)

选择要调试的 apk 文件，打开

5、远程调试配置，端口改为上面的配置的 8700

![1660651765794](20220816-Android%E6%B8%97%E9%80%8F11-AS%E4%B8%8EJeb%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95Apk.assets/1660651765794.png)

6、关键处打上断点，选择要调试的进程

![1660652115211](20220816-Android%E6%B8%97%E9%80%8F11-AS%E4%B8%8EJeb%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95Apk.assets/1660652115211.png)

7、开始调试，添加需要 watch 的变量

![1660652043645](20220816-Android%E6%B8%97%E9%80%8F11-AS%E4%B8%8EJeb%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95Apk.assets/1660652043645.png)

8、 获取 flag 成功



## 0x03 Jeb 动态调试 Apk

**1、使用 Jeb 打开需要调试的 Apk 文件**

可以将 smali 转为 Java 伪代码（快捷键：Tab键），然后在关键的 smali 代码处打上断点

![1660652568624](20220816-Android%E6%B8%97%E9%80%8F11-AS%E4%B8%8EJeb%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95Apk.assets/1660652568624.png)

**2、使用 adb 打开相应的 Activity**

```
adb shell am start -D  -n  com.example.hellojni/com.example.application.IsThisTheRealOne
```

注意：这里 -D 是以调试模式，打开 Activity，如果不使用 -D 就是普通模式调试，在这个案例当中也可以

**3、选择菜单中的 Debugger -> start，开始调试**

打上断点，快捷键：Ctrl + B

![1660652868169](20220816-Android%E6%B8%97%E9%80%8F11-AS%E4%B8%8EJeb%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95Apk.assets/1660652868169.png)

选择对应的进程，点击 Attach ，开始调试

**4、调试，查看变量**

注意：如果查看的变量是字符串类型的话，需要修改 Type 为 string

![1660652992753](20220816-Android%E6%B8%97%E9%80%8F11-AS%E4%B8%8EJeb%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95Apk.assets/1660652992753.png)

5、获取 flag 值成功



## 0x04 结语

本文分享了 AS 和 Jeb 动态调试 Apk，在实战中动态调试 Apk 还是非常重要的，这一篇主要调试的 smali 的代码，如果是 native 的代码，就需要使用 IDA 进行动态调试，将会在下一篇进行分享。









