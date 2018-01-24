---
title: Android 构建篇(4)：合并多个 aar/jar 文件
date: 2017-08-23
tags: 
categories: Android
---

本文阐述合并多个 aar/jar 文件的背景、方案和原理
- 使用 jar 命令
- android-fat-aar
- android-fat-aar 实现原理

<!-- more -->

## 背景
为了让代码解耦，提高复用性，我们将 SDK Project 拆解成几个 module，业务 module 依赖基础 module
但是每个 module 都会单独输出 aar/jar 文件，并不会把依赖的 module 代码打包进去，然而，最后发布时我们却只能提供一个单独的 aar/jar 文件
于是，我们需要合并多个 aar/jar 文件至同一个文件


## 解决方案(一): 使用 jar 命令
如果只有代码，没有资源文件，那么可以使用 jar 命令 完成合并 aar/jar 文件
```
$ jar
用法: jar {ctxui}[vfmn0PMe] [jar-file] [manifest-file] [entry-point] [-C dir] files ...
选项:
    -c  创建新档案
    -t  列出档案目录
    -x  从档案中提取指定的 (或所有) 文件
    -u  更新现有档案
    -v  在标准输出中生成详细输出
    -f  指定档案文件名
    -m  包含指定清单文件中的清单信息
    -n  创建新档案后执行 Pack200 规范化
    -e  为捆绑到可执行 jar 文件的独立应用程序
        指定应用程序入口点
    -0  仅存储; 不使用任何 ZIP 压缩
    -P  保留文件名中的前导 '/' (绝对路径) 和 ".." (父目录) 组件
    -M  不创建条目的清单文件
    -i  为指定的 jar 文件生成索引信息
    -C  更改为指定的目录并包含以下文件
如果任何文件为目录, 则对其进行递归处理。
清单文件名, 档案文件名和入口点名称的指定顺序
与 'm', 'f' 和 'e' 标记的指定顺序相同。

示例 1: 将两个类文件归档到一个名为 classes.jar 的档案中:
       jar cvf classes.jar Foo.class Bar.class
示例 2: 使用现有的清单文件 'mymanifest' 并
           将 foo/ 目录中的所有文件归档到 'classes.jar' 中:
       jar cvfm classes.jar mymanifest -C foo/ .
```
(1) 将所有 jar 包的 classes.jar 解压到某个目录
```
jar -xvf XXX/classes.jar
```
(2) 将解压后的所有 class 文件重新压缩为一个单独的 classes.jar
```
jar -cvfM YYY.jar
```


## 解决方案(二): android-fat-aar
[android-fat-aar](https://github.com/adwiv/android-fat-aar)
### 1. 配置
(1) apply gradle 文件
$projectDir/build.gradle
```
apply from : 'https://raw.githubusercontent.com/adwiv/android-fat-aar/master/fat-aar.gradle'
```
(2) 指定要潜入 aar/jar 的文件
使用 embedded 替换 compile
```
dependencies {
    ...
    embedded project(':tangram')
}
```

### 2. android-fat-aar 不支持的部分
(1) Manifest placeholders
(2) AIDL 文件
(3) 多个 build types


## android-fat-aar 实现原理

### 1. aar 文件是什么? 需要合并哪些内容？
引用 [AAR 文件详解](https://developer.android.com/studio/projects/android-library.html#aar-contents)
>AAR 文件的文件扩展名为 .aar，Maven 工件类型也应当是 aar。文件本身是一个包含以下强制性条目的 zip 文件：
 /AndroidManifest.xml
 /classes.jar
 /res/
 /R.txt
 此外，AAR 文件可能包含以下可选条目中的一个或多个：
 /assets/
 /libs/名称.jar
 /jni/abi 名称/名称.so（其中 abi 名称 是 Android 支持的 ABI 之一）
 /proguard.txt
 /lint.jar

### 2. 原理
本质上是将多个内容文件拷贝到一起，再合并相同名称的文件
#### 2.1 如何确认需要合并的 module?
通过在 dependencies 中使用 embedded 替换 compile，将多个 module 关联起来
```
dependencies {
    ...
    embedded project(':tangram')
}
```
#### 2.2 如何找到需要合并的 module 的内容文件路径
每个 module 在编译时，都会生成 build 文件夹，里面存放 aar 所需的多个内容文件
将需合并的内容文件存放至 exploded-aar 文件夹
```
ext.build_dir = buildDir.path.replace(File.separator, '/');

ext.exploded_aar_dir = "$build_dir/intermediates/exploded-aar";
ext.classs_release_dir = "$build_dir/intermediates/classes/release";
ext.bundle_release_dir = "$build_dir/intermediates/bundles/release";
ext.manifest_aaapt_dir = "$build_dir/intermediates/manifests/aapt/release";
ext.generated_rsrc_dir = "$build_dir/generated/source/r/release";

ext.base_r2x_dir = "$build_dir/fat-aar/release/";
```

#### 2.3 什么时候执行合并？
示例：app module embedded base module
```
$ ./gradlew assembleRelease
To honour the JVM settings for this build a new JVM will be forked. Please consider using the daemon: https://docs.gradle.org/2.14.1/userguide/gradle_daemon.html.
Incremental java compilation is an incubating feature.
:app:preBuild UP-TO-DATE
:app:preReleaseBuild UP-TO-DATE
:app:checkReleaseManifest
:app:preDebugAndroidTestBuild UP-TO-DATE
:app:preDebugBuild UP-TO-DATE
:app:preDebugUnitTestBuild UP-TO-DATE
:app:preReleaseUnitTestBuild UP-TO-DATE
:base:preBuild UP-TO-DATE
:base:preReleaseBuild UP-TO-DATE
:base:checkReleaseManifest
:base:preDebugAndroidTestBuild UP-TO-DATE
:base:preDebugBuild UP-TO-DATE
:base:preDebugUnitTestBuild UP-TO-DATE
:base:preReleaseUnitTestBuild UP-TO-DATE
:base:prepareComAndroidSupportAnimatedVectorDrawable2511Library
:base:prepareComAndroidSupportAppcompatV72511Library
:base:prepareComAndroidSupportSupportCompat2511Library
:base:prepareComAndroidSupportSupportCoreUi2511Library
:base:prepareComAndroidSupportSupportCoreUtils2511Library
:base:prepareComAndroidSupportSupportFragment2511Library
:base:prepareComAndroidSupportSupportMediaCompat2511Library
:base:prepareComAndroidSupportSupportV42511Library
:base:prepareComAndroidSupportSupportVectorDrawable2511Library
:base:prepareReleaseDependencies
:base:compileReleaseAidl
:base:compileReleaseNdk UP-TO-DATE
:base:compileLint
:base:copyReleaseLint UP-TO-DATE
:base:compileReleaseRenderscript
:base:generateReleaseBuildConfig
:base:generateReleaseResValues
:base:generateReleaseResources
:base:mergeReleaseResources
:base:processReleaseManifest
:base:processReleaseResources
:base:generateReleaseSources
:base:incrementalReleaseJavaCompilationSafeguard
:base:compileReleaseJavaWithJavac
:base:compileReleaseJavaWithJavac - is not incremental (e.g. outputs have changed, no previous execution, etc.).
:base:extractReleaseAnnotations
:base:mergeReleaseShaders
:base:compileReleaseShaders
:base:generateReleaseAssets
:base:mergeReleaseAssets
:base:mergeReleaseProguardFiles UP-TO-DATE
:base:packageReleaseRenderscript UP-TO-DATE
:base:packageReleaseResources
:base:processReleaseJavaRes UP-TO-DATE
:base:transformResourcesWithMergeJavaResForRelease
:base:transformClassesAndResourcesWithSyncLibJarsForRelease
:base:mergeReleaseJniLibFolders
:base:transformNative_libsWithMergeJniLibsForRelease
:base:transformNative_libsWithSyncJniLibsForRelease
:base:bundleRelease
:app:prepareComAndroidSupportAnimatedVectorDrawable2511Library
:app:prepareComAndroidSupportAppcompatV72511Library
:app:prepareComAndroidSupportSupportCompat2511Library
:app:prepareComAndroidSupportSupportCoreUi2511Library
:app:prepareComAndroidSupportSupportCoreUtils2511Library
:app:prepareComAndroidSupportSupportFragment2511Library
:app:prepareComAndroidSupportSupportMediaCompat2511Library
:app:prepareComAndroidSupportSupportV42511Library
:app:prepareComAndroidSupportSupportVectorDrawable2511Library
:app:prepareDemoBaseUnspecifiedLibrary
:app:prepareReleaseDependencies
:app:compileReleaseAidl
:app:compileReleaseNdk UP-TO-DATE
:app:compileLint
:app:copyReleaseLint UP-TO-DATE
:app:compileReleaseRenderscript
:app:generateReleaseResValues
:app:generateReleaseResources
:app:mergeReleaseResources
:app:processReleaseManifest
:app:processReleaseResources
:app:generateRJava
Running FAT-AAR Task :generateRJava
:app:generateReleaseBuildConfig
:app:generateReleaseSources
:app:incrementalReleaseJavaCompilationSafeguard
:app:compileReleaseJavaWithJavac
:app:compileReleaseJavaWithJavac - is not incremental (e.g. outputs have changed, no previous execution, etc.).
:app:collectRClass
:app:embedRClass
:app:embedJavaJars
Running FAT-AAR Task :embedJavaJars
:app:mergeReleaseShaders
:app:compileReleaseShaders
:app:embedAssets
Running FAT-AAR Task :embedAssets
:app:generateReleaseAssets
:app:mergeReleaseJniLibFolders
:app:transformNative_libsWithMergeJniLibsForRelease
:app:transformNative_libsWithSyncJniLibsForRelease
:app:embedJniLibs
Running FAT-AAR Task :embedJniLibs
======= Copying JNI from /Users/yutinglin/Documents/src/code/android/demo/app/build/intermediates/exploded-aar/demo/base/unspecified/jni
:app:embedManifests
Running FAT-AAR Task :embedManifests
========== INFO : Loading library manifest /Users/yutinglin/Documents/src/code/android/demo/app/build/intermediates/exploded-aar/demo/base/unspecified/AndroidManifest.xml
========== INFO : Merging main manifest /Users/yutinglin/Documents/src/code/android/demo/app/build/intermediates/bundles/release/AndroidManifest.orig.xml

========== INFO : Merging library manifest /Users/yutinglin/Documents/src/code/android/demo/app/build/intermediates/exploded-aar/demo/base/unspecified/AndroidManifest.xml
========== INFO : Merging manifest with lower AndroidManifest.xml:2:1-17:12
========== INFO : Merging uses-sdk with lower AndroidManifest.xml:7:5-9:41
========== INFO : Merging application with lower AndroidManifest.xml:11:5-15:19
========== INFO : Merging result:SUCCESS
========== INFO : Merged manifest saved to /Users/yutinglin/Documents/src/code/android/demo/app/build/intermediates/bundles/release/AndroidManifest.xml
========== INFO : Merged aapt safe manifest saved to /Users/yutinglin/Documents/src/code/android/demo/app/build/intermediates/manifests/aapt/release/AndroidManifest.xml
:app:extractReleaseAnnotations
:app:mergeReleaseAssets
:app:mergeReleaseProguardFiles UP-TO-DATE
:app:packageReleaseRenderscript UP-TO-DATE
:app:embedProguard
Running FAT-AAR Task :embedProguard
:app:embedLibraryResources
Running FAT-AAR Task :embedLibraryResources
:app:packageReleaseResources
:app:processReleaseJavaRes UP-TO-DATE
:app:transformResourcesWithMergeJavaResForRelease
:app:transformClassesAndResourcesWithSyncLibJarsForRelease
:app:bundleRelease
:app:compileReleaseSources
:app:assembleRelease
:base:compileReleaseSources
:base:assembleRelease

BUILD SUCCESSFUL

Total time: 9.887 secs

```
从日志可看出 task 执行顺序
![android-fat-aar](https://raw.githubusercontent.com/yuting-lin/hexo.github.io/master/source/_posts/Android%E6%9E%84%E5%BB%BA%E7%AF%87/android-fat-aar.png)


## 参考
[Android Studio 分模块自动化构建实战](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0620/3094.html)
[android-fat-aar](https://github.com/adwiv/android-fat-aar)
[AAR 文件详解](https://developer.android.com/studio/projects/android-library.html#aar-contents)
[Gradle学习笔记(四)-- fat-aar.gradle解析](http://www.jianshu.com/p/f88ff677ac95)
[[Android]多module合成单一module技巧](http://www.jianshu.com/p/9b022951571c)