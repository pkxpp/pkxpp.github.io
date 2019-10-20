---
layout: post
comments: true
categories: machinelearning
tag: DeepMimic
---

[TOC]

# 安装依赖库

github主页[1]已经列出来了所有依赖和linux的安装，windows下面遇到了一些问题，不妨记录下来。

## 1.Eigen
* **关键点**：要用管理员权限运行vs，这样编译install的时候会安装到c盘下面

> 所谓的安装是把头文件拷贝到目录，这个库只有.hpp的形式，类似于rapidjson

## 2.freeglut
同Eigen

* 测试安装是否OK，参考[2]

* 编译报错：

```
	1>main.obj : error LNK2001: 无法解析的外部符号 __imp___glutCreateWindowWithExit
	1>main.obj : error LNK2001: 无法解析的外部符号 __imp___glutInitWithExit
	1>main.obj : error LNK2001: 无法解析的外部符号 __imp_glutInitDisplayMode
	1>main.obj : error LNK2001: 无法解析的外部符号 __imp_glutDisplayFunc
	1>main.obj : error LNK2001: 无法解析的外部符号 __imp_glutMainLoop
	1>main.obj : error LNK2001: 无法解析的外部符号 __imp_glutInitWindowPosition
	1>main.obj : error LNK2001: 无法解析的外部符号 __imp_glutInitWindowSize
```

* 解决：

（1）在*#include <gl/freeglut.h>  *前添加宏，依旧报错.参考[3]

```
	#define GLUT_DISABLE_ATEXIT_HACK
```

（3）把x64改成x86就ok了，因为自己用cmake默认生成的freeglut只有win32的选项~，所以需要生成64位的版本。

* 如何生成64位的工程

cmake-gui生成的时候选择带64位的，如下图所示

![cmakeX64](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/cmakeX64.png?raw=true)


## 3.MPI

看到一篇帖子[4]说注意windows下面两个都安装，以为是一样的，就装了一个.后面我装完msmpisdk.msi之后，再安装msmpisetup.exe提示错误。后面发现貌似也没啥问题~


# 总结
* 1.注意各个需要编译的工程的版本，是win32还是x64

# 参考
[1][DeepMimic github主页](https://github.com/xbpeng/DeepMimic)

[2][windows从零搭建OpenGL freeglut环境](https://blog.csdn.net/linian71/article/details/68485494)
[3][解决Opengl编译时error](https://blog.csdn.net/rodgeliao/article/details/7024094)