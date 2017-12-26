---
title: FastCV Demo in Android Studio
date: 2017-12-25 22:27:41
tags:
- Android
- FastCV
- JNI
- cmake
- Android Studio
layout:
updated:
comments: true
categories:
- FastCV
permalink:
---

{% asset_img FastCV.jpg %}
> 版权声明：本文为 *冬夏* 原创文章，可以随意转载，但请注明出处。

最近的项目需要在 Android 手机上进行摄像头视频采集、处理。所以需要使用上计算机视觉的库。一开始选择使用 OpenCV 开源库，具体使用参照我之前的文章
[AndroidStudio使用OpenCV的三种方式](https://dongxiawu.github.io/2017/12/14/AndroidStudio%E4%BD%BF%E7%94%A8OpenCV%E7%9A%84%E4%B8%89%E7%A7%8D%E6%96%B9%E5%BC%8F/)

但是 OpenCV 的速度并不能达到要求，同时无意中发现了一个叫做 FastCV 的计算机视觉库，于是就想试一下。但是发现网络上几乎没有关于 FastCV 的 Demo 在 Android Studio 上成功编译的先例，于是只好利用 google、stackoverflow 和 FastCV 官网的自己自己尝试编译，踩了一些坑，最后终于搞定了。

虽然最终因为某些原因并没有使用 FastCV 的 SDK，但是相信这些经验会对一些人有帮助，所以在这里分享给大家。

<!--more-->

FastCV 是一个计算机视觉函数库，可以看做的是迷你版的 OpenCV，是高通公司开发的基于高通芯片的 SDK，目前支持的操作系统有：
* Android
* Linux
* Windows RT
* Windows Phone 8

FastCV 最大的特点就是快，这是因为高通公司针对自己的芯片做了特别的优化的缘故。缺点也特别明显，就是 FastCV 是闭源的，只提供 API，并不提供源码，而且只能运行在高通的芯片上。

由于 FastCV 已经很久没有更新版本了，上一次更新是 16年3月，所以 FastCV 的 Demo 还是基于 eclipse 的，而且不能直接导入 Android Studio，所以需要移植。

关于 Android JNI 的构建方式，目前有 ndk-build 和 cmake 两种方式，谷歌推荐 cmake，本工程也采用cmake

开发环境: Android Studio3.0 + Ubuntu 16.04 LTS

# 搭建开发环境 #

1. 首先在 FastCV 的官网上注册并下载 FastCV SDK
<https://developer.qualcomm.com/software/fastcv-sdk>

2. 在下载目录下运行

  2.1 修改文件权限

  `sudo chmod a+x fastcv-installer-android-1-7-1.bin`

  2.2 安装软件

  `sudo ./fastcv-installer-android-1-7-1.bin`

  注意，安装 FastCV SDK 需要电脑已经安装 JDK，并且目前还不支持 JDK9，如果你的电脑安装的是 JDK9，那么请记得先切换一下 JDK 版本

# 移植 FastCV 代码 #

1. 新建 Android Studio 工程（我选择移植的是 fastcvdemo 工程，所以新建工程时包名可以仿照 fastcvdemo 工程包名，减少工作量）

2. 将 fastcvdemo/jni 内头文件复制到 project root/app/src/main/cpp/include 文件夹下

   将 fastcvdemo/jni 内源文件复制到 project root/app/src/main/cpp/source 文件夹下，

   将 fastcvdemo/res 内文件复制到 project root/app/src/main/res 文件夹下

   将 fastcvdemo/src 内文件复制到 project root/app/src/main/java 文件夹下

   将 sdk root/lib/Android/lib64/libfastcv.a复制到 project root/app/src/main/jniLibs 文件夹下

# 编写 cmake 脚本

为了让 Android Studio 能正确编译我们的程序，需要编写一个脚本告诉 Android Studio 我们的代码结构，这就是 cmake 的作用

1. 在 project root/app 目录下新建 CMakeLists.txt 文件，并添加以下内容:

      >#指定 cmake 的最小版本，确保能使用某些新特性构建项目
      cmake_minimum_required(VERSION 3.4.1)

      >#输出详细信息
      set(CMAKE_VERBOSE_MAKEFILE on)

      >#设置头文件目录
      set(INCLUDE_DIR "${CMAKE_SOURCE_DIR}/src/main/cpp/include")
      #设置源文件目录
      set(SOURCE_DIR "${CMAKE_SOURCE_DIR}/src/main/cpp/source")
      #设置库目录
      set(LIBRARY_DIR "${CMAKE_SOURCE_DIR}/src/main/jniLibs")

      >#包含头文件
      include_directories(${INCLUDE_DIR})

      >#添加 fastcv 静态库
      add_library(fastcv-lib STATIC IMPORTED)
      set_target_properties( fastcv-lib
      PROPERTIES IMPORTED_LOCATION
      "${LIBRARY_DIR}/libfastcv.a")

      >#添加 log 库
      find_library( log-lib log )

      >find_path(GLES2_INCLUDE_DIR GLES2/gl2.h
      HINTS ${ANDROID_NDK})
      include_directories(${GLES2_INCLUDE_DIR})

      >find_library(GLES2_LIBRARY libGLESv2.so
      HINTS ${GLES2_INCLUDE_DIR}/../lib)

      >#包含子目录
      add_subdirectory(${SOURCE_DIR})

  由于 cmake 具有递归查找功能，所以要在每个包含的子目录下也新建 CMakeLists.txt 文件，具体添加内容查看 Demo

2. 在app目录下的build.gradle 内添加：

        android {

          defaultConfig {
            ...

            externalNativeBuild {
                cmake {
                    arguments "-DANDROID_ARM_NEON=TRUE",
                            "-DVAR_NAME=ARG_1 ARG_2",
                            "-DANDROID_CPP_FEATURES=rtti exceptions"
                    cppFlags "-std=c++11", "-frtti", "-fexceptions"
                }
            }

            ndk {
                //只支持高通指令集
                abiFilters 'arm64-v8a'
            }
        }

        externalNativeBuild {
            cmake{
                path "CMakeLists.txt"
            }
          }
        }


# 总结 #
到这里，就可以正常编译并运行 FastCV Demo 了，Demo 提供的各种图像处理操作都能达到实时性的效果，但是由于 FastCV SDK 的缺点很明显，只支持高通 CPU 和不开源，所以网上资料少也就不奇怪了。


关于FastCV Demo，我上传到了 github上，请自行下载。
<https://github.com/dongxiawu/FastcvDemo>


参考资料：

[1] <https://developer.qualcomm.com/software/fastcv-sdk>

[2] <https://cmake.org/cmake-tutorial/>

[3] <https://developer.android.google.cn/studio/projects/add-native-code.html?hl=zh-cn>
