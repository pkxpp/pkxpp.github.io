---
layout: post
comments: true
categories: machinelearning
tag: DeepMimic
---

[TOC]

还是按照github主页[1]步骤大体来，以及参考[2]，步骤看起来比较简单，但是过程中还是出现了不少问题

# 编译

* 选择x64选项
* 添加头文件和链接配置
* 选择*Release_Swig*编译

最终，会生成一个*DeepMimicCore.py*的文件






# 问题解决


## 1. Windows SDK缺失

* 详细信息

```
	C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\IDE\VC\VCTargets\Microsoft.Cpp.WindowsSDK.targets(46,5): error MSB8036: 找不到 Windows SDK 版本10.0.17134.0。请安装所需的版本的 Windows SDK 或者在项目属性页中或通过右键单击解决方案并选择“重定解决方案目标”来更改 SDK 版本。
```

* 解决

去官网下载对应SDK安装

## 2.swig.exe没有找到

* 详细信息
```
	1>------ 已启动生成: 项目: DeepMimicCore, 配置: Release_Swig x64 ------
	1>Performing Custom Build Tools
	1>PYTHON_INCLUDE:
	1>PYTHON_LIB:
	1>SWIG_DIR:
	1>'\swig.exe' 不是内部或外部命令，也不是可运行的程序
	1>或批处理文件。
	1>C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\IDE\VC\VCTargets\Microsoft.CppCommon.targets(209,5): error MSB6006: “cmd.exe”已退出，代码为 9009。
	1>已完成生成项目“DeepMimicCore.vcxproj”的操作 - 失败。
```

* 解决

环境变量下面添加SWIG_DIR，为压缩包解压之后的路径，其中包含swig.exe

## 3.Eigen错误

* 详细信息

```
	1>f:\study\engine\animation\deepmimic\deepmimic\deepmimiccore\util\mathutil.h(5): fatal error C1083: 无法打开包括文件: “Eigen/Dense”: No such file or directory
	1>KinCharacter.cpp
	1>f:\study\engine\animation\deepmimic\deepmimic\deepmimiccore\util\mathutil.h(5): fatal error C1083: 无法打开包括文件: “Eigen/Dense”: No such file or directory
	1>KinTree.cpp
	1>f:\study\engine\animation\deepmimic\deepmimic\deepmimiccore\util\mathutil.h(5): fatal error C1083: 无法打开包括文件: “Eigen/Dense”: No such file or directory
	1>Motion.cpp
	1>f:\study\engine\animation\deepmimic\deepmimic\deepmimiccore\util\mathutil.h(5): fatal error C1083: 无法打开包括文件: “Eigen/Dense”: No such file or directory
	1>Shape.cpp
```

* 解决

因为我是运行Install工程，会拷贝一份到c盘，路径是这样的C:\Program Files (x86)\Eigen3\include\eigen3。但是教程说的是到include添加到头文件包含中，把前面路径添加上就ok了

## 4.glew头文件找不到

* 详细信息

```
	1>f:\study\engine\animation\deepmimic\deepmimic\deepmimiccore\render\drawmesh.h(17): fatal error C1083: 无法打开包括文件: “GL/glew.h”: No such file or directory
	1>KinCharacter.cpp
	1>f:\study\engine\animation\deepmimic\deepmimic\deepmimiccore\render\drawmesh.h(17): fatal error C1083: 无法打开包括文件: “GL/glew.h”: No such file or directory
```

* 解决

包含glew的头文件

## 5.M_PI没找到
* 详细信息

```
	1>f:\study\engine\animation\deepmimic\deepmimic\deepmimiccore\render\renderstate.h(87): warning C4312: “类型强制转换”: 从“unsigned int”转换到更大的“GLvoid *”
	1>f:\study\engine\animation\deepmimic\deepmimic\deepmimiccore\render\camera.cpp(328): error C2065: “M_PI”: 未声明的标识符
	1>f:\study\engine\animation\deepmimic\deepmimic\deepmimiccore\render\camera.cpp(329): error C2065: “M_PI”: 未声明的标识符
```

* 解决：添加一个宏_USE_MATH_DEFINES

参考：

## 6. glew32.lib找不到
* 详细信息

```
	1>LINK : fatal error LNK1181: 无法打开输入文件“glew32.lib”
```

* 解决

包含lib路径，到*.lib路径，如：F:\study\engine\animation\DeepMimic\glew-2.1.0\lib\Release\x64

## 7. 找不到BulletDynamics.lib
* 详细信息

```
	1>LINK : fatal error LNK1181: 无法打开输入文件“BulletDynamics.lib”
```
* 解决
（1）改一下配置，默认如下图所示，把_vs2010_x64_release这一截去掉

![glew_output_cofig](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/glew_output_config.png?raw=true)

（2）参考[4]改需要的静态库，下表四个即可


| 工程名 | 备注 |
|---|---|
|BulletCollision| |
|BulletDynamics	| |
|BulletSoftBody	| |
|LinearMath| |


## 8.链接错误，找不到符号
* 详细信息

```
	1>  正在创建库 F:\study\engine\animation\DeepMimic\DeepMimic\DeepMimicCore\x64\Release_Swig\_DeepMimicCore.lib 和对象 F:\study\engine\animation\DeepMimic\DeepMimic\DeepMimicCore\x64\Release_Swig\_DeepMimicCore.exp
	1>Camera.obj : error LNK2001: 无法解析的外部符号 __imp_glutGetModifiers
	1>DrawUtil.obj : error LNK2001: 无法解析的外部符号 __imp___glutCreateWindowWithExit
	1>DrawUtil.obj : error LNK2001: 无法解析的外部符号 __imp___glutInitWithExit
	1>DrawUtil.obj : error LNK2001: 无法解析的外部符号 __imp_glutInitDisplayMode
	1>DrawUtil.obj : error LNK2001: 无法解析的外部符号 __imp_glutGet
	1>DrawUtil.obj : error LNK2001: 无法解析的外部符号 __imp_glutInitWindowSize
	1>GroundPlane.obj : error LNK2001: 无法解析的外部符号 "public: __cdecl btStaticPlaneShape::btStaticPlaneShape(class btVector3 const &,float)" (??0btStaticPlaneShape@@QEAA@AEBVbtVector3@@M@Z)
	1>MultiBody.obj : error LNK2001: 无法解析的外部符号 "public: float const * __cdecl btMultiBody::getJointVelMultiDof(int)const " (?getJointVelMultiDof@btMultiBody@@QEBAPEBMH@Z)
	1>MultiBody.obj : error LNK2001: 无法解析的外部符号 "public: void __cdecl btMultiBody::setupPlanar(int,float,class btVector3 const &,int,class btQuaternion const &,class btVector3 const &,class btVector3 const &,bool)" (?setupPlanar@btMultiBody@@QEAAXHMAEBVbtVector3@@HAEBVbtQuaternion@@00_N@Z)
	1>MultiBody.obj : error LNK2001: 无法解析的外部符号 "public: __cdecl btMultiBody::btMultiBody(int,float,class btVector3 const &,bool,bool,bool)" (??0btMultiBody@@QEAA@HMAEBVbtVector3@@_N11@Z)
	1>SimBodyJoint.obj : error LNK2001: 无法解析的外部符号 "public: void __cdecl btMultiBody::addJointTorqueMultiDof(int,float const *)" (?addJointTorqueMultiDof@btMultiBody@@QEAAXHPEBM@Z)
	1>SimBodyJoint.obj : error LNK2001: 无法解析的外部符号 "public: float * __cdecl btMultiBody::getJointPosMultiDof(int)" (?getJointPosMultiDof@btMultiBody@@QEAAPEAMH@Z)
	1>SimBodyJoint.obj : error LNK2001: 无法解析的外部符号 "public: float * __cdecl btMultiBody::getJointVelMultiDof(int)" (?getJointVelMultiDof@btMultiBody@@QEAAPEAMH@Z)
	1>SimCharacter.obj : error LNK2001: 无法解析的外部符号 "public: void __cdecl btMultiBody::setupSpherical(int,float,class btVector3 const &,int,class btQuaternion const &,class btVector3 const &,class btVector3 const &,bool)" (?setupSpherical@btMultiBody@@QEAAXHMAEBVbtVector3@@HAEBVbtQuaternion@@00_N@Z)
	1>SimCharacter.obj : error LNK2001: 无法解析的外部符号 "public: void __cdecl btMultiBody::setupRevolute(int,float,class btVector3 const &,int,class btQuaternion const &,class btVector3 const &,class btVector3 const &,class btVector3 const &,bool)" (?setupRevolute@btMultiBody@@QEAAXHMAEBVbtVector3@@HAEBVbtQuaternion@@000_N@Z)
	1>SimCharacter.obj : error LNK2001: 无法解析的外部符号 "public: void __cdecl btMultiBody::setupPrismatic(int,float,class btVector3 const &,int,class btQuaternion const &,class btVector3 const &,class btVector3 const &,class btVector3 const &,bool)" (?setupPrismatic@btMultiBody@@QEAAXHMAEBVbtVector3@@HAEBVbtQuaternion@@000_N@Z)
	1>SimCharacter.obj : error LNK2001: 无法解析的外部符号 "public: void __cdecl btMultiBody::setupFixed(int,float,class btVector3 const &,int,class btQuaternion const &,class btVector3 const &,class btVector3 const &,bool)" (?setupFixed@btMultiBody@@QEAAXHMAEBVbtVector3@@HAEBVbtQuaternion@@00_N@Z)
	1>SimCharacter.obj : error LNK2001: 无法解析的外部符号 "public: __cdecl btMultiBodyJointLimitConstraint::btMultiBodyJointLimitConstraint(class btMultiBody *,int,float,float)" (??0btMultiBodyJointLimitConstraint@@QEAA@PEAVbtMultiBody@@HMM@Z)
	1>SimJoint.obj : error LNK2001: 无法解析的外部符号 "public: void __cdecl btAngularLimit::set(float,float,float,float,float)" (?set@btAngularLimit@@QEAAXMMMMM@Z)
	1>SimJoint.obj : error LNK2001: 无法解析的外部符号 "public: float __cdecl btHingeConstraint::getHingeAngle(void)" (?getHingeAngle@btHingeConstraint@@QEAAMXZ)
	1>SimRigidBody.obj : error LNK2001: 无法解析的外部符号 "public: void __cdecl btRigidBody::setDamping(float,float)" (?setDamping@btRigidBody@@QEAAXMM@Z)
	1>World.obj : error LNK2001: 无法解析的外部符号 "public: virtual float __cdecl btCollisionShape::getContactBreakingThreshold(float)const " (?getContactBreakingThreshold@btCollisionShape@@UEBAMM@Z)
	1>World.obj : error LNK2001: 无法解析的外部符号 "public: virtual float __cdecl btCollisionShape::getAngularMotionDisc(void)const " (?getAngularMotionDisc@btCollisionShape@@UEBAMXZ)
	1>World.obj : error LNK2001: 无法解析的外部符号 "public: virtual void __cdecl btCollisionShape::getBoundingSphere(class btVector3 &,float &)const " (?getBoundingSphere@btCollisionShape@@UEBAXAEAVbtVector3@@AEAM@Z)
	1>World.obj : error LNK2001: 无法解析的外部符号 "public: virtual void __cdecl btSphereShape::calculateLocalInertia(float,class btVector3 &)const " (?calculateLocalInertia@btSphereShape@@UEBAXMAEAVbtVector3@@@Z)
	1>World.obj : error LNK2001: 无法解析的外部符号 "public: virtual void __cdecl btConvexShape::project(class btTransform const &,class btVector3 const &,float &,float &,class btVector3 &,class btVector3 &)const " (?project@btConvexShape@@UEBAXAEBVbtTransform@@AEBVbtVector3@@AEAM2AEAV3@3@Z)
	1>World.obj : error LNK2001: 无法解析的外部符号 "public: virtual float __cdecl btMultiBodyConstraintSolver::solveGroupCacheFriendlyFinish(class btCollisionObject * *,int,struct btContactSolverInfo const &)" (?solveGroupCacheFriendlyFinish@btMultiBodyConstraintSolver@@UEAAMPEAPEAVbtCollisionObject@@HAEBUbtContactSolverInfo@@@Z)
	1>World.obj : error LNK2001: 无法解析的外部符号 "public: virtual float __cdecl btMultiBodyConstraintSolver::solveGroup(class btCollisionObject * *,int,class btPersistentManifold * *,int,class btTypedConstraint * *,int,struct btContactSolverInfo const &,class btIDebugDraw *,class btDispatcher *)" (?solveGroup@btMultiBodyConstraintSolver@@UEAAMPEAPEAVbtCollisionObject@@HPEAPEAVbtPersistentManifold@@HPEAPEAVbtTypedConstraint@@HAEBUbtContactSolverInfo@@PEAVbtIDebugDraw@@PEAVbtDispatcher@@@Z)
	1>World.obj : error LNK2001: 无法解析的外部符号 "protected: virtual float __cdecl btMultiBodyConstraintSolver::solveSingleIteration(int,class btCollisionObject * *,int,class btPersistentManifold * *,int,class btTypedConstraint * *,int,struct btContactSolverInfo const &,class btIDebugDraw *)" (?solveSingleIteration@btMultiBodyConstraintSolver@@MEAAMHPEAPEAVbtCollisionObject@@HPEAPEAVbtPersistentManifold@@HPEAPEAVbtTypedConstraint@@HAEBUbtContactSolverInfo@@PEAVbtIDebugDraw@@@Z)
	1>World.obj : error LNK2001: 无法解析的外部符号 "protected: virtual float __cdecl btMultiBodyConstraintSolver::solveGroupCacheFriendlySetup(class btCollisionObject * *,int,class btPersistentManifold * *,int,class btTypedConstraint * *,int,struct btContactSolverInfo const &,class btIDebugDraw *)" (?solveGroupCacheFriendlySetup@btMultiBodyConstraintSolver@@MEAAMPEAPEAVbtCollisionObject@@HPEAPEAVbtPersistentManifold@@HPEAPEAVbtTypedConstraint@@HAEBUbtContactSolverInfo@@PEAVbtIDebugDraw@@@Z)
	1>World.obj : error LNK2001: 无法解析的外部符号 "public: __cdecl btCapsuleShape::btCapsuleShape(float,float)" (??0btCapsuleShape@@QEAA@MM@Z)
	1>World.obj : error LNK2001: 无法解析的外部符号 "protected: virtual float __cdecl btSequentialImpulseConstraintSolver::solveGroupCacheFriendlyIterations(class btCollisionObject * *,int,class btPersistentManifold * *,int,class btTypedConstraint * *,int,struct btContactSolverInfo const &,class btIDebugDraw *)" (?solveGroupCacheFriendlyIterations@btSequentialImpulseConstraintSolver@@MEAAMPEAPEAVbtCollisionObject@@HPEAPEAVbtPersistentManifold@@HPEAPEAVbtTypedConstraint@@HAEBUbtContactSolverInfo@@PEAVbtIDebugDraw@@@Z)
	1>DeepMimicCore.obj : error LNK2001: 无法解析的外部符号 __imp_glutPostRedisplay
	1>F:\study\engine\animation\DeepMimic\DeepMimic\DeepMimicCore\_DeepMimicCore.pyd : fatal error LNK1120: 33 个无法解析的外部命令
```

* 解决，参考[6]

（1）glut的符号找不到，是因为编译的是32位的，重新用cmake生成一个x64的，然后编译出来就没有了
（2）bullet的符号找不到，是因为双精度问题，去掉宏


至此，我编译过了DeepMimicCore，高兴一下！！！

# 参考
[1][DeepMimic github主页](https://github.com/xbpeng/DeepMimic)

[2][windows从零搭建OpenGL freeglut环境](https://blog.csdn.net/linian71/article/details/68485494)

[3][VS2017 C++ 程序报错“error C2065: “M_PI”: 未声明的标识符"](https://blog.csdn.net/liu_feng_zi_/article/details/84816763)

[4][windows下Bullet 2.82编译安装（Bullet Physics开发环境配置）](https://www.cnblogs.com/liangliangh/p/3575590.html)

[5][Build error on Windows 10 (error LNK2001: unresolved external symbol)](https://github.com/xbpeng/DeepMimic/issues/75)