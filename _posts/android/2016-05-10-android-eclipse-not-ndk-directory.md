---
layout: post
title: Eclipse配置NDK路径时出现 'Not a valid NDK directory' 错误
category: 移动开发
tags: android
keywords: android,Not a valid NDK directory
---

## 设置NDK路径

在最新的ADT版本`adt-bundle-windows-x86-20140702`中没有集成ndk，会造成在`Windows->Preferences->Android`下没有出现NDK设置项。解决方法如下：

1. 下载eclipse的ndk的插件：`com.android.ide.eclipse.ndk_23.0.2.1259578.jar`
2. 将下载好的`com.android.ide.eclipse.ndk_23.0.2.1259578.jar`复制到`你的adt安装目录下的eclipse->plugins`文件夹内并重启eclipse

## Not a valid NDK directory错误

错误原因：`下载的ndk版本过高造成的`

我使用的ADT版本：`adt-bundle-windows-x86_64-20140702`，我下载的是`android-ndk-r11b`，换成`android-ndk-r10e`版本后在配置NDK路径时，eclipse就不会提示这个错误了。