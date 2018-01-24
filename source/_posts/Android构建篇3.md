---
title: Android 构建篇(3)：Gradle 依赖管理
date: 2017-08-16
tags: 
categories: Android
---

本文阐述 Android 使用 Gradle 构建时的依赖管理，包括
- 基础知识
- 如何让依赖的 Library 不打包到最终的 apk/jar/aar ？
- 如何让多个 module 依赖同一个本地 jar/aar 文件？
- 如何解决多个 module 依赖配置版本冲突？

<!-- more -->

## 基础知识

### 1. 什么是依赖
大多数项目不是完全独立的，Gradle 得了解项目构建或运行时需要的传入的文件，这些传入的文件构成项目的 dependencies(依赖项)
我们可以告知 Gradle 项目的依赖关系，Gradle 会执行 dependency resolution(依赖解析)，可能传入的文件需要从远程仓库下载，也可能在本地文件系统中，或者一个构建依赖另外一个构建(多项目构建)

### 2. 声明依赖
```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.1.1'
    testCompile 'junit:junit:4.12'
}
```

### 3. 获取依赖列表
```
$ ./gradlew app:dependencies --configuration compile
To honour the JVM settings for this build a new JVM will be forked. Please consider using the daemon: https://docs.gradle.org/2.14.1/userguide/gradle_daemon.html.
Incremental java compilation is an incubating feature.
:app:dependencies

------------------------------------------------------------
Project :app
------------------------------------------------------------

compile - Classpath for compiling the main sources.
\--- com.android.support:appcompat-v7:25.1.1
     +--- com.android.support:support-annotations:25.1.1
     +--- com.android.support:support-v4:25.1.1
     |    +--- com.android.support:support-compat:25.1.1
     |    |    \--- com.android.support:support-annotations:25.1.1
     |    +--- com.android.support:support-media-compat:25.1.1
     |    |    +--- com.android.support:support-annotations:25.1.1
     |    |    \--- com.android.support:support-compat:25.1.1 (*)
     |    +--- com.android.support:support-core-utils:25.1.1
     |    |    +--- com.android.support:support-annotations:25.1.1
     |    |    \--- com.android.support:support-compat:25.1.1 (*)
     |    +--- com.android.support:support-core-ui:25.1.1
     |    |    +--- com.android.support:support-annotations:25.1.1
     |    |    \--- com.android.support:support-compat:25.1.1 (*)
     |    \--- com.android.support:support-fragment:25.1.1
     |         +--- com.android.support:support-compat:25.1.1 (*)
     |         +--- com.android.support:support-media-compat:25.1.1 (*)
     |         +--- com.android.support:support-core-ui:25.1.1 (*)
     |         \--- com.android.support:support-core-utils:25.1.1 (*)
     +--- com.android.support:support-vector-drawable:25.1.1
     |    +--- com.android.support:support-annotations:25.1.1
     |    \--- com.android.support:support-compat:25.1.1 (*)
     \--- com.android.support:animated-vector-drawable:25.1.1
          \--- com.android.support:support-vector-drawable:25.1.1 (*)

(*) - dependencies omitted (listed previously)

BUILD SUCCESSFUL
```

### 4. 依赖配置
(1) compile
用于编译源代码
(2) runtime
用于运行时被生成的类使用，默认的，包括 compile
(3) testCompile
用于编译测试代码，默认的，包括 compile、runtime
(4) testRuntime
用于运行测试，默认的，包括 compile、runtime、testCompile
更多信息参考 [Dependency management](https://docs.gradle.org/4.0/userguide/java_plugin.html#tab:configurations)


## 如何让依赖的 Library 不打包到最终的 apk/jar/aar ？
compile 会将依赖库打包进 apk/jar/aar，若无需打包，则使用 provided
$projectDir/build.gradle
```
dependencies {
    provided com.android.support:appcompat-v7:25.1.1
    ...
}
```


## 如何让多个 module 依赖同一个本地 jar/aar 文件？
将 jar/aar 文件放到 $rootProject.projectDir/libs
$projectDir/build.gradle
```
dependencies {
    compile files("$rootProject.projectDir/libs/wup-1.0.0-SNAPSHOT.jar")
    ...
}
```


## 如何解决多个 module 依赖配置版本冲突？
示例：从下图可看到 app module 依赖 demo_library module，二者 Android SDK 版本配置、依赖库 com.android.support:appcompat 版本配置均不一致
![依赖管理 - 版本冲突](https://raw.githubusercontent.com/yuting-lin/hexo.github.io/master/source/_posts/Android%E6%9E%84%E5%BB%BA%E7%AF%87/%E7%89%88%E6%9C%AC%E5%86%B2%E7%AA%81.png)

### 解决方案(一): Force
强制配置该 module 依赖库的版本号
```
...
configurations.all {
   resolutionStrategy {
       force 'org.hamcrest:hamcrest-core:1.3'
   }
}
```
```
$ ./gradlew app:dependencies --configuration compile
To honour the JVM settings for this build a new JVM will be forked. Please consider using the daemon: https://docs.gradle.org/2.14.1/userguide/gradle_daemon.html.
Incremental java compilation is an incubating feature.
:app:dependencies

------------------------------------------------------------
Project :app
------------------------------------------------------------

compile - Classpath for compiling the main sources.
+--- com.android.support:appcompat-v7:24.2.1
|    +--- com.android.support:support-v4:24.2.1
|    |    +--- com.android.support:support-compat:24.2.1
|    |    |    \--- com.android.support:support-annotations:24.2.1
|    |    +--- com.android.support:support-media-compat:24.2.1
|    |    |    \--- com.android.support:support-compat:24.2.1 (*)
|    |    +--- com.android.support:support-core-utils:24.2.1
|    |    |    \--- com.android.support:support-compat:24.2.1 (*)
|    |    +--- com.android.support:support-core-ui:24.2.1
|    |    |    \--- com.android.support:support-compat:24.2.1 (*)
|    |    \--- com.android.support:support-fragment:24.2.1
|    |         +--- com.android.support:support-compat:24.2.1 (*)
|    |         +--- com.android.support:support-media-compat:24.2.1 (*)
|    |         +--- com.android.support:support-core-ui:24.2.1 (*)
|    |         \--- com.android.support:support-core-utils:24.2.1 (*)
|    +--- com.android.support:support-vector-drawable:24.2.1
|    |    \--- com.android.support:support-compat:24.2.1 (*)
|    \--- com.android.support:animated-vector-drawable:24.2.1
|         \--- com.android.support:support-vector-drawable:24.2.1 (*)
\--- project :demo_library
     \--- com.android.support:appcompat-v7:25.1.1 -> 24.2.1 (*)

(*) - dependencies omitted (listed previously)

BUILD SUCCESSFUL
```

### 解决方案(二)：统一配置
通过 Extra Properties 统一配置
$rootProject.projectDir/config.gradle
```
ext {
    android = [
            versionCode         : 1,
            versionName         : "1.0.0",
            compileSdkVersion   : 24,
            buildToolsVersion   : "24.0.3",
            minSdkVersion       : 15,
            targetSdkVersion    : 9,
    ]

    supportLibrary = '24.2.1'
    supportDependencies = [
            appCompat : "com.android.support:appcompat-v7:${supportLibrary}",
    ]

    aspectjLibrary = '1.8.10'
    aspectjDependencies = [
            tools : "org.aspectj:aspectjtools:${aspectjLibrary}",
            runtime : "org.aspectj:aspectjrt:${aspectjLibrary}",
    ]
}
```
$projectDir/build.gradle
```
...
android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.android.buildToolsVersion

    defaultConfig {
        minSdkVersion rootProject.ext.android.minSdkVersion
        targetSdkVersion rootProject.ext.android.targetSdkVersion
        versionCode rootProject.ext.android.versionCode
        versionName rootProject.ext.android.versionName

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile supportDependencies.appCompat
    provided fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    testCompile 'junit:junit:4.12'
}
```


## 参考
[Dependency management](https://docs.gradle.org/4.0/userguide/java_plugin.html#tab:configurations)
[Gradle 依赖项学习总结](http://www.paincker.com/gradle-dependencies)