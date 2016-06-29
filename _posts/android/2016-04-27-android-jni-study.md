---
layout: post
title: JNI学习笔记
category: 移动开发
tags: android
keywords: android,ndk,jni,java
---

## JNI数据类型

	Java 	Native(jni.h)
	boolean jboolean
	byte 	jbyte
	char 	jchar
	short 	jshort
	int	    jint
	long 	jlong
	float	jfloat
	double 	jdouble

注：
1. Java中的返回值void和JNI中的void是完全对应的 
2. Java中的基本数据类型（boolean, byte, char, short, int, long, float, double），在JNI中对应的数据类型只要在前面加上 `j `就对应了（jboolean, jbyte, jchar, jshort, jint, jlong, jfloat, jdouble） 
3. Java中的对象，包括类库中定义的类、接口以及自定义的类接口，都对应于JNI中的 jobject
4. Java中基本数据类型的数组对应与JNI中的 jarray 类型。（type就是上面说的8种基本数据类型）
5. Java中对象的数组对应于JNI中的 jobjectArray 类型。（在Java中一切对象、接口以及数组都是对象）

## 操作Java的数组

JNI可以通过JNIEnv提供的方法，对传过来的Java数组进行相应的操作。

### 操作Java的简单型数组

	JNIEnv提供的函数          数组类型	        本地类型
	GetBooleanArrayElements	jbooleanArray	jboolean
	GetByteArrayElements	jbyteArray	    jbyte
	GetCharArrayElements	jcharArray	    jchar
	GetShortArrayElements	jshortArray	    jshort
	GetIntArrayElements	    jintArray	    jint
	GetLongArrayElements	jlongArray	    jlong
	GetFloatArrayElements	jfloatArray	    jfloat
	GetDoubleArrayElements	jdoubleArray	jdouble

例子：

	JNIEXPORT jint JNICALL Java_IntArray_sumArray(JNIEnv *env, jobject obj, jintArray arr)  
	{  
	    jint *carr;  
	    carr = (*env)->GetIntArrayElements(env, arr, false);   //获得Java数组arr的引用的指针  
	    if(carr == NULL) {  
	        return 0; /* exception occurred */  
	    }  
	    jint sum = 0;  
	    for(int i=0; i<10; i++) {  
	        sum += carr[i];  
	    }  
	    (*env)->ReleaseIntArrayElements(env, arr, carr, 0);  
	    return sum;  
	}  

### 操作对象类型数组

在C/C++代码中，int类型的数组对应JNI中的jintArray，而类似字符串数组这种类型的，在Jni里对应的使用`jobjectArray`来声明

	GetObjectArrayElement(JNIEnv *env, jobjectArray array, jsize index)--返回对应索引值的object.返回的是一个数组元素的值。

	SetObjectArrayElement(JNIEnv* env, jobjectArray array, jsize index, jobject value)--用来设置对应索引元素的值  

使用`SetXXXArrayRegion`与`GetXXXArrayRegion`就是以复制的方式设置与取出Array数组中的某个值。

例如：

	GetIntArrayRegion(array,jsize start,jsize len,*buf)--把jintArray中的元素复制到buffer中
	
	SetIntArrayRegion(array,jsize start,jsize len,*buf)--把buf中的元素copy到jintArray中去


### 操作二维数组和String数组

在Jni中，二维数组和String数组都被视为object数组，因为Array和String被视为object。

下面例子实现了构造并返回一个二维int数组：

	JNIEXPORT jobjectArray JNICALL Java_ObjectArrayTest_initInt2DArray(JNIEnv *env, jclass cls, int size)  {  
	    jobjectArray result;  
	    jclass intArrCls = （*env）->FindClass(env, "[I");              //int数组的class  
	    result = (*env)->NewObjectArray(env, size, intArrCls, NULL);    //二维int数组的实例  
	    for (int i = 0; i < size; i++) {                                //初始化  
	        jint tmp[256];                                   
	        for(int j = 0; j < size; j++) {  
	            tmp[j] = i + j;  
	        }  
	        jintArray iarr = (*env)->NewIntArray(env, size);  
	        (*env)->SetIntArrayRegion(env, iarr, 0, size, tmp);     //将tmp复制到iarr中  
	        (*env)->SetObjectArrayElement(env, result, i, iarr);  
	        (*env)->DeleteLocalRef(env, iarr);  
	    }  
	    return result;  
	}
  
注：当你使用对数组进行访问后，要确保调用相应的ReleaseXXXArrayElements函数，参数是对应Java数组和GetXXXArrayElements返回的指针，如果必要的话，这个释放函数会复制你做的任何变化（这样它们就反射到java数组），然后释放所有相关的资源，避免发生内存泄漏。


## 使用Java对象

	GetFieldID	        得到一个实例的域的ID
	GetStaticFieldID 	得到一个静态的域的ID
	GetMethodID	        得到一个实例的方法的ID
	GetStaticMethodID	得到一个静态方法的ID

以下是c代码：

```
jclass cls = (*env)->FindClass(env, "Lpackagename/classname;"); 
jmethodID methodid = (*env)->GetMethodID(env, cls, "", "(D)V");  
jobject obj = (*env)->NewObjectA(env, cls, id, args);  //获得一实例，args是构造函数的参数，它是一个jvalue*类型。  

```

步骤：

1、获得class引用，jclass cls = (*env)->FindClass(env, "Lpackagename/classname;");

作用：创建一个java类的class引用，注意参数：`Lpackagename/classname;`，其中`L`代表这是在描述一个对象类型，`packagename/classname;`-该class在Java中的的路径，注意一定要以分号`；`结束。

2、获取函数的id，jmethodID methodid = env->GetMethodID(cls, "函数名", "函数签名");第一个是刚刚获得的class引用，第二个是方法的名称，第三个是方法的签名了

## 函数签名

	类型        符号
	boolean	    Z
	byte	    B
	char	    C
	short	    S
	int	        I
	long	    L
	float	    F
	double	    D
	void	    V
	object对象	LClassName; L类名;
	Arrays	    [array-type [数组类型
	methods方法	(argument-types)return-type：(参数类型)返回类型

注：

1、方法参数或者返回值为java中的对象时，签名中必须以`L`加上其路径，且路径必须以`/`分开，自定义的对象也必须遵循本规则：

例如：java.lang.String为“java/lang/String”，com.smile.bean.Student为"Lcom/smile/bean/Student;"

2、方法参数或者返回值为数组类型时，请前加上`[`。


	例1：`[I`表示int[],`[[[D` 表示 double[][][]，即几维数组就加几个[

	例2：（）Ljava/lang/String    --String f()
         (IL/java/lang/Class;)L  --long f(int i,Class c)
         ([B)V                   --String(byte[] bytes)

## 调用Java对象的方法

1、获取你需要访问的Java对象的类：
  
	jclass cls = (*env)->GetObjectClass(env, obj);       // 使用GetObjectClass方法获取obj对应的jclass。   
	jclass cls = (*env)->FindClass(“android/util/log”)   // 直接搜索类名，需要是static修饰的类。  
 
2、获取MethodID：

	jmethodID methodid = (*env)->GetMethodID(env, cls, "方法名", "方法签名"); //GetStaticMethodID(…)，获取静态方法的ID使用

参数解释： 
	env     -->JNIEnv 
	cls     -->第一步获取的jclass 
	callback-->要调用的方法名 
	(I)V    -->函数签名
 
3、调用方法：

	(*env)->CallVoidMethod(env, obj, methodid, params);// CallStaticIntMethod(….) ， 调用静态方法  

参数解释： 

	env-->JNIEnv 
	obj-->通过本地方法传过来的jobject 
	mid-->要调用的MethodID（即第二步获得的MethodID） 
	params-->方法需要的参数（对应方法的需求，添加相应的参数）
 
注：这里使用的是`CallVoidMethod`方法调用，因为没有返回值，如果有返回值的话使用对应的方法，在后面会提到。

	CallVoidMethod                    CallStaticVoidMethod  
	CallIntMethod                     CallStaticVoidMethod  
	CallBooleanMethod                 CallStaticVoidMethod  
	CallByteMethod                    CallStaticVoidMethod  

## JNI操作Java的String对象

从java程序中传过去的`String`对象在本地方法中对应的是`jstring类型`，`jstring类型`和`c`中的`char*`不同，所以如果你直接当做char*使用的话，就会出错。

因此在使用之前需要将`jstring`转换成为`c/c++`中的`char*`，这里使用JNIEnv提供的方法转换。

	const char *str = (*env)->GetStringUTFChars(env, jstr, 0);
	(*env)->ReleaseStringUTFChars(env, jstr, str);

这里使用`GetStringUTFChars`方法将传进来的`jstring类型`的jstr转换成为UTF-8格式，就能够在本地方法中使用了。

注：在使用完你所转换之后的对象之后，需要显示调用`ReleaseStringUTFChars`方法，让JVM释放转换成UTF-8的string的对象的空间，如果不显示的调用的话，JVM中会一直保存该对象，不会被垃圾回收器回收，因此就会导致内存溢出。

	GetStringUTFChars          将jstring转换成为UTF-8格式的char*
	GetStringChars             将jstring转换成为Unicode格式的char*
	ReleaseStringUTFChars      释放指向UTF-8格式的char*的指针
	ReleaseStringChars         释放指向Unicode格式的char*的指针
	NewStringUTF               创建一个UTF-8格式的String对象
	NewString                  创建一个Unicode格式的String对象
	GetStringUTFLength         获取UTF-8格式的char*的长度
	GetStringLength            获取Unicode格式的char*的长度

## Android.mk文件

### 变量解释

你应该定义在`include $(CLEAR_VARS)`和`include $(BUILD_XXXXX)`之间定义。

	include $(CLEAR_VARS) 

CLEAR_VARS这个变量由编译器提供，并且指明了一个GNU Makefile文件，这个功能会清理掉所有以LOCAL_开头的内容（例如 LOCAL_MODULE、LOCAL_FILES、LOCAL_STATIC_LIBRARIES等），除了LOCAL_PATH。这句话是必须的，因为如果所有的变量都是全局的，所有的可控的编译文件都需要在一个单独的GNU中被解析并执行。 

	LOCAL_PATH 表示此时位于工程目录的根目录中。你必须在Android.mk的开头定义，可以这样使用：LOCAL_PATH := $(call my-dir) 

这个变量不会被$(CLEAR_VARS)清除，(call my-dir)的功能由编译器提供，被用来返回当前目录的地址（这里的当前目录里包含Android.mk这个文件本身）。

	LOCAL_MODULE 变量必须被定义，用来区分android.mk中的每一个模块。文件名必须是唯一的，不能有空格。注意，这里编译器会为你自动加上一些前缀和后缀，来保证文件是一致的。

你必须在包含任一的$(BUILD_XXXX)脚本之前定义它。模块的名字决定了生成文件的名字，

例如，如果一个共享库模块的名字是<foo>，那么生成文件的名字就是lib<foo>.so。但是，在你的NDK生成文件中（或者Android.mk或者Application.mk），你应该只涉及(引用)有正常名字的其他模块。

	LOCAL_SRC_FILES 变量必须包含一个C、C++或者java源文件的列表，这些会被编译并聚合到一个模块中。只要列出要传递给编译器的文件，因为编译系统自动计算依赖。注意源代码文件名称都是相对于LOCAL_PATH的，你可以使用路径部分。

例如：LOCAL_SRC_FILES := foo.c toto/bar.c\ 

文件之间可以用空格或Tab键进行分割,换行请用"\"，如果是追加源代码文件的话，请用LOCAL_SRC_FILES +=。 

注意：可以LOCAL_SRC_FILES := $(call all-subdir-java-files)这种形式来包含local_path目录下的所有java文件。 

	LOCAL_OVERRIDES_PACKAGES 此变量可以使其他的模块不加入编译，如源码中DeskClock的android.mk有LOCAL_OVERRIDES_PACKAGES := AlarmClock，使 AlarmClock不会加入到编译系统中，不会生成 AlarmClock.apk。 

	LOCAL_STATIC_LIBRARIES 填写应该链接到这个模块的静态库列表（使用BUILD_STATIC_LIBRARY生成），这仅仅对共享库模块才有意义。

	LOCAL_SHARED_LIBRARIES 填写这个模块在运行时要依赖的共享库模块列表，在链接时需要，在生成文件时嵌入的相应的信息。注意：你仍然需要在Application.mk中把它们添加到程序要求的模块中。
 
	LOCAL_LDLIBS 编译你的模块要使用的附加的链接器选项。这对于使用”-l”前缀传递指定库的名字是有用的。

例如，LOCAL_LDLIBS := -lz表示告诉链接器生成的模块要在加载时刻链接

LOCAL_LDLIBS链接的库不产生依赖关系，一般用于不需要重新编译的库，如库不存在，则会报错找不到。且貌似只能链接那些存在于系统目录下本模块需要连接的库。如果某一个库既有动态库又有静态库，那么在默认情况下是链接的动态库而非静态库。 

如：LOCAL_LDLIBS += -lm –lz –lc -lcutils –lutils –llog  
如果需要指定链接库的路径则直接在后面写 -L再加路径即可 

	BUILD_EXECUTABLE 来生成一个可执行文件 
	BUILD_SHARED_LIBRARY 来生成一个动态库； 
	BUILD_STATIC_LIBRARY 来生成一个静态的库； 
	BUILD_PACKAGE 来生成一个APK；

这些变量是由系统提供的，并且制定给GNU Makefile的脚本，它可以收集所有你定义的“include $(CLEAR_VARS)”中以LOCAL_开头的变量，并且决定哪些要被编译，哪些应该做的更加准确。

	include $(call all-subdir-makefiles) 它的作用就是包含所有子目录中的Android.mk文件 

如果需要编译的模块比较多，我们可能会将对应的模块放置在相应的目录中， 
这样，我们可以在每个目录中定义对应的Android.mk文件（类似于上面的写法）， 
最后，在根目录放置一个Android.mk文件，加入include $(call all-subdir-makefiles)

	LOCAL_C_INCLUDES 填写所需要包含的头文件路径，是从根目录开始的

例如：LOCAL_C_INCLUDES := $(LOCAL_PATH)/../foo
需要在任何包含LOCAL_CFLAGS / LOCAL_CPPFLAGS标志之前。
    
	LOCAL_MODULE_PATH/LOCAL_UNSTRIPPED_PATH 指定最后的目标安装路径

不同的文件系统路径用以下的宏进行选择：

	TARGET_ROOT_OUT：表示根文件系统。
	
	TARGET_OUT：表示system文件系统。
	
	TARGET_OUT_DATA：表示data文件系统。

用法：LOCAL_MODULE_PATH:=$(TARGET_ROOT_OUT)
 
	
### NDK提供的函数宏: 

	my-dir :返回当前 Android.mk 所在的目录的路径，相对于 NDK 编译系统的顶层。这是有用的，在 Android.mk 文件的开头如此定义： 
LOCAL_PATH := $(call my-dir) 
	all-subdir-makefiles : 返回一个位于当前'my-dir'路径的子目录中的所有Android.mk的列表。 
	this-makefile :  返回当前Makefile 的路径(即这个函数调用的地方) 
	parent-makefile :  返回调用树中父 Makefile 路径。即包含当前Makefile的Makefile 路径。 
	grand-parent-makefile ：返回调用树中父Makefile的父Makefile的路径 

注：

在Android.mk中，

	`:=`是赋值的意思；`
	`+=`是追加的意思；
	`$`表示引用某变量的值。
 
Android.mk给变量赋值，同时用的`:=`和`=`，他们分别代表什么意思呢？ 

`:=` 的意思是，它右边赋得值如果是变量，只能使用在这条语句之前定义好的，而不能使用本条语句之后定义的变量； 
`=`，当它的右边赋值是变量时，这个变量的定义在本条语句之前或之后都可以

## Application.mk文件

### 变量解释

	APP_PROJECT_PATH 此变量值必须是你工程根目录的绝对路径。这用于指定JNI生成的.so文件安装路径或拷贝路径.

注：此变量对于第一种方法是可选了，但对于第二种方法却是必须的.

	APP_ABI 在默认情况下，NDK会使用'armeabi' ABI 来生成二进制机器码，这是基于ARMv5TE的浮点运算CPU，这可以通过使用此变量来选项不同的ABI(Application Binary Interface).

例如:支持基于armv7 FPU指令集的设备:
	APP_ABI := armeabi-v7a
支持IA-32指令集：
	APP_ABI := x86
同时支持三种：
	APP_ABI := armeabi armeabi-v7a x86
从NDK-r7版本后，同时支持三种还可以这样写：
	APP_ABI := all

## JNINativeMethod

JNINativeMethod 结构体的官方定义

	typedef struct {  
	  
	const char* name;  
	const char* signature;  
	void* fnPtr;  
	} JNINativeMethod;  
第一个变量name是Java中函数的名字。
第二个变量signature，用字符串是描述了Java中函数的参数和返回值
第三个变量fnPtr是函数指针，指向native函数。前面都要接 (void *)
第一个变量与第三个变量是对应的，一个是java层方法名，对应着第三个参数的native方法名字

## NDK编译问题

### 不要使用最新的API平台来编译so库

例如：使用android-21平台版本编译的.so文件运行在android-15的设备上，有可能会出现`UnsatisfiedLinkError`，`dlopen: failed`以及其他类型的crash或者低下的性能。

### 需要为每个支持的CPU架构提供对应的so文件,否则会报错

### 可以选择在应用市场上传指定ABI版本的APK，来减少apk的大小。

生成不同ABI版本的APK可以在build.gradle中如下配置：

```
android {
    ... 
    splits {
        abi {
            enable true
            reset()
            include 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a' //select ABIs to build APKs for
            universalApk true //generate an additional APK that contains all the ABIs
        }
    }

    // map for the version code
    project.ext.versionCodes = ['armeabi': 1, 'armeabi-v7a': 2, 'arm64-v8a': 3, 'mips': 5, 'mips64': 6, 'x86': 8, 'x86_64': 9]

    android.applicationVariants.all { variant ->
        // assign different version code for each output
        variant.outputs.each { output ->
            output.versionCodeOverride =
                    project.ext.versionCodes.get(output.getFilter(com.android.build.OutputFile.ABI), 0) * 1000000 + android.defaultConfig.versionCode
        }
    }
 }

```

### 只提供armeabi架构的.so文件而忽略其他ABIs的

所有的x86/x86_64/armeabi-v7a/arm64-v8a设备都支持armeabi架构的.so文件，因此似乎移除其他ABIs的.so文件是一个减少APK大小的好技巧。但事实上并不是：这不只影响到函数库的性能和兼容性。