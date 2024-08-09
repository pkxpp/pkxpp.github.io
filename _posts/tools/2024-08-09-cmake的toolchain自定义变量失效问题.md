---
layout: post
comments: true
categories: tools
tags: cmake toolchain
---


在编译我们库的时候，cmake执行到toolchain.cmake的时候，一些自定义变量是空的，但是看参数我是已经传递进去了





# 简介


## toolchain
toolchain的用途官网解释的很清楚\[1\]，主要是为了交叉编译


* Cross-compiling

> Cross-compiling a piece of software means that the software is built on one system, but is intended to run on a different system.

# 分析

## 测试用例
官网代码拿过来整一个简单的测试用例\[2\]

```cpp
MESSAGE("------------------------------------------1" ${ANDROID_NDK})
if( NOT ANDROID_NDK )
  MESSAGE("111")
else()
  MESSAGE("222")
endif()
# the name of the target operating system
set(CMAKE_SYSTEM_NAME Linux)

# which C and C++ compiler to use
set(CMAKE_C_COMPILER /home/alex/eldk-mips/usr/bin/mips_4KC-gcc)
set(CMAKE_CXX_COMPILER
    /home/alex/eldk-mips/usr/bin/mips_4KC-g++)

# location of the target environment
set(CMAKE_FIND_ROOT_PATH /home/alex/eldk-mips/mips_4KC
                          /home/alex/eldk-mips-extra-install )

# adjust the default behavior of the FIND_XXX() commands:
# search for headers and libraries in the target environment,
# search for programs in the host environment
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
MESSAGE("------------------------------------------2")
```
执行cmake命令，把自定义变量ANDROID\_NDK传递过去

```cpp
cmake -DANDROID_NDK=externals/android-ndk-r10e -DCMAKE_TOOLCHAIN_FILE=..\linux.toolchain.cmake ..
```
输出结果：

```cpp
F:\study\compile\cmake\cmake_toolchain\build>cmake -DANDROID_NDK=externals/android-ndk-r10e -DCMAKE_TOOLCHAIN_FILE=..\linux.toolchain.cmake ..
-- Building for: Visual Studio 17 2022
CMake Warning (dev) at CMakeLists.txt:1 (project):
  cmake_minimum_required() should be called prior to this top-level project()
  call.  Please see the cmake-commands(7) manual for usage documentation of
  both commands.
This warning is for project developers.  Use -Wno-dev to suppress it.

------------------------------------------1==externals/android-ndk-r10e
222
------------------------------------------2
------------------------------------------1==externals/android-ndk-r10e
222
------------------------------------------2
-- The C compiler identification is MSVC 19.36.32535.0
-- The CXX compiler identification is MSVC 19.36.32535.0
-- Detecting C compiler ABI info
------------------------------------------1==
111
------------------------------------------2
-- Detecting C compiler ABI info - done
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.36.32532/bin/Hostx64/x64/cl.exe - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
------------------------------------------1==
111
------------------------------------------2
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.36.32532/bin/Hostx64/x64/cl.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done (3.0s)
-- Generating done (0.0s)
-- Build files have been written to: F:/study/compile/cmake/cmake_toolchain/build
```
### 小结
* toolchain.cmake会执行多次\[3\]，细看的话会发现是检查不同的东西(C compiler或者C++ compiler)
* ANDROID\_NDK自定义变量在强两次是有值的，后面两次却没有了



### 原因
* 按照\[3\]\[4\]中的回答来看，首先还是因为toolchain.cmake因为try\_compile会执行多次，这个上面测试结果也可以看得出来
* 然后就是自定义变量，并不会递归的传递下去



### 解决
* 方法一：设置到环境变量即可
* 方法二：使用CMAKE\_TRY\_COMPILE\_PLATFORM\_VARIABLES\[5\]，参考\[2\]中的测试用例

```cpp
set(CMAKE_SYSTEM_NAME Linux)

set(CMAKE_TRY_COMPILE_PLATFORM_VARIABLES ANDROID_NDK)
```
再看打印结果

```cpp
F:\study\compile\cmake\cmake_toolchain\build>cmake -DANDROID_NDK=externals/android-ndk-r10e -DCMAKE_TOOLCHAIN_FILE=..\linux.toolchain.cmake ..
-- Building for: Visual Studio 17 2022
CMake Warning (dev) at CMakeLists.txt:1 (project):
  cmake_minimum_required() should be called prior to this top-level project()
  call.  Please see the cmake-commands(7) manual for usage documentation of
  both commands.
This warning is for project developers.  Use -Wno-dev to suppress it.

------------------------------------------1==externals/android-ndk-r10e
222
------------------------------------------2
------------------------------------------1==externals/android-ndk-r10e
222
------------------------------------------2
-- The C compiler identification is MSVC 19.36.32535.0
-- The CXX compiler identification is MSVC 19.36.32535.0
-- Detecting C compiler ABI info
------------------------------------------1==externals/android-ndk-r10e
222
------------------------------------------2
-- Detecting C compiler ABI info - done
-- Check for working C compiler: C:/Program Files (x86)/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.36.32532/bin/Hostx64/x64/cl.exe - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
------------------------------------------1==externals/android-ndk-r10e
222
------------------------------------------2
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: C:/Program Files (x86)/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.36.32532/bin/Hostx64/x64/cl.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done (3.0s)
-- Generating done (0.0s)
-- Build files have been written to: F:/study/compile/cmake/cmake_toolchain/build
```
可以看到每一次调用toolchain.cmake都会正确的输出自定义变量ANDROID\_NDK的值了



# 参考
\[1\]\[官网文档\]([https://cmake.org/cmake/help/book/mastering-cmake/chapter/Cross%20Compiling%20With%20CMake.html](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Cross%20Compiling%20With%20CMake.html))

\[2\]\[测试用例\]([https://github.com/pkxpp/HowToWriteCmakelistsFile/tree/main/7%20tool%20chain](https://github.com/pkxpp/HowToWriteCmakelistsFile/tree/main/7%20tool%20chain))

\[3\]\[toolchain.cmake执行多次\][https://stackoverflow.com/questions/66700152/why-is-the-toolchain-file-executed-a-few-times-in-cmake](https://stackoverflow.com/questions/66700152/why-is-the-toolchain-file-executed-a-few-times-in-cmake)

\[4\]\[toolchain变量失效\][https://stackoverflow.com/questions/28613394/check-cmake-cache-variable-in-toolchain-file/66706187#66706187](https://stackoverflow.com/questions/28613394/check-cmake-cache-variable-in-toolchain-file/66706187#66706187)

\[5\]\[CMAKE\_TRY\_COMPILE\_PLATFORM\_VARIABLES\]([https://cmake.org/cmake/help/latest/variable/CMAKE\_TRY\_COMPILE\_PLATFORM\_VARIABLES.html](https://cmake.org/cmake/help/latest/variable/CMAKE_TRY_COMPILE_PLATFORM_VARIABLES.html))