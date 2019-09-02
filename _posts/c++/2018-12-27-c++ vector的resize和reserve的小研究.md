---
layout: post
comments: true
categories: c++
---

[TOC]

time: 2018/12/27


# 问题描述
今天遇到一个问题，有同事使用vector的reserve并且直接取第一个元数来用，伪代码如下：





	m_arrsAP.reserve(uCount);
	AP* pAP = &m_arrsAP[0];

结果报错如下:

```
vector subscript out of range
```
当时想到的是size和capacity的区别，所以把代码改成resize，的确也就ok了。但是，也产生了如下一些问题：
疑问如下:
1. 为什么在release下面没有，而在debug下面报错？
2. size和capacity区别的意义是什么？
3. resize和reserve区别的意义又是什么？
4. reserve会改变空间大小，那么地址会变，得去验证一下。

# release和debug的区别
看一下代码就很容易知道了，Debug下面Level为2的时候才有，如果设置了vs的Level也就不会报错。
```
    reference operator[](size_type _Pos)
        {    // subscript mutable sequence
 #if _ITERATOR_DEBUG_LEVEL == 2
        if (size() <= _Pos)
            {    // report error
            _DEBUG_ERROR("vector subscript out of range");
            _SCL_SECURE_OUT_OF_RANGE;
            }

 #elif _ITERATOR_DEBUG_LEVEL == 1
        _SCL_SECURE_VALIDATE_RANGE(_Pos < size());
 #endif /* _ITERATOR_DEBUG_LEVEL */

        return (*(this->_Myfirst() + _Pos));
        }
```

# size和capacity
这两个区别大家一般都知道，size是当前vector里面的元数个数，capacity是当前vector最多可容纳的元数个数，size <= capacity

# resize和reserve
size和capacity区别了解了之后，resize和reserve就自然是改变这两个值相关操作的。**但是这两个函数的实现最终还是有很多看点的**，往下看：

## resize
源码：
```
    void resize(size_type _Newsize)
        {    // determine new length, padding as needed
        if (_Newsize < size())
            _Pop_back_n(size() - _Newsize);
        else if (size() < _Newsize)
            {    // pad as needed
            _Reserve(_Newsize - size());
            _TRY_BEGIN
            _Uninitialized_default_fill_n(this->_Mylast(), _Newsize - size(),
                this->_Getal());
            _CATCH_ALL
            _Tidy();
            _RERAISE;
            _CATCH_END
            this->_Mylast() += _Newsize - size();
            }
        }
```
**说明：**
* 如果_Newsize比现在的size小的话，把多余的元数pop掉
* 如果_Newsize大于现在的size的话，_Reserve负责创建不够的空间，这个时候内存会发生变化
* *_Uninitialized_default_fill_n*函数负责把新增加的空间用第一个元数填充
## _Reserve
```
    void _Reserve(size_type _Count)
        {    // ensure room for _Count new elements, grow exponentially
        if (_Unused_capacity() < _Count)
            {    // need more room, try to get it
            if (max_size() - size() < _Count)
                _Xlen();
            _Reallocate(_Grow_to(size() + _Count));
            }
        }
```
**说明：**
* max_size自己打印的时候是1073741823，所以这里只是检查了一下
* *_Grow_to*函数试图增加50%的capacity的大小，如果_Count比capacity + capacity/2还大，就直接用_Count
* *_Reallocate*重新分配空间，地址会发生变化

## reserve
```
    void reserve(size_type _Count)
        {    // determine new minimum length of allocated storage
        if (capacity() < _Count)
            {    // something to do, check and reallocate
            if (max_size() < _Count)
                _Xlen();
            _Reallocate(_Count);
            }
        }
```
竟然和_Reserve差不多代码~

# 问题回答
测试代码：
```
void VECTOR_TEST::TestSizeAndCapacity()
{
    PrintSizeAndCapacity();

    m_vecTest.reserve(10);
    PrintSizeAndCapacity();

    m_vecTest.resize(5);
    PrintSizeAndCapacity();

    m_vecTest.resize(10);
    PrintSizeAndCapacity();

    m_vecTest.push_back(1);
    PrintSizeAndCapacity();

    m_vecTest.resize(5);
    PrintSizeAndCapacity();

    m_vecTest.reserve(5);
    PrintSizeAndCapacity();
}
```
结果：
```
max size: 1073741823 vector size: 0 vector capacity: 0
max size: 1073741823 vector size: 0 vector capacity: 10
max size: 1073741823 vector size: 5 vector capacity: 10
address: 035104F8
max size: 1073741823 vector size: 10 vector capacity: 10
address: 035104F8
max size: 1073741823 vector size: 11 vector capacity: 15
address: 0351A050
max size: 1073741823 vector size: 5 vector capacity: 15
address: 0351A050
max size: 1073741823 vector size: 5 vector capacity: 15
address: 0351A050
```

## 1. 为什么在release下面没有，而在debug下面报错？
看源码
## 2. size和capacity区别的意义是什么？
就是我们所理解的意义，size是满足动态的需求
## 3. resize和reserve区别的意义又是什么？
reserve和resize其实在内存分配的时候做的事情是一样的，但是reserve几乎都会发生内存分配，从而转移数据，比较耗，所以尽量用resize，这是两者区别的最大意义所在。
## 4. reserve会改变空间大小，那么地址会变，得去验证一下
有，看输出结果



