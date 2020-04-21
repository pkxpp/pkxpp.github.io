---
layout: post
comments: true
categories: go
tags: go protobuf
---

[TOC]

# 编译
参考[1][2]的步骤如下：

* 1.**安装编译器**

这里很容易漏掉，一开始就是因为没有这个步骤，导致找不到protoc
下载对应系统的压缩包，解压之后把protoc.exe放在gopath/bin中，也就是Go安装路径下面的bin中

* 2.安装protobuf-go

```
go install google.golang.org/protobuf/cmd/protoc-gen-go
```

遇到过如下报错

```
can't load package: package google.golang.org/protobuf/cmd/protoc-gen-go: cannot find package "google.golang.org/protobuf/cmd/protoc-gen-go" in any of:
        C:\Go\src\google.golang.org\protobuf\cmd\protoc-gen-go (from $GOROOT)
        C:\Users\peikai\go\src\google.golang.org\protobuf\cmd\protoc-gen-go (from $GOPATH)
```

如果也有同样报错的话就先get一下

```
go get google.golang.org/protobuf/cmd/protoc-gen-go
```

* 编译
指令：
```
protoc -I=$SRC_DIR --go_out=$DST_DIR $SRC_DIR/addressbook.proto
```

自己开发的时候，直接cd到*.proto目录，执行如下简单命令：

```
protoc --go_out=. *.proto
```

# 使用
## 类型
参考[3]中**Scalar Value Types**一节

# 参考
[1][tutorial](https://developers.google.com/protocol-buffers/docs/gotutorial)

[2][Go中protobuf的使用](https://blog.csdn.net/weixin_42117918/article/details/88920221)

[3][Scalar Value Types](https://developers.google.com/protocol-buffers/docs/proto3)