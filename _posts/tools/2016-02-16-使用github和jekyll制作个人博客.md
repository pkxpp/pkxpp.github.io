---
layout: post
comments: true
categories: tools
---

## 入门

###1.使用github page搭建一个简陋的个人博客

（1）参考[1]

（2）了解到了每个github帐号只能搭建一个博客，但是可以为每个项目单独简历一个博客，有两点需要注意：

* github帐号唯一的博客的项目名需要是username.github.io的形式
* 每个项目单独的博客需要简历一个分支，名字为gh-pages


###2.hexo


（1）使用hexo搭建网站非常方便，参考[1]

（2）常用命令

* hexo g	# 文件变化了之后，用于重新生成网站文件
* hexo s	# 用于启动本地服务器，调试
* hexo d	# 上传最新生成的静态网站文件到配置的github上面

（3）上传成功之后打开username.github.io就可以看到最新的网站了



###3.jekyll


（1）还是决定使用jekyll，虽然hexo搭建和使用非常方便，但是现在网上的模版很少，而jekyll的模版非常多[http://jekyllthemes.org/](http://jekyllthemes.org/ "jekyllthemes")

（2）操作

* 选择一个github.io模版
* 修改成自己的
* 使用命令：jekyll serve进行本地调试
* 上传到username.github.io
* 打开username.github.io就可以看到自己的网站了

（3）安装

参考[3]


##问题


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


##参考


[1]http://www.jianshu.com/p/05289a4bc8b2

[2][http://zhidao.baidu.com/link?url=k6epEW6_POVCf88EsosZz2LCyodSbglsdWLujEfXln5CapaLMWntPQq2RyDl0R-wRbgW7PCKeXV-J7v4UBtFS_ZskXzGiV6TSm_Nm_mXiK7](http://zhidao.baidu.com/link?url=k6epEW6_POVCf88EsosZz2LCyodSbglsdWLujEfXln5CapaLMWntPQq2RyDl0R-wRbgW7PCKeXV-J7v4UBtFS_ZskXzGiV6TSm_Nm_mXiK7 "hexo deploy报错")

[3]http://www.pchou.info/web-build/2013/01/05/build-github-blog-page-04.html

[4]http://www.aichengxu.com/view/2404045

[5]http://www.pchou.info/web-build/2013/01/05/build-github-blog-page-04.html