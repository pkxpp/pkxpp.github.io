---
layout: post
comments: true
categories: json
tags: rapidjson
---

[TOC]

# 需求
想把一个数组存成Object形式，所以索引需要从整形转字符串





# 问题
## 1.如何优雅的进行转换
最后也没找到合适的方法

```
// 有问题的写法
doc.AddMember(rapidjson::StringRef(std::to_string(i).c_str()), strContent, allocator);

// 正确的写法
rapidjson::Value strIndex;
strIndex.SetString(std::to_string(i).c_str(), allocator);
doc.AddMember(strIndex, strContent, allocator);
```

* 测试代码
```
    rapidjson::Value strIndex;
    rapidjson::Value strContent(rapidjson::kStringType);
    std::string strIndex1;
    for (int i = 0; i <2; ++i)
    {
        strContent.SetString("test");
        
        // way1: error
        //doc.AddMember(rapidjson::StringRef(std::to_string(i).c_str()), strContent, allocator);

        // way2: error
        strIndex1 = std::to_string(i);
        doc.AddMember(rapidjson::StringRef(strIndex1.c_str()), strContent, allocator);

        // way3: correct
        strIndex.SetString(std::to_string(i).c_str(), allocator);
        doc.AddMember(strIndex, strContent, allocator);
    }
```
## 2.\0000的乱码形式
使用三种方式得到的结果分别如下：

```
// way1
{
    "\u0000": "test",
    "\u0000": "test"
}
// way2
{
    "1": "test",
    "1": "test"
}
// way3
{
    "0": "test",
    "1": "test"
}
```
**原因说明：**

参考[1]，前两种方法都是用的*StringRef*，而这个只是存了字符串的指针，所以出现了不同的结果：

* 方法1，在出了循环之后临时变量就销毁了，所以并不认是字符串；
* 方法2，两次循环指的是同一个地址，所以索引都是最后的1
* 方法3，是猜用copy的方式，在c++里面就是值传递或者深拷贝，是安全的一种方式
# 总结
1. 使用rapidjson在int转字符串的方法写起来还是有点冗长的
2. 使用字符串的时候要特别小心*StringRef*的坑
# 参考
[1][rapidjson中string使用的一点小坑](https://blog.csdn.net/wwwasw/article/details/76473165)