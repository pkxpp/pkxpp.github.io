---
layout: post
comments: true
categories: tools
---

[TOC]

# 简介

网上有很多介绍google performance tools(gperftools)的文章，但是自己在安装的过程还是不断的遇到问题，即使是第二次再装的时候，所以把一些问题记录下来，希望对其他跟我一样遇到问题的人有用。

# 环境

* Linux: Centos-6.5
* 32位系统
* gperftools-4.3

# 安装步骤

* 从github获取最新版本

```
git clone https://github.com/gperftools/gperftools.git
```

* 安装之前执行autogen.sh文件

```
./autogen.sh
```


* 安装

常规命令：

	./configure
	make
	make check(可选)
	make install
	make clean


# 测试

拿 Heap Profiler 作为测试对象，使用[2]中测试Heap check的代码，就懒得写了：

	#include <cstdio>
	#include <cstdlib>
	int* fun(int n)
	{
 	   int *p1=new int[n];
 	   int *p2=new int[n];
 	   return p2;
	}
	int main()
	{
  	  int n;
  	  scanf("%d",&n);
  	  int *p=fun(n);
   	  delete [] p;
  	  return 0;
	}


编译：

	g++ -O0 -g test_heap_checker.cpp -ltcmalloc -o test_heap_checker 

运行：

	env HEAPPROFILE=./mybin.hprof ./test_heap_checker

> 注意：按照代码的意思应该是生成.hprof文件，但是自己结果只生成了一个heap文件（mybin.hprof.0001.heap），但是并不影响后面的操作


pprof分析：

	pprof --text gfs_master /tmp/profile.0100.heap
	
结果：


	[page@localhost perftools]$ pprof --text ./test_heap_checker ./mybin.hprof.0001.heap
	Using local file ./test_heap_checker.
	Using local file ./mybin.hprof.0001.heap.
	Total: 0.0 MB
	     0.0  81.8%  81.8%      0.0  81.8% std::basic_string::erase
	     0.0  18.2% 100.0%      0.0 100.0% __static_initialization_and_destruction_0 (inline)
	     0.0   0.0% 100.0%      0.0 100.0% __do_global_ctors_aux
	     0.0   0.0% 100.0%      0.0 100.0% _dl_signal_cerror
	     0.0   0.0% 100.0%      0.0 100.0% _dl_start_user
	     0.0   0.0% 100.0%      0.0 100.0% _init
	     0.0   0.0% 100.0%      0.0 100.0% global constructors keyed to symbolize.cc
	     0.0   0.0% 100.0%      0.0  81.8% std::basic_string::assign
	     0.0   0.0% 100.0%      0.0  81.8% std::basic_string::replace


说明：

* (1) 第一列表示直接使用了多少内存，单位为MB
* (2) 第四列包含了该方法自己占用的以及所有他调用所占用的内存
* (3) 第二列和第五列分别是第一列和第四列的百分比表示
* (4) 第三列是第二列的一个综合

另外，看pprof的参数，可以生成gif或者pdf文件：


	pprof --gif ./test_heap_checker ./mybin.hprof.0001.heap > a.gif
	pprof --pdf ./test_heap_checker ./mybin.hprof.0001.heap > a.pdf


# 问题记录

## 1.执行autogen.sh报错


	[root@centos6 gperftools]# ./autogen.sh 
	configure.ac:163: error: possibly undefined macro: AC_PROG_LIBTOOL
	      If this token and others are legitimate, please use m4_pattern_allow.
	      See the Autoconf documentation.
	autoreconf: /usr/bin/autoconf failed with exit status: 1


解决：安装libtool

	yum install libtool -y

结果：

	[root@centos6 gperftools]# ./autogen.sh 
	libtoolize: putting auxiliary files in `.'.
	libtoolize: copying file `./ltmain.sh'
	libtoolize: putting macros in AC_CONFIG_MACRO_DIR, `m4'.
	libtoolize: copying file `m4/libtool.m4'
	libtoolize: copying file `m4/ltoptions.m4'
	libtoolize: copying file `m4/ltsugar.m4'
	libtoolize: copying file `m4/ltversion.m4'
	libtoolize: copying file `m4/lt~obsolete.m4'
	configure.ac:150: installing `./compile'
	configure.ac:20: installing `./config.guess'
	configure.ac:20: installing `./config.sub'
	configure.ac:21: installing `./install-sh'
	configure.ac:21: installing `./missing'
	Makefile.am: installing `./depcomp'

## 2.运行报错：找不到链接库

**描述：**

	error while loading shared libraries: libtcmalloc.so.4: cannot open shared object file: No such file or directory


**解决：**

在/etc/ld.so.conf这个文件里面加入新的一行/usr/local/bin，参考[3]。文件内容如下：

	include ld.so.conf.d/*.conf
	/usr/local/lib

## 3.使用导出图片的时候报错

**描述：**


	[page@localhost perftools]$ pprof --pdf ./test_heap_checker ./mybin.hprof.0001.heap > ./a.pdf
	Using local file ./test_heap_checker.
	Using local file ./mybin.hprof.0001.heap.
	sh: dot: command not found


**原因:**

graphviz没有安装，结果参照官网的过程根本就不成功，缺少libANN.so.1等的错误

**解决: 安装CPG key**

最后参考[4]终于搞定了！下载epel-release-6-8.noarch

	wget http://mirror.symnds.com/distributions/fedora-epel/6/i386/epel-release-6-8.noarch.rpm

安装步骤，参考[5]：

* 测试：

```
rpm -ivh epel-release-6-5.noarch.rpm --test
```

* 不知道干嘛：

```
wget https://fedoraproject.org/static/0608B895.txt
mv 0608B895.txt /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
```
* 验证：
```
# rpm -qa gpg*
gpg-pubkey-0608b895-4bd22942
```
* 安装key
```
rpm -ivh epel-release-6-5.noarch.rpm
```

**再重新安装graphviz**

```
yum install 'graphviz*'
```
这里就直接ok了, bingo!!!

* 但是，有些机器还是会报错

部分报错信息如下：

	http://www.graphviz.org/pub/graphviz/stable/redhat/el6/i386/os/graphviz-2.40.20161221.0239-1.el6.i686.rpm: [Errno 14] PYCURL ERROR 22 - "The requested URL returned error: 404 Not Found"
	Trying other mirror.
	
	// 省略...

	Error Downloading Packages:
	  graphviz-plugins-x-2.40.20161221.0239-1.el6.i686: failure: graphviz-plugins-x-2.40.20161221.0239-1.el6.i686.rpm from graphviz-stable: [Errno 256] No more mirrors to try.
	  graphviz-gd-2.40.20161221.0239-1.el6.i686: failure: graphviz-gd-2.40.20161221.0239-1.el6.i686.rpm from graphviz-stable: [Errno 256] No more mirrors to try.
	  graphviz-lang-java-2.40.20161221.0239-1.el6.i686: failure: graphviz-lang-java-2.40.20161221.0239-1.el6.i686.rpm from graphviz-stable: [Errno 256] No more mirrors to try.
	  // 省略...
	   graphviz-stable: [Errno 256] No more mirrors to try.
	  graphviz-plugins-gd-2.40.20161221.0239-1.el6.i686: failure: graphviz-plugins-gd-2.40.20161221.0239-1.el6.i686.rpm from graphviz-stable: [Errno 256] No more mirrors to try.


说是要更新rpm，参考[7]，结果搞了一个多小时还是没鸟用，建议遇到同样问题的朋友，不要再走这条路了，前车之鉴！！！


**最终解决办法：**

因为提示下载不了依赖库，所以直接去官网提供的对应版本的的rpm包安装
从官网[yum repository](http://www.graphviz.org/Download_linux_rhel.php)下载，找到对应版本的（比如自己的linux是l6-i386的版本，可以用uname -a查看自己的linux系统），然后右键要下载的包，复制网址出来，用wget命令分别下载：

* graphviz-x-2.38.0-1.el6.i686.rpm

安装这个的时候很快就安装好了

* graphviz-gd-2.38.0-1.el6.i686.rpm

也是很快就安装好了

* graphviz-libs-2.38.0-1.el6.i686.rpm

这个比较慢一点，因为他把grahpviz也一起安装好了，应该算是依赖库的关系

* graphviz-2.38.0-1.el6.i686.rpm

提示已经安装了，所以这里其实不用安装了

* 测试：
* 
参考[6]
能够成功生成图片，搞定！


# 参考

[1]gperftools目录下的INSTALL文件

[2][Google Perftools简介与使用](http://blog.csdn.net/turkeyzhou/article/details/8794188)

[3][error while loading shared libraries的解決方法 ](http://www.cnblogs.com/amboyna/archive/2008/02/06/1065322.html)

[4][CentOS安装Graphviz（使用EPEL）](http://blog.csdn.net/mtawaken/article/details/12832897)

[5]http://www.thegeekstuff.com/2012/06/enable-epel-repository/

[6][我使用过的Linux命令之dot - 绘制DOT语言脚本描述的图形](http://codingstandards.iteye.com/blog/840055)

[7][yum [Errno 256] No more mirrors to try 解决方法](http://www.360doc.com/content/15/0424/11/22890708_465637436.shtml)