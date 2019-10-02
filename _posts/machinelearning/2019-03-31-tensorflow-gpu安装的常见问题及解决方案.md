---
layout: post
comments: true
categories: machinelearning
tag: tensorflow
---

[TOC]

装tensorflow-gpu的时候经常遇到问题，自己装过几次，经常遇到相同或者类似的问题，所以打算记录一下，也希望对其他人有所帮助





# 基本信息
* tensorflow-gpu
* pip安装（virtualenv等虚拟安装实质也是pip安装，只是建了个独立的环境，不会影响系统环境，查问题比较容易，最多重新再创建一个干净的环境再来）


安装完之后会用*import tensorflow*看是否安装成功，结果报错，主要有碰到下面两大类报错信息：

# 1.ImportError: DLL load failed: 找不到指定的模块 之pywrap_tensorflow.py
报错信息里面有大量的pywrap_xxx相关的脚本报错：

```
Traceback (most recent call last):
  File "E:\study\machinelearning\ENV\lib\site-packages\tensorflow\python\pywrap_tensorflow.py", line 58, in <module>
    from tensorflow.python.pywrap_tensorflow_internal import *
  File "E:\study\machinelearning\ENV\lib\site-packages\tensorflow\python\pywrap_tensorflow_internal.py", line 28, in <module>
    _pywrap_tensorflow_internal = swig_import_helper()
  File "E:\study\machinelearning\ENV\lib\site-packages\tensorflow\python\pywrap_tensorflow_internal.py", line 24, in swig_import_helper
    _mod = imp.load_module('_pywrap_tensorflow_internal', fp, pathname, description)
  File "E:\study\machinelearning\ENV\lib\imp.py", line 242, in load_module
    return load_dynamic(name, filename, file)
  File "E:\study\machinelearning\ENV\lib\imp.py", line 342, in load_dynamic
    return _load(spec)
ImportError: DLL load failed: 找不到指定的模块。

```

这类错误出现的最多，主要有几大类原因：

## （1）Microsoft Visual C++ 2015 Redistributable Update 3 没有装
这个是自己第一次装的时候碰到的，下载[vc_redist.x64.exe](https://www.microsoft.com/en-us/download/confirmation.aspx?id=53587)安装之后就ok了

* 再生波澜

自己今天再装的时候，下载下来发现安装不了，看日志是说我的vs版本比较新，所以不能装。这个时候可以可以看看自己本机的system32下面有没有MSVCP140.DLL这个文件

* 其他解决方案

有些网友说用的比较新的tensorflow，装了2017的Redistributable包就好了，你也可以试试

我再装完2017的包之后，并且检查自己系统中已经有了MSVCP140.DLL文件依旧报同样的错误

## （2）cuda和cudnn版本不一致
这个问题也是非常多的，我装了很多次的cuda基本上没有安装失败过，但是遇到和cudnn版本不一致的情况。因为下载的cuda默认是最新版本的cuda10.0，而我下载的cudnn当时用的旧的，也就是给cuda9.0的，所以后面换了一下也就解决问题了

* [cuda下载](https://developer.nvidia.com/cuda-zone)

![cuda下载](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/cuda_download.png?raw=true)

我这里默认点完自己系统的配置（win10x64）得到的是最新的cuda10-win10，可以点击最右边的Legacy Releases看到更早一点的版本

* cuda安装和验证

一路next貌似没遇到过啥问题
验证的话：在命令行下面输入nvcc -V，看是否OK
另外sample下面的两个是deviceQuery.exe和bandwidthTest.exe执行都没有出现问题过

* [cudnn下载](https://developer.nvidia.com/rdp/cudnn-download)
要登录nvidia developer账号
![cuda下载](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/cudnn-download.png?raw=true)
点开最下面的Archived cuDNN Releases可以看到更多的版本，因为我下载的是cuda-9.0，稳妥起见，我下载的cudnn版本是：*Download cuDNN v7.0.5 (Dec 5, 2017), for CUDA 9.0*
按照道理来讲这里的*Download cuDNN v7.5.0 (Feb 21, 2019), for CUDA 9.0*应该也可以，下次验证再确认一下。

* cudnn安装

在下载的页面可以打开*Installation-Guide*看一下windows的cudnn安装指南，主要有以下操作

（1）把解压缩的cudnn下面的bin、lib和include三个文件夹下面的文件拷贝到cuda安装的目录下面同名的目录下面
cuda路径：C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0

（2）把CUDA路径添加到环境变量的CUDA_PATH中
![cuda下载](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/cuda-path.png?raw=true)
cuda本书在安装的时候会把cuda的安装路径添加的环境变量的path中（**注意**：是在path的最前面，不容易看到），所以不必自己把cuda的路径添加到path中

> 这里自己是把解压后的cudnn放到d盘，比如：D\cuda，然后把D:\\cuda\bin放到了path中，因为网上有些人是这样建议的。但是看cudn的安装指南并没有提及到，所以感觉应该不需要

很遗憾的是，今天保证这里版本一直之后，还是依旧报 = =
## （3）tensorflow-gpu版本不一致

安装tensorflow-gpu的时候一般都是用的默认指令：

```
pip install --upgrade tensorflow-gpu
```

结果是会把tensorflow-gpu的最新版本装上，我的版本情况如下：

（1）python：3.6.0

（2）cuda-9.0

（3）cudnn-7.0

（4）tensorflow-gpu-1.13.0


最新的cuda是10.0了，但是我装的是9.0，所以我把tensorflow-gpu装到1.12.0，然后完美解决问题了。^_^

```
pip uninstall tensorflow-gpu==1.13.0
pip install tensorflow-gpu==1.12.0
```

这里说明tensorflow-gpu1.13.0估计是用了最新的cuda版本中的内容，也算是版本不一致了。

> 如果跟我一样，上面的问题都解决了，那就看看是不是这里版本太新或者太旧了。这里有个插曲，因为我开始不小心把1.12.0输成了1.2.0，结果还是不行，没注意结果纯粹浪费了一段时间。

## （4）其他python库版本问题等

网上有些人还遇到numpy等python库版本等的问题，我倒是没遇到，因为安装tensorflw-gpu的时候会把相关的依赖包都给下载下来

# 2.TensorFlow pip installation issue: cannot import name 'descriptor'之graph_pb2.py
报错信息如下有graph_xxx相关的脚本报错：

```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "F:\study\machinelearning\ENV\lib\site-packages\tensorflow\__init__.py", line 24, in <module>
    from tensorflow.python import pywrap_tensorflow # pylint: disable=unused-import
  File "F:\study\machinelearning\ENV\lib\site-packages\tensorflow\python\__init__.py", line 59, in <module>
    from tensorflow.core.framework.graph_pb2 import *
  File "F:\study\machinelearning\ENV\lib\site-packages\tensorflow\core\framework\graph_pb2.py", line 6, in <module>
    from google.protobuf import descriptor as _descriptor
  File "F:\study\machinelearning\ENV\lib\site-packages\google\protobuf\descriptor.py", line 47, in <module>
    from google.protobuf.pyext import _message
ImportError: DLL load failed: 找不到指定的程序。
```

这个我碰到过两次，都是protobuf的版本高了的缘故，网上搜到的也是这个原因，把protobuf的版本从3.6.1降到3.6.0解决

```
pip list
pip uninstall protobuf
pip install protobuf==3.6.0
pip list

```

# 参考
[1][import error: load dll failed](https://github.com/tensorflow/tensorflow/issues/8385)