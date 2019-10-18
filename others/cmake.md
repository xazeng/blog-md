---
title: cmake 技巧
categories: 未分类
date: 2019-10-10 15:59:55
tags: [software, cmake, vs, install]
---

cmake 相关内容和使用 cmake 编译软件的记录。
<!--more-->

------
## opencv

CMAKE_INSTALL_PREFIX    指定安装目录
BUILD_opencv_world      编译成一个文件，不然会有很多分散的库
BUILD_EXAMPLES          编译例子
```
    cmake -S ./ -B ./build -G "Visual Studio 15 2017 Win64" -DCMAKE_INSTALL_PREFIX="d:/Lib/opencv" -DBUILD_opencv_world=OFF -DBUILD_EXAMPLES=ON
```

安装：vs -> build -> batch build，勾选两个 INSTALL 就可以了
<https://www.cnblogs.com/lzhu/p/8198654.html>

查找：CMAKE_PREFIX_PATH="d:/Lib"

