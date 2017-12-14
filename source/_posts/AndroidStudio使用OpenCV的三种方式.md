---
title: AndroidStudio使用OpenCV的三种方式
date: 2017-12-14 10:08:07
tags:
- Android
- OpenCV
- JNI
- cmake
- Android Studio
layout:
updated:
comments: true
categories:
- OpenCV
permalink:
---

{% asset_img AndroidStudio使用OpenCV的三种方式.jpg %}
> 版权声明：本文为 *冬夏* 原创文章，可以随意转载，但请注明出处。

最近的项目需要在 Android 手机上使用 OpenCV 开源库，于是到网上找教程。没想到网上的教程大多数是 eclipse 的，还有很多是错的，最后还是靠 Stack Overflow、 OpenCV 官网和 Google 解决了这个问题，在这里记录一下，希望能帮到有需要的人。

要在 Android 上使用 OpenCV，总体上来说有三种方法

1. 使用 OpenCV Manager + OpenCV Android SDK
2. 使用 OpenCV Android SDK + OpenCV 动态库
3. 使用 OpenCV 动态库 + JNI

开发环境: Android Studio3.0 + Ubuntu 16.04 LTS

<!--more-->

## 使用 OpenCV Manager + OpenCV Android SDK ##

这种方法最简单，但是也对用户最不友好，因为需要多安装 OpenCV Manager 这个APK，适合在快速验证功能的时候使用。

其实这种方法的原理就是利用 Android 的 Service 来进行跨进程通信，主要流程如下：

{% asset_img OpenCV Manager 工作原理.png %}

具体操作步骤如下：

1. 新建 Android 工程

2. 下载 OpenCV-Android-SDK，下载地址: https://opencv.org/releases.html

3. 点击 File -> new -> Import Module, 选中 OpenCV-Android-SDK/sdk/java 文件夹,点击确定，就会自动识别处模块，如下图所示：
{% asset_img 导入OpenCV.png %}
导入完成后，会在工程目录下发现 OpenCV 的库，settings.gradle 文件也会相应改变

4. 点击 File -> Project Structure，在 Modules 下点击 依赖 OpenCV 的模块（比如 app ），再点击右边的 Dependencies。点击右侧加号选第三个 Module dependency 后选择 openCVLibrary310 后点击完成。
{% asset_img 添加opencv依赖.png %}
完成后会发现 app目录下的 build.gradle 中的 dependencies 增加了 oepncv 的依赖。
到这里，我们就可以在程序里面使用 OpenCV 的类了。

5. 由于 OpenCV 库的版本一般情况下和你的工程的版本号不同，所以需要把 OpenCV 库目录下的 build.gradle 中的编译版本，构建版本等参数设置成和工程一样。

6. 根据手机安装对应的 OpenCV Manager apk，apk 在 OpenCV-Android-SDK/apk 目录下。可以将 apk 复制到手机内存卡里安装，也可以将手机连上电脑，然后通过命令行安装，安装命令：adb install [OpenCV_xxx.apk]

7. 在使用 OpenCV 功能前 使用
OpenCVLoader.initAsync(String Version, Context AppContext,
LoaderCallbackInterface Callback) 这个方法进行初始化，初始化成功之后就可以正常使用 OpenCV 库了。

## 使用 OpenCV Android SDK + OpenCV 动态库 ##

第一种方法有点就是简单，缺点就是必须要安装 OpenCV Manager apk。要用户同时安装两个 apk 并不太现实，所以第一种方法不适合在实际项目中使用。而采用 OpenCV Android SDK + OpenCV 动态库 的方法就可以避免这个问题。

主要步骤（1-5点都和之前一样）：

1. 新建 Android 工程

2. 下载 OpenCV-Android-SDK，下载地址: https://opencv.org/releases.html

3. 点击 File -> new -> Import Module, 选中 OpenCV-Android-SDK/sdk/java 文件夹,点击确定，就会自动识别处模块，如下图所示：
{% asset_img 导入OpenCV.png %}
导入完成后，会在工程目录下发现 OpenCV 的库，settings.gradle 文件也会相应改变

4. 点击 File -> Project Structure，在 Modules 下点击 依赖 OpenCV 的模块（比如 app ），再点击右边的 Dependencies。点击右侧加号选第三个 Module dependency 后选择 openCVLibrary310 后点击完成。
{% asset_img 添加opencv依赖.png %}
完成后会发现 app目录下的 build.gradle 中的 dependencies 增加了 oepncv 的依赖。
到这里，我们就可以在程序里面使用 OpenCV 的类了。

5. 由于 OpenCV 库的版本一般情况下和你的工程的版本号不同，所以需要把 OpenCV 库目录下的 build.gradle 中的编译版本，构建版本等参数设置成和工程一样

6. 将 OpenCV-Android-SDK/sdk/native/libs 目录下全部内容复制到 工程目录/app/src/main/jniLibs 目录下（这里可以针对特定的手机做裁剪，为了方便可以全部复制）

7. 在使用 OpenCV 功能前 使用下面代码加载 OpenCV 动态库
        static {
            System.loadLibrary("opencv_java3");
        }
   然后使用
        OpenCVLoader.initDebug();
   进行初始化。初始化成功之后，就可以正常使用 OpenCV 进行开发了。


## 使用 OpenCV 动态库 + JNI ##

第二种方法由于不需要安装 OpenCV Manager，同时又是使用 OpenCV Android SDK 在 Java 层进行开发，故开发速度非常快，十分适合一般的引用场景。但该方法也有几个缺点：

1. 使用 OpenCV Android SDK 会增大 APK 的体积。
2. 由于在 Java 层进行开发，在某些场景下会导致运行效率不高。

因此引出了第三中方法。该方法使用 OpenCV 动态库 + JNI 的方式进行开发，适合在对计算效率要求比较高的场景下使用。，但由于采用了 JNI（Java Native Interface）技术，开发的难度也是最大的。

主要步骤：

1. 新建 Android 工程

2. 下载 OpenCV-Android-SDK，下载地址: https://opencv.org/releases.html

3. 下载相关工具

  3.1 在打开的项目中，从菜单栏选择 Tools -> Android -> SDK Manager。

  3.2 点击 SDK Tools 标签。

  3.3 选中 LLDB、CMake 和 NDK 旁的复选框，如下图所示。
  {% asset_img JNI相关工具.png %}

4. 将 OpenCV-Android-SDK/sdk/native/libs 目录下全部内容复制到 工程目录/app/src/main/jniLibs 目录下（这里可以针对特定的手机做裁剪，为了方便可以全部复制）

5. 创建新的原生源文件

  要在应用模块的主源代码集中创建一个包含新建原生源文件的 cpp/ 目录，并新建 source 和 include 的子目录。在 source 目录下新建源文件 native-lib.cpp

6. 复制头文件

    将 OpenCV-Android-SDK/sdk/native/jni/include 目录下全部内容复制到 工程目录/app/src/main/cpp/include 目录下

7. 编辑 CMakeLists.txt

  在模块根目录下新建文件 CMakeLists.txt，并添加以下内容：
  > #指定 cmake 的最小版本，确保能使用某些新特性构建项目
  cmake_minimum_required(VERSION 3.4.1)

  >#输出详细信息
  set(CMAKE_VERBOSE_MAKEFILE on)

  >#设置库目录
  set(LIBRARY_DIRS "${CMAKE_SOURCE_DIR}/src/main/jniLibs")

  >#包含头文件目录
  include_directories(src/main/cpp/include)

  >#添加 opencv 的动态库

  >add_library( libopencv_java3 SHARED IMPORTED )

  >#指定库路径

  >set_target_properties( libopencv_java3 IMPORTED_LOCATION "${LIBRARY_DIRS}/${ANDROID_ABI}/libopencv_java3.so" )

  >#设置源文件目录
  aux_source_directory(src/main/cpp/source SOURCE_DIR)

  >add_library( native-lib SHARED ${SOURCE_DIR} )

  >find_library( log-lib log )

  >target_link_libraries( native-lib libopencv_java3 ${log-lib} )

8. 修改 build.gradle
  在模块的 build.gradle 内的 android 闭包内添加：

        // Encapsulates your external native build configurations.
        externalNativeBuild {

            // Encapsulates your CMake build configurations.
            cmake {

                // Provides a relative path to your CMake build script.
                path "CMakeLists.txt"
            }
        }

9. 编写 JNI 接口和 C++ 代码。

  上面 CMakeLists.txt 的作用是将 opencv 的动态库和 native-lib.cpp 链接起来生成新的动态库。

  到目前位置，我们已经能够在 native-lib.cpp 内使用 C++ 编写 OpenCV 程序了。

  接下来的步骤为：
  1. 在 Java 层编写 native 方法
  2. 使用 javah -jni 生成本地接口
  3. 在 C++ 层实现接口
  4. 在 Java 层调用 native 方法。

  由于这部分是普通的 JNI 使用，不属于 opencv 的内容，所以不在这里展开了。

# 总结

到这里，就介绍了 Android 使用 OepnCV 的三种方式。其中我个人推荐第二种方法。

关于第三种方法，我采用的是 cmake 构建，虽然也可以采用 ndk 的方式，但是我个人推荐的是 cmake，因为 cmake 更加简单，应用范围也更广泛，谷歌目前推荐的构建方式也是 cmake。

关于三种方法的 Demo，我上传到了 github上，请自行下载。
<https://github.com/dongxiawu/OpencvDemo>


参考资料：

[1] <https://www.cnblogs.com/yunfang/p/6149831.html>

[2] <https://docs.opencv.org/2.4/doc/tutorials/introduction/android_binary_package/dev_with_OCV_on_Android.html>

[3] <https://docs.opencv.org/2.4.13.4/platforms/android/service/doc/UseCases.html#first-application-start>

[4] <https://developer.android.google.cn/studio/projects/add-native-code.html?hl=zh-cn>

[5] <https://cmake.org/cmake/help/v3.10/>

[6] <http://blog.csdn.net/Jason101123/article/details/78600355>
