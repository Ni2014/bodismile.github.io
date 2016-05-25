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

错误原因：`重复编译包造成的`

错误分析：

我的`app`下的`build.gradle`文件如下：

	android {
	    compileSdkVersion 22
	    buildToolsVersion "22.0.1"
	
	    defaultConfig {
	        minSdkVersion 16
	        targetSdkVersion 22
	        versionCode 5
	        versionName "2.0.5"
	    }
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }
	    lintOptions {
	        abortOnError false
	    }
	}
	
	dependencies {
		//1
	    compile fileTree(dir: 'libs', include: ['*.jar'])
	    testCompile 'junit:junit:4.12'
		//2
	    compile files('libs/所导入的jar包名')
	}

 通过`Add As Library`方法添加jar包后，as都会默认在`dependencies`标签下生成`compile files('libs/所导入的jar包名')`这样一行语句，但是`compile fileTree(dir: 'libs', include: ['*.jar'])`已经`compile`了`libs`下面的所有jar，这样就造成了重复编译。

**解决方式：**

	删除AS生成的所有的`compile files('libs/所导入的jar包名')`，重新build下。

注：此方法对你来说不一定管用。

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