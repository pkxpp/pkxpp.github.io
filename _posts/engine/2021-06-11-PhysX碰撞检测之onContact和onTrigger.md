---
layout: post
comments: true
categories: engine
tags: engine PhysX
---

[TOC]

之前是看文档，大概知道这种用法，今天跑了下例子，记录一下





# 步骤
## 实现回调函数

PhysX总的来说有两种碰撞检测的用法

* （1）主动：自己调用raycast、sweep以及overlap函数去检测
* （2）被动：写回调函数，场景里面一旦发生碰撞，就会回调自己实现的函数里面，做自己的逻辑

* PxSimulationEventCallback

继承这个纯虚类，实现其中的两个函数

```
class PhysXCallback :public PxSimulationEventCallback {
public:
     PhysXCallback();
    virtual void                            onContact(const PxContactPairHeader& pairHeader, const PxContactPair* pairs, PxU32 nbPairs);
    virtual void                            onTrigger(PxTriggerPair* pairs, PxU32 count);
    virtual void                            onConstraintBreak(PxConstraintInfo*, PxU32) {}
    virtual void                            onWake(PxActor**, PxU32) {}
    virtual void                            onSleep(PxActor**, PxU32) {}
    virtual void                            onAdvance(const PxRigidBody*const*, const PxTransform*, const PxU32) {}
};
```

* 注册

把自己写的类new一个实例，丢给*PxSceneDesc*结构，创建物理场景的时候就等于是注册进去了

```
class PxSceneDesc
{
public:
    // 省略...
    /**
    \brief Possible notification callback.
    This callback will be associated with the client PX_DEFAULT_CLIENT.
    Please use PxScene::setSimulationEventCallback() to register callbacks for other clients.
    <b>Default:</b> NULL
    @see PxSimulationEventCallback PxScene.setSimulationEventCallback() PxScene.getSimulationEventCallback()
    */
    PxSimulationEventCallback*    simulationEventCallback;
    /**
    \brief Possible asynchronous callback for contact modification.
    <b>Default:</b> NULL
    @see PxContactModifyCallback PxScene.setContactModifyCallback() PxScene.getContactModifyCallback()
    */
    PxContactModifyCallback*    contactModifyCallback;
    /**
    \brief Possible asynchronous callback for contact modification.
    <b>Default:</b> NULL
    @see PxContactModifyCallback PxScene.setContactModifyCallback() PxScene.getContactModifyCallback()
    */
    PxCCDContactModifyCallback*    ccdContactModifyCallback;
            // 省略...
}
```
可以看到还有另外两个回调函数可以用来注册

* PxShapeFlag::eTRIGGER_SHAPE

只是完成上面两个，加上这个标记*PxShapeFlag::eTRIGGER_SHAPE*，onTrigger就算是ok了。这个标记是说一旦两个碰撞的物件，其中有一个打上了这个标记，就会触发onTrigger事件，从而可以回调到这个自己注册的函数中

```
shape->setFlag(PxShapeFlag::eTRIGGER_SHAPE, true);

struct PxPairFlag
{
    enum Enum
    {
        // 省略
        /**
        \brief Call contact report callback or trigger callback when this collision pair starts to be in contact.
        If one of the two collision objects is a trigger shape (see #PxShapeFlag::eTRIGGER_SHAPE) 
        then the trigger callback will get called as soon as the other object enters the trigger volume. 
        If none of the two collision objects is a trigger shape then the contact report callback will get 
        called when the actors of this collision pair start to be in contact.
        \note Only takes effect if the colliding actors are rigid bodies.
        \note Only takes effect if eDETECT_DISCRETE_CONTACT or eDETECT_CCD_CONTACT is raised
        @see PxSimulationEventCallback.onContact() PxSimulationEventCallback.onTrigger()
        */
        eNOTIFY_TOUCH_FOUND                    = (1<<2),
        // 省略。。。
```

* PxPairFlag

只是完成上面两个，onContact是不会触发的，即使物体已经发生碰撞了。要想解决这个问题，需要重写*FilterShader*函数，并且返回的PxPairFlag要带上标记*PxPairFlag::eNOTIFY_TOUCH_FOUND*才行。

* FilgerShader函数

```
static physx::PxFilterFlags defaultPhysX3FilterShader(
    physx::PxFilterObjectAttributes attributes0,
    physx::PxFilterData filterData0,
    physx::PxFilterObjectAttributes attributes1,
    physx::PxFilterData filterData1,
    physx::PxPairFlags &pairFlags, 
    const void * constantBlock ,
    physx::PxU32 constantBlockSize)
{
// morphe me used filter callback
    bool kinematic0 = physx::PxFilterObjectIsKinematic(attributes0);
    bool kinematic1 = physx::PxFilterObjectIsKinematic(attributes1);
    if (kinematic0 && kinematic1)
    {
        pairFlags = physx::PxPairFlags(); 
        return physx::PxFilterFlag::eSUPPRESS;
    }
    // Support the idea of this being called from a user's filter shader - i.e. don't trample on existing flags
    pairFlags = physx::PxPairFlag::eCONTACT_DEFAULT;
    pairFlags = pairFlags | physx::PxPairFlag::eSOLVE_CONTACT
        | physx::PxPairFlag::eNOTIFY_TOUCH_FOUND
        | physx::PxPairFlag::eNOTIFY_CONTACT_POINTS;
    {
        // now test groups (word0) against ignore groups (word1)
        if ((filterData0.word0 & filterData1.word1) || (filterData1.word0 & filterData0.word1)) 
        {
            pairFlags = physx::PxPairFlags(); // disable all dynamic collisions
            return physx::PxFilterFlag::eSUPPRESS; // this stops the pair being actually passed into ray probe onObjectQuery
        }
        else
        {
            return physx::PxFilterFlag::eDEFAULT;
        }
    }
}
```

# 参考
