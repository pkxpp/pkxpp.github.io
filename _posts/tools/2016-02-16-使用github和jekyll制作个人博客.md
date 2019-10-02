---
layout: post
comments: true
categories: tools
---

## 入门

### 1.使用github page搭建一个简陋的个人博客

（1）参考[1]

（2）了解到了每个github帐号只能搭建一个博客，但是可以为每个项目单独简历一个博客，有两点需要注意：





* github帐号唯一的博客的项目名需要是username.github.io的形式
* 每个项目单独的博客需要简历一个分支，名字为gh-pages


### 2.hexo


（1）使用hexo搭建网站非常方便，参考[1]

（2）常用命令

* hexo g	# 文件变化了之后，用于重新生成网站文件
* hexo s	# 用于启动本地服务器，调试
* hexo d	# 上传最新生成的静态网站文件到配置的github上面

（3）上传成功之后打开username.github.io就可以看到最新的网站了



### 3.jekyll


（1）还是决定使用jekyll，虽然hexo搭建和使用非常方便，但是现在网上的模版很少，而jekyll的模版非常多[http://jekyllthemes.org/](http://jekyllthemes.org/ "jekyllthemes")


（2）安装

* 参考[4]，有截图。对安装过程更有帮助
* 先看参考[3]

（3）操作

* 使用jekyll new创建一个工程调试
* 熟悉了基本框架之后，选择一个github.io模版
* 修改成自己的
* 使用命令：jekyll serve进行本地调试
* 上传到username.github.io
* 打开username.github.io就可以看到自己的网站了

（4）安装注意问题：

* 参考[3]前面写的一些更新之前没有注意，可以好好看一下
* 关于config.yml
 - 配置文件里面写的路径是ruby的安装路径根目录，而不是ruby-dev的；
 - 路径用linux的斜杠方向

## 问题


（1）本地测试localhost:4000没有反应

* 原因：没有安装hexo-server
* 解决：npm install hexo-server --save安装即可

（2）hexo deploy发布报错

* 描述：“ERROR Deployer not found: github”
* 原因：
* 解决：参考[2]

（3）jekyll serve运行报错

* 描述：jekyll 3.1.1 | Error:  Permission denied - bind(2) for 127.0.0.1:4000
* 原因：端口占用
* 解决：换掉端口即可，参考[4]

（4）jekyll报错

* 描述：参考[5]中自己的jekyll serve一直启动不起来，报错
* 原因：怀疑是文章中的说的不是很清楚，_config.yml这个文件不是devkit中的，而是使用jekyll new之后的项目中的_config.yml这个文件
* 解决：jekyll new一个，再操作，本地测试通过


（5）修改端口报错

* 描述：语法错误
* 原因：_config.yml格式上冒号后面需要空格，例如port: 5001
* 解决：port:5001 --> port: 5001

（6）选择国内淘宝镜像，执行命令报错

* 在自己window7下执行以下命令报错：

```
gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
//后面有改了@2019/02/01
gem sources -a https://gems.ruby-china.com/
```


如图所示：

解决：
* 参考[3]评论里面2楼的连接[4]
* 最主要是虾米一段话
> ruby 没有包含 SSL 证书，所以 https 的链接被服务器拒绝。解决方法很简单，首先在这里下载证书 http://curl.haxx.se/ca/cacert.pem, 然后再环境变量里设置 SSL_CERT_FILE 这个环境变量，并指向 cacert.pem 文件。
set SSL_CERT_FILE=C:\path\to\cacert.pem
* 重启cmd命令行搞定！f

（7）文件名导致本地点击没有效果

报错如下


	[2016-05-04 09:52:05] ERROR '/favicon.ico' not found.
	[2016-05-04 09:52:06] ERROR '/2016/05/03/lua学习笔记(2)-元方法/' not found.
	[2016-05-04 09:52:06] ERROR '/favicon.ico' not found.
	[2016-05-04 09:52:07] ERROR '/favicon.ico' not found.

把名字改成lua学习笔记(2)-元方法x.md就可以了，说明还是和名字有关。猜想还是编码问题，比如中文在两个字节的编码，而utf-8是变长的导致不能够解析名字所以就出错了

解决办法：

## 参考


[1](http://www.jianshu.com/p/05289a4bc8b2)

[2][http://zhidao.baidu.com/link?url=k6epEW6_POVCf88EsosZz2LCyodSbglsdWLujEfXln5CapaLMWntPQq2RyDl0R-wRbgW7PCKeXV-J7v4UBtFS_ZskXzGiV6TSm_Nm_mXiK7](http://zhidao.baidu.com/link?url=k6epEW6_POVCf88EsosZz2LCyodSbglsdWLujEfXln5CapaLMWntPQq2RyDl0R-wRbgW7PCKeXV-J7v4UBtFS_ZskXzGiV6TSm_Nm_mXiK7 "hexo deploy报错")

[3][一步步在GitHub上创建博客主页](http://www.pchou.info/web-build/2013/01/05/build-github-blog-page-04.html)

[4][[前端]jekyll+markdown+github搭建个人博客](http://www.aichengxu.com/view/2404045)

[5](http://www.pchou.info/web-build/2013/01/05/build-github-blog-page-04.html)