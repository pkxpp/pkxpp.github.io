---
layout: post
comments: true
categories: engine
tags: engine PhysX
---

[TOC]

在做碰撞检测的时候，返回结果一般用*hasBlock*就可以知道是否碰撞了，但是发现还有一个*hadInitialOverlap*的时候有点疑惑。这个是啥意思？干什么用的？





# *hasBlock*和*hadInitialOverlap*
* *hasBlock*

在HitBuffer的PxHitCallback中

```
template<typename HitType>
struct PxHitCallback
{
    HitType        block;            //!< Holds the closest blocking hit result for the query. Invalid if hasBlock is false.
    bool        hasBlock;        //!< Set to true if there was a blocking hit during query.

    HitType*    touches;        //!< User specified buffer for touching hits.
```

* *hadInitialOverlap*

在PxLocationHit结构体中

```
struct PxLocationHit : public PxQueryHit
{
    PX_INLINE            PxLocationHit() : flags(0), position(PxVec3(0)), normal(PxVec3(0)), distance(PX_MAX_REAL)    {}

    /**
    \note For raycast hits: true for shapes overlapping with raycast origin.
    \note For sweep hits: true for shapes overlapping at zero sweep distance.

    @see PxRaycastHit PxSweepHit
    */
    PX_INLINE bool        hadInitialOverlap() const { return (distance <= 0.0f); }
```

# 测试
拿Sweep测试，测试代码如下

```
    PxSweepBuffer hitSweep;
    PxSphereGeometry sweepShape = PxSphereGeometry(30);
    PxHitFlags hitFlags = PxHitFlags(PxHitFlag::eDEFAULT);
    PxQueryFilterData filterData = PxQueryFilterData(PxQueryFlag::eSTATIC);
    status = gScene->sweep(sweepShape, t, unitDir, PxReal(0.0f), hitSweep, hitFlags, filterData);
    bool bInitOverlap = hitSweep.block.hadInitialOverlap();
```

通过修改半径和Sweep距离来看其中情况

## 情况1：初始的时候没有发生碰撞，Sweep过程中也没有发生碰撞

* 参数

半径 = 30
距离 = 0

* PVD图

![1](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/physx-overlap1.jpg?raw=true)

* 结果

```
hitSweep.hasBlock = false
hitSweep.block.hadInitialOverlap() = false
```

## 情况2：初始的时候没有发生碰撞，Sweep过程中发生碰撞

* 参数

半径 = 30
距离 = 100

* PVD图

![2](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/physx-overlap2.jpg?raw=true)

* 结果

```
hitSweep.hasBlock = true
hitSweep.block.hadInitialOverlap() = false
```

## 情况3：初始的时候发生碰撞，Sweep距离为0

* 参数

半径 = 100
距离 = 0

* PVD图

![3](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/physx-overlap3.jpg?raw=true)

* 结果

```
hitSweep.hasBlock = true
hitSweep.block.hadInitialOverlap() = true
```

# 总结
* 1.是否碰撞用hasBlock就可以了

* 2.有些需求如果想知道一开始就发生碰撞了的话，就用hadInitialOverlap。而且hasBlock也是生效的