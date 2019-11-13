---
layout: post
comments: true
categories: engine
tags: engine physX
---

[TOC]

# 问题描述
有个楼梯的斜面的normal反了，结果会导致模型卡在这个楼梯走不出来，类似于一个封闭的盒子。因为模型现在改不了的缘故，所以就准备技术上先解决一下：**启用double-sided的功能。**


* PVD正面看到楼梯，看不到斜面
![stair front](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/stair_front.png?raw=true)

* PVD反面看到的斜面
![stair back](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/stair_back.png?raw=true)

* 组合起来看到一个内部封闭的空间，胶囊体掉进来就卡主了
![stair box](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/stair_box.jpg?raw=true)

# 创建double-sided的mesh

PhysX默认是只创建一面三角形的，在PVD里面也可以看到从背面看是没有绘制的。创建三角形几何体的时候设置Flag：*PxMeshGeometryFlag::eDOUBLE_SIDED*

```
	PxTriangleMeshGeometry(pTriangleMesh, PxMeshScale(PxVec3(gScale.x, gScale.y, gScale.z),
					PxQuat(gScaleRotaion.x, gScaleRotaion.y, gScaleRotaion.z, gScaleRotaion.w)), PxMeshGeometryFlag::eDOUBLE_SIDED);
```
PVD的double-sided效果，正面是可以看到斜面的了
![stair double-sided](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/stair_double_sided.jpg?raw=true)

这样在PVD里面就可以看到生效了，斜面正反面都可以看到了。但是，胶囊体还是卡在里面~

# 问题解决

* 我们的移动用的move函数，发现原因是move函数默认不支持double-sided

```
	if(PxMeshQuery::sweep(unitDir, impact.mDistance, geom, pose, nbTris, triangles, sweepHit, getSweepHintFlags(sweepTest->mUserParams), &cachedIndex))
```

* *eMESH_BOTH_SIDES*
move函数最终用的sweep函数，最后一个参数是是否支持both sides的，但是默认是false
```
	bool physx::PxMeshQuery::sweep(    const PxVec3& unitDir, const PxReal maxDistance,
									const PxGeometry& geom, const PxTransform& pose,
                                PxU32 triangleCount, const PxTriangle* triangles,
									PxSweepHit& sweepHit, PxHitFlags hintFlags_,
									const PxU32* cachedIndex, const PxReal inflation, bool doubleSided)
	{
```

* 直接修改SDK代码
最后一个参数设置为true
```
	if(PxMeshQuery::sweep(unitDir, impact.mDistance, geom, pose, nbTris, triangles, sweepHit, getSweepHintFlags(sweepTest->mUserParams), &cachedIndex, 0, true))
```

问题解决了

# 总结
* 开了double-sided，效率肯定会下降的，这个问题正确的做法是修改模型