---
layout: post
comments: true
categories: tools
tags: FastBuild
---
[TOC]

记录一下FastBuild分布式编译环境部署以及遇到的一些问题




# 环境

|软件|版本|备注|
|---|---|---|
|系统|win10||
|FBuild.exe|v1.08||
|FBuildWorker.exe|v1.08||

# 安装
参考[1]讲的比较清楚了

* 1.下载
参考[2]地址，直接下载。里面FBuild.exe和FBuildWorker.exe
```
F:.
    FBuild.exe
    FBuildWorker.exe
    LICENSE.TXT
```

* 2.将FBuild.exe添加到环境变量



# 运行

* 1.写Fbuild.bff文件
* 2.运行
```
// 本地编译
FBuild.exe -clean

// 分布式编译
FBuild.exe -clean -dist
```

* 3.结果
下面就表示成功了
```
> FBuild.exe -clean
12>Obj: F:\study\engine\FastBuild\Tutorial\HelloWorld\Out\main.obj
6> Exe: F:\study\engine\FastBuild\Tutorial\HelloWorld\Out\HelloWorld.exe
FBuild: OK: all
Time: 1.929s
```

# 问题

## 1. clang11.bff相关报错
编译FastBuild源码的时候，遇到的问题和参考[1]不一样，是clang11相关的报错
```
>FBuild.exe All-x64-Release
Using VS2019 (v14.29.30133) from C:/Program Files (x86)/Microsoft Visual Studio/2019/Enterprise/VC/Tools/MSVC/14.29.30133
----------------------------------------------------------------------
- Unable to auto-detect Clang - please specify installation manually -
----------------------------------------------------------------------
F:\study\engine\FastBuild\FASTBuild-Src-v1.08\dist_v1.08\External\SDK\Clang\Windows\Clang11.bff(32,33): FASTBuild Error #1009 - Unknown variable '.Set_Path_Here'.
            .Clang11_BasePath = .Set_Path_Here    // <-- Set path here
                                ^
                                \--here
```

**原因：**
* 默认编译clang相关的东西了

**解决：**
* 直接把fbuild.bfff里面关于clang相关的全部注释掉了，我只要编译vs的就行了

## 2. 分布式编译不成功
参考[1]说的我都满足了，但是就是没有在工作机上编译
参考[3]这个文档也很好，也说了一些问题的解决办法

**原因：**
* 共享文件夹的问题，这个很好验证

只要工作机能打开共享文件夹即可，也可以看下共享文件夹下面有没有生成工作机的文件
```
\\server-pc\.fastbuild.brokerage\main\%version%\%hostname%
```
比如我的如下：

![share directory](../img/share%20directory.jpg)


* 防火墙相关

网上以及参考[3]有提到过这个，不过我局域网防火墙是开着的，木有影响。

* -dist参数

需要加-dist的参数，我遇到一个是文件夹下面有个fastbuild.bat，然后里面写的没有带-dist，虽然我在命令行输入的是-dist，但是却一直没生效，最后发现时候都哭死，命名很重要啊！！！
```
FastBuil -clean -dist
```

* 工作机能力太强

我这边把网上很多帖子都看了一遍，结果发现我情况都是对的，但是就是不在工作机工作，任务一直没有发布到工作机。log也显示有workers，工作机也显示connection
![connection](../img/fast%20build%20connection.png)
```
> FBuild.exe -clean -dist -forceremote
2 workers found in '\\10.11.180.117\FASTBuildBroker\main\22.windows\'
```

直到我看到了-forceremote参数，并且试了一下，就里面搞定了。原来是任务只有一个，默认先本地执行了，我本机有8核，所以理论上<8的任务都只会在本地执行了。加完-forceremote参数后，我笑了 ^_^



**解决：**
* 参考[1]基本上解决了问题
* 其他的就看本地配置和参数

![remote worker](../img/remote%20worker.jpg)

## 3.

# 参考
[1][保姆式教你使用FASTBuild对UE4进行联机编译](https://zhuanlan.zhihu.com/p/158400394)

[2][Download](https://fastbuild.org/docs/download.html)

[3][Distributed Troubleshooting](https://fastbuild.org/docs/troubleshooting/distribution.html)
