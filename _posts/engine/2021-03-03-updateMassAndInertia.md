---
layout: post
comments: true
categories: engine
tags: engine PhysX
---

[TOC]

在写代码的时候会经常看到updateMassAndInertia这个函数，能大概知道意思是更新质量和惯性。但是，是个什么样的效果呢？





# 更新
可以做个实验
(1)先创建一个box
(2)然后再attach一个box的shape，然后里面就调用*updateMassAndInertia*接口
(3)再然后detach掉，调用*updateMassAndInertia*

整个过程看下效果

![updateMassAndInertia]()

可以看到这个PhysxActor的重心会根据添加的shape而发生变化

# 不更新
还是上面的实验：
(1)create box
(2)attach box + updateMassAndInertia
(3)detach box，这里不调用*updateMassAndInertia*接口

![not updateMassAndInertia]()

# 总结
* 1.*updateMassAndInertia*影响的质量其实是影响了重心