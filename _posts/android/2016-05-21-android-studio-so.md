## 2.0以上的Android Studio导入so文件的正确姿势

本人使用的AS版本：2.1.1，gradle版本：2.1.0

导入步骤：

1. 将项目切换到`Project`模式下，将so文件直接复制到`app`的`libs`文件夹下面;
2. 在`app`的`build.gradle`文件的`android`选项下添加如下设置，并点击`sync Now`，重新build该工程即可。

    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }

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
	    sourceSets {
	        main {
	            jniLibs.srcDirs = ['libs']
	        }
	    }	
	}

	dependencies {
	    compile fileTree(dir: 'libs', include: ['*.jar'])
	    testCompile 'junit:junit:4.12'
	}