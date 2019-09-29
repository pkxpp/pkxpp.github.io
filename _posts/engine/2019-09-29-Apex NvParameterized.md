---
layout: post
comments: true
categories: engine
tag: engine
---

[TOC]

# 疑问

在看Apex的时候会给破碎模型设置参数，形式如下：

```
	NvParameterized::setParamBool(*actorDesc, "dynamic", false);
	NvParameterized::setParamBool(*actorDesc, "formExtendedStructures", true);
	NvParameterized::setParamF32(*actorDesc, "supportStrength", 1.9f);
	
	NvParameterized::setParamF32(*actorDesc, "defaultBehaviorGroup.damageThreshold", 5.0f);
```

很疑惑，为什么要这么做，以及怎么做的？





# 结构
## Lookup Tree
变量本身的一个树形结构，主要是记录一下某个变量是某个结构体的成员这样的父子关系。这个关系给Definition Tree提供了基础。如下图所示：
![variable-tree](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/variable-tree.jpg?raw=true)

## Definition Tree
用变量的结构构造一个字符串对应的树形结构，如下图所示：

![string-tree](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/string-tree.jpg?raw=true)


## 小结
* **两棵树的匹配关键是：定义的Children的关系树结构需要完全一样**，主要是通过字符串在ParamDefTable中找到offset，然后用这个offset到ParamLookupTable中找到变量的地址以及类型，然后读写这个变量
* 拿到了指针和类型之后，就把指针强制转换为指定类型，然后设置值了

```
	Type *Var = (Type *)((char *)ptr);
	*Var = val;
```

* 两棵树最大的不同是，Lookup Tree不需要parent的指针关系

# 创建和使用
## 创建-构建树
参考存储结构，创建树的时候其实是比较硬核的代码，手K！
* buildTree

```
	void DestructibleActorParam::buildTree(void)
	// 定义父节点
	NvParameterized::DefinitionImpl* ParamDef = &ParamDefTable[184];
	ParamDef->init("simulationFilterData", TYPE_STRUCT, "P3FilterData", true);
	// 定义子节点
	NvParameterized::DefinitionImpl* ParamDef = &ParamDefTable[185];
	ParamDef->init("word0", TYPE_U32, NULL, true);
	// 设置父子关系
	static Definition* Children[4];
	Children[0] = PDEF_PTR(185);
	Children[1] = PDEF_PTR(186);
	Children[2] = PDEF_PTR(187);
	Children[3] = PDEF_PTR(188);
	
	ParamDefTable[184].setChildren(Children, 4);
```

## 使用-查找树
* 查找 *findParam*
查找的时候会根据longName找到子节点的地址，最主要是拿到offset，因为这个offset是变量树找指针位置用的，为关键点！

* handle.index
这个接口是在findParam的时候算出来节点关系，某个子节点在父节点中children数组里面的索引。这个就是用到树的结构，然后查找


# 总结
* 这个方式通用性我觉得并不是很强，因为树都是硬编码的。对于用户只是读写来说还好，就只有这么多参数，你就用吧；但是对于其他程序员来使用这个系统的话会很受束缚
* 树的查找，这里并没有用到二叉树或者BT之类的查找，就把所有的children遍历一遍，应该是量不大，所以随便

