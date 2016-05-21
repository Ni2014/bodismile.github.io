---
layout: post
title: 2.0以上的Android Studio导入so文件的正确姿势
category: 移动开发
tags: android,androidstudio
keywords: androidstudio,so
---

环境：`AndroidStudio版本：2.1.1`，`gradle版本：2.1.0`

导入步骤：

1. 将项目切换到`Project`模式下，将so文件直接复制到`app`的`libs`文件夹下面;
2. 在`app`的`build.gradle`文件的`android`选项下添加`sourceSets`设置，并点击`sync Now`，重新build该工程即可。

完整的`build.gradle`示例如下：

	apply plugin: 'com.android.application'

	android {
	    compileSdkVersion 23
	    buildToolsVersion "23.0.3"
	
	    defaultConfig {
	        applicationId "cn.smile.demo"
	        minSdkVersion 14
	        targetSdkVersion 23
	        versionCode 1
	        versionName "1.0"
	    }
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }
		//重要的是设置下源目录
	    sourceSets {
	        main {
	            jniLibs.srcDirs = ['libs']//将so文件目录指向libs目录
	        }
	    }	
	}

	dependencies {
	    compile fileTree(dir: 'libs', include: ['*.jar'])
	    testCompile 'junit:junit:4.12'
	}

**注：进行上述设置后，so文件会自动打包进apk，不需要做其他额外操作，挺方便的。**