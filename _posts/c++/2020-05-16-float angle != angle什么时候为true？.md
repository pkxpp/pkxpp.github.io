---
layout: post
comments: true
categories: c++
tags: c++ 浮点数
---

[TOC]

看到一段代码，顿时就呆了

```
    if (angle != angle)
    {
        return XMQuaternionIdentity();
    }
```

这样写的用意到底是啥呢？





# 源代码

代码很好理解，就是求两个向量的四元数表示的旋转角度

```
DirectX::XMVECTOR SimStarMan::QuaternionFromToRotation(const XMVECTOR& from, const XMVECTOR& to)
{
    XMVECTOR quatRet;
    XMVECTOR axis, fromN, toN;
    float angle;
    fromN = XMVector3Normalize(from);
    toN = XMVector3Normalize(to);
    const XMVECTOR epsilon = XMLoadFloat3(&XMFLOAT3(0.00001f, 0.00001f, 0.00001f));

    if (XMVector3NearEqual(fromN, toN, epsilon))
    {
        return XMQuaternionIdentity();
    }
    axis = XMVector3Cross(fromN, toN);
    axis = XMVector3Normalize(axis);
    angle = acosf(XMVectorGetX(XMVector3Dot(fromN, toN)));
    if (angle != angle)
    {
        return XMQuaternionIdentity();
    }
    quatRet = XMQuaternionRotationAxis(axis, angle);
    quatRet = XMQuaternionNormalize(quatRet);
    return quatRet;
}
```

# 探究
## NAN
参考[2]中解释了，说IEEE标准规定，NaN和任何数比较都是false，所以只有当angle为NaN的时候， angle != angle才为true

```
float a = acos(2);
cout << "a = " << a << endl;    // a = nan
cout << (a != a) << endl;         // 1
```

## 向量可能有问题
比如某个向量是无限长，正如*XMVector3Normalize*文档提到的一样[3]

> For a vector of length 0, this function returns a zero vector. For a vector with infinite length, it returns a vector of QNaN.

## acosf会出现Nan的情况
如果输入范围有问题，那么结果就会出现NaN，第二条提到了[4]

```
If no errors occur, the arc cosine of arg (arccos(arg)) in the range [0 ; π], is returned.

If a domain error occurs, an implementation-defined value is returned (NaN where supported).

If a range error occurs due to underflow, the correct result (after rounding) is returned.

```

## 如何写更优雅？
参考[2]中有个评论是这么回答的：

> This answer should be updated since std::isnan is now part of the C++11 standard and support has spread out. std::isnan was implemented in Visual Studio starting with Visual Studio 2013.

所以，上面的写法是不是写成下面这样，会更加清晰明了：

```
// old
angle ! = angle
// new
std::isnan(angle)
```

至此，这个问题算是清楚了，那么NaN到底是如何定义的呢？

## NaN定义
参考[5][6]

> IEEE 754 floating point numbers can represent positive or negative infinity, and NaN (not a number).

**小结：**

* NaN在运算中得结果始终都是NaN，不像infinity有可能算出来一个正常值，比如 4/&infin; = 0
* NaN的浮点数IEEE表示是一个范围，节码都是1，所以尾数表示的范围都是NaN

```
Positive infinity is represented by the bit pattern X'7F80 0000'.
Negative infinity is represented by the bit pattern X'FF80 0000'.
A signaling NaN (NANS) is represented by any bit pattern between X'7F80 0001' and X'7FBF FFFF' or between X'FF80 0001' and X'FFBF FFFF'.
A quiet NaN (NANQ) is represented by any bit pattern between X'7FC0 0000' and X'7FFF FFFF' or between X'FFC0 0000' and X'FFFF FFFF'.

     31
     |
     | 30    23 22                    0
     | |      | |                     |
-----+-+------+-+---------------------+
qnan 0 11111111 10000000000000000000000
snan 0 11111111 01000000000000000000000
 inf 0 11111111 00000000000000000000000
-inf 1 11111111 00000000000000000000000
-----+-+------+-+---------------------+
     | |      | |                     |
     | +------+ +---------------------+
     |    |               |
     |    v               v
     | exponent        fraction
     |
     v
     sign

```

# 参考
[1][When is a float variable not equal to itself](https://stackoverflow.com/questions/32900284/when-is-a-float-variable-not-equal-to-itself/32900315)

[2][Checking if a double (or float) is NaN in C++](https://stackoverflow.com/questions/570669/checking-if-a-double-or-float-is-nan-in-c/570694#570694)

[3][XMVector3Normalize](https://docs.microsoft.com/en-us/windows/win32/api/directxmath/nf-directxmath-xmvector3normalize)

[4][acos, acosf, acosl](https://en.cppreference.com/w/c/numeric/math/acos)

[5][Infinity and NaNs](https://www.doc.ic.ac.uk/~eedwards/compsys/float/nan.html)

[6][20.5.2 Infinity and NaN](https://www.gnu.org/software/libc/manual/html_node/Infinity-and-NaN.html)

[7][IEEE User's Guide](https://www.cenapad.unicamp.br/parque/manuais/Xlf/UG42.HTM)
[8][What is difference between quiet NaN and signaling NaN?](https://stackoverflow.com/questions/18118408/what-is-difference-between-quiet-nan-and-signaling-nan)