---
layout: post
comments: true
categories: c++
tags: c++ imgui vsnprintf
---

[TOC]

Imgui显示utf-8编码的中文，但是使用InputText输入的中文是乱码？






# 查虫
## Imgui.InputText显示的内容来源
在imgui_impl_win32.cpp的*ImGui_ImplWin32_WndProcHandler*中处理输入法，把输入的字符通过接口**io.AddInputCharacter**添加到Imgui的buff里面，然后Imgui.InputText在绘制的时候把里面的内容绘制出来

因为输入的是GBK编码，所以在**io.AddInputCharacter**之前需要把文本转成utf-8编码才行。参考[1][2][3].
其实，输入的是什么编码，决定的是VS里面设置的是多字符集还是unicode字符集。

# 解决
先理解几个问题
## 1.vs里面的字符集: unicode字符集和多字节字符集
参考[4]，简单来说

* unicode字符集：里面的每一个字符，英文/中文都是用两个字节宽带来表示的

* 多字节字符集：里面的每一个字符，要么用一个字节表示（英文），要么使用多个字节表示（中文两个字节）

自己测试了一下：在input text控件里面输入中文：啊。在两种字符集下面wParam得到的编码是不一样的

```
// 多字节字符集
0xB0A1    -- 对应的是GBK编码十六进制表示
// unicode字符集
0x554a     -- 对应的是Unicode编码十六进制表示
```

## 2.WM_CHAR/WM_UNICHAR/WM_IME_CHAR/
参考[4][5]

* WM_CHAR：不使用输入法时候输入字符的消息。**如果窗口时unicode，不管输入英文还是中文，只发一个消息，wParam就是字符；如果窗口是ANSI，会发送两个WM_CHAR消息，两个字节合在一起就是一个中文编码**

* WM_UNICHAR：

* WM_IME_CHAR：使用输入法时候输入字符的消息。**如果窗口时unicode，同WM_CHAR一样，不管输入的英文还是中文，都只发一次消息；如果窗口时ANSI，不管输入的英文和中文也同样只发一个消息。**

## 3.CP_ACP/CP_OEMCP代码页
参考[6]
简体中文、繁体中文这两个都是一样的

## 问题解决
针对两种字符集，解决方案如下。
* 多字节字符集

修改输入接收地方的代码，vs设置的字符集是*多字节字符集*，参考[3]中也解释了一下

```
    case WM_CHAR:
    {
        DWORD wChar = wParam;
        if (wChar <= 127)
        {
            io.AddInputCharacter(wChar);
        }
        return 0;
    }
    case WM_IME_CHAR:
    {
        auto& io = ImGui::GetIO();
        DWORD wChar = wParam;
        if (wChar <= 127)
        {
            io.AddInputCharacter(wChar);
        }
        else
        {
            // swap lower and upper part.
            BYTE low = (BYTE)(wChar & 0x00FF);
            BYTE high = (BYTE)((wChar & 0xFF00) >> 8);
            wChar = MAKEWORD(high, low);
            wchar_t ch[6];
            MultiByteToWideChar(CP_ACP, 0, (LPCSTR)&wChar, 4, ch, 3);
            io.AddInputCharacter(ch[0]);
        }
        return 0;
    }
```

* unicode字符集

vs的字符集设置为unicode字符集，都不需要改东西
```
        if (wParam > 0 && wParam < 0x10000)
            io.AddInputCharacterUTF16((unsigned short)wParam);
```

不过注意下，在example里面如果要显示中文的话，需要加一下中文字体。在mian.cpp里面

```
io.Fonts->AddFontFromFileTTF("c:\\Windows\\Fonts\\ArialUni.ttf", 18.0f, NULL, io.Fonts->GetGlyphRangesChineseFull());
```

## 小结
* Imgui是不管你的环境的，它要求InputText输入的Unicode编码，也就是宽字符，可以从上面代码看出来，参考[3]作者也提到

![input text ](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/imgui_input_text.png?raw=true)

这也就是在vs设置为多字符集的时候才需要转换，如果设置为unicode字符集的时候根本就不需要。

# 参考
[1][issues](https://github.com/ocornut/imgui/issues/471)
[2][Q: How can I display and input non-Latin characters such as Chinese, Japanese, Korean, Cyrillic?](https://github.com/ocornut/imgui/blob/master/docs/FAQ.md#q-how-can-i-display-and-input-non-latin-characters-such-as-chinese-japanese-korean-cyrillic)
[3][Problem with chinese or japanese input #1807](https://github.com/ocornut/imgui/issues/1807)
[4][WM_IME_CHAR 与WM_CHAR的区别](https://blog.csdn.net/shuilan0066/article/details/7679825)
[5][WM_CHAR,WM_UNICHAR,WM_IME_CHAR](https://blog.csdn.net/aaa000830/article/details/79585090)
[6][MultiByteToWideChar函数中的CP_ACP和CP_OEMCP参数](https://www.cnblogs.com/C-Sharp2/articles/3143634.html)
[7][Visual Studio——使用多字节字符集与使用Unicode字符集](https://blog.csdn.net/huashuolin001/article/details/95620424)