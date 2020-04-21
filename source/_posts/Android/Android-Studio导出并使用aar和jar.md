---
title: Android Studio导出并使用aar和jar
categories: Android
tags:
  - 打包
  - Android
  - gradle
  - jar
  - aar
description: '使用Android Studio导出aar文件和jar包,并使用它们.'
abbrlink: 6aaba340
date: 2016-06-27 16:58:34
---

今天想往项目中导入Zxing,不知道怎么就想到不使用library的方式而使用包的方式来导入,然后就有了下面的东西了.

## AAR
AAR是Android Library的一种新的二进制分发格式，它把资源也一起打包，这样一来图片和布局资源文件也能够被同时分发。同时AAR还可以包含jar包.

### 1.生成AAR
当我们运行工程后,该工程的/build/outputs/arr下包含Android Studio自动打包的AAR文件

![QQ截图20160627170457.png](https://ooo.0o0.ooo/2016/06/27/5770ed996110e.png)

### 2.使用AAR

将AAR文件拷贝到项目的libs目录下,然后在项目的build.gradle文件中配置即可

	apply plugin: 'com.android.application'
	android {
	    compileSdkVersion 23
	    buildToolsVersion "23.0.3"
	
	    defaultConfig {
	        applicationId "com.qtparking.btool_as"
	        minSdkVersion 11
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
	}
	
	// 添加arr文件的引用  还要在dependencies里面添加引用
	repositories {
	    flatDir {
	        dirs 'libs'
	    }
	}
	
	dependencies {
	    compile fileTree(dir: 'libs', include: ['*.jar'])
	    testCompile 'junit:junit:4.12'
	    compile 'com.android.support:appcompat-v7:23.3.0'
	    compile 'com.android.support:design:23.3.0'
		// 添加AAR文件的引用,文件名字为zxing,类型为aar
	    compile(name:'zxing', ext:'aar')
	}

## Jar

Jar包应该不陌生了,但是如何使用Android Studio导出jar包呢?

### 1.配置gradle任务
在需要打包成jar包的项目的build.gradle添加gradle任务:

	apply plugin: 'com.android.library'
	android {
	    compileSdkVersion 21
	    buildToolsVersion "21.1.2"
	
	    defaultConfig {
	        minSdkVersion 9
	        targetSdkVersion 21
	        testApplicationId "com.android.volley.tests"
	        testInstrumentationRunner "android.test.InstrumentationTestRunner"
	    }
	
	    lintOptions {
	        abortOnError false
	    }
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
	        }
	    }
	}
	
	task makeJar(type: Copy) {
	    delete 'build/libs/lfVolley.jar'	//删除旧的jar包
	    from('build/intermediates/bundles/release/')	// 文件来自哪里
	    into('build/libs/')		// 生成的jar包存放目录
	    include('classes.jar')
	    rename ('classes.jar', 'lfVolley.jar')	// 重命名成我们的jar包
	}
	makeJar.dependsOn(build)
	
	dependencies {
	    compile 'com.android.support:support-v4:21.0.3'
	}

### 2.执行gradle任务
有的文章中推荐我们执行:`./gradview makeJar`.但是,我个人测试是无法成功的.
可以通过Android Studio来执行gradle任务:

#### 2.1在Gradle projects中寻找需要打包的Module:

![QQ截图20160627171318.png](https://ooo.0o0.ooo/2016/06/27/5770efb6a04b6.png)

#### 2.2找到makeJar的gradle任务并双击执行:

![QQ截图20160627171331.png](https://ooo.0o0.ooo/2016/06/27/5770efb697f2a.png)

#### 2.3gradle任务执行完成后显示如下:

![QQ截图20160627171348.png](https://ooo.0o0.ooo/2016/06/27/5770efb6b688f.png)

至此,就完成了jar包的生成,可以直接拷贝到其他项目中使用了.

TODO:
1.有一个小想法,这个是不是和插件化有点关系呢,从来没有了解过插件化,还不知道怎么弄呢.后面有时间再说吧.
2.后面研究下如何导出混淆的aar和jar,现在没有时间研究呢还.