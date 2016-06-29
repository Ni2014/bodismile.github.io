---
layout: post
title: Android Studio 编译错误
category: 移动开发
tags: android
keywords: android studio,Ignoring InnerClasses attribute for an anonymous inner class,Not a PNG file
---

整理AS编译过程中的出现的一些错误

## Ignoring InnerClasses attribute for an anonymous inner class

错误描述：Android studio 导入`jar`到`libs`目录后,编译出现`Ignoring InnerClasses attribute for an anonymous inner class`警告，但程序能够正常运行：

	Error:warning: **Ignoring InnerClasses attribute for an anonymous inner class**
	Error:(cn.bmob.v3.util.The) that doesn't come with an
	Error:associated EnclosingMethod attribute. This class was probably produced by a
	Error:compiler that did not target the modern .class file format. The recommended
	Error:solution is to recompile the class from source, using an up-to-date compiler
	Error:and without specifying any "-target" type options. The consequence of ignoring
	Error:this warning is that reflective operations on this class will incorrectly
	Error:indicate that it is *not* an inner class.

错误原因：`该jar文件在混淆过程中将该匿名内部类的某个属性给混淆啦`

**解决方式：**

	需要重新混淆jar文件，并将警告的那些类都不要混淆。

	-keep public class cn.bmob.v3.util.*{
	    public protected <fields>;
	    public protected <methods>;
	}

## libpng error: Not a PNG file

有时候程序运行出现以下错误：

	AAPT err(Facade for 277489958): libpng error: Not a PNG file
	Error:Execution failed for task ':app:mergeDebugResources'.
	> Some file crunching failed, see logs for details

**解决方法：**

在`app`的build.gradle文件的`android`标签下添加如下两句：
	
	aaptOptions.cruncherEnabled = false
    aaptOptions.useNewCruncher = false

示例文件如下：

	apply plugin: 'com.android.application'

	android {

	    compileSdkVersion 22
	    buildToolsVersion "22.0.1"
	
	    //去掉Android对PNG图片的校验
	    aaptOptions.cruncherEnabled = false
	    aaptOptions.useNewCruncher = false

		......
	}