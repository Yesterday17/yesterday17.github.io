---
layout: _posts
title: Flutter开发初探（1）——环境配置中撞的坑
date: 2018-07-02 23:05:35
category: 开发
tags: flutter
---

大家好，好久不见，我是某昨。  
最近看到[Flutter](https://flutter.io/)挺有趣的，于是准备上手玩玩，正好也是想做个 Android 应用的，于是借这次机会先开始配置环境。

## 一、配置 Android 开发环境

首当其冲的是 `Android` 的开发环境。最新版的 `Android SDK` 已经不提供带有 GUI 的下载，需要通过魔法手段获得最后一个带 GUI 的 `Android SDK`，并且以一种玄学的方式进行更新，下面一一列举：

首先是获取`Android SDK`:

> [The latest version of it(Windows).](https://dl.google.com/android/installer_r24.4.1-windows.exe)

(来源：https://stackoverflow.com/questions/41407396/is-gui-for-android-sdk-manager-gone)

然后是升级 `Android SDK` 时可能会遇到的玄学错误：

> Warning: An error occurred during installation: Failed to move away or delete existing target file: \<The Path Where Your Android SDK Tools are\> Move it away manually and try again..

因此，你需要复制一份`Android SDK Tools`，并且在复制的目录下运行：

> sdkmanager.bat --sdk_root=\<sdkRootPath\> --update

最后是环境变量，你需要设置`$ANDROID_HOME`，指向`Android SDK`的安装目录。

(来源：https://stackoverflow.com/questions/43796568/cant-update-tools-android-sdk-command-line-tools-for-windows)  
这样就基本可以解决 `Android SDK` 的问题了。当然了，如果有网络原因的话。请自行梯子。

## 二、配置编辑器

Flutter 支持两种编辑器：`Intellij IDEA`(包括`Android Studio`)，以及`VSCode`。官方的文档[在此](https://flutter.io/get-started/editor/)。我选用的是`VSCode`，因此在这里，我也只对 `VSCode` 进行阐述。

1.  首先，你需要安装好最新版的`VSCode`。
2.  其次，你需要下载`Flutter`。你可以在[这里](https://flutter.io/sdk-archive/#windows)找到最新的`Flutter SDK`。  
    当然，如果网页无法打开，你也可以使用国内的镜像。下面列举几个国内的镜像。

    - 上海交通大学  
      https://mirrors.sjtug.sjtu.edu.cn/
    - 阿里源  
      https://storage.flutter-io.cn/

    当然了，你也可以选择使用国内的全镜像网站： https://flutter-io.cn/

3.  将`Flutter`加入 Path。

4.  安装`VSCode`插件：搜索`Flutter`即可。

5.  在命令中输入`doctor`，找到`Flutter: Run Flutter Doctor`。

    默认的命令快捷键是`Ctrl+Shift+P`。  
    在使用`Run Flutter Doctor`时，你也可能会遇到这个问题：

    > Android license status unknown.

    在这时候，你只需要运行`flutter doctor --android-licenses`，然后一路`y`就行了。因为之前我们解决了`Android SDK`的更新问题，因此这里应该是没有问题的。如果提示版本过旧，只要根据上述操作更新即可。

由此，`Flutter`的开发环境便构建完成。如果有问题欢迎留言~  
（话说如果留言坏掉了可以私信/邮件我。。。
